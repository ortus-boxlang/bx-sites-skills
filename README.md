# bx-sites-skills

A collection of skills for AI coding agents following the Agent Skills format, focused on `ortus-boxlang/bx-sites` - the BoxLang static site generator. Covers creating and authoring content, configuring/theming/publishing sites, and operating the project's own CI.

## Install

```bash
npx skills add ortus-boxlang/bx-sites-skills
```

For non-interactive installs:

```bash
npx -y skills add ortus-boxlang/bx-sites-skills -y
```

Install a single skill instead of the whole set with
`npx skills add ortus-boxlang/bx-sites-skills/skills/<skill-name>`.

## Available Skills

Each skill is a single, self-contained `SKILL.md` (no bundled resource
files) so it installs correctly via `npx skills add` and the ColdBox/BoxLang
`skills.boxlang.io` catalog alike.

| Skill | Description |
|---|---|
| `bx-sites-getting-started` | Install bx-sites, scaffold a new project (or migrate an existing GitBook/mkdocs/Notion one in), project layout, page frontmatter, linking, build/serve/clean. |
| `bx-sites-content-blocks` | GitBook-style `::: name :::` blocks - cards, columns, stepper, buttons, embeds, page-link/link-preview, prompt, updates, includes, conditional content, OpenAPI widget. |
| `bx-sites-markdown` | Admonitions, footnotes, definition lists, content tabs, code-block annotations, Mermaid, math, tables, icons, responsive images, Alpine.js interactivity. |
| `bx-sites-variables-functions` | Reusable `{{ variables }}` and BoxLang magic functions (`docs/functions.bxs`), including status-badge/rating/progress-bar visualizer recipes. |
| `bx-sites-blog-versioning-i18n` | The blog (`docs/blog/posts/`), versioned docs (`docs/versions/`), translated locales (`docs/i18n/`), and redirects. |
| `bx-sites-content-quality` | Pre-build content checks without a full build - `lint`, `blog:drafts`, `blog:find`, `search:query`. |
| `bx-sites-build` | `build`/`serve`/`clean`/`search-index`, plus `doctor`/`stats`/`audit` diagnostics on a built site. |
| `bx-sites-configuration` | The full `bxsites.yaml`/`bxsites.json` key reference - `baseURL`, `nav`, `redirects`, `markdown`, assets, and more. |
| `bx-sites-themes` | Choosing, customizing, overriding, installing, or writing a theme; the `ThemeProvider` contract. |
| `bx-sites-search` | Search providers - local (MiniSearch), Algolia DocSearch, Pagefind, and wiring up a custom provider. |
| `bx-sites-plugins` | Writing/installing a BxSites plugin (build-lifecycle hooks) or a CLI provider (new `bxSites` verbs). |
| `bx-sites-deployment` | Deploy targets (S3/Azure/GCS/Firebase/FTP/SFTP/rsync/Netlify/Vercel/Cloudflare Pages/GitHub Pages), `package`, and the GitHub Actions publishing workflow. |
| `bx-sites-actions` | Operate and troubleshoot bx-sites' own GitHub Actions workflows (tests, snapshot, release, docs, pages). |

## Source project

- bx-sites repository: https://github.com/ortus-boxlang/bx-sites
- bx-sites actions: https://github.com/ortus-boxlang/bx-sites/actions
