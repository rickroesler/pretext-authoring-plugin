# Feedback-loop inventory: what tells an agent its PreTeXt is right

Research ticket [#7](https://github.com/rickroesler/pretext-authoring-plugin/issues/7) (wayfinder map
[#1](https://github.com/rickroesler/pretext-authoring-plugin/issues/1)).
Snapshot date: 2026-08-28. PreTeXt CLI (command-line interface) 2.51.0 in `.venv/`, core at `~/.ptx/2.51.0/core/`,
`vendor-pretext/` at the same vintage.

Everything below was run. The transcripts are from `scratch/loop-trial/` (gitignored): a copy of
`scratch/demo-book` mutated one breakage at a time, plus `vendor-pretext/examples/sample-book`
(10 320 lines of source) as a realistic-scale timing case.

---

## 0. Headline

**PreTeXt 2.51.0 gives an agent a genuinely good validation signal and a genuinely bad build
signal.** `pretext validate` is machine-shaped by design — a `--report-form terse` mode emits
one tab-separated message per line carrying *file, XPath (XML Path Language), line, check-name, message*, and its
exit code means what it says. `pretext build` is the opposite: it exits **0** on source that
does not validate, silently dropping or passing through the offending markup, and its only
hard failures are XML well-formedness, unresolved `xref`, duplicate `xml:id`, and a missing
external program.

So the loop for an agent is: **validate is the gate; build is not.** Three whole classes of
error (LaTeX inside `<m>`, publication-file typos, missing image files and dead links) are
caught by *nothing at all*, and the only way to see them is to render and look.

Two traps found while measuring, both worth reporting upstream:

* **A broken `jing` reports a clean bill of health.** A `jing` on `$PATH` that exits 1 printing
  nothing (e.g. the crash-on-startup `pyjing` from PyPI `jingtrang` under setuptools ≥ 81)
  makes `pretext validate` print *"PreTeXt source passed validation with no messages"* and exit
  **0** on source containing a flagrant schema violation. Exit 1 with empty stdout is read as
  "invalid, but no messages" rather than "the validator failed". §6.1.
* **A stale report survives a failed run.** When assembly fails (bad `xi:include`,
  non-well-formed XML) no new report is written, but `logs/<main>-validation.txt` from the
  previous run is left in place. An agent that reads the report file rather than stdout reads
  the *last* run's verdict. §6.2.

---

## 1. What is actually installed here

| Thing | Status | Note |
|---|---|---|
| `pretext` CLI | 2.51.0 in `.venv/` | `source .venv/bin/activate` |
| Java | OpenJDK 21.0.12 | present |
| `jing` | **not installed** | so `pretext validate` exits **2** out of the box |
| salve (`--engine salve`) | installed at `~/.ptx/2.51.0/salve/` | node 24.16.0 + npm 11.13.0 present; self-installs on first use |
| `xsltproc` | not installed | not needed — `lxml` runs the XSLT (Extensible Stylesheet Language Transformations) 1.0 stylesheet (§6) |
| `lxml` | 6.1.2 | in the venv |
| `playwright` | 1.62.0 + chromium in `~/.cache/ms-playwright` | works headless (§8) |
| TeX / `xelatex` | absent | `pretext build print` fails; no PDF signals at all |

Out of the box `pretext validate` fails with exit **2** and this message:

```
error: Validation could not be performed.
error: Install jing and make sure it is on your PATH (or name it in executables.ptx), or use
       `pretext validate --method server` or `pretext validate --engine salve`.
```

**Getting jing.** There is no jing in the venv and none is installed by the CLI. Three routes:
`--engine salve` (no Java, npm self-install, already working here); `--method server` (no local
install at all, 16 s per run, uploads the assembled source to a remote service); or a real jing.
`pip install jingtrang` puts `jing.jar` in site-packages but its `pyjing` launcher is broken on
modern setuptools, so the working shim is a two-line script on `$PATH`:

```sh
#!/bin/sh
exec java -jar .../site-packages/jingtrang/jing.jar "$@"
```

This matters for the plugin: **the first-run experience of `pretext validate` on a clean machine
is a failure**, and which fallback the plugin recommends is a real design decision.

---

## 2. The signal table

Latencies are wall-clock on this machine, warm, for `scratch/demo-book` (a 5-file book) with
the `sample-book` (10 320 lines) figure after the slash where measured.

| # | Signal | How to invoke | What it catches | What it misses | Latency |
|---|---|---|---|---|---|
| 1 | **RELAX-NG dev schema** (the real gate) | `pretext validate <target>` (`--engine jing\|salve`, `--method local\|server`, `--report-form full\|terse`) | every parent/child and attribute violation; missing required children; XML well-formedness and unresolvable `xi:include` (as an assembly error); duplicate `xml:id` | anything referential or semantic: `xref` targets, LaTeX, file existence, publication file, prose | 0.9–1.0 s / 1.8 s (jing); 0.6–0.9 s / 1.9 s (salve); 16 s (server) |
| 2 | **Experimental-construct survey** | same run, second section of the report | markup that is in the dev schema but not the production schema (Runestone exercises, `interactive`, `datafile`, dynamic exercises), with per-feature advice from `experimental-features.xml` | nothing it is meant to catch; **not** errors — exit code unaffected | included in #1 |
| 3 | **`validation-plus` stylesheet** | same run, third section; or standalone `lxml` XSLT over the assembled source | ~30 checks RELAX-NG cannot express: images with no `shortdescription`/`@decorative`, Word-import Unicode (`°`, `—`, `“ ”`, U+200B), `fn` in `tabular`, `var` outside WeBWorK, `@pause` outside a slideshow, `sidebyside` width arithmetic, `title` with `m` and no `shorttitle` | still no `xref`/link/file checks; a few checks have XPath bugs (§6.3) | included in #1; 0.03 s standalone |
| 4 | **`pretext build <target>`** | `pretext build web` / `tex` | XML syntax; unresolvable `xi:include`; **unresolved `xref`**; **duplicate `xml:id`**; deprecated elements; missing external programs | schema violations (warns at most), bad LaTeX, missing image files, dead URLs — **exits 0 on all of them** | 2.9 s web / 0.8 s tex warm (14 s first tex run: asset generation) |
| 5 | **Build-time schema warning** | side effect of #4, **only when jing is on `$PATH`** | writes `logs/schema-errors.log` (raw jing, production schema) + `logs/schema-assembled-source.xml` and warns | non-fatal: "Continuing anyway"; fires on ordinary experimental markup, so it is noisy | included in #4 |
| 6 | **`project.ptx` parse** | any `pretext` command | unknown/misspelled target attributes and illegal `@format` values, with the legal value list | attribute *name* in the message is wrong, and a raw Python `TypeError` leaks (§6.4) | ~0.3 s |
| 7 | **`publication.ptx` schema** | **not wired into the CLI** — run `jing schema/publication-schema.rng publication/publication.ptx` by hand | typo'd publication attributes and elements | nothing runs it; and the shipped schema requires `epub/cover`, which the CLI's own generated publication file omits → one false positive | 0.3 s |
| 8 | **Schema query** (`what may go inside X?`) | 52-line `lxml` script over `pretext.rng` (§7) | allowed children + attributes of an element, per context, offline, in 30 ms; dev-vs-production diff shows what is experimental | flat sets, not sequence/cardinality; no prose | 0.03 s |
| 9 | **jing's "expected" list** | a by-product of #1 | jing names *every element allowed at that point* in the error text — the single richest repair hint in the toolchain | salve truncates the same list to 12 names + "… (58 more)" | free |
| 10 | **Rendered HTML, fetched** | `pretext view web --no-launch` then `curl http://localhost:<port>/output/web/<page>.html` | page exists, HTTP (Hypertext Transfer Protocol) status, title, script/img/iframe counts, structure via `lxml` | anything MathJax/knowl JavaScript produces — the served HTML is pre-JavaScript | 1–4 ms per page after a ~3 s server start |
| 11 | **Headless browser** | `playwright` chromium (in the venv, browsers installed) | final DOM (Document Object Model): `mjx-container` counts, `mjx-merror`, console/page errors, full-page screenshot | does not *fail* on bad LaTeX — MathJax degrades silently (§8) | 5.2 s including browser launch |
| 12 | **PDF signals** | `pretext build print` | — | no TeX installed: `PTX:ERROR: cannot locate executable ... xelatex`, exit 1. Page counts, overfull boxes, float placement: **unavailable here** | n/a |
| 13 | **Link / `xref` checking** | none | — | dead external `<url href>`: nothing checks it. `xref` to a missing target *is* caught, but by the **build**, not by validate | n/a |

---

## 3. `pretext validate` in detail

### 3.1 What it actually does

Three examinations of one assembled tree, per the report preamble:

1. jing (or salve) against the **development** schema `pretext-dev.rng` — these are genuine errors;
2. a second run against the **production** schema `pretext.rng` — the difference between the two
   is by construction the set of experimental constructs, and is reported as a survey, not errors;
3. the `validation-plus` XSLT stylesheet.

Exit codes are documented and reliable (given a working validator): **0 = valid, 1 = invalid,
2 = validation could not be performed.** Experimental constructs do not affect the exit code.

### 3.2 `xi:include` handling — the good part

Validation assembles the modular source first, so it validates the whole book, not a file. To
keep attribution, the CLI knits the includes itself with a stamping loader that records
`@pi:source-uri` on each included root, then maps assembled elements back to their origin file
(`os.path.relpath(uri, start=source_dir)` — a path relative to the source directory, so
`exercises/cyclic.xml` stays distinct from `cyclic.xml`). Consequence for an agent:

* `file:` names the author's own file — usable;
* `line:` is a line of the **assembled** source, deposited at `logs/<main>-assembled.xml`, and is
  explicitly *never* a line of the author's file;
* `path:` is a numbered XPath into the assembled tree, e.g.
  `/pretext[1]/book[1]/chapter[2]/section[3]/exercise[1]/statement[1]`.

So an agent gets file + XPath but **not file + line**. Turning `file` + `path` into a cursor
position in the author's file is left to the caller — a small, obvious job for the plugin.

### 3.3 Terse form — the machine interface

`--report-form terse` writes `logs/<main>-validation.txt` as
`file \t xpath \t line \t check \t message`:

```
sec-section-name.ptx	/pretext[1]/book[1]/chapter[1]/section[1]/paragraph[1]	94	schema	error: element "paragraph" not allowed here; expected element "activity", "algorithm", … "worksheet"
ch-physics-test.ptx	/pretext[1]/book[1]/chapter[2]/section[3]/exercise[1]/statement[1]	191	experimental	found attribute "correct", but no attributes allowed here
sec-section-name.ptx	/pretext[1]/book[1]/chapter[1]/section[1]/p[1]	94	unicode-em-dash	PTX:WARNING: A run of text contains a Unicode character for an em-dash (U+2014 …)
```

The `check` column is a stable machine-readable category: `schema`, `experimental`, or the
stylesheet's own message id (`unicode-degree`, `image-description-missing`,
`title-math-no-shorttitle`, `var-outside-webwork`, `pause-outside-slideshow`, …). An agent can
filter on it — for example ignore `experimental` while drafting, or treat
`image-description-missing` as a TODO rather than a defect.

The full form is for humans: same four locators plus a text excerpt and, for experimental
constructs, several sentences of advice pulled from `experimental-features.xml`:

```
The attribute "correct" is not allowed on this element.
    file: ch-physics-test.ptx
    path: /pretext/book/chapter[2]/section[3]/exercise[1]/statement
    line: 191
    text: <statement correct="yes">
    advice: Several attributes here are experimental switches for
            interactive exercises (for example a programming language,
            or adaptive behavior). The attribute names and their
            values are aligned with the Runestone platform and may
            change.
    check: experimental
```

### 3.4 jing vs salve

Same findings, different words — and the difference matters for an agent.

Same violation, jing:

```
error: element "paragraph" not allowed here; expected element "activity", "algorithm",
"argument", "aside", "assemblage", … "video", "warning" or "worksheet"
```

salve:

```
error: <paragraph> is not allowed here.
error: Missing required content. Expected one of: <p>, <blockquote>, <pre>, <image>, <video>,
<audio>, <program>, <console>, <tabular>, <figure>, <table>, <listing>, … (58 more).
```

**jing enumerates every element permitted at that point; salve truncates at 12.** That
enumeration is the most useful repair hint in the whole toolchain — it answers "what may go here"
*in context*, which is strictly better than any static schema lookup (§7). Recommendation for the
plugin: prefer jing when available; treat salve as the no-Java fallback and expect to fall back
to a schema query for the full list.

Timings (three runs each, demo book / sample-book): jing 0.93, 1.00, 0.98 s / 1.82 s;
salve 0.77, 0.62, 0.86 s / 1.92 s; `--method server` 16.1 s. salve is slightly *faster* than jing
locally (no JVM (Java Virtual Machine) start) and both are fast enough for a save hook even on a real book.

`--method server` needs no local install but takes 16 s and posts the assembled source to a remote
service — a privacy consideration worth a line in the plugin docs.

### 3.5 Noise at realistic scale

`pretext validate` on PreTeXt's own `examples/sample-book`, by check:

```
    160 experimental
     23 image-description-missing
     10 var-outside-webwork
      2 pause-outside-slideshow
      1 schema
```

Exit code 1 — **PreTeXt's own sample book does not currently validate** (one genuine error:
`rune.xml … de-object[9] … found attribute "reduce", but no attributes allowed here`). Any agent
policy of "iterate until validate is silent" must be scoped to *messages in files the agent
touched*, or it will chase 195 pre-existing messages in a real manuscript.

---

## 4. Build signals

### 4.1 Where messages land

* **stdout** carries a level-tagged, ANSI (American National Standards Institute)-coloured stream (`warning:`, `error:`, `fatal:`), and
  repeats every error in a summary block at the end, which is the easiest thing to parse.
* **`logs/<timestamp>.log`** — one uncoloured file per run, `LEVEL   : message`. Note the errors
  are re-emitted into the log at `INFO` level inside the summary block, so grepping the log for
  `ERROR` is not sufficient; grep for `PTX:ERROR`.
* **`logs/schema-errors.log`** + **`logs/schema-assembled-source.xml`** — written only when jing
  is on `$PATH`, holding raw jing output against the *production* schema.
* XSL (Extensible Stylesheet Language) messages are surfaced inline, prefixed with `*`, e.g. `* PTX:DEPRECATE: …`.

### 4.2 Exit codes

`pretext build` exits 1 only on: unparseable `project.ptx`; non-well-formed XML; an unresolvable
`xi:include`; an unresolved `xref`; a duplicate `xml:id`; a missing external program. It exits
**0** on schema-invalid source, on unknown elements, on missing image files and on broken LaTeX.

### 4.3 The build-time schema warning

With jing present, a build checks the assembled source against the production schema and warns
without stopping:

```
warning: PreTeXt source did not pass schema validation (11 messages); unexpected output may result. Continuing anyway.
warning:   Messages: .../logs/schema-errors.log
warning:   Line numbers refer to the assembled source: .../logs/schema-assembled-source.xml
warning:   Run `pretext validate` for a report that names the source files.
```

Because it uses the *production* schema, ordinary Runestone/`interactive` markup trips it: the
demo book raises 11 such "errors" while `pretext validate` (dev schema) is clean. An agent must
not treat this warning as a defect signal.

---

## 5. Deliberate breakages: who catches what

Each row is one mutation of `scratch/demo-book`, applied alone, then `pretext validate web`
(jing) and `pretext build web`. Verbatim messages follow the table.

| Breakage | validate | build web | Caught by |
|---|---|---|---|
| Invalid element (`<paragraph>`) | **1** | **0** | validate only; build warns `PTX:DEPRECATE` |
| Invented element (`<blah>`) | **1** | **0** | validate only; build emits its **text into the HTML**, no warning |
| `<m>` outside `<p>` | **1** | **0** | validate only; renders anyway |
| Missing required `<title>` on a section | **1** | **0** | validate only |
| `<xref ref="no-such-target"/>` | **0** | **1** | build only |
| Duplicate `@xml:id` | **1** | **1** | both (validate reports it as an assembly/ID error, not a schema message) |
| Non-well-formed XML | **1** | **1** | both, with file+line+column **in the author's file** |
| `xi:include` of a missing file | **1** | **1** | both (assembly fails; **no report written** — §6.2) |
| Bad LaTeX in `<m>` (`\frac{1}{2`, `\notacommand`) | **0** | **0** | **nothing** |
| `<image source="nope.png"/>` (file absent) | **0** | **0** | **nothing** (validation-plus does flag the missing description) |
| Dead external `<url href>` | **0** | **0** | **nothing** |
| Typo in `publication.ptx` (`levl="2"`, `<bogus-element/>`) | **0** | **0** | **nothing in the CLI**; jing + `publication-schema.rng` catches it |
| Typo in `project.ptx` (`format="htlm"`) | — | **1** | build, with a misleading attribute name (§6.4) |

Verbatim, in order of usefulness.

**Unresolved `xref`** — the best error message in the toolchain, and build-only:

```
error: * PTX:ERROR:   a cross-reference ("xref") uses references [no-such-target] that do not
point to any target, or perhaps point to multiple targets.  Maybe you typed an @xml:id value
wrong, maybe the target of the @xml:id is nonexistent, or maybe you temporarily removed the
target from your source, or maybe an auxiliary file contains a duplicate.  Your output will
contain some placeholder text that you will not want to distribute to your readers.
fatal: There were errors during the build.  See above.
```

**Non-well-formed XML** — the only signal that gives file, line *and* column in the author's own
file:

```
error: Error loading .../source/sec-section-name.ptx: Opening and ending tag mismatch:
p line 6 and section, line 8, column 11 (sec-section-name.ptx, line 8)
error: Could not assemble source for validation: …
```

**Duplicate `xml:id`** — validate reports against the merged file, build reports properly:

```
(validate) error: ID sec-section-name already defined, line 96, column 31 (merged.xml, line 96)
(build)    error: * PTX:ERROR: the @xml:id value "sec-section-name" should be unique, but is
                  authored multiple times.
```

**Deprecated element, build exit 0** — note the phrase *silently not appear*:

```
warning: * PTX:DEPRECATE: (2015-03-13) the "paragraph" element is deprecated and any contained
content will silently not appear, replaced by functional equivalent "paragraphs" (1 time)
*              located at: pretext/book/chapter/section/paragraph
*              located within: "sec-section-name" (xml:id), "Section Title" (title)
```

**Invented element `<blah>`** — build exit 0, no message at all, and grepping the built page shows
its content shipped to the reader:

```
$ grep -o "Kept\.\|invented element content\|E=mc" output/web/sec-section-name.html | sort | uniq -c
      1 E=mc
      1 Kept.
      1 invented element content
```

**Bad LaTeX** — validate 0, build 0, no console error, no `mjx-merror`. The only trace is in the
rendered text of the page:

```
Broken: \(\frac{1}{2\) and unknown macro \notacommand𝑥.
```

An unbalanced brace leaves the raw `\( … \)` delimiters in the text; an unknown macro is typeset
as literal text inside a `mjx-container`. Both are detectable by a heuristic over rendered text —
a surviving backslash or `\(` in body text — and by nothing else in the toolchain.

**`validation-plus` messages** (from one `<p>` pasted out of a word processor, plus an
undescribed image):

```
sec-section-name.ptx	…/p[1]	94	unicode-degree	PTX:WARNING: A run of text contains a Unicode character for a degree symbol (U+00B0 …) the symbol will behave better as the empty element "<degree/>"
sec-section-name.ptx	…/p[1]	94	unicode-em-dash	PTX:WARNING: … will behave better as the empty element "<mdash/>" …
sec-section-name.ptx	…/p[1]	94	unicode-double-quotes	PTX:WARNING: … should be replaced by the "<q>" element enclosing content …
sec-section-name.ptx	…/figure[1]/image[1]	96	image-description-missing	PTX:WARNING: You have an image without any description and do not declare the image to be decorative. Because of this, output may not be accessible. …
```

**`project.ptx`** — good value list, wrong attribute name, leaked Python error:

```
fatal: Failed to parse project.ptx. Check the entire file, including all targets, and fix the following errors:
error: One of the targets has an attribute with illegal value: @targets="htlm" is not allowed.
       Pick from the values: 'html', 'latex', 'pdf', 'epub', 'epub_nozip', 'kindle', 'braille',
       'revealjs', 'beamer', 'webwork' or 'custom'.
error: 'NoneType' object is not subscriptable
```

**`publication.ptx`** — nothing in the CLI notices; run jing by hand and it does:

```
$ jing vendor-pretext/schema/publication-schema.rng publication/publication.ptx
publication.ptx:112:26: error: attribute "levl" not allowed here; expected attribute
  "chapter-start", "level" or "part-structure"
publication.ptx:113:21: error: element "bogus-element" not allowed anywhere; expected the
  element end-tag or element "blocks", "equations", "exercises", "figures", "footnotes",
  "openproblems" or "projects"
publication.ptx:412:10: error: element "epub" incomplete; missing required element "cover"
```

(The third is a false positive against the CLI's own generated publication file.)

---

## 6. The `validation-plus` stylesheet

`vendor-pretext/schema/pretext-validation-plus.xsl` (981 lines, XSLT **1.0**, shipped at
`~/.ptx/<version>/core/schema/`) is where PreTeXt puts checks a grammar cannot express. Its
templates, by subject:

* **Deprecations with explanation** — `@filebase`, `cite`, `circum`, `sage[@copy]`, `image[@copy]`,
  `author/xref`, `latex-image-preamble` without `@syntax`, `@workspace`, `jslibrary`.
* **Real-time structural checks** — `sidebyside` panel widths and margins that must sum sensibly;
  `sidebyside/exercise`; `slate` dimensions; captions on `figure|table|listing|list`; `tabular`
  geometry; `fn` inside `tabular` (use `tn`); `blocks[@randomize='no']`; `evaluation` shape.
* **Accessibility** — `image` with neither `shortdescription`/`description` nor `@decorative`.
* **Word-processor residue** — text containing `°`, `×`, `–`, `—`, `‘ ’`, `“ ”`, and zero-width
  space, each with the PreTeXt element to use instead.
* **Context checks** — `var` outside `webwork`, `@pause` outside a `slideshow`, `image[@pg-name]`
  outside `webwork`, `fillin` inside `m`/`md`, WeBWorK `tabular` rule widths.
* **Advice** — `title` containing `m` with no `shorttitle`.

**Running it standalone needs no xsltproc.** `lxml` executes it in ~0.03 s over the assembled
source; the report is the transform's text output (the stylesheet sets `method="text"`), not
`xsl:message`:

```python
from lxml import etree
tr = etree.XSLT(etree.parse("vendor-pretext/schema/pretext-validation-plus.xsl"))
print(str(tr(etree.parse("book/logs/main-assembled.xml"))))
```

```
################################################################
PTX:WARNING: /pretext[1]/book[1]/chapter[1]/title[1]
You have a title containing m but no shorttitle.
Because this title will be used many places, errors may result.
Please add a shorttitle.
[title-math-no-shorttitle]
################################################################
```

Two constraints: it is XSLT 1.0 with a `<!ENTITY % entities SYSTEM "../xsl/entities.ent">`, so it
must be run from a checkout where `../xsl/` exists; and `$single.line.output='yes'` gives the
sortable one-line form the CLI uses. Running it directly is worth doing only for a
faster-than-`validate` inner loop; otherwise take it from the consolidated report.

### 6.1 Trap: a broken jing reports success

With a `jing` on `$PATH` that exits 1 and prints nothing, on source containing
`<paragraph>definitely invalid</paragraph>`:

```
$ printf '#!/bin/sh\nexit 1\n' > bin/jing && chmod +x bin/jing
$ PATH=bin:$PATH pretext validate web
…
PreTeXt source passed validation with no messages.
$ echo $?
0
```

The CLI treats jing exit 1 as "invalid, here are the messages" and exit ≥ 2 as "jing failed"
(`utils.py:355`), so a validator that crashes with exit 1 and an empty stdout is indistinguishable
from a clean pass. This is exactly what PyPI `jingtrang`'s `pyjing` does under setuptools ≥ 81
(`ModuleNotFoundError: No module named 'pkg_resources'` on stderr, exit 1). Upstream fix: require
at least that jing produced *some* output, or probe the validator once at startup.

### 6.2 Trap: the report file outlives its run

When assembly fails, `pretext validate` errors out before writing a report — but
`logs/<main>-validation.txt` from the previous run is still on disk, with the previous verdict.
Observed directly: after a bad `xi:include` the report still listed the `<m> is not allowed here`
message from two mutations earlier. An agent must key off the **exit code and stdout**, never off
the presence or content of the report file, or delete the report before each run.

### 6.3 Bug: `title-math-no-shorttitle` never fires below chapter level

```xslt
<xsl:template match="title[m]">
  <xsl:if test="parent::chapter|appendix|preface|…|section|subsection|…">
```

Only the first arm of the union is a `parent::` step; the rest are relative *child* steps
(`appendix`, `section`, …), which can never match a child of `title`. Confirmed empirically: a
`<m>` in a chapter title raises `title-math-no-shorttitle`; the same `<m>` in a section title
raises nothing. One-line upstream fix (`parent::*[self::chapter or self::section or …]`).

### 6.4 Bug: `project.ptx` errors name the wrong attribute

Both `format="htlm"` and an invented `outputdir="web"` are reported as `@targets=…`:

```
error: Either one of the targets or the root project element has an extra attribute it
       shouldn't: targets="web"
error: 'NoneType' object is not subscriptable
```

The offending attribute's own name never appears, and a raw Python `TypeError` is printed as if it
were a second user-facing error. The value enumeration in the `@format` case is genuinely good and
worth keeping.

---

## 7. Answering "what may go inside `<statement>`?" mechanically

**Yes, and cheaply.** `scratch/loop-trial/rng-children.py` is 52 lines of `lxml` over
`pretext.rng`, and runs in **0.03 s**:

```
$ python rng-children.py statement
statement: 3 definition(s) in vendor-pretext/schema/pretext.rng

--- defined in <define name="Statement">, line 1492
    children  (19): aside, audio, biographical, blockquote, console, figure, historical, image,
                    list, listing, p, pre, program, sage, sbsgroup, sidebyside, table, tabular, video
    attributes(0): (none)

--- defined in <define name="StatementExercise">, line 3432
    children  (19): … same …
--- defined in <define name="StatementExerciseWW">, line 4132
    children  (5): image, instruction, p, pre, tabular
```

The interesting part is that the question **has three answers**: `statement` is defined three times
in the grammar, and the WeBWorK one is a different language. A reference table in a skill file
cannot express that; a query can.

Pointed at the development schema it also answers *what is experimental here* by difference:

```
$ python rng-children.py statement vendor-pretext/schema/pretext-dev.rng
--- defined in <define name="Statement">, line 1492
    children  (22): … + datafile, interactive, query
--- defined in <define name="TrueFalse">, line 540
    children  (1): p
    attributes(1): correct
```

Two implementation facts anyone rebuilding this must know, both learned by getting them wrong:

1. **`pretext-dev.rng` is `<include href="pretext.rng"/>` plus additions** — the loader must
   follow `<include>` or the dev schema looks almost empty.
2. **`combine="choice"` means union, not override.** `BlockStatement` is defined once in
   `pretext.rng` and three more times in `pretext-dev.rng`; taking the last definition (the
   natural "later wins" instinct) reports `statement` as accepting only `<query>`. The schema
   README states the invariant explicitly: the dev overlay is *purely additive*, `include` +
   new patterns + `combine="choice"` only, which is what makes "dev-only" a sound definition of
   experimental.

Limits of this prototype: it returns a flat *set* of permitted children and attributes, losing
sequence, cardinality and interleave (so it cannot say "`title` must come first", the actual
content of the `missing required element "title"` error). Filling that in means interpreting
`<group>`/`<optional>`/`<oneOrMore>`/`<interleave>` — a bigger job, and probably not worth it
because **jing already answers the contextual question better** (§3.4, signal #9): its "expected
element …" list is the exact set permitted *at that point in that document*. A probe document plus
jing is a legitimate and more accurate oracle than any static query.

Other routes to the same information, for the record: the literate schema source
`schema/pretext.xml` (4 259 lines, itself a PreTeXt article — buildable, and the place that
explains *why* the grammar is shaped as it is); the rendered `schema-litprog` HTML; and Siefken's
per-element browser, all already documented in `skills/pretext/references/10-schema.md`. The
script's advantage is that it is offline, version-locked to the installed CLI
(`~/.ptx/<version>/core/schema/`), and returns a machine-readable set.

---

## 8. Rendered-output inspection

**Serving.** `pretext view web --no-launch` (note: `--no-launch`, not `--no-launch-browser`)
starts a static server, prints the port, and takes ~3 s:

```
Server will soon be available at http://localhost:8128
The target `web` will be available at http://localhost:8128/output/web
```

`curl` then works normally — 1–4 ms per page — and `lxml` parses the result:

```
$ curl -s http://localhost:8128/output/web/ch-physics-test.html
HTTP 200 20598b 0.001210s
 title: Kinematics Smoke Test
 h1: ['Chapter 2 Kinematics Smoke Test']
 scripts: 14 imgs: 2 iframes: 0
 has MathJax: True | has knowl: True
```

Files can equally be read straight off disk from `output/web/`; the server only matters when
JavaScript needs a real origin. Structural assertions an agent can make cheaply from the served
HTML: page exists for each expected `xml:id`, `<title>`/`<h1>` match the intended titles, image
`src` files resolve, expected number of exercises.

**Headless browser.** `playwright` 1.62.0 is in the venv and chromium is installed; a launch +
navigate + full-page screenshot + console capture is **5.2 s**:

```
status 200 title Kinematics Smoke Test
MathJax rendered containers: 1
console issues: [('warning', 'Missing template or content area for user dropdown')]
```

It gives an agent: a screenshot for a vision pass, `mjx-container` counts (did the math typeset at
all?), `mjx-merror` nodes (MathJax's own error markup), and console/page errors. What it does *not*
give is a failure on bad LaTeX: as shown in §5, `\frac{1}{2` produced **zero** `mjx-merror` nodes,
zero console errors, and simply left `\(\frac{1}{2\)` in the text. The workable check is textual —
scan the rendered `inner_text` for surviving `\(`, `\)`, or `\word` sequences.

**PDF.** Nothing is available here: no TeX, so `pretext build print` fails with
`PTX:ERROR: cannot locate executable with configuration name PdfMethod.XELATEX as command
xelatex` (exit 1). `pretext build tex` succeeds and produces `.tex` in 0.8 s, so *LaTeX generation*
can be smoke-tested without TeX, but page counts, overfull `\hbox` warnings, float placement and
"does the figure land on the right page" are unmeasurable without a TeX install. `pymupdf` and
`PyPDF2` are in the venv, so page counting is available the moment a PDF exists.

---

## 9. Gaps — checks an author needs that nothing automates

Ordered by how much damage they do. This list feeds the upstream-recommendations ticket.

1. **LaTeX inside `<m>`/`<me>`/`<md>` is unchecked, end to end.** Validate passes, build passes,
   HTML renders, no console error. Every math error reaches the reader. This is the single largest
   hole for an agent writing a STEM (science, technology, engineering and mathematics) textbook, where most of the authored characters are LaTeX. A
   check is feasible: MathJax and KaTeX both expose a parse-only API, and `\newcommand` macros from
   `docinfo/macros` are available to the checker.
2. **`build` exits 0 on schema-invalid source, and ships the garbage.** An invented element's text
   is passed through into the HTML with no message at all. Any agent loop that treats a successful
   build as "it worked" is wrong. Either the build should fail on dev-schema violations, or the
   plugin must always gate on `validate` and never on `build`.
3. **`publication.ptx` is unvalidated by the CLI.** A misspelled attribute or element is silently
   ignored, so a publisher setting that does nothing is indistinguishable from one that works. The
   schema exists (`schema/publication-schema.rng`) and jing catches these; it is simply not wired
   in. (Its `epub/cover` requirement needs relaxing first — it rejects the CLI's own template.)
4. **Referenced files are never checked to exist.** `<image source="nope.png"/>` builds clean and
   produces a broken image. Same for `@source` on other elements and for asset paths in the
   publication file.
5. **External links are never checked.** A dead `<url href>` produces no signal anywhere. (A
   link-checker is I/O-bound and noisy, so this belongs in a separate opt-in pass, not the build.)
6. **No file+line in the author's own source.** Validation gives file + XPath + a line number in
   the assembled file. Every editor integration, and every agent applying a fix, needs
   file + line + column in the file it is about to edit. The mapping is mechanical
   (`@pi:source-uri` already exists in the pipeline) but is not done.
7. **No incremental or single-file validation.** Every check assembles the whole book. It is fast
   today (1.8 s on a 10 000-line book) but there is no way to ask "is this fragment I just wrote
   valid in this context", which is what an agent editing one section actually wants.
8. **The validator's own failure is indistinguishable from success** (§6.1), and the report file
   outlives a failed run (§6.2). Both make silent false-negatives possible in an automated loop.
9. **No prose/pedagogy signal at all, and none possible from the toolchain** — is the notation
   consistent, does the exercise match the section, are the units right, is the cross-reference
   pointing at the *right* target rather than merely an existing one. This is the part of "is my
   PreTeXt right" that only a reader (human or model) can answer, and it is where an agent adds
   value that no linter can.
10. **PDF-side signals are unreachable without a TeX install** — and a TeX install is a large ask
    for a newcomer. The plugin should be explicit that print output is unverified unless the
    author installs TeX, and should smoke-test `build tex` as a partial substitute.
11. **Message noise has no per-file or per-diff filter.** `validate` reports the whole book;
    PreTeXt's own sample book emits 196 messages and exit 1. An agent needs "messages in the files
    I changed", which today must be done by post-filtering the terse report on the `file` column.

---

## 10. Recommended loop for the plugin

1. `pretext validate <target> --report-form terse` after every edit — **the gate**. Parse the tab-
   separated report; key off exit code, not the file. Filter `check == "experimental"` out of the
   error set (report it once, as an advisory). Filter by the `file` column to the files just
   touched.
2. Prefer `--engine jing` for the "expected element …" enumeration; fall back to `--engine salve`
   (no Java) and, only if both are impossible, `--method server`.
3. `pretext build web` second — for `xref`, `xml:id` and deprecation errors that validate cannot
   see. Treat exit 0 as *necessary, not sufficient*; grep stdout for `PTX:ERROR`, `PTX:WARNING`
   and `PTX:DEPRECATE` regardless of exit code.
4. Query the schema (`rng-children.py` over `~/.ptx/<version>/core/schema/pretext.rng`) *before*
   writing unfamiliar markup, rather than iterating against the validator.
5. Render and look, for anything containing math or images: fetch the page (curl or disk), and for
   math-heavy pages run the browser pass and scan rendered text for surviving backslashes.
