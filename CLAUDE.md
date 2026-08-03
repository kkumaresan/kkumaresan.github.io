# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for kkumaresan.com, hosted on GitHub Pages. As of the
`v2` branch the site is a **Jekyll** site (it was previously a single static
`index.html`; that simplification was reversed deliberately to gain layouts,
includes, and a blog).

## Development

Local preview requires Ruby >= 2.7. macOS system Ruby is 2.6, so install a newer
one first (`brew install ruby`, or rbenv), then:

```
bundle install
bundle exec jekyll serve      # http://localhost:4000
```

`bundle exec jekyll build` writes to `_site/` (git-ignored).

Deployment is unchanged from v1: push the branch, and GitHub Pages builds it
server-side. The build targets **GitHub Pages' native Jekyll**, so only
whitelisted plugins may be used (`jekyll-feed`, `jekyll-seo-tag`,
`jekyll-sitemap` are configured). Adding an unsupported plugin requires
switching to a GitHub Actions build, which also requires changing the Pages
source in repo settings.

There is **no `.nojekyll`** file — it was removed, since it disables the Jekyll
build. Do not reintroduce it.

## Architecture

```
_config.yml              Site config, plugins, permalinks (/writing/:title/)
Gemfile                  Pins the github-pages gem set for local/prod parity
_layouts/default.html    Shell: <head>, .sheet wrapper, topbar, footer
_layouts/post.html       Article layout (meta rail, title, .prose body)
_includes/head.html      Meta, Google Fonts, CSS, favicon, seo/feed tags
_includes/topbar.html    Primary nav
_includes/hero.html      Name, metadata grid, portrait
_includes/footer.html    CTA + contact
_includes/sections/*     The six numbered home-page sections
index.html               Home — front matter + include calls only
writing/index.html       Blog index (permalink /writing/)
_posts/*.md              Blog posts
assets/css/main.css      All styling (plain CSS, no Sass)
profile.jpg  CNAME       Portrait; custom domain
```

Content lives in the section includes, not in `index.html` — edit
`_includes/sections/*.html` to change home-page copy.

## Design System (assets/css/main.css)

Direction is "Systems Dossier": a technical specification sheet rendered in
near-black. Structure is exposed rather than decorated.

- **Palette** — near-black ground (`--bg` #000, `--surface` #0A0A0A), hairline
  rules (`--line` #1D1D1D), ink (`--text` #EDEDED, `--muted` #A1A1A1,
  `--faint` #7A7A7A). Deliberately monochrome: white is the accent. `--link`
  (#3D8BFF) is the only chromatic value and is reserved for interactive states.
- **Type** — Playfair Display (display), Geist (body), Geist Mono (labels,
  numerals, metadata). Loaded from Google Fonts. Playfair is a wide face, so
  display elements use `text-wrap: balance` and body copy `text-wrap: pretty`
  to avoid orphans; prefer that over shrinking the type scale.
- **Layout** — every section is a two-column grid: a `--rail` gutter holding the
  plotted section numeral, and the content column capped at `--measure` (66ch).
  The rail collapses below 900px.
- **Motion** — one staggered load reveal (`.reveal`), plus scroll-linked reveals
  behind `@supports (animation-timeline: view())` so unsupporting browsers
  render content visible rather than blank. `prefers-reduced-motion` is honoured.

Contrast note: `--faint` is set at #7A7A7A specifically to clear 4.5:1 on black
at the 10.5–11px sizes used for mono labels. Do not darken it.

The `body::before` glow is `position: absolute`, anchored to the top of the
document rather than the viewport — deliberate, so the light source does not
trail the reader down the page. Do not switch it back to `fixed`.

## Image Handling

Profile image can be resized/optimized with macOS `sips` command (pre-authorized in `.claude/settings.local.json`).
