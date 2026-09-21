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

- **`u:@<key>`** refs are legal in `pages[].sections[]` and `siteSections[]`, and must name an entry in `variants[]` whose `sectionType` matches the section's `type`. The uploader supplies each entry's `source` from `sections/<key>.astro`; a `<key>.sample.json` becomes the variant's showcase content — matched by FILENAME only (`<key>.sample.json`), wherever it sits in the bundle folder (recommended: `samples/<key>.sample.json` — see the skill's `SKILL.md` for the folder layout; a flat root also works, the uploader doesn't care).
- **Plain `u:<id>` refs are rejected** — a bundle is self-contained. Platform variant ids (from the catalog) are fine.
- **Chrome (`siteSections`) may use platform variants or bundle `u:@<key>` refs** (WVF chrome allowed since 2026-09-04 — see the variant skill's chrome rules: context props injected, root in normal flow). Exactly one navbar with `position: "header"` is required.
- **`key`**: lowercase letters/digits/hyphens, 2–48 chars, unique, equal to the file basename. **`name` = the key in English Title Case, word for word** ("price-menu" → "Price Menu") so the dashboard label always leads back to the file.
- **`mood` is a CLOSED enum — never invent values** (an unknown mood rejects the whole upload): `editorial` `luxurious` `airy` `minimal` `technical` `precise` `corporate` `playful` `warm` `casual` `bold` `brutal` `calm` `dense` `dark`, max 4 per variant. From `variant-check` 0.1.29 the CLI reads `variant.json`/`template.json` next to the `.astro` (or one level up) and checks this enum itself — an invented mood is an error locally instead of a rejected upload, and `--strict` also fails an empty `mood`/`description`, matching the submit gate. Map the design's feel to the NEAREST listed mood ("elegant" → `luxurious`, "romantic" → `warm`, "fun" → `playful`, "structured" → `precise`, "direct" → `bold`, "calm/quiet" → `calm`).
- **Energy is a different axis from composition.** `bold`/`brutal`/`playful` are the loud end, `calm` the quiet one; `airy` means empty space and `minimal` means few elements — neither implies a slow pace, and a spacious section with a 96px accent headline is `bold` AND `airy`. Without `calm`, quiet sections pile up under `minimal`/`airy` until the mood stops separating variants inside one bundle. And mood describes the LOOK, never the subject: "personal" or "for a clinic" belong in `description`/`fits`.
- **`description` and `mood` are REQUIRED** (`description` 15–300 chars) — they are what the AI wizard's variant catalog reads to decide whether this variant fits a buyer's site, not editor-facing copy. Write `description` as a factual layout clause, no superlatives ("Two-column hero with a large photo on the right", not "A stunning modern hero"). Write `fits` as the content shape this variant actually needs ("short headline + one strong photo", not a restatement of the description). **Both in English prose**, same convention as `name` and every built-in variant's own catalog text — see "Language" below. A bundle missing either on a variant fails upload with a clear per-key message.
- **Max 8 `pages`**, exactly one with `slug: ""` (the homepage). Other slugs are slugified on insert. Several pages may reference the same `u:@<key>` — one .astro, reused with different `content` per page.
- Omit `subdomain` — the platform derives it. Listing metadata goes in the top-level `listing` block (below), not inside the site manifest.
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
| `asset:<filename>` | Exact match to the uploaded file's name — never translated, never altered | It's an identity, not a language string; a filename outside `[A-Za-z0-9._-]` is rejected, never silently renamed |

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
  "radius": 12,                                // 0-24 px → --radius; every rounded-* token follows it
  "layout": "fullscreen",                      // "fullscreen" | "boxed"
  "maxWidth": "6xl",                           // "3xl" 768px | "4xl" 896 | "5xl" 1024 | "6xl" 1152 | "7xl" 1280
  "spacing": "normal",                         // "compact" | "normal" | "spacious"  (section rhythm x0.65 / x1 / x1.35)
  "textGradient": "subtle",                    // "none" | "subtle" | "vibrant"
  "pageTransitions": "none"                    // "none" | "fade" — site-wide cross-fade between pages, added 2026-09-11
}
```

### Layout keys: what each one actually does, and how to read it off a source

These are the Style-tab settings the buyer can change later, so a conversion should set them to what the SOURCE does — not leave them on defaults and compensate inside the variants.

| Key | Effect on the live site | Read it from the source |
|---|---|---|
| `layout` | `fullscreen`: sections span the viewport (default). `boxed`: the whole page body is capped at `maxWidth`, centred, with a shadow, and the body background (`bodyBackground` image/SVG) paints on `<html>` AROUND it — that is the only mode where the body background is visible. | A source whose page sits as a framed column on a coloured/patterned backdrop is `boxed`; anything edge-to-edge is `fullscreen`. |
| `maxWidth` | Sets `--site-max-width`, which every `<Container>` uses (`3xl` 48rem/768px · `4xl` 56rem/896 · `5xl` 64rem/1024 · `6xl` 72rem/1152 · `7xl` 80rem/1280). In `boxed` mode it caps the body too. | The source's container `max-width` (`max-w-6xl`, `1152px`, `72rem`). Pick the nearest; never hard-code a width inside a variant to fake a different one. |
| `spacing` | Multiplies every outer `<section>`'s `py-*` through Tailwind's `--spacing` var: `compact` 0.65, `normal` 1, `spacious` 1.35. No per-variant work; viewport-height heroes (`min-h-[Nvh]`) are unaffected by design. | Write variants with `normal` rhythm (`py-14 sm:py-20`-ish) and let this key carry the source's overall airiness. A source with `py-32` everywhere is `spacious`, not 32 hard-coded in each file. |
| `radius` | `--radius`; the theme radius scale (`rounded-sm/md/lg/xl`) derives from it. 0 = sharp. | The source's dominant corner radius on cards/buttons. |
| `textGradient` | `--gradient-from/--gradient-to`, consumed by headlines that opt in (`bg-clip-text text-transparent` + `linear-gradient(to right, var(--gradient-from), var(--gradient-to))`). `none` collapses both to `foreground` (plain text); `subtle` = primary → primary mixed 30% toward accent; `vibrant` = primary → accent. | Default is `subtle`. Set `none` when the source's headlines are flat — otherwise a gradient headline appears that the source never had. |
| `pageTransitions` | `fade`: CSS `@view-transition` cross-fade on navigation, zero JS, ignored by unsupported browsers and by `prefers-reduced-motion`. | Astro sources with `<ClientRouter />`, or any source with SPA-like page fades → `fade`. |

`colorMode`, `bodyBackground` (an image or a platform SVG pattern behind a boxed page) and `fonts.headingSize`/`bodySize` round out the block; all are optional and default sensibly.

An invalid hex or unsupported font name in `theme.colors`/`theme.fonts` now REJECTS the upload with a clear per-field error (fixed 2026-09-11 — it used to swap silently for the default, which is how the wrong heading font shipped on two live templates before anyone noticed). Double-check hex codes and font names regardless; a wrong-but-valid value (a legal hex that's just not the source's actual color) still passes through unnoticed.

### Fonts: match, never default

`fonts.heading`/`fonts.body` accept ONLY the platform's 93 Google Fonts families (expanded from 60, 2026-09-11). **An unknown name now REJECTS the whole bundle upload with a clear error naming the field** — it used to swap silently for the default with zero signal, which is how two live templates shipped with the wrong heading font before anyone noticed. When the source template's font genuinely isn't on the list, **find the closest listed family by classification — do NOT fall back to Inter** unless the source really is a neutral neo-grotesque. Judge by letterforms: geometric vs humanist, serif contrast, x-height, width, weight.

The full list, grouped: modern sans `Inter Poppins Roboto "Open Sans" Lato Montserrat Raleway Nunito "Nunito Sans" "Work Sans" "DM Sans" "Plus Jakarta Sans" Manrope "Space Grotesk" Rubik Mulish Quicksand Karla Barlow Archivo Kanit Cabin "Josefin Sans" Comfortaa "IBM Plex Sans" "Source Sans 3" "PT Sans" "Noto Sans" Ubuntu "Fira Sans"`; display/condensed `Oswald "Bebas Neue" Anton "Archivo Black" "Roboto Condensed" "Titillium Web"`; serif `"Playfair Display" Merriweather Lora Bitter "Roboto Slab" "PT Serif" "Libre Baskerville" "Crimson Text" "EB Garamond" "Cormorant Garamond" "Source Serif 4" "IBM Plex Serif" "Abril Fatface"`; script `"Dancing Script" Pacifico Caveat "Shadows Into Light" Satisfy "Great Vibes" "Permanent Marker" "Indie Flower" Kalam`; mono `"JetBrains Mono" "Roboto Mono" "Fira Code" "Space Mono" "IBM Plex Mono"`.

Added 2026-09-11 (modern sans, display, serif, mono — fills the gap for the "new wave" grotesques and editorial serifs that kept showing up in Astro/Tailwind sources): modern sans `Lexend Outfit Sora Figtree Onest Urbanist "Hanken Grotesk" "Albert Sans" "Be Vietnam Pro" "Public Sans" "Red Hat Display" Epilogue Chivo Sen "Instrument Sans" "Schibsted Grotesk" "Familjen Grotesk" "Libre Franklin"`; display `"Bricolage Grotesque" Unbounded Syne Geist`; serif `Fraunces Marcellus "Instrument Serif" "Zilla Slab" Newsreader Prata`; mono `"Geist Mono" "DM Mono"`. **Lexend, Marcellus, Instrument Serif, Instrument Sans, and Geist are now supported directly** — stop mapping these to a substitute, use them as-is.

The renderer requests every weight 100–900 for whichever family you pick (fixed 2026-09-11 — it used to request only 400/500/600/700, so `font-light`/`font-thin` silently rendered at 400 with no error, since browsers never synthesize a lighter weight than what's loaded). Most families are variable and genuinely have the whole range; a handful are static single-weight (`Marcellus`, `Instrument Serif`, `Prata`, `Abril Fatface`) — picking one of these and then using `font-semibold`/`font-bold` in your variant renders faux-bold (browser-synthesized), which is expected, not a bug. If a design leans on real weight contrast, prefer a family with a wider range.

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

Custom font FILES (@font-face) cannot be carried at all — same rule: translate to the nearest listed family. Pick the two dominant font families only (heading + body), map the SOURCE template's brand color to `primary` and its secondary brand to `secondary`, and read the next section before you fill `accent`. Then let the sections speak tokens (`bg-primary`, `var(--font-heading)`) — that is what makes the whole template re-themeable by the buyer.

### `accent` is a brand color, never the page's pale band tint

This is the single easiest thing to get wrong here, because the wrong answer looks right for as long as you are working on it.

A converted template almost always has a soft off-white band — `#f9f3ee`, `#f0f0f8`, `#e9e2d7`. It is tempting to call that the template's "highlight" and put it in `accent`. **Do not.** `accent` is a BRAND color slot: every palette the buyer can pick from the Style tab puts a saturated color there (teal `#0d9488`, amber `#d97706`, blue `#2563eb`), and `accent-foreground` is recomputed for contrast against whatever lands in it. A pale tint in that slot makes every `bg-accent` surface in your sections look correct in YOUR theme and unreadable in theirs — and you will not see it, because you are looking at your own palette the whole time.

The band tint has **no slot at all**, and that is deliberate: `muted` and `card` are DERIVED from `background` (mixed 6% and 2% toward `foreground`), so a band stays a shade off the page under every palette and in dark mode. Your source's specific beige becomes a derived warm grey. That is the trade the platform makes on purpose — see the variant skill's `wvf.md` §6.1 for the full table and for which token each kind of surface should use.

**So what goes in `accent` when the source has only one brand color?** In order:

1. **A real second color from the source**, even a small one — the link color, a CTA hover, a badge, a chart series, the color on a "new" pill. Sources have more of these than they appear to.
2. **A variation of `primary`** — a deeper or brighter shade of the same hue. Highlights then read as brand, which is almost always what the source intended anyway.
3. **`primary` repeated exactly.** Redundant, but honest and harmless: accent surfaces simply become primary surfaces.

**Do not omit the key to avoid deciding** — a missing `accent` defaults to amber `#f59e0b` (`#fbbf24` dark), which is a loud design choice, not a neutral one. And never put a near-white or near-background value there.

## Power-word markers in text

Any section text field can carry inline emphasis the site renders for the buyer: `**bold**`, `*italic*`, `==highlight==`, `%%block%%`, `@@circled@@`, `++brush++`, `__underline__`, `^^accent^^`, `[text](url)`. The rendering is central and automatic (see the variant skill's `wvf.md` §2.5) — nothing in a bundle declares or enables it. Two rules that matter here:

- **Use them sparingly in `template.json` section `content`**, where they DO render — one emphasized fragment in a headline is a design decision, a page full of them is noise. They are the seller's tool, so prefer leaving the choice to the buyer unless the source design clearly emphasizes a specific word.
- **Never in `*.sample.json`.** Neither preview surface runs the marker pass, so a marker there shows as raw `==…==` in the variant's own marketplace listing and reads as a bug.

And when authoring the `.astro` itself: a marker pair only survives inside ONE text node, so a field whose value you split across elements (per-word animation) silently loses them — same `wvf.md` §2.5.

## Images

Four legal forms, everywhere (manifest `content` and `.sample.json` alike):

- `"asset:<filename>"` — the seller's OWN image (their own photography/art — never someone else's stock library, icon pack, or a competitor's product shot; licensing still applies, just to a narrower thing). Added 2026-09-12: ship the actual file (PNG/WebP/JPG, ≤ 2 MB, filename `[A-Za-z0-9._-]` only, ≤ 20 files / 8 MB per bundle) somewhere in the bundle folder — recommended `assets/<filename>` (see `SKILL.md`), matched by filename only, same as `.sample.json`. The platform stores it once at a permanent platform-level location, shared by every buyer (never duplicated per site), cleaned up only once no variant references it any more. SVG is NOT an accepted `asset:` format yet (unsanitized SVG can carry script) — convert decorative art to PNG/WebP first. Use this whenever the source's own imagery is what makes the design — a hero photograph, a product shot, a logo mark — not a generic scene a stock query would serve just as well.
- `"__IMG__:<english search query>"` — resolved to a distinct Unsplash photo per occurrence at upload. Best default when there's no real asset from the source worth carrying over. **"Per occurrence" is literal**: the same query written in `template.json` and again in a `.sample.json` resolves to TWO different photos, so the template preview and that variant's marketplace preview will show different pictures of the same subject. When a slot must look identical in both, use `asset:<filename>` — one file, one stored key, same image everywhere.
- `"__ILLU__:<2-4 keywords>"` — resolved to a PLATFORM ILLUSTRATION (`/illu/<id>.svg`) that is recolored to the buyer's live theme at serve time. Use for decorative flat-illustration slots where a photo would feel wrong (abstract values/features, blobs, wave dividers, small scene spots: kopi, warung, wedding rings, kurir, kamera, grafik, kalender — Indonesian or English keywords both match). An unmatched query resolves to an EMPTY field, never a broken image; photos stay the default for heroes, galleries, products, and people.
- A direct `https://images.unsplash.com/...` or pexels URL.

Nothing else is accepted in an image field: a relative path (`assets/logo.png`, `./foto.jpg`) fails the dry run with *"Must be a URL or image path"*. `variant-check` 0.1.30 reports the same thing locally as `content-image-url`. (Until 2026-09-18 the dry run also rejected `__ILLU__:` in those fields, although the import resolves it — that was a platform bug, now fixed; it needs an `apps/server` deploy.)

Anything from the source under a license that doesn't transfer to the platform (stock photography, someone else's icon set, a purchased asset pack) is still **never** carried over.

## `listing`: the catalog copy, written once, here

A template is sold from a listing, and every field of that listing is something
this conversion already knows. Write them into `template.json` as a `listing`
block and the seller publishes without typing any of it (and without spending an
AI credit on the dashboard's "write my description" button):

```json
"listing": {
  "name": "Studio Kalastra",
  "title": "Studio Kalastra - Template Website Agency, Creative Studio & Konsultan",
  "description": "Template satu halaman untuk agency kreatif ...",
  "category": "Portofolio & Agency",
  "tags": ["agency", "creative studio", "konsultan", "portofolio"]
}
```

- **`name`** (2-60) is the SHORT name, and it is what every catalog card shows
  under the thumbnail. Two or three words, the studio/brand the template is
  built around — never the keyword line.
- **`title`** (3-80) is the SEO line, shown on the detail page and in search
  results. This is where the keywords go. Keeping these two apart is the whole
  point: one field had to be both, so every card truncated a keyword string
  mid-word.
- **`description`** (≤5000) is markdown, the listing body a buyer reads. Say
  what the template is for, what is inside (pages, sections), and who it fits.
  Factual; no superlatives.
- **`category`** must be one of the platform's own, exactly: `Bisnis & Jasa` ·
  `Toko Online` · `Kuliner` · `Portofolio & Agency` · `Pendidikan & Kursus` ·
  `Kesehatan & Kecantikan` · `Properti & Konstruksi` · `Travel & Wisata` ·
  `Event & Wedding` · `Profil Personal` · `Komunitas & Organisasi` ·
  `Teknologi & SaaS` · `Lainnya`. An invented category is rejected by the
  upload — and would put the template in a facet the sidebar does not list.
- **`tags`** up to 8, free-form, lowercase, the words a buyer would search.

Everything here is a PROPOSAL: it is stored on the draft and prefills the
publish form, where the seller edits it and sets the price. Nothing in this
block creates a listing or sells anything.

## What happens on upload

1. Every `.astro` is compiled **strict** (`--strict` clean is the bar; warnings are errors).
2. The manifest is validated; dry run ("Periksa") reports everything without writing.
3. On create: variants are created under the seller's account with status **pending** (auto-submitted for admin review), the local refs are rewritten to real ids, and the template draft site appears under /templates.
4. The listing can be published from the listing editor **after every variant is approved** — the source check there names any that are still waiting.
