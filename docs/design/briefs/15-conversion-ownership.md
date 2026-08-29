# Brief 15 — Conversion ownership: what the plugin does itself versus delegates

Ticket: [#15](https://github.com/rickroesler/pretext-authoring-plugin/issues/15). Status: **for grilling, not decided.**

## 1. The question

How much manuscript conversion does the plugin own? (a) drive a converter and repair its output;
(b) LLM (large language model)-native conversion chapter-by-chapter validated by `pretext validate`; (c) a hybrid where a
converter runs first and the LLM fixes what it drops. For which input formats does each apply, and
what fidelity does a newcomer get? The answer defines `/pretext:convert` — or rules it out of v0.1.

## 2. Evidence

- **conversion-landscape.md §0** — `pretext import` was **removed** from the CLI (command-line interface) on 2026-08-24
  ("an experimental feature that never reached a stable enough state to be useful"); its replacement
  `@pretextbook/import` (npm, v0.9.0, 2026-08-22) is "not a CLI command — it is a library behind the
  VS Code extension … and the pretext.plus web app."
- **conversion-landscape.md §5 (head-to-head, same 137-line LaTeX input)** — plasTeX: 20 schema
  errors, **239 changed lines against 232** of output, 4 `\autoref`s lost with no trace.
  `@pretextbook/import`: 14 errors of which **6 are deliberate `<TODO>` markers**, **29 changed lines
  against 232** — "roughly an eighth of the plasTeX repair", "Information lost with no trace: none
  observed". Unknown macros become `<TODO type="unknown-macro">`, so validation "fails on purpose and
  points at every one … precisely the affordance an LLM repair pass wants."
- **conversion-landscape.md §6** — "The plugin should **not** write a LaTeX parser"; "**No CLI entry
  point for the maintained route** … A 12-line Node wrapper gave a terminal agent full access to it —
  the plugin can own that thin shim (and should surface `res.warnings`, which is a ready-made review
  list)"; "**The repair pass is the product.**"
- **conversion-landscape.md §5, finding 3** — after the plasTeX output validated cleanly,
  `pretext build web` still failed: `PTX:ERROR: width (50) should be given as a percentage` — "a
  conversion loop that stops at `validate` hands the author a document that does not build. **The loop
  must be `validate` → `build`.**"
- **conversion-landscape.md §5, finding 2** — "**No converter produces exercise semantics.**"
  `<exercises>`/`<exercise>`/`<task>`/`<hint>`/`<answer>`/`<solution>` is untouched by plasTeX, by
  `@pretextbook/import`, and by pandoc-pretext. "For a textbook this is the single largest manual
  bill, and it needs judgement — my attempt to script the `<ol>`→`<exercises>` transform produced
  mismatched tags because nested parts must become `<task>`." (Reinforced by CLI 2.51.0 deprecating
  ordered-lists-as-exercise-parts.)
- **conversion-landscape.md §6 table** — Word/.docx/RTF (Rich Text Format)/HTML: **pandoc-pretext only**
  (`@pretextbook/import` does not accept `.docx`). Jupyter: nothing maintained (`jpconvert` abandoned
  2019). MyST: no bridge. PDF/handwritten: LLM directly, "which is what the community already does".
- **conversion-landscape.md §2.2 (the Guide)** — Farmer/UTMOST (Undergraduate Teaching in Mathematics with Open Software and Textbooks) is the only route the Guide endorses,
  "95 percent correct … 20 times less effort"; and the Guide's stated prerequisite: be able to build
  the sample article and book *before* converting. Also §2.2: "LaTeX allows authors enough freedom
  that it is impossible to accurately discern intent in a totally automated way."
- **conversion-landscape.md §2.8** — community is already doing LLM conversion (Gemini
  section-by-section; Fitch converted handwritten mathematics); Beezer: preliminary AI runs on good
  LaTeX "approached the quality of Farmer's tool", results range "incredible" to "extremely
  frustrating".
- **conversion-landscape.md §4.1 item 9** — the importer's generated `project.ptx` is **rejected by
  CLI 2.51.0** (child elements vs. attributes) and had to be replaced with one from `pretext new`.
- **prior-art.md, Pattern 1** — the validate-fix loop with a fix-naming validator is the pattern that
  works in production (anthropics docx/pptx); **Anti-pattern**: the 1★ skill whose correctness "relies
  entirely on the underlying converter not erroring."
- **pretextbook-org-survey.md §2 (updated after #21)** — confirms and sharpens conversion-landscape's
  finding: `pretext-tools` (the `oscarlevin.pretext-tools` VS Code extension) already wraps
  `@pretextbook/import` as an **"import wizard"** — a registered VS Code command (`importProject`)
  with its own cleanup rules and a preview-before-write UI. But the underlying package **still has
  no CLI (command-line interface) entry point of its own** — the wizard is editor-only, reachable
  only from inside VS Code. This is decisive for a terminal/agent context (Codex, a headless Claude
  Code run): there is nothing to shell out to without the 12-line Node wrapper brief #15 already
  proposes. ⇒ the plugin's thin shim is not a stopgap until upstream ships a CLI, it is the only way
  an agent reaches this converter at all; keep owning it.
- **pretextbook-org-survey.md §7.1, the `community` repo's Day1Notes.md wishlist (updated after
  #21)** — an on-record, first-party (if informal) community ask for exactly this tool, independent
  of the `pretext-dev` origin thread already in the map: *"Community control with review... Perhaps
  AI to go from lite documents to pretext? ... What subset of LaTeX converts to PreTeXt?"* —
  corroborates that AI-assisted, validate-gated conversion is a named gap, not a speculative bet.

## 3. Options

| Option | Cost | Benefit |
|---|---|---|
| **A. Own nothing — document the routes.** | Leaves the newcomer at exactly the step the Guide already fails them on; forfeits the clearest measured win in the research. | Zero maintenance; no npm dependency. |
| **B. Thin shim + repair loop (hybrid (c)).** Node wrapper over `@pretextbook/import`, surface `res.warnings`, replace the manifest with `pretext new`'s, then iterate `validate` → `build`, resolving `<TODO>`s and authoring exercise semantics. **(Updated after #21)** confirmed as the *only* agent-reachable route: `pretext-tools`'s own "import wizard" wraps the same package but as a VS Code-only command (`importProject`), with no CLI — there is no upstream CLI to wait for. | A Node/npm dependency and one small script to keep current with a v0.x library; the manifest workaround is a moving target. | 29/232 measured repair cost; the `<TODO>` markers make repair targeted rather than speculative; matches Rob's own ask (schema + validate-iterate) and the `community` repo's on-record AI-conversion wishlist item (org-survey §7.1). |
| **C. LLM-native conversion (b)** — read the `.tex`, emit PreTeXt directly, gated by validate→build. | Loses the importer's `<TODO>` trail and macro preservation; token cost scales with the book; fidelity unmeasured at book scale ("extremely frustrating" tail). | No toolchain dependency; handles PDF/handwriting/Jupyter/MyST where nothing else exists. |
| **D. B for LaTeX/Markdown, pandoc for Word, C as the documented fallback, Farmer/UTMOST for whole open-licensed books.** | Most surface to document; three routes to keep honest. | Matches the evidence per-format instead of forcing one mechanism. |

## 4. Recommendation

**Option D, built on B** — the plugin owns *one* thing: the **shim + repair loop**, and routes
everything else.

- **LaTeX and Markdown:** shim `@pretextbook/import`, then repair. Fidelity a newcomer should be
  promised: structure, prose, math, macros and cross-references arrive largely intact (~29 repair
  lines per 232 on a clean sample); frontmatter placement, `<figure>`/`<table>` child order, `@width`
  units, `<introduction>` wrapping and every `<TODO>` need fixing; **exercise semantics must be
  authored from scratch** and that is the largest bill.
- **Word/RTF/HTML:** pandoc-pretext, then the same repair loop; warn that Word carries no theorem or
  exercise semantics to preserve.
- **PDF, handwriting, Jupyter, MyST:** LLM-native (C), explicitly labelled lower-fidelity, still gated
  by validate→build.
- **Whole open-licensed textbook:** point at Farmer/UTMOST, as the Guide does — do not compete with it.

The loop is `validate` → `build`, never `validate` alone (the `@width` finding), and repair is scoped
to the converted files. `/pretext:convert` is worth shipping in v0.1 *as a thin wrapper over the shim
and the loop* — the repair pass is the product, and it is the same machinery the core skill already
needs. Two upstream recommendations fall out and belong in #19: the rejected generated `project.ptx`,
and `\autoref` in the unknown-macro table.

**(Updated after #21)** The recommendation is unchanged but now stronger, not weaker: org-survey §2
confirms `pretext-tools`'s own "import wizard" is a thin VS Code-only wrapper around the same
`@pretextbook/import` package with **no CLI of its own** — so the plugin's shim is not filling a
temporary upstream gap, it is the only way a terminal/agent context (Codex or a headless Claude Code
run) reaches this converter at all. And the `community` repo's Day1Notes.md wishlist (org-survey
§7.1) is on-record, first-party demand for exactly this ("AI to go from lite documents to pretext"),
independent of the `pretext-dev` origin thread — the ask is corroborated twice over now, not once.

## 5. Grilling questions (defaults)

1. Do we take an npm/Node dependency in v0.1? *Default: yes, but degrade gracefully — if Node is absent the doctor says so and conversion falls back to (C).*
2. Do we pin `@pretextbook/import`? *Default: yes, a pinned version, re-verified per plugin release; it is v0.9.0 and moving.*
3. Do we work around the broken generated `project.ptx` (replace with `pretext new`'s) or fail loudly? *Default: work around it, print what we did, and file upstream.*
4. Conversion granularity: whole project at once, or chapter-by-chapter? *Default: whole project for structure, then repair chapter-by-chapter (matches community practice and keeps diffs reviewable).*
5. Does the plugin author exercise semantics automatically, or propose them? *Default: propose per exercise division and confirm once, then apply — it needs judgement and it is the largest bill.*
6. `<TODO>` markers: resolve automatically or list for the author? *Default: resolve mechanically where the target is unambiguous (e.g. `\autoref` → `<xref>`), list the rest.*
7. Do we ever call the remote validation service (`--method server`, 16 s, uploads source) during conversion? *Default: no — privacy; prefer jing/salve, say so in docs.*
8. Do we support `.docx` in v0.1 given pandoc is an extra install? *Default: document the route, doctor-check for pandoc, don't bundle it.*
9. What do we promise a newcomer about fidelity? *Default: publish the measured numbers (29/232 on clean LaTeX) rather than an adjective, and state that exercises are hand-authored.*
10. Does conversion enforce the Guide's prerequisite (build the sample book first)? *Default: soft — offer `pretext new` + first build before converting, don't block.*

## 6. Hard-to-reverse → ADR (Architecture Decision Record) candidates

- **ADR: the plugin does not implement a converter** (no LaTeX parser; delegate to `@pretextbook/import`/pandoc and own the repair loop). Reversing means owning a parser forever.
- **ADR: external-toolchain dependency policy** (may the plugin require Node/npm, and what degrades when it is missing) — affects the doctor, packaging, and the Codex portability story.
- **ADR: the conversion loop contract is `validate` → `build`, scoped to converted files.** Shared with brief #10/#12; worth stating once.
