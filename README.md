# html-to-webto-template

An AI skill (Claude Code, Cursor, and other agents that read `SKILL.md`) that converts a **full HTML template** — a multi-section landing page or a whole site export, any CSS framework or none — into a **webto template bundle**: `template.json` + `sections/*.astro` (Webto Variant Format) + `*.sample.json`, uploadable at [webto.id](https://app.webto.id/templates) → Templates → **Upload Bundle**.

The platform compiles every section strict, auto-submits the variants for review, and assembles a template draft you can list on the marketplace.

## Requires the variant skill

Each `sections/*.astro` file follows the WVF rules of the companion skill — install **both**, side by side:

```bash
# Claude Code (project-level)
git clone https://github.com/webto-id/variant-skill  .claude/skills/html-to-webto-variant
git clone https://github.com/webto-id/template-skill .claude/skills/html-to-webto-template

# Claude Code (global): same commands into ~/.claude/skills/
```

Or download the zips from <https://docs.webto.id/downloads/html-to-webto-template.zip> (and `html-to-webto-variant.zip`) and extract to the same places.

## Use

Ask your agent, for example:

> Konversi template HTML ini menjadi bundle template webto. Chrome-nya petakan ke variant platform, section unik jadi WVF, lalu validasi tiap file dengan `variant-check --strict`.

Per-file check while authoring:

```bash
npx @webto-id/variant-check sections/hero-1.astro --type hero --strict --content hero-1.sample.json --out preview.html
```

The final gate is the platform's **Periksa** (dry run) on the Upload Bundle page — it validates the whole manifest with every problem addressed by JSON path, and writes nothing until clean.

## Contents

| Path | What it is |
|---|---|
| `SKILL.md` | The workflow: segment → classify chrome vs sections → author/reuse → theme extraction → assemble → validate |
| `references/manifest.md` | The exact `template.json` format, `u:@<key>` refs, limits, theme extraction |
| `examples/demo-bundle/` | A complete bundle (Kopi Senja) that passes the platform dry run |

## Limits

12 authored variants per bundle · 2 MB payload · 128 KB per `.astro` · 16 KB per sample · 8 pages per template · images via `__IMG__:<query>` sentinels or unsplash/pexels URLs only.

## Docs

- Bundle format + upload flow: <https://docs.webto.id/marketplace/upload-template/>
- WVF format spec: <https://docs.webto.id/marketplace/format-variant/>

## License

MIT — see [LICENSE](LICENSE). The templates *you* generate with this skill are yours.
