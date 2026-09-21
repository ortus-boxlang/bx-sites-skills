---
name: bx-sites-markdown
metadata:
  version: "1.0"
description: Write advanced Markdown in bx-sites (ortus-boxlang/bx-sites) - admonitions/callouts, footnotes, definition lists, content tabs, code-block annotations (line numbers, highlighted lines, diff markers, terminal frames, the live tryboxlang playground), Mermaid diagrams, math (KaTeX), GFM tables, icons, responsive images, and Alpine.js interactivity. Use this whenever a user wants a callout box, tabbed content, syntax-highlighted code with extras, a diagram, math notation, a table, an icon, an image, or a small interactive widget in bx-sites Markdown. For GitBook-style `::: name :::` blocks (cards, steppers, buttons, includes...), use bx-sites-content-blocks instead.
---

# BxSites Markdown Extensions

Beyond standard Markdown, bx-sites turns on admonitions, footnotes, and
definition lists (Flexmark extensions via bx-markdown), plus its own content
tabs, math, and fenced-code annotations. Configurable via `bxsites.yaml`'s
`markdown`/`mermaid`/`math` keys - see the `bx-sites-configuration` skill
for every key. Tables, `~~strikethrough~~`, `- [ ]` task lists and the
in-page TOC are always on with no toggle.

## Admonitions

On by default:

```markdown
!!! note "Heads Up"
    This is an admonition. Its content is regular markdown - **bold**,
    `code`, [links](../index.md) and lists all work exactly as normal.
```

The body must stay indented 4 spaces (or a tab); the block ends at the first
non-indented, non-blank line. Blank lines inside are fine (just start a new
paragraph). The type becomes the box's icon/color; omit the `"Title"` and
the type's own capitalized name is used. 12 canonical types (many synonyms
resolve to the same color):

| Type | Synonyms | Color |
|---|---|---|
| `note` | (fallback for unknown types) | Blue |
| `abstract` | `summary`, `tldr` | Light blue |
| `info` | `todo` | Cyan |
| `tip` | `hint`, `important` | Teal |
| `success` | `check`, `done` | Green |
| `faq` | `question`, `help` | Lime |
| `warning` | `caution`, `attention` | Orange |
| `fail` | `failure`, `missing` | Light red |
| `danger` | `error` | Red |
| `bug` | | Pink |
| `example` | | Purple |
| `quote` | `cite` | Gray |

**Collapsible**: prefix the type with `???` (starts collapsed) or `???+`
(starts open) instead of `!!!`; the heading is clickable to toggle either way.

Turn admonitions off entirely: `markdown: { enableAdmonition: false }`.

## Footnotes

Off by default - `markdown: { enableFootnotes: true }`.

```markdown
Here's a claim that needs backing up[^1].

[^1]: Here's the backup.
```

Definitions are collected and rendered as a numbered list at the bottom of
the page regardless of where they're written in source.

## Definition lists

Off by default - `markdown: { enableDefinitionLists: true }`.

```markdown
Term
:   Its definition.

Second term
:   First definition.
:   Second definition.
```

## Content tabs

Always on, no config. Group alternative content (languages, platforms)
behind clickable tabs - `=== "Title"`, body indented the same 4
spaces/tab as an admonition. Consecutive `=== "..."` blocks (at most one
blank line apart) form one tab group; a tab's content is full Markdown
(code fences, lists, admonitions, anything).

```markdown
=== "Java"
    ```java
    System.out.println( "Hi" );
    ```

=== "BoxLang"
    ```bx
    println( "Hi" )
    ```
```

## Code blocks

Fenced code, syntax-highlighted client-side (highlight.js) - the language
after the opening ` ``` ` selects the grammar. bx-sites also registers its
own BoxLang grammar under `bx`/`boxlang`/`bxs`/`bxm`/`cfscript`.

**Line numbers, highlighted lines, titles** - any combination on the fence's
info string, no config:

````markdown
```bx hl_lines="2" linenums="1" title="add.bx"
numeric function add( required numeric a, required numeric b ) {
	return a + b
}
```
````

`linenums="N"` starts the gutter at `N`; `hl_lines` takes space-separated
line numbers/ranges (`"2 4-6"`), counted from the top regardless of where
`linenums` starts; `title` adds a small title bar.

**Diff markers / terminal frame**:

````markdown
```bx title="add.bx" insert="3-4" delete="7"
...
```

```bash frame="terminal" title="user@boxlang"
box install bx-sites
```
````

`insert`/`delete` take the same space-separated numbers/ranges as `hl_lines`,
rendered as a tinted row + `+`/`–` gutter marker (spelled out, not `ins`/`del`,
kept as attributes not literal prefixes so the fence stays real copy-pasteable
source). `frame="terminal"` swaps the plain title bar for a macOS-style
terminal window; `frame="code"` is the explicit default. A fence tagged
`diff` with real `git diff` output is highlighted by highlight.js's own
`diff` grammar - no bx-sites-specific syntax needed.

**Live BoxLang playground** - tag a fence `tryboxlang` instead of a language
name to embed a live [try.boxlang.io](https://try.boxlang.io) editor:

````markdown
```tryboxlang title="Closures" height="450px" readonly="false"
user = { name: "Luis", getFullName: () => "Luis Majano" }
println( user.getFullName() )
```
````

`title` (none), `height` (`450px` default), `readonly` (`false` default) are
all optional.

## Diagrams

Opt-in: `mermaid: true` in `bxsites.yaml` (see `bx-sites-configuration`).
Then any ` ```mermaid ` fenced block renders as a live
[Mermaid](https://mermaid.js.org/) diagram (flowcharts, sequence diagrams,
class diagrams, Gantt charts, and more).

## Math

Opt-in: `math: true` in `bxsites.yaml`. [KaTeX](https://katex.org/) typesets
`$inline$` and `$$block$$` math written directly in Markdown. A `$`
immediately touching whitespace is left alone (`$5 and $10` isn't misread as
a formula).

## Tables

Standard GFM pipe tables, always on:

```markdown
| Feature      | Community | Enterprise |
| ------------ | :-------: | ---------: |
| Themes       |    10     |         10 |
| Multi-locale |    Yes    |        Yes |
```

- `---` under the header turns the table on; colons on that row control
  alignment (`:---` left, `:---:` center, `---:` right, no colons = left).
- Cell content is regular inline Markdown (`code`, **bold**, *italic*,
  links).
- A literal `|` inside a cell's plain text needs `\|` (inside inline code it
  doesn't - the code span already protects it).
- A short row is padded with empty cells; a long row's extra cells are
  dropped (`markdown.tableOptions.appendMissingColumns`/`discardExtraColumns`).
- Every table auto-wraps in `.bxsites-table-wrap` for horizontal scroll on
  wide tables and a sticky header on tall ones - no config, no markdown
  needed.
- For a status chip / star rating in a cell, use a magic function - see the
  `bx-sites-variables-functions` skill. For a reader-sortable/filterable
  table, use Alpine.js (below) instead of a plain pipe table.

## Layout

A page's `layout` frontmatter picks which template the active theme renders
it through, instead of that theme's defaults:

```markdown
---
title: We raised a Series A
layout: press-release
---
```

Resolves **both** the outer shell and the body - `layout: press-release`
renders through `.theme/press-release.bxm` (or the active built-in theme's
own, if it ships one) as the whole page, not just inside the normal
`layout.bxm` shell. `layout: home` is the built-in example: `bootstrap`'s
`home.bxm` is a marketing homepage with no sidebar/TOC rail that never
includes the page's own Markdown body. A `layout:` naming a file the active
theme doesn't have falls back to `layout.bxm`/`page.bxm` rather than failing
the build. Unset (the default) keeps a page on its default templates - a
blog post still gets `blog-page.bxm` (and blog listing pages `blog.bxm`)
when the active theme has them, unaffected either way. See
`bx-sites-themes`'s own "Multiple layouts per page" for the full resolution
chain and how to add a theme's own alternate templates.

## Icons

A page's `icon` frontmatter (and a nav/card/button `icon` attribute) accepts
either a plain emoji or a named icon from eight bundled libraries (no CDN):

```markdown
icon: rocket                    # bare name -> Phosphor, regular weight
icon: lucide:rocket             # Lucide
icon: phosphor-bold:rocket      # Phosphor, bold weight
icon: tabler:rocket             # Tabler
icon: custom:my-icon            # project's own docs/assets/icons/my-icon.svg
```

Phosphor ships all six weights, each its own prefix: `phosphor-thin:`,
`phosphor-light:`, `phosphor:`/bare (regular), `phosphor-bold:`,
`phosphor-fill:`, `phosphor-duotone:`. Names match the library's own gallery
exactly (lowercase, hyphenated, e.g. `book-open`, `arrow-up-right`). Font
Awesome is deliberately not bundled (its Duotone style/most of v6+ is
Pro-only, not redistributable). The same `[library:]name`/emoji syntax works
anywhere an `icon` is accepted (frontmatter, `nav.json` entries, card/button
attributes) - resolved through one shared cache, so referencing the same
icon twice only reads its SVG once.

## Images

Write an image the normal way - plain Markdown, file-relative to the page:

```markdown
![A freshly built site](../assets/screenshot.png)
```

Every eligible `docs/assets/**` image (`.png`/`.jpg`/`.jpeg`) automatically
gets resized/WebP variants and a `<picture>`/`srcset` rewrite at build time -
no new syntax, on by default. SVGs and animated GIFs are copied through
unchanged (already resolution-independent / frame-unaware resize would
flatten them); a remote `<img src="https://...">` is left untouched; an
image already narrower than every configured width is left as-is (unless
WebP re-encoding is on). Breakpoints/formats are set via `bxsites.yaml`'s
`assets.images` key - see `bx-sites-configuration`.

**Galleries** - for a responsive grid of images with an optional
click-to-enlarge lightbox, use the `::: image-gallery` content block (see
`bx-sites-content-blocks`) rather than hand-rolled HTML - it lists images
explicitly or discovers them from `docs/assets/gallery/{name}/`, and its
lightbox is enabled by `bxsites.yaml`'s `imageGallery: true`.

**Captions, alignment, framing** - plain block-level HTML passes
through untouched (CommonMark's own HTML-block rule), no bx-sites syntax:

```markdown
<figure>
  <img src="../assets/screenshot.png" alt="The build output">
  <figcaption>A freshly built site</figcaption>
</figure>
```

## Alpine.js interactivity

Every page already loads Alpine.js (it powers the built-in dark-mode toggle)
- drop `x-data`/`x-show`/`@click`/etc. straight onto raw HTML in your
Markdown, no config, no extra `<script>` tag. **Reach for a purpose-built
block first** when one exists (see `bx-sites-content-blocks`): an
expandable/collapsible admonition for a collapsible section, content tabs
for grouped alternatives, a stepper for a numbered walkthrough, a button for
a styled link. Alpine is for content with its own client-side state that
those don't cover - a copy-to-clipboard button, a live filter, a
client-sortable table:

```markdown
<div x-data="{ copied: false }">
  <button type="button" class="bxsites-button bxsites-button--secondary bxsites-button--small"
    @click="navigator.clipboard.writeText( 'box install bx-sites' ); copied = true; setTimeout( () => copied = false, 1500 )">
    <span x-show="!copied">Copy install command</span>
    <span x-show="copied" x-cloak>Copied!</span>
  </button>
</div>
```

For a sortable/filterable table, put the data in `x-data` and render rows
with `x-for`/`x-text` instead of pipe-table syntax - the hand-written
`<table>` still gets the same responsive-scroll/sticky-header treatment
automatically. Things to know: Alpine is core (can't be disabled), currently
vendored `alpinejs@3.14.1` (no CDN), and its default build needs
`unsafe-eval` under a strict CSP - don't rely on it if that's unavailable.
Keep widgets small and self-contained; this isn't for a full client-side app.
