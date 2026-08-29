# Setup and Toolchain

## What PreTeXt actually is

Two repositories, one language:

| Piece | Repo | Role |
|---|---|---|
| Core | `PreTeXtBook/pretext` | The XML vocabulary (`schema/`), the XSLT that transforms it (`xsl/`), CSS/JS themes, worked examples (`examples/`), and the Guide's own source (`doc/guide/`). |
| CLI | `PreTeXtBook/pretext-cli` | `pip install pretext`. Wraps the core in a project model (`project.ptx`, targets, asset caching) so you never write a Makefile. |

A build is: assemble the modular XML into one tree → run an XSLT stylesheet
(`pretext-html.xsl`, `pretext-latex.xsl`, `pretext-epub.xsl`, …) → post-process in Python
(generate images, bundle EPUB, compile LaTeX).

The CLI downloads a matching copy of the core into `~/.ptx/<version>/core/` on first use.
That directory is the ground truth for what your installed version actually does — grep it
when the Guide and the behaviour disagree.

## Install

```bash
python3 -m venv .venv
.venv/bin/pip install "pretext[all]"
.venv/bin/pretext --version
```

`[all]` adds `pelican` (multi-target landing pages) and `prefigure` (accessible diagrams).
`pipx install pretext` also works and manages the venv for you.

**Put the venv's `bin` directory on `PATH` before building.** The CLI invokes helper
executables by bare name (`playwright`, `node`, `xelatex`, `asy`, `sage`). Calling
`.venv/bin/pretext` without activating the venv makes those lookups fail — see
[08-gotchas.md](08-gotchas.md).

```bash
source .venv/bin/activate     # or: export PATH="$PWD/.venv/bin:$PATH"
```

## What each output needs

| Target | Requires | Notes |
|---|---|---|
| `html` (web) | Python only | Node ≥ 18 if you want a theme other than the default, or a custom SCSS theme. |
| `pdf` / `latex` | TeX (TeX Live, MiKTeX, TinyTeX); `xelatex` by default | `format="latex"` emits `.tex` without compiling — useful to inspect the preamble with no TeX installed. |
| `epub` / `kindle` | Node + npm, **and Playwright with a browser** | Playwright screenshots interactives for the static book. See below. |
| `latex-image` (TikZ/PGFPlots) | TeX | Generates SVG for HTML, PDF for print. |
| `sageplot` | SageMath ≥ 10 | |
| `asymptote` | nothing, by default | `asy-method="server"` uses a remote service; set `local` if `asy` is installed. |
| `prefigure` | `pip install pretext[prefigure]` | Accessible SVG diagrams; some system libs needed. |
| `webwork` | a WeBWorK 2.19+ server, or a local `pg` checkout | `https://webwork-ptx.aimath.org` is the community server. |
| `mermaid` | `mmdc` (Node) | |
| `validate` (jing engine) | Java | Or use `--engine salve` (Node, self-installing) or `--method server`. |

### Playwright, for EPUB/Kindle

The EPUB pipeline renders a static preview image of every `<interactive>` and `<video>`
through a headless browser. Install the browser once:

```bash
.venv/bin/python -m playwright install chromium
```

If the CLI reports `[Errno 2] No such file or directory: 'playwright'`, the venv is not on
`PATH`. Fix that rather than reinstalling.

## Editor

VS Code with the `pretext-tools` extension (Oscar Levin) — snippets, schema-aware
completion, build/view commands, and the `salve` validator. `.ptx` is the conventional
extension; `.xml` gets broader editor support but fewer PreTeXt features.

## Cloud alternative

`https://github.com/PreTeXtBook/pretext-codespace` — a GitHub template with a devcontainer
that has TeX, Sage, Node and the CLI preinstalled. The same image runs locally through
Docker Desktop + VS Code Dev Containers (~5 GB). This is the path to recommend to a
co-author who does not want a toolchain.

## Verified on this machine (2026-08-28)

CLI 2.51.0, Python 3, Node 24.16, Java present; no TeX, no Sage.
`html`, `latex`, `epub` and `kindle` targets all built successfully. `pdf` requires TeX.
