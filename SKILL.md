---
name: html-to-webto-template
description: Convert a FULL HTML template (a multi-section landing page or a whole site export, any CSS framework or none) into a webto marketplace TEMPLATE BUNDLE — template.json + sections/*.astro (Webto Variant Format) + *.sample.json — uploadable at webto.id → Templates → "Upload Bundle". Use whenever someone asks to "convert this HTML template to webto", "jadikan template webto", "buat template webto dari HTML/website ini", or wants to sell a full-page design (not just one section) on the webto.id marketplace. REQUIRES the html-to-webto-variant skill as a companion for authoring each .astro file.
---

# HTML template → webto template bundle

A webto TEMPLATE is a full site (theme + navbar/footer chrome + pages of sections) a buyer clones in one click. This skill turns an arbitrary HTML template into the **bundle** webto's uploader accepts:

```
my-template/
  template.json               # the manifest: site name, theme, chrome, pages, variants[] metadata
  sections/hero-1.astro       # one WVF file per authored section (key = file basename)
  samples/hero-1.sample.json  # optional showcase content per section
  assets/hero-photo.png       # optional: seller's own image, referenced as "asset:hero-photo.png"
```

The uploader pairs every file by its FILENAME alone (`hero-1.astro` ↔ `hero-1.sample.json` ↔ `template.json`'s `variants[].key`) — the folder each one sits in is never inspected. The layout above is the recommended convention (keeps a bundle with many sections/samples/assets scannable); a flat folder with everything at root still uploads identically. Pick one and be consistent within a bundle.

The platform compiles every `.astro` **strict**, validates the whole manifest, creates the variants under the seller's account (auto-submitted for admin review), and assembles a **template draft** the seller lists from the listing editor.

## Read first

- `references/manifest.md` — the exact `template.json` format, `u:@<key>` refs, limits, theme extraction.
- The **html-to-webto-variant** skill — every `.astro` here follows its rules (WVF subset, tokens, zero dead text, `--strict`). Do not guess; read it.

## Source: an Astro + Tailwind repo

A source repo (not just rendered HTML) is the best case for parity — the design tokens, component boundaries, and sample data are all there verbatim, and there's a live demo to diff against. Work in this order instead of scraping HTML:

1. **Read the theme first.** `tailwind.config.*` or a v4 `@theme` block in the global CSS gives the exact palette, radius scale, and font stack — map these into `template.json`'s `theme` partial directly rather than eyeballing colors off a screenshot. Modern sources often use `oklch(...)` colors; convert each to hex (`oklch()` → sRGB hex) before writing `theme.colors` — the platform's theme schema is hex/rgb, not oklch.
2. **Map `src/components/**` 1:1 to variants.** A component-per-file source usually segments itself — each section-level component (`Hero.astro`, `PricingTable.tsx`, …) becomes one `sections/<key>.astro`. Resist merging or splitting differently than the source already did; the 1:1 mapping is also what makes revisions traceable later.
3. **Screenshot the live demo per section, and probe interaction**, before writing any WVF. Load the deployed demo (or `npm run dev`/`npm run build && npm run preview` locally) with Playwright, screenshot each `section[id]`/landmark, and — for anything that looks interactive — actually click it (accordion arrows, tabs, carousel dots) and diff `innerText`/visibility before/after. One pass here tells you definitively which sections are decoration and which are functionally interactive and MUST keep a script (workflow step 3 below) — guessing from a static screenshot alone under-counts interactivity almost every time.
4. **`src/data/*.json` or a content collection is your sample content.** Use it verbatim (trimmed to fit `MAX_SOURCE_BYTES`/`.sample.json` limits) instead of inventing placeholder copy — it's already realistic and already matches the section's actual shape.

For a source that is HTML only (no repo, no live demo), or built on a different framework/CSS approach, use the Workflow below as written.

## Workflow

1. **Segment** the source HTML top-to-bottom into sections. Classify each as **chrome** (navbar / announcement banner / footer) or a **page section**.
   **Multi-page source** (a site export with several HTML files — index.html, about.html, …): repeat the segmentation per file; each HTML page becomes one `pages[]` entry (`{ title, slug, sections[] }` — the homepage keeps `slug: ""`, max 8 pages per template). Extract chrome ONCE, from whichever page carries the fullest navbar/footer. Dedupe across pages: when two pages share a section design, author ONE `sections/<key>.astro` and reference the same `u:@<key>` from both with different `content`. The navbar links every page automatically (renderer-injected `pages` context prop) — never hand-write per-page nav links.
2. **Chrome: prefer platform variants; author WVF only when the design demands it.** A close platform `navbar`/`banner`/`footer` match with the source's branding in `content` is zero review-wait and battle-tested. When the source's chrome is genuinely distinctive, chrome MAY be a WVF file (since 2026-09-04) — it reads the renderer-injected context props (`pages`, `linkPrefix`, `currentSlug`, `colorMode`; footer also `footerPages`; `siteName` is an ordinary editable field with an injected fallback) and must keep its root in normal flow (`position: fixed` is a compile error; a `u:` navbar is wrapper-sticky automatically). Site pages appear in the navbar via `pages`; `content.links` is only for EXTRA destinations.
3. **Never degrade an interactive design to dodge script review.** If the source section is functionally interactive (slider/carousel arrows, tabs, accordion, filters), author the `<script is:inline>` per the variant skill's 4 — script review is a NORMAL, expected step, not a cost to engineer around. Downgrading a slider to a bare scroll strip trades the seller's design for your convenience; don't. The effect library (`data-wv-effect`) covers entrance/scroll MOTION only — it is never a substitute for functional interactivity.
4. **Each page section: reuse or author.** If a platform variant is a close match (see the type/variant catalog in webto's docs — `referensi-schema`, or `templates/CATALOG.md` when working inside the repo), reference it directly with real `content` — fewer files, zero review wait. Otherwise author a WVF file per the variant skill, one `sections/<key>.astro` per distinct design.
5. **Theme extraction.** Pull the source's palette and typography into the manifest's `theme` partial (`colors`, `darkColors`, `fonts`, `radius`, `maxWidth`, `spacing`). Fonts outside the platform's 93-family list (expanded 2026-09-11 — check the list in `references/manifest.md` first, several "new wave" fonts like Lexend/Marcellus/Instrument Serif/Geist are supported directly now) **reject the upload with an error** — fix the name or map it, don't guess and hope: **never settle for Inter when the source uses something with character**; match the source font to the NEAREST listed family (`references/manifest.md` § "Fonts: match, never default" has the mapping table — Futura→Poppins, Gotham→Montserrat, Didot→Playfair Display, Garamond→EB Garamond, ...). Then make every authored section USE the tokens — a template whose sections hardcode the palette defeats the theme (`hardcoded-color` blocks submit anyway).
6. **Content and images.** All visible copy becomes section `content` (the manifest) or field defaults (the .astro). Images: `asset:<filename>` for the source's OWN photography/art the seller has the rights to (ship the file under `assets/`, PNG/WebP/JPG ≤ 2 MB — see Hard limits); otherwise `__IMG__:<english query>` sentinels (the platform resolves them to Unsplash photos at upload) or direct unsplash/pexels URLs. **Never carry someone else's asset URLs** (the source's stock photography, a purchased icon pack, a competitor's product shot) — you do not hold a licence to redistribute those.
7. **Assemble `template.json`.** Pages reference authored sections as `variant: "u:@<key>"`; the `variants[]` array declares each key's `sectionType`, `name`, and **required** `description`/`mood` (`fits` optional but strongly recommended). **`name` MUST be the English Title-Case form of the key, word for word** (key `price-menu` → name "Price Menu"; key `hero-salon-split` → name "Hero Salon Split") — a name in another language or with different wording makes the source file untraceable from the dashboard. `description`/`mood`/`fits` are catalog copy the AI wizard reads to match this variant to a buyer's site — not editor labels: write `description` as a factual layout clause with no superlatives, `fits` as the content shape the variant needs, `mood` from the closed enum (see `references/manifest.md`). A bundle must be self-contained: plain `u:<id>` refs are rejected.
8. **Validate per file**: `npx @webto-id/variant-check sections/<key>.astro --type <sectionType> --strict --content samples/<key>.sample.json --out preview.html` — fix every ✖ and ▲, open the preview in both themes.
9. **Check parity, don't just eyeball it.** For each section, screenshot the SOURCE (the live demo, or the original HTML rendered) and the compiled `preview.html` at 390px, 768px, and 1280px, and diff them side by side — a plain look catches the obvious misses but reliably misses the specific ones: a heading that wrapped differently, a bento photo that didn't fill its column height, a title that measured 37.2px instead of the source's 24px. Run this per section as you finish it, not once at the end — fixing one section's typography scale is cheap; discovering all ten need the same fix during final review is not.
10. **Revisions**: to iterate on an ALREADY-uploaded template, use the listing editor's **"Update dari Bundle"** button (or /templates/upload?siteId=...). Matching is by each variant's bundle key (= file basename): identical sources are skipped ("Sama"), changed ones become a new version ("Versi baru" — re-enters review; buyers keep the approved version meanwhile), new keys become new variants. Partial uploads are supported:
    - **Patch a section**: upload ONLY the changed `.astro` file(s), no template.json — metadata stays as stored and the site structure is untouched.
    - **Add or restructure**: upload template.json + just the changed/NEW files — declared variants without an attached file keep their stored version, so adding one variant means its `.astro` + the manifest. When template.json is included it is authoritative: it overwrites the draft's structure and theme.
11. **Final gate**: the platform's "Periksa" (dry run) on webto.id → Templates → Upload Bundle. The pairing table there also has a per-section **Pratinjau** button (in-browser compile + render with the sample content) — tell the user to eyeball every section there before creating anything. It re-compiles everything strict and reports manifest problems addressed by JSON path (`pages[1].sections[3]: …`). Nothing is created until the dry run is clean.
12. **Deliver** the folder, plus the "What will not be copied" list below filled in for THIS conversion (even if some rows don't apply) — the seller can paste it straight into the listing description. After upload: the variants sit in the admin review queue; once approved, the seller publishes the listing from the template's listing editor.

## Fidelity checklist

Go through this explicitly before calling a conversion done — "looks right" skips exactly the items that don't announce themselves:

- **Typography scale at all 3 breakpoints** (mobile/tablet/desktop) — not just desktop. A heading that matches at 1280px and is wrong at 390px is a common miss.
- **Spacing rhythm** between sections and between elements within a section — the source's vertical rhythm, not whatever the base schema's defaults happen to produce.
- **Radius** — cards, buttons, images, badges all using the theme's actual radius scale, not a guessed value.
- **Shadows** — present where the source has them, absent where it doesn't; matching depth, not just "some shadow."
- **Hover and focus states** — a source with hover-lift cards or animated underlines that ships static is a visible downgrade a buyer will notice within a minute of using the editor.
- **Dark mode** — every section previewed in both themes, not assumed to "just work" from token usage alone.
- **Motion** — entrance animations, scroll-driven effects (see the variant skill's `wvf.md` §3.1), and hover transitions the source has, reproduced via `data-wv-effect` or CSS — see step 3 above on never silently dropping FUNCTIONAL interactivity to dodge script review; decorative motion has no such excuse either.

## What will not be copied (tell the buyer)

Some things are correctly out of reach for a marketplace variant — not a gap to apologize for, a boundary to state plainly. Hand this list to whoever's writing the listing description so buyer expectations match what's actually being sold:

- **WebGL/canvas/3D** (`<canvas>` is a compile error, and rightly so — see the variant skill's §2.1 allowed-tags list).
- **Components that need an npm package** — any interactivity that would require importing a library (rich carousels with physics, chart libraries, animation engines beyond CSS) has to be rebuilt from the WVF script subset or a CSS-only equivalent, which is not always visually identical.
- **Content that's random or different on every page load** — the renderer has no client-side data fetching; content is exactly what's in the section's saved fields.
- **Floating/fixed chrome** (a docked nav, a fixed-position theme toggle) — `position: fixed` is a universal compile error (WVF chrome, tightened 2026-09-04), including for navbar/banner.
- **Licensed/branded fonts and images the seller doesn't own** — replaced with the nearest matched Google Font and Unsplash/Pexels/platform-illustration images; never redistribute someone else's asset files or a paid font's files. (The seller's OWN photography/art is fine via `asset:<filename>` — see step 6.)

## Hard limits

| What | Limit |
|---|---|
| Authored variants per bundle | 12 |
| Whole upload payload | 2 MB |
| One `.astro` source | 128 KB |
| One `.sample.json` | 16 KB |
| Pages per marketplace template | 8 (one must be the homepage, `slug: ""`) |
| Variants per account (total) | 100 |
| One `asset:` image | 2 MB, PNG/WebP/JPG only (no SVG yet) |
| `asset:` images per bundle | 20 files, 8 MB total |

## Quality bar

The bundle is merchandise. Every section must survive the buyer's edits: content-shape robustness (1 item … max items, missing optionals), both themes, mobile, zero dead text, `<AddImageButton>` on image lists — all per the variant skill's `design.md`. The template as a whole must read coherently on ONE theme: if two sections disagree about what `primary` means, the conversion is not done.
