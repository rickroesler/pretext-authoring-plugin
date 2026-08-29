# Interactives and Media

The `<interactive>` element embeds live, manipulable content in HTML output, with an
automatically screenshotted static image standing in for it in PDF, EPUB, and Kindle.

> **Status:** `<interactive>` is one of the five fragments in the development-schema
> overlay (`interactive`, `exercise-dev`, `solutions-dev`, `proof-like`,
> `frontmatter-dev`), so `pretext validate` lists it as experimental rather than as an
> error. See [10-schema.md](10-schema.md) for what that classification means. It has rendered in HTML for years and is
> widely used; the per-platform attributes and the static-representation mechanism are
> what may still shift.
>
> **The Guide's chapter on this is a TODO.** The real documentation is
> `examples/sample-article/sample-article.xml` in the core repo — search it for
> `<interactive`. Everything below is taken from there.

## The three ways to specify one

1. **Shorthand attribute** — `geogebra="…"`, `desmos="…"`, `circuitjs="…"`, `calcplot3d="…"`
2. **`iframe="…"`** — any URL or local HTML file the vendor lets you embed
3. **`platform="…"` plus `<slate>` children** — for `geogebra`, `jsxgraph`, `d3`,
   `javascript`, `sage`, `doenetml`, where you supply code

Common attributes: `xml:id` (needed for the generated preview file), `label` (stable
reader-facing id), `width` (percentage), `aspect` (`"15:10"` or a bare number like `1.5`),
`design-width` (force a pixel width), `resize-behavior`, `preview` (your own static image),
`source` (space-separated list of JS/CSS files), `css`.

Children: `<description>` (accessibility — required in practice), `<instructions>` (shown
to the reader below the frame), `<slate>`, `<static>`.

Always wrap in a `<figure>` with a caption so it is numbered and cross-referenceable.

**An `<interactive>` may never be a `<sidebyside>` panel** — nor may `<video>` or
`<audio>`, nor a `<figure>` that occupies a panel contain one, nor a `<stack>`. The reason
is mechanical: static conversions render each of these *as* a `sidebyside` (preview image +
QR code + links), and side-by-sides must not nest. So two simulations cannot sit next to
each other; stack them as consecutive figures. The schema blocks the direct cases and the
validation-plus stylesheet checks arbitrary depth.

## PhET — the workhorse for physics

PhET simulations embed as an iframe. Take the URL from the simulation's "Embed" option, or
build it from the sim name.

```xml
<figure xml:id="fig-phet-forces">
  <caption>Explore net force and acceleration.</caption>
  <interactive xml:id="phet-forces"
    iframe="https://phet.colorado.edu/sims/html/forces-and-motion-basics/latest/forces-and-motion-basics_en.html"
    width="95%" aspect="15:10"
    preview="preview/phet-forces.png">
    <description><p>A simulation of a tug-of-war: pullers of adjustable strength on each
    side of a cart, with readouts for the net force and the resulting acceleration.</p></description>
    <instructions><p>Add pullers to each side and watch the net force readout before
    releasing the cart.</p></instructions>
  </interactive>
</figure>
```

Give `preview` a hand-taken screenshot when you can. Otherwise the build screenshots the
page with a headless browser — which for PhET usually captures the loading screen. Same
applies to any `iframe` interactive that animates on load.

## GeoGebra

Three routes, in increasing order of control:

```xml
<!-- 1. A published material, by ID from geogebra.org/m/KGn2d4Qd -->
<interactive xml:id="ggb-triangle" geogebra="KGn2d4Qd" width="70%" aspect="9:5"
             reset-icon="yes">
  <description><p>…</p></description>
</interactive>

<!-- 2. A material, then modified through the GeoGebra Apps API -->
<interactive xml:id="ggb-train" platform="geogebra" width="75%" aspect="2:1">
  <description><p>…</p></description>
  <slate xml:id="train" surface="geogebra" material="D4s2v4ft" aspect="2:1">
    setAxisSteps(1, 1, 5, 1);
    setCoordSystem(-0.7, 8, -6, 56);
    enableShiftDragZoom(false);
  </slate>
</interactive>

<!-- 3. Your own .ggb file from the external directory -->
<interactive xml:id="ggb-cycloid" platform="geogebra" width="100%" aspect="2:1">
  <slate xml:id="cycloid" surface="geogebra" source="code/epicycloid.ggb" aspect="2:1">
    setCoordSystem(-15, 5, -5, 5);
    setAnimating("θ", true);
  </slate>
</interactive>
```

One API command per line. `base64="…"` accepts a `.ggb` encoded with Ctrl-Shift-B in the
desktop app, in place of `source`.

Control attributes on `<interactive>`: `toolbar`, `algebra-input`, `reset-icon`,
`shift-drag-zoom`, `zoom-buttons` (each `yes`/`no`), and `material-width` /
`material-height` in pixels to declare the material's internal coordinate size.

Gotcha: PreTeXt resizes the viewport anchored at the top-left, which does **not** rescale
axes — content can fall outside the frame. Call `setCoordSystem(...)` in the slate to put
it back.

Licence caution: materials on geogebra.org carry a non-commercial licence you accept
before downloading the source. Check it before shipping one in a book you sell.

## JSXGraph — for diagrams you write yourself

```xml
<interactive xml:id="kinematics" platform="jsxgraph" aspect="6:7"
             source="code/kinematics.js">
  <description><p>A position–time graph with a draggable point; the tangent line and its
  slope update as the point moves.</p></description>
</interactive>
```

The JS file lives in the external directory. This is the best option for a bespoke physics
widget — a phasor, a wave superposition, a draggable free-body diagram — since you own the
code and it has no external dependency at read time.

## Desmos, CalcPlot3D, CircuitJS

```xml
<interactive xml:id="desmos-decay" desmos="ttox1bvxku" width="60%" aspect="2:3"/>

<interactive xml:id="cp3d-wavefunction" calcplot3d="type=z;z=sin(api*x)sin(bpi*y);…"
             variant="controls" width="95%" aspect="3:2"/>

<interactive xml:id="circuit-rc" circuitjs="$ 1 0.000005 …" width="90%" aspect="4:3"/>
```

`circuitjs` is a full spice-like circuit simulator — the right tool for a
circuits chapter. Build the circuit at falstad.com and export its URL fragment.
`calcplot3d` takes a long semicolon-and-ampersand-separated state string;
`variant` is `minimal`, `controls`, or `application`.

## Sage cells and code

```xml
<sage>
  <input>
    t = var('t')
    plot(20*t - 4.9*t^2, (t, 0, 4), axes_labels=['t (s)', 'y (m)'])
  </input>
  <output></output>
</sage>
```

Executes in the reader's browser against the public Sage Cell Server. Consequences:
you cannot add packages (request them on the `sage-cell` Google group), and the cell can
only read files from an allow-listed set of hosts (GitHub raw, Dropbox).

`<program>` holds non-executed code listings; `interactive="activecode"` makes it runnable
(Python, JS, HTML, SQL in a plain build; C, C++, Java, Octave and more on Runestone),
`interactive="codelens"` makes it a step-through visualiser.

`<sage>` blocks with expected `<output>` can be doctested with
`xsl/pretext-sage-doctest.xsl` — worth wiring up if your book's computations must stay
correct across Sage releases.

## Video

```xml
<figure xml:id="fig-pendulum-video">
  <caption>Slow-motion footage of a pendulum.</caption>
  <video xml:id="pendulum" youtube="DJWEqYcxUl8" preview="pendulum-still.jpg"/>
</figure>
```

`youtube="ID"` (or `youtubeplaylist`, `vimeo`), or `source="movies/lab.mp4"` for your own
file in the external directory. Note that `xml:id` on a `<video>` is **not** a
cross-reference target — it names the generated preview ("poster") file, which is why it is
optional. Same for `xml:id` on a `latex-image`/`sageplot`/`asymptote`: it becomes the image
filename. `<audio>` is simpler: no preview, no aspect ratio, `source` or `href`. Thumbnails are downloaded from YouTube into
`generated/youtube/` automatically; supply your own `preview` to override.
`<video privacy="yes"/>` (the default) in the publication file suppresses tracking cookies.

## Static representations for print

Everything above degrades to an image in PDF/EPUB/Kindle:

- `preview="…"` — your image from the external directory. Best quality; use it for
  anything that matters.
- Otherwise Playwright screenshots the live page into `generated/preview/`.
- QR codes are generated into `generated/qrcodes/` and printed beside the static image so
  a PDF reader can reach the live version. This needs `<baseurl href="…"/>` under
  `<html>` in the publication file — **set it early**, or your print edition has no route
  back to the interactive content.
- `<static>` inside an `<interactive>` lets you author a bespoke replacement (a `figure`,
  a `tabular`) rather than accepting a screenshot.

Global layout: `<interactives resize-behavior="fixed-height">` under `<html>`, with
per-type overrides (`calcplot3d`, `circuitjs`, `d3`, `desmos`, `doenetml`, `geogebra`,
`iframe`, `javascript`, `jsxgraph`, `sage`).

## Practical advice

Every embedded third-party simulation is a dependency you do not control: URLs move,
Flash-era sims disappear, licences change. For content central to your book, prefer
JSXGraph or `latex-image` that you own. Reserve PhET and GeoGebra for enrichment, always
supply a `preview` and a real `<description>`, and keep `baseurl` set so print readers can
still get there.
