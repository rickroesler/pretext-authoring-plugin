# Brief 14 — The math pack: what is math-specific versus core

Ticket: [#14](https://github.com/rickroesler/pretext-authoring/issues/14). Status: **for grilling, not decided.**

## 1. The question

Which authoring knowledge is specific to mathematics textbooks and belongs in a math pack — WeBWorK,
Sage cells, theorem/proof/definition structures and numbering, PreFigure/TikZ/Asymptote, notation
macros, math-specific exercise idioms — versus what belongs in the discipline-agnostic core? What is
the pack's file shape, and what does that imply for the later physics-pack refactor?

## 2. Evidence

- **pretext-support-survey.md, Category table** — the top categories are *not* discipline-specific:
  images/diagrams ~44, markup/schema ~34, exercises/WeBWorK ~33, HTML-CSS ~28, build errors ~25,
  install ~14. Only the WeBWorK half of the exercises bucket and parts of the images bucket are
  math-flavoured; the rest is every author's pain regardless of subject.
- **feedback-loop-inventory.md §7** — `statement` has three grammar definitions, one of which,
  `StatementExerciseWW`, permits only `image, instruction, p, pre, tabular` — "the WeBWorK one is a
  different language." ⇒ WeBWorK is a genuinely separate sub-vocabulary, not a core variant.
- **feedback-loop-inventory.md §6 / §2 signal 3** — `validation-plus` has context checks for
  `var` outside `webwork`, `image[@pg-name]` outside `webwork`, WeBWorK `tabular` rule widths, and
  `fillin` inside `m`/`md`. On PreTeXt's own sample book, `var-outside-webwork` fires **10** times.
- **review-criteria.md P14, C8, C9** — stable `@label` for every Runestone/WeBWorK interactive,
  derived from `docinfo/document-id` + `edition`, decided before first hosting. C1/C2: macros live in
  `docinfo/macros`, must be semantically named, and **must not contain digits** — "MathJax will
  silently fail to read that macro and everything defined after it." C16/C17: `\text{}` only; never
  author `align`/`gather`/`equation` inside `<m>`/`<md>`.
- **review-criteria.md A10** — commutative diagrams should use `amscd` inside `<md>` (MathJax renders
  it accessibly) rather than a drawing tool. A5: tabular long-descriptions for graphs.
- **github-issues-survey.md, WeBWorK + Images sections** — WW generation is "the most failure-prone
  asset type" (cli#662); `.pg` filename collisions and chunking bugs are **open** (pretext#2699/#2700);
  `format="custom"` cannot drive WW generation (cli#419). Images: remote Asymptote server can silently
  return bad output, open since 2022 (pretext#1869); PreFigure has live edge-case bugs
  (pretext#2927/#2838).
- **pretext-support-survey.md, Recurring Q8** — authors ask *before building* for a recommendation
  between PreFigure, TikZ and raster fallbacks; and Q4, the pipx isolation trap that hides `prefig`.
- **conversion-landscape.md §2.8** — Ameya Kolarkar: Gemini "converts figures and graphs to Prefigure
  format — this has been amazing and a massive time-saver!" ⇒ PreFigure generation is a real,
  demanded, LLM-shaped task.
- **prior-art.md, Anti-patterns** — "one skill, 51 trigger phrases" works against the core+pack split;
  Pattern 2 (quarto-authoring): one reference file per feature area with an explicit
  "don't read this unless…" gate. **Portability**: Codex caps SKILL.md at 8 KB, so the *core* router
  must stay small — packs must not inflate it.
- **pretext-dev-survey.md (d)** — the entire dynamic/Runestone exercise vocabulary is experimental and
  "among the most likely parts of PreTeXt to change"; WeBWorK's `var` is standard *inside* WeBWorK.
- **skills/pretext/** (current state) — 10 references, of which `04-physics-markup.md` and
  `09-units-vocabulary.md` are clearly discipline-specific and the other eight are not. This is the
  empirical split ratio: ~2 pack files to ~8 core files.

## 3. What the evidence says is core vs. pack

**Core (discipline-agnostic):** `<m>`/`<md>`/`<mrow>` mechanics and the `p`-bottleneck; macros in
`docinfo/macros` (every STEM book has them, and C2's digit rule is universal); `theorem`/`definition`/
`example`/`proof` (they are PreTeXt block elements, not math elements — a CS or physics book uses them
identically); `exercise`/`task`/`hint`/`answer`/`solution`; numbering and `<solutions>`; images and
the PreFigure/TikZ/Asymptote *pipeline* (top pain category for everyone); accessibility.

**Math pack:** WeBWorK authoring (`<webwork>`, PG, `var`, `@pg-name`, server config, `.pg` collisions,
seed debugging) — justified by the separate `StatementExerciseWW` grammar; Sage cells and `sageplot`;
`amscd` commutative diagrams; math-notation conventions and worked macro sets; math-flavoured exercise
idioms and the numbering conventions math faculty expect.

**Contested:** PreFigure (mathematically-flavoured but the tool of choice for *any* diagram) and
theorem-environment *conventions* (as opposed to elements).

## 4. Options for the pack's file shape

| Option | Cost | Benefit |
|---|---|---|
| **A. Pack = extra `references/` files inside the core skill**, gated by a "read only if…" row in the router. | Inflates the core skill's directory; every author downloads every pack; router table grows toward the anti-pattern. | Zero new machinery; simplest possible portability story. |
| **B. Pack = a second skill** (`pretext-math`) with its own SKILL.md + references, that assumes core. | Two skills to keep in sync; cross-skill invocation is fuzzier in Codex than in Claude Code. | Clean boundary; core SKILL.md stays under 8 KB; physics pack is a symmetric copy; packs can be installed selectively. |
| **C. Pack = plugin-level component set** (commands/agents per discipline). | Claude-Code-only — violates the map's portability decision. | Best discovery inside Claude Code. |
| **D. No packs in v0.1** — one skill, math content in core, split later. | The later split is exactly the hard-to-reverse rename we want to avoid; contradicts the map. | Fastest to ship. |

## 5. Recommendation

**Option B — the math pack is a second, portable skill (`pretext-math`) that depends on the core
skill**, with three v0.1 references: WeBWorK authoring, Sage/`sageplot`, and math notation/macros +
theorem-and-numbering conventions. Reasoning: the WeBWorK grammar really is a different language
(three `statement` definitions), which is the sharpest natural seam in the evidence; and the Codex
8 KB cap makes "keep the core router small" a hard constraint rather than an aesthetic one.
Everything else the ticket lists — theorem/proof *elements*, PreFigure/TikZ/Asymptote, exercise
structure, numbering machinery — stays **core**, because the support survey shows those are the
highest-volume pains for all authors, not math authors.

**Implication for the physics refactor:** the current skill's `04-physics-markup.md` and
`09-units-vocabulary.md` become `pretext-physics`; the other eight references become core. The pack
contract is therefore: *a pack is a skill of the same shape, adding a vocabulary layer and a
conventions layer, and never redefining core mechanics.* That contract is what makes the physics pack
a copy rather than a redesign.

## 6. Grilling questions (defaults)

1. Second skill or gated references? *Default: second skill.*
2. Does the math pack ship in v0.1, or does v0.1 ship core only with the pack shape proven by physics? *Default: math pack ships, minimal (3 references), because the map says math first.*
3. Is WeBWorK in the pack or core? *Default: pack — it is a separate grammar and a separate server story.*
4. Is PreFigure core or math pack? *Default: **core** — images are the #1 support category across all disciplines, and the pipx trap is an environment issue, not a math issue.*
5. Are theorem/proof/definition core? *Default: elements core; discipline *conventions* (numbering style, `amscd`) in the pack.*
6. Does the pack carry templates? *Default: yes, one worked math chapter; templates are the highest-value pack artifact per prior-art Pattern 4.*
7. How does the core skill point at packs without listing them all in its router? *Default: one router row — "discipline-specific markup → load the matching `pretext-<discipline>` skill if installed".*
8. Do packs get their own version pin? *Default: yes, same CLI-version metadata as core.*
9. Sage: pack or out of scope? *Default: pack, thin — the support evidence is mostly install pain, which the doctor already covers.*
10. Runestone/CS: a third pack or core? *Default: neither in v0.1; the experimental-markup warning in core covers the risk.*

## 7. Hard-to-reverse → ADR candidates

- **ADR: the pack contract and the core/pack boundary** (what a pack may and may not contain; skill naming). Renaming skills breaks invocations, and re-cutting the boundary means moving reference files that other components cite.
- **ADR: WeBWorK lives in the math pack, not core** — determines whether the core skill can be trusted alone by a WeBWorK author, and is visible to users.
- (Not an ADR: which individual reference file a given topic lands in — cheap to move.)
