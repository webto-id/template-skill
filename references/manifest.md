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
      "description": "Two-column hero with a large photo on the right", "mood": ["warm"],
      "fits": "short headline + one strong photo" }
  ]
}
```

## Rules

- **`u:@<key>`** refs are legal in `pages[].sections[]` and `siteSections[]`, and must name an entry in `variants[]` whose `sectionType` matches the section's `type`. The uploader supplies each entry's `source` from `sections/<key>.astro`; a `<key>.sample.json` next to it becomes the variant's showcase content.
- **Plain `u:<id>` refs are rejected** — a bundle is self-contained. Platform variant ids (from the catalog) are fine.
- **Chrome (`siteSections`) may use platform variants or bundle `u:@<key>` refs** (WVF chrome allowed since 2026-09-04 — see the variant skill's chrome rules: context props injected, root in normal flow). Exactly one navbar with `position: "header"` is required.
- **`key`**: lowercase letters/digits/hyphens, 2–48 chars, unique, equal to the file basename. **`name` = the key in English Title Case, word for word** ("price-menu" → "Price Menu") so the dashboard label always leads back to the file.
- **`mood` is a CLOSED enum — never invent values** (an unknown mood rejects the whole upload): `editorial` `luxurious` `airy` `minimal` `technical` `precise` `corporate` `playful` `warm` `casual` `bold` `brutal` `dense` `dark`, max 4 per variant. Map the design's feel to the NEAREST listed mood ("elegant" → `luxurious`, "romantic" → `warm`, "fun" → `playful`).
- **`description` and `mood` are REQUIRED** (`description` 15–300 chars) — they are what the AI wizard's variant catalog reads to decide whether this variant fits a buyer's site, not editor-facing copy. Write `description` as a factual layout clause, no superlatives ("Two-column hero with a large photo on the right", not "A stunning modern hero"). Write `fits` as the content shape this variant actually needs ("short headline + one strong photo", not a restatement of the description). **Both in English prose**, same convention as `name` and every built-in variant's own catalog text — see "Language" below. A bundle missing either on a variant fails upload with a clear per-key message.
- **Max 8 `pages`**, exactly one with `slug: ""` (the homepage). Other slugs are slugified on insert. Several pages may reference the same `u:@<key>` — one .astro, reused with different `content` per page.
- Omit `subdomain` and any listing metadata — the platform derives the one and the listing editor owns the other.
- Section `content` is validated against the section's real schema (base + the COMPILED extension for `u:@` refs), with every problem reported at once, addressed by path. `content: {}` is legal but hollow — fill real showcase copy.

## Language

A bundle mixes catalog metadata (read by the AI wizard, in English) with actual site content (read by the buyer, in the site's own language) — the same distinction the companion variant skill's `wvf.md` §1.2b covers for a single `.astro` file, extended to everything else in a bundle:

| Field | Language | Why |
|---|---|---|
| `variants[].name` | **English Title Case**, word-for-word from `key` | Dashboard label must trace back to the source file |
| `variants[].description` / `.fits` / `.mood` | **English prose** (`mood` from the closed enum, which is already English) | Same AI-wizard catalog every built-in variant's `description` feeds — see `wvf.md` §1.2b |
| `.astro` field doc comments inside each `sections/<key>.astro` | **English prose** | See `wvf.md` §1.2b — this manifest doesn't change that rule |
| `siteName`, `aiDescription`, `pages[].sections[].content`, chrome `content` | The bundle's own `language` (Bahasa Indonesia by default) | This is the actual showcase copy a buyer previews before replacing it — real content, not catalog metadata |
| `<key>.sample.json` | Same as `content` above | It's the variant's own showcase content, shown standalone in the marketplace preview |
| `__IMG__:<query>` | English search query | Passed straight to Unsplash search |
| `__ILLU__:<keywords>` | Indonesian or English, either matches | Matched against a bilingual keyword index |

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

Beware: color/font values that fail validation are silently replaced by defaults — double-check hex codes.

### Fonts: match, never default

`fonts.heading`/`fonts.body` accept ONLY the platform's 60 Google Fonts families (an unknown name is silently swapped for the default — the design's typography just vanishes). When the source template's font is not on the list, **find the closest listed family by classification — do NOT fall back to Inter** unless the source really is a neutral neo-grotesque. Judge by letterforms: geometric vs humanist, serif contrast, x-height, width, weight.

The full list, grouped: modern sans `Inter Poppins Roboto "Open Sans" Lato Montserrat Raleway Nunito "Nunito Sans" "Work Sans" "DM Sans" "Plus Jakarta Sans" Manrope "Space Grotesk" Rubik Mulish Quicksand Karla Barlow Archivo Kanit Cabin "Josefin Sans" Comfortaa "IBM Plex Sans" "Source Sans 3" "PT Sans" "Noto Sans" Ubuntu "Fira Sans"`; display/condensed `Oswald "Bebas Neue" Anton "Archivo Black" "Roboto Condensed" "Titillium Web"`; serif `"Playfair Display" Merriweather Lora Bitter "Roboto Slab" "PT Serif" "Libre Baskerville" "Crimson Text" "EB Garamond" "Cormorant Garamond" "Source Serif 4" "IBM Plex Serif" "Abril Fatface"`; script `"Dancing Script" Pacifico Caveat "Shadows Into Light" Satisfy "Great Vibes" "Permanent Marker" "Indie Flower" Kalam`; mono `"JetBrains Mono" "Roboto Mono" "Fira Code" "Space Mono" "IBM Plex Mono"`.

Common commercial/system fonts → nearest listed:

| Source font | Use |
|---|---|
| Helvetica, Arial, Neue Haas, Aktiv | Inter |
| Futura, Century Gothic, Avant Garde | Poppins (or Josefin Sans for elegant/tall) |
| Avenir, Circular, Graphik, Sofia Pro | Manrope or DM Sans |
| Gotham, Proxima Nova, Montserrat-like caps | Montserrat |
| Brandon Grotesque | Josefin Sans |
| Gill Sans, Verdana | Cabin or Lato |
| Frutiger, Myriad, Segoe UI | Open Sans or Source Sans 3 |
| DIN, Eurostile | Barlow or Archivo |
| Trade Gothic / any condensed grotesque | Oswald or Roboto Condensed |
| Impact, Druk, Compacta | Anton or Archivo Black |
| Didot, Bodoni | Playfair Display (Abril Fatface for poster sizes) |
| Garamond (any cut) | EB Garamond (Cormorant Garamond for lighter, more fashion) |
| Baskerville | Libre Baskerville |
| Caslon, Minion, Sabon | Crimson Text or Lora |
| Georgia, Times New Roman | PT Serif or Merriweather |
| Rockwell, Museo Slab, Clarendon | Roboto Slab or Bitter |
| Courier, Consolas, SF Mono | Roboto Mono or JetBrains Mono |
| Snell Roundhand, Allura, wedding scripts | Great Vibes (Dancing Script for casual) |
| Handwritten/marker | Caveat, Permanent Marker, or Kalam |

Custom font FILES (@font-face) cannot be carried at all — same rule: translate to the nearest listed family. Map the SOURCE template's brand color to `primary`, its secondary brand to `secondary`, warm highlights to `accent`; pick the two dominant font families only (heading + body). Then let the sections speak tokens (`bg-primary`, `var(--font-heading)`) — that is what makes the whole template re-themeable by the buyer.

## Images

Three legal forms, everywhere (manifest `content` and `.sample.json` alike):

- `"__IMG__:<english search query>"` — resolved to a distinct Unsplash photo per occurrence at upload. Best default.
- `"__ILLU__:<2-4 keywords>"` — resolved to a PLATFORM ILLUSTRATION (`/illu/<id>.svg`) that is recolored to the buyer's live theme at serve time. Use for decorative flat-illustration slots where a photo would feel wrong (abstract values/features, blobs, wave dividers, small scene spots: kopi, warung, wedding rings, kurir, kamera, grafik, kalender — Indonesian or English keywords both match). An unmatched query resolves to an EMPTY field, never a broken image; photos stay the default for heroes, galleries, products, and people.
- A direct `https://images.unsplash.com/...` or pexels URL.

The source template's own assets (its `/img/...`, CDN links, stock previews) are **never** carried over — licensing.

## What happens on upload

1. Every `.astro` is compiled **strict** (`--strict` clean is the bar; warnings are errors).
2. The manifest is validated; dry run ("Periksa") reports everything without writing.
3. On create: variants are created under the seller's account with status **pending** (auto-submitted for admin review), the local refs are rewritten to real ids, and the template draft site appears under /templates.
4. The listing can be published from the listing editor **after every variant is approved** — the source check there names any that are still waiting.
