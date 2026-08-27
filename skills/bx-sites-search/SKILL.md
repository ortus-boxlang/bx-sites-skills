---
name: bx-sites-search
metadata:
  version: "1.0"
description: Configure and troubleshoot search in a bx-sites (ortus-boxlang/bx-sites) site - the default local static/client-side provider (MiniSearch, search-index.json, Cmd/Ctrl+K palette), Algolia DocSearch, Pagefind, and wiring up a fully custom provider (e.g. Meilisearch) via a theme override. Use this whenever a user wants to turn search on/off, switch providers, or debug why search results look wrong.
---

# BxSites Search Providers Reference

`search: true`/`false` (in `bxsites.yaml` - see `bx-sites-configuration`) is
the master on/off switch regardless of which `searchProvider.provider` is
active.

## `local` (default)

Fully static/client-side - the mkdocs default approach. At `build` time,
`SearchIndexer` writes `site/search-index.json`: one entry per page with
`title`, `url`, frontmatter `tags`, every heading's text, and a truncated
plain-text body copy. In the browser, `assets/search.js` fetches it once and
builds a [MiniSearch](https://lucaong.github.io/minisearch/) index (prefix +
typo-tolerant fuzzy matching, both on by default; title weighted highest,
then tags, then headings, then body). No server/database/external service.

**Shortcuts**: `/` focuses the sidebar box; Cmd/Ctrl+K opens a
command-palette overlay (reuses the same MiniSearch index, `local` only);
`Escape` closes either.

```bash
bxSites search-index
```
rebuilds just the index (`build` already runs this; useful standalone - see
`bx-sites-build`). `bxSites search:query --query="..."` (see
`bx-sites-content-quality`) sanity-checks what a real search would surface.

## `algolia`

```yaml title="bxsites.yaml"
search: true
searchProvider:
  provider: algolia
  algolia:
    appId: ABC123
    apiKey: a1b2c3d4e5f6...   # search-only public key, NEVER an admin key
    indexName: my-docs
    insights: false
```

`appId`/`apiKey`/`indexName` required. With Algolia active: no
`search-index.json` is built, no MiniSearch/`search.js` shipped - results
come from Algolia's own hosted index, populated by DocSearch's crawler or
your own Algolia Crawler config (register separately - bx-sites only wires
up the client widget). Each theme renders an empty
`#bxsites-search-algolia` container; `layout.bxm` loads `@docsearch/css`/
`@docsearch/js` from jsDelivr and calls `docsearch({...})`. Gets Cmd+K for
free from DocSearch itself.

## `pagefind`

```yaml title="bxsites.yaml"
search: true
searchProvider:
  provider: pagefind
  pagefind: { bin: pagefind, options: [] }
```

Both keys optional (`bin` default `"pagefind"`, resolved against `PATH`;
`options` extra raw CLI flags, e.g. `["--exclude-selectors", ".no-index"]`).
**The `pagefind` CLI must already be installed and on `PATH`** - bx-sites
shells out to it, doesn't install it. A missing/failing binary fails the
build loudly (`BxSites.PagefindFailed`) rather than degrading silently.
Right after every doc tree + `sitemap.xml`/`llms.txt` are written, bx-sites
runs `pagefind --site <siteDir> [...options]` against the *entire* built
`site/` (indexes a multi-version/multi-locale site in one pass, unlike
`local`'s per-tree index - see `bx-sites-blog-versioning-i18n` for
versions/locales). Writes into `site/pagefind/` - self-hosted, no CDN. No
`search-index.json`; `bxSites search-index` is a no-op for this provider.

## Choosing a provider

| | `local` | `algolia` | `pagefind` |
|---|---|---|---|
| Server/account | No | Yes (Algolia) | No |
| Indexed from | `search-index.json` | Algolia's hosted index | Built `site/` HTML |
| Fuzzy/prefix | Yes (MiniSearch) | Yes (Algolia) | Yes (Pagefind) |
| Extra install | None | None (client-only) | `pagefind` CLI on `PATH` at build time |
| Best for | Most projects, zero setup | Large sites wanting hosted analytics/tuning | Large multi-version/multi-locale sites, full-page indexing, no hosted account |

## Building a fourth/custom provider

`searchProvider.provider` accepts any string - `bxsites.yaml` only validates
the three built-in providers and freely allows an arbitrary sub-block
alongside it (`searchProvider.meilisearch: {...}`). There's no plugin hook
for the search UI itself; the built-in themes render nothing for an
unrecognized provider name, so wiring one up is a project-level theme
override (see `bx-sites-themes`):

1. **Configure it** - any shape (unvalidated):
   ```yaml
   search: true
   searchProvider:
     provider: meilisearch
     meilisearch: { host: https://my-project.meilisearch.io, apiKey: "...", indexName: my-docs }
   ```
2. **Eject a theme** - `bxSites theme:new --theme=bootstrap` copies the
   built-in theme into project `theme/` (project-theme-wins resolution).
3. **Add the mount point** in `theme/search.bxm` - it already branches on
   `variables.searchProviderName` for `local`/`algolia`/`pagefind`; add a
   branch: `<bx:if variables.searchProviderName eq 'meilisearch'><div
   id="bxsites-search-meilisearch"></div></bx:if>`.
4. **Load the client and wire it up** in `theme/layout.bxm` - the existing
   Algolia block is a `<bx:if variables.searchEnabled and
   variables.searchProviderName eq 'algolia'>` guard; add the equivalent for
   the new provider's own widget, reading its config back out of
   `variables.siteConfig.searchProvider.meilisearch.*`.
5. **Index the built site, if not crawler-hosted** - Algolia's own crawler
   populates out of band; a self-hosted engine like Meilisearch/Pagefind
   needs something to push documents after `build` writes `site/`. Use a
   plugin's `onBuildComplete( siteDir, config )` hook (see
   `bx-sites-plugins`) - `site/search-index.json` is still built even for
   an unrecognized provider (`SearchProviderRegistry.usesLocalIndex()`
   defaults `true`), so it's ready to use as the push payload.
