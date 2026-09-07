# template.json — the bundle manifest

One JSON object. Two halves: a **site definition** (what the admin import format uses) and the **variants[] declarations** for the bundle's own WVF files.

```jsonc
{
  "siteName": "Kopi Senja",              // 1-100 chars; the subdomain derives from it
  "language": "id",                       // default "id"
  "siteType": "website",                 // "website" | "landing" | "personal"
  "aiDescription": "Kedai kopi ...",     // optional; seeds AI context for buyers
  "theme": { ... },                       // PARTIAL theme — only what differs (below)
  "siteSections": [                       // chrome: platform variant id or "u:@<key>"
    { "type": "navbar", "position": "header", "variant": "centered",
      "content": { "siteName": "Kopi Senja" } },
    { "type": "footer", "position": "footer", "variant": "simple", "content": {} }
  ],
  "pages": [
    { "title": "Beranda", "slug": "",     // slug "" = homepage (required once)
      "sections": [
        { "type": "hero", "variant": "u:@hero-split",   // ← bundle variant, by key
          "content": { "headline": "..." } },
        { "type": "features", "variant": "cards",        // ← platform variant, by id
          "content": { "heading": "...", "features": [ ... ] } }
      ] }
  ],
  "variants": [                           // one entry per sections/<key>.astro
    { "key": "hero-split", "sectionType": "hero", "name": "Hero Split",
      "description": "Hero dua kolom dengan foto besar", "mood": ["warm"],
      "fits": "headline pendek + satu foto kuat" }
  ]
}
```

## Rules

- **`u:@<key>`** refs are legal in `pages[].sections[]` and `siteSections[]`, and must name an entry in `variants[]` whose `sectionType` matches the section's `type`. The uploader supplies each entry's `source` from `sections/<key>.astro`; a `<key>.sample.json` next to it becomes the variant's showcase content.
- **Plain `u:<id>` refs are rejected** — a bundle is self-contained. Platform variant ids (from the catalog) are fine.
- **Chrome (`siteSections`) may use platform variants or bundle `u:@<key>` refs** (WVF chrome allowed since 2026-09-04 — see the variant skill's chrome rules: context props injected, root in normal flow). Exactly one navbar with `position: "header"` is required.
- **`key`**: lowercase letters/digits/hyphens, 2–48 chars, unique, equal to the file basename. **`name` = the key in English Title Case, word for word** ("price-menu" → "Price Menu") so the dashboard label always leads back to the file.
- **`mood` is a CLOSED enum — never invent values** (an unknown mood rejects the whole upload): `editorial` `luxurious` `airy` `minimal` `technical` `precise` `corporate` `playful` `warm` `casual` `bold` `brutal` `dense` `dark`, max 4 per variant. Map the design's feel to the NEAREST listed mood ("elegant" → `luxurious`, "romantic" → `warm`, "fun" → `playful`).
- **Max 8 `pages`**, exactly one with `slug: ""` (the homepage). Other slugs are slugified on insert. Several pages may reference the same `u:@<key>` — one .astro, reused with different `content` per page.
- Omit `subdomain` and any listing metadata — the platform derives the one and the listing editor owns the other.
- Section `content` is validated against the section's real schema (base + the COMPILED extension for `u:@` refs), with every problem reported at once, addressed by path. `content: {}` is legal but hollow — fill real showcase copy.

## Theme extraction (`theme`)

A PARTIAL of the site theme; unspecified keys get platform defaults. Useful keys:

```jsonc
{
  "colorMode": "light",                       // "light" | "dark" | "system"
  "colors":     { "primary": "#7c3aed", "secondary": "#64748b",
                  "background": "#ffffff", "foreground": "#0f172a", "accent": "#f59e0b" },
  "darkColors": { /* same 5 keys */ },
  "fonts":  { "heading": "Poppins", "body": "Inter",
              "headingSize": 48, "bodySize": 16 },   // 24-96 / 12-24 px
  "radius": 12,                                // 0-24 px
  "maxWidth": "6xl",                           // "3xl".."7xl"
  "spacing": "normal",                         // "compact" | "normal" | "spacious"
  "textGradient": "subtle"                     // "none" | "subtle" | "vibrant"
}
```

Beware: color/font values that fail validation are silently replaced by defaults — double-check hex codes. Map the SOURCE template's brand color to `primary`, its secondary brand to `secondary`, warm highlights to `accent`; pick the two dominant font families only (heading + body). Then let the sections speak tokens (`bg-primary`, `var(--font-heading)`) — that is what makes the whole template re-themeable by the buyer.

## Images

Two legal forms, everywhere (manifest `content` and `.sample.json` alike):

- `"__IMG__:<english search query>"` — resolved to a distinct Unsplash photo per occurrence at upload. Best default.
- A direct `https://images.unsplash.com/...` or pexels URL.

The source template's own assets (its `/img/...`, CDN links, stock previews) are **never** carried over — licensing.

## What happens on upload

1. Every `.astro` is compiled **strict** (`--strict` clean is the bar; warnings are errors).
2. The manifest is validated; dry run ("Periksa") reports everything without writing.
3. On create: variants are created under the seller's account with status **pending** (auto-submitted for admin review), the local refs are rewritten to real ids, and the template draft site appears under /templates.
4. The listing can be published from the listing editor **after every variant is approved** — the source check there names any that are still waiting.
