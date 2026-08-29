# PreTeXt for a physics book

`skills/pretext/` is the reusable Claude Code skill (symlinked from `~/.claude/skills/pretext`).
The reference documents are the skill's own reference files, so there is one copy and
future sessions load them automatically when PreTeXt comes up.

- `skills/pretext/SKILL.md` — quick start and the rules that matter
- `skills/pretext/references/` — setup, project anatomy, core markup, **physics markup**,
  exercises, interactives, build & publish, gotchas, unit vocabulary
- `skills/pretext/templates/` — a validated physics chapter, a multi-target `project.ptx`,
  a physics `docinfo.ptx`

## What's in this directory

| Path | What |
|---|---|
| `.venv/` | PreTeXt CLI 2.51.0 (`source .venv/bin/activate` before building) |
| `vendor-pretext/` | Shallow clone of `PreTeXtBook/pretext` — the XSL, schema, `examples/`, and the Guide's source. Grep this when docs and behaviour disagree. |
| `scratch/demo-book/` | Throwaway project used to verify every claim in the docs. Builds web, latex, epub, kindle. |

## Verified here

```bash
source .venv/bin/activate
cd scratch/demo-book
pretext validate --engine salve
pretext build web && pretext build tex && pretext build ebook && pretext build kindle
```

PDF needs a TeX installation, which this machine does not have.

## Plan

The design of a generic PreTeXt authoring plugin is being charted as a wayfinder map on
this repo's issues: [Map: a validated design for a PreTeXt authoring plugin](https://github.com/rickroesler/pretext-authoring/issues/1).
