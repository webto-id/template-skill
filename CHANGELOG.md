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

