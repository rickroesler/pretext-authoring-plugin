# pretext-authoring-plugin

A public repo (`rickroesler/pretext-authoring-plugin`) charting the design of a Claude Code
plugin for authoring textbooks in [PreTeXt](https://pretextbook.org). Started
2026-08-28 after a [pretext-dev thread with Rob Beezer](https://groups.google.com/g/pretext-dev/c/0D90duVchZg)
(PreTeXt's maintainer), who is receptive to the idea.

**Goal:** a discipline-agnostic authoring plugin — skills, commands, agents, hooks,
templates, and per-discipline "packs" — packaged as a Claude Code plugin but authored
to the portable Agent Skills format (SKILL.md + references) so the core works in other
agents too. Claude-only features (hooks, subagents, MCP) stay extras, never load-bearing.

The plugin will be built in this repo. It is currently in the design phase; building starts
once the design map below is complete.

## Status

The design is being charted as a wayfinder map on this repo's GitHub Issues:
[Map: a validated design for a PreTeXt authoring plugin](https://github.com/rickroesler/pretext-authoring-plugin/issues/1)
(issue #1). Research tickets #2–#9 are done (author pain surveys, the feedback-loop
inventory, conversion landscape, prior art, licensing, review criteria). Design-decision
tickets #10–#19 are open — component inventory, math pack scope, advanced-author track,
knowledge-currency strategy, conversion ownership, review agents, testing, licensing/
handoff, and upstream recommendations to `pretext`/`pretext-cli`.

Expected eventual home: the PreTeXtBook GitHub org, once useful, maintained by Rob's
team. Licence not yet decided — see `docs/research/licensing.md` (MIT recommended for
what we author; PreTeXt core is GPL, the Guide is GFDL).

## Method

Development is guided by [Matt Pocock's agent skills](https://github.com/mattpocock/skills), which run
in Claude Code as slash commands against this repo's GitHub Issues:

- `/wayfinder` — charts the design as a map (issue #1) of decision tickets, worked one at a time
  (research tickets AFK, grilling/prototype tickets with a human in the loop).
- `/grilling` and `/domain-modeling` — how each decision ticket is resolved.
- `/to-spec` → `/to-issues` (or `/to-tickets`) → `/implement` — the path from the finished design
  to build issues and code, once the map is done.
- `/triage`, `/code-review` — for incoming issues and PRs when the plugin exists.

`docs/agents/` holds the conventions those skills read (issue-tracker operations, labels).

## Repo layout

| Path | What |
|---|---|
| `skills/pretext/` | The current skill — physics-flavoured (SI units, LaTeX/MathJax math, TikZ/PreFigure diagrams, WeBWorK, PhET/GeoGebra/JSXGraph). Symlinked from `~/.claude/skills/pretext`. Will be refactored into a discipline-agnostic core plus discipline packs (math pack first). |
| `skills/pretext/references/` | Setup, project anatomy, core markup, physics markup, exercises, interactives, build & publish, gotchas, unit vocabulary, schema. |
| `skills/pretext/templates/` | A validated physics chapter, a multi-target `project.ptx`, a physics `docinfo.ptx`. |
| `docs/research/` | 8 evidence documents backing the closed research tickets (support/dev/GitHub surveys, conversion landscape, feedback-loop inventory, prior art, licensing, review criteria). |
| `docs/design/briefs/` | Decision briefs for the open design tickets. |
| `docs/design/prototypes/` | Design prototypes, e.g. the newcomer-first-hour storyboard. |
| `docs/agents/` | Conventions for how agent skills use this repo's issue tracker. |

## Local toolchain (gitignored, not committed)

| Path | What |
|---|---|
| `.venv/` | PreTeXt CLI 2.51.0 — recreate with `python -m venv .venv && pip install "pretext[all]"`. |
| `vendor-pretext/` | Shallow clone of `PreTeXtBook/pretext` — XSL, schema, `examples/`, and the Guide's source. Grep this when docs and behaviour disagree. |
| `scratch/` | Throwaway projects used to verify claims in the docs against a real build. |

## How to use the skill today

Symlink (or copy) `skills/pretext/` into `~/.claude/skills/pretext` and it loads
automatically whenever a session mentions PreTeXt, `.ptx` files, or `pretext build`:

```bash
ln -s "$(pwd)/skills/pretext" ~/.claude/skills/pretext
```

It is verified against PreTeXt CLI 2.51.0 for `pretext validate --engine salve` and
`pretext build web|tex|ebook|kindle` (PDF needs a TeX installation this repo's
development machine doesn't have).
