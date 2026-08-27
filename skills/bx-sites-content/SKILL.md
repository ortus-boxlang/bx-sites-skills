---
name: bx-sites-content
metadata:
  version: "1.0"
description: Create and author content for a bx-sites (ortus-boxlang/bx-sites) project - scaffolding docs/pages/posts/versions/locales, page frontmatter and nav, GitBook-style content blocks (cards, tabs, expandables, steppers, buttons, prompts, embeds, includes, conditional content), Markdown extensions (admonitions, footnotes, math, Mermaid diagrams, tables, code annotations), reusable {{ variables }} and magic functions, the blog, versioning, i18n, redirects, images/icons, and content-quality checks (lint/check/stats). Use this whenever a user wants to write, structure, restyle, or enrich a bx-sites project's Markdown content, or scaffold new pages/posts/versions/locales.
---

# BxSites Content Authoring

BxSites (`ortus-boxlang/bx-sites`) is a BoxLang-based static site generator in
the spirit of mkdocs/GitBook: point it at a `docs/` (or `src/`) folder of
Markdown and it builds a themed, searchable static site. This skill covers
**writing and structuring content** in a bx-sites project. For build/serve,
theming, plugins, search providers, and deployment, use the
`bx-sites-publishing` skill instead.

Every command below runs as `bxSites <verb> [options]` (or
`boxlang bxSites <verb> [options]` where the `PATH` shim isn't installed,
e.g. CI). Every verb accepts `--projectRoot=<path>` to target a project other
than the current directory. **CLI flags always use `--flag=value`, never a
bare positional value**, for a verb's primary argument.

## Project layout

```text
my-docs/
├── bxsites.yaml            # site config (bxsites.json also supported)
├── deployments/*.json      # deploy targets (see bx-sites-publishing)
├── theme/                  # optional project theme override (see bx-sites-publishing)
└── docs/                   # or src/ - every .md file here is a page
    ├── index.md
    ├── functions.bxs       # magic functions, see reference/variables-functions.md
    ├── nav.json            # optional explicit nav (alternative to bxsites.yaml's nav key)
    ├── 404.md               # optional custom 404 page
    ├── robots.txt           # optional hand-authored robots.txt (overrides generated one)
    ├── assets/              # images, downloads, icons - copied to site/assets/
    ├── includes/            # reusable content fragments, spliced via ::: include
    ├── blog/
    │   ├── authors.yml
    │   └── posts/            # every .md here (any depth) is a blog post
    ├── versions/
    │   └── <name>/            # a full docs/-shaped tree, snapshot via version:new
    └── i18n/
        └── <code>/             # a full docs/-shaped tree, one per locale
```

Folder nesting under `docs/` becomes nav nesting automatically (override with
an explicit `nav`, see `bx-sites-publishing`'s configuration reference).
`assets/`, `blog/`, `versions/`, and `i18n/` are reserved folder names with
special meaning - don't repurpose them for ordinary content.

## Scaffolding

| Verb | Purpose |
|---|---|
| `new [path] [--name=] [--theme=] [--description=] [--format=yaml\|json]` | Scaffold a new project (`docs/` + `bxsites.yaml`) |
| `page:new --path=guides/setup.md [--title=] [--description=] [--icon=] [--tags=] [--order=]` | Scaffold a single page at an arbitrary `docs/`-relative path, frontmatter pre-filled |
| `page:rename --from=guides/old.md --to=guides/new.md` | Move a page, rewrite every relative Markdown link that pointed at it, and stamp `redirect_from` on it automatically |
| `post:new --title="My Post" [--slug=] [--date=] [--authors=] [--categories=] [--tags=] [--draft]` | Scaffold a blog post at `docs/blog/posts/<slug>.md` (`--draft` defaults `true`; pass `--!draft` to publish immediately) |
| `version:new --name=1.0` | Snapshot the *current* `docs/` tree into `docs/versions/1.0/` (excludes `assets/`, `versions/`, `i18n/`, `blog/`) |
| `i18n:new --code=es` | Scaffold `docs/i18n/es/`, seeding `index.md` from the default locale's own |
| `migrate --source=<path> [--from=gitbook\|mkdocs\|markdown-zip\|notion]` | Convert an existing GitBook export, mkdocs project, plain zip of Markdown, or Notion export into `docs/` + `nav.json` |

## Page frontmatter

```markdown
---
title: Deployment
order: 2
hidden: false
description: How to deploy a built BxSites site.
tags: [guides, deployment]
icon: 🚀
summary: Everything you need to publish a built site.
ogImage: assets/deployment-card.png
toc: true
redirect_from: [guides/old-deploy-guide]
---

# Deployment
Your content here.
```

- `title` - nav/page title (defaults to filename)
- `order` - sibling sort order in nav (lower first; omitted sorts last, alphabetically)
- `hidden: true` - excludes from nav and search, still built
- `description` - meta description / social card (falls back to site-wide `description`)
- `tags` - array; renders as badges and feeds the site-wide `/tags/` index; boosts search relevance
- `icon` - emoji or a named icon (`rocket`, `lucide:rocket`, `phosphor-bold:rocket`, `tabler:rocket`, `custom:my-icon` from `docs/assets/icons/my-icon.svg`) - see [reference/markdown-extensions.md](reference/markdown-extensions.md#icons)
- `summary` - one-line lead-in rendered under the title (distinct from `description`, which is meta-only)
- `ogImage` - per-page social card image, overrides the site-wide one
- `toc: false` - hides the page's own "On this page" TOC even with 2+ headings
- `redirect_from` - array of old pretty-URL segments (no leading/trailing slash, no extension) that should redirect here; `page:rename` stamps this automatically - see [reference/blog-versioning-i18n.md](reference/blog-versioning-i18n.md#redirects)

It's a small hand-rolled parser, not full YAML: inline lists (`tags: [a, b]`),
block lists (`- item`), and `>`/`|` block scalars work; nested
objects/maps do not.

## Linking between pages

Link with a file-relative path to the target's `.md` source, exactly as if
browsing the repo on disk - `[Deployment](guides/deployment.md)`,
`[back](../getting-started.md#add-pages)`. BxSites rewrites it to the built
pretty-URL at build time, resolved against the *linking* page's own folder.
Absolute URLs, `mailto:`, and links already starting with `/` are left alone.

## Advanced content options

Everything below is available in every page's Markdown with **no
`bxsites.yaml` config needed**, unless noted:

| Need | Syntax | Reference |
|---|---|---|
| Callout/note box | `!!! note "Title"` (indented body) | [markdown-extensions.md#admonitions](reference/markdown-extensions.md#admonitions) |
| Collapsible callout | `??? tip "..."` / `???+ tip "..."` | same |
| Footnotes *(opt-in)* | `text[^1]` ... `[^1]: note` | same |
| Definition lists *(opt-in)* | `Term\n:   Definition` | same |
| Content tabs | `=== "Title"` (indented body, repeat) | same |
| Code block extras | ` ```lang hl_lines="2" linenums="1" title="x" insert="3-4" delete="7" frame="terminal" ` | same |
| Live BoxLang playground | ` ```tryboxlang title="..." height="450px" readonly="false" ` | same |
| Diagrams *(opt-in `mermaid: true`)* | ` ```mermaid ` fenced block | same |
| Math *(opt-in `math: true`)* | `$inline$` / `$$block$$` | same |
| GFM tables | pipe tables, always on, responsive-scroll wrapper automatic | [markdown-extensions.md#tables](reference/markdown-extensions.md#tables) |
| Icons | `icon: rocket` / `lucide:rocket` / `custom:my-icon` | [markdown-extensions.md#icons](reference/markdown-extensions.md#icons) |
| Collapsible section (plain) | `::: expandable "Q?"` ... `:::` | [content-blocks.md#expandable](reference/content-blocks.md#expandable) |
| Card grid | `::: cards` > `::: card title=... icon=... href=...` | [content-blocks.md#cards](reference/content-blocks.md#cards) |
| Side-by-side layout | `::: columns` > `::: column width="60%"` | [content-blocks.md#columns](reference/content-blocks.md#columns) |
| Numbered walkthrough | `::: stepper` > `::: step "Title" color="success"` | [content-blocks.md#stepper](reference/content-blocks.md#stepper) |
| Download card | `::: file src="assets/spec.pdf" title="..."` | [content-blocks.md#file](reference/content-blocks.md#file) |
| Print/PDF page break | `::: pagebreak` | [content-blocks.md#page-break](reference/content-blocks.md#page-break) |
| CTA button(s) | `::: button "Label" href="..." style="primary"` (wrap several in `::: buttons`) | [content-blocks.md#buttons](reference/content-blocks.md#buttons) |
| Embedded video/media | `::: embed url="https://youtube.com/..."` | [content-blocks.md#embed](reference/content-blocks.md#embed) |
| Rich internal page preview | `::: page-link href="../guide.md"` | [content-blocks.md#page-link](reference/content-blocks.md#page-link) |
| Rich external link preview | `::: link-preview url="..." title="..."` | [content-blocks.md#link-preview](reference/content-blocks.md#link-preview) |
| Reusable AI prompt block | `::: prompt description="..." expanded="preview"` | [content-blocks.md#prompt](reference/content-blocks.md#prompt) |
| Dated changelog list | `::: updates` > `::: update date="YYYY-MM-DD" tags="..."` | [content-blocks.md#updates-changelog](reference/content-blocks.md#updates-changelog) |
| Reusable content snippet | `::: include src="beta-notice.md"` (files live in `docs/includes/`) | [content-blocks.md#reusable-content-includes](reference/content-blocks.md#reusable-content-includes) |
| Reader-toggled variant content | `::: audience-switcher key=... options=...` + `::: conditional key=... value=...` | [content-blocks.md#conditional-content](reference/content-blocks.md#conditional-content) |
| Interactive API reference *(opt-in `openapi: true`)* | `::: openapi src="assets/openapi/api.yaml" [operation="GET /books"]` | [content-blocks.md#openapi](reference/content-blocks.md#openapi) |
| Reusable site facts | `{{ dotted.path }}` from `bxsites.yaml`'s `variables` | [variables-functions.md](reference/variables-functions.md) |
| Reusable BoxLang logic (badges, ratings, progress bars, ...) | `{{ $name(args) }}` from `docs/functions.bxs` | [variables-functions.md](reference/variables-functions.md) |
| Responsive images | plain `![alt](../assets/img.png)` - resize/WebP/`<picture>` rewrite is automatic | [markdown-extensions.md#images](reference/markdown-extensions.md#images) |
| Client-side interactive widgets (filter, sort, copy button) | plain Alpine.js `x-data`/`x-show`/`@click` HTML blocks | [markdown-extensions.md#alpinejs-interactivity](reference/markdown-extensions.md#alpinejs-interactivity) |

`::: name ... :::` blocks nest (an expandable can contain a card grid, for
example) and each maps directly to a GitBook block of the same name, which is
why a GitBook export migrates cleanly with `bxSites migrate --from=gitbook`.

## Blog, versioning, and i18n

All three are **convention over configuration** - no `bxsites.yaml` key turns
them on, just a folder:

- **Blog** - `docs/blog/posts/*.md` (any depth). `date`/`authors`/`categories`/
  `tags`/`summary`/`image`/`slug`/`draft` frontmatter; `docs/blog/authors.yml`
  for author bios. Builds `/blog/`, per-category/per-year/per-author pages,
  RSS, and `/blog/stats/`. Full reference: [reference/blog-versioning-i18n.md#blog](reference/blog-versioning-i18n.md#blog).
- **Versioning** - `docs/versions/<name>/`, cut with `bxSites version:new
  --name=1.0`. Full reference: [reference/blog-versioning-i18n.md#versioning](reference/blog-versioning-i18n.md#versioning).
- **i18n** - `docs/i18n/<code>/`, mirroring `docs/` page-for-page; register
  the locale's label/direction in `bxsites.yaml`'s `i18n.locales`. Full
  reference: [reference/blog-versioning-i18n.md#i18n](reference/blog-versioning-i18n.md#i18n).
- They compose: `docs/versions/<name>/i18n/<code>/`.

## Content-quality workflow

Run these before calling content "done" - none of them mutate `docs/`:

| Verb | Checks | Exit code |
|---|---|---|
| `lint` | Heading-level skips (`##` → `####`); blog posts with a missing/invalid `date` | `1` if anything found |
| `build` then `check` | Broken internal links/images; missing `alt` text (fails); orphaned pages (informational only) | `1` if broken links/images or missing alt |
| `build` then `stats` | Read-only report: pages/words, versions/locales, blog counts, tags, search-index size, output size | always `0` |
| `blog:drafts` | Lists every post with `draft: true` (excluded from real builds) | always `0` |
| `blog:find [--author=] [--category=] [--tag=] [--since=] [--until=] [--drafts]` | Filter posts without a full build | always `0` |
| `search:query --query="..." [--limit=10]` | Sanity-check what a visitor's search would surface, against a built `search-index.json` | - |
| `doctor` | Environment/config health check (JVM, `docs/` exists, config parses, required modules installed, theme override valid) | `1` if any check fails |

Typical pass before telling a user content is ready:

```bash
bxSites lint
bxSites build
bxSites check
```

## Output expectations

When scaffolding or editing content, report: which file(s) were
created/changed, the exact CLI command used (if any), and - for a
content-quality pass - the specific `lint`/`check` findings with file:line
context and the minimal fix. Prefer the scaffolding verbs (`page:new`,
`post:new`, `page:rename`, ...) over hand-writing frontmatter from scratch,
since they fill in required fields and (for `page:rename`) keep links/redirects
correct automatically.
