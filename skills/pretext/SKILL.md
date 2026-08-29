---
name: pretext
description: Author and build books in PreTeXt, the XML markup language for STEM textbooks, with emphasis on physics texts (SI units, LaTeX/MathJax math, TikZ and PreFigure diagrams, auto-graded exercises, WeBWorK, embedded PhET/GeoGebra/JSXGraph simulations) and on producing web HTML, print PDF, and EPUB/Kindle from one source. Use when the user mentions PreTeXt, .ptx files, pretextbook.org, `pretext build`, project.ptx, publication.ptx, a Runestone or WeBWorK textbook, or asks to write, structure, convert, validate, or publish a textbook in this system.
---

# PreTeXt

PreTeXt is an XML vocabulary for scholarly books. One source tree converts to HTML,
LaTeX/PDF, EPUB, Kindle, braille, Jupyter, and slides. Authors write *structure*;
a publication file (not the source) decides presentation.

## Quick start

```bash
pip install "pretext[all]"          # Python 3.10+; use a venv
pretext new book -d my-book         # scaffold
cd my-book
pretext build web                   # -> output/web/
pretext view web                    # local server on :8128
pretext validate --engine salve     # schema check, no Java needed
```

Three files govern everything: `project.ptx` (targets), `publication/publication.ptx`
(presentation), `source/docinfo.ptx` (facts the content needs — macros, packages, external dir).

## Non-negotiable rules

1. **Validate before debugging output.** `pretext validate` names the file, the XPath, and the line.
   A confusing build warning is usually a downstream symptom of an invalid tree.
2. **`p` is the bottleneck.** Display math *and* lists live inside a `<p>` — `<md>` as a
   direct child of `<statement>` is invalid. Punctuation goes *immediately after* `</md>`,
   outside the element.
3. **Never `pip install` outside a venv**, and put the venv's `bin` on `PATH` — the CLI shells out
   to sibling executables (`playwright`, `node`) by bare name.
4. **`xml:id` for cross-references, `label` for Runestone/WeBWorK.** Fix `label` values before
   publishing; changing one orphans student records.
5. Match the change to its home: words → `docinfo`; appearance → publication file.

## Workflows

- Starting a book, project layout, targets, directories → [references/02-project-anatomy.md](references/02-project-anatomy.md)
- Prose, divisions, math, figures, tables, cross-refs → [references/03-core-markup.md](references/03-core-markup.md)
- **Physics: units, vectors, constants, diagrams** → [references/04-physics-markup.md](references/04-physics-markup.md)
- Exercises, solutions, auto-grading, WeBWorK → [references/05-exercises-and-assessment.md](references/05-exercises-and-assessment.md)
- PhET, GeoGebra, JSXGraph, Desmos, Sage cells → [references/06-interactives.md](references/06-interactives.md)
- Building/publishing HTML, PDF, EPUB, Kindle → [references/07-build-and-publish.md](references/07-build-and-publish.md)
- Install and per-output prerequisites → [references/01-setup-and-toolchain.md](references/01-setup-and-toolchain.md)
- **Verified pitfalls — read before a long debugging session** → [references/08-gotchas.md](references/08-gotchas.md)
- Complete SI unit / prefix vocabulary → [references/09-units-vocabulary.md](references/09-units-vocabulary.md)
- **The schema — content models, and what "experimental" means** → [references/10-schema.md](references/10-schema.md)

`templates/` holds a schema-valid physics chapter, a multi-target `project.ptx`, and a
physics `docinfo.ptx`. Copy from these rather than writing markup from memory.

## Authoritative sources

- **Schema (the contract; wins over the Guide):**
  <https://pretextbook.org/doc/schema-litprog/html/pretext.html> — the comprehensive
  literate source. Per-element lookups (parents, children, attributes):
  <https://siefkenj.github.io/pretext-book/docs/reference/all-tags>.
- Guide: <https://pretextbook.org/doc/guide/html/> — its source is `doc/guide/` in the core repo.
- Core repo (XSL, schema, examples): <https://github.com/PreTeXtBook/pretext>
  — `examples/sample-article/` is the only real documentation of `<interactive>`.
- CLI repo: <https://github.com/PreTeXtBook/pretext-cli>
- Support: the `pretext-support` Google group. Run `pretext support` and paste the output.
