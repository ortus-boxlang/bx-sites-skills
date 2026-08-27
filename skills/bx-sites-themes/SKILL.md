---
name: bx-sites-themes
metadata:
  version: "1.0"
description: Choose, customize, override, install, or write a theme for a bx-sites (ortus-boxlang/bx-sites) site - the 10 built-in themes, air-gapped/offline considerations, the ThemeProvider contract (layout.bxm/page.bxm), color-only customization via extraCss, ejecting/overriding a theme, writing one from scratch, installing a published theme from ForgeBox, importing an mkdocs/jekyll/hugo theme, and the homepage hero banner. Use this whenever a user wants to change a bx-sites site's look, brand colors, or write/override its templates.
---

# BxSites Themes Reference

Themes are native BoxLang `.bxm` templates - no separate template engine or
build step.

## Built in

| Theme | Base | Notes |
|---|---|---|
| `bootstrap` (default) | Bootstrap 5, vendored | Poppins font, brand gradient navbar |
| `material` | Hand-rolled Material-style CSS | Card layout, elevation shadows, Roboto |
| `tailwind` | Tailwind Play CDN | Utility-class driven, no build step, **not air-gapped** |
| `docsy` | Forked from `material` | Read the Docs/Docsy-inspired navy reference-manual look |
| `slate` | Forked from `material` | Stripe/Slate-inspired - permanently dark sidebar |
| `docusaurus` | Forked from `material` | Bold full-width colored navbar, rounded cards |
| `justthedocs` | Forked from `material` | Minimalist; search box at top of sidebar |
| `vuepress` | Forked from `material` | Green accent, soft rounded corners |
| `gitbook` | Forked from `material` | Centered reading column, serif headings |
| `notion` | Forked from `material` | Borderless sidebar, near-grayscale, generous whitespace |

The seven `material`-forked themes reuse `material`'s exact `.bxm` templates
unchanged except a scoped CSS-class-prefix rename (and, for `justthedocs`,
one relocated `<bx:include>` line) - same full feature set, same
air-gapped-capable behavior. `bootstrap`/`material`/`tailwind` share the
BoxLang brand palette (`#00FF78 -> #00DBFF` gradient, `#FFF500` accent); the
seven gallery themes each use their own distinct palette.

Every theme ships the same feature set regardless of palette: in-page TOC,
breadcrumbs, prev/next links, highlight.js code blocks with copy buttons,
self-hosted webfonts, dark/light toggle (Alpine.js, `localStorage`
remembered), responsive header + collapsible sidebar, `/` and Cmd/Ctrl+K
search shortcuts, repo/edit-page/last-updated line, Download-Markdown link,
opt-in footer, version switcher, themed `404.html` (override with
`docs/404.md`), custom logo/favicon, collapsible nav, Google Analytics,
social share cards, page tags/icon/summary, nav override support, extra
CSS/JS injection, admonitions/footnotes/definition lists, content tabs, code
annotations, responsive images, Mermaid, math - see `bx-sites-markdown` and
`bx-sites-content-blocks` for that content-side syntax, and
`bx-sites-configuration` for every key mentioned here.

```yaml title="bxsites.yaml"
theme: { name: material }
```

## Installing a published theme

```bash
bxSites install:theme --name=bx-sites-theme-blog1 [--version=1.0.0]
```

Downloads from ForgeBox into `themes/bx-sites-theme-blog1/` at the project
root, validating the `ThemeProvider` contract before finishing (a broken
package fails at install time, not at the next `build`). No separate
activation step (unlike a plugin - see `bx-sites-plugins`) - just set
`theme.name` to match. Browse published themes under ForgeBox's
`bxsites-themes` category.

## Air-gapped / offline sites

Works with zero internet access by default for `bootstrap`, `material`, and
the seven gallery themes, with the default `local` search provider (see
`bx-sites-search`) - Bootstrap CSS/JS, highlight.js, Alpine.js, MiniSearch
all vendored (`resources/assets/vendor/`), no CDN tag anywhere. Turning on
`mermaid` vendors it the same way.

Still reach the network only when turned on: `tailwind`'s utility engine
(CDN JIT compiler - not air-gapped-capable); Mermaid's `elk`-layout diagrams
lazy-load one chunk from jsDelivr; `math` loads KaTeX from a CDN;
`searchProvider.provider: algolia` and `analytics.provider: google`
inherently talk to a hosted endpoint. For a genuinely air-gapped deployment:
stick to `bootstrap`/`material`/a gallery theme, the `local` search
provider, avoid `elk`-layout Mermaid, leave `math`/Algolia/analytics off.

## The `ThemeProvider` contract

A theme is a folder with:

- **`layout.bxm`** (required) - outer HTML shell + nav. Receives
  `variables.page`, `variables.nav`, `variables.siteConfig`,
  `variables.themeDir`, `variables.basePath` (root-relative, ends `/` -
  prefix every internal `href`/`src` with it rather than hardcoding a
  leading `/`). Includes `page.bxm` via `#variables.themeDir#/page.bxm`.
- **`page.bxm`** (required) - article body. Renders `variables.page.contentHtml`
  (already-converted markdown).
- **`search.bxm`** (optional) - search box markup, included only when
  `search: true`.
- **`assets/`** (optional) - theme CSS/JS, copied to `site/assets/theme/`.

Also available: `variables.page.editUrl`/`.lastUpdated` (empty strings if
unconfigured), `variables.siteConfig.repo`/`.social`/`.footer`,
`variables.versions` (`[ { label, url } ]`, "Latest" first) +
`variables.currentVersion`, and a shared icon include
(`<bx:include template="#variables.moduleAssetsDir#/icons.bxm">`, defines
`bxsitesIcon( name )`).

A theme folder missing either required file fails fast with
`BxSites.InvalidTheme` at build time.

**Resolution order**: project `theme/` override → installed
`themes/<name>/` matching `theme.name` → a built-in theme named `theme.name`.

## Customizing colors without a full override

Each built-in theme reads its palette from CSS custom properties on `:root`
(re-declared under `[data-theme="dark"]`). `extraCss` (see
`bx-sites-configuration`) loads *after* the theme's own stylesheet, so a
same-specificity re-declaration wins:

```yaml title="bxsites.yaml"
extraCss: [ assets/brand.css ]
```

```css title="docs/assets/brand.css"
:root {
	--bxsites-gradient-start: #7C3AED;
	--bxsites-gradient-end: #DB2777;
	--bxsites-accent: #FBBF24;
	--bxsites-link: #7C3AED;
	--bxsites-link-hover: #9F5AF0;
}

[data-theme="dark"] {
	--bxsites-link: #C4B5FD;
	--bxsites-link-hover: #DDD6FE;
}
```

Every built-in theme guarantees `--bxsites-gradient-start`/`-end`,
`--bxsites-accent`, and the `--bxsites-step-*` set (backing the
`::: stepper` block - see `bx-sites-content-blocks`) under those exact
names. Only `bootstrap`, `slate`, and `notion` also expose
`--bxsites-bg`/`-text`/`-sidebar-bg`/`-sidebar-text`/`-border`/`-link`/
`-link-hover`/`-code-bg` under those names (`justthedocs` aliases all but
the two `-sidebar-*`); every other theme (`material`, `tailwind`, `docsy`,
`docusaurus`, `vuepress`, `gitbook`) uses its own internal custom-property
names for that group - open that theme's own `assets/style.css` to find its
real names first. Anything beyond color/font needs a real override.

## Overriding a theme

Drop `layout.bxm` + `page.bxm` (and optionally `search.bxm`/`assets/`) into
a `theme/` folder at the project root - the built-in themes under this
module's `resources/themes/` are good starting points to copy:

```bash
bxSites theme:new --theme=bootstrap
```

then edit only what's needed - e.g. swap the brand palette/font in
`theme/assets/style.css`. `bxSites build`/`serve` pick up `theme/`
automatically, no config change needed - it takes precedence over
`theme.name` entirely. **All-or-nothing**: once a project `theme/` exists it
needs its own `layout.bxm` + `page.bxm` even for a CSS-only change (missing
either fails with `BxSites.InvalidTheme`) - for CSS-only, prefer `extraCss`
above instead.

## Writing a theme from scratch

The absolute minimum - no Bootstrap/Tailwind, no dark mode, no search UI:

```bx title="theme/layout.bxm"
<!-- theme/layout.bxm -->
<bx:script>
	function renderNav( required array nodes ) {
		var html = "<ul>"
		for ( var node in arguments.nodes ) {
			html &= "<li>"
			html &= len( node.url )
				? '<a href="' & variables.basePath & node.url & '">' & encodeForHTML( node.title ) & '</a>'
				: encodeForHTML( node.title )
			if ( node.children.len() ) {
				html &= renderNav( node.children )
			}
			html &= "</li>"
		}
		return html & "</ul>"
	}
</bx:script>
<bx:output>
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>#encodeForHTML( variables.page.title )# - #encodeForHTML( variables.siteConfig.name )#</title>
	<link rel="stylesheet" href="#variables.basePath#assets/theme/style.css">
</head>
<body>
	<header><a href="#variables.basePath#">#encodeForHTML( variables.siteConfig.name )#</a></header>
	<nav>#renderNav( variables.nav )#</nav>
	<main>
</bx:output>
<bx:include template="#variables.themeDir#/page.bxm">
<bx:output>
	</main>
</body>
</html>
</bx:output>
```

```bx title="theme/page.bxm"
<!-- theme/page.bxm -->
<bx:output>
<article>
	<h1>#encodeForHTML( variables.page.title )#</h1>
	#variables.page.contentHtml#
</article>
</bx:output>
```

`variables.page.contentHtml` is already fully converted (syntax
highlighting, admonitions, tabs, math, all of it) - there's nothing left to
parse, only lay out. Add breadcrumbs/tags/prev-next/dark-mode/search by
copying the pattern from a built-in theme's own `page.bxm`/`layout.bxm` (a
built-in `search.bxm` is only included when `search: true`).

## Importing a theme from another SSG

```bash
bxSites theme:import --source=mkdocs --path=/path/to/theme --name=my-imported-theme
```

`--source` is `mkdocs`, `jekyll`, or `hugo`. Best-effort conversion into a
`themes/<name>/` scaffold - a starting point, not lossless. Safe to re-run
against the same `--name` (`.bxm` files overwritten, new asset folders
merged in).

## Homepage hero banner

No directive/config needed - plain HTML any page (typically `index.md`) can
drop in, using CSS every built-in theme already ships:

```markdown title="docs/index.md"
<div class="bxsites-hero">
	<img class="bxsites-hero__banner" src="assets/home-banner.jpg" alt="...">
	<div class="bxsites-hero__actions">
		<a class="bxsites-hero__btn bxsites-hero__btn--primary" href="getting-started.md">Get Started</a>
		<a class="bxsites-hero__btn bxsites-hero__btn--secondary" href="https://github.com/your/repo">View on GitHub</a>
	</div>
</div>
```
