# Exercises and Assessment

## Anatomy

The simplest exercise is a question:

```xml
<exercise><p>Compute the escape velocity from the Moon.</p></exercise>
```

The moment you add a `hint`, `answer`, or `solution`, the question must be wrapped in
`<statement>`:

```xml
<exercise xml:id="ex-escape" label="ex-escape-moon">
  <title>Escape velocity</title>
  <statement><p>Compute the escape velocity from the Moon.</p></statement>
  <hint><p>Set the total mechanical energy to zero.</p></hint>
  <answer><p><m>2.4</m><nbsp/><quantity><unit base="meter" prefix="kilo"/><per base="second"/></quantity></p></answer>
  <solution><p><m>v = \sqrt{2GM/R}</m>; substitute the lunar mass and radius.</p></solution>
</exercise>
```

`answer` is the short result; `solution` is the route. Several of each are allowed.
Titles are strongly encouraged — in HTML an inline exercise is often born in a knowl whose
clickable text is the title.

### Multi-part

```xml
<exercise>
  <introduction><p>A 2 kg cart rolls down a 30° incline from rest.</p></introduction>
  <task><statement><p>Find the acceleration.</p></statement>
        <hint><p>Resolve gravity along the slope.</p></hint></task>
  <task><statement><p>Find the speed after 3 m.</p></statement></task>
  <conclusion><p>Compare with the frictionless prediction.</p></conclusion>
</exercise>
```

`<task>` nests up to three levels (the limit is LaTeX's list labels).

## Where exercises live

| Placement | Numbering | Notes |
|---|---|---|
| Inline in a division | shares the block counter (Example 5.2.65, Checkpoint 5.2.66) | interrupts the narrative |
| Inside `<exercises>` | its own sequence within the division | the end-of-section/chapter set |
| Inside `<reading-questions>` | own sequence | comprehension checks |
| Inside `<worksheet>` | own sequence | print layout, `workspace` attribute |
| `<project>` / `activity` / `exploration` / `investigation` | block counter, or its own | labs, extended problems |

`<exercisegroup>` wraps a run of divisional exercises under shared instructions
(a required `<introduction>`, optional `<conclusion>` and `<title>`) — exactly the
"Problems 12–20: a block of mass m…" pattern.

Divisional exercises number automatically. A `number` attribute pins one, but once you
override one you generally have to set every subsequent one; save it for a mature edition.

## Solutions: visibility and collection

Two independent mechanisms.

**Visibility where the exercise is born** — publication file, twenty switches
(four components × five exercise types), all defaulting to `yes`:

```xml
<common>
  <exercise-inline     statement="yes" hint="yes" answer="no"  solution="no"/>
  <exercise-divisional statement="yes" hint="yes" answer="no"  solution="no"/>
  <exercise-worksheet  statement="yes" hint="no"  answer="no"  solution="no"/>
  <exercise-reading    statement="yes" hint="yes" answer="no"  solution="no"/>
  <exercise-project    statement="yes" hint="yes" answer="yes" solution="yes"/>
</common>
```

**Collection elsewhere** — the `<solutions>` division, which regenerates chosen components
in a new location:

```xml
<!-- in backmatter: an appendix covering the whole book -->
<solutions divisional="answer" admit="odd">
  <title>Answers to Odd-Numbered Exercises</title>
</solutions>

<!-- at the end of a chapter: everything in that chapter -->
<solutions inline="solution" divisional="hint solution">
  <title>Hints and Solutions</title>
</solutions>
```

Attributes name the exercise type (`inline`, `divisional`, `reading`, `worksheet`,
`project`); the value is a space-separated list of `statement hint answer solution`.
`scope="<xml:id>"` retargets which division is scanned. `admit="odd|even|all"` filters by
serial number.

Placement decides scope: inside `backmatter` → an appendix over the whole book; inside a
chapter → a section-like division covering that chapter.

Keeping solutions out of a public repository: put them in a separate file named by
`publication/source/@private-solutions` and keep that file in a private sibling
repository. There is also a solution-manual stylesheet
(`xsl/pretext-solution-manual-latex.xsl`) for a standalone instructor volume.

## Interactive (auto-graded) exercises

Powered by Runestone components. They work in a plain HTML build — the reader gets
immediate right/wrong feedback locally. Hosted on a Runestone server they additionally
record progress in a gradebook. Every one of them also has a **static** rendering for
print/PDF/braille, with an `answer` and `solution` synthesised from the correctness data
and your `<feedback>`.

> **Status:** the whole interactive-exercise vocabulary is the `exercise-dev` fragment of
> the development-schema overlay: `choices`/`choice`, `feedback`, `statement/@correct`,
> `fillin/@answer|@mode|@name`, `evaluation`/`evaluate`/`test`,
> `numcmp`/`strcmp`/`jscmp`/`mathcmp`, `setup`/`setupScript`/`eval`, `matching`,
> `cardsort`, `areas`, `response`. Validation reports these as **experimental**, not as
> errors — they are legal and they build, but their markup may change without a
> deprecation cycle. `solutions/@scope`, `@reading` and `@worksheet` are experimental too
> (fragment `solutions-dev`), as is `proof` as a general block (`proof-like`).
> Keep `pretext validate` in your loop and read its experimental inventory.

Put a stable `label` on every interactive exercise before you publish. Runestone keys its
database on `document-id` + `edition` + `label`; changing a label orphans student records.

### True/False — signalled by `@correct` on the statement

```xml
<exercise label="ex-tf-momentum">
  <statement correct="yes"><p>Momentum is conserved in an isolated system.</p></statement>
  <feedback><p>Internal forces cancel pairwise by Newton's third law.</p></feedback>
</exercise>
```

### Multiple choice — signalled by `<choices>` after the statement

```xml
<exercise label="ex-mc-units">
  <statement><p>Which is a unit of force?</p></statement>
  <choices randomize="yes">
    <choice correct="yes">
      <statement><p>newton</p></statement>
      <feedback><p>Yes: <m>1\,\text{N} = 1\,\text{kg}\cdot\text{m}/\text{s}^2</m>.</p></feedback>
    </choice>
    <choice>
      <statement><p>joule</p></statement>
      <feedback><p>That is energy — force times distance.</p></feedback>
    </choice>
  </choices>
</exercise>
```

Several `correct="yes"` choices make it multi-select automatically;
`multiple-correct="yes"` on `<choices>` forces the multi-select presentation even with one
correct answer. Each `<choice>` must have both a `<statement>` and a `<feedback>` — the
feedback is what makes the static solution useful.

### Fill in the blank — `<fillin>` in the statement plus `<evaluation>`

```xml
<exercise label="ex-fill-speed">
  <statement>
    <p>A car travels <quantity><mag>100</mag><unit base="meter"/></quantity> in
       <quantity><mag>5</mag><unit base="second"/></quantity>; its speed is
       <fillin answer="20" mode="number"/> m/s.</p>
  </statement>
  <evaluation>
    <evaluate>
      <test correct="yes">
        <numcmp use-answer="yes" tolerance="0.1"/>
        <feedback><p>Right — <m>v = d/t</m>.</p></feedback>
      </test>
    </evaluate>
  </evaluation>
</exercise>
```

- `<fillin>` needs `answer` (static) or `ansobj` (dynamic), plus `mode` = `string`
  (default), `number`, or `math`. `width`/`characters` size the printed blank.
- One `<evaluate>` per blank, matched by order — or by `name` if each `fillin` has one.
- Comparisons: `<numcmp>` (`value` + `tolerance`, or `min`/`max`, or `use-answer="yes"`),
  `<strcmp>` (regex by default; `literal="yes"`, `case="insensitive"`, `strip="no"`),
  `<jscmp>` (arbitrary JS; the response is `ans`, all responses are `ans_array`).

`tolerance` is the reason this type is worth the trouble in physics — it accepts a range of
numerical answers rather than one exact string.

### Randomised problems — `<setup>`

A `<setup>` sibling of `<statement>` makes the problem dynamic: a `<setupScript>` of
JavaScript builds objects on a namespace `v`, seeded by `RNG` so each reader gets a stable
variant with a "new version" button. `<eval object="…"/>` interpolates a value into the
statement or solution. `seed` is required so static output is reproducible.

`RNG.random()`, `RNG.randSign()`, `RNG.randInt(a,b)`, `RNG.randUniform(a,b)`,
`RNG.randDiscrete(a,b,dx,nonzero)`.

Shared libraries via `<jsimports><jslibrary source="code/kinematics.mjs"/></jsimports>`.
They must be genuine ES modules using `export`, and must not touch `window` or `document`
— the static build runs them under Node, outside a browser. A library named by `url`
(remote) is refused unless the publication file approves it explicitly:

```xml
<dynamics>
  <remote-libraries><library url="https://example.org/vectors.mjs"/></remote-libraries>
</dynamics>
```

### Other types

`<response/>` after the statement → short-answer/essay box (only gradable on a Runestone
server; `attachment="yes"` on the exercise lets students upload a file).
`<program>` after the statement → coding exercise. Also Parsons problems, matching,
clickable-area, and Runestone-only timed assessments (`time-limit` on `<exercises>`) and
group work (`groupwork="yes"` on a `<worksheet>`).

## WeBWorK

For a physics course, WeBWorK is the mature path to randomised, unit-aware, numerically
tolerant homework. A `<webwork>` element goes inside an `<exercise>` (optionally between
`<introduction>` and `<conclusion>`).

Four ways to fill it:

```xml
<!-- 1. A problem from the Open Problem Library -->
<webwork source="Library/Rochester/setMechanics/ball-toss.pg"/>

<!-- 2. Perl-free, no randomisation -->
<webwork>
  <statement><p>The acceleration of gravity is <var name="'9.8'" width="5"/> m/s².</p></statement>
</webwork>

<!-- 3. Randomised, with PG code -->
<webwork>
  <pg-code>
    Context("Numeric");
    $v0 = random(10, 30, 1);
    $t  = random(2, 6, 1);
    $x  = Compute("$v0*$t - 4.9*$t**2");
  </pg-code>
  <statement>
    <p>A ball is thrown upward at <m><var name="$v0"/></m> m/s.
       Its height after <m><var name="$t"/></m> s is <var name="$x" width="10"/> m.</p>
  </statement>
  <solution><p><m>y = v_0 t - \tfrac12 g t^2</m>.</p></solution>
</webwork>

<!-- 4. An external .pg file, included as text -->
<webwork><xi:include parse="text" href="pg/projectile.pg"/></webwork>
```

`@name` on `<var>` is passed to PG's `Compute()`, so mind Perl semantics: `'8^2'` → 64,
but bare `8^2` is a bitwise XOR → 10.

Configure the server in the publication file:

```xml
<webwork server="https://webwork-ptx.aimath.org" course="anonymous"
         user="anonymous" password="anonymous" static-processing="webwork2"/>
```

`static-processing="local"` with `pg-location` pointed at a local `pg` checkout is much
faster if you have one. Then:

```bash
pretext generate webwork    # fetch static representations — required before any output shows
pretext build web
```

Which types stay live in HTML is a publisher choice:
`<webwork inline="dynamic" divisional="static" project="dynamic"/>` under `<html>`.

Caveat from the Guide: not every OPL problem is PreTeXt-compatible. Older macros do not
emit PreTeXt output, and the failure mode is either "did not return valid XML" or, worse,
valid-but-wrong output. Read the PDF with solutions exposed to check what you actually got.
