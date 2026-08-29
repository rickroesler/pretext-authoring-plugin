# Licensing: what the plugin may embed, and what it should carry itself

Research ticket: [#8](https://github.com/rickroesler/pretext-authoring-plugin/issues/8) (wayfinder map [#1](https://github.com/rickroesler/pretext-authoring-plugin/issues/1)).

Sources inspected directly in `vendor-pretext/` (shallow clone of `PreTeXtBook/pretext`, commit-current as of 2026-08-28) plus the GitHub API for `PreTeXtBook/pretext-cli` and the org's other repos. Everything under "Quotes" below is copied verbatim from those files; everything under "Interpretation" is my reasoning, not legal advice.

## 1. Per-component licence table

| Component (vendor-pretext path) | Licence | Evidence |
|---|---|---|
| Whole repo default (`COPYING`, and the identical header on nearly every file) | **GPL (GNU General Public License), "version 2 or version 3 of the License (at your option)"** | `COPYING`: *"PreTeXt is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 2 or version 3 of the License (at your option)."* Copyright (C) 2013-2026 Robert A. Beezer. |
| `schema/` (RELAX-NG: `pretext.rnc/.rng/.xml`, `publication-schema.*`) | GPL v2-or-v3 (same header, `schema/README.md`) | Same COPYING block, Copyright (C) 2017-2026 Beezer. |
| `examples/` (templates: `hello-world/`, `minimal/`, `sample-article/`, etc.) | GPL v2-or-v3, per-file (checked `hello-world.xml`, `minimal/source/main.ptx`, `minimal/project.ptx`, `minimal/publication/publication.ptx` — each carries the full GPL header individually) | `examples/README.md` same block; individual `.xml`/`.ptx` files carry it too. |
| `examples/sample-book/` (derived from Judson's *Abstract Algebra*) | GPL v2-or-v3 as part of the repo, **but** underlying content copyright is **Thomas W. Judson**, not Beezer | `legal/copyright-holders.md`: *"Thomas W. Judson — 17 — `examples/sample-book/` — the content — as-is"*. The directory's own GPL header is present, but the "as-is" flag means Judson's separate copyright notice must not be altered — this file's status is murkier than pure-Beezer templates (see §3). |
| `examples/showcase/source/` | GPL v2-or-v3, copyright **The PreTeXt Organization** (not Beezer personally) | `legal/copyright-holders.md` row, "as-is". |
| `examples/showcase/generated/latex-image/` | GPL v2-or-v3 wrapper, copyright **Glyph & Cog, LLC (Limited Liability Company)** for the produced files | `legal/copyright-holders.md` row, "as-is". These are tool-generated images — avoid using as templates. |
| `xsl/` (stylesheets) | GPL v2-or-v3 | `xsl/README.md` same header. |
| `xsl/entities.ent`, `xsl/pretext-epub.xsl`, `xsl/pretext-text-utilities.xsl` | GPL wrapper, copyright **O'Reilly Media, Inc.** | `legal/copyright-holders.md`: *"as-is — the headers quote O'Reilly's own permission for code examples."* Don't lift code from these three files without reading their embedded O'Reilly permission notice. |
| `css/` (theme SCSS (Sassy CSS)/CSS) | GPL v2-or-v3 | `css/README.md` same header. |
| `js/pretext.js`, `js/knowl.js`, `js/mathjax_startup.js`, `js/pretext_add_on.js`, `js/pretext_search.js` | GPL v2-or-v3 (project's own JS; consolidated here from the now-obsolete `JS_core`/`CSS_core` repos, per those repos' READMEs: *"This repository is obsolete. PreTeXt JS is now in the main PreTeXt repository."*) | Header on `pretext.js` doesn't repeat the licence block but the file lives under the repo's blanket `COPYING`; no separate notice contradicts it. |
| `js/jquery.min.js` | **MIT** (third-party, vendored) | File header: `/*! jQuery v3.3.1 | (c) JS Foundation and other contributors | jquery.org/license */`. Not PreTeXt's own code — do not treat as GPL. |
| `doc/guide/` (the PreTeXt Guide text/prose) | **GNU Free Documentation License (GFDL) v1.3 or later, no Invariant Sections, no Cover Texts** | `doc/guide/COPYING`: *"Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.3 or any later version published by the Free Software Foundation; with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts."* Copyright (C) 2013-2026 Robert A. Beezer, David Farmer, Alex Jordan, Mitchel T. Keller (per `legal/copyright-holders.md`). |
| `doc/guide/developer/` | GFDL, additional co-holders Beezer/Farmer/Jordan (no Keller) | `legal/copyright-holders.md` row. |
| `legal/gpl-license-v2.txt`, `legal/gpl-license-v3.txt`, `doc/guide/COPYING`, `doc/guide/appendices/gfdl-pretext.xml`, `examples/sample-book/gfdl-mathbook.xml` | Verbatim FSF (Free Software Foundation) licence texts — not modifiable at all | `legal/copyright-holders.md`: *"as-is — verbatim license texts, whose own terms forbid modification."* |
| `script/` (the `pretext` Python launcher script, precursor to the CLI (command-line interface)) | GPL v2-or-v3 | Same blanket COPYING; not separately notated. |
| **`pretext-cli`** (separate GitHub repo, what authors actually run) | **GPL-3.0** | `gh api repos/PreTeXtBook/pretext-cli/license` → `spdx_id: "GPL-3.0"`; `LICENSE` file is the plain GPLv3 text (no "or later" option language visible in the file itself, unlike the monorepo's COPYING). |

### Pattern across the whole PreTeXtBook org (for precedent)

```
pretext            NOASSERTION   (GitHub's detector can't parse the "v2 or v3, your option" COPYING)
pretext-cli        GPL-3.0
html-static        GPL-3.0
pretext-tools      MIT
a11y               MIT
pretext-codespace  MIT
JS_core            (none — repo obsolete, folded into pretext)
CSS_core           (none — repo obsolete, folded into pretext)
Enhancements       (none)
JS_lib             (none)
pretext-projects   (none)
PROSE              (none)
pretext-docker     (none)
community          (none)
PreTeXt-website     (none)
```

**Interpretation:** the org runs a de-facto split. The document-processing engine and its schema/XSL/CSS/JS/examples (`pretext`, `pretext-cli`, `html-static`) are copyleft (GPL-2-or-3, or GPL-3-only for the newer `pretext-cli`). Peripheral, ecosystem/developer-experience tooling that doesn't itself carry the document-processing logic (`pretext-tools`, `a11y` resources-and-tests, `pretext-codespace` a project template) is **MIT**. A Claude Code plugin that assists authors — analogous to `pretext-tools`/`pretext-codespace` in kind, not to the engine itself — fits the MIT bucket by the org's own precedent.

## 2. What each licence permits, reasoned through

### PreTeXt's own stated principle (important, and easy to miss)

`README.md` principle 8: *"PreTeXt is free: the software is available at no cost, with an open license. **The use of PreTeXt does not impose any constraints on documents prepared with the system.**"*

**Interpretation:** this is the GPL's standard "mere use of a GPL tool doesn't make your output GPL" position (the same reasoning that lets people compile proprietary code with GCC (GNU Compiler Collection)), made explicit for PreTeXt. It protects **the textbooks that authors write using the plugin** — a book built with our skill is not thereby GPL or GFDL. It does **not** relicense the schema/XSL/templates/Guide text themselves, and it does not, by itself, license *our plugin* (which embeds copies of that material) under anything.

### (a) Reference text paraphrased/derived from the Guide → GFDL 1.3+

- **Quote:** *"with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts"* — this is the most permissive GFDL configuration; there's no unremovable boilerplate to carry forward.
- **Interpretation:** Copyright law, not license wording, decides whether a "paraphrase" is a derivative work. Facts and procedures (e.g. "the `<exercise>` element takes a `<statement>` and one or more of `<hint>`/`<answer>`/`<solution>`") are not copyrightable and can be restated freely in our own words with no GFDL obligation. Close paraphrase that tracks the Guide's own sentence structure, examples, or explanatory phrasing is a derivative work and inherits GFDL: it must carry a copy of the license, preserve the title/authorship acknowledgement, and (per GFDL §4) list the document as a modification. Direct quotation of Guide prose (even short passages used as reference snippets) is unambiguously GFDL-covered content.
- **Recommendation:** favor rewriting-from-facts over paraphrase-from-prose wherever practical (this also produces better, more Claude-native reference text). Where we do lift or closely track Guide wording, license *that reference file* under GFDL 1.3-or-later and carry the attribution block below — don't try to launder it into MIT.

### (b) Templates copied from `examples/` → GPL v2-or-v3

- **Quote (recurring header, verbatim in every `.ptx`/`.xml` template file checked):** *"PreTeXt is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License... PreTeXt is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY..."*
- **Interpretation:** copying `examples/minimal/source/main.ptx` or `examples/hello-world/hello-world.xml` into our plugin's `templates/` directory is redistribution of a GPL-licensed file. GPL doesn't require *us* to relicense our own separately-written code just because we ship a GPL file alongside it — GPLv3 §5's "aggregate" language (and the equivalent long-standing GPLv2 practice) treats independent works bundled on the same medium as **mere aggregation**, not a combined/derivative work, provided they aren't compiled/linked into one program and each retains its own license. A markdown skill's scripts sitting next to a copied `.ptx` template file, used as inert example data rather than executed/imported code, is a strong fit for that carve-out — but it is not a bright line, and "how tightly does the skill's tooling operationalize the template" matters (see §4 flag).
- **Recommendation:** keep each copied template file's original GPL header **intact and unmodified**, keep templates in their own clearly-labeled directory (e.g. `templates/upstream-pretext/`) rather than interleaved with our MIT code, and don't merge/rewrite them beyond what's needed for the plugin's purpose. Avoid `examples/sample-book/` (Judson copyright, murkier "as-is" status) and `examples/showcase/generated/` (Glyph & Cog copyright) as template sources — use the clean Beezer/PreTeXt-Organization-authored examples (`hello-world`, `minimal`, `sample-article`, `showcase/source`) instead.

### (c) Schema excerpts → GPL v2-or-v3

- Same header, same reasoning as (b). A short excerpt of `pretext.rnc` (e.g. "the content model for `<exercise>`") reproduced in a reference doc is a partial copy of a GPL file. Short factual excerpts of a formal grammar arguably carry thin copyright protection on their own, but the safe, low-friction approach is identical to templates: keep excerpts attributed, keep the GPL notice on any substantial copied block, and prefer *linking/describing* the schema over reproducing large verbatim chunks.
- **Recommendation:** treat schema excerpts like templates — GPL-covered, attributed, kept separate from MIT code, minimized to what's functionally needed (a validation error is far more useful to an author than a wall of RELAX-NG).

### (d) Scripts we write ourselves → our choice, recommend MIT

- These are original works with no PreTeXt GPL code inside them (we are not copying pretext-cli/xsl processing logic — we're calling `pretext validate`/`pretext build` as an external CLI, which is normal-use, not linking).
- **Recommendation: MIT**, matching the org's own precedent for peripheral tooling (`pretext-tools`, `a11y`, `pretext-codespace` are all MIT). This is also the friendliest choice for Claude Code marketplace distribution — no copyleft obligations flow onto the marketplace infrastructure or onto other plugins packaged alongside it, and it's the license form GitHub/npm/PyPI tooling and most reviewers expect with zero friction.

## 3. Recommendation: split licensing for the plugin repo

Adopt the same split the PreTeXtBook org already uses for itself:

1. **Top-level `LICENSE` = MIT.** Covers everything we author ourselves: `SKILL.md` files written in our own words, hooks, commands, agent definitions, scripts/MCP (Model Context Protocol) glue, and any original reference prose that doesn't closely track Guide wording. This is the license a transfer to `PreTeXtBook` org can adopt with zero friction — it matches their own `pretext-tools`/`a11y`/`pretext-codespace` pattern, so Rob's team doesn't have to negotiate a new license just to host it.
2. **A `THIRD-PARTY-NOTICES.md` (or `NOTICE`) file at the repo root** cataloguing every piece of embedded upstream material, its own license, and where it lives:
   - GPL v2-or-v3 files copied from `vendor-pretext/examples/` and `vendor-pretext/schema/` (templates, schema excerpts) — kept in their own subdirectories, headers intact.
   - GFDL 1.3-or-later reference material derived from `doc/guide/` — kept in its own subdirectory (e.g. `skills/pretext/references/guide-derived/`), each file carrying the attribution block below.
3. **Don't relicense copied upstream files.** Never strip a GPL header off a template or replace it with MIT; never present Guide-derived reference text as if it were our own original prose with no upstream tie. Keeping the boundaries file-level and directory-level is what makes both (a) the "mere aggregation" GPL argument and (b) a frictionless move to the PreTeXtBook org hold up.
4. This mirrors, almost exactly, how the *upstream* `pretext` repo itself is organized (GPL/GFDL core content, with the org running MIT for adjacent tooling) — so proposing it to Rob should read as "we followed your own pattern," not as a new ask.

## 4. Ready-to-paste attribution / NOTICE text

**Top-level `LICENSE`:** standard MIT license text, copyright line `Copyright (c) 2026 Rick Roesler` (update on transfer to PreTeXtBook org / Rob Beezer per whatever the org's transfer convention is).

**`THIRD-PARTY-NOTICES.md`** (paste as-is, filling in the bracketed file lists as the plugin's contents solidify):

```markdown
# Third-Party Notices

This plugin's own code, hooks, commands, agents, and originally-written
reference text are licensed under the MIT License (see LICENSE).

This plugin also embeds unmodified or lightly-adapted material from the
PreTeXt project (https://pretextbook.org, https://github.com/PreTeXtBook/pretext),
copyright (C) 2013-2026 Robert A. Beezer and contributors, under its own
licenses as noted below. Embedding this material does not relicense it;
each file below remains under its original license.

## GPL-licensed material (templates, schema excerpts)

Files under `templates/upstream-pretext/` and `references/schema-excerpts/`
are copied or excerpted from the PreTeXt project's `examples/` and `schema/`
directories and are licensed under the GNU General Public License, version 2
or version 3, at your option. See `LICENSE-GPL-2.txt` and `LICENSE-GPL-3.txt`
(copied verbatim from PreTeXt's `legal/`) for the full license text.

  Copyright (C) 2013-2026 Robert A. Beezer (and, where noted in individual
  file headers, additional co-authors / The PreTeXt Organization).

  PreTeXt is free software: you can redistribute it and/or modify it under
  the terms of the GNU General Public License as published by the Free
  Software Foundation, either version 2 or version 3 of the License (at
  your option).

Files: [list populated as templates/schema excerpts are added]

## GFDL-licensed material (Guide-derived reference text)

Files under `skills/pretext/references/guide-derived/` contain text derived
from or closely following *The PreTeXt Guide* and are licensed under the GNU
Free Documentation License, Version 1.3 or any later version published by
the Free Software Foundation, with no Invariant Sections, no Front-Cover
Texts, and no Back-Cover Texts.

  The PreTeXt Guide, Copyright (C) 2013-2026 Robert A. Beezer, David Farmer,
  Alex Jordan, Mitchel T. Keller. Source: https://github.com/PreTeXtBook/pretext,
  doc/guide/.

Files: [list populated as guide-derived references are added]

## Other third-party code

- jQuery (bundled by PreTeXt itself, not by this plugin) — MIT License,
  (c) JS Foundation and other contributors. Not directly embedded here;
  noted for completeness if any PreTeXt-generated build output is ever
  vendored alongside this plugin.
```

**Per-file header to keep on any copied GPL template/schema excerpt** (do not remove or rewrite the existing upstream header — just leave it as found in `vendor-pretext/`; this note is for cases where a header must be re-added, e.g. after re-extracting a schema fragment):

```
<!-- Derived from the PreTeXt project (https://pretextbook.org).
     Copyright (C) 2013-2026 Robert A. Beezer and contributors.
     Licensed under the GNU General Public License v2 or v3, at your
     option. See THIRD-PARTY-NOTICES.md and LICENSE-GPL-{2,3}.txt. -->
```

**Per-file header for Guide-derived reference markdown:**

```
<!-- Derived from The PreTeXt Guide (https://pretextbook.org), doc/guide/.
     Copyright (C) 2013-2026 Robert A. Beezer, David Farmer, Alex Jordan,
     Mitchel T. Keller. Licensed under the GNU Free Documentation License,
     v1.3 or later, no Invariant Sections, no Cover Texts.
     See THIRD-PARTY-NOTICES.md. -->
```

## 5. What I'm confident about vs. what needs a lawyer or Rob's explicit OK

**Confident (license texts are unambiguous, quoted above):**
- Core repo default is GPL v2-or-v3; `doc/guide/` is GFDL 1.3+; `pretext-cli` is GPL-3.0; jQuery is MIT; the org's peripheral tooling repos are MIT.
- "Use of PreTeXt doesn't constrain output documents" is PreTeXt's own stated position and covers authors' books, not our embedded copies of PreTeXt's own material.
- MIT is the right license for our own original code/prose, and matches the org's own precedent for this *kind* of repo (peripheral tooling, not the engine).

**My interpretation, not settled law — flag for Rob and/or a lawyer before public release:**
1. **Whether paraphrased Guide text triggers GFDL.** I'm reasoning from general derivative-work doctrine (facts free, close paraphrase not free); there's no bright-line test, and reasonable people could disagree about how much rewriting is "enough." Rob, as a co-holder of the Guide's copyright, can simply bless liberal reuse in writing, which would moot this question — worth asking him directly given he's already receptive (per the wayfinder map's origin thread).
2. **Whether "mere aggregation" cleanly covers a Claude Code skill.** GPLv3 §5's aggregation carve-out was written with software linking/compiling in mind. A skill's `SKILL.md` that *instructs* an LLM to read and use a GPL template file, and scripts that programmatically consume schema excerpts, sit in a genuinely gray zone between "independent works merely bundled" and "one operational system." I've recommended the structure (separate directories, intact headers, minimal excerpting) that makes the aggregation argument as strong as possible, but this is exactly the kind of question FSF-savvy counsel — or, more cheaply, Rob's own comfort level as the copyright holder — should settle before this ships publicly or transfers to the org.
3. **`examples/sample-book/` and Judson's copyright.** The "as-is" flag in `legal/copyright-holders.md` signals this file needs care beyond the standard GPL header; I've recommended simply avoiding it as a template source rather than resolving the ambiguity, which sidesteps the question but doesn't answer it.
4. **Claude Code marketplace distribution mechanics.** I don't know precisely how the marketplace packages/serves plugin repos (git clone vs. some other bundling), and whether that distribution channel itself imposes terms in tension with GPL/GFDL redistribution requirements (e.g. can a Guide-derived GFDL file's required license copy travel through the marketplace's packaging). Worth a direct check against Anthropic's plugin marketplace docs, or asking Anthropic, before relying on my assumption that shipping a git repo with a NOTICE file is sufficient.
5. **The GPL "or later" question on transfer.** Upstream's own COPYING says "v2 or v3, at your option" but the newer `pretext-cli` repo ships plain GPL-3.0 only. If the PreTeXtBook org has quietly moved to GPL-3-only for anything new, our GPL-covered embedded files should track whichever choice Rob's team actually wants going forward — worth confirming rather than assuming "v2 or v3" is still the standing default for new/adjacent repos.
