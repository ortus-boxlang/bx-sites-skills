---
name: bx-sites-getting-started
metadata:
  version: "1.0"
description: Install bx-sites (ortus-boxlang/bx-sites) and scaffold a new site, or bring an existing GitBook/mkdocs/Notion project into one - project layout, page frontmatter, linking between pages, and the basic build/serve/clean loop. Use this whenever a user wants to start a new bx-sites project, add/move a plain docs page, or migrate an existing docs project in. For advanced Markdown/content-block syntax, use bx-sites-content-blocks/bx-sites-markdown; for bxsites.yaml keys, themes, or deployment, use bx-sites-configuration/bx-sites-themes/bx-sites-deployment; for the blog/versions/i18n, use bx-sites-blog-versioning-i18n.
---

# BxSites Getting Started

BxSites (`ortus-boxlang/bx-sites`) is a BoxLang-based static site generator in
the spirit of mkdocs/GitBook: point it at a `docs/` (or `src/`) folder of
Markdown and it builds a themed, searchable static site - not just for docs,
also blogs/marketing sites/knowledge bases.

Every command runs as `bxSites <verb> [options]` (or
`boxlang bxSites <verb> [options]` where the `PATH` shim isn't installed -
e.g. a CI runner). Every verb accepts `--projectRoot=<path>`. **CLI flags
always use `--flag=value`, never a bare positional value**, for a verb's
primary argument.

## Prerequisites

BxSites needs the BoxLang runtime plus `bx-markdown`, `bx-esapi`, `bx-yaml`,
`bx-image` (all installed automatically as `box.json` dependencies):

```bash
# Install BoxLang itself first, if not already present
curl -fsSL https://install.boxlang.io/ | bash
# or BVM (side-by-side version management):
curl -fsSL https://install-bvm.boxlang.io/ | bash && bvm install latest && bvm use latest

# Then install bx-sites
install-bx-module bx-sites   # OS binary installer
# or
box install bx-sites          # CommandBox
```

Either installer drops a `bxSites` script on `PATH`.

## Scaffold a new project

```bash
bxSites new my-docs [--name="My Project Docs"] [--theme=material] [--description=...] [--format=yaml|json]
cd my-docs
```

Creates:

```text
my-docs/
├── docs/
│   ├── assets/
│   └── index.md
└── bxsites.yaml
```

- `--theme` defaults to `bootstrap` - see the `bx-sites-themes` skill for
  all 10 built-in options.
- `--format` defaults to `yaml` (scaffolds `bxsites.yaml`, the
  default/preferred format); `json` scaffolds `bxsites.json` instead. Both
  are fully supported and equivalent - see the `bx-sites-configuration`
  skill for the full key reference.

## Bringing an existing project in

```bash
bxSites migrate --source=<path> [--from=gitbook|mkdocs|markdown-zip|notion]
```

Converts an existing project into `docs/` + `nav.json`:

- `--from=gitbook` (default) - needs a `SUMMARY.md`-rooted GitBook export.
  `{% block %}` syntax becomes its bx-sites equivalent (`::: name`
  directives - see `bx-sites-content-blocks`).
- `--from=mkdocs` - needs `mkdocs.yml`; most syntax carries over unchanged
  since mkdocs-material's own admonition/tabs/math conventions already
  *are* bx-sites' native syntax.
- `--from=markdown-zip` - a plain `.zip` of Markdown files, no proprietary
  format to translate.
- `--from=notion` - a Notion "Export as Markdown & CSV" archive (`.zip` or
  an extracted folder); strips Notion's id-suffixed filenames and turns
  the duplicated leading `# Heading` into real `title` frontmatter.

All four print a conversion summary and a list of anything that needs a
manual look - nothing is silently dropped, but an existing
`bxsites.yaml`/`docs/nav.json` at the destination is overwritten, so review
before committing.

## Project layout

```text
my-docs/
├── bxsites.yaml            # site config (bxsites.json also supported)
├── deployments/*.json      # deploy targets (see bx-sites-deployment)
├── theme/                  # optional project theme override (see bx-sites-themes)
└── docs/                   # or src/ - every .md file here is a page
    ├── index.md
    ├── functions.bxs       # magic functions, see bx-sites-variables-functions
    ├── nav.json            # optional explicit nav (alternative to bxsites.yaml's nav key)
    ├── 404.md               # optional custom 404 page
    ├── robots.txt           # optional hand-authored robots.txt (overrides generated one)
    ├── assets/              # images, downloads, icons - copied to site/assets/
    ├── includes/            # reusable content fragments, spliced via ::: include
    ├── blog/                # see bx-sites-blog-versioning-i18n
    ├── versions/            # see bx-sites-blog-versioning-i18n
    └── i18n/                # see bx-sites-blog-versioning-i18n
```

Folder nesting under `docs/` becomes nav nesting automatically (override
with an explicit `nav` - see `bx-sites-configuration`). `assets/`, `blog/`,
`versions/`, and `i18n/` are reserved folder names with special meaning -
don't repurpose them for ordinary content. A project that isn't really
"docs" in spirit (a marketing site, a portfolio) can use `src/` instead of
`docs/` with zero other changes - every verb looks for `docs/` first and
falls back to `src/`.

## Adding and scaffolding pages

```bash
bxSites page:new --path=guides/setup.md [--title=] [--description=] [--icon=] [--tags=] [--order=]
```

Scaffolds a single page at an arbitrary `docs/`-relative path with
frontmatter pre-filled. Every `.md` file under `docs/` becomes a page.

```bash
bxSites page:rename --from=guides/old.md --to=guides/new.md
```

Moves a page, rewrites every relative Markdown link across `docs/**` that
pointed at it, and stamps `redirect_from` on the moved page automatically
(see `bx-sites-blog-versioning-i18n` for redirects). Always prefer this
over hand-moving a file.

### Page frontmatter

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
- `icon` - emoji or a named icon (`rocket`, `lucide:rocket`, `phosphor-bold:rocket`, `tabler:rocket`, `custom:my-icon` from `docs/assets/icons/my-icon.svg`) - see `bx-sites-markdown`
- `summary` - a one-line lead-in rendered under the title (distinct from `description`, which is meta-only)
- `ogImage` - per-page social card image, overrides the site-wide one
- `toc: false` - hides the page's own "On this page" TOC even with 2+ headings
- `redirect_from` - array of old pretty-URL segments that should redirect here; `page:rename` stamps this automatically

It's a small hand-rolled parser, not full YAML: inline lists (`tags: [a, b]`),
block lists (`- item`), and `>`/`|` block scalars work; nested
objects/maps do not.

## Linking between pages

Link with a file-relative path to the target's `.md` source, exactly as if
browsing the repo on disk - `[Deployment](guides/deployment.md)`,
`[back](../getting-started.md#add-pages)`. BxSites rewrites it to the built
pretty-URL at build time, resolved against the *linking* page's own folder.
Absolute URLs, `mailto:`, and links already starting with `/` are left alone.

## Downloading a page as Markdown

Every built page also gets its own original `.md` source published
alongside it (`docs/guides/deployment.md` → `site/guides/deployment.md`,
next to `site/guides/deployment/index.html`) - a "Download Markdown" link
appears on the page itself, next to "Edit this page." Always on, no config.

## Build, serve, clean

```bash
bxSites build           # render docs/**.md into site/
bxSites serve           # build and serve locally with live reload
bxSites clean           # remove site/ and build cache
```

`serve` builds, serves `site/` at `http://127.0.0.1:8080/` by default
(`--port=`/`--host=` to change), and rebuilds automatically on any saved
change under `docs/`, the site config, or a project `theme/` override - a
native file watcher, only reconverting what changed, so it stays fast even
on a large site. See `bx-sites-build` for `doctor`/`stats`/`check` and a
CI-grade pre-publish gate, and `bx-sites-deployment` for shipping the
result.
