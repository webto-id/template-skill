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
