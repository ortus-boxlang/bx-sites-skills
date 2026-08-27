---
name: bx-sites-build
metadata:
  version: "1.0"
description: Build, serve, and diagnose a bx-sites (ortus-boxlang/bx-sites) site - build/serve/clean/search-index, doctor (environment/config health), stats (page/word/blog/search-index/output-size report), and check (broken internal links/images, missing alt text, orphaned pages) against an already-built site/. Use this whenever a user wants to run/troubleshoot a bx-sites build, verify a build actually produced valid output, or get a CI-grade quality gate before deploying. For raw docs/ Markdown checks that don't need a build, use bx-sites-content-quality; for shipping site/ somewhere, use bx-sites-deployment.
---

# BxSites Build & Diagnostics

Every command runs as `bxSites <verb> [options]` (or
`boxlang bxSites <verb> [options]` where the `PATH` shim isn't installed -
e.g. a CI runner). Every verb accepts `--projectRoot=<path>`.

## `build`

```bash
bxSites build
```

Renders `docs/**.md` (or `src/`) into a static site in `site/` - also
builds the search index (unless `search: false`, or `searchProvider` is set
to a provider like `algolia`/`pagefind` that doesn't use it - see
`bx-sites-search`), runs the `pagefind` CLI against the finished `site/`
when that provider is active, and copies theme + `docs/assets/**` into
`site/`.

## `serve`

```bash
bxSites serve [--port=8080] [--host=127.0.0.1]
```

Builds and serves `site/` locally with live reload. Runs in the foreground
until interrupted (Ctrl+C). A native BoxLang file watcher (not a poll loop)
reacts to a saved change immediately, and only reconverts the page(s) that
actually changed rather than the whole site.

## `search-index`

```bash
bxSites search-index
```

Rebuilds `site/search-index.json` standalone, without re-rendering pages or
copying assets - `build` already runs this automatically; this verb exists
for when only the index needs refreshing. Only ever covers the main `docs/`
tree, even on a project with `docs/versions/`/`docs/i18n/` - a real `build`
writes each tree's own scoped index instead. No-op for the `algolia`/
`pagefind` search providers (neither uses a local index) - see
`bx-sites-search`.

## `clean`

```bash
bxSites clean
```

Removes `site/` and any build cache, leaving `docs/` and the site config
alone.

## `doctor`

```bash
bxSites doctor
```

A one-shot environment/config health check - the "run this before filing a
bug report" verb. Checks the JVM version, that `docs/` exists, that
`bxsites.yaml`/`.json` actually parses and validates, that the required
BoxLang modules (`bx-markdown`, `bx-esapi`, `bx-yaml`, `bx-image`) are
installed and activated, and - if a project-level `theme/` override exists
- that it satisfies the `layout.bxm`/`page.bxm` contract (see
`bx-sites-themes`). Exits `1` if any check fails, `0` otherwise. Nothing
here mutates a project.

## `stats`

```bash
bxSites build
bxSites stats
```

A read-only summary report of an already-built `site/` (run `build` first).
Reports:

- Total page/word counts, plus a per-tree breakdown once there's more than
  one tree (a version, or a non-default locale)
- Version/locale names present
- Blog post/category/author/year-active counts (`none` if no blog)
- Distinct tag count across the whole site
- Search-index entry count and file size (`none` if search is off or a
  non-local provider is active)
- Total file count and on-disk size of the built `site/`

Always exits `0` - purely informational, not a pass/fail gate (that's
`check`'s job).

## `check`

```bash
bxSites build
bxSites check
```

A CI-grade content quality gate over an already-built `site/` (run `build`
first). Checks for:

- **Broken internal links/images** - any `<a href>`/`<img src>` pointing at
  a page or asset that doesn't exist in `site/`. Fails the check.
- **Missing alt text** - any `<img>` with no `alt` attribute at all. An
  empty `alt=""` (correct markup for a purely decorative image) is not
  flagged. Fails the check.
- **Orphaned pages** - pages in `site/` not reachable by following links
  from any tree's own homepage. Informational only, never fails the check -
  a page deliberately left out of nav (frontmatter `hidden: true`) is
  *supposed* to only be reachable by direct link.

Exits `1` when there are broken links/images or missing-alt images, `0`
otherwise (orphaned pages never affect the exit code). Deliberately
internal-links-only - no HTTP requests to check external URLs.

## A CI-grade pre-publish gate

```bash
bxSites build
bxSites check
bxSites stats
```

`check` catches broken content; `stats` gives a sanity-check summary before
handing off to `bx-sites-deployment`. See `bx-sites-content-quality` for
`lint`/`blog:drafts`/`blog:find`/`search:query`, which check raw `docs/`
source without needing a build first.

## A build can report success while producing broken output

A crashed BoxLang process's exit code doesn't always propagate reliably
through every calling shell script. When a fix doesn't appear to take
effect after a "successful" build, don't trust a green exit code alone -
confirm `site/` was actually (re)written with the expected content, and run
`doctor`/`check` to catch a config or content problem the build step itself
didn't surface.
