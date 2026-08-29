# GitHub issues survey: PreTeXtBook/pretext and PreTeXtBook/pretext-cli

Research ticket [#4](../../../../issues/4) (wayfinder map [#1](../../../../issues/1)).
Snapshot date: 2026-08-28. Sources: `gh issue list` against both repos (`--state all --limit 300`
for the categorised sample, `--state open --limit 500` for label distributions), plus a live
CLI session (`pretext` 2.51.0 in `.venv/`) and a build of `scratch/demo-book`.

## 1. What the two trackers look like

| | pretext (core: schema, XSL (Extensible Stylesheet Language), `pretext/pretext` script) | pretext-cli (Python CLI wrapper) |
|---|---|---|
| Open issues (label-distribution pull) | 359 | 33 |
| Sample pulled (all-state, newest 300) | 134 open / 166 closed | 31 open / 269 closed |
| Oldest issue in sample | 2022-07-19 | 2022-03-07 |
| Top open labels | enhancement (43), feature request (35), bug (20), contributor project (20), code cleanup (15), z-sage (14), braille (13), z-webwork (8), z-index (8), accessible (8), CSS (7) | almost unlabelled (one issue tagged "Useful Error Messages"); this repo triages mostly by title/comments, not labels |
| Milestones | mostly stale (last real one "1.8", still open, no due date); a 2022 "Far-Distant Future" bucket | same pattern — milestones are not an active roadmap signal in either repo |

**Implication for the plugin design:** don't build tooling that reads GitHub milestones/labels as
a roadmap source — maintainers plan primarily in issue threads, PR descriptions, and the
pretext-dev/pretext-support Google Groups. The map's own "Notes" (Rob's asks, the mailing-list
thread) are a better roadmap signal than repo metadata.

## 2. Categorised issue table (representative sample)

Full numeric counts by author-facing category (keyword pass over the 300+300 sample; an issue can
match more than one category):

| Category | pretext | pretext-cli |
|---|---:|---:|
| images (image/figure/PreFigure/asymptote assets) | 73 | 48 |
| pdf/LaTeX | 69 | 40 |
| html/css/knowl/MathJax | 56 | 8 |
| cli-usage (commands, tracebacks, error messages) | 5 | 70 |
| install/dependency | 10 | 51 |
| deploy (gh-pages, Actions, custom domain) | 2 | 44 |
| codespaces/devcontainer | 3 | 33 |
| runestone | 31 | 18 |
| webwork | 12 | 15 |
| validation | 7 | 12 |
| build (xslt/saxon/lxml plumbing) | 4 | 9 |
| docs-gap | 10 | 7 |
| import/conversion | 16 | 7 |

Reading the split: **pretext** (core) issues are dominated by *rendering correctness* across
targets — a construct works in HTML but not PDF/EPUB (Electronic Publication)/slides, or a WeBWorK/Runestone
interactive misbehaves in one output format. **pretext-cli** issues are dominated by *toolchain
friction* — install, CLI ergonomics, deploy, and Codespaces/devcontainer setup. This maps cleanly
onto the map's two tracks: newcomer pain is concentrated in pretext-cli's install/CLI/deploy/
Codespaces categories; advanced-author pain is concentrated in pretext's cross-target rendering
categories.

Below is a curated table of the issues most load-bearing for plugin design (number, title, state,
gist, plugin implication). "Gist" summarises the report; "Implication" is what the plugin should
do about it, not what upstream should do.

### Install / environment

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| cli#224 | Improve python version management | closed | Many pretext-support threads traced back to the *Python* install, not pretext itself | A setup/doctor skill should check Python/venv health before touching pretext-specific things |
| cli#218 | A better error message for pdf2svg issue [windows] | closed | Missing `pdf2svg` produced a cryptic image-build failure on Windows | Doctor check: verify per-target external executables (`pdf2svg`, `pdftoppm`, Ghostscript, `jing`, node) exist before build, with install hints per OS (operating system) |
| cli#257 | Document ubuntu/wsl setup steps | closed | WSL (Windows Subsystem for Linux) users hit "neither Ghostscript nor pdftoppm" with no guidance | Same as above — WSL is common enough to get its own doctor branch |
| cli#488 / #676 | Codespace should respect/install via `requirements.txt` | closed | Codespace template drifted from the pinned CLI version, breaking builds after a rebuild | Plugin should always check `requirements.txt` pinning vs. installed `pretext --version` before acting |
| cli#3161 (pretext) | Asymptote `-offscreen` misparsed by older Asymptote, silently no image | closed 2026-08-20 | A supported-looking flag silently produced *no error and no image* — the worst failure mode | Doctor/build-failure skill must specifically check for "expected asset file, got nothing" as its own class, not just non-zero exit |
| cli#1055 | Remove `pretextbook`/`pretext-cli` from PyPI? | open | Old package name `pretextbook` still installable and gives a stale version | Skill should warn if `pip show pretextbook` (not `pretext`) is what's installed |
| cli#1200 | Executables model has drifted from core's `pretext.cfg` (no `jing` entry) | open | CLI's own config model doesn't know about `jing`, which the CLI's own `validate` needs | Confirms the demo-build finding below: `jing` is *not* auto-managed by the CLI, unlike other executables |

### CLI usage / ergonomics

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| cli#688 | Unclear what `init` means | closed | npm users expect `init` to scaffold a project; `pretext init` instead refreshes CLI-managed files and can leave a broken project | Now resolved by the CLI itself (see §3) — command help text and the plugin's own guidance must say "`new` to create, `init` to refresh/upgrade files" |
| cli#249 | Never show traceback unless `-v debug` requested | closed | Design goal explicitly stated by maintainers: no Python tracebacks for non-coder authors | Confirms: default verbosity should stay `info`; a skill should never suggest `-v debug` as a first troubleshooting step to an author (only when asked to file a bug) |
| cli#547 / #597 | Nicer / better error messages for invalid `project.ptx` | closed | Malformed manifests used to produce cryptic errors | Largely fixed now (see demo-build test below — mismatched XML tag names produce file+line pointers), but still worth a "explain this manifest error" helper |
| cli#254 | `pretext generate ...`: what goes in the blank? | closed | Users didn't know `generate` takes an *asset type* (`webwork`, `latex-image`, ...), not a target | Command surface changed since (now `generate [asset-types]... -t target`); skill should give the current signature verbatim rather than guessing from old docs |
| cli#780 | `at_exit` `sys.exit` conflict producing a stack trace on success paths | closed | Internal cleanup-handler bug leaked a traceback even on a fine exit | Reinforces: don't treat "a traceback appeared" as necessarily fatal — check exit code too |
| cli#1117 | Failed asset generation not being detected | closed 2026-05-29 (recent) | A core error-message change broke the CLI's own detection of failed asset builds (Asymptote, WeBWorK) — assets silently didn't get flagged as failed | **Known-unfixed-adjacent**: asset-generation failure detection is fragile and has regressed at least once; plugin should independently check that expected output files exist after `generate`/`build`, not just trust the exit code |
| cli#414 | `/tmp` directories not cleaned up after successful build | closed | 38 GB (gigabytes) of `/tmp` accumulated on a staging machine | `--save-tmp-dirs` exists for debugging; plugin should never pass it by default and should be aware temp dirs are created per build |

### Build / validate

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| cli#568 | Validate `project.ptx` on every build | closed | Historical request, since implemented — manifest is now validated as part of build | Confirmed live: a broken manifest fails fast with file/line info |
| cli#1189 | "Heads-up: local validation rework in core pretext.py" | open 2026-07-10 | Maintainer-to-maintainer coordination note: core's validation internals were reworked (#3004, refined by #3027, #3033), CLI needs to track it | **Roadmap signal**: validation internals are actively being refactored as of mid-2026; a plugin that shells out to `pretext validate` is insulated from this churn (good), one that parses `pretext.py` internals directly would not be |
| pretext#3076 / #3075 | "Validation-plus" carrying forward old Schematron rules (xml:id/@label advice, WeBWorK+LaTeX `@` clash) | open | Schematron was retired (#2991); its context-dependent checks are being reimplemented in `pretext-validation-plus.xsl`, one at a time, and not all are done yet | Some "soft" author mistakes (missing `xml:id`, WeBWorK statements with raw `@` in LaTeX macros) are **known gaps** validation doesn't yet catch — worth a plugin-side lint pass until upstream lands them |
| cli#806 / #691 / #812 | If PDF build fails, dump `.tex`; better `tikz` error messages; log PDF failures | open | LaTeX/tikz compile failures currently point at *generated* file paths, not the author's source, and sometimes aren't logged as failures at all | **Known-unfixed**: PDF/LaTeX build failures are the weakest link in error attribution; plugin should map generated `.tex` line numbers back to source only with real substitution logic, and treat a "green" `pretext build` with a suspiciously small/absent PDF as suspect |
| cli#664 | Verbosity of log messages for XSLT (Extensible Stylesheet Language Transformations)-produced errors | closed but recurring pattern | XSLT `xsl:message` output is verbose and hard to read for authors | Plugin's build-log summarizer should filter/re-rank XSLT log noise rather than dumping the full log at an author |

### Deploy

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| cli#476 | Add support for custom domains | closed (12 comments) | Custom-domain (`CNAME`) deploys had real gotchas | Now supported, but... |
| cli#503 | `git deploy` breaks silently after a custom domain is set | closed | GitHub silently recreates a `CNAME` file that blocks pushes, no warning given | **Known-unfixed-flavoured**: silent breakage after a working custom-domain deploy is a recurring shape of bug across this repo's deploy issues; plugin should proactively check for `CNAME` drift before/after `pretext deploy` |
| cli#725 / #644 | "Make two deploy methods cohesive" / GitHub Action for `pretext deploy` | closed | Two divergent deploy paths existed: `pretext deploy` (pushes to `gh-pages` branch) vs. a GitHub Actions-based workflow (needs Pages set to "GitHub Actions" source) — easy to misconfigure by mixing them | Plugin must detect *which* deploy method a repo is using (branch vs. Action) before recommending a fix, and never suggest commands for the other method |
| cli#493 | `git deploy` hangs when password is wrong | open | Deploy can hang indefinitely waiting on a credential prompt rather than failing | Plugin driving `pretext deploy` non-interactively should set a timeout and treat a hang as a credential problem, not a build problem |
| cli#897 | Improve directions for setting up gh authentication | open | Users default to setting up personal access tokens that aren't actually required | Skill should give the *current*, minimal auth story (SSH (Secure Shell) or gh-cli credential helper), not the historically-recommended PAT (personal access token) flow |
| cli#837 | Deployment to local directory | open (13 comments) | No first-class "deploy to a folder / non-GitHub host" path; users want to self-host | **Roadmap-ish gap**: still no CLI support for non-GitHub-Pages deploy targets (Netlify/Cloudflare Pages are also open requests, cli#739) — plugin should treat "deploy" as GitHub Pages-only capability today |

### Images / assets (Asymptote, PreFigure, LaTeX images)

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| pretext#2412 | 100% width latex-images in "naked" images not 100% width in PDF | closed | Subtle PDF-only sizing bug | Cross-target visual QA (quality assurance) is genuinely hard to automate; flag as an area authors should manually check PDF proofs, not assume WYSIWYG (What You See Is What You Get) parity with HTML |
| pretext#2140 | Recompile latex images as needed (multi-pass) | closed | Some TikZ/LaTeX images need multiple compiler passes (à la `latexmk`) that PreTeXt didn't automatically do | If a plugin drives repeated builds, be aware first-pass image builds can legitimately look different from second-pass |
| cli#690 | Difficult to find image `.tex` files in case of error | closed | CLI's reported failing filename didn't match the actual generated `.tex` path | Even where "fixed," this class of path-mismatch bug has recurred (see `--latex` / `-l` flag added to `build` to dump LaTeX source for inspection) — plugin should always pass `-l`/`--latex` when diagnosing a PDF build problem |
| pretext#1869 | Check for bad results from the (remote) Asymptote server | open since 2022 | The *shared* Asymptote server used by `asy-method=server` can silently return bad output and PreTeXt doesn't notice | **Known-unfixed**: default/remote Asymptote rendering is not verified; recommend local Asymptote (`asy-method="local"`) for anything beyond trivial diagrams, consistent with cli#805's "consider making local the default" |
| pretext#2927 / #2838 | PreFigure diagrams mis-flagged as duplicate-label / interacting badly with `list-of` | closed, recent (2026) | PreFigure integration still has real edge-case bugs in mid-2026 | PreFigure is an actively-developing dependency; treat its interactions with exercises/list-of as an area to re-check per PreTeXt-CLI upgrade |

### WeBWorK

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| cli#662 | Improve WeBWorK generation: don't abort on a single problem failure | closed | One broken WW (WeBWorK) problem used to fail the whole `generate` run | Fixed, but confirms: WW generation is the most failure-prone asset type; always check `logs/` for individual-problem failures even on an overall success |
| pretext#2699 / #2700 | Generated `.pg` filenames collide (`1.pg` twice); chunking level not always applied | open | Real, currently-live bugs in the WeBWorK asset pipeline | **Known-unfixed**: flag WeBWorK authors to check generated filenames for collisions after adding a second WW question to the same section |
| pretext#3008 | HTML+WeBWorK: `hN` heading template reached without `$heading-level` | closed 2026-07-08 (very recent) | A shortened Preview Activity (with `task`s removed) broke WW's heading-level assumption | WeBWorK activities are structurally sensitive to `task` nesting; don't casually restructure WW-bearing activities |
| pretext-cli#419 | Custom format doesn't support WebWorK problem-set generation | closed | `format="custom"` in the manifest can't drive WW generation | Plugin should warn if an author with WeBWorK content tries a custom-format target |

### Runestone

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| pretext#2458 | FITB (fill-in-the-blank): unspecified `@mode` silently produces a broken exercise | closed 2025-06-18 | Missing an attribute produces a *silently* broken JSON blob that fails to load in Runestone, with no build error | **Known-unfixed shape** (validation-plus is only slowly catching these, see #3075/#3076 above): FITB and similar Runestone-only constructs can pass `pretext validate` and `pretext build` cleanly yet fail at runtime on Runestone. Plugin should maintain its own checklist of Runestone-fragile constructs (`fillin` mode, `activity`+`introduction` placement, timed exams) beyond what `validate` catches |
| pretext#2635 / #2294 | Activity introductions / FITB elements not retained inside `activity` | open | `activity` (Runestone-specific wrapper) has multiple open placement/rendering bugs | Treat `activity` as one of the least mature elements; recommend testing on a real Runestone deploy, not just local HTML build |
| pretext#2586 / #2683 | Timed-exam exercises repeatedly go missing from `runestone-manifest.xml` | open / closed-then-recurred | The *same* bug (timed exams absent from the manifest) has recurred more than once | Confirmed recurring regression — good candidate for a "known gotchas" warning list entry with a recheck-after-upgrade note |
| cli#513 / #432 | "Make deploying to Runestone easy" / Combined CI (continuous integration)/CD (continuous deployment) of PreTeXt+Runestone | closed (PROSE-labeled, i.e. contributor-project) | Runestone deployment is a separate, still partially-manual workflow from GitHub Pages deploy | Plugin's "deploy" skill needs a distinct Runestone path (different from `pretext deploy`), likely documentation-and-checklist rather than a driven command |

### Docs gaps

| # | Title | State | Gist | Plugin implication |
|---|---|---|---|---|
| pretext#1970 | Guide's image-folder instructions were wrong (says root/images, actually needs `external/`) | closed | Confirmed real docs/behaviour mismatch that persisted until reported | Bake authoritative "where do images/assets go" guidance into the skill from the current schema/examples, not by paraphrasing the Guide prose, which has lagged behavior before |
| pretext#2626 | Tutorial suggests building a target unavailable in the reader's own Codespace template | closed 2025-08-11 | The official "Your First PreTeXt Document" tutorial and the Codespace template can drift out of sync | Plugin's own quickstart must be generated from the *actual* `pretext new` template output, re-derived per CLI version, not hand-copied from the Guide |
| pretext#2141 | `pretext/pretext` (devscript) docs out of date, example needed | closed | Confirms devscript/legacy-script docs lag behind the actual CLI | Prefer capturing devscript's `--help` output directly (as this survey did) over trusting prose docs |
| pretext-cli#228 | "CLI builds of repo docs" — Rob manually maintains build of sample article/book/Guide | closed (3 comments) | The Guide, sample article, sample book, etc. are themselves built by the same toolchain and can drift when the CLI changes | Any bundled example content in the plugin should be periodically rebuilt/verified against the vendored `pretext` version, exactly per Rob's own practice |

## 3. Known-and-unfixed problems the plugin should proactively warn about

These are still open (or closed-and-recurred) as of 2026-08-28 and are squarely author-facing:

1. **Silent asset-generation failure.** Asymptote's `-offscreen` flag (pretext#3161, closed
   2026-08-20 — very recent, so likely still present in older Asymptote installs authors have not
   upgraded), the remote Asymptote server (pretext#1869, open since 2022), and the historical
   asset-failure-detection regression in the CLI (cli#1117) all share one shape: **the build
   reports success but the expected image/asset is missing or wrong.** The plugin should verify
   expected output files exist (non-trivial size, right count) after any `generate`/`build`, not
   trust exit code 0 alone.
2. **Validation doesn't yet catch every author mistake.** The Schematron→`pretext-validation-plus.xsl`
   migration (tracked by pretext#2991 "Schematron retired") is still landing checks incrementally
   (pretext#3075, #3076 open). Fill-in-the-blank without `@mode` (pretext#2458) is a known
   *validates-fine-but-silently-broken-on-Runestone* case. Treat `pretext validate` (exit 0) as
   necessary, not sufficient, especially for Runestone-only constructs.
3. **PDF/LaTeX failures are the weakest error-attribution path.** Generated `.tex` file names and
   locations don't reliably map back to `.ptx` source lines (cli#690, cli#806, cli#691, cli#812 —
   several still open). Always build print/PDF targets with `-l`/`--latex` so the generated LaTeX
   is available for inspection, and treat a suspiciously-fast or suspiciously-small PDF as a build
   failure even at exit 0.
4. **Deploy has two incompatible methods that are easy to cross-wire** (`pretext deploy` → `gh-pages`
   branch vs. a GitHub Actions workflow → Pages "Actions" source; cli#725, cli#644, cli#737).
   Detect which one a repo already uses before recommending deploy troubleshooting steps.
5. **Custom-domain deploys silently break** when GitHub recreates a `CNAME` file (cli#503, still
   the accepted shape of the bug as of the survey; no linked fix commit found in this window).
6. **Runestone-specific constructs (`activity`, `fillin`, timed exams) have a track record of
   silently misbehaving or vanishing from `runestone-manifest.xml`** even when the PreTeXt build
   is clean (pretext#2635, #2294, #2586/#2683 — a recurring bug, not a one-off). These need a
   post-build Runestone-specific checklist, not just `pretext build`/`validate` success.
7. **`jing` (the local RelaxNG validator) is not installed or managed by the CLI itself** — see the
   demo-build finding in §4. Authors following a plain `pip install pretext[all]` will find
   `pretext validate` (default `--method local`) exits 2 ("could not be performed") until they
   separately install Java+jing or switch to `--method server` / `--engine salve`.

## 4. Roadmap signals for a plugin design to anticipate

- **Validation internals are mid-refactor** (pretext#3004 merged 2026-07-05, refined by #3027,
  #3033; cli#1189 "heads-up" tracking issue is open). The CLI-facing `pretext validate` contract
  (exit codes, `--report-form terse` machine format) looks stable and is the right integration
  surface; don't couple to `pretext.py` internals.
- **Braille is being re-architected** (cli#1193, open 2026-07-14: new `pretext/lib/braille_translate.py`
  module referenced by PreTeXtBook/pretext#3037). Braille/tactile output (`braille`,
  `braille-emboss`, `braille-electronic`, `tactile` formats — all present in `devscript --help`)
  is an active area; a plugin shipping braille guidance should expect near-term format changes.
- **No `pretext import` command exists today.** LaTeX→PreTeXt conversion is handled by external,
  community tools (e.g. David Farmer's LaTeX-to-PTX converter, referenced in cli#271) rather than
  a first-party CLI subcommand. The map's Notes list `pretext import` as something to anticipate —
  treat it as **aspirational/roadmap, not shipped**; the plugin's "convert an existing manuscript"
  story currently has to be a skill-driven, LLM-assisted transform, not a CLI wrapper.
- **Codespaces/devcontainer is a living, still-fragile surface** (33 open-issue keyword hits in
  pretext-cli): recent open issues include sudo-less Docker image breaking `installPandoc.sh`
  (cli#1073), `postCreateCommand` limitations (cli#965), and Node.js missing from workflow images
  (cli#1026, open). A plugin that offers a "set up Codespaces" flow should re-verify the current
  devcontainer template rather than hard-coding one, since it has been rewritten multiple times.
- **Deploy targets are widening slowly.** Cloudflare Pages support is an open request (cli#739);
  local-directory deploy is an open, actively-discussed request (cli#837, 13 comments). Design the
  plugin's deploy skill around "GitHub Pages today, pluggable target tomorrow" rather than assuming
  GitHub Pages is the only destination long-term.
- **MathJax 4 migration is a long-running open tracking issue** (pretext#1841, opened 2022-10-02,
  still open with 34 comments) — math rendering behavior in HTML/EPUB output should be expected to
  change under a future PreTeXt-CLI upgrade.
- **`pretext-cli` v2 manifest (`ptx-version="2"`) is the modern baseline**; issues referencing v1/
  "legacy" manifests (e.g. cli#502 `ptx_version` handling) are almost all closed — the plugin
  should assume `ptx-version="2"` manifests and not spend design effort on legacy-manifest support.

## 5. CLI capability matrix (pretext-cli 2.51.0, verified live in `.venv/`)

Commands as of `pretext --help`: `build`, `deploy`, `devscript`, `generate`, `init`, `new`,
`support`, `update-project`, `upgrade`, `validate`, `view`. (Older issues refer to a single
`pretext update` command; it has since split into `update-project`, which updates project files
to match the installed CLI, and `upgrade`, which upgrades the installed CLI via pip — resolving
the historical `init`-vs-`update` naming confusion in cli#688.)

Global flags on the bare `pretext` invocation: `-v/--verbosity {debug|info|warning|error|critical}`
(default `info`), `--version`, `-t/--targets` (lists targets defined in `project.ptx` and exits),
`--save-tmp-dirs` (keep temp build dirs — debugging only, never default).

| Command | Purpose | Key flags | Output channel | Exit-code behaviour | Agent-friendliness notes |
|---|---|---|---|---|---|
| `pretext new [book\|article\|course\|demo\|hello\|slideshow]` | Scaffold a new project | `-d/--directory PATH`, `-u/--url-template URL` | stdout: short human log ("Generating new PreTeXt project...", "Generated resource file ...") | 0 on success; fails if target directory already has files, needs check first | Idempotency risk — no `--force`; check dir is empty before calling. No project.ptx warning is emitted since none exists yet. |
| `pretext init` | Regenerate/refresh CLI-managed scaffolding files (`project.ptx`, `.gitignore`, `devcontainer.json`, `requirements.txt`, `pretext-cli.yml`, `pretext-deploy.yml`, `installpandoc.sh`) for the current CLI version | `-r/--refresh` (force even if already initialized; back up existing as `*.bak`), `-f/--file <name>` (target just one file), `-s/--system` (reinstall core resources/npm) | stdout log; **always prints a `warning:` line to stdout (not stderr) when no `project.ptx` is found**, then falls back to a global config | 0 normally | Does NOT create a new project (that's `new`) — this is a maintenance/upgrade command. `-s` is the "something's broken, reinstall core" hammer. |
| `pretext build [target]` | Build one target (or default = first target) | `--clean` (wipe output dir first), `-g/--generate` (force asset regen), `-q/--no-generate` (skip asset regen even if stale), `-t/--theme` (theme only), `-l/--latex` (also drop LaTeX source in output, for PDF targets), `-x/--xmlid ID` (build subtree only), `--no-knowls` (hyperlinks instead of knowls, for partial previews), `--deploys` (build every target marked for deploy), `-i/--input PATH` (override manifest's source file) | stdout: staged progress log ("Generating any needed assets" → "Preparing to build..." → per-XSL-stage messages → "Finished build for target X" → "Success! Built requested target(s) without errors."); **also writes a timestamped file to `./logs/YYYYMMDD-HHMMSS.log`** (same content, `LEVEL : message` format) every invocation | 0 on success, non-zero on failure (schema/XML errors surface as `error:`-prefixed stderr-ish lines even though captured on stdout in this session — verify per-channel in your own harness); a malformed manifest or unwell-formed XML fails fast with **file+line pointers into the author's own source** | `--latex` should be passed by default when diagnosing any PDF build, since generated `.tex` paths are otherwise hard to trace back (cli#690, #806). `-x/--xmlid` is useful for testing a single section without a full rebuild. Every build leaves a fresh log file — an agent should read the newest file in `logs/` after any build for full detail, not just captured stdout. |
| `pretext view [target]` | Serve+preview a built target locally | `-a/--access {public\|private}`, `-p/--port INT`, `-b/--build` (build first), `-g/--generate` (generate assets first), `--no-launch` (don't open a browser — **use this from an agent**), `-r/--restart-server`, `-s/--stop-server`, `-d/--stage` (view staged deploy), `--default-server` (force plain Python server even in a Codespace) | stdout log with the server URL | Long-running / blocking unless `--stop-server` is used; not naturally scriptable | **Always pass `--no-launch`** in a non-interactive/agent context — default behaviour tries to open a real browser. Use `-s/--stop-server` to clean up rather than killing the process, since it manages a background server registry keyed by port. |
| `pretext validate [target]` | Validate source against the RelaxNG schema + "validation-plus" checks | `--method {local\|server}` (local jing vs. remote validation service), `--report-form {full\|terse}` (terse = one tab-separated message/line, **machine-readable**), `--engine {jing\|salve}` (salve = npm-based, no Java needed, experimental, ignored under `--method server`) | stdout: staged log; **writes a human-readable report to `logs/main-validation.txt`** on completion, referencing the *assembled* source (`logs/main-assembled.xml`), not the author's individual `.ptx` files, for line numbers | **Documented exit codes: 0 = valid, 1 = invalid, 2 = validation could not be performed** (e.g. `jing` not installed/on PATH) | Live-tested: this venv had no `jing` on PATH — `--method local` (the default) exited 2 with a clear remediation message ("Install jing ... or use `--method server` or `--engine salve`"). `--method server` worked over the network (hit `mathgenealogy.org:9000`) and produced a clean validation, so it is the safer default for a plugin unless the author's environment is known-good. Use `--report-form terse` for programmatic parsing; the default `full` report is meant for humans and includes a lot of preamble explaining the report format itself. |
| `pretext generate [asset-types...]` | Generate/regenerate assets (WebWorK, latex-image, sageplot, asymptote, prefigure, youtube, codelens, datafile, interactive, qrcode, mermaid, myopenmath, dynamic-subs, references, stack, gdscript, or `all`) | `-t/--target NAME`, `-x/--xmlid ID` (single element), `-q/--only-changed`, `--all-formats`, `--clean` (wipe cache first), `-f/--force` (ignore cache), `-s/--slow` (longer timeout for interactive screenshots) | stdout progress log | 0 on success; per cli#1117, a failed individual asset (e.g. one broken Asymptote figure) has not always been reliably surfaced as a failure — **verify output files independently** | Don't assume asset-count == success-count; historically some asset types (WebWorK especially, cli#662) can fail per-item without failing the whole `generate` call — that's now considered correct behavior (don't abort the whole run on one bad problem) but means a naive "exit code 0 ⇒ all assets built" assumption is wrong. |
| `pretext deploy` | Push a built target to GitHub Pages (`gh-pages` branch) | `-u/--update-source` (also commit+push source), `-s/--stage-only` (stage, don't deploy), `-p/--preview` (preview the staged deploy, don't deploy), `--no-push` (commit to `gh-pages` locally but skip the network push — useful when auth is broken) | stdout log with git-remote-derived messages (repo/user, URLs) | Requires git + configured GitHub remote; can **hang** on bad credentials (cli#493) rather than failing fast | For a non-interactive agent, always prefer `--stage-only`/`--preview` first and inspect before an actual `deploy`; set a wrapper timeout since credential-prompt hangs are a known failure mode; detect whether the repo already deploys via a GitHub Actions workflow (`pretext-deploy.yml`) before invoking this — the two deploy paths are easy to cross-wire (cli#725). |
| `pretext support` | Dump environment/version/manifest info for a pretext-support bug report | none besides `-v` | stdout: structured plain-text dump (CLI version + PyPI link, core-resources commit hash, OS string, Python version+path, cwd, project path, then the literal contents of `project.ptx`) | 0 | Good building block for a plugin "doctor"/diagnostic skill — it's the maintainers' own canonical "what do we need to know" bundle; an agent troubleshooting a build should capture this output alongside the newest `logs/*.log` file. |
| `pretext update-project` | Sync project files to match the installed CLI version | `-b/--backup`, `-f/--force` | stdout log | 0 normally | This is the "my project.ptx/templates are stale relative to my installed CLI" command — distinct from `upgrade` (which changes the installed CLI itself). |
| `pretext upgrade` | `pip`-upgrade the installed `pretext-cli` package | none besides `-v` | stdout | 0 normally; network/pip failures propagate pip's own exit code | Straightforward; agent should re-run `pretext --version` after to confirm the upgrade landed, and expect `update-project` may then be needed too. |
| `pretext devscript` | Direct alias to the legacy `pretext/pretext` core script | Full legacy surface: `-c/--component` (18 asset/component types — prefigure, asy, sageplot, mermaid, latex-image, webwork, stack, references, pg-macros, dynamic, gdscript, youtube, play-button, qrcode, preview, mom, math, theme, trace, datafile, latex-package, doc, all, tikz-deprecated), `-f/--format` (40+ output formats: svg/pdf/pdf-plus/pdf-fo/png/eps/mml/braille/braille-emboss/braille-electronic/tactile/speech/source/text/markdown/markdown-zip/latex/latex-plus/fo/html/html-zip/html-scorm/revealjs/beamer/epub-*/jupyter*/assembly-*/webwork-sets/all), `-p/--publication FILE`, `-x/--parameters KEY VALUE...`, `-X/--XSL FILE`, `-M/--method`, `-r/--restrict XMLID`, `-s/--server URL` (WeBWorK only), `-o/--output FILE`, `-d/--directory DIR`, `-a/--abort` (abort on first recoverable error, vs. default keep-going), `-z/--tgz` (WeBWorK problem sets as a compressed archive), `-V/--validate {full\|terse}` | argparse-style help/usage; same log conventions as the wrapping CLI | Exit codes not separately documented at this layer; inherits from the underlying script | This is the **full power-user surface** the higher-level `build`/`generate`/`validate` commands are built on top of — e.g. it's the only place `braille`, `braille-emboss`, `speech`, `pdf-fo`, and the `assembly-*` (pre-processor-output) formats are directly reachable. A plugin targeting accessibility formats (braille, speech) or wanting the assembled/pre-processed XML for its own tooling should know this exists, but should prefer the higher-level commands wherever they cover the same ground, since `devscript`'s flags are lower-level and less validated. |

### Demonstrated behaviours (from building `scratch/demo-book`)

- `pretext new book -d demo-book` created `project.ptx`, `requirements.txt`, `source/`,
  `publication/`, `assets/`, `README.md` — no `.git` repo, no devcontainer files by default (those
  come from `pretext init` / are conditional on git-tracking per cli#801).
- `pretext -t` (bare, no subcommand) printed the two configured target names (`web`, `print`) and
  exited — a cheap way for an agent to enumerate targets without parsing `project.ptx` XML itself.
- `pretext build web` succeeded end-to-end including a live network fetch of Runestone Services
  from `runestone.academy`'s CDN (Content Delivery Network) (even for a plain `book` template with no explicit Runestone
  content) — **builds are not fully offline** by default.
- `pretext validate web` (default `--method local`) failed with exit 2 in this environment because
  `jing` is not on PATH, even though `java` is present — jing is a separate binary dependency the
  CLI does not vendor/install itself, unlike Node/npm assets. `--method server` succeeded and wrote
  `logs/main-validation.txt`.
- Introducing a mismatched XML tag (`<title>Demo<badtag></title>`) made `pretext validate` fail at
  the assembly stage with an exact, useful error: `Could not assemble source for validation:
  Opening and ending tag mismatch: badtag line 8 and title, line 8, column 32 (main.ptx, line 8)` —
  file and line number both point directly at the author's real source file. This is a strong,
  agent-friendly signal for "fix this XML" tasks.
- Every `build`/`generate`/`validate` call writes to `./logs/` in the project (timestamped `.log`
  files for build/generate, `main-validation.txt` + `main-assembled.xml` for validate) — an agent
  should treat `logs/` as the durable machine-readable record, and stdout as a live progress stream
  that duplicates but does not always fully replace it.

## Sources

- `gh issue list --repo PreTeXtBook/pretext --state all --limit 300 --json number,title,state,labels,createdAt,closedAt,body,comments`
- `gh issue list --repo PreTeXtBook/pretext-cli --state all --limit 300 --json number,title,state,labels,createdAt,closedAt,body,comments`
- `gh issue list --repo PreTeXtBook/pretext --state open --limit 500 --json number,title,labels` (label distribution)
- `gh issue list --repo PreTeXtBook/pretext-cli --state open --limit 500 --json number,title,labels` (label distribution)
- `gh api repos/PreTeXtBook/pretext[-cli]/milestones`
- Live CLI session: `pretext --help`, `pretext {new,init,build,view,validate,generate,deploy,support,update-project,upgrade,devscript} --help`, and a real build/validate cycle in `scratch/demo-book` (pretext-cli 2.51.0)
