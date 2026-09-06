---
name: html-to-webto-template
description: Convert a FULL HTML template (a multi-section landing page or a whole site export, any CSS framework or none) into a webto marketplace TEMPLATE BUNDLE — template.json + sections/*.astro (Webto Variant Format) + *.sample.json — uploadable at webto.id → Templates → "Upload Bundle". Use whenever someone asks to "convert this HTML template to webto", "jadikan template webto", "buat template webto dari HTML/website ini", or wants to sell a full-page design (not just one section) on the webto.id marketplace. REQUIRES the html-to-webto-variant skill as a companion for authoring each .astro file.
---

# HTML template → webto template bundle

A webto TEMPLATE is a full site (theme + navbar/footer chrome + pages of sections) a buyer clones in one click. This skill turns an arbitrary HTML template into the **bundle** webto's uploader accepts:

```
my-template/
  template.json            # the manifest: site name, theme, chrome, pages, variants[] metadata
  sections/hero-1.astro    # one WVF file per authored section (key = file basename)
  hero-1.sample.json       # optional showcase content per section
```

The platform compiles every `.astro` **strict**, validates the whole manifest, creates the variants under the seller's account (auto-submitted for admin review), and assembles a **template draft** the seller lists from the listing editor.

## Read first

- `references/manifest.md` — the exact `template.json` format, `u:@<key>` refs, limits, theme extraction.
- The **html-to-webto-variant** skill — every `.astro` here follows its rules (WVF subset, tokens, zero dead text, `--strict`). Do not guess; read it.

## Workflow

1. **Segment** the source HTML top-to-bottom into sections. Classify each as **chrome** (navbar / announcement banner / footer) or a **page section**.
   **Multi-page source** (a site export with several HTML files — index.html, about.html, …): repeat the segmentation per file; each HTML page becomes one `pages[]` entry (`{ title, slug, sections[] }` — the homepage keeps `slug: ""`, max 8 pages per template). Extract chrome ONCE, from whichever page carries the fullest navbar/footer. Dedupe across pages: when two pages share a section design, author ONE `sections/<key>.astro` and reference the same `u:@<key>` from both with different `content`. The navbar links every page automatically (renderer-injected `pages` context prop) — never hand-write per-page nav links.
2. **Chrome: prefer platform variants; author WVF only when the design demands it.** A close platform `navbar`/`banner`/`footer` match with the source's branding in `content` is zero review-wait and battle-tested. When the source's chrome is genuinely distinctive, chrome MAY be a WVF file (since 2026-09-04) — it reads the renderer-injected context props (`pages`, `linkPrefix`, `currentSlug`, `colorMode`; footer also `footerPages`; `siteName` is an ordinary editable field with an injected fallback) and must keep its root in normal flow (`position: fixed` is a compile error; a `u:` navbar is wrapper-sticky automatically). Site pages appear in the navbar via `pages`; `content.links` is only for EXTRA destinations.
3. **Each page section: reuse or author.** If a platform variant is a close match (see the type/variant catalog in webto's docs — `referensi-schema`, or `templates/CATALOG.md` when working inside the repo), reference it directly with real `content` — fewer files, zero review wait. Otherwise author a WVF file per the variant skill, one `sections/<key>.astro` per distinct design.
4. **Theme extraction.** Pull the source's palette and typography into the manifest's `theme` partial (`colors`, `darkColors`, `fonts`, `radius`, `maxWidth`, `spacing`). Then make every authored section USE the tokens — a template whose sections hardcode the palette defeats the theme (`hardcoded-color` blocks submit anyway).
5. **Content and images.** All visible copy becomes section `content` (the manifest) or field defaults (the .astro). Images: `__IMG__:<english query>` sentinels (the platform resolves them to Unsplash photos at upload) or direct unsplash/pexels URLs. **Never carry the source template's own asset URLs** — you do not hold a licence to redistribute them.
6. **Assemble `template.json`.** Pages reference authored sections as `variant: "u:@<key>"`; the `variants[]` array declares each key's `sectionType`, `name`, and optional `description`/`mood`/`fits`. A bundle must be self-contained: plain `u:<id>` refs are rejected.
7. **Validate per file**: `npx @webto-id/variant-check sections/<key>.astro --type <sectionType> --strict --content <key>.sample.json --out preview.html` — fix every ✖ and ▲, open the preview in both themes.
8. **Revisions**: to iterate on an ALREADY-uploaded template, re-upload the (edited) bundle via the listing editor's **"Update dari Bundle"** button (or /templates/upload?siteId=...). Matching is by each variant's bundle key: identical sources are skipped ("Sama"), changed ones become a new version ("Versi baru" — re-enters review; buyers keep the approved version meanwhile), new keys become new variants. template.json is authoritative on update — it overwrites the draft's structure and theme.
9. **Final gate**: the platform's "Periksa" (dry run) on webto.id → Templates → Upload Bundle. The pairing table there also has a per-section **Pratinjau** button (in-browser compile + render with the sample content) — tell the user to eyeball every section there before creating anything. It re-compiles everything strict and reports manifest problems addressed by JSON path (`pages[1].sections[3]: …`). Nothing is created until the dry run is clean.
9. **Deliver** the folder. After upload: the variants sit in the admin review queue; once approved, the seller publishes the listing from the template's listing editor.

## Hard limits

| What | Limit |
|---|---|
| Authored variants per bundle | 12 |
| Whole upload payload | 2 MB |
| One `.astro` source | 128 KB |
| One `.sample.json` | 16 KB |
| Pages per marketplace template | 8 (one must be the homepage, `slug: ""`) |
| Variants per account (total) | 100 |

## Quality bar

The bundle is merchandise. Every section must survive the buyer's edits: content-shape robustness (1 item … max items, missing optionals), both themes, mobile, zero dead text, `<AddImageButton>` on image lists — all per the variant skill's `design.md`. The template as a whole must read coherently on ONE theme: if two sections disagree about what `primary` means, the conversion is not done.
