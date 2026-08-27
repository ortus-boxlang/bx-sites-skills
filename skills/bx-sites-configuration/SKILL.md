---
name: bx-sites-configuration
metadata:
  version: "1.0"
description: Full bxsites.yaml/bxsites.json key reference for a bx-sites (ortus-boxlang/bx-sites) project - baseURL, robots.txt, nav, redirects, markdown options, repo/social/footer, lastUpdated, analytics, ogImage/generateOgImages, extraCss/extraJs, the assets/image pipeline, pageActions, and the plugins/i18n/blog/variables keys. Use this whenever a user asks what a bxsites.yaml key does, how to set the site's base URL/sub-path, how to customize the nav, or wants to tune the responsive-image/asset-bundling pipeline. For themes, search providers, and deployment config, use bx-sites-themes/bx-sites-search/bx-sites-deployment instead.
---

# BxSites Configuration Reference

One site config at the project root: `bxsites.yaml` (or `.yml`, default/
preferred) or `bxsites.json` (fully supported). If more than one is present,
`bxsites.yaml` wins, then `bxsites.yml`, then `bxsites.json`. Only `name` is
required - everything else defaults as shown. A partial `theme` object
merges one level deep (`{theme: {name: material}}` keeps the default empty
`options`).

```yaml title="bxsites.yaml - every key and its default"
name: "My Docs"
description: ""
baseURL: "/"
theme:
  name: bootstrap
  options: {}
  logo: ""
  favicon: ""
search: true
searchProvider:
  provider: local
  algolia: { appId: "", apiKey: "", indexName: "", insights: false }
nav: []
markdown:
  enableAdmonition: true
repo:
  url: ""
  editUri: ""
social: []
footer: false
lastUpdated: false
mermaid: false
math: false
analytics:
  provider: ""
  id: ""
ogImage: ""
generateOgImages: false
extraCss: []
extraJs: []
assets:
  fingerprint: true
  bundle: true
  images: { enabled: true, widths: [400, 800, 1200, 1600], formats: [original, webp] }
plugins: []
i18n:
  defaultLocale: { code: en, label: English }
  locales: []
blog:
  postsPerPage: 10
  feed: true
variables: {}
```

## `name` / `description`

`name` (required) - site name, shown in header/brand mark and page titles.
`description` - fallback `<meta name="description">`/`og:description` for
any page without its own `description` frontmatter (see
`bx-sites-getting-started` for page frontmatter).

## `baseURL`

Controls prefixing for every internal link/asset/nav entry, and doubles as
the canonical URL for `sitemap.xml`/`robots.txt`/`llms.txt`/canonical tags.

- **Blank or `"/"`** (default) - root-relative links (`/page/`); no
  `sitemap.xml`, no `Sitemap:` line, no canonical tags (no domain to build
  them from).
- **A bare path** (`"my-docs"` or `"/my-docs/"`) - sub-path served
  (`/my-docs/page/`); still no sitemap/canonical (no absolute domain).
- **A full URL** (`"https://docs.example.com/"`) - the path portion is used
  the same as a bare path, **and** `sitemap.xml`/`robots.txt`'s `Sitemap:`
  line/every page's `<link rel="canonical">` are generated. A version/locale
  tree points its canonical at its own URL, not the main site's.

`llms.txt` is always written regardless (absolute URLs when `baseURL`
provides them, `basePath`-relative otherwise).

## `robots.txt`

`robots: true` (default) writes `Allow: /` for every crawler plus a
`Sitemap:` line (when `baseURL` is a full URL). `robots: false` writes
`Disallow: /` and no `Sitemap:` line - a crawler opt-out only, **not access
control** (the site is still fully reachable by URL - see
`bx-sites-deployment` for real access restriction). Drop a hand-authored
`docs/robots.txt` to bypass the generated one entirely (copied byte-for-byte,
`robots` key ignored once this file exists).

## `theme`

- `theme.name` - `bootstrap`/`material`/`tailwind`/a gallery theme name, or a
  custom theme's own name - see `bx-sites-themes`
- `theme.logo`/`theme.favicon` - path (resolved against `docs/assets/`,
  prefixed with `baseURL`) or absolute URL
- `theme.options.colorMode` - `"auto"` (default, follows OS)/`"light"`/`"dark"`
  first-visit default; a visitor's own toggle choice always wins after
- `theme.options.navCollapsible` - `false` (default): every nav section
  always expanded. `true`: sections get a collapse toggle; the section
  containing the current page always starts open.
- `theme.options.navExpandAll` - only with `navCollapsible: true`. `true`
  (default): every section starts open. `false`: every section starts
  collapsed except the current page's.
- `theme.options.tocPosition` - `"top"` (default, inline at article top) or
  `"sticky"` (pinned right-hand column on wide viewports; a collapsible
  top-pinned bar below that width)
- `theme.options.pageMetaPosition` - `"bottom"` (default, footer note) or
  `"top"` for the edit-page/download-markdown/last-updated row
- `theme.options.pageActionsPosition` - `"top"` (default) or `"bottom"` for
  the [`pageActions`](#pageactions) dropdown

## `search` / `searchProvider`

`search: true`/`false` is the master switch regardless of provider.
`searchProvider.provider`: `"local"` (default, MiniSearch + static index),
`"algolia"` (needs `algolia.appId`/`apiKey`/`indexName`), `"pagefind"`
(`pagefind.bin`/`options`, both optional), or any other string for a fully
custom provider wired via a theme override. Full comparison and setup in
`bx-sites-search`.

## `nav`

Empty array (default) = infer from `docs/`'s folder/file structure (honoring
page `order`/`hidden` frontmatter). A non-empty array replaces inference
entirely - array order becomes nav order; a page not referenced is still
built, just unlinked (same as `hidden: true`). Each entry is either:

- a bare `docs/`-relative path string (`"guides/setup.md"`), title from that
  page's own frontmatter/filename
- an object `{ title, path, icon, children }` - `path`/`icon`/`children` all
  optional; a `title`-only entry with no `path` is an unlinked group
  heading; explicit `title`/`icon` override the page's own nav label/icon
  (the page's real `<h1>`/`<title>` is untouched)

```yaml title="bxsites.yaml"
nav:
  - index.md
  - title: Main Components
    children:
      - title: Quick Start
        path: guides/setup.md
      - guides/deployment.md
```

For a nav large enough to clutter `bxsites.yaml`, move it to `docs/nav.json`
(same array shape as the whole file's top-level content) - `bxsites.yaml`'s
own non-empty `nav` always wins over it if both exist. Only the main tree
honors either; a `docs/versions/<name>/` tree always infers from its own
folder structure.

## `redirects`

Site-wide `from`/`to` pairs, main tree only. See the
`bx-sites-blog-versioning-i18n` skill for the full picture including the
per-page `redirect_from` frontmatter alternative.

```yaml title="bxsites.yaml"
redirects:
  - from: old-guide
    to: guides/new-guide/
```

## `markdown`

Forwarded as-is to bx-markdown - not redefined/validated by bx-sites beyond
`enableAdmonition` (bx-markdown itself defaults it `false`; bx-sites
defaults it `true`). See `bx-sites-markdown` for the syntax each key
controls.

| Key | Default | Effect |
|---|---|---|
| `enableAdmonition` | `true` | `!!!`/`???`/`???+` callouts |
| `enableFootnotes` | `false` | `[^label]` footnotes |
| `enableDefinitionLists` | `false` | `Term\n:   Definition` lists |
| `autoLinkUrls` | `true` | Auto-links bare URLs/emails |
| `anchorLinks` | `true` | Clickable anchor link per heading |
| `anchorSetId` / `achorSetName` *(sic)* | `true` | `id`/`name` attrs on headings |
| `anchorWrapText` | `false` | Wraps whole heading text in the anchor |
| `anchorClass` | `"anchor"` | CSS class on the anchor `<a>` |
| `anchorPrefix` / `anchorSuffix` | `""` | Raw HTML before/after heading text |
| `enableYouTubeTransformer` | `false` | Auto-embeds bare YouTube links |
| `codeStyleHTMLOpen`/`Close` | `<code>`/`</code>` | Inline code wrapper |
| `fencedCodeLanguageClassPrefix` | `"language-"` | Class prefix highlighter/Mermaid key off |
| `tableOptions.columnSpans` | `true` | Honors `colspan` merged cells |
| `tableOptions.appendMissingColumns` | `true` | Pads a short row |
| `tableOptions.discardExtraColumns` | `true` | Drops a long row's extra cells |
| `tableOptions.className` | `"table"` | CSS class per `<table>` |
| `tableOptions.headerSeparationColumnMatch` | `true` | `---` row must match header column count |

## `repo`

`repo.url` adds a header repo-icon link. `repo.editUri` (e.g.
`"edit/main/docs/"`) plus `repo.url` builds an "Edit this page" link per
page.

## `social` / `footer`

`social` - array of `{ url, icon, label }`, rendered in the footer only when
`footer: true`. `icon` is one of `github`/`twitter`(`x`)/`youtube`/
`linkedin`/`facebook`/`bluesky`/`threads`/`slack`/`patreon`/`rss`/`email`
(generic link glyph otherwise). `footer: true` adds a copyright line, the
`social` links, and a "Built with BxSites" credit to every page.

## `lastUpdated`

`false` default. `true` adds a "Last updated" line sourced from `git log` on
each page's own file at build time - silently omitted where git has no
history for it (fresh repo, zip download, no git installed), rather than
breaking the build.

## `analytics`

Google Analytics only currently: `provider: "google"` + `id: "G-ABC123"`.

## `ogImage` / `generateOgImages`

`ogImage` - default social-card image (path resolved like `theme.logo`, or
absolute URL); a page's own frontmatter `ogImage` always wins for that page.
`generateOgImages: true` renders a real 1200x630 PNG per page lacking its
own `ogImage` (pure `java.awt`, no headless browser/network needed).

## `extraCss` / `extraJs`

Arrays of stylesheet/script URLs appended after the theme's own assets, each
resolved like `theme.logo`. `extraJs` entries load with `defer`. When
`assets.bundle` is on (default) and every entry is a local project file,
they're bundled into one fingerprinted file each instead of one tag per
entry - one external/missing entry falls the whole list back to per-URL tags.

## `assets`

The image-resizing/bundling pipeline, on by default with sane settings -
usually nothing to touch.

- `assets.fingerprint` (`true`) - content-hash names generated variants/
  bundles for safe far-future caching; never renames a project's own
  originals under `docs/assets/`.
- `assets.bundle` (`true`) - concatenates `extraCss`/`extraJs` into one
  fingerprinted file each (pure BoxLang/JVM, no Node toolchain).
- `assets.images.enabled` (`true`) - resize/WebP + `<picture>` rewrite for
  eligible images; `false` falls back to plain unprocessed copying.
- `assets.images.widths` (`[400, 800, 1200, 1600]`) - breakpoints; a width
  at/above an image's own is skipped (never upscaled).
- `assets.images.formats` (`["original", "webp"]`).

## `mermaid` / `math` / `openapi`

All `false` by default - `true` loads the client-side library (Mermaid/
KaTeX/Swagger UI) and activates the corresponding Markdown syntax. See
`bx-sites-markdown` and `bx-sites-content-blocks` for the syntax itself.

## `pageActions`

`false` default. `true` adds a "Copy" dropdown (Copy page/link, View as
Markdown, Open in ChatGPT/Claude, Export as PDF, Report an issue, Share on
X/LinkedIn/Facebook) - press `c` to toggle it too. Several sub-items only
render when their prerequisite is set (`baseURL` full URL for link/share
items, `repo.url` for "Report an issue"). No extra JS shipped unless this is
on.

## `plugins`

`[]` default - array of BoxLang module names to activate. See
`bx-sites-plugins`.

## `i18n` / `blog` / `variables`

See the `bx-sites-blog-versioning-i18n` and `bx-sites-variables-functions`
skills for the full picture - these keys are metadata/tuning for
content-authoring features, not build/deploy concerns.
