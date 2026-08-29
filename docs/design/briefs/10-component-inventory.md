# Brief 10 — Component inventory and the v0.1 cut line

Ticket: [#10](https://github.com/rickroesler/pretext-authoring-plugin/issues/10). Status: **for grilling, not decided.**

## 1. The question

Which components does the plugin contain (skills, commands, agents, hooks, tools/MCP, templates,
packs), which make v0.1, and — for each — what author pain it answers, which persona it serves, and
whether it is portable-skill content or a Claude-Code-only extra? Plus: are any Claude-only features
allowed to be load-bearing (map's default: none)?

## 2. Evidence

- **feedback-loop-inventory.md §0, §10** — `pretext validate <target> --report-form terse` is the gate
  (file/XPath/line/check per line, exit 0/1/2, 0.9–1.0 s jing, 1.8 s on a 10 320-line book);
  `pretext build` "exits **0** on schema-invalid source, on unknown elements, on missing image files
  and on broken LaTeX" (§4.2) and ships an invented element's text into the HTML (§5). ⇒ a
  validate→build repair loop is the core component, and it must never gate on build alone.
- **feedback-loop-inventory.md §1** — "the first-run experience of `pretext validate` on a clean
  machine is a failure": no jing, exit 2. ⇒ environment doctor is a *prerequisite* to the loop.
- **pretext-support-survey.md, Category table + Recurring questions #1** — 284 threads; images ~44,
  markup/schema ~34, exercises/WeBWorK ~33, HTML-CSS ~28, build errors ~25, install ~14. Q1 ("where
  does this config attribute go?") is asked "nearly verbatim" for dark mode, `author.deprecations.all`,
  `author.tools`, braille, favicons, custom XSL. ⇒ config-placement lookup skill.
- **pretext-support-survey.md, Recurring questions #4, #7 and "What a validating agent would have
  caught"** — pipx venv isolation, Windows lxml/cp1252/permissions cluster: "none of these are markup
  mistakes; they're environment mismatches an agent with a 'known Windows issues' checklist … could
  flag proactively."
- **github-issues-survey.md §2 (table split)** — pretext-cli = install/CLI/deploy/Codespaces friction
  (newcomer); pretext core = cross-target rendering (advanced). §3.1: silent asset-generation failure —
  "the build reports success but the expected image/asset is missing" ⇒ post-build artifact check.
- **review-criteria.md, Summary** — ~50 criteria in three tiers; the mechanical tier is *already
  implemented* in `pretext-validation-plus.xsl` and "reusable as a standalone XPath/script"
  (A1, A14, A15, A16, C11, C12, C18); a large LLM-judgeable tier (A4, C10, P3, P13) remains.
- **prior-art.md, Patterns 1–7 / Anti-patterns** — validate-fix loop with a fix-naming validator
  (anthropics docx/pptx, 172k★); task-router SKILL.md + one reference per feature area
  (quarto-authoring, 485★); build-as-hard-gate (TinyUSB `sphinx-build -W`); template-first;
  "thin skills with no validation and no depth see near-zero adoption" (1★ negative datapoint);
  "one skill, 51 trigger phrases" is an anti-pattern.
- **prior-art.md, Portability constraints** — portable: `SKILL.md` + `references/` + `scripts/` +
  `assets/`, frontmatter `name`/`description` only. Non-portable: `commands/`, `agents/`,
  `hooks/hooks.json`, `.mcp.json`, `allowed-tools`. **Codex hard-caps SKILL.md body at 8 KB.**
- **conversion-landscape.md §6** — `@pretextbook/import` has **no CLI entry point**; "a 12-line Node
  wrapper gave a terminal agent full access to it" ⇒ a thin shim is cheap and high-value.
- **pretext-dev-survey.md (d)** — `experimental-features.xml` lists the whole dynamic-exercise
  vocabulary as "among the most likely parts of PreTeXt to change"; production markup is low-churn.

## 3. Options for the v0.1 cut

| Option | Cost | Benefit |
|---|---|---|
| **A. Thin core:** one `pretext` skill + validate/build loop script + doctor. | Leaves the top three pain categories (images, exercises, config placement) to raw model knowledge; risks the 1★ "thin skill" failure mode. | Smallest surface, ships fastest, trivially portable. |
| **B. Router core + reference set + 3 scripts (doctor, validate-loop, schema-query) + templates; math pack; commands as thin wrappers; no hooks/MCP.** | ~10 reference files and 3 scripts to write and keep current; two-persona routing to design. | Matches every prior-art winner; covers the four highest-count pain categories; 100 % portable core with Claude-only sugar on top. |
| **C. B + reviewer agents (lint + LLM reviewers) + save hook + schema MCP server.** | Agents/hooks/MCP are Claude-only and would be load-bearing; MCP duplicates a 52-line script that runs in 0.03 s; hook noise on a book with 196 pre-existing messages. | Best in-Claude UX; automatic validation on save. |
| **D. B + lint reviewer only (validation-plus lifted as a script), reviewers deferred.** | One extra script; overlaps `pretext validate` output. | Gives a per-file/per-diff filter the CLI lacks (§9.11) without Claude-only dependencies. |

## 4. Recommendation

**Option B for v0.1, with D's lint reviewer folded in as a script rather than an agent** — and
**no Claude-only feature load-bearing**, per the map. Reasoning: every high-adoption prior-art skill
is "task-router + deep references + a validator that names the fix"; every low-adoption one lacks the
validator. The evidence says the plugin's differentiated value is (i) the doctor that makes
`pretext validate` work at all on a clean machine, (ii) the validate→build repair loop, and (iii)
schema-shaped knowledge the converters and the Guide leave to the author. Agents, hooks and MCP add
nothing that a script plus a skill cannot do, cost portability, and cannot travel to Codex — defer to
v0.2 as optional sugar. LLM reviewers wait until the review-criteria ticket (#16) fixes their rubric.

## 5. Candidate inventory

Type key: S=skill, C=command, A=agent, H=hook, T=tool/script, TPL=template, P=pack.
Portability: **P**=portable skill content, **CC**=Claude-Code-only.

| # | Component | Type | Persona | Pain it answers (cite) | Port | v0.1? |
|---|---|---|---|---|---|---|
| 1 | `pretext` core skill (task-router SKILL.md, <8 KB) | S | both | prior-art Pattern 2 + Codex 8 KB cap | P | **in** |
| 2 | `references/` (setup, project anatomy, core markup, exercises, build/publish, gotchas, schema, config-placement) | S | both | support-survey categories: markup ~34, exercises ~33, build ~25 | P | **in** |
| 3 | **Environment doctor / preflight** (`scripts/doctor`) — jing/salve/Java/node/TeX/pdf2svg/pipx-isolation/Windows checks; wraps `pretext support` | T | newcomer | feedback-loop §1 (validate exits 2 out of the box); support-survey Q4, Q7; gh cli#218/#224/#257 | P (script) | **in** |
| 4 | **validate→build repair loop** (`scripts/check`) — terse parse, filter `check==experimental`, filter by touched file, then `build` for xref/xml:id | T | both | feedback-loop §0, §4.2, §9.11; conversion-landscape §5.3 (`@width` builds-but-validates) | P | **in** |
| 5 | **Config-placement lookup** (project vs publication vs docinfo) reference | S | both | support-survey Recurring Q1 (≥6 features, asked near-verbatim) | P | **in** |
| 6 | **Schema query** (`scripts/rng-children.py`, 52 lines, 0.03 s, dev-vs-prod diff) | T | advanced | feedback-loop §7 (`statement` has three answers; a table cannot express it) | P | **in** |
| 7 | **Experimental/deprecated-markup warning** reference + check | S+T | advanced | pretext-dev (d) full experimental list; feedback-loop §3.3 `check=experimental` | P | **in** |
| 8 | Templates (project.ptx, publication.ptx, docinfo, chapter) | TPL | newcomer | prior-art Pattern 4; licensing §3 (own directory, headers intact) | P | **in** |
| 9 | **Conversion shim** around `@pretextbook/import` + repair loop | T+S | newcomer | conversion-landscape §6 ("no CLI entry point"; 29/232 repair lines) | P | **in** (see #15) |
| 10 | **Lint reviewer** lifted from `pretext-validation-plus.xsl` (per-file/per-diff filter) | T | both | review-criteria A1/A14/A15/A16/C11/C12/C18 "directly reusable"; feedback-loop §9.11 | P | **in (thin)** |
| 11 | Math pack | P | both | map decision (math-first); see #14 | P | **in** |
| 12 | Post-build artifact verification (expected assets exist, non-trivial size) | T | both | github-issues §3.1 (silent asset failure, cli#1117, pretext#1869) | P | **in (small)** |
| 13 | `/pretext:new`, `/pretext:check`, `/pretext:convert`, `/pretext:doctor` | C | both | ergonomics only — each wraps a skill/script | CC | in, **non-load-bearing** |
| 14 | LLM reviewers (accessibility, consistency, pedagogy) | A | advanced | review-criteria LLM-judgeable tier (A4, A13, C10, P3, P13) | CC (skill-able) | **out** → #16 |
| 15 | Save/pre-build hook (auto-validate `.ptx`) | H | both | feasible (0.9 s) but noisy: sample-book emits 196 messages, exit 1 (§3.5) | CC | **out** |
| 16 | Schema MCP server | T | advanced | duplicates #6 at higher cost; feedback-loop §7 says jing's "expected" list beats static query | CC | **out** |
| 17 | Physics pack (refactor of current skill) | P | both | map: physics second | P | **out** |
| 18 | Deploy/Runestone troubleshooting reference | S | both | support-survey Q2 (deploy silently succeeds); gh cli#725/#503/#493 | P | **out** (v0.2) |
| 19 | Rendered-output/visual check (curl + playwright, backslash scan for bad LaTeX) | T | advanced | feedback-loop §8, §9.1 (LaTeX in `<m>` unchecked end-to-end) | P (heavy deps) | **out** (v0.2) |

## 6. Grilling questions (one at a time, with defaults)

1. Is the core one skill or several from day one? *Default: one router skill + references; split only when SKILL.md nears 8 KB.*
2. Does the doctor ship as a script or as prose in a reference? *Default: script — it must produce a verdict, not advice.*
3. jing, salve or server as the recommended default engine? *Default: jing if present (its "expected element" list is the best repair hint), salve as no-Java fallback, server last (16 s + uploads source).*
4. Does the loop stop at `validate` clean, or require `build` clean too? *Default: both — `@width` proves validate-clean ≠ build-clean.*
5. Is the lint reviewer a script that re-runs `validation-plus`, or a post-filter over the terse report? *Default: post-filter (free, no XSLT/entity path problems).*
6. Are slash commands allowed to contain any logic? *Default: no — pure wrappers, so Codex loses nothing.*
7. Namespace: `pretext:` or `ptx:`? *Default: `pretext:` (matches the CLI and survives the org handoff).*
8. Does v0.1 include conversion at all? *Default: yes, as the thin shim only (see brief #15).*
9. Post-build artifact verification: in v0.1 or v0.2? *Default: v0.1, minimal (count + non-zero size).*
10. Any Claude-only feature load-bearing? *Default: none.*

## 7. Hard-to-reverse → ADR candidates

- **ADR: portability constraint** — portable Agent Skills core, Claude-only features never load-bearing, SKILL.md under Codex's 8 KB cap. Reverses badly once references and commands are written against it.
- **ADR: skill split / core+pack boundary** — renaming or resplitting skills breaks user invocations and pack layout.
- **ADR: plugin + command namespace** — public identifier; survives the PreTeXtBook handoff.
- **ADR: validate-is-the-gate, build-is-not** — the loop contract every component depends on.
