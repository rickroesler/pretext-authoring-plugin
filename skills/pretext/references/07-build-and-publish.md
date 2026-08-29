# Building and Publishing

## The commands

```bash
pretext build [target]        # omit target → the first in project.ptx
pretext build web -g          # force regeneration of all assets
pretext build web -q          # skip generation entirely (fast prose iteration)
pretext build print --latex   # keep the .tex next to the PDF
pretext build web --clean     # wipe the output dir first
pretext build web -x sec-waves  # build only this subtree
pretext build --deploys       # every target with a deploy-dir

pretext view web              # local server, http://localhost:8128
pretext view web -b           # build first
pretext generate latex-image  # regenerate one asset class
pretext generate -t web --all-assets
pretext validate --engine salve
pretext deploy
pretext -v debug build web    # full logging; also see logs/
pretext support               # paste this into a support request
```

Since CLI 1.7 assets regenerate automatically when their source changes, so `pretext
generate` is mostly for forcing or narrowing. WeBWorK is the exception worth remembering:
run `pretext generate webwork` after adding problems.

Single-file mode, no project needed (CLI ≥ 2.19):
`pretext build pdf -i ~/Desktop/paper.ptx`, `pretext build html -i …` (produces a portable
single HTML file).

## Validate first, always

```bash
pretext validate --engine salve      # Node, self-installing, no Java
pretext validate                     # needs jing on PATH (Java)
pretext validate --method server     # remote jing, nothing to install
pretext validate --report-form terse # one tab-separated message per line
```

The report names the source **file**, an XPath into the assembled tree, a line number in
`logs/main-assembled.xml`, and an excerpt. It runs three checks: the development schema
(errors), the production schema (an inventory of *experimental* constructs you are relying
on — not errors), and a "validation-plus" stylesheet for things a schema cannot express.

A build can succeed on invalid source and a valid document can still produce warnings; the
two checks are independent. But when output looks wrong, validate before anything else.

If a message is too vague, bisect: comment out an `xi:include` for a chapter, rebuild, and
see whether the problem leaves with it.

## Target recipes

### Web HTML

```xml
<target name="web" format="html" deploy-dir="."/>
```

Set in the publication file: `chunking level` (1 = a page per chapter, 2 = per section),
the theme (`default-modern` with a `palette`, or `tacoma`/`greeley`/`boulder`/`salem`/`denver`,
or `custom` with your own SCSS `entry-point`), `search variant="textbook"`,
`tableofcontents focused="yes"`, and `<knowl …/>` to choose which block types are born
folded. `<resources host="cdn"/>` loads CSS/JS remotely at the cost of custom themes.
`<platform portable="yes"/>` collapses the whole book into one HTML file.

Test through a server, not `file://` — knowls, Sage cells and videos are blocked by
browser security otherwise. `pretext view` does this for you.

### Print PDF

```xml
<target name="print" format="pdf" latex-engine="xelatex"
        publication="publication-print.ptx" deploy-dir="pdf"/>
<target name="tex" format="latex"/>   <!-- inspect the preamble without compiling -->
```

```xml
<latex print="yes" sides="two" font-size="10" open-odd="add-blanks" pageref="yes">
  <page crop-marks="letter"><geometry>margin=1in</geometry></page>
  <cover front="cover-front.pdf" back="cover-back.pdf"/>
</latex>
```

`print="no"` gives a screen PDF (hyperlinks, one-sided, no page references);
`print="yes"` flips the defaults toward a physical book. `latex-style` selects a bundled
style (`AIM`, `chaos`, `CLP`, `dyslexic-font`, `guide`, `texstyle`).
`<insertions pagebreaks="id1 id2"/>` forces breaks. `draft="yes"` enables LaTeX draft mode.

### EPUB and Kindle

These are **two different builds** because the math is encoded differently, and this is
the single most important fact about ebook output:

| Target | Math | For |
|---|---|---|
| `format="epub"` | SVG (rendered offline by MathJax) | Apple Books, Kobo, Calibre, everything except Kindle |
| `format="kindle"` | MathML | Amazon KDP only |

(Verified: the two builds of the same source contain `<svg>` and `<math>` respectively.)
Apple's MathML support is poor, Kindle's SVG support is poor — hence the split.

```xml
<target name="ebook"  format="epub"   output-dir="epub"/>
<target name="kindle" format="kindle" output-dir="kindle"/>
```

Both produce a file called `main.epub`, indistinguishable by name — **give them different
`output-dir`s** or you will upload the wrong one.

Requirements: Node + npm, Playwright with a browser installed, and the venv on `PATH`.
The publication file must set `source/directories/@generated` and
`docinfo/directories/@external` so images can be bundled.

Cover: `<epub><cover front="cover.jpg"/></epub>`, path relative to the external directory,
JPEG or PNG, **2048 × 1280** (tall × wide). Without one, a plain generated cover is used.

Kindle also wants raster images at ≥ 200 DPI, 300 preferred.

Supported document types: `book` (with or without `part`) and `article` only. A
`slideshow`, `letter` or `memo` halts the conversion.
`chunking` and `tableofcontents` levels are ignored (and say so) — reading systems handle
both themselves.

Validate before uploading:

```bash
epubcheck output/epub/main.epub
```

Most marketplaces require a clean `epubcheck`. GUI and online versions exist; see
<https://www.w3.org/publishing/epubcheck/>.

Preview: Kindle Previewer (macOS/Windows) for the Kindle file, Apple Books or Calibre for
the EPUB. Some books crash Kindle Previewer; the KDP web uploader's preview is the
fallback, and the only option on Linux.

**Distribution rule:** the Kindle build must go to Amazon KDP *directly*. If you use an
ebook aggregator for the standard EPUB, untick Amazon — the aggregator would push the
SVG-math EPUB to Kindle and the mathematics will not render.

### Runestone

```xml
<target name="rs" format="html" platform="runestone"/>
```

Requires `docinfo/document-id` and a `label` on every interactive component. Decide those
before your first upload and never change them.

## Deploying

```bash
pretext deploy                # push output to the gh-pages branch
pretext deploy --stage-only   # assemble in output/stage and inspect first
```

For a single target, `deploy` publishes the `web` output. For several, give each target a
`deploy-dir` and put a landing page in `site/` at the project root; everything is
assembled into `output/stage` and pushed together.

```xml
<target name="web"    format="html" deploy-dir="."/>
<target name="print"  format="pdf"  deploy-dir="pdf"/>
<target name="ebook"  format="epub" deploy-dir="epub"/>
```

`cname` on `<project>` sets a custom domain.

Any static host works — the output uses only relative links and needs no server
configuration. Netlify Drop is a fast way to get a shareable URL for a support question.

## Publishing decisions worth making early

**A landing page** — one stable, memorable URL, above all your outputs, listing the online
version, the PDF, the ebook, errata, and how to contact you. It is not the book's
`index.html`. Host it on a domain **you own**, so the book can move institutions without
breaking every link anyone has made to it.

**Say where home is, inside the book.** One sentence in the preface or colophon giving the
address of the canonical free version. Copies circulate; a reader holding a stale one
should be one sentence from the current one.

**Print-on-demand**, roughly in order of convenience → professionalism:
Lulu (trivial setup, no ISBN needed, good for class drafts) · Amazon KDP (automatic Amazon
distribution; their content-validation team may query a book that is also free online) ·
IngramSpark (own ISBN required, ~$49 setup, distribution indistinguishable from a
commercial publisher). Budget for a cover PDF to their spec.

**Versioned URLs** so old adoptions keep working:

```
https://example.org/physics/latest        →  current HTML
https://example.org/physics/physics.pdf   →  current PDF
https://example.org/physics/html-2026-08-15
```

**Instructor edition**: build it from the same source with a different publication file
(all solution components visible, a watermark) plus `<commentary>` elements — a titled,
unnumbered block designed to be included or excluded by
`<version include="instructor"/>`. Numbered items are prohibited inside `<commentary>`
precisely so both versions number identically.

## Keeping up

`pretext upgrade` (CLI ≥ 2.12) or `pip install pretext --upgrade`;
`--pre` for nightlies; `pretext update-project` inside the project afterwards; then bump
`requirements.txt` once a build succeeds. Deprecations are announced on the
`pretext-announce` group and warn at build time.
