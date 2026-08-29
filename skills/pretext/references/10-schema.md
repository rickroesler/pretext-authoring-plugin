# The Schema

The RELAX-NG schema is the formal specification of the PreTeXt vocabulary and, in the
project's own words, *the contract between authors and implementers*: if your source
validates, a conversion should render it accurately or tell you why it cannot. When the
Guide and the build disagree, the schema wins. Read it before guessing.

## Where it is

| Form | Where |
|---|---|
| Documented (literate) source — **the one to read** | one page: <https://pretextbook.org/doc/schema-litprog/html/pretext.html> (this is the comprehensive, current rendering — 54 sections, dated 2026-08-26 at the time of writing). Chunked: `…/schema-litprog/html/`. PDF: `…/schema-litprog/pretext.pdf`. Source: `schema/pretext.xml` in the core repo. |
| Compact syntax, generated from the above | `schema/pretext.rnc` |
| XML syntax (what `jing` actually runs) | `schema/pretext.rng` |
| Development overlay | `schema/pretext-dev.rnc` / `.rng` |
| W3C XSD | `schema/pretext.xsd` |
| Per-element browser (Jason Siefken et al.) | <https://siefkenj.github.io/pretext-book/docs/reference/all-tags> |
| Advisory prose attached to experimental reports | `schema/experimental-features.xml` |

Your installed copy is at `~/.ptx/<version>/core/schema/` — use that one, since it matches
the CLI you are actually building with.

**Use the browser for lookups, the literate source for understanding.** Each element page
(e.g. `…/docs/reference/elements/quantity`) lists description, allowed children, allowed
parents, and attributes — which answers "can X go inside Y?" in one step. `pretext.xml`
explains *why* the grammar is shaped that way, which is what stops you fighting it.

Only `pretext.xml` accepts pull requests; every other file is derived.

## Reading RELAX-NG compact syntax

Patterns describe how elements and attributes combine.

| Notation | Meaning |
|---|---|
| `,` | sequence — these, **in this order** |
| `&` | interleave — these, in any order |
| <code>&#124;</code> | choice |
| `?` `*` `+` | zero-or-one, zero-or-more, one-or-more |
| `=` | define a pattern |
| <code>&#124;=</code> | *add to* an existing pattern (`combine="choice"`) |

Named patterns (`BlockDivision`, `TextParagraphItem`) are the vocabulary's real joints;
element names are just leaves hanging off them. To answer "where may this go", find the
pattern that names it and then find who references that pattern.

## Production vs development: what "experimental" means

There are two grammars, and the relationship between them is load-bearing.

`pretext-dev.rnc` is a **purely additive overlay**: a bare `include` of the production
schema, wholly new named patterns, and additions to production patterns only via `|=`.
Never a replacement, never a restriction. So the development language is a strict superset
of the production language *by construction*, and validation checks that invariant on
every run.

That is what lets `pretext validate` split its report in two:

- a message from the **development** schema is a **genuine error** — invalid under both;
- a construct valid only under the **development** schema is **experimental** — legal, and
  it builds, but its markup may change without a deprecation cycle.

The experimental section is an inventory, not a fault list. Read it each time; it tells you
which parts of your book are standing on ground that may move.

The prose attached to each experimental report comes from `experimental-features.xml` and
is purely decorative — what gets flagged is decided by comparing the two grammars, so a
stale entry can never mislead you, and an entry simply stops firing when the construct is
promoted.

### The complete experimental surface

Five fragments, from the last section of `pretext.xml`:

| Fragment | What it covers |
|---|---|
| `interactive` | the whole `<interactive>` / `<slate>` / `<static>` / `<instructions>` vocabulary |
| `exercise-dev` | interactive exercises: `choices`/`choice`, `feedback`, `statement/@correct`, `fillin/@answer|@mode|@name`, `evaluation`/`evaluate`/`test`, `numcmp`/`strcmp`/`jscmp`/`mathcmp`, `setup`/`setupScript`/`eval`, `matching`, `cardsort`, `areas`, `response`, `program-preamble`/`program-postamble` |
| `solutions-dev` | `solutions/@scope`, `@reading`, `@worksheet` |
| `proof-like` | `proof` and its relatives as a general block |
| `frontmatter-dev` | additions to the front matter |

Also dev-only: `myopenmath`, `datafile`, `dynamic`, `jsimports`/`jslibrary`, `xhtml:` slates.

Counted from the grammars of core 2.51.0: **352 elements in production, ~55 more in
development.** For a physics book, note that *both* the interactive-simulation vocabulary
and the auto-graded-exercise vocabulary are entirely on the development side. They work,
they are widely used, and they are the two things most likely to change under you.

## The content models that actually bite

These are transcribed from `pretext.rnc`; each is a rule people break repeatedly.

### Three tiers of text

The schema distinguishes what you may write *by where the text can end up*. Text that gets
recycled into a table of contents or an index is deliberately restricted.

| Tier | Used for | Allows |
|---|---|---|
| **Simple text** | personal names, and similar | characters and empty elements only |
| **Short text** (`TextShort`) | index headings (`h`, `see`, `seealso`), `shorttitle`, `creator`, `instruction`, `line`, `notation` entries, bibliography fields | characters, generators, verbatim, groups, **inline math**, music |
| **Long text** (`TextLong`) | `title`, `subtitle`, `caption`, `fn` | short text **plus** `xref` and `url` |
| **Paragraph** (`TextParagraphItem`) | `p` | long text **plus** `md`, `cd`, lists, footnotes, `notation`, `idx` |

Consequences: a `<title>` or `<caption>` **may** carry an `<xref>` or `<url>` and inline
math, but **never a footnote, display math, or a list** — titles and captions get recycled
into the table of contents, the list of figures, and the index, so nothing that moves or
expands is allowed. For a title whose mathematics does not survive that trip, supply a
`<plaintitle>` alternate (it takes plain `text` only). Index headings are stricter still:
short text, so no `xref` there either.

(Verified against the validator: `<xref>` inside a chapter `<title>` raises no message.)

### `p` is the bottleneck

`TextParagraphItem` includes `MathDisplay`, `List`, `CodeDisplay`, `Footnote`, `Notation`,
`Index`. So:

- **display math (`md`, `me`, `men`) lives inside a `<p>`** — never as a sibling of one;
- **lists (`ol`, `ul`, `dl`) live inside a `<p>`** — same rule, same reason;
- therefore a **nested list needs a `<p>` inside the outer `<li>`** to contain it;
- `<cd>` (code display) is likewise *within* a paragraph, while `<pre>` is a block in its
  own right — the monospace analogue of a paragraph.

One exception worth knowing: a list may stand bare as a `sidebyside` panel, and bare at the
top level of a slideshow `slide`.

### `statement` does not take display math directly

```
BlockStatement = BlockText | Figure | Aside | SideBySide | SideBySideGroup | Sage
BlockText      = Paragraph | BlockQuote | Preformatted | Image | Video | Audio |
                 Program | Console | Tabular
```

No `MathDisplay` in that list — hence `<md>` as a direct child of `<statement>` is an
error. Wrap it in the `<p>` that the sentence needs anyway. Note also that no numbered
division block (a theorem, an example) may appear inside a statement.

### `md`: single-line and multi-row are different elements in disguise

| | single-line `md` | multi-row `md` |
|---|---|---|
| content | text | `mrow`, one per displayed line |
| `xml:id` | allowed — it is the cross-reference target | **not allowed** — the container is not a target; its `mrow`s are |
| `label` | not used | not used |
| `number` | `yes`/`no`, default `no` | default `no`, applies to each `mrow` that does not override |
| `tag` | allowed, mutually exclusive with `number` | on the `mrow` |

`<intertext>` must be *sandwiched*: it may not lead, may not trail, and two may not be
adjacent.

### Divisions: flat or structured, never both

A `chapter` (and `section`, and `article`) is either

- **flat** — blocks and `paragraphs`, then at most **one each** of `worksheet`, `handout`,
  `reading-questions`, `exercises`, `solutions`, `references`, `glossary`, in any order; or
- **structured** — an optional `introduction`, then `section`/`printout` children with
  specialised divisions interleaved (any number), then an optional `conclusion`.

Once you subdivide, loose paragraphs must move into `<introduction>`. And if you want two
`<exercises>` divisions in one chapter, that chapter has to be structured.

### `tabular` is not a child of `figure`

`tabular` never takes a number directly. Bare, it sits between paragraphs as unnumbered,
uncaptioned content (it is in `BlockText`). The `<table>` wrapper is the *only* route to a
number and a title. A `figure` may, however, hold a single paragraph — a captioned
numbered display of mathematics is honestly a figure.

### `sidebyside` panels

```
Panel = FigurePanel | Poem | Tabular | Image | Program | Console |
        Paragraph | Preformatted | List | Stack | Exercise
```

- At least two panels — a lone component has nothing to sit beside.
- Pure layout: no title, no caption, **no `xml:id`**, not a cross-reference or index target.
- **`audio`, `video` and `interactive` may never be a panel**, nor sit in a `stack`, nor
  inside a `figure` that is a panel. Static conversions render each of those *as* a
  `sidebyside` (preview image + QR code + links), and a `sidebyside` must never nest.
  So you cannot put two PhET simulations side by side; stack them as separate figures.
- `<stack>` accumulates untitled, uncaptioned items vertically within one panel.
- `<exercise>` may be a panel, to lay out a `worksheet` or `handout` compactly. The schema
  permits it elsewhere; validation-plus warns.
- `widths` on the `sidebyside` overrides any width on the panel contents.

### `xml:id` that is not a cross-reference target

Two places use `xml:id` to **name a generated file**, not to be pointed at:

- `ImageCode` (a `latex-image`, `sageplot`, `asymptote`) — it becomes the image filename.
- `video` — it becomes the preview/poster filename, which is why it is optional.

### Printouts

`worksheet` and `handout` are the two "printout" divisions; a `handout` is a `worksheet`
without the course-identification attributes. Both may hold `<page>` children or content
directly. The `workspace` attribute (on theorems, definitions, examples, projects,
exercises, tasks, proofs, and `p`) is a **hint**: honoured inside a printout, ignored
elsewhere, and validation-plus will say so. A structured division may hold any number of
printouts, interleaved with ordinary divisions — that is how a workbook is authored.

## Validation as a reading aid

```bash
pretext validate --engine salve        # Node, self-installing
pretext validate --report-form terse   # one tab-separated message per line
```

The report gives file, XPath into the assembled tree, line number in
`logs/main-assembled.xml`, an excerpt, and which of three checks fired: the development
schema (`schema` — errors), the production-schema comparison (`experimental`), and the
`validation-plus` stylesheet, which makes the checks no RELAX-NG grammar could express
(a `workspace` outside a printout, an `interactive` nested too deep in a `sidebyside`, a
`pause` outside a slideshow).

The schema lags the code by design — features are not added until reasonably stable — so
something can work and still not validate. That is a signal to check whether you are
relying on an unfinished feature, not a licence to ignore the report.

## Modular source: what may be a file's root element

`start` names the elements that can be the root of a fragment file:

`pretext`, `docinfo`, `part`, `chapter`, `section`, `subsection`, `subsubsection`,
`printout`, `slideshow`, `paragraphs`, `reading-questions`, `exercises`, `subexercises`,
`solutions`, front matter and back matter (book and article forms), `preface`,
`acknowledgement`, `appendix` (book and article forms), `index`, `references`, `glossary`,
`figure`, `webwork`.

Anything else cannot be split into its own `xi:include`d file.
