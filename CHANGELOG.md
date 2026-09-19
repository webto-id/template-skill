# Changelog

## 0.1.26 — 2026-09-19

- **Chrome now honours the owner's per-page navigation switches.** A WVF navbar received the site's pages UNFILTERED (each platform navbar filters `showInNavbar` inside its own component, so the injected prop never was), and `footerPages` arrived as `{ label, url }` while the authoring contract documents `{ title, slug }` — so an uploaded footer rendered empty links and "show in footer" looked broken. Both fixed platform-side, needs an `apps/site` deploy; bundles need no change.
- `SKILL.md` step 2 now states it: render `pages` in the navbar and `footerPages` in the footer as-is, `content.links` only for extra destinations. New warnings from CLI 0.1.31: `chrome-nav-pages-missing`, `chrome-footer-pages-missing`.

## 0.1.25 — 2026-09-19

- **`__ILLU__:` really is legal in image fields now.** The upload's validator rejected it with *"Must be a URL or image path"* although every import resolves it — a bundle using platform illustrations for its logo cloud failed the dry run while the identical content written as `__IMG__:` passed. Platform fix, needs an `apps/server` deploy; nothing in a bundle has to change.
- `manifest.md` now states what an image field refuses: a relative path (`assets/logo.png`, `./foto.jpg`) is not one of the four legal forms. `variant-check` 0.1.30 reports it locally as `content-image-url` instead of leaving it for the dry run.

## 0.1.24 — 2026-09-18

- **`variant-check` reads `template.json` on every per-file run** (`@webto-id/variant-check` 0.1.29). Running it on `sections/<key>.astro` now picks up the bundle manifest one level up with no flag, so `mood`, `description`, `sectionType` and `key` are checked locally against the platform's own enum instead of failing at upload — a 12-variant bundle rejected for four invented moods after a clean local run is what prompted this. `--manifest <file>` points at one explicitly, `--no-manifest` opts out.
- **Error**: a mood outside the enum, more than 4 of them, a `description` under 15 characters, an unknown `sectionType`, a malformed or duplicated `key`, more than 12 variants in one upload, or a `sectionType` that disagrees with `--type`. **Warning**: `description`/`mood` absent (a patch-mode re-upload may omit them; `--strict` promotes both), a field name the upload would strip in silence (`moods`, `tags`), a legacy `hero-*` alias, and an `.astro` with no matching entry.
- `SKILL.md` step 8 says to assemble the manifest (step 7) first, so both halves are validated on the same run.

## 0.1.23 — 2026-09-18

- **New mood value: `calm`** (platform enum; needs an `apps/server` deploy), for the quiet end the list never had — `bold`, `brutal` and `playful` covered loud, and quiet sections piled up under `minimal`/`airy` until mood stopped separating variants inside one bundle. `manifest.md` also states that **energy is a different axis from composition** (`airy` = empty space, `minimal` = few elements; a spacious section with a 96px accent headline is `bold` AND `airy`) and that mood describes the LOOK, never the subject — "personal" or "for a clinic" belong in `description`/`fits`. New mappings: "structured" → `precise`, "direct" → `bold`, "calm/quiet" → `calm`.

## 0.1.22 — 2026-09-17

- `SKILL.md` workflow step 2 no longer says "Chrome: prefer platform variants". Navbar, banner and footer are converted as WVF like every other section, so the header and footer carry the template's design; a platform chrome variant is reused only when it genuinely matches the source. The technical rules are unchanged (context props, root in normal flow, automatic sticky), plus a reminder to put `data-site-name` on the brand text. Matches the README change in 0.1.21.

## 0.1.21 — 2026-09-17

- `README.md` usage example no longer tells the model to map chrome to platform variants. Navbar, banner and footer can be authored as bundle WVF since 0.1.1, and forms since 0.1.20 — the skill decides per section whether to reuse a platform variant or author one, so the prompt only needs to ask for the conversion and the validation.

## 0.1.20 — 2026-09-17

- **Forms are authorable.** A contact/order form no longer has to be one of the ten platform looks: write a `form`-type variant whose layout wraps `<FormFields />` (variant skill 0.1.33, `wvf.md` §5c) — the platform renders fields, honeypot, Turnstile and the submit runtime; you render the card, the pill, the underlined inputs and the sticker button. `SKILL.md` step 7 says so. Until now the form was the one section that could not follow a converted template's design.

## 0.1.19 — 2026-09-17

- `manifest.md` theme block now covers **every layout key** and says what each one does on the live site, verified against the renderer: `layout` (`fullscreen` | `boxed` — boxed caps the body at `maxWidth`, centres it with a shadow, and is the only mode where `bodyBackground` is visible; it was not documented at all), `maxWidth` with its **pixel map** (`3xl` 768 · `4xl` 896 · `5xl` 1024 · `6xl` 1152 · `7xl` 1280 — so a source's `max-w-6xl`/`1152px` maps directly), `spacing` (multiplies every section's `py-*` by 0.65 / 1 / 1.35; write variants at normal rhythm and let this key carry the source's airiness), `radius`, `textGradient` (`none` = flat headlines; `subtle` = primary → 30% toward accent; `vibrant` = primary → accent — set `none` when the source's headlines are flat), `pageTransitions`. Each row also says how to read the value off the source instead of leaving defaults and compensating inside variants.

## 0.1.18 — 2026-09-17

- **Fixed the instruction that was causing the mistake it was meant to prevent.** The theme paragraph said *"map … warm highlights to `accent`"*, and on a converted template the warm highlight IS the pale band tint — so that sentence put a near-white value in the `accent` slot in **7 of one seller's 8 templates**. `accent` is a BRAND color slot: every palette a buyer can pick from the Style tab puts a saturated color there, and `accent-foreground` is recomputed for contrast against it, so a pale tint makes every `bg-accent` surface look right in your theme and unreadable in theirs. The variant skill's `wvf.md` §6.1 has the full rule, but it is read while writing `.astro` files — long after `template.json` decided the palette — so the rule now lives here too, at the moment of the decision.
- **Answers the question that follows immediately: what goes in `accent` when the source has only one brand color?** In order: a real second color from the source (link color, CTA hover, badge, chart series — sources have more of these than they appear to); a variation of `primary`; or `primary` repeated exactly, which is redundant but harmless. **Not** omitting the key — that defaults to amber `#f59e0b` (`#fbbf24` dark), a loud design choice rather than a neutral one — and never a near-white or near-background value.
- States that the source's band tint has **no slot at all**: `muted` and `card` are derived from `background` (mixed 6% and 2% toward `foreground`), which is what makes a band stay a shade off the page under every palette and in dark mode. The specific beige is traded for that guarantee, on purpose.

## 0.1.17 — 2026-09-16

- Limits corrected — the table still claimed a flat "100 variants per account", which no longer exists. Now: **12 authored variants per UPLOAD** (payload-bound: 12 x 128 KB already fills the 2 MB budget), **24 per TEMPLATE in total** (grow past one upload's worth with a partial upload — `template.json` + just the new file), and **300 STANDALONE variants per account**, which a bundle's variants no longer count against at all. The old flat cap made the platform's own limits contradict each other (10 template drafts x 12 variants = 120 > 100) and walled sellers off permanently, since an approved variant can never be deleted.

## 0.1.16 — 2026-09-16

- New `manifest.md` § **Power-word markers in text**: inline emphasis (`**bold**`, `==highlight==`, `%%block%%`, `@@circled@@`, `++brush++`, `__underline__`, `^^accent^^`, `[text](url)`) renders automatically in `template.json` section `content` — nothing declares or enables it — but **never** in `*.sample.json`, because no preview surface runs the marker pass and a marker there shows raw in the variant's own listing. Use them sparingly in content: emphasis is the buyer's tool. Authoring consequence (a field split across elements loses its markers) and the mechanism itself live in the variant skill's `wvf.md` §2.5.

## 0.1.15 — 2026-09-12

- Documented that `__IMG__:<query>` resolves to a DIFFERENT photo per occurrence — the same query in `template.json` and in a `.sample.json` gives two different pictures of the same subject, which reads as a bug when comparing the template preview against that variant's marketplace preview. Use `asset:<filename>` when a slot must be identical in both.

## 0.1.14 — 2026-09-12

- Recommended folder layout is now `sections/<key>.astro`, `samples/<key>.sample.json`, `assets/<filename>` (was: samples and assets flat at bundle root). Pairing has always matched every file by filename alone, regardless of which folder it's in — this is a convention change only, not a platform change; a flat bundle still uploads identically. The bundled `examples/demo-bundle` now follows the new layout.
- `SKILL.md`'s "Content and images" step and "what will not be copied" list now mention `asset:<filename>` (0.1.13 added the feature but missed updating these two spots).

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
