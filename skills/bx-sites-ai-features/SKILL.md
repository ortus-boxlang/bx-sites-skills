---
name: bx-sites-ai-features
metadata:
  version: "1.0"
description: Turn on bx-sites' (ortus-boxlang/bx-sites) AI-facing features - bxsites.yaml's mcp:true (writes a full-text, untruncated site/mcp-index.json + site/mcp-manifest.json + per-tree site/mcp-nav.json on every build, for bxSites Cloud's read-only MCP server) and installing this project's own AI agent skill pack (npx skills add, coldbox ai skills install, or bxSites skills:install). Use this whenever a user wants their published site's content reachable by an MCP-aware AI agent, or wants to install/update the bx-sites-skills pack itself in a project.
---

# BxSites AI Features

Two independent, AI-facing features: exposing a *published site's own
content* to AI agents (`mcp`), and installing *this skill pack* so an AI
coding assistant already knows bx-sites' own conventions.

## MCP server (`mcp: true`)

```yaml title="bxsites.yaml"
mcp: true
```

Every `build` then writes three files alongside the rendered pages, which
[bxSites Cloud](https://bxsites.io/cloud) reads to expose a public,
read-only [MCP](https://modelcontextprotocol.io/) server for the *published*
site (the same idea as GitBook's "MCP servers for published docs") - so
Claude, ChatGPT, or any other MCP-aware client can search/browse/retrieve
content directly instead of scraping rendered HTML. Building the files is
free with `bxSites build` alone; actually serving them over the network as
a live MCP server needs a paid bxSites Cloud plan. `false` (default) skips
the whole step - no extra cost, nothing written.

**Independent of `search`/`searchProvider`** - unrelated switches. `mcp`
never touches the visitor-facing search box, and turning `search` off never
disables `mcp-index.json`. This is also why it can't reuse
`search-index.json`: that file only exists for the `"local"` provider and is
deliberately truncated (400 chars/entry, shipped to every visitor's
browser); `mcp-index.json` is fetched server-side by bxSites Cloud and keeps
the **complete**, untruncated page body.

### The three files

- **`site/mcp-index.json`** - one entry per non-hidden page/post: `title`,
  `url`, `tags`, `headings` (every `h1`-`h6`, in order), `body` (complete
  plain text, HTML stripped, never truncated), `type` (`"page"`/`"post"`),
  `categories` (post only, `[]` for a page), `updatedAt` (ISO-8601, or `""`
  when unavailable).
- **`site/mcp-manifest.json`** - one per build (site root only): every tree
  that got its own index/nav (`path`, `label`, `version`, `locale`,
  `default`), so bxSites Cloud's MCP server can offer version/locale-aware
  tools without guessing bx-sites' own directory conventions.
- **`site/mcp-nav.json`** - one per tree (main, `/next/`, each
  `/versions/<name>/`, each locale sub-tree): that tree's own nav, the exact
  same nested `{ title, url, order, icon, children }` shape the theme's
  sidebar renders from. A group heading with no `index.md` of its own has an
  empty `url`.

Written once per rendered tree (`mcp-index.json`/`mcp-nav.json`) or once per
build (`mcp-manifest.json`) - same "once per tree vs. once per build" split
`search-index.json` and the version/locale system already use (see
`bx-sites-blog-versioning-i18n`).

## Installing this skill pack in a project

[`ortus-boxlang/bx-sites-skills`](https://github.com/ortus-boxlang/bx-sites-skills)
(this repository) is the official skill pack - every skill is a single
self-contained `SKILL.md`, so any install path below installs it correctly.
Works with any assistant supporting the Agent Skills format (Claude Code,
Cursor, Codex, and others).

### `npx skills add` (any project, needs only Node.js)

```bash
npx skills add ortus-boxlang/bx-sites-skills           # whole pack
npx -y skills add ortus-boxlang/bx-sites-skills -y     # non-interactive (CI/scripts)
npx skills add ortus-boxlang/bx-sites-skills/skills/bx-sites-deployment   # one skill
```

### `coldbox ai skills install` (ColdBox CLI)

```bash
coldbox ai skills install ortus-boxlang/bx-sites-skills/bx-sites-deployment
```

See the [BoxLang Skills Directory](https://skills.boxlang.io/) for the
catalog this pulls from.

### `bxSites skills:install` (bx-sites' own wrapper)

A thin wrapper over `npx skills add` that installs straight into the
current project - handy right after scaffolding, so the assistant knows
bx-sites from the very first prompt:

```bash
bxSites skills install
bxSites skills:install --skill=bx-sites-deployment   # one skill
```

Requires Node.js/`npx` on `PATH`, the same requirement `npx skills add` has
on its own.

### Verifying an install

Look for a new `SKILL.md` under the assistant's own skills directory (e.g.
`.claude/skills/bx-sites-getting-started/SKILL.md` for Claude Code), or just
ask the assistant something a skill covers and check the answer matches
this pack's own documentation.
