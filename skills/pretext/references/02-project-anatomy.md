# Project Anatomy

## Scaffold

```bash
pretext new book -d physics-book      # also: article, course, demo, hello, slideshow
```

```
physics-book/
├── project.ptx              # the manifest: what targets exist
├── requirements.txt         # pins the CLI version this project builds with
├── assets/                  # YOUR external files: photos, .ggb, .csv, .js
├── generated-assets/        # machine-owned: TikZ→SVG, previews, QR codes. Never edit.
├── publication/
│   └── publication.ptx      # presentation decisions
├── source/
│   ├── main.ptx             # the one file with <pretext>; xi:includes everything
│   ├── docinfo.ptx          # macros, packages, external dir — facts the content needs
│   ├── frontmatter.ptx
│   ├── ch-*.ptx             # one file per chapter
│   └── backmatter.ptx
└── output/                  # build products; .cache/ holds generated-asset cache
```

`pretext new demo` produces a much richer project worth reading once.
`pretext init` retrofits `project.ptx` onto an existing tree; `pretext init --refresh`
regenerates the managed files with `.bak` copies of anything you edited.
After upgrading the CLI, run `pretext update-project`.

## The three configuration files

### `project.ptx` — targets

```xml
<?xml version="1.0" encoding="utf-8"?>
<project ptx-version="2">
  <targets>
    <target name="web"    format="html"  deploy-dir="."/>
    <target name="print"  format="pdf"   deploy-dir="pdf" latex-engine="xelatex"
            publication="publication-print.ptx"/>
    <target name="tex"    format="latex"/>
    <target name="ebook"  format="epub"/>
    <target name="kindle" format="kindle"/>
  </targets>
</project>
```

`ptx-version="2"` is required. `name` is what you type (`pretext build print`);
`format` is one of `html pdf latex epub kindle braille webwork custom`.
Omit the target name to build the first one.

Defaults, all overridable: source `source/main.ptx`, publication
`publication/publication.ptx`, output `output/<name>/`, stage `output/stage`.
Paths on a `<target>` are **relative to the corresponding path on `<project>`**, so
`<project source="src">` plus `<target source="main-print.ptx">` means `src/main-print.ptx`.

Useful `<target>` attributes: `output-filename` (single-file formats),
`deploy-dir` (required on every target you want `pretext deploy` to publish),
`latex-source="yes"` (keep the `.tex` alongside the PDF), `platform="runestone"`,
`compression="zip|scorm"`, `standalone="yes"`, `xsl` (required with `format="custom"`).

A `<stringparams>` child passes XSL string parameters:

```xml
<target name="draft" format="pdf">
  <stringparams author.tools="yes"/>
</target>
```

Non-standard tool paths go in a separate `executables.ptx` at the project root
(`latex pdflatex xelatex asy mermaid sage pdfeps node perl fop`), **not** in `project.ptx`
— that changed in CLI 2.0.

### `source/docinfo.ptx` — what the content needs

Everything here changes the *words or the meaning*. Remove an item and the book is a
different book, or does not build.

```xml
<docinfo>
  <document-id edition="1">physicsbook</document-id>   <!-- stable forever; Runestone key -->
  <blurb shelf="Physics">A calculus-based introductory mechanics text.</blurb>
  <initialism>IPM</initialism>

  <directories external="../assets"/>                   <!-- relative to main.ptx -->

  <math-package latex-name="physics" mathjax-name="physics"/>
  <macros>
    \newcommand{\vect}[1]{\mathbf{#1}}
    \DeclareMathOperator{\curl}{curl}
  </macros>

  <latex-image-preamble>
    \usepackage{tikz, pgfplots}
    \usetikzlibrary{arrows.meta, decorations.markings, calc}
  </latex-image-preamble>

  <rename element="activity" xml:lang="en-US">Laboratory</rename>
  <cross-references text="type-global"/>
  <numbering equations="yes"/>
</docinfo>
```

Settle `docinfo` early. Changing `<cross-references text=...>` late means re-reading every
sentence you wrote around an `<xref>`.

### `publication/publication.ptx` — presentation

Nothing here changes your words. Change it freely, at any time, per target.
The file `pretext new` writes is itself an annotated reference to every option — read it
before searching the Guide.

```xml
<publication>
  <common>
    <chunking level="2"/>                    <!-- 0=one page, 1=chapter, 2=section … -->
    <tableofcontents level="2"/>
    <exercise-divisional statement="yes" hint="yes" answer="no" solution="no"/>
    <watermark scale="1.2">DRAFT</watermark>
  </common>

  <source>
    <directories generated="../generated-assets"/>
    <!-- optional: customizations="…" private-solutions="…" -->
    <!-- <version include="labs videos"/> -->
  </source>

  <numbering>
    <divisions level="2" chapter-start="1"/>
    <equations level="2"/>
    <figures distinct="no" level="2"/>
  </numbering>

  <latex print="no" sides="one" font-size="10" latex-style="texstyle">
    <page crop-marks="none"><geometry></geometry></page>
    <cover front="cover-front.pdf" back="cover-back.pdf"/>
  </latex>

  <html>
    <css theme="default-modern" palette="blue-red" provide-dark-mode="yes"/>
    <search variant="textbook"/>
    <tableofcontents focused="yes" preexpanded-levels="1"/>
    <baseurl href="https://example.org/physics/"/>
    <knowl theorem="no" proof="yes" example="yes" exercise-inline="yes"/>
    <interactives resize-behavior="fixed-height">
      <geogebra resize-behavior="responsive"/>
    </interactives>
    <feedback href="mailto:you@example.edu"/>
  </html>

  <epub><cover front="cover.jpg"/></epub>
</publication>
```

**Every path in a publication file is relative to `source/main.ptx`** — not to the
publication file, and not to the working directory. This is the single most common
cause of "the file cannot be found".

The natural pattern for a physics book is several publication files sharing one source:
`publication.ptx` (web, solutions hidden inline, knowled), `publication-print.ptx`
(`latex print="yes" sides="two"`, solutions collected at the back),
`publication-instructor.ptx` (everything visible, watermarked).

## External vs generated files

| | External | Generated |
|---|---|---|
| Owned by | you | the build |
| Examples | photographs, `.ggb`, `.csv`, custom JS | TikZ→SVG, YouTube thumbnails, interactive previews, QR codes |
| Declared in | `docinfo/directories/@external` | `publication/source/directories/@generated` |
| Default | none (declare nothing if you have none) | `generated/` next to `main.ptx` |

Referencing an external file **omits the directory name**: with
`<directories external="../assets"/>` and a file at `assets/photos/pendulum.jpg`, write
`<image source="photos/pendulum.jpg"/>`. This is what lets you rename or move the assets
directory without touching your prose.

The generated directory's subdirectory names are **fixed**: `asymptote`, `dynamic_subs`,
`latex-image`, `mermaid`, `prefigure`, `preview`, `problems`, `qrcodes`, `sageplot`,
`trace`, `youtube`.

A declared external directory that does not exist halts the build.

## Modular source

```xml
<!-- source/main.ptx -->
<pretext xml:lang="en-US" xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include href="./docinfo.ptx"/>
  <book xml:id="intro-physics">
    <title>Introductory Physics</title>
    <xi:include href="./frontmatter.ptx"/>
    <xi:include href="./ch-kinematics.ptx"/>
    <xi:include href="./backmatter.ptx"/>
  </book>
</pretext>
```

- The `xmlns:xi` declaration is needed on the **outermost element of every file that uses
  `xi:include`**, not just `main.ptx`.
- One outermost element per file. Two `<chapter>`s cannot share a file.
- `href` is relative to the including file.
- Validation and building always point at `main.ptx`.
- Start with a single file. Modularise once you are past beginner errors — an
  `xi:include` you can comment out is also the fastest way to bisect a bad build.

## Version control

Commit `source/`, `publication/`, `assets/`, `project.ptx`, `requirements.txt`.
Ignore `output/`, `generated-assets/`, `.cache/`, `logs/`. `pretext new` writes a
`.gitignore` that already does this.
