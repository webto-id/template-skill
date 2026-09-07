# Changelog

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
