---
name: bx-sites-publishing
metadata:
  version: "1.0"
description: Configure, build, theme, extend, and publish a bx-sites (ortus-boxlang/bx-sites) project - bxsites.yaml/bxsites.json site configuration, choosing/overriding/writing/importing themes, search providers (local/Algolia/Pagefind), plugins (writing hooks, installing from ForgeBox), building/serving locally, and deploying to S3/Azure/GCS/Firebase/FTP/SFTP/rsync/Netlify/Vercel/Cloudflare Pages/local/GitHub Pages, packaging a zip, or migrating an existing GitBook/mkdocs/Notion project in. Use this whenever a user wants to set up, theme, configure, extend, build, or ship a bx-sites site - as opposed to writing its Markdown content (see the bx-sites-content skill for that).
---

# BxSites Configuration, Theming & Publishing

BxSites (`ortus-boxlang/bx-sites`) is a BoxLang-based static site generator.
This skill covers **site-level operations**: configuration, themes, search
providers, plugins, building, and deployment. For writing/structuring
Markdown content itself, use the `bx-sites-content` skill instead.

Every command runs as `bxSites <verb> [options]` (or
`boxlang bxSites <verb> [options]` where the `PATH` shim isn't installed -
e.g. a CI runner, a module registered by hand). Every verb accepts
`--projectRoot=<path>`. **CLI flags always use `--flag=value`, never a bare
positional value**, for a verb's primary argument.

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

## Build & serve

| Verb | Purpose |
|---|---|
| `build` | Render `docs/**.md` (or `src/`) into `site/` - search index, `sitemap.xml`, `robots.txt`, `llms.txt`, theme + `docs/assets/**` |
| `serve [--port=8080] [--host=127.0.0.1]` | Build and serve locally with live reload (native file watcher, only reconverts what changed); foreground until Ctrl+C |
| `search-index` | Rebuild `site/search-index.json` standalone (build already runs this) |
| `clean` | Remove `site/` and build cache, leaving `docs/`/config alone |
| `doctor` | Environment/config health check (JVM, `docs/` exists, config parses, required modules installed, theme override valid) - exits `1` on failure |
| `stats` | Read-only report on a built `site/`: pages/words, versions/locales, blog, tags, search-index size, output size |
| `check` | CI-grade quality gate on a built `site/`: broken internal links/images (fails), missing `alt` text (fails), orphaned pages (informational) |

Typical local iteration loop: `bxSites serve`. Typical CI/pre-publish gate:

```bash
bxSites build
bxSites check
bxSites stats
```

## Configuration (`bxsites.yaml` / `bxsites.json`)

One config file at the project root - `bxsites.yaml`/`.yml` is the default
and preferred format (what `new` scaffolds unless `--format=json`);
`bxsites.json` is fully supported. Both produce identical results and every
key is named/shaped the same in either. Only `name` is required. See
[reference/configuration.md](reference/configuration.md) for the complete
key-by-key reference (`baseURL`, `nav`, `redirects`, `markdown`, `repo`,
`social`, `footer`, `lastUpdated`, `analytics`, `ogImage`/`generateOgImages`,
`extraCss`/`extraJs`, `assets`, `plugins`, `i18n`, `blog`, `variables`,
`pageActions`, `search`/`searchProvider`, `mermaid`, `math`, `openapi`).

**`baseURL`** is the one key with the widest blast radius - it controls
whether links stay root-relative or absolute, whether `sitemap.xml`/RSS
feeds/canonical tags/social-share links generate at all, and any sub-path
prefix (e.g. GitHub Project Pages needs
`baseURL: "https://<user>.github.io/<repo>/"`).

## Themes

Ten built-in themes, set via `theme.name` in the config:
`bootstrap` (default), `material`, `tailwind`, plus seven `material`-forked
gallery themes (`docsy`, `slate`, `docusaurus`, `justthedocs`, `vuepress`,
`gitbook`, `notion`). All air-gapped-capable (vendored CSS/JS, no CDN)
except `tailwind` (its utility engine is a CDN-loaded JIT compiler). Full
palette/customization/writing-from-scratch reference:
[reference/themes.md](reference/themes.md).

| Verb | Purpose |
|---|---|
| `theme:new --theme=material` | Eject a built-in theme into project `theme/` for customizing (fails rather than overwrite an existing `theme/`) |
| `install:theme --name=... [--version=]` | Download a published theme from ForgeBox into `themes/<name>/` |
| `theme:import --source=mkdocs\|jekyll\|hugo --path=<dir> --name=<dest>` | Best-effort convert another SSG's theme into a `themes/<name>/` scaffold |

Resolution order at build time: project `theme/` override → an installed
`themes/<name>/` matching `theme.name` → a built-in theme named `theme.name`.
A theme is just `layout.bxm` + `page.bxm` (required) plus optional
`search.bxm`/`assets/` - see reference/themes.md for the full contract and a
worked "swap the brand palette" example. For a color/font-only tweak, prefer
`extraCss` targeting the theme's CSS custom properties over a full override.

## Search

`search: true` (default) builds a static client-side index (MiniSearch); set
`searchProvider.provider` to `algolia` or `pagefind` to swap providers, or
anything else for a fully custom one wired up via a theme override. Details,
tradeoffs, and the Meilisearch worked example live in
[reference/search.md](reference/search.md).

## Plugins

A plugin is just another BoxLang module - `box install` it, then opt it in
by name in `bxsites.yaml`'s `plugins` array (installing alone never
activates it). Full hook reference (`onConfig`, `onPageMarkdown`,
`onPageHtml`, `onNav`, `onSearchIndex`, `onSitemap`, `onBuildComplete`) and
the CLI-provider mechanism for registering new `bxSites` verbs:
[reference/plugins.md](reference/plugins.md).

| Verb | Purpose |
|---|---|
| `plugin:new --name=my-plugin [--dest=]` | Scaffold a plugin module skeleton |
| `install:plugin --name=... [--version=]` | Download a published plugin from ForgeBox into `boxlang_modules/` |

## Migrating an existing project in

| Verb | Purpose |
|---|---|
| `migrate --source=<path> [--from=gitbook\|mkdocs\|markdown-zip\|notion]` | Convert an existing project into `docs/` + `nav.json` |

`--from=gitbook` (default) needs a `SUMMARY.md`-rooted export;
`--from=mkdocs` needs `mkdocs.yml`; `--from=markdown-zip`/`--from=notion`
accept a `.zip` (Notion also accepts an already-extracted folder). All four
print a conversion summary and a list of anything that needs a manual look -
nothing is silently dropped, but an existing `bxsites.yaml`/`docs/nav.json`
at the destination is overwritten, so review before committing.

## Deployment

`site/` is a plain static site - host it anywhere. Full per-target config
shapes (S3-compatible, Azure, GCS, Firebase, FTP/SFTP, rsync, Netlify,
Vercel, Cloudflare Pages, local, GitHub Pages), the GitHub Actions
multi-version-publishing workflow, and access-restriction options:
[reference/deployment.md](reference/deployment.md).

| Verb | Purpose |
|---|---|
| `deploy --entry=<name> [--verbose]` | Build, then ship via whatever target `deployments/<name>.json` declares |
| `deploy --target=local\|github-pages [flags] [--verbose]` | Flag-only shorthand for the two simplest targets - no `deployments/` file needed |
| `deploy [--verbose]` | No flags - deploy to *every* `deployments/*.json` entry, off one shared build; one failure doesn't stop the rest |
| `gh-deploy [--branch=gh-pages] [--remote=origin] [--message=...]` | Build, force-push to a `gh-pages`-style branch (one commit per deploy); requires a git repo + remote |
| `package [--output=<path>]` | Build, then zip `site/`'s contents into a single archive (defaults to `site.zip`) |

**Secrets always come from an environment variable, never a literal value**
in `deployments/*.json` - every field ending `EnvVar` names the env var
holding the real secret, resolved live at deploy time, so the JSON file
itself is always safe to commit. A field that's a *path* to a credential
file you manage yourself (an SSH key, a GCP service-account JSON) is the one
exception.

```bash
bxSites deploy --entry=production --verbose
```

```json title="deployments/production.json"
{ "target": "s3", "bucket": "my-docs-site", "accessKeyIdEnvVar": "AWS_ACCESS_KEY_ID", "secretAccessKeyEnvVar": "AWS_SECRET_ACCESS_KEY" }
```

## Output expectations

When configuring, theming, or deploying a project, report: the exact
`bxsites.yaml`/`bxsites.json` keys changed (with the reasoning for
non-obvious ones like `baseURL`), which CLI verb was run and its outcome
(build/check/deploy summary line), and - for a new `deployments/*.json` -
confirm no secret literal was written, only an `*EnvVar` reference. Prefer
running `bxSites doctor` before troubleshooting a build/deploy failure, and
`bxSites check`/`stats` after a build to confirm it actually produced valid
output (a crashed build can still report a green exit code in some CI
shells - verify `site/` was actually written).
