---
name: bx-sites-themes
metadata:
  version: "1.0"
description: Choose, customize, override, install, or write a theme for a bx-sites (ortus-boxlang/bx-sites) site - the 10 built-in themes, air-gapped/offline considerations, the ThemeProvider contract (layout.bxm/page.bxm, plus optional search.bxm/blog.bxm/blog-page.bxm and any frontmatter-named layout), the layout: frontmatter key (outer shell + body), bootstrap's marketing homepage (home.bxm), color-only customization via extraCss, ejecting/overriding a theme, writing one from scratch (template variables, shared helpers like bxsitesNavHref/bxsitesIcon, which shared scripts to load), the page-actions dropdown and image gallery chrome, installing a published theme from ForgeBox, importing an mkdocs/jekyll/hugo theme, and the homepage hero banner. Use this whenever a user wants to change a bx-sites site's look, brand colors, or write/override its templates.
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
BoxLang brand palette (`#00FF78 -> #00DBFF` gradient); their accents differ
(`bootstrap` `#FFF500`, `material` `#00C853`, `tailwind` `#38bdf8`). The
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

Recent chrome, in all 10 themes unless noted (config keys, including every
`theme.options` key such as `accentColor`/`accentColorDark` - a no-CSS
alternative to the `extraCss` recipe below - live in `bx-sites-configuration`):

- **Page actions dropdown** - `pageActions: true`, placed by
  `theme.options.pageActionsPosition` (`top`|`bottom`). Items: Copy page, Copy
  link, View as Markdown, Open in ChatGPT/Claude, Edit on GitHub, Export as
  PDF, Share on X/LinkedIn/Facebook. Each item only appears when the page has
  what it needs: Markdown items need `page.markdownUrl`; copy link, AI and
  share items need `page.canonicalUrl` (set `baseURL`); Edit needs `page.editUrl`.
- **"Report an issue"** link in the page-meta row when `repo.url` is set.
- **Prev/next cards**, and **course lesson nav** (badge plus prev/next cards)
  when the page belongs to a `::: course` (`page.course`).
- **TOC** - `theme.options.tocPosition`: `top` (default) or `sticky` (side rail).
- **Click-to-enlarge images** - `imageGallery: true` ships a lightbox for
  every content image (opt out per image with `class="no-zoom"`).
- **Locale dropdown** when `i18n` locales exist; **"Powered by BxSites"**
  badge, unconditional (`bxsites-logo.svg`).

```yaml title="bxsites.yaml"
theme: { name: material }
```

## Installing a published theme

```bash
bxSites install:theme --name=bx-sites-theme-blog1 [--version=1.0.0]
```

Downloads from ForgeBox into `<contentRoot>/.themes/bx-sites-theme-blog1/`
(inside the project's resolved content root, not the bare project root - see
"Resolution order" below), validating the `ThemeProvider` contract before
finishing (a broken package fails at install time, not at the next `build`).
Re-installing the same name deletes the existing `.themes/<name>/` first, with
no prompt (unlike `theme:import`, which merges - see below). No separate
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
  leading `/`). Includes the resolved body template via
  `#variables.themeDir#/#variables.bodyFile#` (`page.bxm` unless a page/section
  resolves to something else - see "Multiple layouts per page" below).
- **`page.bxm`** (required) - article body. Renders `variables.page.contentHtml`
  (already-converted markdown).
- **`search.bxm`** (optional) - search box markup, included only when
  `search: true`.
- **`blog.bxm`** (optional) - an alternate outer shell for the blog's own
  listing/category/archive/author/stats pages. Falls back to `layout.bxm`
  when a theme doesn't have it.
- **`blog-page.bxm`** (optional) - an alternate body for an individual blog
  post, rendered under the theme's normal `layout.bxm`. Falls back to
  `page.bxm` when a theme doesn't have it.
- **`assets/`** (optional) - theme CSS/JS, copied to `site/assets/theme/`.
- **`<name>.bxm`** (optional) - any extra template a page can opt into by
  frontmatter `layout: <name>` - `home.bxm` is the built-in example (see
  "Multiple layouts per page" below). Not enforced; a name no theme file
  matches simply falls back.

Also available in every template (a from-scratch theme can rely on all of
these; missing optional values are defaulted to empty):

- `variables.page` always carries: `title`, `contentHtml`, `toc`
  (`[ { level, id, text } ]`), `prevPage`/`nextPage` (`{ title, url }`),
  `breadcrumbs`, `course` (`id` is `""` when the page isn't a lesson), `editUrl`,
  `lastUpdated`, `description`, `tags`, `icon`, `iconHtml`, `summary`,
  `ogImage`, `markdownUrl`, `canonicalUrl` (empty without `baseURL`),
  `urlPath`, `isFallbackContent`.
- `variables.siteConfig` (`repo`, `social`, `footer`, `theme.options`, ...),
  `variables.moduleAssetsDir`, `variables.data` (data files),
  `variables.locales` / `currentLocale` / `currentLocaleDir` (`ltr`|`rtl`,
  for `<html dir>`), and `variables.strings` (the locale's UI labels, e.g.
  `strings.toggleDarkMode`; `variables.stringsResolver.format()` fills
  placeholders in them).
- Project `$`-prefixed magic functions are bound bare, like the helpers below.
- `variables.versions` (`[ { label, url, kind } ]`, `kind` one of
  `"current"`/`"released"`/`"next"` - see `bx-sites-blog-versioning-i18n`) +
  `variables.currentVersion`. Order depends on `versions.default`: unset, it is
  "Latest" (`current`) then released versions newest first; set, it is that
  version (`current`), the other released versions, then "Next" (`next`) last.

Shared includes (`<bx:include template="#variables.moduleAssetsDir#/icons.bxm">`
and `.../locale-flags.bxm`):

- `bxsitesIcon( name )` - raw SVG for `github`, `twitter`/`x`, `rss`,
  `youtube`, `linkedin`, `facebook`, `bluesky`, `threads`, `slack`, `patreon`,
  `email`, `edit`, `clock`, `markdown`, `globe`, `moon`, `sun`, `copy`,
  `link`, `print`, `sparkle`, `chevron-down`, `flag`, `outline`; an unknown
  name returns the generic globe.
- `bxsitesUrlAttr( value )` - escapes only `&`, `"`, `<` (keeps `/` intact)
  for a double-quoted attribute.
- `bxsitesNavHref( url, basePath )` - the `href` for a nav node: prefixes
  `basePath` to docs-relative URLs, keeps absolute ones (`https:`, `mailto:`,
  `//host`), turns `javascript:`/`data:`/`vbscript:` into `#`, and
  attribute-escapes. Always use it for nav links instead of concatenating.
- `bxsitesLocaleFlag( code )` (locale-flags.bxm) - flag emoji for a locale code.

Three more helpers are bound into every theme's render scope (built-in or
custom) the same way, no include needed:

- `themeAssetUrl( "style.css" )` / `moduleAssetUrl( "copy-code.js" )` -
  resolve a theme's own top-level `assets/` file (or one of this module's
  shared scripts) to its basePath-prefixed, cache-busted URL. Use these
  instead of a hardcoded `assets/theme/...`/`assets/...` path for anything
  a layout links directly - only top-level files are fingerprinted, never
  one in a subdirectory.
- `renderVersionSwitcher( variables.versions, variables.currentVersion )` -
  builds the whole version-switcher `<select>`, grouping a `"next"` entry
  behind a divider after every released version and labeling `"current"`
  with "(Current)" - a theme wanting its own styling passes an optional
  third argument of extra CSS classes instead of reimplementing the markup.

A theme folder missing either required file fails fast with
`BxSites.InvalidTheme` at build time; a `theme.name` that matches no
override, installed, or built-in theme fails with `BxSites.ThemeNotFound`.
`blog.bxm`/`blog-page.bxm` are never enforced - same "optional, falls back"
shape `search.bxm` already has.

Both override locations below live inside the project's *resolved content
root* (`docs/`, `src/`, a custom `source:`, or the project root itself for
`source: .`) - not the bare project root. See `bx-sites-configuration` for
how `source`/`exclude` resolve that root.

**Resolution order**: `<contentRoot>/.theme/` override → installed
`<contentRoot>/.themes/<name>/` matching `theme.name` → a built-in theme
named `theme.name`.

## Multiple layouts per page

A theme can offer more than one outer shell and more than one body template,
letting a blog (or any other content type) look different from the rest of
the site while reusing the same theme's chrome/assets. Two independent
resolution chains:

- **Outer shell** - `layout.bxm`, except the blog's own listing/category/
  archive/author/stats pages, which use `blog.bxm` when the active theme has
  it. A page's own frontmatter wins here too (see below).
- **Body** - `page.bxm` by default. A blog post uses `blog-page.bxm` when the
  active theme has it. Either way, a page's own frontmatter always wins when
  set:

```markdown title="docs/marketing/press-release.md"
---
title: We raised a Series A
layout: press-release
---
```

`layout: press-release` resolves **both** slots: this one page renders
through `.theme/press-release.bxm` (or the active built-in theme's, if it has
one) as its outer shell *and* as its body. A `layout:` naming a file the
active theme doesn't have falls back to `layout.bxm`/`page.bxm` rather than
failing the build, so switching themes never breaks a page that named one
theme's own custom layout.

Caveat: this two-slot behavior is for doc pages. For a blog **post** a
frontmatter `layout:` resolves only the body (the shell stays `layout.bxm`,
since a post's shell hint is empty); the blog's own listing pages use `blog.bxm`
as the shell.

```markdown title="docs/index.md - an alternate outer shell for one page"
---
title: BxSites
layout: home
---
```

That's how a marketing homepage works: `bootstrap` ships a `home.bxm` that
is **not** a derivative of `layout.bxm` - no sidebar, no TOC rail, a
horizontal marketing nav instead of the docs hamburger - and it never
includes `#variables.bodyFile#`, so the whole page is hardcoded in the
template and that page's own Markdown body goes unused while it renders.
Drop `layout: home` and the page falls straight back to the normal
`layout.bxm`/`page.bxm` rendering, body included. `home.bxm` exists **only in
`bootstrap`**; in the other 9 themes `layout: home` falls back to
`layout.bxm`/`page.bxm`.

So a frontmatter `layout:` is all-or-nothing across both slots: the named
template has to be a complete outer shell that renders everything itself,
which is exactly why `home.bxm` never includes `#variables.bodyFile#`
(including it there would include itself). Body-only alternates are the ones
a theme ships per content type - `blog-page.bxm` for a post, rendered under
the normal `layout.bxm`. For a custom body inside the usual shell, branch
inside `page.bxm` on `variables.bodyFile` instead (below).

`bootstrap` ships `blog.bxm`/`blog-page.bxm` (and `home.bxm`) as working
examples to copy; the other built-in themes don't yet, and fall back to
`layout.bxm`/`page.bxm` for blog content the same way any incomplete
`.theme/` override would.

### Which layout/body is active

Every `.bxm` a page renders through - `layout.bxm`, `page.bxm`, `blog.bxm`,
`blog-page.bxm`, or a project's own custom one - can read which files
actually resolved, the same bare way it already reads `variables.page`/
`variables.data`:

- `variables.layoutFile` - the outer shell in use, e.g. `"layout.bxm"`,
  `"blog.bxm"`, or a frontmatter-named one like `"home.bxm"`
- `variables.bodyFile` - the body in use, e.g. `"page.bxm"`,
  `"blog-page.bxm"`, or a frontmatter-named one
- `variables.page.layout` - the page's own raw frontmatter `layout:` value,
  if it set one; `""` otherwise

Useful for a body class hook, or branching without a separate template:

```bx title=".theme/layout.bxm"
<body class="layout-#reReplace( variables.bodyFile, '\.bxm$', '' )#">
```

```bx title=".theme/page.bxm"
<bx:if variables.bodyFile == "blog-page.bxm">
	<!-- blog-post-only chrome -->
</bx:if>
```

## Customizing colors without a full override

Each built-in theme reads its palette from CSS custom properties on `:root`
(re-declared under `[data-theme="dark"]`). `extraCss` (see
`bx-sites-configuration`) loads *after* the theme's own stylesheet, so a
same-specificity re-declaration wins:

```yaml title="bxsites.yaml"
extraCss: [ assets/brand.css ]
```

```css title="docs/assets/brand.css (gradient lines: not for tailwind)"
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

Every built-in theme guarantees `--bxsites-accent`, `--bxsites-link`/
`-link-hover`, the `--bxsites-table-*` set (table colors), and the
`--bxsites-step-*` set (backing the `::: stepper` block - see
`bx-sites-content-blocks`) under those exact names. Beyond that it varies:

- `--bxsites-gradient-start`/`-end` - every theme **except `tailwind`**, which
  hardcodes Tailwind gradient utilities, so overriding them there does nothing
  (use `--bxsites-accent`/`theme.options.accentColor` or a real override).
- `--bxsites-bg`/`-text`/`-muted`/`-border` - `bootstrap`, `justthedocs`,
  `notion`, `slate`, `tailwind`.
- `--bxsites-code-bg` - `bootstrap`, `justthedocs`, `notion`, `slate` (not `tailwind`).
- `--bxsites-sidebar-bg`/`-sidebar-text` - `bootstrap`, `notion`, `slate` only.

`material`, `docsy`, `docusaurus`, `vuepress`, and `gitbook` use their own
internal custom-property names for the bg/text/border group - open that
theme's own `assets/style.css` to find the real names first. Anything beyond
color/font needs a real override.

## Overriding a theme

Drop `layout.bxm` + `page.bxm` (and optionally `search.bxm`/`assets/`) into
a `.theme/` folder inside the project's resolved content root (`docs/.theme/`,
`src/.theme/`, or wherever `source:` points - see `bx-sites-configuration`)
- the built-in themes under this module's `resources/themes/` are good
starting points to copy:

```bash
bxSites theme:new --theme=bootstrap
```

`--theme` accepts any of the 10 built-in names; it copies the whole theme
folder recursively (bootstrap's `includes/page-helpers.bxm` and
`includes/site-helpers.bxm` come along) and throws `BxSites.ThemeOverrideExists`
if `<contentRoot>/.theme` already exists (remove it or edit it directly).
Then edit only what's needed - e.g. swap the brand palette/font in
`.theme/assets/style.css`. `bxSites build`/`serve` pick up `.theme/`
automatically, no config change needed - it takes precedence over
`theme.name` entirely. **All-or-nothing**: once a project `.theme/` exists it
needs its own `layout.bxm` + `page.bxm` even for a CSS-only change (missing
either fails with `BxSites.InvalidTheme`) - for CSS-only, prefer `extraCss`
above instead.

## Writing a theme from scratch

The absolute minimum - no Bootstrap/Tailwind, no dark mode, no search UI:

```bx title=".theme/layout.bxm"
<!-- .theme/layout.bxm -->
<bx:include template="#variables.moduleAssetsDir#/icons.bxm">
<bx:script>
	function renderNav( required array nodes ) {
		var html = "<ul>"
		for ( var node in arguments.nodes ) {
			html &= "<li>"
			html &= len( node.url )
				? '<a href="' & bxsitesNavHref( node.url, variables.basePath ) & '">' & encodeForHTML( node.title ) & '</a>'
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
	<link rel="stylesheet" href="#themeAssetUrl( 'style.css' )#">
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

```bx title=".theme/page.bxm"
<!-- .theme/page.bxm -->
<bx:output>
<article>
	<h1>#encodeForHTML( variables.page.title )#</h1>
	#variables.page.contentHtml#
</article>
</bx:output>
```

`themeAssetUrl( 'style.css' )` above (rather than a hardcoded
`assets/theme/style.css`) is what makes a theme's own CSS/JS pick up
content-hash cache-busting automatically - see `bx-sites-configuration`'s
`assets.fingerprint`. It only fingerprints a file directly inside a
theme's own `assets/`, never one in a subdirectory (a `@font-face`/`@import`
dependency would break if renamed without rewriting what references it) -
`themeAssetUrl()` returns the plain `assets/theme/<path>` for those. The
build prints a non-fatal warning when a theme `.bxm` hardcodes
`assets/theme/...` (or a shared script path) instead of using the helpers.

**Shared scripts a layout must load itself** (a from-scratch layout gets none
automatically; use `moduleAssetUrl( 'x.js' )`). Always shipped, always load:
`code-lines.js`, `copy-code.js`, `admonition-collapse.js`, `nav-toggle.js`,
`toc-collapse.js`, `tabs.js`, `highlight-boxlang.js`,
`conditional-content-init.js`, `course-progress.js`, `prompt-init.js`,
`contact-form-init.js` (plus vendored highlight.js and Alpine.js under
`assets/vendor/`). Only copied into the site when the config turns them on, so
gate the tag the same way:

| Script | Gate |
|---|---|
| `search.js` (+ vendored MiniSearch) | `search: true` with the `local` provider |
| `page-actions-init.js` | `pageActions: true` |
| `image-gallery-init.js` | `imageGallery: true` |
| `mermaid-init.js` (+ vendored mermaid) | `mermaid: true` |
| `math-init.js` (+ KaTeX from a CDN) | `math: true` |
| `openapi-init.js` (+ vendored Swagger UI) | `openapi: true` |
| `cloud-content-init.js` | `cloud.contentBlocks: true` |
| `comments-init.js` | `cloud.comments: true` (no built-in layout loads it; a theme using `::: comments` must add the tag) |

Gotcha: the vendored Bootstrap CSS/JS is copied into the site only when
`theme.name` is `bootstrap`. A `.theme/` override copied from bootstrap but
run under another `theme.name` (or a from-scratch theme) won't find
`assets/vendor/bootstrap/`.

`variables.page.contentHtml` is already fully converted (syntax
highlighting, admonitions, tabs, math, all of it) - there's nothing left to
parse, only lay out. Add breadcrumbs/tags/prev-next/dark-mode/search by
copying the pattern from a built-in theme's own `page.bxm`/`layout.bxm` (a
built-in `search.bxm` is only included when `search: true`).

The example above hardcodes `page.bxm`, which is fine for a minimal theme -
swap that line for `<bx:include template="#variables.themeDir#/#variables.bodyFile#">`
(a built-in theme's own `layout.bxm` already does) to opt into "Multiple
layouts per page" above, so a project can add its own `.theme/blog-page.bxm`
or `.theme/<name>.bxm` frontmatter override later without touching
`layout.bxm` again.

## Importing a theme from another SSG

```bash
bxSites theme:import --source=mkdocs --path=/path/to/theme --name=my-imported-theme
```

`--source` is `mkdocs`, `jekyll`, or `hugo` - only those template-based
themes are importable (not React/Vue component themes). `--path` resolves
against the project root (relative paths and `~/` work). Best-effort conversion
into a `<contentRoot>/.themes/<name>/` scaffold - a starting point, not lossless.
Safe to re-run against the same `--name` (`.bxm` files overwritten, new asset
folders merged in), but if `.themes/<name>/` already exists it asks
`[y/N]` first: answering no exits 0 having imported nothing, a closed stdin
(CI, pipes) exits 1, and `--force` skips the prompt. Contrast `install:theme`,
which deletes an existing `.themes/<name>/` and replaces it with no prompt.

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

`bxsites-hero__btn--primary`/`--secondary` are the same two accent styles
every theme already uses elsewhere - swap, drop, or add buttons freely, and
resize/replace the banner's own image via a `docs/assets/`-relative `src`
the same way any other image resolves.

A homepage that needs more than a banner - hero plus feature grids, a
comparison table, an ecosystem strip, a richer footer - is a full alternate
outer shell instead: name one in frontmatter (`layout: home`), per "Multiple
layouts per page" above. `bootstrap`'s `home.bxm` is a usable reference for
that shape, but not a drop-in starting point: it is bx-sites' own marketing
page (hardcoded copy, a comparison table, links into `course/` lessons), uses
its own `bxh-*` classes with `moduleAssetUrl( 'home.css' )` rather than
`bxsites-hero*` (those classes live in each theme's `assets/style.css` and
serve the plain-HTML banner above), and depends on
`includes/site-helpers.bxm` (`homeIcon()`). Copy it and strip it down; it
exists only in `bootstrap`.
