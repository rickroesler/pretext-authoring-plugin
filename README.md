# PreTeXt for a physics book

`docs/` is a symlink to `~/.claude/skills/pretext/`, the reusable Claude Code skill.
The reference documents are the skill's own reference files, so there is one copy and
future sessions load them automatically when PreTeXt comes up.

- `docs/SKILL.md` — quick start and the rules that matter
- `docs/references/` — setup, project anatomy, core markup, **physics markup**,
  exercises, interactives, build & publish, gotchas, unit vocabulary
- `docs/templates/` — a validated physics chapter, a multi-target `project.ptx`,
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
