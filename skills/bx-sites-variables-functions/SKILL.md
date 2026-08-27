---
name: bx-sites-variables-functions
metadata:
  version: "1.0"
description: Use reusable {{ variables }} and BoxLang "magic functions" in bx-sites (ortus-boxlang/bx-sites) Markdown - bxsites.yaml's variables block, docs/functions.bxs, context variables (page/siteConfig/nav/versions/locales), and visualizer recipes (status badges, star ratings, progress bars, trend arrows) including inside table cells. Use this whenever a user wants to avoid repeating a fact across pages, wants a status chip/rating/progress bar in a page or table, or wants to write reusable BoxLang logic callable from Markdown.
---

# BxSites Variables & Magic Functions

Two ways to keep repeated facts/logic out of Markdown - both share one
syntax:

```text
{{ dotted.path }}          # a reusable variable
{{ $name(arg1, arg2) }}    # a magic function call
```

## Reusable variables

Add a `variables` block to `bxsites.yaml` (see `bx-sites-configuration`),
any shape, flat or nested:

```yaml title="bxsites.yaml"
variables:
  company: "Ortus Solutions"
  product:
    name: "BoxLang"
    supportEmail: "support@example.com"
```

Reference by dotted path from any page:

```markdown
# Welcome to {{ company }}

We build {{ product.name }} tools. Need help? Write us at
{{ product.supportEmail }}.
```

Resolved once, at build time, against `bxsites.yaml`'s current `variables` -
change it once, every page picks it up on the next build.

`variables` is a single, project-wide block - not itself translatable per
locale. A multilingual project wanting different text per language should
use a magic function switching on `siteConfig.i18n.defaultLocale.code`, or
keep the value locale-neutral.

## Magic functions

Add `docs/functions.bxs` (or `src/functions.bxs`) - a plain BoxLang script.
Any function named with a leading `$` becomes callable from `{{ }}` in
Markdown, and bare (no `$`) from a project `theme/` `.bxm` override (see
`bx-sites-themes`):

```bx title="docs/functions.bxs"
function $shout( text ) {
	return uCase( arguments.text ) & "!"
}

function $badge( label, kind = "info" ) {
	return '<span class="badge bg-' & arguments.kind & '">' & arguments.label & '</span>'
}
```

```markdown
{{ $shout('this is important') }}

Status: {{ $badge('Stable', 'success') }}
```

A magic function can return anything `toString()`-able (plain text, HTML, a
number) - it's spliced into the page's Markdown *before* conversion, so
returning real HTML works exactly as expected. A function without a leading
`$` is a private helper other `$`-functions in the same file can call bare;
`{{ }}` can never call it directly.

**Calling from a theme override** - a magic function is bound directly into
template scope, so `theme/page.bxm`/`layout.bxm` can call it bare:

```bx title="theme/page.bxm"
<p class="build-banner">#$shout( 'built with boxlang' )#</p>
```

### Context variables

Available bare, with no argument, inside any magic function body:

| Variable | What it is |
|---|---|
| `siteConfig` | The resolved `bxsites.yaml` config |
| `page` | The current page (see caveat below) |
| `nav` | This tree's own nav tree |
| `basePath` | Root-relative base path, ending with `/` |
| `versions` | Version-switcher entries - `[ { label, url } ]` |
| `currentVersion` | Which `versions` entry is rendering now |
| `locales` | Language-switcher entries - `[ { code, label, url, dir, flag } ]` |
| `currentLocale` | Which `locales` entry's code is rendering now |
| `currentLocaleDir` | `"ltr"`/`"rtl"` for the current locale |

**`page` isn't equally complete everywhere.** Called from Markdown, `page`
is this page's own struct as loaded from disk - `title`/`description`/
`tags`/`icon`/`summary`/`ogImage`/`urlPath`/`relativePath`/`body`/etc. exist,
but fields only known once the whole tree has converted (`toc`,
`prevPage`/`nextPage`, `breadcrumbs`, `editUrl`/`lastUpdated`, `iconHtml`,
`markdownUrl`, `canonicalUrl`) don't yet. Called bare from `page.bxm`, `page`
is fully enriched, all of those included. Every other context variable is
identical in both places.

### Argument syntax

Simple, comma-separated literals or variable references only - no nested
calls or expressions:

- Numbers: `{{ $discount(20) }}`
- Quoted strings: `{{ $greet('World') }}` or `{{ $greet("World") }}`
- Booleans: `{{ $badge('Beta', true) }}`
- A dotted variable reference: `{{ $greet(product.name) }}`

## Visualizer recipes

A magic function returning HTML is a general-purpose way to get GitBook-style
visual cells (a star rating, a colored chip, a progress bar) without a
database-backed column picker. Drop these (or adapt them) into
`docs/functions.bxs`:

```bx
function $stars( required numeric rating, numeric max = 5 ) {
	var filled = min( max( round( arguments.rating ), 0 ), arguments.max )
	var stars = repeatString( "★", filled ) & repeatString( "☆", arguments.max - filled )
	return '<span title="' & arguments.rating & ' out of ' & arguments.max & '" style="color:#f5a623;letter-spacing:2px">' & stars & '</span>'
}

function $badge( required string label, string kind = "info" ) {
	var palette = {
		"info"    : { "bg" : "#e0edff", "fg" : "#1d4ed8" },
		"success" : { "bg" : "#dcfce7", "fg" : "#15803d" },
		"danger"  : { "bg" : "#fee2e2", "fg" : "#b91c1c" },
		"warning" : { "bg" : "#fef9c3", "fg" : "#854d0e" }
	}
	var pick = palette.keyExists( arguments.kind ) ? palette[ arguments.kind ] : { "bg" : "#f1f5f9", "fg" : "#475569" }
	return '<span style="display:inline-block;padding:0.1em 0.6em;border-radius:999px;font-size:0.85em;font-weight:600;background:'
		& pick.bg & ";color:" & pick.fg & '">' & encodeForHTML( arguments.label ) & "</span>"
}

function $progress( required numeric percent ) {
	var pct = min( max( arguments.percent, 0 ), 100 )
	return '<span style="display:inline-block;width:120px;height:8px;background:#e5e7eb;border-radius:999px;overflow:hidden;vertical-align:middle"><span style="display:block;height:100%;width:'
		& pct & '%;background:#2563eb"></span></span> ' & pct & "%"
}

function $trend( required numeric value ) {
	var isUp = arguments.value >= 0
	var arrow = isUp ? "▲" : "▼"
	var color = isUp ? "#16a34a" : "#dc2626"
	var sign = isUp ? "+" : ""
	return '<span style="color:' & color & ';font-weight:600">' & arrow & " " & sign & numberFormat( arguments.value, "0.0" ) & "%</span>"
}
```

> **Note**: when writing these into a real `docs/functions.bxs`, every
> literal `#` in a hex color above must be doubled (`##`) inside a BoxLang
> string, since `#...#` is interpolation syntax - e.g. `"##f5a623"` not
> `"#f5a623"`. Shown single here for readability.

Usage: `` `{{ $stars(4) }}` ``, `` `{{ $badge('Stable', 'success') }}` ``,
`` `{{ $progress(72) }}` ``, `` `{{ $trend(4.2) }}` ``.

**Inside a table cell** - `{{ }}` resolves against raw Markdown before
tables are even parsed (see `bx-sites-markdown` for table syntax), so any
magic function works inside a pipe table cell, the closest thing here to
GitBook's Select/Rating columns:

```markdown
| Feature | Status | Rating |
| --- | --- | --- |
| Dark mode | {{ $badge('Stable', 'success') }} | {{ $stars(5) }} |
```

## Showing the syntax literally

A `{{ }}` inside a fenced code block (3+ backticks) is left completely
untouched. A `{{ }}` in *inline* code (single or double backticks) is
protected too. A `{{ }}` whose contents don't look like a variable path or
a `$name(...)` call (some other templating engine's own `{{ }}` shown in
prose) is left untouched rather than erroring - only a token that *looks*
like a variable/magic-function reference but doesn't resolve fails the
build.

## Scope

- `functions.bxs` is project-wide - one file, loaded once, available on
  every page across the main tree and every version/locale tree (see
  `bx-sites-blog-versioning-i18n`). No need to duplicate it into
  `docs/versions/<name>/` or `docs/i18n/<code>/`.

## Reserved names

A theme override calling a magic function bare works because every loaded
function is bound directly into that template's rendering scope, alongside
the built-ins it already reads. Avoid naming a private helper (no `$`
prefix) any of: `page`, `nav`, `siteConfig`, `themeDir`, `basePath`,
`moduleAssetsDir`, `versions`, `currentVersion`, `locales`,
`currentLocale`, `currentLocaleDir`, `strings`, `requiredFiles`,
`stringsResolver` - a `$`-prefixed magic function can never collide with any
of these since none start with `$`.

## Errors

- `BxSites.UnknownVariable` - a `{{ dotted.path }}` doesn't match anything in
  `variables`.
- `BxSites.UnknownFunction` - a `{{ $name(...) }}` doesn't match any
  `$`-function in `functions.bxs`.
- `BxSites.InvalidFunctions` - `functions.bxs` has a BoxLang syntax error.
- `BxSites.InvalidConfig` - `variables` is present but isn't an object.
