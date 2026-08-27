---
name: bx-sites-blog-versioning-i18n
metadata:
  version: "1.0"
description: Set up and write a blog, versioned docs, and translated (i18n) locales in bx-sites (ortus-boxlang/bx-sites) - docs/blog/posts frontmatter and authors.yml, post:new, categories/archives/RSS, docs/versions/<name> and version:new, docs/i18n/<code> and i18n:new, composing versions with locales, theme-chrome translation strings, and redirects (frontmatter redirect_from and bxsites.yaml's redirects). Use this whenever a user wants to add a blog post, cut a new docs version, add a translated locale, or keep an old URL working after a page moves.
---

# BxSites Blog, Versioning, i18n & Redirects

All three are **convention over configuration** - no `bxsites.yaml` key
turns them on, just a folder.

## Blog

Drop posts under `docs/blog/posts/` and BxSites builds `/blog/` (paginated),
a category page per category, a year-archive page per calendar year, an
author page per author, RSS feeds, and `/blog/stats/` - zero config. A
project with no `docs/blog/posts/` folder simply has no blog.

### Scaffolding a post

```bash
bxSites post:new --title="My New Post" [--slug=] [--date=] [--authors=] [--categories=] [--tags=] [--draft]
```

`--draft` defaults `true`; pass `--!draft` to publish immediately.

### Writing a post

Every `.md` under `docs/blog/posts/`, any depth, is a post. Subfolders are
purely for editing convenience - a post's URL/sort/archive are all derived
from frontmatter, never from where the file lives:

```markdown title="docs/blog/posts/announcing-2-0.md"
---
title: Announcing BoxLang 2.0
date: 2026-08-15
authors: [lmajano]
categories: [Releases]
tags: [boxlang, release]
summary: A faster runtime, a smaller footprint, and a few surprises.
image: assets/blog/boxlang-2-cover.png
---

A short intro paragraph or two.

<!-- more -->

The rest of the post - left out of the excerpt on /blog/ and category
pages, still renders in full on the post's own page.
```

- `date` (required) - sets sort order (newest first) and `<pubDate>`
- `authors` - ids matching `docs/blog/authors.yml`, or a plain unlinked name
- `categories` - taxonomy, each gets its own `/blog/category/<slug>/` page +
  feed. Unrelated to `tags`.
- `tags` - the usual site-wide tags (badges, feeds `/tags/`)
- `summary` - one-line excerpt for lists/RSS (falls back to a plain-text
  truncation if omitted and no `<!-- more -->`)
- `image` - featured image (`docs/assets/`-relative or full URL); also
  becomes `og:image`/Twitter card unless `ogImage` overrides it; gets the
  same responsive `<picture>`/WebP treatment as any image (see
  `bx-sites-markdown`)
- `slug` - overrides `/blog/<slug>/` (defaults from filename)
- `draft: true` - excluded from a real `build` entirely; `serve` previews it
  with a visible "🚧 Draft" banner so you can proofread locally
- Every ordinary page frontmatter key (`icon`, `description`, `ogImage`,
  `toc`) also works on a post - see `bx-sites-getting-started`

`docs/assets/blog/` is just a conventional subfolder for post
covers/author photos - not enforced, any `docs/assets/**` path works.

### Authors

`docs/blog/authors.yml`, optional, one entry per id, referenced by a post's
`authors` list:

```yaml title="docs/blog/authors.yml"
lmajano:
  name: Luis Majano
  title: CEO, Ortus Solutions
  bio: >
    Founder of Ortus Solutions and creator of ColdBox, WireBox, and BoxLang.
  url: https://github.com/lmajano
  email: lmajano@ortussolutions.com
  socials:
    github: https://github.com/lmajano
    twitter: https://x.com/lmajano
```

Only `name` is required. An author gets their own `/blog/authors/<id>/`
page only once at least one post credits them. **Avatar by convention** -
`docs/assets/blog/authors/<id>.{jpg,jpeg,png,webp,svg}` is picked up
automatically; an explicit `avatar` key always overrides it.

### Nav entry

A "Blog" nav entry is added automatically once there's at least one
non-draft post (appended last by default). To place it explicitly (and
suppress the auto entry), add your own `nav`/`docs/nav.json` entry with a
bare `url` (bypasses the usual `path`-must-be-a-real-page rule):

```yaml title="bxsites.yaml"
nav:
  - path: index.md
  - title: Blog
    url: blog/index.html
    icon: lucide:newspaper
  - path: about.md
```

### Feed

`/blog/feed.xml` (RSS 2.0) - written when `baseURL` is a full URL (same
requirement as `sitemap.xml`) and `blog.feed` isn't `false`. Each category
also gets `/blog/category/<slug>/feed.xml`. Both capped to `blog.feedLimit`
(default `25`; `0` = uncapped). Config lives in `bxsites.yaml`'s `blog` key
- see `bx-sites-configuration`.

### Restyling the blog

Blog pages render through the exact same `layout.bxm`/`page.bxm` as every
other page - no separate blog theme (see `bx-sites-themes`). The generated
markup (post cards, meta line, pager, author profile, browse-by-year/
category links) uses fixed CSS classes (`blog-post-card`, `blog-post-meta`,
`blog-post-featured-image`, `blog-draft-badge`, `blog-pager`,
`blog-author-profile`, `blog-archive-links`/`blog-category-links`) -
restyle them via `extraCss`, or change structure via a theme override. The
post-card/pager/author-profile markup itself is generated by `BlogBuilder.bx`,
not read from a theme template - it can be restyled with CSS but not
replaced with a custom template.

## Versioning

Add a `docs/versions/` folder; each direct subfolder inside is built as its
own fully self-contained doc tree alongside the regular `docs/` (always
built as "Latest"):

```text
docs/
├── index.md
├── guides/
└── versions/
    ├── 1.0/
    │   ├── index.md
    │   └── guides/
    └── 2.0/
```

Each version folder is a normal `docs/`-shaped tree - own `index.md`, own
nav, own pages - built to `site/versions/<name>/`, sharing the project's
single `bxsites.yaml` config/theme.

**Cutting a version**:

```bash
bxSites version:new --name=1.0
```

Snapshots the *current* `docs/` tree (excludes `assets/`, `versions/`,
`i18n/`, `blog/` - each is its own separately-loaded tree). Typical
workflow: finish docs for a release, cut the version right before starting
the next release's docs. There's no "un-cut" verb; editing an already-cut
version's pages is just editing the file directly - `page:new`/
`page:rename`/`post:new` always target the main `docs/` tree only.

Version names sort **newest-first, numerically** (`2.0` before `10.0`);
every theme renders a version-switcher automatically once more than one
version exists. `sitemap.xml`/`llms.txt` include every version.

**Out of scope**: search is scoped per-tree (separate `search-index.json`
per version) except with the `pagefind` provider, which indexes the whole
built site (see `bx-sites-search`); no deprecated/EOL flag or custom label -
a version's switcher label is always its folder name.

## i18n

Translated content lives in `docs/i18n/<code>/`, mirroring `docs/`
page-for-page:

```text
docs/
├── index.md
├── guides/setup.md
└── i18n/
    ├── es/
    │   ├── index.md
    │   └── guides/setup.md
    └── ar/
        └── index.md
```

`<code>` is both the folder name and the built URL prefix
(`/es/guides/setup/`) - letters/digits/hyphens only (`es`, `pt-BR`,
`zh-Hans`). The regular `docs/` tree is always the default locale, built
unprefixed at the root.

**Scaffolding a locale**:

```bash
bxSites i18n:new --code=es
```

Seeds `index.md` from the default locale's own. Then register the
label/direction in `bxsites.yaml`:

```yaml title="bxsites.yaml"
i18n:
  defaultLocale: { code: en, label: English }
  locales:
    - { code: es, label: Español }
    - { code: ar, label: العربية, dir: rtl }
    - { code: pt-BR, label: "Português (Brasil)", flag: 🇧🇷 }
```

`defaultLocale` only needs setting if the default isn't English. A folder
with no matching `locales` entry still builds (using its bare code as
label) - `locales` supplies display metadata, not the on/off switch.

`bxSites i18n:status` reports per-locale translation coverage (which pages
exist at the same relative path for each locale) - always exits `0`,
purely informational.

**Untranslated pages** - a page missing from `docs/i18n/es/` still builds at
its expected URL, showing the default locale's content with a "not
translated yet" notice. Nothing 404s. Every locale's nav is always the exact
same shape as the default's.

**Language switcher** - appears automatically once more than one locale
exists, no opt-in.

**Composing with versions**: `docs/versions/<name>/i18n/<code>/` mirrors
that version's own structure. Switching version always drops to that
version's own default locale; switching locale always stays on the current
version.

**Theme chrome (UI strings)** - the surrounding UI text (search placeholder,
"On this page," 404 page, etc.) translates per locale too. `de`/`es`/`it`/
`ja` ship a built-in translation; any other code falls back to English.
Override specific keys per locale:

```yaml title="bxsites.yaml"
i18n:
  locales:
    - code: fr
      label: Français
      strings:
        searchPlaceholder: Rechercher dans la documentation...
        onThisPage: Sur cette page
```

Overridable keys: `searchPlaceholder`, `searchAriaLabel`, `searchNoResults`,
`toggleDarkMode`, `toggleNavigation`, `toggleSection` (`{title}`),
`repository`, `version`, `language` (`{label}`), `onThisPage`,
`editThisPage`, `downloadMarkdown`, `lastUpdated`, `notTranslatedNotice`
(`{locale}`), `notFoundTitle`, `notFoundBody`, `tagsTitle`, `pageNavigation`.

**Out of scope**: blog chrome stays in English; `lastUpdated`'s date value
isn't locale-formatted; RTL is baseline (mirrors layout, a few decorative
details like an admonition's accent-bar side don't flip); no machine
translation - every locale file is hand-authored. `custom:` icons and
`::: include` both resolve against the project's shared `docs/assets/`
regardless of locale.

## Redirects

Keep an old URL working after a page moves/renames - a static HTML stub
redirects a stale bookmark/search result to the new URL.

**Per-page** - frontmatter `redirect_from` on the page that replaced the old
URL:

```markdown title="docs/guides/new-setup.md"
---
title: New Setup
redirect_from:
  - guides/old-setup
  - setup
---
```

Each entry is a pretty-URL segment (no leading/trailing slash, no
extension). Scoped to whichever tree the page belongs to (a version's page
redirects within that version, a locale's within that locale).

**Site-wide** - `bxsites.yaml`'s `redirects` array, for an old URL that
never belonged to one specific page:

```yaml title="bxsites.yaml"
redirects:
  - from: old-guide
    to: guides/new-guide/
  - from: moved-to-another-site
    to: https://example.com/docs
```

`to` is a root-relative path (resolved against `baseURL`) or a full
`https://` URL. Only applies to the main site tree.

`bxSites page:rename --from=... --to=...` (see `bx-sites-getting-started`)
stamps `redirect_from` automatically on top of rewriting every relative
Markdown link that pointed at the old path - always prefer it over
hand-moving a file.

**Conflicts fail the build** rather than silently overwriting: a redirect's
`from` colliding with a real page, or two redirects targeting the same
`from`.

**Out of scope**: blog posts don't get `redirect_from` (use a `redirects`
config entry instead); no wildcard/pattern redirects - every `from` is one
exact path.
