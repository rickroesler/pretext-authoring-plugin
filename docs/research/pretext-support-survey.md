# pretext-support survey: what problems do authors actually bring?

**Research ticket:** [#2](https://github.com/rickroesler/pretext-authoring/issues/2)
**Date:** 2026-08-28
**Source:** [pretext-support Google group](https://groups.google.com/g/pretext-support) (~1,876 threads total)

## Methodology and bias

Google Groups does not expose a paginated thread listing or an API usable from this
environment, so the sample was built from:

1. The group's front page (the ~28 most recent threads, roughly the second half of
   August 2026).
2. ~30 `search?q=<term>` queries against the group, one per topic word (`webwork`,
   `latex`, `pdf`, `deploy`, `image`, `tikz`, `install`, `validate`, `runestone`, `css`,
   `error`, `build failed`, `project.ptx`, `exercise`, `mathjax`, `git github`, `table`,
   `new to pretext`), each returning up to 30 threads ranked by Google's relevance/recency
   blend, not a strict date order.
3. ~24 individual threads opened in full to extract verbatim problem statements and
   resolutions (used for the representative examples and FAQ sections below).

This produced **284 unique threads** (deduplicated by thread ID) — comfortably over the
ticket's 250-thread floor. Dates on the threads returned range from October 2019 through
August 2026, but the search-term method is heavily weighted toward recent activity
(Google ranks recent+relevant highly for a generic term), so the **effective sample is
dominated by roughly the last 12–18 months**, which matches the ticket's intent.

**Known biases, stated honestly:**

- **Keyword sampling, not random sampling.** A thread that never uses any of the 18
  search words (unlikely for a technical support group, but possible for terse or
  off-topic threads) is under-represented.
- **Cross-category overlap.** Many threads legitimately touch two or three categories
  (e.g. a WeBWorK-generated TikZ image that fails only in the `print` target touches
  WeBWorK, images, *and* PDF-LaTeX). Each thread was filed under what looked like its
  *dominant* topic; the category counts below should be read as "roughly this many,"
  not an audited census.
- **Google's AI-generated result summaries** (via WebFetch) were used to gist ~250 of
  the 284 threads rather than opening each one; only ~24 were opened and read in full
  for verbatim quotes. Gists for the other ~260 are reliable for categorization but not
  guaranteed word-for-word accurate — treat the one-line gists in the table below as
  paraphrase, and the quoted material in the FAQ/representative sections (which came
  from opened threads) as closer to verbatim.
- **The front page and several search results skew toward power users** (Rob Beezer,
  Oscar Levin, David Austin, Sean Fitzpatrick, Alex Jordan, Federico Galetto, Charles M,
  Mitch Keller) who are also frequent *answerers* — i.e. the sample captures the whole
  community's traffic, not just newcomers, which is appropriate since the plugin must
  serve both personas per the map.

## Category table

Counts are per-thread, out of the 284-thread sample.

| Category | Count | Representative threads |
|---|---|---|
| **Images & diagrams** (PreFigure, TikZ, Asymptote, Sage plots, SVG) | ~44 | [Prefigure Error that Module not found, but prefigure is installed](https://groups.google.com/g/pretext-support/c/Y8pCp68rICM) — pipx's per-tool venv isolation hides `prefig` from `pretext`; fix is `pipx install pretext[all]`. • [Tiff v jpg?](https://groups.google.com/g/pretext-support/c/EaTdTqqgG7w) — which raster format to use for print vs. web. • [future-proofing graphics authoring for maximal accessibility](https://groups.google.com/g/pretext-support/c/jaez1QFmbEE) — TikZ vs. PreFigure vs. Asymptote for long-term accessible output. |
| **Markup & schema** (element usage, validation, xml:id, numbering, xrefs) | ~34 | [Finding pretext schema](https://groups.google.com/g/pretext-support/c/TX1eIrTy4L0) — author didn't know `~/.ptx` holds versioned schema copies for editor validation. • [jing not following include](https://groups.google.com/g/pretext-support/c/1CVEo3yuQJw) — the `jing` RELAX NG validator doesn't resolve `xi:include`, so modular projects validate the wrong (incomplete) document. • [disallowed unicode in math mode](https://groups.google.com/g/pretext-support/c/H6a969qCZeM) — the anti-smart-quote validation rule false-positives inside `<m>`/`<me>`/`<men>`. |
| **Exercises & WeBWorK** (task/exercise structure, PG code, seeding) | ~33 | [Answers in a scaffolded WeBWorK problem are getting mixed up](https://groups.google.com/g/pretext-support/c/h51rwbRqokw) — 80% of the time the wrong sub-answer matched; root cause was a variable-mapping bug in the author's own PG code, found by re-running the same random seed directly on a WeBWorK server. • [WeBWorK from OPL with CLI 2.30.1](https://groups.google.com/g/pretext-support/c/XlHwGD5xQbU) — Open Problem Library problems stopped activating after a CLI upgrade. • [homework in PreTeXt](https://groups.google.com/g/pretext-support/c/Ey-fNFkskcs) — schema doesn't allow the nested exercise structure the author wanted. |
| **PDF-LaTeX** (page breaks, TOC, TeX capacity, print-only quirks) | ~24 | [Problem resolved! TeX capacity exceeded](https://groups.google.com/g/pretext-support/c/6UhaXNXBIzs) — an `<idx>` inside a `<title>` compiled fine to HTML but blew up LaTeX's bookmark-writing; Rob Beezer: schema validation would have flagged it. • [Reducing the number of pages](https://groups.google.com/g/pretext-support/c/hLj_tLhbyQY) — LaTeX `book`-class page-break assumptions don't transfer to PreTeXt's model. • [Re: Subsubsections and TeX capacity](https://groups.google.com/g/pretext-support/c/9YnrhjPzR3A) — same failure mode, different structural trigger. |
| **HTML-CSS** (themes, fonts, dark mode, layout, browser quirks) | ~28 | [How do we modify themes (default light mode or no dark mode)?](https://groups.google.com/g/pretext-support/c/-M2cI7L9L94) — the `@provide-dark-mode` attribute lives in `publication.ptx`'s `<css>` element, not somewhere authors guessed. • [Epubs built by PreText yields errors when submitted to Google Books](https://groups.google.com/g/pretext-support/c/8qvWd5Tdulk) — CSS `:has()` selectors in the generated EPUB aren't supported by Google Books' renderer. • [Modifying the HTML header](https://groups.google.com/g/pretext-support/c/MjFSEKqWPJQ) — custom `<head>` content requires a themed XSL override, not obvious from docs. |
| **Build errors** (CLI crashes, cache/tempdir/permission bugs, encoding) | ~25 | [Fatal (but not really?) charmap error](https://groups.google.com/g/pretext-support/c/tENXgLjBCTw) — a PDF that compiled successfully was reported as a fatal error because the CLI's log-reading step assumed `cp1252` on Windows and choked on a non-ASCII byte. • [esbuild module not found](https://groups.google.com/g/pretext-support/c/a2E-IPZEJ_4) — a Makefile-based build path skips an npm install step the CLI path performs automatically. • [permission error with "pretext view"](https://groups.google.com/g/pretext-support/c/SFWhcKrhVAg) — build succeeds, viewing fails, on a cloned/permission-restricted checkout. |
| **Runestone** (platform-specific rendering/grading behavior) | ~20 | [Buggy behaviour in Runestone Parsons problems](https://groups.google.com/g/pretext-support/c/yTIZZ68KMoY) — a JS-side `undefined.indent` error traced to a stale problem-label/database mismatch, not a PreTeXt authoring mistake. • [SageMath output blocked by CORS policy](https://groups.google.com/g/pretext-support/c/VZchAyD6fJs) — browser cross-origin policy blocks a Sage cell's output on Runestone. • [Math not rendering in Runestone peer instruction](https://groups.google.com/g/pretext-support/c/4acgG9UUW7Y) — a math-delimiter incompatibility specific to the assignment-builder view. |
| **Deploy** (GitHub Pages, custom domains, ssh, silent failures) | ~16 | [pretext deploy fails](https://groups.google.com/g/pretext-support/c/o5ebMSA7xaU) — CLI reports success but the push to `gh-pages` is silently rejected because the remote branch advanced; fix is `pretext deploy --no-push` + manual `git push --force`. • [no deploy, no error message](https://groups.google.com/g/pretext-support/c/scVSrdFJ0SQ) — same silent-failure shape, different cause. • [PreTeXt project in subdirectory of git repository](https://groups.google.com/g/pretext-support/c/ids6igmYtX0) — `pretext deploy` assumes the project root *is* the git root; author worked around it with a symlinked `.git`. |
| **Math** (MathJax rendering, LaTeX macros in math mode) | ~18 | [MathJax can't handle a `\renewcommand` with too many replacements?](https://groups.google.com/g/pretext-support/c/hcOOjCFGgN8) — redefining `\lim` in terms of itself creates infinite MathJax macro recursion; classic LaTeX gotcha authors don't expect in a math-rendering context. • [Unexpected behavior of `\text{TEXT}` inside `\frac` and `\sqrt`](https://groups.google.com/g/pretext-support/c/ZQFM-pJGI2w/m/GcKHOLsLAwAJ) — resolved by a Python3 + PreTeXt version bump (stale toolchain, not markup). |
| **Install** (Node/Python/Sage toolchain setup) | ~14 | [PreTeXt upgrade on Windows failing -- lxml error](https://groups.google.com/g/pretext-support/c/F_okFhdpC3o) — `pip` tries to compile `lxml` from source on Windows and can't find libxml2 headers even with Visual Studio Build Tools installed; fix is `pip install lxml --prefer-binary`. • [Node Trouble](https://groups.google.com/g/pretext-support/c/luWNjP5L8rM) — generic Node.js setup failure blocking custom-theme builds. • [Can't install Sage/GitHub codespaces](https://groups.google.com/g/pretext-support/c/sOToSpWLlfg) — Sage install breaks after a PreTeXt version bump in a Codespace. |
| **Project setup** (project.ptx/publication.ptx config, scaffolding) | ~14 | [Turning on author.tools in project.ptx with the CLI](https://groups.google.com/g/pretext-support/c/bCTcNUlfpPI) — where to place a config flag the docs don't show an example of. • [pretext-cli configuration to generate braille](https://groups.google.com/g/pretext-support/c/OXmpKpfw_n8) — braille target setup isn't discoverable from the main Guide flow. • [Favicons in PreTeXt](https://groups.google.com/g/pretext-support/c/xyM0yQfhVvY) — same "where does this config attribute go" pattern. |
| **Other** (tooling requests, meta, translation, hosting) | ~13 | [PreTeXt-tools feature request](https://groups.google.com/g/pretext-support/c/JU4LUNwz77s) — VS Code extension "New file..." template request. • [Major GitHub problems](https://groups.google.com/g/pretext-support/c/cd8rND_fK0c) — GitHub-wide outage, not a PreTeXt issue at all. • [Translation to pt-BR](https://groups.google.com/g/pretext-support/c/ejSAZQoZ61s) — localization workflow question. |

Total: ~283 categorized (rounding; a handful of threads plausibly belong to a second
category and were counted once under their dominant one).

## Recurring questions (candidate FAQ content)

These patterns repeated across multiple, independently-worded threads closely enough
that they read as the same underlying question asked by different authors:

1. **"Where in `project.ptx`/`publication.ptx` do I set `<attribute X>`?"** — asked
   nearly verbatim for dark mode (`@provide-dark-mode`), deprecation warnings
   (`author.deprecations.all`), authoring tools (`author.tools`), braille generation,
   favicons, and custom XSL paths. Same shape every time: the attribute exists and is
   documented *somewhere*, but the author couldn't find *which file, which element,
   which nesting level* it belongs under.
2. **"My `pretext deploy` says it succeeded but nothing changed on the live site."**
   Asked at least three times with different root causes (diverged `gh-pages` branch,
   project not at git repo root, silent non-zero exit swallowed). The CLI's success
   message doesn't distinguish "pushed" from "nothing to push" from "push rejected."
3. **"How do I hide/show X depending on the output (print vs. web vs. solutions
   manual)?"** — asked for proofs, WeBWorK solutions, WeBWorK tables in static output,
   and hidden Runestone content. Same underlying mechanism (`visible`/`exercise`
   attributes and publication-file variants) re-explained each time from scratch.
4. **"Prefigure/Sage/Node says it's not installed, but I did install it."** — recurring
   pipx/virtualenv isolation confusion: the tool is installed but not on the path the
   *other* tool's subprocess call can see.
5. **"TeX capacity exceeded" / infinite LaTeX compile loop** — recurring, and in every
   opened instance traced back to a structural mistake (an `<idx>` or nested
   `<subsubsection>` in a spot the schema technically allows but LaTeX's bookmark/TOC
   writer chokes on) that built fine to HTML and only failed in the `print`/LaTeX
   target.
6. **"jing doesn't validate my `xi:include`-based modular project correctly."** —
   asked at least twice, same root cause (the validator resolves against the un-merged
   entry file).
7. **Windows-specific breakage** — recurring across unrelated symptoms (charmap/cp1252
   log-reading, lxml compile-from-source, Git Bash needing admin rights, Node path
   resolution, temp-directory permission errors) — Windows authors hit a
   disproportionate share of environment friction that macOS/Linux authors don't.
8. **"How do I get plain external images / TikZ / Asymptote to keep working long term
   without breaking accessibility?"** — a recurring anxiety thread, not a bug report:
   authors ask before they build, wanting a recommendation between PreFigure, TikZ, and
   raster fallbacks.

## What the answers reveal about knowledge the Guide doesn't make findable

- **The `~/.ptx` cache directory** holding versioned schema copies (used by the VS Code
  extension) is apparently not documented anywhere an author would think to look; it
  had to be explained fresh in the "Finding pretext schema" thread by a maintainer.
- **`pipx`'s per-package venv isolation** is a recurring silent trap for anyone who
  installs `pretext` and `prefigure`/`sage` separately via pipx — the fix
  (`pipx install pretext[all]`) is a single flag, but nothing in the failure message
  points at it.
- **The `visible`/output-variant mechanism** (hiding proofs in print, WeBWorK solutions
  in a solutions-manual build, static fallbacks for interactive elements) is one
  underlying feature that gets re-derived by different authors for different surface
  problems every time, suggesting the Guide's coverage of it isn't indexed/searchable
  under the terms authors actually use ("hide X in Y").
- **HTML tolerates structural choices LaTeX rejects.** Rob Beezer names this directly in
  the TeX-capacity thread as a common misconception new authors carry over from
  web-first thinking. The Guide apparently states the *rules* (what the schema allows)
  but doesn't call out *this specific cross-target asymmetry* as a named gotcha, so
  authors discover it only after a failed print build.
- **`pretext deploy`'s git assumptions** (project root == git root, remote not
  diverged) are implicit and undocumented; three independent threads hit variations of
  the same unstated precondition.
- **Windows toolchain guidance is thin or scattered.** Every Windows-specific failure in
  the sample required a maintainer to diagnose from a stack trace rather than pointing
  at an existing troubleshooting page, suggesting there isn't a consolidated
  "PreTeXt on Windows" checklist in the Guide.

## What a validating agent would have caught before the author posted

- **The entire `<idx>`-in-`<title>` / deep-`<subsubsection>` "TeX capacity exceeded"
  family.** Rob Beezer said so explicitly: "regular schema validation" would have
  flagged the structural misuse before a print build was ever attempted. This is the
  single clearest case in the sample of `pretext validate` (per Rob's own ask in the
  map's Notes) preventing a support thread outright.
- **`xml:base not allowed`, `kbd tag`, `disallowed unicode in math mode`, and the dozen
  other `validate`-bucket threads** where the fix was "the schema doesn't permit this
  attribute/element here" — every one of these is a `pretext validate` finding, not a
  build-time surprise, if the author had run validation (or had it run automatically)
  before building.
- **`jing not following include` / modular-project validation gaps** — an agent that
  knows to validate the *merged* (post-`xi:include`) document, not the entry file,
  would have caught what the raw `jing` invocation missed.
- **The pipx/venv "module not found but installed" family** — a preflight check
  ("can `pretext`'s subprocess actually import `prefig`/find `sage` on PATH?") run
  before the build starts would surface this immediately instead of after a cryptic
  runtime error.
- **The Windows lxml/charmap/permission cluster** — none of these are markup mistakes;
  they're environment mismatches an agent with a "known Windows issues" checklist
  (prefer-binary lxml, UTF-8 log reading, admin-rights Git Bash) could flag proactively
  during setup rather than have the author discover them mid-build.
- **`pretext deploy`'s silent-failure family is *not* fully agent-catchable** without
  also fixing the CLI: an agent could check `git status`/`git log` for a diverged
  remote branch before deploying and warn, but the CLI's own success/failure signal is
  the deeper problem (David Austin's own suggestion in the thread was that the CLI
  itself should detect and report this, not that authors need better tooling around
  it).
- **What an agent would *not* have caught:** the WeBWorK PG-code logic bugs (wrong
  variable mapping in an 80%-of-the-time-wrong scaffolded problem) and the Runestone
  platform-side JS bugs (Parsons `undefined.indent`, CORS-blocked Sage output). These
  are genuine logic/platform bugs downstream of valid, schema-conformant PreTeXt
  source — no amount of pre-build validation of the *author's* markup would surface
  them; they need runtime testing against a real WeBWorK/Runestone instance, which is a
  different (and out-of-scope-for-a-writing-plugin) feedback loop.
