# Manuscript conversion landscape: LaTeX / Markdown / Word → PreTeXt

Research ticket [#6](https://github.com/rickroesler/pretext-authoring-plugin/issues/6) (wayfinder map [#1](https://github.com/rickroesler/pretext-authoring-plugin/issues/1)).
Snapshot date: 2026-08-28.

Sources: the installed PreTeXt CLI (`.venv/`, 2.51.0), a purpose-installed 2.49.1 and a
purpose-installed `@pretextbook/import` 0.9.0 in `scratch/convert-trial/`; the
`PreTeXtBook/pretext-cli` CHANGELOG and commit history via the GitHub API; the PreTeXt Guide
source in `vendor-pretext/doc/guide/`; the RELAX-NG schema at `~/.ptx/2.51.0/core/schema/`;
and a sweep of GitHub plus the pretext-support / pretext-dev / pretext-announce Google Groups.

Two trials were run locally on the same 137-line LaTeX sample: the dead `pretext import` (§3)
and the live TypeScript importer that replaced it (§4). §5 compares them.

---

## 0. Headline

Three things changed in the last six weeks, and all three post-date most of what is written
about PreTeXt conversion:

1. **`pretext import` is gone.** It shipped in CLI 2.4.0 (2024-05-02) and the 2.51.0 changelog
   (2026-08-28 — the release in `.venv/`) records:
   > `pretext import` command (LaTeX conversion via plastex) completely removed. It was an
   > experimental feature that never reached a stable enough state to be useful.
2. **A TypeScript replacement is live and much better**: `@pretextbook/import` (npm, v0.9.0,
   2026-08-22) over `@unified-latex/unified-latex-to-pretext`. It is not a CLI command — it is a
   library behind the VS Code extension's *Import Project* command and the pretext.plus web app.
3. **The community has already moved to LLM-assisted conversion.** On pretext-support in
   March–April 2026, faculty recommend Gemini section-by-section; Rob Beezer says he uses Claude
   Code for PreTeXt development and that AI runs on clean LaTeX "approached the quality of
   [Farmer's] tool".

For the plugin, the question is therefore not "which converter do we drive?" but "where in an
already-crowded, already-LLM-shaped pipeline do we add value?" The trials below say: **not in
parsing LaTeX — that is solved well enough by `@pretextbook/import` — but in the repair pass
after it, which is exactly schema knowledge plus a `validate`/`build` loop.**

---

## 1. Capability matrix

**good** = usable as-is · **partial** = present but wrong element/attribute/order · **lost** =
silently dropped · **broken** = ill-formed or non-parsing output · **flagged** = deliberately
marked for a human.

| Route | Inputs | Math | Figures | Tables | Exercises | Cross-refs | Theorem envs | Maintenance | How you run it |
|---|---|---|---|---|---|---|---|---|---|
| **`@pretextbook/import`** (unified-latex, TS) | `.tex`, `.md`, `.ptx`, `.zip`, `.tar.gz`, pasted text | **good** (`<md number="yes">`, modern elements) | **partial** (child order wrong, `width="0.5\textwidth"`) | **good** (`<title>` misplaced only) | **lost as semantics** (`<ol>`, no `<exercises>`); `exam` class supported upstream | `\ref`/`\label` **good**; `\autoref` **flagged** as `<TODO>` | **good** | active (Levin/Siefken; releases through 2026-08) | npm library; VS Code *Import Project*; pretext.plus |
| **pandoc-pretext** (`pretext.lua`) | anything pandoc reads: Markdown, LaTeX, **docx**, HTML, RTF, JATS | good; `\intertext` broken, `\text{}` in body text dropped | good (alt → `<shortdescription>`); empty figures on RTF | good (full `<col>` model, colspan) | none (README "future work") | `\label` → `xml:id` | good, with the bundled custom LaTeX reader | active (Levin; rewritten 2026-07-15 for pandoc 3.0 writer API) | `pandoc in.md -t pretext.lua -o out.ptx` |
| **SL3X / UTMOST service** (David Farmer) | whole LaTeX repo | strong | strong | strong | hand-tuned | strong | strong | Farmer; repo public but README says "Not yet ready to be used" | email `farmer@aimath.org`; free for open licences |
| **`pretext import`** (plasTeX) | single/modular `.tex` | inline good; `\[…\]` good; `equation` **broken** | **broken** (non-parsing `@width`) | **partial** | **lost** | `\ref`/`\eqref` partial; `\autoref` **lost** | good only when `\newtheorem` name is a PreTeXt element | **removed 2026-08-24** | `pip install pretext==2.49.1` |
| **LLM-assisted** (Gemini / Claude, current community practice) | anything, incl. PDFs and handwriting | reported good; Prefigure diagram generation praised | — | — | — | — | — | ad hoc, no shared prompts | copy-paste, section by section |

---

## 2. Route by route

### 2.1 `pretext import` — the plasTeX importer (dead)

`pretext import <file.tex> [-o dir]`, help text "Experimental: convert a latex file to pretext".
It drove [plasTeX](https://plastex.github.io/plastex/) (upstream PyPI `plastex>=3,<4` — there is
**no** `PreTeXtBook/plastex` fork; that repo 404s) with a PreTeXt-specific Jinja renderer: 33
`*.jinja2s` templates added in one commit,
[`6798022` "LaTeX to PreTeXt importer (#719)"](https://github.com/PreTeXtBook/pretext-cli/commit/67980228),
2024-05-01, by Oscar Levin. Improved once ([#732](https://github.com/PreTeXtBook/pretext-cli/issues/732),
modular-file support, image generation disabled), then untouched for two years. Removed in
[`20fefe3` (PR #1219)](https://github.com/PreTeXtBook/pretext-cli/commit/20fefe36), 2026-08-24;
the 2.50.0 pip package already has no `pretext/plastex/` directory and no `import` subcommand,
so **2.49.1 is the last version that has it**.

Its ceiling is readable straight off the templates (still installed at
`scratch/convert-trial/venv249/lib/python3.12/site-packages/pretext/plastex/`):

- `Crossref.jinja2s` defines `ref`, `eqref`, `cref`/`Cref`, `pageref` — **not `autoref`**. An
  unhandled macro falls to plasTeX's default renderer and emits nothing.
- `Thms.jinja2s` is literally `<{{ obj.thmName }}>` — the element is named after the LaTeX
  environment. `theorem`, `definition`, `example`, `lemma` are also PreTeXt elements and survive;
  `\newtheorem{claim}`, `{fact}`, `{principle}`, `{law}` (common in physics) would emit elements
  the schema rejects outright.
- `math.jinja2s` computes every display as `obj.mathjax_source[2:-2]` — a blind two-character
  trim. Correct for `\[…\]`, catastrophic for `\begin{equation}` (see §3.2 item 3).
- `Floats.jinja2s` emits `<caption>` for both `figure` and `table`; PreTeXt `<table>` takes
  `<title>`, and `<figure>` wants the caption **before** the image
  (`pretext.rnc:918` — `element figure { MetaDataCaption, Landscape?, (Image | …) }`).
- `graphicx.jinja2s` emits `width="{{ image.width.px }}"`, which for a `\textwidth`-relative
  width interpolates to `&images/img-0001.png-width;&px;` — undefined XML entities, file does not
  parse.
- Numbered equations use `id=`, not `xml:id=`.
- **No exercise handling at all** — no `<exercise>`, `<task>` or `<solution>` in any template.

There is essentially no complaint history in the tracker (only #719, #732 and the removal): the
feature seems to have been little-used rather than heavily litigated.

### 2.2 What the PreTeXt Guide says

The Guide's one appendix on the subject —
`vendor-pretext/doc/guide/appendices/latex-conversion.xml`, published at
[pretextbook.org/doc/guide/html/latex-conversion.html](https://pretextbook.org/doc/guide/html/latex-conversion.html)
— names **no tool at all**. It names a person:

> As part of the UTMOST project, we offer a service to help convert existing textbooks from
> LaTeX to PreTeXt. The service is free if you are planning to release your book with an open
> license. The conversion will only be 95 percent correct, but that means it will take you 20
> times less effort than converting it yourself.

The prescribed workflow: push the *entire, unedited* LaTeX source to GitHub (add `davidfarmer` as
collaborator if private), email `farmer@aimath.org`, receive a PR, review for "consistent
mis-interpretations of your intent", iterate with Farmer, then take ownership of the last 5%. It
front-loads a prerequisite that matters directly to the plugin's newcomer track:

> Before converting your book, familiarize yourself with PreTeXt to the point of being able to
> compile the sample article and sample book into HTML and PDF. […] That way, you will be on
> familiar ground when you finish the last 5 percent of the conversion.

And it closes the door on dual-sourcing:

> This service is for authors who wish to consider having PreTeXt as the "official" source of
> their textbook. It is not feasible to maintain LaTeX source and expect to have all of the
> features of PreTeXt.

The author FAQ (`doc/guide/author/author-faq.xml`) is blunter about automation:

> There have been many attempts to convert TeX/LaTeX to more modern document formats. They are
> not hard to find — none is satisfactory. We know because we have spent many years trying to
> adapt them to our purposes.

with the reason, which is the design constraint on *any* converter, LLM included:

> LaTeX allows authors enough freedom that it is impossible to accurately discern intent in a
> totally automated way.

Its only concrete self-help advice is a search-and-replace tip: author with `\(…\)` rather than
`$…$` so a mechanical pass to `<m>…</m>` is tractable.

The website page [pretextbook.org/conversions.html](https://pretextbook.org/conversions.html) is
broader than the Guide and lists three routes: the Farmer/UTMOST service, `oscarlevin/pandoc-pretext`,
and `virgilpierce/jpconvert` for Jupyter notebooks (2 stars, last pushed 2019-11-19 — abandoned).
It frames the pandoc writer as "an aid for initiating a permanent conversion from a legacy format."

The long-standing community position, from pretext-support
["Converting from LaTeX to PreTeXt"](https://groups.google.com/g/pretext-support/c/HjY118eApeQ)
(Aug 2018): Farmer argues there is no automated way and never will be, because there is no
specification of "proper" LaTeX markup — he illustrates with 67 different definitions of the
macro `\be`. In the same thread Bob Plantz reports converting a 560-page book: tried automation
extensively, gave up, did it by hand, recommends others do the same.

### 2.3 `@pretextbook/import` + `unified-latex-to-pretext` — the live first-party route

This is what replaced `pretext import`, and it is where the effort now goes.

- **Engine**: `@unified-latex/unified-latex-to-pretext` in
  [siefkenj/unified-latex](https://github.com/siefkenj/unified-latex) (pushed 2026-08-18).
  Created 2024-05 by Jason Siefken; **Oscar Levin is now the main contributor** — PR #156 (merged
  2026-08-18) adds the `exam` class (questions/parts/choices/solutions), top-level divisions for
  article/book/slideshow, Beamer + `tikzpicture`, the full `align`/`equation`/`gather`/`multline`/
  `alignat` set, `thebibliography`/`\bibitem` plus BibTeX/CSL, and `<bibinfo>`. Modules confirm
  scope: `create-table-from-tabular.ts`, `exam-subs.ts`, `biblio-csl.ts`, `break-on-boundaries.ts`,
  `expand-user-defined-macros.ts`, `katex-subs.ts` + `katex-support.json` (reports macros
  MathJax/KaTeX won't support), `dropped-subs.ts`.
- **Wrapper**: [PreTeXtBook/pretext-tools](https://github.com/PreTeXtBook/pretext-tools), a
  monorepo (pushed 2026-08-29) with `import` (v0.9.0), `latex-pretext`, `latex-style-pretext`,
  `markdown-style-pretext`, `remark-pretext`, `ptxast`, `format`, `schema`, `visual-editor`,
  `vscode-extension` (v1.3.0). `packages/import/SPEC.md` is 839 lines.
- **Pipeline**: extract → analyze roots → expand `\input`/`\include` or `xi:include` → clean →
  convert → split into one file per division at `splitLevel` → generate `project.ptx` /
  `publication.ptx` → route assets. Output is a **buildable project**, not a fragment. Pure
  TypeScript — deliberately no pandoc, plasTeX or Python dependency, so the VS Code extension
  needs no external toolchain.
- **Cleaning** is a TypeScript port of [davidfarmer/PreprocessLaTeX](https://github.com/davidfarmer/PreprocessLaTeX)
  (Farmer's JS preprocessor, last commit 2026-04-27): drop comments, normalize whitespace, expand
  includes, rewrite plain-TeX font directives, scrub presentation macros. Every mutation is
  reported as a structured `CleaningWarning` for a review UI — see the three warnings in §4.1.
- **Its own SPEC §7 "Known limitations"**: image references are **not rewritten** (`<image source>`
  still points at original paths); asset basenames flattened so same-named images in different
  directories "collide silently"; "Only the **first author** is imported; `\and` co-authors are
  dropped"; ".bib files are copied but **bibliographies are not converted**"; "Detection
  heuristics favor LaTeX: a Markdown document containing `\section` or `\begin{` anywhere is
  detected as LaTeX"; no zip-bomb guards.
- **Surfaces**: VS Code `pretext-tools` v1.3.0 commands `pretext-tools.importProject` and
  `pretext-tools.convertText` (selection-level, announced 2024-12-12 on pretext-announce as
  "Much improved 'inline' converters", all marked experimental); and pretext.plus's web wizard.
  **There is no CLI entry point** — which is the gap a Claude Code plugin can fill trivially, as
  §4 demonstrates with a 12-line Node script.

### 2.4 Pandoc — `oscarlevin/pandoc-pretext`

Pandoc has no native PreTeXt writer.
[oscarlevin/pandoc-pretext](https://github.com/oscarlevin/pandoc-pretext) (GPL-2.0, 17 stars,
created 2019, **rewritten 2026-07-14/15** for the pandoc ≥3.0 custom-writer API) is the only
serious one, and its README positions it as the prototype for a case to make PreTeXt an official
pandoc format. Three files: `pretext.lua` (32 KB writer), `pretext-environments.lua`
(environment map), `pretext-latex-reader.lua` (a custom *reader* that pre-declares `\newtheorem`
for every known environment so pandoc gives amsthm environments full theorem treatment even when
the declaration lives in a class or style file).

Coverage per README: nested divisions with `<introduction>`/`<conclusion>` wrapping; theorem
environments → `<theorem>`/`<lemma>`/`<definition>` + `<statement>`, `<proof>` (run-in header
stripped, parenthetical recovered as `<title>`, QED tombstone removed); `\label` → `xml:id`;
`<m>`/`<md>` with one `<mrow>` per aligned line; the full pandoc table model including `<col>`
alignment/widths and colspan; `<figure><caption><image>` with alt → `<shortdescription>`;
`<program language>`; `<fn>`, `<url>`, `<xref>`, `<term>`/`<alert>`; raw `{=pretext}` passthrough;
`tikzpicture` → `<latex-image>` with the custom reader.

Its own **Limitations**, verbatim from the README:

> 1. Citations become bare `<xref>` elements pointing at the citation keys; you must create
>    `<biblio>` entries with matching `xml:id`s yourself.
> 1. Features with no PreTeXt equivalent (line breaks, horizontal rules, unrecognized divs,
>    non-PreTeXt raw blocks) are preserved as searchable XML comments […]
> 1. An image appearing inline mid-sentence is emitted where it stands, which is not valid
>    PreTeXt […]
> 1. Multi-paragraph footnotes are flattened to a single `<fn>`.

and it recommends the same loop we care about: "Validating the output against the PreTeXt schema
will point out anything that needs manual attention, e.g. `jing pretext.rng manual.ptx`."

User-reported failures: [#7](https://github.com/oscarlevin/pandoc-pretext/issues/7) `\intertext`
is ignored, so `align*` becomes an `<me>` wrapping an `aligned` environment the PreTeXt builder
cannot process; [#6](https://github.com/oscarlevin/pandoc-pretext/issues/6) numbers inside
`\text{}` in body text are silently eaten — *"Scientists are concerned that if the September
sea-ice falls between and square miles…"*.

Announcement and testing feedback: pretext-dev
["New LaTeX (and more) to PreTeXt conversions: testing requests"](https://groups.google.com/g/pretext-dev/c/uTObEQIzCu8)
(2026-07-15). Levin: *"converting pandoc's own manual from markdown to pretext results in a
document with zero schema violations; previously there were hundreds."* Beezer tested a 120K
CS-textbook chapter **from RTF** the next day: output validated clean, code and preformatted
material converted accurately, but chapter titles nested wrongly inside `<introduction>`,
paragraph content spuriously wrapped in `<term>`, code emitted line-by-line instead of as one
block, and images poorly recognized (empty `<figure>`s). His verdict: *"a very good head-start
towards a conversion"*, with AI post-processing suggested for the systematic errors.

Pandoc is not installed in this workspace, so no local trial of this route was run; the
assessment above is documentary.

### 2.5 Markdown / MyST

- **PreTeXt now has its own Markdown dialect.** `@pretextbook/remark-pretext` (v0.1.0,
  2026-08-21) defines *markdown-style PreTeXt* = CommonMark + GFM +
  [remark-directive](https://github.com/remarkjs/remark-directive), e.g.
  `::::theorem[Pythagorean Theorem]{#thm-pythagoras}` containing `:::proof`, plus a Python-style
  `Name:`+indent dialect. `@pretextbook/markdown-style-pretext` supplies LSP completions and lint
  for it in VS Code and the pretext.plus Monaco editor. Inline text directives `:name[…]` are
  **not implemented** — they become `<TODO>` placeholders.
- The reverse direction exists too: `@pretextbook/ptxast-util-to-mdast` (PreTeXt AST → mdast).
- **MyST-MD / Jupyter Book: no bridge found.** Searched GitHub and the web; nothing connects
  MyST or jupyter-book to PreTeXt. Treat as *probable but not exhaustively verified* — Google
  Groups is not full-text searchable with the tools available here.
- Jupyter notebooks: `virgilpierce/jpconvert`, linked from the PreTeXt website, is abandoned
  (last push 2019-11-19).

### 2.6 Word / .docx

No official workflow exists. What is actually available:

- **pandoc-pretext directly**: `pandoc file.docx -t pretext.lua -o out.ptx`. Levin's original
  2019 pitch was exactly this — *"Pandoc can read word files (among many others), even with math
  in them."* The repo ships `examples/testdoc.docx` → `examples/testdoc.ptx`. Fidelity inherits
  pandoc's docx reader: math (OMML → TeX) and heading structure survive; **theorem semantics
  cannot**, because Word has none — everything lands as headings and paragraphs unless the author
  fenced it first.
- **docx → LaTeX → `pretext import`** is possible in principle and documented nowhere; the
  plasTeX end of that chain no longer exists.
- Beezer's RTF experiment (§2.4) is the closest real word-processor data point: valid output,
  but empty figures, fragmented code, spurious `<term>`.
- **pretext.plus's import wizard does not accept `.docx`** — only `.tex`, `.md`, `.ptx` and
  archives.
- PreTeXt's own `validation-plus` stylesheet has a check specifically for word-processor
  provenance: *"A run of text contains a Unicode character for an en-dash […] Likely this was
  introduced in a conversion of source material authored in a word-processor."* Both trials below
  tripped it.

### 2.7 Community converters

| Project | What | State |
|---|---|---|
| [davidfarmer/SL3X](https://github.com/davidfarmer/SL3X) | "Structured LaTeX to XML", Python — the engine behind the "95% correct" UTMOST service, made public Jan 2026 | last commit 2026-05-17; README still says *"This project is in the process of converting from python2 to python3. Not yet ready to be used."* Commit messages ("hacks for Howell exercises") show per-book hand-tuning is the norm |
| [davidfarmer/PreprocessLaTeX](https://github.com/davidfarmer/PreprocessLaTeX) | Browser/JS LaTeX preprocessor | created 2026-04-09, last commit 2026-04-27; now the upstream of `@pretextbook/import`'s cleaning rules |
| [stephen-flood/python-latex-pretext](https://github.com/stephen-flood/python-latex-pretext) | "A simple script to convert a fragment of LaTeX to PreTeXt" | last push 2024-07-20, single author |
| [virgilpierce/jpconvert](https://github.com/virgilpierce/jpconvert) | Jupyter notebooks → PreTeXt; linked from pretextbook.org | abandoned 2019 |
| pretext.plus / PreTeXtPlus org | hosted front end for `@pretextbook/import` — paste/upload wizard, warnings review, preview, "Converted vs Native" choice | active (all repos pushed within the last month) |

`PreTeXtBook/pretext` `script/` contains **no** conversion tools (only `cssbuilder`, `dynsub`,
`jsbuilder`, `mbx`, `mjsre`, `utilities`, `webwork`), and there is no `latex2pretext`/`tex2ptx`
anywhere in the PreTeXtBook org. GitHub repos matching "pretext latex convert" are individual
books mid-conversion, not tooling.

### 2.8 LLM-assisted conversion in the community

This is the most decision-relevant finding for the plugin, because it is already happening.

- pretext-support, ["converting course materials into PreTeXt"](https://groups.google.com/g/pretext-support/c/G_FQX9pHKNY)
  (Mar–Apr 2026). David Austin asked for conversion tools. **Mark Fitch** recommended **Gemini**,
  reporting success even converting handwritten mathematics, with output that worked after
  pasting into a default document. **Ameya Kolarkar** endorsed Gemini section-by-section, noting
  it handles obsolete packages and *"converts figures and graphs to Prefigure format - this has
  been amazing and a massive time-saver!"* Levin pointed at pretext.plus. **Rob Beezer** praised
  the Gemini approach, said he uses **Claude Code** for PreTeXt development, and asked for
  prompt-engineering details.
- pretext-dev, ["LaTeX to PreTeXt"](https://groups.google.com/g/pretext-dev/c/cf-dhCnUDlM)
  (2026-01-20). Daniel Hodgins proposed a Python GUI wrapper because Farmer's route needs email
  and pandoc needs setup "that may intimidate non-technical users". Farmer: start with vanilla
  LaTeX; most packages are formatting, not semantics. **Beezer**: he is running AI experiments
  from PDF and LaTeX (motivated partly by braille transcription); results range from
  *"incredible"* to *"extremely frustrating"*; more investment → better outcomes; and preliminary
  AI runs on good LaTeX **approached the quality of Farmer's tool**.
- pretext-dev, ["Claude is a good PreTeXt reference tool"](https://groups.google.com/g/pretext-dev/c/DDctrrKoBDE)
  (2026-07). Brad Miller found Claude answered publication-file questions correctly; Beezer
  replied that this knowledge comes straight from the Guide and asked that effort go into
  improving the docs.
- pretext-dev, ["Claude skill for pretext authoring"](https://groups.google.com/g/pretext-dev/c/0D90duVchZg)
  (2026-08-28/29) — this project's own origin thread. Beezer: not aware of anyone having built
  one; he has accumulated "useful rules and hints" and is unsure they map onto the Skills format;
  feed in the **schema and example documents**, and use the **validation script so Claude can
  iterate to error-free output**. **No PreTeXt authoring or conversion Claude skill exists
  publicly.**

---

## 3. Trial A: `pretext import` (plasTeX) on a 137-line LaTeX article

Reproducible from `scratch/convert-trial/` (gitignored).

### 3.1 Setup

`sample.tex` (137 lines) is a hand-written `article` exercising exactly the constructs the ticket
names: `\section`/`\label`, `amsthm` `theorem`/`definition`/`example` on a shared counter, a
`proof`, two `\newcommand` macros, inline and display math, a numbered `equation` with `\eqref`,
an `itemize`, a `tabular` in a `table` float with `\caption`+`\label`, an `\includegraphics` in a
`figure` float, four `\autoref` cross-references, and a nested `enumerate` exercise list.

```
$ python3 -m venv venv249 && ./venv249/bin/pip install "pretext==2.49.1"
$ ./venv249/bin/pretext import sample.tex -o out249
…
WARNING: Using default renderer for autoref
WARNING: Using default renderer for hline
Conversion complete
```

Output: `main.ptx` + one file per section — plus a `.paux`, an `__init__.py` and a `__pycache__/`
left behind in the output directory.

### 3.2 What survived, what broke

**Survived cleanly.** Prose paragraphs; inline math (`$f\colon\R\to\R$` → `<m>f \colon \mathbb
{R}\to \mathbb {R}</m>`); `\emph` → `<em>`; sectioning and titles; `amsthm` environments →
`<theorem>`/`<definition>`/`<example>` with `<title>` from the bracket argument; `proof` →
`<proof>`; list nesting; `--` → en-dash; `<`/`>` escaped as `{\lt}`/`{\gt}` in math; unnumbered
display math → `<me>`; per-section file split with `xi:include` wiring.

**Broke — thirteen distinct classes**, in the order the toolchain surfaces them.

1. **`docinfo.ptx` is referenced but never written.** `main.ptx` opens with
   `<xi:include href="./docinfo.ptx" />`; no such file is produced, so `pretext validate` fails
   before any schema check runs.
2. **LaTeX labels become illegal `xml:id`s.** `\label{sec:limits}` → `xml:id="sec:limits"`, and
   `xml:id` must be an NCName: `error: xml:id : attribute value sec:limits is not an NCName`.
   Every `\label` using a `prefix:name` convention — i.e. essentially every real manuscript.
3. **Numbered display math is textually corrupted.**

   ```latex
   \begin{equation}
   \label{eq:eps-delta}
     |x - a| < \delta \quad\text{implies}\quad |f(x) - f(a)| < \eps
   \end{equation}
   ```
   ```xml
   <men id="eq:eps-delta">
       egin{equation}  \label{eq:eps-delta} |x - a| {\lt} \delta \quad \text{implies}\quad |f(x) - f(a)| {\lt} \varepsilon \end{equatio
   </men>
   ```

   `egin{equation}` … `\end{equatio` — the `[2:-2]` slice ate `\b` and `n}`. The `\label` is left
   as raw LaTeX inside the math, and the element uses `id=` rather than `xml:id=`.
4. **`\includegraphics` produces non-parsing XML.**

   ```xml
   <image source="images/img-0001.png" width="&images/img-0001.png-width;&px;">
   <shortdescription>
   \includegraphics[width=0.5\textwidth ]{epsilon-delta.png}
   </shortdescription>
   </image>
   ```

   `error: EntityRef: expecting ';'`. The real filename survives only inside `<shortdescription>`,
   where it displaces the alt text an accessible book needs, and `source` points at an
   `images/img-0001.png` that was never generated.
5. **`\autoref` is dropped silently.** `See \autoref{sec:evt} for the main consequence.` →
   `See  for the main consequence.` Four references vanished, leaving grammatical holes. `\eqref`
   fared better but emitted `<xref ref="section-sec-limits.ptx#eq:eps-delta">1</xref>` — a file
   path in `@ref` and the resolved number hard-coded as link text.
6. **Frontmatter uses a retired shape.** The importer emits
   `<frontmatter><titlepage><title/><author/></titlepage></frontmatter>` with `<abstract>`
   *outside* `<frontmatter>`. The 2.51.0 schema wants
   `frontmatter { MetaDataTitleOptional, Bibinfo, TitlePage, Abstract? }` with
   `titlepage { titlepage-items {empty} }` and `<author><personname>`. Six errors from this block
   alone.
7. **`<table>` is malformed** — caption emitted *after* `</tabular>` as `<caption>` (PreTeXt wants
   `<title>` first); `\hline` dropped; header row not distinguished; no `<col>` for the `lll`
   alignment; every `<cell>` wraps content in a `<p>`, which the schema rejects.
8. **`<figure>` children are in the wrong order** — image then caption; schema wants caption first.
9. **Lists are emitted as siblings of `<p>`**; PreTeXt requires `<ul>`/`<ol>` inside a `<p>`.
10. **Exercises lose their semantics entirely.** `\subsection*{Exercises}` became
    `<subsection xml:id="a0000000002"><title>Exercises</title><ol>…`. PreTeXt's whole exercise
    apparatus — `<exercises>`, `<exercise>`, `<statement>`, `<task>` for multi-part problems,
    `<hint>`/`<answer>`/`<solution>` — is untouched. And CLI 2.51.0 deprecates the very shape it
    produced: *"Ordered-lists as a way to structure multi-part exercises has been deprecated (use
    `<task>` instead)."*
11. **`\textbf` → `<term>`**, which in PreTeXt means "defining occurrence of a technical term" and
    drives the index. `\textbf{(Challenge.)}` became `<term>(Challenge.)</term>`; `<alert>` is
    correct.
12. **Author macros are expanded away.** `\R` and `\eps` flattened to `\mathbb {R}` and
    `\varepsilon` everywhere rather than carried into `<docinfo><macros>`. Nothing is lost, but
    the author's notation layer is, and every later edit is against expanded math.
13. **Deprecated math elements.** `pretext build` reports
    `PTX:DEPRECATE: (2026-05-11) the "me" and "men" elements have been deprecated. The
    replacement is an "md" with no "mrow" children`. `<men>` is now rejected outright; `<me>`
    still builds with a warning.

Also pervasive: spaces injected inside math (`\mathbb {R}`, `\delta `), and literal U+2013
en-dashes where `validation-plus` wants `<ndash/>`.

### 3.3 Validation

`pretext validate` (2.51.0, `--engine salve`; `jing` is not on this box) refused to run three
times before the source was even assemblable:

| Attempt | Result |
|---|---|
| as imported | `Error loading … docinfo.ptx: No such file or directory` |
| + hand-written `docinfo.ptx` | `xml:id : attribute value sec:limits is not an NCName` |
| + colons stripped from ids | `EntityRef: expecting ';'` (the `<image width>` entities) |
| + image width fixed | assembles; **20 schema errors + 2 validation-plus warnings** |

The 20: 6 frontmatter, 1 `<men>` placement, 4 table, 3 figure/image, 4 list placement, 2 "missing
required content". The 2 warnings: en-dashes.

### 3.4 Repair cost

Repaired by hand to a build-clean state in `scratch/convert-trial/proj-repair/`.

- **Untouched:** every prose paragraph, all inline math, the `<em>`/`<term>` runs, the
  theorem/definition/example/proof shells, `<ul>` content, section titles.
- **Rewritten:** all of `main.ptx`; the `<table>` block; the `<figure>` block; the whole exercises
  division; the numbered equation; and four `<xref>`s reconstructed **by reading the original
  `.tex`**, because the importer left no trace of them.
- Diff: **239 changed lines against 232 lines of importer output** — the salvage is the prose,
  not the markup.
- One repair resisted scripting: transforming the `<ol>` into `<exercises>` produced mismatched
  tags on the nested sub-parts, because the nesting has to become `<task>`, not another
  `<exercise>`. Judgement, not regex.

```
$ pretext validate --engine salve
PreTeXt source passed validation with no messages.
$ pretext build web
Success!  Built requested target(s) without errors.
```

---

## 4. Trial B: `@pretextbook/import` 0.9.0 on the same file

The package has no CLI, so the trial drove the library directly — a 12-line Node script
(`scratch/convert-trial/tsimport/run.mjs`), which is itself the finding that a plugin can wrap
this route in minutes:

```js
import { importProjectFromFiles, detectSourceFormat } from "@pretextbook/import";
const src = fs.readFileSync("../sample.tex", "utf8");
const res = importProjectFromFiles({ "sample.tex": src });   // → res.outputFiles, res.warnings
```

### 4.1 What it produced

`detectSourceFormat` → `latex`. Output: `source/main.ptx` (232 lines), `project.ptx`,
`publication/publication.ptx` — a complete project, plus three structured cleaning warnings:

```json
[ {"severity":"info","kind":"archaic","macro":"maketitle","occurrences":1,"action":"delete"},
  {"severity":"info","kind":"presentation","macro":"centering","occurrences":2,"action":"delete"},
  {"severity":"warning","kind":"presentation","category":"latex_fonts","macro":"textbf",
   "occurrences":2,"action":"anomaly","examples":["\\textbf","\\textbf"]} ]
```

**Wins over Trial A, on identical input.** Author macros preserved verbatim in
`<docinfo><macros>`; modern math elements (`<md xml:id="eq-eps-delta" number="yes">`) with the
equation body *intact*; `\label{eq:eps-delta}` → `xml:id="eq-eps-delta"` with the colon already
sanitized; `<xref ref="eq-eps-delta"/>` resolved correctly; the table emitted with `top="minor"`,
`bottom="minor"` rules and bare `<cell>` content; lists correctly nested inside `<p>`; `--` →
`<ndash/>` (the element, not the raw character); `<alert>` rather than `<term>` for `\textbf`;
`x^2` normalized to `x^{2}`; and the whole document is well-formed XML on the first pass.

**Unknown macros are flagged, not dropped.** Every `\autoref` became

```xml
<TODO type="unknown-macro"><!--todo: unknown macro "\autoref"--><c>\autoref</c></TODO>
```

`<TODO>` is not a schema element, so validation *fails on purpose* and points at every one. This
is the single biggest behavioural difference from plasTeX, which dropped them in silence — and it
is precisely the affordance an LLM repair pass wants.

**Remaining defects (real, not deliberate).**

1. `<author>` is placed inside `<docinfo>`; it belongs in `<frontmatter><bibinfo>`.
2. `<abstract>` sits directly in `<article>`, outside `<frontmatter>` (and no `<frontmatter>` is
   generated at all).
3. `<table>`'s `<title>` is emitted *after* `</tabular>`; the schema wants it first.
4. `<figure>` puts `<image>` before `<caption>`; the schema wants caption first.
5. `image width="0.5\textwidth"` — the LaTeX length is passed through raw; PreTeXt needs a
   percentage. (No `<shortdescription>` either, which `validation-plus` flags as an accessibility
   problem.)
6. **Exercises still lose their semantics** — `<subsection><title>Exercises</title><p><ol>…`,
   no `<exercises>`/`<exercise>`/`<task>`. Same gap as plasTeX. (The upstream engine gained `exam`
   class support in PR #156, but an `enumerate` under a heading called "Exercises" is not that.)
7. A division that has subdivisions must wrap its leading blocks in `<introduction>`; the
   importer does not, which is what the lone `<subsection> is not allowed here` error means.
8. `Bolzano--Weierstrass` became `Bolzano <ndash/> Weierstrass` — spaces injected around the
   dash.
9. **The generated `project.ptx` is rejected by CLI 2.51.0.** It writes child elements
   (`<target name="web"><format>html</format><source>…</source></target>`); the CLI wants
   attributes (`<target name="web" format="html" />`) and errors with
   `Either one of the targets or the root project element has an extra attribute it shouldn't:
   targets="html"` … then `'NoneType' object is not subscriptable`. The manifest had to be
   replaced with one from `pretext new` before anything could be validated. Worth reporting
   upstream.

### 4.2 Validation and repair cost

| Stage | Result |
|---|---|
| as imported, with the importer's own `project.ptx` | CLI cannot parse the manifest at all |
| + manifest from `pretext new` | **14 schema errors** — of which **6 are the deliberate `<TODO>` markers** |
| + frontmatter, table/figure order, image width, and the 6 `<xref>`s written in | 1 error (`<subsection> is not allowed here`) |
| + leading blocks wrapped in `<introduction>` | schema clean; 1 accessibility warning |
| + `<shortdescription>` | `PreTeXt source passed validation with no messages.` / `pretext build web` → `Success!` |

**Diff: 29 changed lines against 232 lines of output** — roughly **an eighth** of the plasTeX
repair, and none of it required reading the original `.tex` except to name the four `\autoref`
targets, which the `<TODO>` markers pointed straight at.

---

## 5. What the two trials say together

| | plasTeX (`pretext import`) | `@pretextbook/import` 0.9.0 |
|---|---|---|
| Well-formed XML on first pass | **no** (3 blocking failures) | yes |
| Schema errors after assembling | 20 | 14 (**6 of them intentional `<TODO>`s** → 8 real) |
| Lines changed to reach build-clean | 239 / 232 | **29 / 232** |
| Information lost with no trace | 4 `\autoref`s, `\hline`, author macros | none observed |
| Reached `pretext build` success | yes, after a rewrite | yes, after a light pass |

Three findings generalise beyond the choice of converter:

1. **Every converter gets the *shape* right and the *schema details* wrong** — child order
   (`<figure>` caption-first, `<table>` title-first), where a block may live (`<ul>` inside `<p>`;
   leading blocks inside `<introduction>` once a division has subdivisions), and which container a
   thing belongs in (`<author>` under `<bibinfo>`). These are exactly the rules the schema
   encodes and a human newcomer does not know — the highest-yield thing the plugin can hold.
2. **No converter produces exercise semantics.** `<exercises>` / `<exercise>` / `<task>` /
   `<hint>` / `<answer>` / `<solution>` is untouched by plasTeX, by `@pretextbook/import`, and by
   pandoc-pretext (whose README lists it under "future work"). For a textbook this is the single
   largest manual bill, and it needs judgement — my attempt to script the `<ol>`→`<exercises>`
   transform produced mismatched tags because nested parts must become `<task>`.
3. **Passing `pretext validate` does not mean the book builds.** After the plasTeX source
   validated cleanly, `pretext build web` still failed on
   `PTX:ERROR: width (50) should be given as a percentage (such as "40%"), or as the string
   "auto"` — an `@width` constraint no RELAX-NG schema can express. A conversion loop that stops
   at `validate` hands the author a document that does not build. **The loop must be
   `validate` → `build`**, which is a stronger form of Rob's ask on the origin thread.

---

## 6. Conclusion: which routes are viable, per input format

| Input | Recommended route | Expect to repair |
|---|---|---|
| **LaTeX** (clean, amsthm-based) | `@pretextbook/import` (npm library, or VS Code *Import Project*, or pretext.plus), then an LLM repair pass against the schema | frontmatter placement; `<figure>`/`<table>` child order; `@width` units; `<introduction>` wrapping; every `<TODO>` marker; **all** exercise semantics |
| **LaTeX** (messy, custom macro layer, unusual class) | same, but budget for the cleaning warnings; consider the UTMOST/Farmer service for a whole book with an open licence | as above plus per-macro decisions the cleaner flags as `anomaly` |
| **LaTeX** (whole open-licensed textbook, author has time) | Farmer / UTMOST service — still the highest-fidelity route and the only one the Guide endorses | "the last 5 percent", in Farmer's own framing |
| **Markdown** | `@pretextbook/import` (it routes Markdown through `remark-pretext`), or pandoc-pretext; markdown-style PreTeXt is the native dialect going forward | inline `:name[…]` directives (unimplemented → `<TODO>`); note the detection heuristic misreads Markdown containing `\section` as LaTeX |
| **Word / .docx / RTF / HTML** | **pandoc-pretext only** — `pandoc file.docx -t pretext.lua -o out.ptx`. `@pretextbook/import` does not accept `.docx` | all theorem/exercise semantics (Word has none to carry); figures come through poorly; expect en-dash and spurious-`<term>` cleanup |
| **Jupyter notebooks** | nothing maintained (`jpconvert` abandoned 2019) — convert to Markdown first | everything |
| **MyST** | no bridge exists | — |
| **PDF / handwritten / anything else** | LLM directly, which is what the community already does | everything, but community reports say this is now competitive |

**Implication for the conversion-ownership decision.** The plugin should **not** write a LaTeX
parser. Two of the three viable parsing routes are actively maintained by the same person
(Oscar Levin) who maintains the CLI, and both are already better than anything a plugin would
build. What neither of them has, and what this trial shows is where all the remaining cost sits:

- **No CLI entry point for the maintained route.** `@pretextbook/import` is a library behind a VS
  Code command and a web app. A 12-line Node wrapper gave a terminal agent full access to it —
  the plugin can own that thin shim (and should surface `res.warnings`, which is a
  ready-made review list).
- **The repair pass is the product.** 29 lines of schema-shaped fixes, six `<TODO>` markers to
  resolve, and an exercise division to author from scratch — all of it driven by
  `pretext validate` then `pretext build`, iterating to clean. That is precisely the loop Rob
  Beezer asked for on the origin thread, and precisely what faculty are currently doing by hand
  with Gemini in a browser.
- **The manifest incompatibility (§4.1 item 9) is a concrete upstream recommendation**, as is
  `\autoref` support in the unknown-macro table.
