# Uploading over MCP (optional)

Without this, your work ends at a folder: the seller uploads it in the browser
and you never learn which draft or variant ids it became. With the seller MCP
server connected you can dry-run, create and update the draft yourself, and keep
the ids in `webto.lock.json`.

**It is optional.** If the `webto` MCP server is not connected, deliver the
folder and point the seller at Templates → Upload Bundle, exactly as before.
Never ask the seller to paste a token into the chat.

## Setup (the seller does this once)

1. Dashboard → Settings → Security → **Token akses (agen AI)** → create a token
   ("Baca & tulis draft"). It is shown once.
2. Put it in the environment as `WEBTO_TOKEN`, and add to the project's
   `.mcp.json`:

```json
{
  "mcpServers": {
    "webto": {
      "type": "http",
      "url": "https://api.webto.id/mcp",
      "headers": { "Authorization": "Bearer ${WEBTO_TOKEN}" }
    }
  }
}
```

The token never goes in the file itself — `.mcp.json` gets committed.

## What the tools can and cannot do

| Tool | Does |
|---|---|
| `list_template_drafts` | The seller's drafts: `id`, subdomain, listing state, `listing.snapshot` (`fresh` / `stale` / `unknown`) and the listing's public `catalogUrl` / `livePreviewUrl` |
| `list_my_variants` | Variants the seller owns: `id`, `bundleKey`, status, `originSiteId` |
| `check_template_source` | Readiness report for one draft |
| `export_template_bundle` | The draft as a bundle folder again, plus `lock` |
| `create_template_draft` | Upload a bundle as a NEW draft |
| `update_template_draft` | Update an existing draft from a bundle |
| `begin_asset_upload` | Start an upload session for `asset:<filename>` images |
| `get_preview_url` | Fresh signed preview links for one draft (`previewUrl`, `previewPages`, `expiresAt`) |
| `refresh_listing_snapshot` | Put the draft's current version on sale in its existing listing (scope `listings:write`; see below) |
| `get_listing_urls` | A listing's two public addresses (`catalogUrl`, `livePreviewUrl`), its status and snapshot freshness |

There is deliberately no tool that creates a listing, sets a price or publishes.
Those stay with the seller in the dashboard; say so instead of looking for a way.

## The loop

1. **Run the CLI first** (workflow step 8). The server's dry run is rate
   limited (30 per 10 minutes); `variant-check` is not. Arrive with files that
   already pass locally.
2. **Images.** If the bundle ships `assets/`, call `begin_asset_upload`, then
   POST each file to the returned `url` yourself:

   ```bash
   curl -X POST "$URL" -H "Authorization: Bearer $WEBTO_TOKEN" \
     -F "uploadSessionId=$SESSION" -F "file=@assets/hero-photo.png"
   ```

   Never put image bytes (base64) in a tool call. Pass
   `assets: { uploadSessionId }` and the `assetManifest` to the write tool.
3. **Dry run.** Call `create_template_draft` (or `update_template_draft` with
   `targetSiteId`) WITHOUT `confirm`. Nothing is written. Read the whole report:
   per-variant lint, `manifestErrors` addressed by JSON path, `warnings`.
4. **Fix and repeat** until `ok: true`.
5. **Show the seller what will happen, then write.** Call again with
   `confirm: true`. For an update, first relay `manifestDiff` — see below.
   If the dry run carries `nameTaken`, ask first — see
   [A taken listing name](#a-taken-listing-name).
6. **Save the lock.** A real write returns `lock`; write it verbatim to
   `webto.lock.json` beside `template.json`, replacing the old file. It is the
   complete lock — every variant of the draft, `version` filled, the revision
   after the write — exactly what `export_template_bundle` would return at that
   moment, so it is safe to save as-is after any write, patch mode included.
   What this write moved is in `changed`, beside the lock (see
   [`webto.lock.json`](#webtolockjson)). Writes are limited to 10 per
   10 minutes; you should need one.
7. **Look at the draft before you report.** The write returns `previewUrl`
   (home page) and `previewPages` (`[{ title, slug, url }]`, one signed link
   per page, valid 12 hours). Open every page in a browser at 390, 768 and
   1280 px — the draft as the platform renders it, with its real theme, not
   the CLI preview — and compare against the source. Fix, update, look again.
   Without a browser, tell the seller this check was not done. The links
   expire after 12 hours; `get_preview_url` with the draft id gives fresh
   ones. A dry run of `update_template_draft` carries `savedPreview`: the draft
   as it is saved NOW, without your change.

## A taken listing name

The `listing.name` (else `listing.title`) you propose becomes the listing's
preview address, `tpl-<name>.wpage.id`, when the seller later lists the
template. That address is **permanent**: renaming the template afterwards does
not move it. When another site already holds it, the dry run says so:

```json
"nameTaken": { "name": "Rumah Ceria", "previewSubdomain": "tpl-rumah-ceria-2", "permanent": true }
```

**Ask the seller** — do not choose for them:

- **Keep the name** → call again with `confirm: true` **and**
  `acceptNameSuffix: true`. The listing will live at the suffixed address.
- **Rename** → change `listing.name` in `template.json` and dry-run again.

A confirm without `acceptNameSuffix` is refused with the same message
(`ok: false`, nothing written). No `nameTaken` → nothing to ask. An update
whose proposal names what is already stored, or a draft that already has a
listing (its address is fixed), is never flagged. The dashboard asks the seller
again, with the address current at that moment, when they create the listing.

## Updating: export before you touch anything

`update_template_draft` with a `manifest` REPLACES the draft's pages, sections
and theme. The seller may have edited the draft in the Site Editor since the
folder on disk was written — a font, a heading, a whole new section — and an
upload from a stale `template.json` silently destroys that work.

So, to revise an existing draft:

1. `export_template_bundle` → overwrite the local folder with what it returns
   (`manifest` → `template.json`, each `files[]` entry at its `path`, `lock` →
   `webto.lock.json`). Check `notCarried` and tell the seller what it lists.
2. Make your changes on top of that.
3. Dry run, sending `baseRevision` = `lock.revision`. `manifestDiff.lines` is
   what the upload would change in the draft; every line should be one you
   intended.
4. To change section CODE only, omit `manifest`: patch mode matches variants by
   key and leaves the site structure and the seller's edits untouched. No
   `baseRevision` needed — nothing structural is rewritten.

### `baseRevision`: the server checks that you exported

`lock.revision` is a fingerprint of the draft (theme, language, pages, sections
with their styles and visibility, chrome) at the moment of the export. A lock
from before 2026-09-28 says `r1-…` and is refused once with its own message:
export again. An update that carries a `manifest` MUST send it
back as `baseRevision`:

- **Missing** → refused. You skipped the export; do it.
- **Stale** (`revisionConflict: true`) → refused. The seller edited the draft in
  the Site Editor, or uploaded from elsewhere, after your export. Export again,
  re-apply YOUR change on top of the new files, dry-run again. Do not try to
  "merge" by resending your old folder — that is the overwrite this prevents.
- A successful write returns a fresh `lock` with the NEW revision. Save it, or
  your own next update is refused as stale. This applies to structural writes
  (with a `manifest`): patch mode does not change `revision`, because section
  code is not part of the fingerprint — an unchanged revision after a patch is
  correct. Check `changed` for the new variant versions instead.

This is a refusal, not a warning, for a token. There is no override flag.

### Theme: send it whole, or not at all

`manifest.theme` is a total replacement: any key you leave out of a theme you
DO send goes back to the platform default (the dry run lists each one, e.g.
`Tema radius: 12 → 8`). To change pages or content without touching the theme,
omit the `theme` block entirely — the draft's theme is then left exactly as it
is, and the report says so. An export always contains the full theme, so
working on top of an export is safe either way.

## After an update: the listing still sells the OLD version

A listed template is sold as a **snapshot** — a locked copy taken when the
listing was published or last refreshed. Updating the draft does NOT change
what buyers get, and nothing refreshes the snapshot on its own (a half-finished
save would go on sale, and a refresh is refused while a variant awaits review).

So when a confirmed `update_template_draft` returns a `listing` block:

```json
"listing": { "id": "…", "status": "active", "snapshot": "stale", "sourceUpdatedAt": "…",
             "catalogUrl": "https://webto.id/templates/…", "livePreviewUrl": "https://tpl-….wpage.id", "next": "…" }
```

- `snapshot: "stale"` — the draft changed since the snapshot. **Tell the seller
  and ASK** whether this version should go on sale now. Do not decide for them:
  they may want to finish more changes, or wait for a variant's review.
- `snapshot: "unknown"` — the snapshot predates version tracking (or its
  source is gone). Say so; one refresh starts tracking.
- `snapshot: "fresh"` — nothing to do.
- No `listing` key — the draft has no listing; nothing to do.

Tell them too that **buyers who already bought are unaffected** either way —
their sites are separate copies.

Only after an explicit yes:

1. `refresh_listing_snapshot` with `listingId`, WITHOUT `confirm` — the dry run.
   It returns `changes.lines` (what the refresh would change: pages added or
   removed, which pages' sections differ, theme, navbar/footer) and `blockers`.
   If `blockers` is not empty (usually a variant still in review), relay them
   and stop: the refresh would be refused.
   When it helps the seller decide, put the draft's `previewUrl` (the version
   being worked on) next to `livePreviewUrl` (the version on sale now) and let
   them compare.
2. Same call with `confirm: true`. The result says `snapshot: "fresh"` and
   carries `catalogUrl` + `livePreviewUrl`: open `livePreviewUrl` to check it,
   then give the seller both.

**Never** call it with `confirm: true` without the seller's explicit approval of
selling this version — not even when the dry run is clean, and not in bulk
because "they asked you to update the templates". Updating a draft and putting
it on sale are two different decisions.

It needs the **`listings:write`** scope, which `templates:write` does NOT
include: a token made only for drafts cannot touch what buyers get. Without it
the tool is not in `tools/list`; tell the seller they can refresh from the
dashboard (Marketplace → Listing → **Perbarui snapshot**, or **Perbarui semua
yang tertinggal**), or create a token with **Listing marketplace → Perbarui
snapshot** ticked.

## The listing's public URLs

A listing has two public addresses, both without a token and without expiry:

- `catalogUrl` — its page in the template catalog (`webto.id/templates/<listing title slug>`).
- `livePreviewUrl` — the template **as sold**: the listing's snapshot
  (`tpl-<template name>.wpage.id`). Not the same as a draft's `previewUrl`, which is
  signed, expires after 12 hours and shows the version being worked on.

They ride on every `listing` block (`list_template_drafts`, a confirmed
`update_template_draft`) and on `refresh_listing_snapshot`; `get_listing_urls`
with a `listingId` returns them on their own (scope `templates:read`). Both are
`null` while the listing is not active — `urlsNote` says why; only the seller
can activate a listing, in the dashboard.

Give the seller both **once their listing is active** and **after every
snapshot refresh**. If `snapshot` is `stale`, say that `livePreviewUrl` still
shows the older version on sale.

## `webto.lock.json`

```json
{
  "lockVersion": 1,
  "siteId": "…",
  "subdomain": "tpl-studio-kalastra",
  "exportedAt": "2026-09-21T10:00:00.000Z",
  "revision": "r2-3f9a1c0b7d2e4a68",
  "variants": { "hero-studio-split": { "variantId": "…", "version": 3, "status": "approved" } }
}
```

It answers "which draft is this folder?" (`siteId` → `targetSiteId`), "what
did the draft look like when I last saw it?" (`revision` → `baseRevision`) and
"has this key been uploaded before, and at which version?".

Every write tool returns the same complete lock an export does, so you never
merge locks by hand. Beside it, a confirmed write returns `changed`: the
variants this write moved, by key.

```json
"changed": { "hero-studio-split": { "from": 2, "to": 3 }, "faq-new": { "from": null, "to": 1 } }
```

- `from: null` — the key is new to this draft (on `create_template_draft`
  every key is).
- `to: null` — the key left the lock: no page or chrome uses it any more.
- A key that is absent did not move. An empty `{}` means no variant changed.
- `changed: null` — the platform could not read the lock before the write. The
  lock itself is still complete; only the comparison is missing.

A variant you patched but that is missing from `changed` was identical to the
stored version (the dry run said "unchanged"). If `lock` is `null` after a
successful write, `next` says so: call `export_template_bundle` and save its
lock. It is not part of an upload — the platform's
uploader steps over it — and it is safe to commit: ids are not secrets.

**Reusing a variant in another template:** a bundle must stay self-contained,
so a raw `u:<variantId>` ref is rejected. Copy the `.astro` (and sample) into
the new bundle unchanged, with the same `name` and `sectionType` in
`variants[]`: a byte-identical source under the same name is recognised and the
existing variant is reused as-is, review status included, instead of a
duplicate being minted. Change one byte and it is a new variant.

## When a call fails

- `401` — token missing, expired or revoked. Ask the seller for a new one.
- `403` — the token lacks the scope (read-only, or no `listings:write` for a
  snapshot refresh), or the account is suspended.
- `429` / `TOO_MANY_REQUESTS` — wait for the stated seconds. Do not loop.
- `isError: true` with a `code` — the platform refused for a stated reason
  (ownership, draft limit, size). Relay the message; do not retry unchanged.
