# Prototype: the newcomer's first hour

Prototype artifact for ticket [#11](https://github.com/rickroesler/pretext-authoring/issues/11)
(wayfinder map [#1](https://github.com/rickroesler/pretext-authoring/issues/1)).
Status: **draft for human reaction** — nothing here is a decision.

This is a *scripted storyboard*, not a spec and not a transcript of a real session. It imagines
one hour with the plugin for three opening states:

- **Variant A** — "I have a LaTeX manuscript."
- **Variant B** — "I have a Markdown or Word draft."
- **Variant C** — "I have only an outline."

Every agent utterance is grounded in the research base; citations are bracketed and refer to
`docs/research/<file>.md` by section, e.g. `[feedback-loop §3.4]` =
`docs/research/feedback-loop-inventory.md`, section 3.4. Short names used:
`feedback-loop`, `conversion`, `gh-issues` (github-issues-survey), `support` (pretext-support-survey),
`dev-list` (pretext-dev-survey), `prior-art`, `review-criteria`, `licensing`.

**Markers.** `[AGENT ACTS]` = runs a command or edits files without asking. `[AGENT ASKS]` = stops
for a human decision. `[AGENT VALIDATES]` = runs a machine check and interprets its exit code.
`[BREAKS]` = the scripted failure and the agent's exact words.

**Baseline machine** (the one everything below was measured on): PreTeXt CLI 2.51.0 in a venv,
OpenJDK 21, node 24, `lxml`, `playwright`+chromium — but **no `jing` and no TeX**
[feedback-loop §1]. That is deliberately the awkward-but-common case.

---

## Variant A — "I have a LaTeX manuscript"

```
 1  USER: I have a linear algebra book in LaTeX (~40 files, amsthm, custom macros).
 2        I have never used PreTeXt. Can you get me to a book I can look at?
 3
 4  [AGENT ACTS] Preflight, before touching the manuscript. One pass, ~15 s:
 5        $ python3 --version; which -a pretext; pretext --version; pip show pretextbook
 6        $ java -version; which jing; node --version; which xelatex pdf2svg pdftoppm
 7        $ pretext support            # maintainers' own canonical env dump [gh-issues §5]
 8
 9  [AGENT SAYS] "Here is what your machine can and cannot do:
10        - pretext 2.51.0, Python 3.12, in a venv. Good — never pip install PreTeXt outside a
11          venv; the CLI shells out to sibling executables (node, playwright) by bare name.
12        - `pretextbook` (the old PyPI name) is NOT installed. Good — it ships a stale CLI
13          [gh-issues, Install/env, cli#1055].
14        - Java 21 present, but `jing` is NOT on PATH. Out of the box `pretext validate` will
15          exit **2 — 'validation could not be performed'** [feedback-loop §1; gh-issues §3.7].
16        - node 24 present, so the LaTeX importer and the `salve` validator are both available.
17        - No TeX. `pretext build print` will fail with 'cannot locate executable ... xelatex'
18          [feedback-loop §2, signal 12]."
19
20  [AGENT ASKS] "Validation is the gate for everything that follows, so pick one:
21        (a) install a `jing` shim (2 lines over jingtrang's jar) — I recommend this, because
22            jing enumerates *every element allowed at that point*, which is the single richest
23            repair hint in the toolchain; salve truncates the list at 12 [feedback-loop §3.4];
24        (b) `--engine salve` — no Java, npm self-installs, marginally faster, shorter hints;
25        (c) `--method server` — nothing to install, but 16 s per run and it uploads your
26            assembled source to a remote service [feedback-loop §1, §3.4]."
27        USER: (a).
28
29  [AGENT ACTS] Installs jingtrang, writes the shim, then **probes the validator** with a file
30        it knows is invalid.
31  [AGENT VALIDATES] "Probe passed: the deliberately-broken fragment came back as invalid. I check
32        this because a `jing` that exits 1 printing nothing makes `pretext validate` announce
33        'passed validation with no messages' and exit 0 on flagrantly invalid source — that is
34        exactly what jingtrang's own `pyjing` launcher does under setuptools >= 81
35        [feedback-loop §6.1]."
36
37  [AGENT SAYS] "About TeX: I can get you HTML today and it will be trustworthy. PDF I cannot
38        verify at all without a TeX install — no page counts, no overfull boxes, no float
39        placement [feedback-loop §9.10]. I'll smoke-test `pretext build tex` (which needs no TeX)
40        so at least the LaTeX *generation* is exercised, and flag print as unverified."
41  [AGENT ASKS] "Install TeX now (large, slow), or defer print until the web book is right?"
42        USER: defer.
43
44  [AGENT ACTS] Scaffold. Checks the directory is empty first — `pretext new` has no `--force`
45        and fails on a non-empty directory [gh-issues §5].
46        $ pretext new book -d linalg && cd linalg && pretext -t     # -> web, print
47
48  [AGENT ACTS] Convert. "`pretext import` was removed from the CLI on 2026-08-24; its
49        replacement `@pretextbook/import` is an npm *library* with no CLI, so I'm driving it
50        through a 12-line Node shim [conversion §0, §2.3, §4]."
51        $ node .../ptx-import.mjs ~/linalg-tex/main.tex -o ./
52        -> source/*.ptx, project.ptx, publication/publication.ptx, + 11 cleaning warnings
53  [AGENT SAYS] "Three of those warnings need your eye later — `\textbf` used 44 times is
54        reported as an `anomaly`, because in PreTeXt bold is either `<alert>` or `<term>` and
55        `<term>` drives the index [conversion §4.1]."
56  [AGENT ACTS] Replaces the importer's `project.ptx` with the one from `pretext new`: the
57        importer writes child elements where CLI 2.51.0 wants attributes, and the CLI dies with
58        'targets="html"' plus a raw `NoneType` traceback [conversion §4.1 item 9, §6.4].
59
60  [AGENT VALIDATES] $ pretext validate web --report-form terse --engine jing ; echo $?
61        -> exit 1; 61 lines in logs/main-validation.txt (file / xpath / line / check / message)
62  [AGENT SAYS] "Reading that report the way it has to be read [feedback-loop §3.3, §3.5, §10]:
63        - 22 `experimental` — advisory only, they do not affect the exit code; I'll report once.
63a       - 6 are deliberate: `<TODO type="unknown-macro">` markers the importer emits where it
63b         met `\autoref`. It fails validation *on purpose* and points at every one — the older
63c         importer dropped them in silence and left grammatical holes [conversion §4.1, §3.2].
64        - 19 `schema` errors, all the same four shapes every converter gets wrong: `<figure>`
65          wants its caption *before* the image, `<table>` wants `<title>` first, `<author>`
66          belongs in `<frontmatter><bibinfo>`, and a division with subdivisions must wrap its
67          leading blocks in `<introduction>` [conversion §5, finding 1].
68        - 14 `image-description-missing` — accessibility, and I'll treat them as TODOs, not
69          defects, until you tell me what the figures show [review-criteria A1, A4].
70        Note the line numbers are lines of the *assembled* book, never of your files; the `file`
71        column is real, the `path` column is an XPath [feedback-loop §3.2]."
72
73  [AGENT ACTS] Repairs, then re-validates after each batch. Four rounds, ~29 changed lines out
74        of 232 — the measured cost on the trial manuscript [conversion §4.2, §5].
75  [AGENT VALIDATES] -> exit 0, "passed validation with no messages."
76
77  [AGENT VALIDATES] $ pretext build web ; echo $?      # exit 0 is necessary, NOT sufficient
78  [BREAKS] "Build exit 0 — and I do not believe it yet. `pretext build` exits 0 on
79        schema-invalid source and ships invented elements' text straight to the reader
80        [feedback-loop §0, §5]. Grepping stdout and the newest ./logs/*.log for PTX:ERROR /
81        PTX:WARNING / PTX:DEPRECATE (errors are re-emitted at INFO level inside the summary
82        block, so grepping for ERROR alone misses them) [feedback-loop §4.1]:
83          PTX:ERROR: width (50) should be given as a percentage (such as \"40%\"), or \"auto\"
84        That is an `@width` unit constraint no RELAX-NG schema can express — your source
85        validates clean and still does not build. This is why the loop is validate *then* build
86        [conversion §5, finding 3]."  -> fixed, rebuild, clean.
87
88  [AGENT VALIDATES] Post-build asset check: every `<image source>` resolves to a file that
89        exists and is non-trivially sized. "Nothing in the toolchain checks this — a missing
90        image builds clean, exits 0 and produces a broken image [feedback-loop §5, §9.4]; and
91        asset-failure detection has regressed upstream before [gh-issues §3.1, cli#1117]."
92        -> 2 figures never came across: the importer does not rewrite image paths and flattens
93           basenames, so same-named images in different directories collide silently
94           [conversion §2.3, SPEC §7].
95
96  [AGENT VALIDATES] $ pretext view web --no-launch     # --no-launch, not --no-launch-browser
97        $ curl -s localhost:8128/output/web/sec-eigenvalues.html | (structure + math check)
98  [BREAKS] "Two `\(` delimiters survive into the rendered text of ch. 4. Bad LaTeX inside
99        `<m>`/`<md>` is checked by *nothing*: validate passes, build passes, MathJax renders no
100       error and no console message — the only trace is a surviving backslash in the body text
101       [feedback-loop §9.1, §5, §8]. Both are unbalanced braces in your `\newcommand` uses."
102
103 [AGENT ASKS] "One thing I will not do silently. Your `\subsection*{Exercises}` blocks came
104       through as `<ol>` inside a `<p>` — no converter produces exercise semantics, not this
105       one, not pandoc, not plasTeX [conversion §5, finding 2]. Turning them into
106       `<exercises>/<exercise>` with nested parts as `<task>` is judgement, not a regex — my
107       scripted attempt produced mismatched tags [conversion §3.4]. Also: PreTeXt deprecated
108       ordered-lists-as-exercise-parts in favour of `<task>` [conversion §3.2 item 10].
109       Shall I do chapter 1 as a worked example for you to approve before I do the other 11?"
110
111 [AGENT ASKS] "Ready to deploy. First: this repo has no `pretext-deploy.yml`, so you're on the
112       `pretext deploy` -> `gh-pages` branch path, not the GitHub Actions path. The two are easy
113       to cross-wire and I will not mix their advice [gh-issues, Deploy, cli#725/#644]."
114 [AGENT ACTS] $ pretext deploy --stage-only    # then inspect, then deploy, with a wall-clock
115       timeout — `pretext deploy` can hang forever on a bad credential prompt [cli#493]
116 [BREAKS] "Deploy reported success. I checked anyway and it did *not* publish: the CLI's
117       message does not distinguish 'pushed' from 'nothing to push' from 'push rejected', and
118       this exact silent success is one of the most-repeated questions on pretext-support
119       [support, recurring Q2; gh-issues Deploy cli#503]. `git log origin/gh-pages` is unchanged.
120       Fix: `pretext deploy --no-push` then `git push --force`, which is the community's own
121       remedy [support, Deploy row]."
122
123 END STATE (~55 min): validating, building, viewable web book; 11 chapters of exercises still
124       semantically flat; print target unverified; 14 images undescribed.
```

---

## Variant B — "I have a Markdown/Word draft"

```
 1  USER: I have chapters 1-3 as Word .docx and the rest as Markdown. Never used PreTeXt.
 2
 3  [AGENT ACTS] Same preflight as Variant A, plus one branch: `which pandoc`.
 4  [AGENT SAYS] "Your two formats go down two different roads, and only one of them is
 5        first-party [conversion §6]:
 6        - **Markdown** -> `@pretextbook/import` (it routes Markdown through `remark-pretext`).
 7        - **Word** -> `pandoc-pretext` ONLY. `@pretextbook/import` does not accept `.docx`;
 8          pretext.plus's wizard doesn't either [conversion §2.6].
 9        pandoc is not installed here, so the Word half is blocked until we install it."
10  [AGENT ASKS] "Install pandoc (+ the three-file `oscarlevin/pandoc-pretext` writer), or convert
11        the Word chapters to Markdown first and use one road for everything?"
12        USER: install pandoc.
13
14  [AGENT SAYS] "Two warnings before I start, both from measured behaviour:
15        - the importer's format detector favours LaTeX — a Markdown file containing `\section`
16          or `\begin{` *anywhere* is detected as LaTeX [conversion §2.3]. I'll grep your .md
17          files for those first and pass the format explicitly if I find any.
18        - Word carries no theorem or exercise semantics to carry across. Everything arrives as
19          headings and paragraphs. Beezer's own RTF test — the closest real data point — came
20          out schema-valid but with chapter titles nested wrongly inside `<introduction>`, body
21          text spuriously wrapped in `<term>`, code emitted line-by-line instead of as one
22          block, and figures arriving empty [conversion §2.4, §2.6]."
23
24  [AGENT ACTS] $ pretext new book -d stats-text && cd stats-text
25  [AGENT ACTS] $ pandoc ch01.docx -t pretext.lua -o source/ch01.ptx     (x3)
26              $ node .../ptx-import.mjs ch04.md ... -o ./                (x9, Markdown)
27              + replace the importer's project.ptx with the scaffolded one [conversion §4.1 #9]
28
29  [AGENT VALIDATES] $ pretext validate web --report-form terse --engine jing ; echo $?
30        -> exit 1; 240 messages. Filtered by the `file` column to what I just wrote
31        [feedback-loop §3.3, §10] and by the `check` column:
32  [AGENT SAYS] "The dominant class is word-processor residue, and PreTeXt has a check built
33        specifically for it: `unicode-em-dash`, `unicode-double-quotes`, `unicode-degree`,
34        plus zero-width spaces — each naming the element to use instead (`<mdash/>`, `<q>`,
35        `<degree/>`) [feedback-loop §5, §6; review-criteria A16]. 96 of those. They are
36        warnings, not errors, but they degrade braille and LaTeX fidelity, so I'll fix them
37        mechanically — it is a pure substitution, no judgement.
38        Then 41 real schema errors, mostly the `<introduction>`-wrapping and `<ul>`-inside-`<p>`
39        shapes [conversion §5 finding 1; SKILL rule 2: `p` is the bottleneck], and 18
40        `image-description-missing`.
41        And 12 `<TODO>` markers from the Markdown side: inline `:name[...]` directives are not
42        implemented in `remark-pretext` yet [conversion §2.5]."
43
44  [AGENT ASKS] "pandoc turned every `\cite` into a bare `<xref>` pointing at a citation key with
45        no `<biblio>` to land on, and flattened your multi-paragraph footnotes into one `<fn>`
46        — both are documented pandoc-pretext limitations, not repairable from the output alone
47        [conversion §2.4]. Do you have the .bib, or shall I stub the `<biblio>` entries?"
48
49  [AGENT ACTS] Repair rounds, re-validating each time.
50  [AGENT VALIDATES] -> exit 0.
51
52  [AGENT VALIDATES] $ pretext build web ; echo $?     -> 0, then grep the log anyway
53  [BREAKS] "Two things the exit code hid [feedback-loop §4.1, §5]:
54        - `PTX:DEPRECATE: the \"paragraph\" element is deprecated and any contained content
55          will **silently not appear**` — pandoc emitted three of them. Build stayed at 0.
56        - an `<xref>` to a section your Word file referenced by name but never labelled:
57          `a cross-reference uses references [sec-anova] that do not point to any target ...
58          Your output will contain placeholder text you will not want to distribute.` That one
59          is build-only — `validate` cannot see it, and it *does* fail the build [feedback-loop
60          §5, breakage table]. This is why I run both, in that order."
61
62 [AGENT VALIDATES] Asset sweep: 14 of 22 Word images are absent from `external/`. "Word's images
63       were embedded, not filed; pandoc wrote `<image source>` paths that point nowhere, and
64       nothing in PreTeXt checks that a referenced file exists [feedback-loop §9.4]. Also note
65       the Guide's own image-folder instructions were wrong for years — assets go under
66       `external/`, not a root `images/` [gh-issues, Docs gaps, pretext#1970]."
67
68 [AGENT VALIDATES] $ pretext view web --no-launch + curl + a playwright pass on the math-heavy
69       chapters (5.2 s, gives `mjx-container` counts and a screenshot) [feedback-loop §8].
70 [AGENT SAYS] "Word's equations came through OMML->TeX and mostly typeset. Three did not: they
71       contain `\mbox{}`, which is never sanctioned inside PreTeXt math — `\text{}` is the only
72       correct form [review-criteria C16]. Nothing flags this; I found it by reading."
73
74 [AGENT ASKS] "Your chapters 1-3 have exercises as bold-headed paragraphs. Same call as any
75       conversion: no tool produces `<exercises>/<exercise>/<task>` and it needs judgement
76       [conversion §5 finding 2]. Chapter 1 as a sample first?"
77
78 [AGENT ACTS] Deploy, same detection and same verification as Variant A: which deploy method,
79       `--stage-only` first, timeout, then confirm `origin/gh-pages` actually moved
80       [gh-issues Deploy; support Q2].
81 [AGENT SAYS] "Two preconditions `pretext deploy` never states and three separate support
82       threads hit: your project root must *be* the git root, and the remote must not have
83       diverged [support, Deploy row]. Both hold here."
84
85 END STATE (~60 min): building, viewable; 14 images to re-export by hand from Word; citations
86       stubbed; exercises flat; print unverified.
```

---

## Variant C — "I have only an outline"

```
 1  USER: I'm teaching intro discrete math in the spring. I have a 2-page outline and nothing
 2        else. I want to write it in PreTeXt.
 3
 4  [AGENT ACTS] Same preflight (jing route, TeX absent). Faster here — nothing to convert, so
 5        node is only needed for `--engine salve`, which we are not using.
 6  [AGENT SAYS] "TeX absent matters less for you than for someone with a manuscript: you're
 7        writing web-first and I can verify HTML properly. I'll flag print as unverified and we
 8        can install TeX before your first print proof [feedback-loop §9.10]."
 9
10 [AGENT ASKS] Five questions before any file is written, because each one is expensive to
11        change later:
12        1. "Book, article, or course? (`pretext new` scaffolds book / article / course / demo /
13           hello / slideshow.) [gh-issues §5]"
14        2. "Will this ever be hosted on Runestone or WeBWorK? If yes, we must decide
15           `docinfo/document-id` and `edition` *now* and never change them — the `label`
16           values derived from them key student database records, and changing one orphans
17           student progress [review-criteria P14, C9]."
18        3. "What licence? It determines whether the Farmer/UTMOST conversion service and much
19           of the community tooling is open to you, and it is the one thing that's awkward to
20           change after publication [licensing §1]."
21        4. "An `xml:id` naming convention — `sec-topic` or `topic-sec`? I'll apply it
22           uniformly. The Guide is emphatic that ids stay readable and carry no position
23           numbers, but it blesses no single scheme, so this is your house style, not a
24           community rule [review-criteria C3, C4, contested]."
25        5. "Do you want the experimental Runestone/interactive vocabulary available? It is in
26           the dev schema but not production, and it is explicitly 'among the most likely parts
27           of PreTeXt to change' [dev-list §(d)]. I'd say no for a first book."
28
29 [AGENT ACTS] $ pretext new book -d discrete && cd discrete && pretext build web && pretext view
30        web --no-launch
31 [AGENT SAYS] "Before we write a word: the template builds and serves. That is deliberate — the
32        Guide's own advice to newcomers is to get the sample book compiling *before* doing
33        anything else, so that when something breaks later you know it's your source
34        [conversion §2.2]. Note the build fetched Runestone services over the network; builds
35        are not fully offline even for a plain book [gh-issues §Demonstrated behaviours]."
36
37 [AGENT ACTS] Outline -> skeleton. One file per chapter, `xi:include`d from `main.ptx`; every
38        division gets `<title>` and `<xml:id>` on creation, not retroactively [review-criteria
39        C4, C10].
40 [AGENT SAYS] "I'm querying the schema for the content model of each container *before* writing
41        it rather than guessing and letting the validator correct me — that lookup is a 30 ms
42        offline query over `pretext.rng` [feedback-loop §7, §10 step 4]. It matters because the
43        answer is context-dependent: `<statement>` is defined three times in the grammar and
44        the WeBWorK one is a different language [feedback-loop §7]."
45
46 [AGENT VALIDATES] After every chapter file, not at the end:
47        $ pretext validate web --report-form terse --engine jing ; echo $?
48        "I keep this at exit 0 continuously. On a book that has never validated, messages
49        accumulate — PreTeXt's own sample book emits 196 messages and exits 1
50        [feedback-loop §3.5]. Starting clean is the one thing you can do now that you cannot
51        do later."
52
53 [BREAKS] First real error, and it is the one every newcomer hits:
54        `error: element "md" not allowed here; expected element "activity", "algorithm", ...`
55        [AGENT SAYS] "Display math and lists live *inside* a `<p>` — `<md>` as a direct child of
56        `<statement>` is invalid [SKILL rule 2]. Note what jing gave me: it enumerated every
57        element allowed at that point. That list is better than any static schema table and it
58        is why I picked jing over salve, which truncates it at 12 [feedback-loop §3.4]."
59
60 [AGENT ACTS] Writes ch. 1 prose + 6 exercises with `<statement>`/`<hint>`/`<answer>`/
61        `<solution>` and multi-part items as `<task>`, not nested `<ol>`.
62 [AGENT SAYS] "Structural rules I'm applying that the schema will *not* catch for you:
63        an `<exercise>` needs a `<statement>` wrapper as soon as it has a hint/answer/solution;
64        `<answer>` is the short result, `<solution>` the worked route; `<task>` nests three deep
65        and a fourth is an error [review-criteria P1, P3, P4]."
66
67 [AGENT VALIDATES] $ pretext validate ... -> exit 0.  $ pretext build web -> exit 0, log grepped.
68 [AGENT VALIDATES] Rendered check: curl the pages, then a playwright pass on the math.
69 [BREAKS] "One thing to fix in your macros: `\vec2` — MathJax silently fails to read a macro
70        whose name contains a digit, *and everything defined after it* [review-criteria C2].
71        Nothing in PreTeXt flags this; math errors reach the reader unchecked, end to end
72        [feedback-loop §9.1]."
73
74 [AGENT SAYS] "Two accessibility TODOs I'm recording rather than inventing: the three figures
75        you sketched need `<shortdescription>` (<=125 chars) or `decorative=\"yes\"` — never
76        neither [review-criteria A1, A3]. I won't write a description of a diagram I can't see."
77
78 [AGENT ASKS] "Deploy the two finished chapters now, so the pipeline is proved while the book is
79        small? I recommend yes — deploy problems are much easier to diagnose on a 2-chapter
80        book than on a finished one."
81 [AGENT ACTS] $ git init && ... && pretext deploy --stage-only    -> inspect -> deploy (timeout set)
82 [BREAKS] "Deploy said success and the site is live — but I verified rather than trusting it:
83        `git log origin/gh-pages` moved, and the served page returns 200 with your chapter
84        title. The CLI's success message cannot distinguish pushed / nothing-to-push / rejected
85        [support Q2; gh-issues Deploy], and if you later add a custom domain, GitHub will
86        recreate a `CNAME` file that silently blocks subsequent pushes [gh-issues, cli#503]."
87
88 END STATE (~50 min): 2 of 9 chapters drafted, continuously validating, built, deployed, live
89        URL in hand; conventions (ids, document-id, licence) fixed and recorded; 3 image
90        descriptions and the print target outstanding.
```

---

## Where the flow stalls

Each stall is a point where *a skill alone — prose the model reads — cannot carry the step*,
either because the work is deterministic and repetitive (wants a script/tool), or because it
must happen at a moment the model doesn't control (wants a hook/command), or because it needs
judgement over content (wants an agent). Portability caveat throughout: per the map and
[prior-art, Portability], commands/agents/hooks/MCP are Claude-only and must never be
load-bearing — every stall below needs a skill-level fallback path.

| # | Stall | Why a skill alone is insufficient | Component implied |
|---|---|---|---|
| 1 | **First-run `pretext validate` fails (exit 2, no jing)** and the fix is a multi-step install with a hand-written shim [feedback-loop §1] | Deterministic, OS-branching, and must run *before* anything else; prose instructions will be re-derived every session | **Command** `/pretext:doctor` + a script (Python/venv, CLI version, `pretextbook` vs `pretext`, java/jing/node/TeX/pdf2svg, pipx isolation [support], Windows lxml/cp1252 branch [support §Windows]) |
| 2 | **A broken jing reports a clean bill of health** [feedback-loop §6.1] | A silent false-negative that invalidates the whole loop; needs a probe run once per session, not advice | **Script**, invoked by the doctor and by any validate wrapper |
| 3 | **`@pretextbook/import` has no CLI** [conversion §2.3, §4] | The plugin must *ship* the shim; a skill can only describe it | **Tool/script** (Node shim, surfacing `res.warnings` as a review list) |
| 4 | **The importer's `project.ptx` is rejected by CLI 2.51.0** [conversion §4.1 #9] | Mechanical fixup on every single import | **Script** (+ an upstream bug report) |
| 5 | **Validate output needs post-processing**: whole-book scope, `experimental` noise, file+XPath but never file+line in the author's own file [feedback-loop §3.2, §3.5, §9.6, §9.11] | Parsing tab-separated reports, filtering by changed files, mapping XPath→cursor position — repetitive and exact | **Tool** (validate wrapper: run, parse, filter by `file`/`check`, resolve to source line) |
| 6 | **LaTeX inside `<m>`/`<md>` is unchecked end to end** [feedback-loop §9.1] | The largest hole in the toolchain for a STEM book; needs an actual parser (MathJax/KaTeX parse-only API + `docinfo/macros`), not a rule | **Tool** (math checker) |
| 7 | **`publication.ptx` is never validated by the CLI** [feedback-loop §2 signal 7, §9.3] | A typo'd publisher switch is indistinguishable from one that works; the schema exists but is unwired (and needs its `epub/cover` false positive suppressed) | **Script** (jing over `publication-schema.rng`) |
| 8 | **Referenced files are never checked to exist; asset generation fails silently** [feedback-loop §9.4; gh-issues §3.1] | Post-build file-existence/size sweep; exit code 0 proves nothing | **Script**, run after every build |
| 9 | **Exercise semantics** — no converter produces `<exercises>/<exercise>/<task>`, and scripting the `<ol>` transform produces mismatched tags [conversion §3.4, §5 finding 2] | Genuinely requires judgement over content; the single largest manual bill in any conversion | **Agent** (subagent per chapter, with a human-approved worked example first) |
| 10 | **Rendered-output inspection** — serve, curl, playwright, scan for surviving backslashes [feedback-loop §8] | Multi-process lifecycle (`view --no-launch`, `--stop-server`) plus a vision/text pass | **Command** + tool, and an **agent** for the reading pass |
| 11 | **Deploy: silent success, two cross-wireable methods, hangs on bad credentials** [support Q2; gh-issues Deploy cli#725/#493/#503] | Requires method detection, a wall-clock timeout, and post-hoc verification the CLI does not do | **Command** wrapping deploy with detect → stage → verify |
| 12 | **Validate-on-save** — the loop only works if it actually runs after each edit [feedback-loop §10; prior-art pattern 1] | The model cannot guarantee it runs it; 0.6–1.0 s is fast enough for a save hook | **Hook** (Claude-only; must degrade to "the skill tells you to run it") |
| 13 | **Review layer** — image descriptions matching intent, informative titles, `<preface>` misuse, notation consistency [review-criteria, LLM-judgeable tier] | Not mechanical, not in the toolchain at all; needs reading | **Agent** (review pass), distinct from the validate loop |
| 14 | **"Where does this config attribute go?"** (project vs publication vs docinfo) — the single most repeated support question [support §Recurring Q1] | A lookup, answerable from the schema + Guide, needed constantly and mid-flow | **Skill** (this one genuinely is a skill) + possibly the schema-query script [feedback-loop §7] |

Read together, stalls 1–2 say the newcomer flow **begins** with something a skill cannot do; 5–8
say the validate→build loop needs a machine layer under it; 9 and 13 say the value-add is
judgement, which is agent-shaped; 11–12 say the endpoints (save, deploy) need harness hooks. A
skill-only plugin can deliver Variant C tolerably and Variants A/B badly.

---

## Questions for the human reacting to this storyboard

1. **Is the preflight the right first move, or does it front-load friction?** The storyboard
   spends 3–5 minutes on environment before the author sees anything. The alternative is to
   scaffold and build first (proving the toolchain empirically, per the Guide's own advice
   [conversion §2.2]) and only diagnose when something fails. Which does a newcomer tolerate?
2. **Which validation route should the plugin recommend by default?** Real jing gives the
   "expected element" enumeration that makes repair cheap, but needs a hand-built shim; salve
   needs no Java but truncates hints; server needs nothing but takes 16 s and uploads the
   author's source [feedback-loop §1, §3.4]. Should the plugin ever *install* anything on the
   author's machine, or only tell them how?
3. **How much should the agent do without asking?** The storyboard has it acting freely on
   mechanical repairs (Unicode substitutions, child order, `<introduction>` wrapping) and asking
   on anything with judgement in it (exercise semantics, image descriptions, bibliography
   stubs). Is that line in the right place — and is the "chapter 1 as a worked example first"
   gate worth the round trip?
4. **Is TeX-absent the right default assumption?** It means print is unverified for the whole
   first hour [feedback-loop §9.10]. Should the plugin push harder for a TeX install up front,
   ship a container/Codespace recipe, or accept web-first and say so loudly?
5. **Should the first hour end at deploy at all?** Variant C deploys a 2-chapter skeleton to
   prove the pipeline early; A and B deploy a nearly-finished book. Is "deploy early on purpose"
   good advice, or does a half-empty public URL do harm?
6. **Exercise semantics is the biggest single cost and no tool touches it** [conversion §5
   finding 2]. Is a per-chapter subagent the right shape, or should the plugin decline the job
   and instead teach the author the `<exercises>/<exercise>/<task>` model?
7. **Which of the 14 stalls are in the v0.1 cut?** Stalls 1, 3, 5, 8 look load-bearing for any
   newcomer; 6, 7, 13 are higher-value but bigger builds. What is the minimum set that makes
   the first hour honest?
8. **Do the three variants cover the real population?** Rob's own community evidence says
   faculty are already converting by pasting into Gemini section-by-section, and that the
   UTMOST/Farmer service is the endorsed route for a whole open-licensed book [conversion §2.8,
   §2.2]. Should there be a Variant D — "I have a whole book and should not be doing this in an
   agent at all" — whose correct answer is a referral?
