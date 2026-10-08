---
title: "Introducing AzeForge and AzeMark: one source, four publishing formats"
description: "I built AzeMark for readable technical documents and AzeForge to compile them into self-contained HTML, SVG, PNG, and PDF. Here is how to write a first document and try the tools."
date: 2026-10-08
tags: [azeforge, azemark, technical-writing, open-source]
image: /assets/img/azeforge-linkedin-card.png
---

Last week I published [@aruzone/aze-forge](https://www.npmjs.com/package/@aruzone/aze-forge), together with AzeMark, a readable technical document language I wrote. Both are open source under the [MIT License](https://github.com/aruzone/aze-forge/blob/main/LICENSE).

The idea is straightforward. Write the explanation, equations, diagrams, and data in one readable source. Compile that source into the formats you need to share it, without rebuilding the document for each destination.

[AzeForge](https://azeforge.com) produces deterministic, self-contained HTML, SVG, PNG, and PDF artifacts. AzeMark is the language you write. AzeForge is the compiler that validates it and produces the output.

<!--more-->

<figure class="figure-wide">
  <img src="{{ '/assets/img/azeforge-linkedin-card.png' | relative_url }}" alt="AzeForge launch artwork with its red logo and HTML, SVG, PNG, and PDF output formats" width="1200" height="627">
  <figcaption>AzeMark source becomes a self-contained artifact for the web, an image, or print.</figcaption>
</figure>

## Why another technical document language?

Markdown is a good starting point for technical writing. Headings, paragraphs, lists, and code are easy to read in a text editor. The difficulty starts when a document needs a derivation, a circuit, a geometry construction, or a plot alongside the prose.

Those pieces often live in separate tools. You export an image, insert it into a document, then repeat the export when something changes. Publishing a web version and a printable version adds another place for the material to drift.

I wanted the technical content to stay in the source, where it can be edited and reviewed with the explanation. AzeMark extends Markdown with structured, typed blocks. An equation is an equation block. A plot describes its series and axes. A circuit describes components and connections, rather than pointing to a screenshot of a schematic.

This also gives the compiler something to check. Unknown fields and broken references become diagnostics instead of silently disappearing into the output. That validation checks the notation and structure, not whether an engineering assumption or scientific conclusion is correct. The author still owns those decisions.

## What you can write in AzeMark

Ordinary Markdown remains the prose layer. Structured blocks cover the material that needs more than a paragraph or a code fence:

- Mathematics, including equations and step-by-step derivations.
- Diagrams, including flowcharts, graphs, trees, architecture diagrams, sequence diagrams, and state models.
- Data visualization, including function plots, measured series, and charts.
- Geometry constructions, chemical formulas, reactions, and molecular structures.
- Engineering notation, including circuits, digital timing, control diagrams, and free-body diagrams.
- Document content, including typed tables, algorithms, worked examples, statements, bibliographies, figures, and callouts.

The [AzeMark authoring guide](https://github.com/aruzone/aze-forge/blob/main/docs/language/00-authoring-azemark.aze.md) explains the block syntax. The [category guides and examples](https://github.com/aruzone/aze-forge/tree/main/docs/language) contain complete documents you can read, copy, and compile.

## A first document you can compile

Here is a small engineering note. Save it as `resistor-note.aze.md`:

```text
---
azemark: 2
title: A resistor measurement
---

# A resistor measurement

For a 220 ohm resistor with 5 V across it, Ohm's law gives
an expected current of about 22.7 mA.

:::: equation
id: ohms-law
----
V = I * R
::::

:::: callout
variant: note
title: Measurement conditions
----
Record the resistor tolerance and temperature before comparing
this estimate with a measured current.
::::
```

The `azemark: 2` front matter declares the language version. Each document-level block opens with exactly four colons and a type, uses `----` to separate its header from its body, and closes with four colons. The equation body uses readable mathematical notation. It does not require raw LaTeX.

Install the CLI with Node.js 22 or 24 on macOS or Ubuntu. On npm 11 and later, allow the Puppeteer install script so it can fetch the pinned browser engine used for visual rendering:

```bash
npm install -g @aruzone/aze-forge --allow-scripts=puppeteer
azeforge validate resistor-note.aze.md
azeforge render resistor-note.aze.md --output resistor-note.html
```

On older npm versions, use `npm install -g @aruzone/aze-forge` without the `--allow-scripts` option. The [installation guide](https://github.com/aruzone/aze-forge/blob/main/docs/development.md#browser-engine-and-offline-installs) covers the browser dependency and offline setup. Windows support is not currently available.

Successful validation is silent. Open `resistor-note.html` to read the rendered note. To produce the other formats, keep the source and change the output extension:

```bash
azeforge render resistor-note.aze.md --output resistor-note.svg
azeforge render resistor-note.aze.md --output resistor-note.png
azeforge render resistor-note.aze.md --output resistor-note.pdf
```

For a local browser preview that updates as you edit:

```bash
azeforge serve resistor-note.aze.md --port 0
```

The command prints the local URL to open.

## Add a plot without maintaining a separate image

Suppose a teaching note needs to explain a quadratic function. Add this block to the same document:

```text
:::: plot
id: quadratic
x-axis:
  label: x
y-axis:
  label: y
----
- kind: function
  expression: x^2
  domain:
    min: -2
    max: 2
::::
```

The source specifies the function and its domain, and AzeForge renders the plot. If you change the expression or the range, the next render updates it in the output. There is no separately exported plot image to replace.

For measured data, the plot family also accepts authored line and scatter series. The [visualization examples](https://github.com/aruzone/aze-forge/blob/main/docs/language/03-visualization.aze.md) cover cooling measurements, an RC step response, error bars, and logarithmic axes.

## What the compiler adds

### Repeatable output

AzeForge is designed for deterministic builds. Its documented contract is byte-identical artifacts for the same source under the same compiler and rendering setup. Each artifact carries the content hash of the document it represents.

For reports and review workflows, that makes output changes traceable to an input or toolchain change. Pin the compiler version and rendering environment when reproducibility matters. Do not assume that changing versions or engines will leave the bytes unchanged.

### Files that do not need the publishing tool

The output is self-contained. Fonts are embedded, scripts are not emitted, and artifacts do not make network requests. The compiler and its rendering dependencies stay on the machine doing the build, rather than travelling with the document.

HTML works for a standalone web document. SVG provides a vector image. PNG provides a raster image for places that expect one. PDF provides a printable document. These formats serve different destinations, but they come from the same source.

### Errors before publication

Invalid source produces diagnostics with source ranges. AzeForge refuses to replace an existing artifact with partial output when compilation fails. That matters when a publishing command runs repeatedly while a document is being edited.

For editor or automation integrations, validation can return JSON diagnostics:

```bash
azeforge validate resistor-note.aze.md --diagnostics json
```

The package also exposes a Node.js compiler API. The [README](https://github.com/aruzone/aze-forge#library) documents the supported entry points if you want to integrate compilation into an existing publishing system.

## Where I would use it

### Technical documentation and engineering notes

Keep an architecture diagram beside its explanation, or a circuit beside the equations and assumptions used to discuss it. Reviewers can inspect the technical declarations along with the prose. Publish HTML for reading online and PDF for a review packet.

### Educational material

Write a lesson with a derivation, a geometry figure, a function plot, and a worked example. Use the same source for a web handout and a printable copy. When an exercise changes, rebuild both rather than editing each version independently.

### Scientific content and reports

Combine formulas, reactions, measured plots, typed tables, and references in one document. Export a figure as SVG or PNG for a presentation, or render the report as PDF. AzeMark expresses the technical content; it does not replace the measurements, analysis, or domain review behind it.

The common requirement is one readable source that needs to produce several self-contained artifacts consistently.

## Try the browser playground

I also published [azeforge.com](https://azeforge.com) with [documentation](https://azeforge.com/docs/0.6.4/), examples, and a [browser playground](https://azeforge.com/playground).

In the playground, you can describe what you need in plain English, review the generated AzeMark, edit it, and render the result as PDF, SVG, PNG, or HTML. For example, ask for a short teaching note on Ohm's law with an equation and a measurement callout. Inspect the source before exporting it, especially any values or assumptions the generator introduces.

The hosted playground currently requires an access token. If you want to work locally, the npm package and `azeforge serve` provide a separate route. The published machine-readable grammar also gives generators an explicit syntax to target, but generated content still needs review.

<figure>
  <img src="{{ '/assets/img/azeforge-logo-transparent.png' | relative_url }}" alt="AzeForge red geometric A logo" width="1254" height="1254" loading="lazy" style="width: 160px; max-width: 100%; margin-inline: auto; border: 0;">
</figure>

## Start with one document

Take a technical note you already maintain. Put its prose in AzeMark, add one equation or plot, and render it as HTML and PDF. That is enough to see whether keeping the content in one source fits your workflow.

- [Read the documentation](https://azeforge.com/docs/0.6.4/).
- [Explore the complete AzeMark examples](https://github.com/aruzone/aze-forge/tree/main/docs/language).
- [Try the playground](https://azeforge.com/playground).
- [Install @aruzone/aze-forge from npm](https://www.npmjs.com/package/@aruzone/aze-forge).
- [Read the source or report an issue](https://github.com/aruzone/aze-forge).

More improvements and capabilities are planned for future releases. If you try AzeForge, share the kind of document you are writing and where the language or output gets in the way. A concrete example is the most useful feedback.
