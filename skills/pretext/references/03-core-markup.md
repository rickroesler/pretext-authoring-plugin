# Core Markup

## Divisions

`book` → `part`? → `chapter` → `section` → `subsection` → `subsubsection`.
(`article` → `section` → …) You may not skip a level. Every division needs a `<title>`
and should have an `xml:id`.

A division's content may be flat, or wrapped: `<introduction>` … subdivisions …
`<conclusion>`. Once a division has subdivisions, loose paragraphs must go inside
`<introduction>`.

Specialised divisions: `exercises`, `reading-questions`, `worksheet`, `solutions`,
`references`, `glossary`, `objectives`, `outcomes`.

```xml
<chapter xml:id="ch-kinematics" xmlns:xi="http://www.w3.org/2001/XInclude">
  <title>Motion in One Dimension</title>
  <objectives>
    <ul><li><p>Relate position, velocity, and acceleration.</p></li></ul>
  </objectives>
  <introduction><p>Text before the first section.</p></introduction>
  <xi:include href="sec-velocity.ptx"/>
  <exercises xml:id="exs-ch-kinematics"><title>Chapter Exercises</title></exercises>
  <conclusion><p>Summary.</p></conclusion>
</chapter>
```

## Blocks

Numbered, cross-referenceable units inside a division. All share one serial counter by
default, so Remark 5.2.64 sits next to Example 5.2.65 and scanning works.

- **Theorem-like**: `theorem`, `lemma`, `corollary`, `proposition`, `claim`, `fact`,
  `identity`, `algorithm` — with `<statement>` and zero or more `<proof>`.
- **Definition-like**: `definition`, `axiom`, `conjecture`, `principle`, `heuristic`.
- **Example-like**: `example`, `question`, `problem` — `<statement>` + `<solution>`, or
  bare paragraphs.
- **Remark-like**: `remark`, `note`, `warning`, `insight`, `convention`, `observation`.
- **Project-like**: `project`, `activity`, `exploration`, `investigation` — may run on
  their own counter (`<projects distinct="yes"/>` in the publication file).
- **Figure-like**: `figure`, `table`, `listing`, `list` — captioned.
- Unnumbered: `paragraphs` (a titled run of prose), `assemblage`, `aside`, `blockquote`.

```xml
<theorem xml:id="thm-work-energy">
  <title>Work–Energy Theorem</title>
  <statement>
    <p>The net work equals the change in kinetic energy,<md>
      <mrow>W_{\text{net}} \amp = \Delta K = \tfrac12 m v_f^2 - \tfrac12 m v_i^2</mrow>
    </md>.</p>
  </statement>
  <proof><p>Integrate Newton's second law along the path.</p></proof>
</theorem>
```

## Mathematics

LaTeX syntax, restricted to what MathJax supports (broadly `amsmath`).

| Element | Use |
|---|---|
| `<m>` | inline. Keep it *short* — shorter than a long word, nothing tall. |
| `<me>` | one unnumbered display line |
| `<men>` | one numbered display line |
| `<md>` | display; as a container of `<mrow>` for multiple lines |
| `<mdn>` | same, all lines numbered |

```xml
<p>The Lorentz force is<md>
  <mrow>\vb{F} \amp = q\left(\vb{E} + \vb{v}\times\vb{B}\right)</mrow>
  <mrow>       \amp = q\vb{E} + q\vb{v}\times\vb{B}</mrow>
</md>.</p>
```

Rules that are not optional:

1. **Display math goes inside a `<p>`.** `<md>` directly inside `<statement>`, `<li>`, or a
   division is a schema error. This is the general rule, not a math rule: `p` is the
   bottleneck between structure and prose, and display math, lists, and `<cd>` all live
   inside it. See [10-schema.md](10-schema.md).
2. **Punctuation goes immediately after `</md>` or `</m>`, outside the element** — no
   whitespace, no comment between. PreTeXt then places it correctly for print, HTML, and
   braille.
3. `\amp` for `&`, `\lt` for `<`, `\gt` for `>`. Always. These are XML-reserved.
4. No `\\` inside `<md>` — each `<mrow>` *is* a line. `\\` is legal only inside a
   `matrix`/`pmatrix`/`bmatrix`/`cases` environment.
5. Never author `\begin{align}`, `\begin{gather}`, `\begin{equation}` — PreTeXt picks the
   environment. First `\amp` per row is an alignment point, later ones alternate
   separator/alignment (always an odd number per row). No `\amp` anywhere → `gather`.
6. `\text{ }` is the only way into text mode; put the spaces *inside* it and never put math
   inside it.
7. Macro names may not contain digits — MathJax silently stops reading the macro block.
8. Do not nest macro definitions; define each from the ground up.

Numbering: `number="yes|no"` on `<md>` or per-`<mrow>` (default `no`); a symbolic
`tag="star|dagger|hash|…"` for local references, mutually exclusive with `number`.
`break="no"` on `<md>` prevents page breaks.

Cross-reference targets: a **single-line** `<md>` may carry `xml:id` and *is* the target.
A **multi-row** `<md>` may not — it is only a container; give the `xml:id` to the `<mrow>`
you actually want to point at.

`<intertext>` inserts a sentence mid-alignment, and must be *sandwiched* between `<mrow>`s
— it may not lead, may not trail, and two may not be adjacent.

Extra LaTeX packages come in pairs so both pipelines agree:

```xml
<math-package latex-name="physics" mathjax-name="physics"/>
<math-package latex-name="cancel"  mathjax-name="cancel"/>
<math-package latex-name="xfrac"   mathjax-name=""/>       <!-- LaTeX only -->
```

Both attributes must be present. Check the
[MathJax extension list](https://docs.mathjax.org/en/latest/input/tex/extensions/index.html)
before assuming a package exists on the HTML side.

## Text, inline

What you may write depends on where the text can travel. `<title>`, `<subtitle>`,
`<caption>` and `<fn>` hold **long text**: characters, styling, groups, inline math,
`<xref>` and `<url>` — but **no footnote, no display math, no list**, because titles and
captions get recycled into the table of contents, the list of figures, and the index.
Index headings and `<shorttitle>` are stricter (**short text**: no `xref` either). Only a
`<p>` takes everything. Full table in [10-schema.md](10-schema.md).

`em`, `alert`, `term` (definition-defining), `q`/`sq` (quotes — never type `"`),
`c` (inline code; `<cd>` is a code *display* used within a paragraph, while `<pre>` is a
block in its own right), `pubtitle`, `articletitle`, `abbr`, `acro`, `init`, `foreign`,
`fn` (footnote), `nbsp`, `mdash`, `ndash`, `ellipsis`, `degree`, `dblprime`.

For a temperature in prose use `<degree/>`, not `<m>73^\circ</m>`.
Write ordinals on the baseline: 1st, 2nd — not `2^{\mathrm{nd}}`.

## Lists

```xml
<ul marker="disc"><li><p>…</p></li></ul>
<ol marker="1." cols="2"><li><p>…</p></li></ol>
<dl width="narrow"><li><title>Term</title><p>…</p></li></dl>
```

Lists live **inside a `<p>`**, like display math. So a nested list needs a `<p>` inside the
outer `<li>` to hold it. (The one exception: a bare list may be a `sidebyside` panel, or sit
at the top level of a slideshow `slide`.)

A list item that is *only* an `<m>` (no wrapping `<p>`, no adjacent text, not even a
trailing comma) gets `\displaystyle` automatically. Any stray character disables it.

## Figures, images, side-by-side

```xml
<figure xml:id="fig-incline">
  <caption>A block on a frictionless incline.</caption>
  <image source="diagrams/incline.svg" width="60%">
    <description><p>A block of mass m rests on a plane inclined at angle theta;
    arrows show the weight, the normal force, and the component along the slope.</p></description>
  </image>
</figure>
```

`<description>` is not optional in practice — it is the accessible description, and for a
physics diagram it is often the only way a screen-reader user gets the physics. Describe
what the figure *shows*, not that it is a figure.

`<sidebyside>` lays panels out horizontally; it needs at least two panels.

```xml
<sidebyside widths="45% 45%" margins="auto" valign="middle">
  <figure><caption>Free-body diagram.</caption><image source="fbd.svg"/></figure>
  <figure><caption>Motion diagram.</caption><image source="motion.svg"/></figure>
</sidebyside>
```

Inside a `<figure>`, captioned panels become sub-captioned: Figure 5.2.b.
`<sbsgroup>` stacks several `<sidebyside>` with shared layout — a grid.
`<stack>` accumulates untitled items vertically inside one panel.

`<sidebyside>` is pure layout: no title, no caption, **no `xml:id`**, not a cross-reference
or index target. And **`audio`, `video` and `interactive` may never be a panel** (nor sit
in a `stack`, nor in a `figure` that is a panel) — static conversions render each of them
*as* a `sidebyside`, and side-by-sides must not nest.
To size or centre a *single* image use `width` on the `<image>` itself, not a `sidebyside`.

## Tables

```xml
<table xml:id="tab-constants">
  <title>Physical constants</title>
  <tabular halign="left" valign="top" top="major">
    <col/><col halign="right"/>
    <row header="yes" bottom="minor"><cell>Symbol</cell><cell>Value</cell></row>
    <row><cell><m>c</m></cell><cell>…</cell></row>
  </tabular>
</table>
```

Rules: `<table>` is the captioned wrapper and the **only** route to a number and a title;
`<tabular>` is the grid and never takes a number itself. A bare `<tabular>` may sit between
paragraphs as unnumbered, uncaptioned content, but it is **not** a child of `<figure>`.
(A `<figure>` may hold a single `<p>` — a captioned, numbered display of mathematics is
honestly a figure.) Line weights are `none|minor|medium|major` on
`top`/`bottom`/`left`/`right`, settable on `tabular`, `col`, `row`, or `cell`.

## Cross-references

```xml
<xref ref="thm-work-energy"/>
<xref ref="thm-work-energy" text="title"/>
<xref ref="eq-lorentz" detail="a"/>
<xref first="ex-1" last="ex-5"/>
<xref provisional="the chapter on thermodynamics"/>   <!-- placeholder, no target yet -->
```

`xml:id` is the internal handle; `label` is the *reader-facing* stable identifier that
names HTML pages, generated image files, and Runestone database rows. Set `label` values
deliberately and never change them once published.

`<cross-references text="…"/>` in `docinfo` sets the default phrasing
(`global|local|hybrid|type-global|type-local|type-hybrid|title|phrase-global|…`).

## Index and notation

```xml
<idx>angular momentum</idx>
<idx><h>momentum</h><h>angular</h></idx>
<idx><h>momentum</h><see>impulse</see></idx>

<notation>
  <usage><m>\vb{L}</m></usage>
  <description>angular momentum</description>
</notation>
```

Then in `backmatter`: `<index><title>Index</title><index-list/></index>` and
`<appendix><title>Notation</title><notation-list/></appendix>`. A physics book earns a lot
from `notation-list` — it becomes the symbol table readers keep flipping back to.

## Author tools

- `<xref provisional="…"/>` for a forward reference to unwritten material.
- XML comments whose first four non-blank characters are `todo`:
  `<!-- ToDo: add the damped case -->`.
- Build with `<stringparams author.tools="yes"/>` to surface both in the output.
