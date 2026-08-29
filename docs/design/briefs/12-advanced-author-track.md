# Brief 12 — What the advanced-author track must get right

Ticket: [#12](https://github.com/rickroesler/pretext-authoring-plugin/issues/12). Status: **for grilling, not decided.**

## 1. The question

For an author who already has a PreTeXt book: what must the plugin get right to be trusted, and what
would make them stop using it? Which candidate capabilities are skill knowledge, which need a tool,
and which are out of reach?

## 2. Evidence

- **feedback-loop-inventory.md §5 (breakage table)** — an *invented* element builds with exit 0, no
  message, and its text is shipped into the HTML (`grep` shows "invented element content" in
  `sec-section-name.html`). A deprecated element warns that content "will silently not appear."
  ⇒ hallucinated markup is the single trust-destroying failure, and only `validate` catches it.
- **feedback-loop-inventory.md §3.4** — jing "enumerates every element permitted at that point";
  salve truncates at 12 + "… (58 more)". §7: `statement` has **three** definitions in the grammar
  (`Statement`, `StatementExercise`, `StatementExerciseWW`) — "a reference table in a skill file
  cannot express that; a query can."
- **feedback-loop-inventory.md §3.5** — PreTeXt's own `examples/sample-book` emits 196 messages and
  **exit 1**. "Any agent policy of 'iterate until validate is silent' must be scoped to messages in
  files the agent touched." ⇒ on an existing book, an unscoped loop is an instant trust failure.
- **feedback-loop-inventory.md §2 signal 7 / §9.3** — `publication.ptx` is **never validated by the
  CLI (command-line interface)**; `levl="2"` and `<bogus-element/>` are silently ignored; jing + `publication-schema.rng`
  catches them (with one false positive: shipped schema requires `epub/cover`).
- **pretext-support-survey.md, Recurring Q1** — "where does this config attribute go" asked
  near-verbatim for `@provide-dark-mode`, `author.deprecations.all`, `author.tools`, braille,
  favicons, custom XSL (Extensible Stylesheet Language). And Q5 / "What a validating agent would have caught": Rob Beezer said schema
  validation would have caught the `<idx>`-in-`<title>` "TeX capacity exceeded" family.
- **pretext-support-survey.md, "HTML tolerates structural choices LaTeX rejects"** — named by Beezer
  as a common misconception; discovered only after a failed print build.
- **github-issues-survey.md §2 split** — core-repo issues are dominated by cross-target rendering
  correctness (HTML fine, PDF/EPUB (Electronic Publication)/Runestone broken): images 73, pdf/LaTeX 69, html/css/MathJax 56.
  §3.3: PDF/LaTeX failures are "the weakest error-attribution path" (cli#690/#806/#691/#812 open);
  always build with `-l/--latex`. §3.6: Runestone `activity`/`fillin`/timed exams silently misbehave
  even when validate and build are clean (pretext#2458, #2635, #2586/#2683).
- **review-criteria.md P14, C8, C9** — every Runestone/WeBWorK interactive **must** carry a stable
  `@label`, decided before first hosting: "changing it orphans database records/student progress";
  `label` ≠ `xml:id`; `document-id` "must never change". Mechanical checks.
- **pretext-dev-survey.md (a), (d)** — churn is ~1 notice per 1–2 weeks but almost always
  deprecate-then-remove with the old form still working; hard breaks are confined to the
  `experimental-features.xml` vocabulary. (c): maintainers steer authors to publication-file switches
  rather than custom XSL/CSS.
- **feedback-loop-inventory.md §9.1, §9.10** — LaTeX inside `<m>`/`<md>` is unchecked end to end
  ("every math error reaches the reader"); PDF signals are unreachable without a TeX install.

## 3. Options

| Option | Cost | Benefit |
|---|---|---|
| **A. Knowledge-only** — deep references, no new tooling; rely on `pretext validate`. | Leaves publication.ptx unvalidated, diff-scoping unsolved, label stability unchecked; advanced authors hit the 196-message wall on run one. | Cheapest, fully portable, no maintenance of scripts. |
| **B. Knowledge + three small tools:** diff-scoped validate report, `publication.ptx` jing check, schema context query. | Three scripts to maintain; publication-schema false positive (`epub/cover`) must be suppressed. | Fixes the three trust-critical gaps the CLI genuinely lacks; each is <60 lines. |
| **C. B + a "safe-edit contract"**: never touch generated files, never restructure WW (WeBWorK)/Runestone-bearing activities, always preserve `@label`, always run `build tex`/`-l` before claiming PDF works. | Constrains the agent, may frustrate authors who *want* restructuring. | Directly answers "what would make them stop using it" — the failure modes are all silent damage to a working book. |
| **D. C + cross-target proof tooling** (playwright render check, PDF page/box inspection). | Needs TeX + browsers; unavailable on most author machines (§8, §12). | Only route to the biggest core-repo pain category. |

## 4. Recommendation

**Option C.** The advanced author's trust is destroyed by *silent damage to a working book*, not by
missing features. The three things the plugin must get right, in order:

1. **Never emit markup the schema does not permit** — query the schema (or jing's "expected" list)
   before writing unfamiliar markup, rather than iterating against the validator; and never treat a
   clean `build` as evidence, because invented elements pass it.
2. **Scope every check to the diff.** On a real book the baseline is not zero (196 messages in
   PreTeXt's own sample), so report by touched file and never "fix" pre-existing findings unasked.
3. **Preserve identifiers and structure.** `@label`, `xml:id`, `document-id` are one-way doors;
   restructuring WeBWorK/Runestone activities has known breakage (pretext#3008).

Then: publication-vs-source-vs-docinfo placement is *skill knowledge plus one jing invocation* (the
schema exists, it is just not wired in); experimental/deprecated awareness is skill knowledge fed by
`experimental-features.xml`; PDF/EPUB/Runestone cross-target correctness is **out of reach** in v0.1
and should be stated as such — advise `-l/--latex`, `build tex` smoke tests, and "proof the PDF
yourself", rather than pretending to verify it. Option D moves to v0.2 behind a capability check.

## 5. Grilling questions (defaults)

1. Diff-scoping: files-touched, or a stored baseline of pre-existing messages? *Default: files-touched (simpler, no state to go stale).*
2. Do we ship the `publication.ptx` jing check knowing it false-positives on `epub/cover`? *Default: yes, with that one check suppressed and reported upstream.*
3. Schema-context answers: static query script, or a jing probe-document oracle? *Default: script first (offline, 0.03 s); document the probe trick for the ambiguous cases.*
4. Is "never restructure a WeBWorK/Runestone-bearing activity without asking" a hard rule? *Default: yes — hard rule.*
5. Does the plugin ever edit `publication.ptx` unprompted? *Default: no — propose the change, name the file/element/attribute, let the author apply it.*
6. Custom XSL/CSS: support or steer away? *Default: steer to publication switches first (maintainers' own repeated ask), support custom XSL as documented last resort.*
7. Large-book performance: any special handling? *Default: none needed — 1.8 s validate on 10 320 lines; revisit only with a counterexample.*
8. Do we claim anything about PDF correctness? *Default: no. State "print output is unverified unless TeX is installed"; always pass `-l`.*
9. Runestone-fragile constructs: maintained checklist, or defer to upstream? *Default: ship a short checklist (fillin `@mode`, activity placement, timed exams) with a recheck-after-upgrade note.*
10. Deploy: drive `pretext deploy`, or advise only? *Default: advise + detect which of the two deploy methods the repo uses; never run a deploy that can hang on credentials without a timeout.*

## 6. Hard-to-reverse → ADR (Architecture Decision Record) candidates

- **ADR: diff-scoped checking contract** (what "clean" means on a pre-existing book) — every reviewer, hook and command inherits it.
- **ADR: identifier-stability rule** (`@label`/`xml:id`/`document-id` are never rewritten by the agent) — a violation is unrecoverable for a hosted book.
- **ADR: the plugin does not claim cross-target (PDF/EPUB/Runestone) verification in v0.1** — sets the trust boundary publicly and is awkward to walk back after authors rely on it.
