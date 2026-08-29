# pretext-authoring

A Claude Code plugin for authors writing textbooks in PreTeXt. Currently in the design phase: a wayfinder
map on GitHub Issues (#1) charting what a broadly useful authoring plugin should contain;
the plugin itself will be built here once the map is done.

- `docs/research/`, `docs/design/` — evidence and decision material for the map
- `.venv/`, `vendor-pretext/`, `scratch/`, `reference-skill-draft/` — local only (gitignored); see README.md.
  `reference-skill-draft/` is the earlier physics-flavoured skill (symlinked from `~/.claude/skills/pretext`).

## Agent skills

### Issue tracker

Issues live in this repo's GitHub Issues (`gh` CLI); external PRs are not a triage surface. See `docs/agents/issue-tracker.md`.
