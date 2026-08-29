# Physics Markup

Everything here is verified against PreTeXt core 2.51.0 by building HTML and LaTeX output.

## 1. Physical quantities and units

Author a physical quantity with `<quantity>`, never as prose or as math. PreTeXt then
produces correct spacing, upright unit symbols, and a real `siunitx` `\SI{}{}` in LaTeX,
and it survives the trip to braille.

```xml
<quantity><mag>9.807</mag><unit base="meter"/><per base="second" exp="2"/></quantity>
```

- HTML: `9.807 <sup>m</sup>⁄<sub>s<sup>2</sup></sub>`
- LaTeX: `\SI{9.807}{\meter\per\second\tothe{2}}`

Structure: at most one `<mag>`, then any number of `<unit>` (numerator) and `<per>`
(denominator). Each takes `base` (required), `prefix`, and an integer `exp`.

```xml
<quantity><mag>1.5</mag><unit base="tesla"/></quantity>
<quantity><mag>13.6</mag><unit base="electronvolt"/></quantity>
<quantity><mag>6.674\times 10^{-11}</mag>
          <unit base="newton"/><unit base="meter" exp="2"/><per base="gram" prefix="kilo" exp="2"/></quantity>
```

Every part is optional:
- `<mag>` alone — a bare number kept typographically consistent with its neighbours.
- units alone — the natural form inside a table that explains what a unit *is*, or after a
  numeric result you computed in `<m>`.

The full prefix and base vocabulary is in [09-units-vocabulary.md](09-units-vocabulary.md).
An unrecognised name is reported at build time, so trying it is cheap.

### The scientific-notation trap

The Guide says `<mag>` accepts LaTeX. **In HTML output only `\pi` is handled.** Anything
else — `\times`, `^{-31}` — is emitted literally, so
`<mag>9.109\times 10^{-31}</mag>` renders as the characters `9.109\times 10^{-31}` on the
web while rendering correctly in the PDF. (The HTML stylesheet special-cases `\pi` and
nothing else; siunitx v3 is configured with `parse-numbers=false`, which is why LaTeX
copes.)

For a physics book this matters on almost every page. The verified workaround: put the
number in `<m>` and leave the units in a magnitude-less `<quantity>`:

```xml
<m>9.109\times 10^{-31}</m><nbsp/><quantity><unit base="gram" prefix="kilo"/></quantity>
```

which yields a real MathJax span plus `kg` in HTML, and `\(9.109\times 10^{-31}\)~\si{\kilo\gram}`
in LaTeX. Adopt this as the house style for any magnitude that is not a plain decimal, and
be consistent — mixed styles show up as inconsistent spacing.

### Constants table

```xml
<table xml:id="tab-constants">
  <title>Physical constants used in this text</title>
  <tabular halign="left" top="major">
    <col/><col/><col halign="left"/>
    <row header="yes" bottom="minor">
      <cell>Quantity</cell><cell>Symbol</cell><cell>Value</cell>
    </row>
    <row>
      <cell>speed of light</cell><cell><m>c</m></cell>
      <cell><m>2.998\times 10^{8}</m><nbsp/><quantity><unit base="meter"/><per base="second"/></quantity></cell>
    </row>
    <row>
      <cell>Planck constant</cell><cell><m>h</m></cell>
      <cell><m>6.626\times 10^{-34}</m><nbsp/><quantity><unit base="joule"/><unit base="second"/></quantity></cell>
    </row>
  </tabular>
</table>
```

Give it an `xml:id` and `<xref>` to it; better still, put it in an appendix and let the
index point at it.

## 2. The `physics` LaTeX package

Verified working in both pipelines. Put in `docinfo`:

```xml
<math-package latex-name="physics" mathjax-name="physics"/>
```

- HTML: emits `\require{physics}` into the macro block; MathJax 4 autoloads the extension.
- LaTeX: emits `\usepackage{physics}` into the preamble.

You then get, in `<m>`/`<md>`, the notation a physics text lives on:

| Macro | Renders |
|---|---|
| `\vb{v}`, `\vu{r}`, `\va{a}` | bold vector, unit vector, arrow vector |
| `\dv{x}{t}`, `\dv[2]{x}{t}` | ordinary derivative |
| `\pdv{f}{x}`, `\pdv{f}{x}{y}` | partial derivatives |
| `\dd{x}`, `\dd[3]{r}` | upright differentials |
| `\abs{x}`, `\norm{v}`, `\eval{…}` | sized delimiters |
| `\div`, `\curl`, `\grad`, `\laplacian` | vector calculus operators |
| `\ev{A}`, `\bra{ψ}`, `\ket{φ}`, `\braket{ψ}{φ}`, `\mel{a}{H}{b}` | Dirac notation |
| `\Re`, `\Im`, `\qty()`, `\order{x^2}` | misc |

**Caveat:** `physics` is famously aggressive about redefining `\div`, `\Re`, `\Im`, and
`\vb` clashes with `bm`. It also does not always agree macro-for-macro with the MathJax
extension. Build both HTML and PDF for one chapter before committing to it.

The conservative alternative is your own semantic macros:

```xml
<macros>
  \newcommand{\vect}[1]{\mathbf{#1}}
  \newcommand{\uvec}[1]{\hat{\mathbf{#1}}}
  \newcommand{\deriv}[2]{\frac{d #1}{d #2}}
  \newcommand{\pderiv}[2]{\frac{\partial #1}{\partial #2}}
  \DeclareMathOperator{\curlop}{curl}
  \newcommand{\curl}[1]{\curlop\left(#1\right)}
</macros>
```

This is what PreTeXt actually recommends. Semantic names (`\vect`, not `\v`) let you change
vector notation across the whole book by editing one line. Two constraints from the core:
macro names may contain **no digits** (MathJax stops reading the whole block, silently),
and macros must **not call each other** — one conversion isolates definitions and a
nested call becomes undefined.

## 3. Diagrams

Four source-described image languages, all children of `<image>`, all rendered to SVG for
HTML and PDF for print by `pretext generate`.

| Element | Language | Needs | Best for |
|---|---|---|---|
| `latex-image` | TikZ / PGFPlots / circuitikz | TeX | free-body diagrams, ray diagrams, circuits, field lines, data plots |
| `prefigure` | PreFigure (XML) | `pip install pretext[prefigure]` | diagrams that must be *accessibly* described and navigable |
| `asymptote` | Asymptote | remote server by default | 3D surfaces, projections |
| `sageplot` | SageMath | Sage ≥ 10 | anything you can compute |
| `mermaid` | Mermaid | `mmdc` | flow/state diagrams |

```xml
<figure xml:id="fig-fbd">
  <caption>Free-body diagram for a block on an incline.</caption>
  <image xml:id="img-fbd" width="55%">
    <description><p>Three arrows act on a block on a slope: the weight straight down,
    the normal force perpendicular to the surface, and friction up the slope.</p></description>
    <latex-image>
      \begin{tikzpicture}[>=Stealth, scale=1.2]
        \draw[thick] (0,0) -- (5,0) -- (5,2.5) -- cycle;
        \node[draw, fill=gray!25, minimum size=8mm, rotate=26.6] (b) at (3.2,1.35) {};
        \draw[->, thick] (b) -- ++(0,-1.4) node[below] {$m\vb{g}$};
        \draw[->, thick] (b) -- ++(-0.63,1.25) node[above] {$\vb{N}$};
        \draw[->, thick] (b) -- ++(-1.1,-0.55) node[left] {$\vb{f}$};
        \draw (0.9,0) arc (0:26.6:0.9) node[midway, right] {$\theta$};
      \end{tikzpicture}
    </latex-image>
  </image>
</figure>
```

The TikZ libraries you use must be loaded in `docinfo/latex-image-preamble`:

```xml
<latex-image-preamble>
  \usepackage{tikz, pgfplots}
  \usepackage[siunitx, RPvoltages]{circuitikz}
  \usetikzlibrary{arrows.meta, decorations.markings, calc, angles, quotes, patterns}
  \pgfplotsset{compat=1.18}
</latex-image-preamble>
```

Your `docinfo/macros` are available inside `latex-image` too, so `\vb{v}` in the prose and
`\vb{v}` in the diagram label stay in sync. (Reported exception: inside **Asymptote**,
`\newcommand{\foo}{\operatorname{foo}}` behaves better than `\DeclareMathOperator`.)

Data files for a PGFPlots `\addplot table {…}`: keep the CSV in your external directory,
but reference it in the TikZ code as `external/data/whatever.csv`. PreTeXt copies the
external directory to a temporary directory literally named `external` before compiling,
regardless of what you named yours.

`pretext generate latex-image` regenerates only those; `-x img-fbd` regenerates one.
`--all-formats` produces every format (needed if EPUB and PDF want different ones).

### Photographs and vector art

External files, `<image source="…"/>`, path relative to the external directory.
SVG for HTML and EPUB, PDF for LaTeX, PNG for anything raster (300 DPI if it must survive
Kindle). Inkscape is the recommended editor; save "Plain SVG" for the build and keep an
"Inkscape SVG" alongside for future editing.

## 4. Accessible descriptions

For a physics text this is a real editorial task, not a checkbox. Every `<image>`,
`<video>`, and `<interactive>` takes a `<description>` holding one or more `<p>`.

Write what the reader needs to do the physics: which quantities are shown, their
directions, what varies along each axis, what the trend is. "A graph of velocity against
time" is useless; "Velocity rises linearly from zero to 20 m/s over the first 4 seconds,
then stays constant" is the figure.

`prefigure` goes further — it produces SVG with a navigable, described structure — and is
worth the setup cost for the handful of diagrams that carry the most conceptual load.

## 5. Numbering for a physics text

In the publication file:

```xml
<numbering>
  <divisions level="2"/>
  <equations level="2"/>      <!-- Equation 5.2.7 -->
  <figures distinct="yes" level="2"/>
</numbering>
```

Physics books cite equations constantly, so consider `<numbering equations="yes"/>` in
`docinfo` to number every display, and `<xref ref="eq-…"/>` throughout. Splitting figures
onto their own counter (`distinct="yes"`) is defensible in a heavily illustrated text;
PreTeXt's default keeps them in the shared block sequence, and its argument — that a
shared counter makes scanning work — is worth reading in the Guide before overriding.

## 6. Worksheets and labs

`<worksheet>` is a print-oriented division with margin control, page breaks
(`<pagebreak/>`), and `workspace="1in"` on exercises to leave room for working. It is the
natural container for a lab handout or problem session.

```xml
<worksheet xml:id="ws-projectile">
  <title>Projectile Motion Lab</title>
  <exercise workspace="2in">
    <statement><p>Measure the range for five launch angles and tabulate.</p></statement>
  </exercise>
</worksheet>
```

Configure margins and headers under `<common><worksheet margin="0.75in">` in the
publication file, including `first-page-header`/`running-footer` with `left`/`center`/`right`.

Rename `activity` to `Laboratory` with `<rename element="activity">Laboratory</rename>`
in `docinfo` if that is your book's vocabulary.
