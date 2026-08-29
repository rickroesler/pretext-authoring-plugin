# Gotchas

Each of these was hit and confirmed while building a real physics chapter with CLI 2.51.0,
or is called out explicitly in the PreTeXt source. Ordered by how much time they cost.

## 0. `p` is the bottleneck — display math *and* lists live inside it

The schema pattern `TextParagraphItem` (what a `<p>` may contain) is the only place that
admits `md`, `me`, `men`, `cd`, `ol`, `ul`, `dl`, `fn`, `notation`, and `idx`. Everything
below in items 1 and 19 is a consequence of that one fact. A nested list therefore needs a
`<p>` inside the outer `<li>`. (Two narrow exceptions: a bare list may be a `sidebyside`
panel or a top-level item in a slideshow `slide`.)

## 1. Display math must live inside a `<p>`

```xml
<!-- INVALID: "<md> is not allowed here." -->
<statement>
  <p>For constant acceleration,</p>
  <md><mrow>v \amp = v_0 + at</mrow></md>
</statement>

<!-- VALID -->
<statement>
  <p>For constant acceleration,<md>
    <mrow>v \amp = v_0 + at</mrow>
  </md>.</p>
</statement>
```

Display math is part of the sentence, not a sibling of it. Note the period **immediately**
after `</md>` — no whitespace, no comment. That is how PreTeXt gets the punctuation right
in print, HTML, and braille alike.

## 2. LaTeX inside `<mag>` is not rendered in HTML

Only `\pi` is special-cased. `<mag>9.109\times 10^{-31}</mag>` renders the literal
characters `9.109\times 10^{-31}` on the web, while producing correct output in the PDF.
Use `<m>…</m><nbsp/><quantity><unit …/></quantity>` for any magnitude in scientific
notation. Full explanation in [04-physics-markup.md](04-physics-markup.md).

## 3. `pretext` invoked by path, without the venv on `PATH`

```
fatal: [Errno 2] No such file or directory: 'playwright'
```

The CLI shells out to helper executables by bare name. Running `.venv/bin/pretext` leaves
`.venv/bin` off `PATH`, so `playwright`, `node`, `xelatex` and friends are not found. The
EPUB build prints "Finished build for target ebook" and then a block of fatal errors, so
it reads as a partial success. Activate the venv (or export the path) before building.

## 4. Interactive markup is *experimental*

`<interactive>`, `<choices>`, `<feedback>`, `statement/@correct`, `fillin/@answer`,
`<evaluation>`, `<setup>` are all in the development schema only. `pretext validate`
inventories them under "experimental constructs in use" — that section is **not** a list
of errors, it is a list of things whose markup may change without a deprecation cycle.
Read the inventory each time you validate; it is how you learn what may break under you.

## 5. Every publication-file path is relative to `main.ptx`

Not to the publication file, not to the working directory. `generated="../generated-assets"`
means "up one from `source/`". A path that looks right and is silently ignored is almost
always this.

## 6. `<image source="…">` omits the external directory name

With `<directories external="../assets"/>` and a file at `assets/photos/lab.jpg`, write
`source="photos/lab.jpg"`. Including `assets/` breaks it. And a *declared* external
directory that does not exist halts the build outright.

## 7. Macro names may not contain digits

`\newcommand{\v1}{…}` makes MathJax stop reading the macro block **silently**, taking every
macro defined after it — including the ones PreTeXt appends for `\amp`, `\lt`, `\gt`. The
symptom is math that fails all over the book for no visible reason.

## 8. Macros must not call each other

```
\newcommand{\vect}[1]{\mathbf{#1}}
\newcommand{\bvect}[1]{\bar{\vect{#1}}}      ← breaks in at least one conversion
```

One conversion isolates macro definitions and pulls in only those it sees used nearby, so a
nested call can be undefined at the point of use. Define each from the ground up.
(The `\DeclareMathOperator` advice in the Guide conflicts with this; the conflict is
acknowledged there.)

## 9. `\\` is banned in `<md>`; PreTeXt chooses the environment

Each `<mrow>` is a line. `\\` is legal only inside `matrix`/`pmatrix`/`bmatrix`/`cases`,
which build structure *within* one formula. Never author `\begin{align}`,
`\begin{equation}`, or `\begin{gather}` yourself.

Also: `\matrix{}` and `\pmatrix{}` are recognised by MathJax but **not** by LaTeX. Use the
`\begin{matrix}` environments so both pipelines agree.

## 10. Ampersand counting in aligned math

Per row: *n* alignment points and *n−1* column separators — always an odd number of
`\amp`. If macros hide your ampersands, force the environment with
`alignment="gather|align|alignat"` and, for `alignat`, set `alignat-columns`.

## 11. EPUB and Kindle both produce `main.epub`

Different math encodings, identical filename. Give the targets different `output-dir`s.
Uploading the SVG-math EPUB to Amazon produces a book whose mathematics does not render —
and if you use an aggregator, untick Amazon for exactly this reason.

## 12. An EPUB cover has a fixed size

2048 pixels tall by 1280 wide, JPEG or PNG, path relative to the external directory. Kindle
additionally wants raster images at ≥ 200 DPI (300 preferred).

## 13. QR codes for print need `baseurl`

Static images of interactives are printed with a QR code pointing at the live version — but
only if `<html><baseurl href="https://…/"/></html>` is set (trailing slash included).
Set it before you generate a print edition, not after.

## 14. `pretext generate webwork` is not automatic enough

WeBWorK content will not appear in *any* output until static representations have been
fetched. When problems are missing, run `pretext generate webwork` first.

## 15. Not every OPL problem works

Older PG macros do not emit PreTeXt output. Best case you get "did not return valid XML";
worst case you get valid-looking output that is silently incomplete. Read the PDF with
hints, answers, and solutions exposed to check what actually came back.

## 16. Runestone `label`s are permanent

The database key is `document-id` + `edition` + `label`. Change a `label` after publishing
and student records are orphaned. Decide all three before the first upload; use a new
`edition` when you want a clean slate.

## 17. Only one `project.ptx`, and it is not PreTeXt

The manifest does not conform to the PreTeXt schema despite the extension, and the CLI
expects exactly one. `ptx-version="2"` is required. Executables moved out to
`executables.ptx` in CLI 2.0 — old documentation showing them inside `project.ptx` is
stale, as is documentation showing target settings as child *elements* rather than
attributes (the EPUB chapter of the Guide still does this).

## 18. GeoGebra resizing does not rescale axes

PreTeXt resizes the applet viewport anchored at the top-left. Content can end up outside
the visible frame. Call `setCoordSystem(...)` in the `<slate>` to restore it.

## 19. `\displaystyle` in list items is easy to lose

A list item containing *only* an `<m>` gets `\displaystyle` automatically. Any other
character — including a trailing comma — turns it off. (Chicago style says the comma
should not be there anyway.)

## 20. External data files for PGFPlots use a fixed path

Your external directory is copied to a temporary directory literally named `external`
before LaTeX runs, whatever you called it. So write
`\addplot table {external/data/deer-weights.csv};` — PreTeXt does not scan your TikZ code
to rewrite paths.

## 21. Deprecated: `directory.images` and `external` in the publication file

The old `images/` convention and the `directory.images` string parameter are ignored and
warn. `external` belongs in `docinfo`, `generated` in the publication file.

## 22. No footnote, display math, or list in a title or caption

`title`, `subtitle`, `caption` and `fn` take **long text** — inline math, `xref` and `url`
are fine; a footnote, an `<md>`, or a list is not. The restriction is deliberate: these
strings are recycled into the table of contents, the list of figures, and the index, so
nothing that moves or expands may ride along. Index headings and `<shorttitle>` are one
tier stricter (**short text**: no `xref`). Use `<plaintitle>` for a title whose mathematics
does not survive the trip.

## 23. A multi-row `<md>` cannot carry `xml:id`

Only a *single-line* `<md>` is a cross-reference target. A multi-row one is just a
container — put the `xml:id` on the `<mrow>` you mean. The error message misdirects: adding
`xml:id` commits the grammar to the single-line branch, so what you get is
`<mrow> is not allowed here`, once per row. Neither form uses `label`. And
`<intertext>` must be sandwiched between `<mrow>`s: never first, never last, never two
adjacent.

## 24. `interactive`, `video` and `audio` cannot be `sidebyside` panels

Static conversions render each of them *as* a `sidebyside` (preview + QR + links), and
side-by-sides must not nest — so a `figure` in a panel may not contain one either. Two PhET
simulations cannot sit next to each other; stack them as consecutive figures.

## 25. `tabular` is not a child of `figure`

`tabular` never takes a number itself. Bare, it sits between paragraphs, unnumbered and
uncaptioned; `<table>` is the only wrapper that gives it a number and a title. Conversely a
`<figure>` *may* hold a single `<p>`.

## 26. A flat division allows only one of each specialised division

A chapter is either flat (blocks and `paragraphs`, then **at most one each** of `worksheet`,
`handout`, `reading-questions`, `exercises`, `solutions`, `references`, `glossary`) or
structured (`section`/`printout` children, with any number of specialised divisions
interleaved). Two `<exercises>` divisions in one chapter means that chapter must be
structured.

## 27. `xml:id` sometimes names a file rather than a target

On a `video`, and on a `latex-image`/`sageplot`/`asymptote`, `xml:id` generates the preview
or image *filename*. It is not a cross-reference target there, which is why it is optional.

## 28. Two stale instructions in the Guide's own text

Both are rejected by the 2.51.0 schema:

- `<cross-references text="…"/>` in `docinfo` — the element is
  `<defaults><xrefs text="…"/></defaults>`.
- `<numbering equations="yes"/>` in `docinfo`, to number every display globally — not
  permitted. Number per-element with `number="yes"` on `<md>` or `<mrow>`; the publication
  file's `<numbering><equations level="2"/></numbering>` only sets how deep the *number*
  runs, not whether displays are numbered.

More generally: when the Guide and the build disagree, the schema wins. Read
`~/.ptx/<version>/core/schema/pretext.rnc` — the `docinfo` grammar starts at the
`element docinfo {` line and its allowed children are the `Configuration |=` alternatives
that follow.

## Debugging order

1. `pretext validate --engine salve`. Read the file/XPath/line it gives you.
2. Read `logs/` — timestamped, debug-level. Or rebuild with `pretext -v debug build web`.
3. A build warning names the *ancestor chain* (`book/chapter/section/theorem/statement/p/xref`)
   and the nearest `<title>` and `xml:id`, but never the file — because the project is
   processed as one assembled tree. Grep your source for the title or id.
4. Bisect by commenting out `xi:include`s.
5. `pretext support`, then the `pretext-support` Google group.
