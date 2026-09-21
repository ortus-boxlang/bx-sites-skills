---
name: bx-sites-variables-functions
metadata:
  version: "1.0"
description: Use reusable {{ variables }} and BoxLang "magic functions" in bx-sites (ortus-boxlang/bx-sites) Markdown - bxsites.yaml's variables block, docs/functions.bxs, context variables (page/siteConfig/nav/versions/locales), visualizer recipes (status badges, star ratings, progress bars, trend arrows) including inside table cells, and structured Data Files (docs/data/*.yaml/.toml/.json, or a docs/data/*.bx data class for computed data) reachable as data.<file>. Use this whenever a user wants to avoid repeating a fact across pages, wants a status chip/rating/progress bar, wants a team roster/pricing table/feature matrix reachable from every page, or wants to write reusable BoxLang logic callable from Markdown. For the ::: for/::: if content blocks that loop/branch over data.*, use bx-sites-content-blocks.
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
Markdown, and bare (no `$`) from a project `.theme/` `.bxm` override (see
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
template scope, so `.theme/page.bxm`/`layout.bxm` can call it bare:

```bx title=".theme/page.bxm"
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

## Data Files

Reusable variables are great for a flat, one-off fact (`company`,
`supportEmail`) but awkward for anything with real shape - a team roster, a
pricing table, a feature matrix. Drop a `docs/data/*.yaml`/`.yml`/`.toml`/
`.json` file instead, and its whole content - object or array, any shape -
becomes reachable as `data.<file>` from every page, the same `{{ }}` syntax
`variables`/`page` already use. Each file's basename (extension stripped)
is one top-level key under `data`:

```text title="docs/ layout"
docs/
├── index.md
└── data/
    ├── team.yaml
    └── pricing.json
```

```yaml title="docs/data/team.yaml"
- name: Luis Majano
  role: CEO
- name: Jon Clausen
  role: CTO
```

```json title="docs/data/pricing.json"
{ "free": { "price": 0, "seats": 3 }, "pro": { "price": 29, "seats": 20 } }
```

```markdown title="docs/pricing.md"
The Pro plan is **${{ data.pricing.pro.price }}/mo** for up to
{{ data.pricing.pro.seats }} seats.
```

No `docs/data/` folder at all simply means no `data` - the same
opt-in-by-presence shape `docs/functions.bxs`/`docs/blog/authors.yml`
already use. If more than one file shares a basename across extensions,
`.bx` wins first (see Data classes below), then `.yaml`, `.yml`, `.toml`,
`.json` - pick one format per basename rather than relying on that order.

### Data classes

For data that needs *computing* rather than just parsing (a discounted
price, a value assembled from several sources), drop a real BoxLang class
instead - `docs/data/Pricing.bx` (PascalCase) becomes `data.pricing` (class
basename, first letter lowercased):

```bx title="docs/data/Pricing.bx"
class {
	struct function getData() {
		return { "free": { "price": 0 }, "pro": { "price": 29 } }
	}

	numeric function getDiscountedPrice( required string plan, required numeric pct ) {
		var base = getData()[ arguments.plan ].price
		return base - ( base * arguments.pct )
	}
}
```

**`getData()` is required** on every data class - it's what runs
automatically whenever `data.pricing` is used bare, exactly like a parsed
YAML/JSON root. Any other public method is callable too, directly from
`{{ }}`, with the same argument syntax a magic function call already uses:

```markdown
Discounted for early adopters: **${{ data.pricing.getDiscountedPrice("pro", 0.2) }}/mo**
```

Only public methods are reachable this way - a `private function` stays a
genuine implementation detail, same as a non-`$` helper in `functions.bxs`.
A `.bx` file under `docs/data/` is the same project-owner-authored trust
tier as `functions.bxs` - never something a docs-only contributor's
Markdown can reach into.

**One narrow limitation**: loading a data class needs its own resolved path
to be a valid BoxLang class name (no hyphens/spaces). Running `bxSites` from
inside the project (the common case) always works; it only bites with an
explicit `--projectRoot` outside the current directory whose own path
contains a hyphen/space - see `BxSites.UnsupportedDataClassPath` below.

### Consuming data

A scalar `{{ data.x.y }}` works anywhere `{{ }}` already does. For a loop -
a team grid, a pricing table - there are three places to put it:

- **A theme override** - `data` is bound bare into `layout.bxm`/`page.bxm`
  the same way `page`/`siteConfig` already are (see `bx-sites-themes`); the
  natural home for data that belongs on *every* page. A data class here is
  the live instance itself - call `getData()` explicitly, no `{{ }}`-only
  auto-invoke.
- **A magic function** - reads `data` bare (one of the same "supporting
  variables" `page`/`siteConfig` already are), loops/branches with real
  BoxLang, returns a Markdown/HTML fragment. Renders server-side, visible to
  a search crawler with no JavaScript needed.
- **`::: for`/`::: if` directly in Markdown** - see `bx-sites-content-blocks`
  for the full syntax; a dotted-path-only loop/conditional, no comparison
  operators, that binds straight from a page with no magic function needed.

### Scope

`docs/data/` is project-wide, loaded once - the same single-load scope
`functions.bxs` already has. Every version/locale tree sees the identical
`data`; don't duplicate it into `docs/versions/<name>/` or
`docs/i18n/<code>/`, it isn't read from there. Flat directory only, no
subfolder recursion in this first version. `data` is a reserved `{{ }}`
name like `page` - a `bxsites.yaml` `variables.data` entry is shadowed by
`docs/data/`'s own struct, and `functions.bxs` can't declare a function
named `data`.

### Data file errors

- `BxSites.InvalidDataFile` - a data file failed to parse, or a data class
  failed to compile/instantiate.
- `BxSites.MissingDataMethod` - a data class has no public `getData()`.
- `BxSites.UnknownDataMethod` - `{{ data.x.someMethod(...) }}` names a
  method that doesn't exist (or isn't public) on that data class.
- `BxSites.NotCallable` - calling a method on a `.yaml`/`.json`-backed key
  (no methods to call, it isn't a data class instance).
- `BxSites.UnsupportedDataClassPath` - a data class's resolved path contains
  a character invalid in a BoxLang class name.
- `BxSites.InvalidForTarget` - a `::: for`'s path resolved to something
  that's neither an array nor a struct.

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
`stringsResolver`, `data` - a `$`-prefixed magic function can never collide
with any of these since none start with `$`.

## Errors

- `BxSites.UnknownVariable` - a `{{ dotted.path }}` doesn't match anything in
  `variables`.
- `BxSites.UnknownFunction` - a `{{ $name(...) }}` doesn't match any
  `$`-function in `functions.bxs`.
- `BxSites.InvalidFunctions` - `functions.bxs` has a BoxLang syntax error.
- `BxSites.InvalidConfig` - `variables` is present but isn't an object.
