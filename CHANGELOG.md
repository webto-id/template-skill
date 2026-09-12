# Changelog

## 0.1.13 — 2026-09-12

- New image form: `"asset:<filename>"` — the seller's OWN image (their own photography/art, not a stock library or someone else's assets). Ship the file (PNG/WebP/JPG, ≤ 2 MB, filename `[A-Za-z0-9._-]` only, ≤ 20 files / 8 MB per bundle) alongside `template.json`; the platform stores it once at a permanent platform-level location, shared by every buyer (never duplicated per site), and cleans it up only once no variant references it any more. SVG is not accepted yet (unsanitized SVG can carry script). Before this, a source's own art could never be carried over at all — this narrows that to "not someone ELSE's art," which is what the licensing concern actually was.

## 0.1.12 — 2026-09-11

- The renderer now requests every font weight **100–900**, not just 400/500/600/700 (heading) or 400/500/600 (body). A seller verified `ayudira-beauty` live and found 18 elements using `font-light` (weight 300) silently rendering at 400 — the weight was never loaded, and browsers never synthesize a LIGHTER face than what's available (only faux-bold heavier). Fixed at the platform level, live now with no action needed on already-uploaded bundles (theme is re-read on every render).
- Documented: a handful of whitelisted families are static single-weight (`Marcellus`, `Instrument Serif`, `Prata`, `Abril Fatface`) — `font-semibold`/`font-bold` on these renders browser-synthesized faux-bold, which is expected. Prefer a wider-range family when a design leans on real weight contrast.

## 0.1.11 — 2026-09-11

- Font whitelist expanded **60 → 93** families: 30 curated additions verified live against the actual Google Fonts CSS API (Lexend, Outfit, Sora, Figtree, Onest, Urbanist, Hanken Grotesk, Albert Sans, Be Vietnam Pro, Public Sans, Red Hat Display, Epilogue, Chivo, Sen, Instrument Sans, Schibsted Grotesk, Familjen Grotesk, Libre Franklin, Bricolage Grotesque, Unbounded, Syne, Geist, Fraunces, Marcellus, Instrument Serif, Zilla Slab, Newsreader, Prata, Geist Mono, DM Mono). **Lexend, Marcellus, Instrument Serif/Sans, and Geist no longer need a substitute** — use them as-is.
- **An unsupported `theme.fonts`/invalid `theme.colors` value now REJECTS the bundle upload with a clear per-field error**, not a silent swap. This came from a seller finding two of their own already-live templates had shipped with the wrong heading font (Lexend/Marcellus, both pre-dating this expansion) — `--strict`, the upload dry-run, and admin review all reported clean because the platform silently substituted the default with zero signal anywhere. Fixed at the platform level; this release just documents it.
- New `theme.pageTransitions: "none" | "fade"` (default `"none"`) — a site-wide CSS cross-fade between page navigations, no JS, respects `prefers-reduced-motion`. Relevant to nearly every Astro-source conversion: `<ClientRouter />` is Astro's own recommended default, so most Astro templates ship SPA-like navigation that a bare conversion otherwise drops.

## 0.1.10 — 2026-09-11

Grew out of a real conversion's parity-with-source report (`shadcn-astro-zolt-landing-page` → `bimo-portfolio`, 6th conversion by the same seller). Four skill-only changes, no platform behavior involved:

- New **"Source: an Astro + Tailwind repo"** section — a source repo (not just rendered HTML) deserves a different work order: read `tailwind.config`/`@theme` for exact tokens (including `oklch()` → hex conversion), map `src/components/**` 1:1 to variants, screenshot the live demo per section AND probe interactivity with Playwright before writing any WVF (a static screenshot alone reliably under-counts what's actually interactive), and use `src/data/*.json`/content collections as sample content instead of inventing copy.
- New step 9, **"Check parity, don't just eyeball it"** — screenshot source vs. compiled preview at 390/768/1280px per section, as you finish each one. The real-world case for this: a heading that measured 37.2px instead of the source's 24px, a bento photo that didn't fill its column, and a heading that wrapped differently — all invisible to a plain look, all caught by an actual side-by-side.
- New **Fidelity checklist** section — typography at all 3 breakpoints, spacing rhythm, radius, shadows, hover/focus states, dark mode, motion — an explicit list to run through before calling a conversion done.
- New **"What will not be copied"** section — WebGL/canvas, npm-dependent components, per-render-random content, floating/fixed chrome, the source's own licensed fonts/assets. Meant to be hand-copied into the listing description so buyer expectations match what's actually for sale.

## 0.1.9 — 2026-09-10

- `references/manifest.md`: new **Language** table — a bundle mixes AI-wizard
  catalog metadata (`variants[].name`/`description`/`fits`/`mood`, always
  English prose) with real showcase content (`siteName`, `aiDescription`,
  section `content`, `.sample.json` — the bundle's own `language`, Bahasa
  Indonesia by default) with per-`.astro` field doc comments (English prose,
  see the companion variant skill's `wvf.md` §1.2b). The example
  `description`/`fits` values were themselves in Indonesian, contradicting
  the rule stated right next to them — rewritten to English to match every
  built-in variant's own catalog text.

## 0.1.0 — 2026-09-04

- First public release, matching the platform's Upload Bundle feature (webto.id →
  Templates → Upload Bundle) and `@webto-id/variant-check` ≥ 0.1.6.
- `SKILL.md` workflow, `references/manifest.md` bundle spec, and the Kopi Senja
  `examples/demo-bundle/` (verified end-to-end against the platform dry run).

## 0.1.1 — 2026-09-04

- Chrome (navbar/banner/footer) may now be a bundle WVF variant (`u:@<key>` on
  `siteSections`) — platform variants remain the low-friction default. See the
  variant skill's chrome rules (context props injected, root in normal flow).

## 0.1.2 — 2026-09-05

- SKILL.md step 1 spells out **multi-page conversion**: one `pages[]` entry per
  source HTML file (homepage `slug: ""`, max 8 pages), chrome extracted once
  from the fullest page, shared section designs deduped into a single
  `u:@<key>` referenced with different `content`, nav links left to the
  renderer-injected `pages` prop.
- manifest.md Rules: the 8-page limit and cross-page `u:@<key>` reuse made
  explicit.
- Final-gate step now mentions the uploader's per-section **Pratinjau** button
  (in-browser compile + render on the template.json theme, before anything is
  created server-side).

## 0.1.3 — 2026-09-06

- Workflow step 8: **revisions via "Update dari Bundle"** — re-uploading an
  edited bundle over an existing template matches variants by bundle key
  (identical sources skipped, changed ones become a new version and re-enter
  review, new keys become new variants; template.json overwrites the draft's
  structure/theme).

## 0.1.4 — 2026-09-06

- Revision step now covers **partial uploads**: patch a section by uploading
  only its changed `.astro` (no template.json; matched by bundle key, site
  structure untouched), or add/restructure with template.json + just the
  changed/new files — declared variants without an attached file keep their
  stored version.

## 0.1.5 — 2026-09-07

Field-report hardening (three findings from real conversions):

- **Never degrade interactive designs to dodge script review** (new step 3):
  sliders/tabs/accordions keep their `<script is:inline>`; the effect
  library covers motion only, never functional interactivity.
- `variants[].name` **must be the English Title-Case form of the key, word
  for word** ("price-menu" → "Price Menu") so dashboard labels always lead
  back to the source file.
- Inline image editing is now lint-enforced platform-side
  (`edit-image-missing` fails `--strict` when a content-driven `<img>`
  lacks `data-edit-image`).


## 0.1.6 — 2026-09-07

- `references/manifest.md`: `mood` is documented as the CLOSED 14-value enum
  (`editorial luxurious airy minimal technical precise corporate playful warm
  casual bold brutal dense dark`, max 4) with nearest-mood mappings
  ("elegant" → `luxurious`, "romantic" → `warm`, "fun" → `playful`). A real
  conversion invented "elegant"/"romantic" and the whole upload was rejected;
  the platform's upload page now also reports unknown moods per variant key
  before anything is sent.

## 0.1.7 — 2026-09-07

- Fonts: when the source template's font is outside the platform's 60-family
  list, match it to the NEAREST listed family instead of defaulting to Inter.
  `references/manifest.md` § "Fonts: match, never default" carries the grouped
  whitelist and a commercial-font mapping table (Futura → Poppins, Gotham →
  Montserrat, Didot → Playfair Display, Garamond → EB Garamond, ...). Unknown
  names are silently swapped for the default by the platform, so a wrong name
  erases the typography without any error.

## 0.1.8 — 2026-09-07

- Images: a third legal form, `"__ILLU__:<2-4 keywords>"`, resolves to a
  platform illustration (`/illu/<id>.svg`) recolored to the buyer's live
  theme at serve time — for decorative flat-illustration slots where a photo
  would feel wrong. Indonesian and English keywords both match; an unmatched
  query resolves to an empty field, never a broken image. Photos stay the
  default for heroes, galleries, products, and people.
