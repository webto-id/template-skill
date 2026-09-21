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
| `list_template_drafts` | The seller's drafts: `id`, subdomain, listing state |
| `list_my_variants` | Variants the seller owns: `id`, `bundleKey`, status, `originSiteId` |
| `check_template_source` | Readiness report for one draft |
| `export_template_bundle` | The draft as a bundle folder again, plus `lock` |
| `create_template_draft` | Upload a bundle as a NEW draft |
| `update_template_draft` | Update an existing draft from a bundle |
| `begin_asset_upload` | Start an upload session for `asset:<filename>` images |

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
6. **Save the lock.** A real write returns `lock`; write it verbatim to
   `webto.lock.json` beside `template.json`. Writes are limited to 10 per
   10 minutes; you should need one.

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

`lock.revision` is a fingerprint of the draft (theme, pages, sections, chrome)
at the moment of the export. An update that carries a `manifest` MUST send it
back as `baseRevision`:

- **Missing** → refused. You skipped the export; do it.
- **Stale** (`revisionConflict: true`) → refused. The seller edited the draft in
  the Site Editor, or uploaded from elsewhere, after your export. Export again,
  re-apply YOUR change on top of the new files, dry-run again. Do not try to
  "merge" by resending your old folder — that is the overwrite this prevents.
- A successful write returns a fresh `lock` with the NEW revision. Save it, or
  your own next update is refused as stale.

This is a refusal, not a warning, for a token. There is no override flag.

### Theme: send it whole, or not at all

`manifest.theme` is a total replacement: any key you leave out of a theme you
DO send goes back to the platform default (the dry run lists each one, e.g.
`Tema radius: 12 → 8`). To change pages or content without touching the theme,
omit the `theme` block entirely — the draft's theme is then left exactly as it
is, and the report says so. An export always contains the full theme, so
working on top of an export is safe either way.

## `webto.lock.json`

```json
{
  "lockVersion": 1,
  "siteId": "…",
  "subdomain": "tpl-studio-kalastra",
  "exportedAt": "2026-09-21T10:00:00.000Z",
  "revision": "r1-3f9a1c0b7d2e4a68",
  "variants": { "hero-studio-split": { "variantId": "…", "version": 3, "status": "approved" } }
}
```

It answers "which draft is this folder?" (`siteId` → `targetSiteId`), "what
did the draft look like when I last saw it?" (`revision` → `baseRevision`) and
"has this key been uploaded before?". It is not part of an upload — the platform's
uploader steps over it — and it is safe to commit: ids are not secrets.

**Reusing a variant in another template:** a bundle must stay self-contained,
so a raw `u:<variantId>` ref is rejected. Copy the `.astro` (and sample) into
the new bundle unchanged, with the same `name` and `sectionType` in
`variants[]`: a byte-identical source under the same name is recognised and the
existing variant is reused as-is, review status included, instead of a
duplicate being minted. Change one byte and it is a new variant.

## When a call fails

- `401` — token missing, expired or revoked. Ask the seller for a new one.
- `403` — the token is read-only, or the account is suspended.
- `429` / `TOO_MANY_REQUESTS` — wait for the stated seconds. Do not loop.
- `isError: true` with a `code` — the platform refused for a stated reason
  (ownership, draft limit, size). Relay the message; do not retry unchanged.
