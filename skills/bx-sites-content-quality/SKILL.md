---
name: bx-sites-content-quality
metadata:
  version: "1.0"
description: Check content quality in a bx-sites (ortus-boxlang/bx-sites) project without a full build - lint (heading-level skips, invalid blog post dates), list/filter draft and published blog posts (blog:drafts, blog:find), and sanity-check the search index (search:query). Use this whenever a user wants to validate raw docs/ Markdown, find draft or specific blog posts, or test what a search query would surface. For build-time checks on the rendered site/ (broken links, missing alt text, stats, environment health), use bx-sites-build instead.
---

# BxSites Content-Quality Checks

Author-facing checks over raw `docs/` Markdown source, run before (or
instead of) a full build. Every command runs as `bxSites <verb> [options]`
(or `boxlang bxSites <verb> [options]`) and accepts `--projectRoot=<path>`.

## `lint`

Pre-build content quality pass over raw `docs/` Markdown - distinct from
`check` (see `bx-sites-build`), which only inspects an already-built `site/`.

```bash
bxSites lint
```

Checks for:

- **Heading level skips** - a page body jumping straight from `##` to
  `####` with no `###` in between (confusing structure, bad for
  accessibility). Lines inside a fenced code block are never mistaken for
  headings.
- **Blog post date issues** - a `docs/blog/posts/**` post with a missing or
  invalid frontmatter `date` (a real `build` itself throws on this the
  moment it loads posts - `lint` surfaces it as a finding first instead).

Exits `1` when either check finds anything, `0` otherwise.

## `blog:drafts`

Lists every blog post whose frontmatter sets `draft: true` - a real `build`
always skips drafts, so this is the only place their existence is surfaced.

```bash
bxSites blog:drafts
```

Always exits `0`.

## `blog:find`

Filters blog posts by author/category/tag/date range, without running a
full build.

```bash
bxSites blog:find [--author=] [--category=] [--tag=] [--since=] [--until=] [--drafts]
```

- `--author`, `--category`, `--tag` - case-insensitive exact match against
  any of the post's own values
- `--since`, `--until` - a date; only posts on/after `--since` and/or
  on/before `--until` match
- `--drafts` - include draft posts too (excluded by default)

Every filter is optional and independent - passing none lists every
published post. See `bx-sites-blog-versioning-i18n` for post frontmatter.

## `search:query`

Runs a keyword query against an already-built `site/search-index.json` -
run `bxSites build` or `bxSites search-index` first (see `bx-sites-build`).

```bash
bxSites search:query --query="getting started" [--limit=10]
```

- `--query` (required) - space-separated search terms
- `--limit` - maximum results to return, defaults to `10`

Ranks results using the same relative field weighting the client-side
search widget uses (title, then tags, then headings, then body), so a
project can sanity-check what a real visitor's search would surface without
opening a browser. See `bx-sites-search` for the underlying providers -
this verb only ever reads the main `docs/` tree's local index, not a
version/locale tree's own, and is a no-op with the `algolia`/`pagefind`
providers (neither uses a local `search-index.json`).

## A typical pre-publish content pass

```bash
bxSites lint
bxSites blog:drafts        # confirm nothing meant to be published is still draft
bxSites build
bxSites check               # broken links/alt text - see bx-sites-build
```
