# pretext-authoring-plugin

A Claude Code plugin for authors writing textbooks in PreTeXt. Currently in the design phase: a wayfinder
map on GitHub Issues (#1) charting what a broadly useful authoring plugin should contain;
the plugin itself will be built here once the map is done.

- `docs/research/`, `docs/design/` — evidence and decision material for the map
- `.venv/`, `vendor-pretext/`, `scratch/`, `reference-skill-draft/` — local only (gitignored); see README.md.
  `reference-skill-draft/` is the earlier physics-flavoured skill (symlinked from `~/.claude/skills/pretext`).

## Agent instruction files

This file (`AGENTS.md`, the open convention at https://agents.md/) is the single source of agent
instructions for this repo. `CLAUDE.md` only imports it (`@AGENTS.md`) — put nothing there that
other agents would need. The same pattern is what the plugin will scaffold into authors' book repos.

## Agent skills

### Issue tracker

Issues live in this repo's GitHub Issues (`gh` CLI (command-line interface)); external PRs (pull requests) are not a triage surface. See `docs/agents/issue-tracker.md`.
