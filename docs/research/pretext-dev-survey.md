# Survey: pretext-dev and pretext-announce (last ~12 months)

Ticket: #3. Question: what is in flux, and what do maintainers wish authors did?

## Method and sampling bias

Google Groups (`groups.google.com/g/pretext-dev`, `.../pretext-announce`) has no API
available to this session; access was via `WebFetch` against the group listing page
(~30 most-recent threads) and `.../search?q=<term>` result pages (~30 hits per query,
also recency-biased), plus direct fetches of individual thread URLs. `WebFetch`
summarizes fetched HTML with a small model rather than returning raw text, so quotes
below were re-fetched with an explicit "verbatim, no paraphrase" instruction and
cross-checked for consistency across two independent fetches before being reported as
exact. Browser tools (chrome-devtools MCP) were available as a fallback for scrolling
older results but were not needed for the search-term approach; the playwright MCP was
not tried (flagged as possibly broken).

**Known biases / limits:**
- Every listing and search view is capped at roughly the 30 most recent matches, so
  coverage of the *tail* (e.g. April–June 2025, most of 2023) is thinner and depends on
  whichever search term happened to surface it. Counts of "how many [CHANGE] notices in
  12 months" below are a **lower bound** from a ~30-item announce window that already
  reaches back to January 2025 — the real rate is at least this high.
  Volume is high enough that the front page of pretext-announce alone (30 items) only
  reaches back to March 2025, meaning a full 12-month count would need pagination this
  session could not access.
- Search snippets are frequently truncated (Google Groups' own preview text); several
  quotes below are the longest verifiable fragment, not the full message, and are
  labeled accordingly.
- No message IDs/permalinks were captured (WebFetch summarized rather than returned
  hrefs), so citations below are by thread title + author + date, matching what's
  visible in the group UI.
- The local `vendor-pretext` clone is a **shallow clone (depth 1)**: `git log` shows
  exactly one commit (`929e25b`, dated 2026-08-27), so `git log --since=... -- schema/`
  is **useless for churn analysis** (returns 1, trivially the single commit present).
  Historical schema churn could not be cross-checked from the clone; only the clone's
  *current-state* `schema/experimental-features.xml` was usable, and it is the single
  best authoritative source for "what's still moving" (below).

## (a) Dated list of schema/behaviour changes — churn estimate

Pattern observed on pretext-announce: nearly every `[CHANGE]` notice from Rob Beezer
follows the same shape — announce the new form, keep the old form working with a
**deprecation warning** for a transition period, then remove it later in a separate
`[DEPRECATE]`/removal notice. Hard, no-warning breaks are rare and are concentrated in
the Runestone-tied "dynamic exercise" vocabulary (which `experimental-features.xml`
explicitly calls out as never entering the production schema without a compatibility
path).

Dated list (most recent 30 items visible on pretext-announce, i.e. back to roughly
Jan 2025 for `[CHANGE]`-style items; not exhaustive — see sampling note above):

| Date | Type | Item | Note |
|---|---|---|---|
| 2026-08-26 | CHANGE | side-by-side in the schema | two-or-more panels now required in `<sidebyside>` |
| 2026-08-21 | CHANGE | Ordered lists as exercise "parts", default numbering | numbering cycle adjustment |
| 2026-08-19 | CHANGE | Schema validation, arguments | production schema now paired with a separate "dev" schema add-on so authors/devs can opt into experimental constructs without destabilizing the production grammar |
| 2026-08-08 | NEW/CHANGE | Vertical alignment of "slide" content | |
| 2026-08-07 | UPDATED | Slide Shows | schema + Guide doc expanded |
| 2026-08-04 | NEW | Project-like printouts | |
| 2026-08-04 | NEW | Built-in screen reader (announce) | |
| 2026-08-04 | NEW | Table Notes, `#tn` | new element |
| 2026-08-03 | CHANGE | Managed directories | WeBWorK representations removed from a managed dir (exact scope unconfirmed — search re-fetch failed to relocate this thread; flagged as low-confidence) |
| 2026-07-31 | CHANGES | `#docinfo` and publisher variables | `docinfo/cross-references/@text` → `docinfo/defaults/xrefs/@text`; **old form still works, with a deprecation warning** — direct quote confirms the graceful-migration pattern |
| 2026-07-28 | CHANGE | Runestone database identifiers | `@label` introduced as an alternative to `@xml:id` |
| 2026-07-27 | CHANGE | Numbering options, esp. "project-like" | legacy numbering options removed/consolidated |
| 2026-07-26 | CHANGE | Hypothes.is via publisher file | |
| 2026-07-23 | NEW | Upgrades to Reveal.js slideshows | |
| 2026-07-22 | CHANGE | Interactives and side-by-sides | audio/video/interactive restricted from nesting inside `sidebyside` for static output |
| 2026-07-20 | UPDATE | PreTeXt-tools v1.0.0 | includes validator improvements |
| 2026-07-10 | UPDATED | Offline MathJax → v4 | |
| 2026-07-08 | CHANGE | Division companions for traditional divisions | intro/conclusion elements for chapters/sections |
| 2026-06-30 | CHANGE | Reserved labels | identifiers reserved for HTML functionality |
| 2025-11-05 | DEPRECATE | Hacked-in Runestone HTML | code supporting the old approach removed |
| 2025-11-03 | DEPRECATE | `@permid` attribute | an experiment-only attribute, now deprecated |
| 2025-08-26 | (dev list) | Deprecated `@focused` and `@preexpanded` levels | |
| 2025-03-08 | CHANGE | Commentary element | now deprecated; migrate to `@component` attributes |
| 2025-03-05 | CHANGE | Extensible arrows | `extpfeil` package backward-compat removed |

**Churn estimate:** roughly **one `[CHANGE]`/`[DEPRECATE]`/`[NEW]` announce-list notice
every 1–2 weeks** over the sampled window — i.e., dozens per year. Despite that
frequency, the actual **breakage rate for markup an author wrote a year ago is low**:
almost every change ships with a deprecation warning and a working fallback (the
`#docinfo` migration above is the clearest example — old path "still works, with a
deprecation warning"). The exceptions are constructs already flagged as experimental in
`schema/experimental-features.xml` (WeBWorK/Runestone dynamic-exercise vocabulary:
`setup`, `evaluation`, `eval`, `areas`/`area`, `blocks`/`block`, `matching`, interactive
`program`, etc.) — those *can* break without a deprecation cycle because they were never
promised stable. **Practical rule for the plugin:** treat production-schema markup as
low-churn/safe across a year; treat anything the current `experimental-features.xml`
names as high-churn and flag it to the author.

## (b) Design discussions a plugin should track or could inform

- **Calendar / date-aware authoring** — "Design Discussion: Calendar and date-aware
  authoring for course materials" (Oscar Levin, Sean Fitzpatrick et al., Aug 27, 2026).
  Early-stage: "an automated calendar (probably implemented as a `tabular`) is
  something the dates work might lead to." Relevant to a plugin that scaffolds course
  syllabi/schedules — watch this thread for the eventual element/attribute shape before
  building calendar tooling.
- **"LaTeX No LaTeX"** (Rob Beezer, Mitch Keller, Aug 27, 2026) — thread title suggests a
  LaTeX-free authoring/workflow discussion; only a narrow fragment was retrievable
  ("there is no way I saw everything. Fixed now (also for #book), so do a pull and take
  it for a spin before adding to your campus repository" — reads like a bug-fix
  follow-up inside the thread, not the design pitch itself). Flagged for a deeper
  read before the plugin design finalizes its stance on LaTeX-free/math-lite authoring
  paths; this session's summarizing fetch could not surface the top-level design post.
- **Schema validation** (Rob Beezer, Aug 19, 2026; matches the announce `[CHANGE]`
  above) — production schema now has a paired "dev" schema so authors/tooling can
  opt into experimental constructs deliberately rather than by accident. Directly
  relevant to the plugin's validation story: a hook or skill step that runs
  `pretext validate` should default to the **production** schema and only opt into
  the dev schema when the author explicitly wants bleeding-edge elements.
- **Built-in screen reader** (announced 2026-08-04; discussed further in a dev
  thread, Oscar Levin / valeriom, Aug 18) — voice selection follows document
  `@language`; open question whether spoken *commands* (not just content) will be
  translated too. Relevant to the plugin's accessibility guidance.
- **Image zoom (and interaction with PreFigure)** (Andrew Scholer, Rob Beezer et al.,
  13 participants, Aug 25, 2026) — motivated by Asymptote images getting clipped by
  their bounding box on rotation; proposal is a zoom/expand-to-new-window affordance,
  especially for 3D figures. Relevant to the plugin's PreFigure/Asymptote guidance and
  to any image-authoring template.
- **"Claude is a good PreTeXt reference tool"** (Bradley Miller, Rob Beezer, Jul 19,
  2026) — Bradley Miller: "That's how Claude knows what it knows. I have been very
  careful, more careful than elsewhere, to be sure the publisher switches are
  documented." Signal that maintainers actively write documentation *with* the
  expectation that LLMs will consume it — supports designing the plugin's reference
  material to lean on the Guide's publisher-switch docs rather than re-deriving them.
- General pattern across many 2026 threads (see AI/LLM section below): maintainers are
  already using Claude for day-to-day PreTeXt development work (schema design, XSL,
  numbering, docs, conversions) — the plugin isn't introducing AI to this community,
  it's formalizing a practice already underway.

## (c) Authoring practices maintainers repeatedly ask for or warn against

- **Validate before reporting a bug / before committing.** The `[CHANGE] Schema
  validation, arguments` and "Validation improvements" threads, plus the frequent
  "Thousands of warnings" thread (gedeoned…@gmail.com / Rob Beezer, 2024-03-01), show
  a recurring maintainer ask: run `pretext validate` (or the equivalent schema check)
  and fix warnings rather than let them accumulate silently. Directly matches Rob
  Beezer's ask in the "Claude skill" thread: "have Claude iterate with that tool until
  it got something error-free."
- **Don't hand-roll what a publisher-file switch already does.** Recurring across
  threads (docinfo/publisher variables, Hypothes.is via publisher file, print/CSS
  options) — maintainers steer authors toward `publication.ptx` switches instead of
  custom XSL/CSS overrides. Bradley Miller's comment about being careful to document
  "publisher switches" specifically (so Claude/authors find them) reinforces this.
- **Migrate off deprecated paths promptly, but they will keep working for a while** —
  every `[CHANGE]`/`[DEPRECATE]` notice sampled paired the new form with a working old
  form plus a warning; maintainers don't yank things overnight, but they do want authors
  to move rather than suppress the warning.
- **Don't treat the dynamic/interactive exercise vocabulary as stable.** Rob Beezer's
  `experimental-features.xml` prose is explicit and repeated across many entries: this
  vocabulary is "under active development in concert with the Runestone platform" and
  "among the most likely parts of PreTeXt to change" — authors leaning on Runestone-tied
  interactivity should expect churn.
- **Report with a minimal, buildable reproduction.** Implicit across the "bug"-shaped
  threads (Asymptote bug, dark-mode link visibility, tabular-in-figure, HTML id
  collisions for WeBWorK) — maintainers ask for/triage from small repros, not whole
  books.

## (d) Experimental/deprecated elements the skill must warn about

Straight from the local clone's `vendor-pretext/schema/experimental-features.xml`
(current, authoritative — this file is explicitly the "survey of experimental
constructs that validation produces," compared automatically against the production
RELAX-NG grammar, so an entry here is real and current as of the clone's date):

| Elements / attributes | Why flagged |
|---|---|
| `interactive`, `slate`, `instructions` | Large per-platform attribute vocabulary (GeoGebra, Desmos, CircuitJS, JSXGraph, iframe) still evolving; static-representation mechanism for print/PDF unsettled |
| `eval` | Dynamic/randomized-exercise value substitution; under active co-development with Runestone |
| `setup`, `variable`, `formula`, `condition`, `evaluation`, `evaluate`, `test`, `de-object`, `de-random`, `de-number`, `de-expression`, `de-evaluate`, `jsimports`, `jslibrary`, `config-json`, `setupScript`, `postRenderScript` | Whole dynamic-exercise machinery; "among the most likely parts of PreTeXt to change" |
| `choices`, `choice`, `feedback` | Multiple-choice/True-False; markup "reasonably stable in practice" but attributes/feedback placement still subject to change |
| `blocks`, `block` | Parsons problems; language/indentation markup still moving |
| `matching`, `matches`, `match`, `premise`, `response`, `cardsort` | Matching/card-sort pairing markup still moving |
| `areas`, `area`, `select`, `itself`, `for`, `logic`, `numcmp`, `strcmp`, `mathcmp`, `jscmp` | Fill-in-the-blank comparison/evaluation vocabulary |
| `fillin` (element itself standard; certain attributes experimental) | mode/name-binding/width attributes are dynamic-exercise, not the element |
| `var` (outside WeBWorK) | Standard inside WeBWorK; experimental when used to bind a fill-in blank |
| `program`, `datafile`, `query`, `program-preamble`, `program-postamble` | ActiveCode/coding exercises, Runestone-aligned |
| `exercise`/`task`/`statement`/`solution` (certain attributes only) | Experimental switches (language, adaptive behavior), not the elements themselves |
| `dynamic`, `static` | Alternate-representation-for-non-interactive-output mechanism unsettled |
| `solutions` (certain attributes) | Finer-grained solution-visibility control still experimental |
| `myopenmath` | Functional but narrow; markup may change as integration matures |
| `stack` | STACK/Moodle question embedding; developing with server integration |
| `shortlicense`, `website`, `address` | Structured front-matter for bibliographic info still being designed |
| `argument`, `justification`, `reasoning`, `explanation` | Alternate "proof"-like elements; final naming/list still under discussion |

Deprecated (confirmed via announce/dev threads, not in the experimental file since
these already have replacements):
- `docinfo/cross-references/@text` → replaced by `docinfo/defaults/xrefs/@text` (old
  form works with a deprecation warning, 2026-07-31).
- `commentary` element → deprecated in favor of `@component` attributes (2025-03-08).
- `extpfeil`-package-based extensible-arrows backward compatibility → removed
  (2025-03-05).
- `@permid` attribute → deprecated, was experiment-only (2025-11-03).
- `@focused` / `@preexpanded` levels → deprecated (2025-08-26 per dev-list mention).
- "Hacked-in Runestone HTML" support code → removed (2025-11-05).

**Caveat:** the experimental-features file is generated by diffing two RELAX-NG
grammars in the *current* clone; since the local clone is shallow (depth 1), this
table reflects the schema's state as of 2026-08-27 only — it cannot show what was
added to or removed from this list over the past year, only what's on it *now*. Treat
it as "the list to check before every release," not a historical record.

## (e) Named maintainers per topic area (who to ask)

| Topic area | Name(s) seen owning/answering it repeatedly |
|---|---|
| Schema design, core PreTeXt language, releases, general dev-list traffic | **Rob Beezer** (project lead; author of nearly every `[CHANGE]`/`[DEPRECATE]` notice and `experimental-features.xml`) |
| Printouts, print/PDF layout, slide shows, course-materials tooling (PreTeXt-tools), calendar/date-aware authoring | **Oscar Levin** |
| WeBWorK integration, exercise/webwork ids, MathJax-for-WeBWorK | **Alex Jordan** |
| Runestone platform integration, person/assignment pages, JSON/XML question representations | **Bradley Miller** (Runestone side) |
| Accessibility, braille/tactile graphics, image descriptions, PreFigure captions | **David Austin** |
| CSS/theming, interactives, ordered-list markers, consolidation of stylesheets | **Andrew Scholer** |
| HTML output bugs (dark mode, tabular-in-figure/margin, permalinks), schema/label identifiers | **Jeremy Sylvestre** / **Sean Fitzpatrick** (both recur across HTML/CSS bug threads) |
| LMS embedding / link behavior | **Mitch Keller** |
| MyOpenMath / STACK integration | **Mark Fitch** (MyOpenMath), **Michael Obiero** / **Georg Osang** (STACK) |
| WeBWorK build/tooling issues | **Alex Jordan**, cross-posts with **Rob Beezer** |

## (f) AI/LLM-related discussion

### The origin thread — "Claude skill for pretext authoring"
`https://groups.google.com/g/pretext-dev/c/0D90duVchZg` — Rick Roesler, Rob Beezer.

- Rick Roesler (Aug 28, 2026, 10:23 PM): "Has anyone built this?"
- Rick Roesler (Aug 28, 2026, 11:28 PM): offered to build Claude skills for PreTeXt
  authoring, describing skills as "typically focused on a single task and triggered by
  a small set of utterances," and mentioned building it for his own algebra-based
  physics book.
- Rob Beezer (Aug 28, 2026, **11:02:44 PM** — timestamp confirmed on repeat fetch):
  "Not that I am aware of." And, on his own accumulated practice using Claude:

  > **"I imagine I have a pile of useful rules anhd hints accumulated."**

  (Verbatim, including the typo "anhd" for "and" — confirmed identically across two
  independent re-fetches with an explicit no-paraphrase instruction.) This is the
  quote the map's "Not yet specified → Contribution path for Rob's accumulated Claude
  rules/hints" item refers to; it establishes that Rob has an *informal, personal*
  collection, not a published one — the plugin design needs to invent the mechanism
  for him to contribute it (a PR template, an issue, a direct file drop into a
  `references/` directory, etc.), not just ask for a link.

- Rob Beezer (following message, ~12:02 AM): "Don't forget the schema and all of the
  example documents — I think those might help where the 'real' documentation is not
  so good." And on validation: "The pretext/pretext script will run validation (versus
  the schema... I would think you could have Claude iterate with that tool until it
  got something error-free."

These two asks (bake in schema + examples; validate-and-iterate loop) are already
recorded verbatim in the map (issue #1 Notes). This survey adds the "pile of rules and
hints" quote and confirms it as Rob's own words, not a paraphrase.

### Other AI/LLM-adjacent traffic on pretext-dev (search: claude/chatgpt/LLM/AI)

The dev list shows routine, already-normalized use of Claude by maintainers
themselves, mostly as an aside inside otherwise-technical threads (this session's
fetch surfaced short paraphrased mentions, not full quotes, for most of these — treat
line items below as leads to re-read in full, not settled facts):

- **"Claude is a good PreTeXt reference tool"** (Bradley Miller, Rob Beezer, Jul 19,
  2026) — see quote in section (b) above; the closest thing to a second AI-focused
  thread besides the origin one.
- "labeled images in cardsort problems" (Aug 27) — noted Claude "automatically added
  markup elements without explicit instruction" while helping someone.
- "tabular inside figure" (Aug 25), "HTML ids... webwork exercises" (Aug 21), "Docinfo
  changes" (Aug 2), "redundant files in js/ and js/dist/" (Aug 1) — Rob Beezer
  describes using Claude as part of his own schema/XSL/build work.
- "Recording student work without runestone or scorm" (Jun 1) raises a *concern*
  (not endorsement) about LLMs having access to open textbook content — worth reading
  in full given the plugin will read/write book source; flagged, not confirmed in
  detail this session.
- "Replacing node dependency" (Jul 29) mentions, as an aside/anecdote, "AI agents
  completing [a] Rust project in 11 days costing $165,000" — general AI-coding-cost
  color, not PreTeXt-specific; low relevance.
- Several more (Validation improvements, Braille conversion progress, Additions to The
  Guide, Numbers, Formatting of HTML output, Numbering consistency with LaTeX,
  WeBWorK in the pre-processor) each contain a brief "thanks Claude for..." or
  "Claude suggested..." aside per the search summary — consistent picture of Claude
  already being an informal pair-programmer for the maintainers, but none were read in
  full this session; re-fetch individually before quoting any of them.

**Net read for the plugin design:** the community — starting with the project lead —
is already receptive to and actively using Claude for PreTeXt work. The plugin isn't
persuading skeptics; it's formalizing/packaging a practice several maintainers already
do ad hoc, with Rob Beezer specifically wanting (1) schema + examples baked in and (2)
a validate-and-iterate loop, and holding an informal personal rule set he hasn't yet
found a way to contribute.
