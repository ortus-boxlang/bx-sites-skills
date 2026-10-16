---
name: bx-sites-content-blocks
metadata:
  version: "1.0"
description: Write GitBook-style content blocks in bx-sites (ortus-boxlang/bx-sites) Markdown - expandables, card grids, columns, steppers, responsive image galleries, download/file cards, page breaks, buttons, embeds, page-link/link-preview cards, reusable AI prompt blocks, dated changelogs (updates), reusable content includes, reader-toggled conditional content, the OpenAPI/Swagger widget, a premium contact form, data-driven ::: for/::: if loops and conditionals, a course index, bxSites Cloud content blocks (`::: cloud`), and a moderated comment thread (`::: comments`). All use the same `::: name ... :::` container syntax. Use this whenever a user wants to add a card, tabs-like grid, stepper, CTA button, embed, contact form, Cloud content block, comments thread, data-driven loop, or any `::: ... :::` block to a bx-sites page. For plain Markdown extensions (admonitions, tabs, code annotations, math, tables, icons), use bx-sites-markdown instead; for the underlying docs/data/*.yaml files and data classes, use bx-sites-variables-functions; for course manifests, use bx-sites-blog-versioning-i18n.
---

# BxSites Content Blocks

GitBook-style content blocks, on top of everything in the `bx-sites-markdown`
skill. Every block uses the same `::: name ... :::` container syntax - a
bare `:::` on its own line closes whichever block is currently open. Blocks
can nest (an expandable containing a card grid, for instance). No
`bxsites.yaml` config needed unless noted. Each maps to a GitBook block of
the same name, which is why `bxSites migrate --from=gitbook` (see
`bx-sites-getting-started`) can convert them directly.

Good to know across all blocks:

- A block with no body can self-close on one line:
  `::: file src="a.pdf" :::` is the same as the two-line form.
- An unknown `::: name` isn't an error - its content renders inside a
  `bxsites-directive-unknown` wrapper.
- Child-only blocks (`card`, `column`, `step`, `update`, `image`, and a
  `button` nested in `buttons`) only make sense inside their parent
  (`cards`, `columns`, `stepper`, `updates`, `image-gallery`, `buttons`).

## Expandable

Plain collapsible section - no callout icon/color (for that, use a
collapsible admonition `???`, see `bx-sites-markdown`).

```markdown
::: expandable "Is this different from a collapsible admonition?"
Yes - this has no type/icon/color, just a plain expand/collapse section.
Add `open="true"` to start it expanded.
:::
```

## Cards

A grid of link cards. `title`, `icon`, `image`, `href` are all optional (no
`href` renders a non-clickable card). `icon` resolves the same way
frontmatter `icon` does (see `bx-sites-markdown`). If a card has both,
`image` wins and `icon` is ignored.

```markdown
::: cards
::: card title="Getting Started" icon="phosphor-duotone:rocket-launch" href="../getting-started.md"
Install, scaffold and build your first site.
:::
::: card title="Themes" icon="phosphor-duotone:palette" href="themes.md"
Customize a built-in theme or write your own.
:::
:::
```

## Columns

Side-by-side layout. `::: column` accepts an optional `width` (CSS
length/percentage, e.g. `"40%"`); columns with no explicit width share the
row equally. Only `<number>` plus `%`, `px`, `rem` or `em` is honored
(`"40%"`, `"20rem"`); anything else (`calc()`, `auto`, a bare number) is
ignored.

```markdown
::: columns
::: column width="60%"
The wider column.
:::
::: column
The narrower one.
:::
:::
```

## Stepper

Numbered, connected sequence of steps. Optional `color` on a step
(`success`, `warning`, `danger`, or omit for default) flags its marker
independent of position:

```markdown
::: stepper
::: step "Back up your data" color="success"
Routine, safe to run any time.
:::
::: step "Optional: enable telemetry" color="warning"
Skip this one if you're not sure.
:::
::: step "Delete the old install" color="danger"
Irreversible - make sure the backup above finished first.
:::
:::
```

Marker/line/palette colors are themeable via CSS custom properties (see the
`bx-sites-themes` skill).

## Image gallery

A responsive grid of images. Optional `columns` is `2`, `3`, or `4`
(default `3` - any other value falls back to `3`). Two ways to fill it:

List each image explicitly with `::: image` children - `src` is required,
`alt` and `caption` are both optional:

```markdown
::: image-gallery columns="3"
::: image src="../assets/favicon.png" alt="BoxLang icon" caption="Icon"
:::
::: image src="../assets/home-banner.jpg" alt="Home banner"
:::
::: image src="../assets/og-image.png" caption="Social preview"
:::
:::
```

Or let bxSites discover them by convention - drop files named
`{name}-{order}.{jpg,jpeg,png,webp,gif}` into `docs/assets/gallery/{name}/`
and reference them by `name` alone, no children needed. Files are sorted
numerically by `{order}`:

```markdown
::: image-gallery name="showcase" columns="3"
:::
```

- Convention mode carries no per-image `alt`/`caption` - use `::: image`
  children when either matters. A gallery with neither `::: image` children
  nor a `name`, or an `::: image` missing its `src`, renders a visible
  "invalid" placeholder in place of that item rather than failing the build.
- `src` is file-relative, so every image still picks up the usual responsive
  `<picture>`/WebP treatment for free (see `bx-sites-markdown`).
- The **click-to-enlarge lightbox** needs `imageGallery: true` in
  `bxsites.yaml` (default `false`) - the same opt-in pattern `openapi`/
  `mermaid`/`math` already use, and wired into all ten themes. Unset, the
  grid still renders, just without the lightbox JS.
- With `imageGallery: true`, **every content image** on the page is
  click-to-enlarge, not only `::: image-gallery` items. Keyboard: `Escape`
  closes, `ArrowLeft`/`ArrowRight` cycle through ALL zoomable images on the
  page in document order (it wraps, and hides prev/next when only one image
  is zoomable).
- Opt out per image with `class="no-zoom"` or `data-no-zoom`. Images under
  120px wide, and images inside `a[href]`, `button`, `nav`, `header`,
  `footer` or `.bxsites-card`, are skipped automatically. An image outside
  `<main>`/`<article>` can opt in with `data-bxsites-lightbox`.

## File

A download card for a PDF, video, or any other asset. `src` is resolved
against the site root (the content root, with `basePath` prefixed), not
against `docs/assets/` - so the `assets/` segment must be written out
(`assets/spec.pdf` for `docs/assets/spec.pdf`). An absolute URL or a
`/`-rooted path is left untouched.

```markdown
::: file src="assets/spec.pdf" title="API Specification"
:::
```

## Page break

Forces a page break for print/PDF export (e.g. browser "Print to PDF"). No
attributes; renders as a subtle divider on screen, only forces a real break
in print output.

```markdown
::: pagebreak
:::
```

## Buttons

GitBook-style call-to-action button - `::: button` alone, or several in a
row inside `::: buttons`. Only `"Label"` and `href` are required for most
buttons:

```markdown
::: buttons
::: button "Read the docs" href="../getting-started.md" icon="phosphor-duotone:book-open" size="large"
:::
::: button "Star on GitHub" href="https://github.com/org/repo" style="secondary" target="_blank"
:::
::: button "Coming soon" disabled="true"
:::
:::
```

Optional attributes:

- `style="primary"` (solid accent) or `"secondary"` (default, outline)
- `size="small"`, `"medium"` (default), `"large"`
- `icon="..."` - same resolution as a card's icon
- `target="_blank"` - opens in a new tab (`rel="noopener noreferrer"` added automatically)
- `disabled="true"` - inert, unclickable, no `href` needed (a button with
  no `href` at all also renders inert)

## Embed

Responsive iframe for a recognized provider: YouTube, Vimeo, CodePen,
Spotify, Loom, Figma. Anything else falls back to a plain "visit ↗" link
card instead of a broken iframe.

```markdown
::: embed url="https://www.youtube.com/watch?v=dQw4w9WgXcQ" title="A demo"
:::
```

## Page link

Rich preview card linking to another page in this site - `href` follows the
same file-relative convention as an ordinary page link. Title/icon/summary
are pulled automatically from the target page's own frontmatter (stays in
sync if that page is renamed).

```markdown
::: page-link href="../getting-started.md"
:::
```

## Link preview

Same card shape as page-link, but for an *external* URL - no page to pull
metadata from, so every field is an explicit attribute. Only `url` is
required; `title` falls back to the bare URL; `description`/`image` are
optional. No build-time fetch of the target (keeps builds fast/reliable).

```markdown
::: link-preview url="https://boxlang.io" title="BoxLang" description="A dynamic, multi-paradigm JVM language." image="https://boxlang.io/og.png"
:::
```

## Prompt

A styled, copyable container for a reusable AI prompt (bx-sites' equivalent
of GitBook's Prompt block). The block body *is* the prompt text (full
Markdown - headings/lists/code all format normally); gets a "Copy" button
that copies the exact source text.

```markdown
::: prompt description="Summarizes a pull request for a changelog entry" icon="phosphor-duotone:git-pull-request"
Summarize the following pull request diff as a single changelog entry,
written for an end user rather than a developer.
:::
```

`description` (one-line summary) and `icon` (defaults to a sparkle glyph)
are optional. `expanded="preview"` clamps a long prompt to a fade-out
preview until clicked; `expanded="hidden"` starts fully collapsed; omit (or
`"full"`) to always show it in full. `collapsed` is an alias of `hidden`
and `partial` an alias of `preview`. There is no "Open in AI providers"
menu - bx-sites never talks to a third-party AI provider.

## Updates (changelog)

Dated, taggable changelog list. `::: update` accepts `date="YYYY-MM-DD"` and
an optional comma-separated `tags`.

```markdown
::: updates
::: update date="2026-01-15" tags="feature,fix"
Added dark mode and fixed a footer alignment bug.
:::
::: update date="2026-01-01"
Initial release.
:::
:::
```

A page containing `::: updates` also gets its own `feed.xml` once
`bxsites.yaml`'s `baseURL` is a full URL (same requirement as `sitemap.xml`
- see `bx-sites-configuration`).

## Reusable content (includes)

`::: include src="..."` splices another file's raw Markdown in at that
point - becomes real page content (its own headings/paragraphs/nested
blocks), not a wrapped widget. Put partials under `docs/includes/` (a
reserved folder, like `assets/`/`versions/`/`i18n/`/`blog/`) - files there are
never built as their own page and never appear in nav/search/sitemap/tags.

- A **bare** `src` (no leading `./`/`../`) always resolves against the
  current tree's own `docs/includes/`, regardless of how deep the including
  page is nested - `::: include src="beta-notice.md"` or
  `::: include src="legal/terms.md"` for a subfolder.
- A `./`/`../`-prefixed `src` resolves file-relative to the *including*
  page's own directory instead, the same convention as an ordinary page link.
- A version/locale tree gets its own `includes/` the same way -
  `docs/versions/2.0/includes/`, `docs/i18n/es/includes/` - not shared with
  the main tree's `docs/includes/`.
- An included file can itself include another; a circular chain throws
  `BxSites.CircularInclude` at build time.
- The include line must stand alone, with no closing `:::`. Unlike other
  body-less blocks, `::: include src="x" :::` on one line is NOT recognized
  and is left as literal text.

## Conditional content

Shows one of several variants of a block based on the *reader's own choice*
(remembered in their browser's `localStorage`) - there's no server-side
identity in a static site.

```markdown
::: audience-switcher key="plan" options="free:Free,pro:Pro"
:::

::: conditional key="plan" value="free"
The Free plan includes basic search.
:::

::: conditional key="plan" value="pro"
The Pro plan adds AI-assisted search and unlimited team seats.
:::
```

- `::: conditional key="..." value="..."` marks one variant. `key` is
  whatever preference you're switching on (`"plan"`, `"os"`, `"language"`,
  anything); `value` is the setting this variant shows for. **Every variant
  always renders in the HTML** (hidden client-side, never omitted), so a
  reader with JS disabled or a search crawler still sees every variant.
- `::: audience-switcher key="..." options="value:Label,value:Label,..."` is
  an optional ready-made control - one button per option, switching every
  `::: conditional` block sharing that `key` anywhere on the page. Not
  required: a link ending `?plan=pro` sets the same preference on load, and
  `window.bxSitesSetPreference( key, value )` can drive it from custom UI.

## OpenAPI

Interactive Swagger UI widget for an OpenAPI/Swagger spec (JSON or YAML),
GitBook's OpenAPI block equivalent. Requires `bxsites.yaml`'s `openapi: true`
(see `bx-sites-configuration`; unset, the placeholder renders but stays
inert and ships no extra JS/CSS). `src` resolves the same way as
`::: file` (site root plus `basePath`, so write the `assets/` segment).

```markdown
::: openapi src="assets/openapi/example.yaml" title="Bookshelf API"
:::
```

Add `operation="METHOD /path"` to drop just one endpoint inline instead of
the full reference (method is case-insensitive; path must match the spec
exactly, `{param}` placeholders included):

```markdown
::: openapi src="assets/openapi/example.yaml" operation="GET /books"
:::
```

Everything (schemas, "Try it out") renders client-side straight from the
spec - "Try it out" calls the spec's own `servers[0].url` directly from the
visitor's browser, so that server must allow CORS from wherever the docs are
hosted. No manual, spec-less version exists - if there's no spec yet, either
write just enough spec to cover one endpoint, or describe it as ordinary
content (a parameters table, fenced `http`/`json` request/response pairs,
optionally walked through with a `::: stepper`).

## Contact form

> **Premium feature.** A `::: contact-form` block always renders as a real,
> complete form - inputs, labels, submit button, all of it - but *submitting*
> it only works once the project's own bxSites Cloud plan includes working
> forms. On a plan that doesn't, a reader who submits anyway sees a friendly
> "upgrade to enable this form" message instead of their message going
> anywhere - nothing to configure here to turn that on or off.

A labeled contact/lead-gen form, submitted with `fetch()` to bxSites Cloud
rather than a page reload:

```markdown
::: contact-form id="demo-request" to="sales@acme.com" fields="name:text,email*:email,message:textarea" submitLabel="Send"
:::
```

- `id` - this form's own slug, matching a form configuration (notification
  target, spam filtering) set up in bxSites Cloud, not here - never
  validated against what actually exists there. Defaults to `"contact"`.
- `to` - optional, purely informational (real delivery routing is
  configured server-side by the account admin).
- `fields` - required; comma-separated `name:type` pairs. Add a trailing
  `*` right after a field's name (before its `:`) to mark it required, e.g.
  `email*:email`. Supported types: `text`, `email`, `textarea`, `number`; anything
  else falls back to a plain `text` input. A field's label is derived from
  its name (`full-name` -> "Full Name").
- `submitLabel` - the submit button's text; defaults to `"Send"`.

Every form carries a hidden honeypot field a real visitor never sees or
fills in - bxSites Cloud's own spam filtering uses it, no config needed.

## Loop and conditional (data-driven)

`::: for` and `::: if` render their content against data files -
`docs/data/*.yaml`/`.toml`/`.json`, or a `docs/data/*.bx` data class (see
`bx-sites-variables-functions`) - addressed by dotted path. Unlike every
block above, both take a bare expression instead of `key="value"`
attributes, deliberately as narrow as `{{ }}` itself: no comparison
operators in this first version.

```markdown
::: for member, idx in data.team
{{ idx }}. **{{ member.name }}** - {{ member.role }}
:::
```

`::: for <item>, <index> in <dotted.path>` binds `<item>`/`<index>` using
BoxLang's own two-variable `for` loop semantics - item + 1-based index for
an array, key + value for a struct, identical syntax either way:

```markdown
::: for name, enabled in data.flags
- {{ name }}: {{ enabled }}
:::
```

The path may also end in a method call on a data class,
e.g. `::: for plan, i in data.pricing.getPlans()` or
`::: if data.pricing.hasDiscount("pro")`.

`::: if <dotted.path>` renders only when the resolved value is truthy (an
empty array/struct/string, `0`, `false` all count as falsy). Chain any
number of `::: elseif <dotted.path>` and a trailing bare `::: else` for real
`if`/`elseif`/`else` semantics - the first truthy branch wins, a later
condition is never even resolved, and one trailing `:::` closes the whole
chain (no `:::` needed before each `elseif`/`else`, though it still parses
if you write it that way):

```markdown
::: if data.flags.darkModeDefault
Dark mode is on by default.
::: elseif data.flags.betaBanner
Beta features are enabled, though dark mode isn't on by default.
::: else
Nothing special about this build.
:::
```

Both bodies take ordinary Markdown and even other content blocks, including
a nested `::: for`/`::: if`. A real comparison (`==`, `&&`, ...) needs a
magic function instead - see `bx-sites-variables-functions`. Full picture on
`data.*` (theme override, magic function, or these two directives) is in
that skill's Data Files section.

`page` and `data` are **reserved loop-variable names** - `::: for page, i in
...` doesn't throw, it renders an inline `<div class="bxsites-directive-invalid">`
in place of the loop (they'd shadow the real context variables); pick any
other name. A malformed `::: for` spec, a `::: if` with no path, and a
`::: else` given a condition render that same invalid marker. Only two
things fail the build: a `::: for` whose path resolves to something that's
neither an array nor a struct (`BxSites.InvalidForTarget`), and an
unresolvable dotted path in either directive (`BxSites.UnknownVariable`).

## Course index

`::: course id="..." :::` renders a whole course's lessons as one numbered,
linked index, built from a `docs/data/courses.yaml` manifest rather than
hand-authored:

```markdown
::: course id="getting-started" :::
```

Takes only a bare `id` - no `href`/body of its own; the lessons and their
order come entirely from the manifest. See the `bx-sites-blog-versioning-i18n`
skill's Courses section for the manifest format, scoped lesson-to-lesson
navigation, and reader progress tracking.

## Cloud content block

`::: cloud slug="press/release1"` bakes a published bxSites Cloud content
block into the page at build time (sanitized HTML, so it reads with no JS and
is real SEO text). It is the one directive that makes an outbound HTTP
request during a build.

```markdown
::: cloud slug="press/release1"
:::
```

- Needs a bxSites Cloud account with Content Blocks, plus in `bxsites.yaml`
  `cloud.contentBlocks: true` and `cloud.apiUrl` (your Cloud instance), and
  the `BXSITES_CLOUD_TOKEN` environment variable. Without
  `cloud.contentBlocks`/`cloud.apiUrl` the build fails with
  `BxSites.CloudContentNotConfigured`; with it on but no token, with
  `BxSites.MissingCloudToken`.
- `cloud.contentBlocksOnError`: `"fail"` (default, throws
  `BxSites.CloudFetchFailed`) or `"skip"` (warns and renders an empty
  placeholder). A draft or expired block counts as a failed fetch.
- `cloud-content-init.js` (shipped when `cloud.contentBlocks` is on) swaps in
  a fresher version on page load if one exists.

## Comments

`::: comments` (or `::: comments id="..."`) mounts a moderated visitor
comment thread. Nothing is fetched at build time; `comments-init.js` loads
the thread client-side on every page view.

```markdown
::: comments
:::
```

- Needs bxSites Cloud Comments and `cloud.comments: true` in `bxsites.yaml`.
  Without `cloud.comments`, the build does not fail: the block renders an
  inert, empty `bxsites-comments--disabled` container, no `comments-init.js`
  is shipped, and the build prints one warning naming the first page
  affected. (`::: cloud` is different: it still fails the build without
  `cloud.contentBlocks`, because it has to fetch real content.)
- The thread id defaults to the page's `urlPath` (one thread per page); set
  `id` to share a thread across pages or to host two on one page.
