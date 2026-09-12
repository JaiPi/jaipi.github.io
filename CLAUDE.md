# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout across branches

**The checked-out `main` branch contains only `README.md`.** The actual site lives on other branches — do not assume the working tree is the site:

- `origin/hyde` — the real site: a Jekyll blog built on the [Hydejack](https://hydejack.com/) theme (Hydejack Starter Kit). This is where nearly all work happens.
- `origin/test-jekyll` — an abandoned earlier attempt (minima-style, bare `_config.yml` + `index.md`). Do not build on it.
- `main` — effectively empty.

Before editing site content, confirm which branch is checked out. To inspect the site without switching branches, use `git show origin/hyde:<path>`.

There is no CI workflow, Netlify config, or build script anywhere in the repo — GitHub Pages builds `jaipi.github.io` directly from the published branch.

## Commands (run from a checkout of `hyde`)

```bash
bundle install                 # requires Bundler: gem install bundler
bundle exec jekyll serve       # local dev server at http://localhost:4000/
JEKYLL_ENV=production bundle exec jekyll build   # production build into _site/
```

Ruby, Bundler, and Jekyll are **not installed** in this environment. Any command above will fail until Ruby + `gem install bundler` are set up; don't report a build as verified unless it actually ran.

Two behaviors differ between environments and only appear with `JEKYLL_ENV=production`:
- Built-in search is disabled in development to save build time.
- `compress_html` is skipped in development (`compress_html.ignore.envs: [development]`).

There is no test suite and no linter.

## Theme model: what you can and cannot edit

The theme is consumed as a **remote theme**, not vendored: `_config.yml` sets `remote_theme: hydecorp/hydejack@v9`. Layouts, includes, and stylesheets from the theme are fetched at build time and are **not present in this repo** (`_layouts/` holds only a `.gitkeep`). Consequences:

- To change a layout, you must copy the file out of the upstream Hydejack repo into a local `_layouts/`, which then shadows the theme's version.
- The `Gemfile` also declares `gem "jekyll-theme-hydejack"`, but the `theme:` key in `_config.yml` is commented out — `remote_theme` is what actually takes effect.
- This is the **free** version of Hydejack. Features gated behind PRO (`grid`/`resume`/`portfolio` layouts, dark mode, forms) are configured in `_config.yml` but will not render. `dark_mode` settings are inert.

The supported extension points, all present in the repo:

| File | Purpose |
|---|---|
| `_sass/my-inline.scss` | CSS inlined into every page's `<head>`. Holds the custom `.btn` style used by `_includes/button.html`. |
| `_sass/my-style.scss` | CSS loaded as a normal stylesheet. |
| `_includes/my-head.html` | Injected into `<head>`. Currently lazy-loads Gumroad embed scripts via `IntersectionObserver`. |
| `_includes/my-body.html` | Injected at end of `<body>`. Re-runs Cloudflare email-obfuscation decoding after each client-side navigation. |
| `_data/strings.yml` | All soft-coded UI strings — change wording or translate here, not in layouts. |
| `_data/variables.yml` | Typographic scale, content/sidebar widths, breakpoints. Consumed by the theme's Sass. |
| `_data/authors.yml` | Author bio, photo, and sidebar social links. The `jaipi` key must stay in sync with `author:` in `_config.yml`. |
| `_data/social.yml` | Icon + URL-prefix metadata for social network keys used in `authors.yml`. |

Hydejack loads pages as a single-page app (`hy-push-state`). Any script added via `my-head.html` / `my-body.html` must re-initialize on the `hy-push-state-after` / `load` event rather than only on initial page load — both existing includes do exactly this.

## Content structure

Content is organized by **featured categories**, a Hydejack mechanism: a file in `_featured_categories/<slug>.md` creates a listing page at `/<slug>/`, and posts under `<slug>/_posts/` are collected into it.

- `blog/_posts/` → `_featured_categories/blog.md` → `/blog/`
- `projects/_posts/` → `_featured_categories/projects.md` → `/projects/` (`no_groups: true`, so not grouped by date)
- `scripts/` → a `layout: page` docs section reached via `scripts/README.md`, which the `jekyll-readme-index` plugin maps to `permalink: /scripts/`. Its pages are still the upstream Hydejack documentation (install/config/build/deploy/writing), lightly trimmed — not original content.
- `index.html` (`layout: blog`, `cover: true`) is the landing page; `author.md` (`layout: about`) is the CV/résumé page.

Sidebar navigation is **not** derived from the filesystem — it comes from the `menu:` list in `_config.yml`. Adding a section requires an entry there.

Post permalinks are `/:categories/:year-:month-:day-:title/` and pagination is set to 2 posts per page.

### Conventions when adding content

- The `<!--author-->` comment in `index.html` and `author.md` is a Hydejack marker that expands to the author blurb from `_data/authors.yml`.
- PDF/download links use the local include: `{% include button.html url="..." label="..." %}` (styled by `.btn` in `my-inline.scss`).
- Posts supply responsive images as an `image.path` + `image.srcset` hash keyed by width (`1920w`/`960w`/`480w`), with the downscaled files committed next to the original using the `@0,5x` / `@0,25x` suffix convention.
- Most of the existing posts and all `scripts/` pages are carried-over Hydejack sample content and carry `sitemap: false`. Real content should omit that.
- `jekyll-optional-front-matter` and `jekyll-titles-from-headings` are active with `remove_originals: true`, so a markdown file without front matter still renders and takes its title from the first heading.

## Site identity

Title `JaiPi`, author Jaime Pignatelli. Accent `rgb(79,177,186)`, theme color `rgb(25,55,71)`. Google Fonts are deliberately off (privacy default since Hydejack 9.2); the site uses the visitor's system font stack.
