---
name: bx-sites-deployment
metadata:
  version: "1.0"
description: Deploy or package a built bx-sites (ortus-boxlang/bx-sites) site - deployments/*.json targets (S3-compatible, Azure, GCS, Firebase, FTP/SFTP, rsync, Netlify, Vercel, Cloudflare Pages, local, GitHub Pages), secrets-in-env-vars conventions, bxSites package, the GitHub Actions multi-version-publishing workflow, and restricting who can reach a deployed site. Use this whenever a user wants to ship a built bx-sites site somewhere, set up CI/CD publishing, or gate access to a deployed site.
---

# BxSites Deployment Reference

`site/` is a plain static site - host it anywhere that serves static files.
`bxSites deploy` ships it there directly. Build it first (or let `deploy`
build it for you) - see `bx-sites-build`.

## The `deploy` command - three invocation shapes

```bash
bxSites deploy --entry=production [--verbose]                    # 1. named deployments/<name>.json
bxSites deploy --target=local|github-pages [flags] [--verbose]   # 2. flag-only, no deployments/ file
bxSites deploy [--verbose]                                       # 3. every deployments/*.json entry
```

1. **`--entry=<name>`** dispatches to whatever `deployments/<name>.json`
   declares - every target except `local`/`github-pages` needs this (more
   config than a couple of flags can carry).
2. **`--target=<name>` with its own flags** - shorthand for the two simplest
   targets only, no `deployments/` folder at all: `local` (`--destination=<path>`)
   and `github-pages` (`[--branch] [--remote] [--message]`, same defaults as
   `gh-deploy`).
3. **Neither flag** - every `deployments/*.json` entry deployed in turn, off
   **one shared build** (not rebuilt per target). One target failing doesn't
   stop the rest; the command exits non-zero only if at least one failed,
   and the summary reports the count (`Deployed to 2/3 target(s) (1 failed)`).

`--verbose` prints a progress line as the build and each target start/finish,
instead of just the final summary.

## Secrets

**Always from an environment variable, never a literal value** in
`deployments/*.json`. Every field ending `EnvVar` names the *env var*
holding the real secret, resolved live at deploy time - so the JSON file
itself is always safe to commit. A field that's a *path* to a credential
file already managed separately (an SSH key, a downloaded GCP service-
account JSON) is the one exception - a plain field, since the file itself
(not its path) is what's kept out of version control. Locally, those env
vars can come from a `.env` file (BoxLang loads it automatically); in CI,
set them as real runner secrets.

## Targets

### `local`

No `deployments/` entry needed - the only target that doesn't.

```bash
bxSites deploy --target=local --destination=/path/to/somewhere
```

### `github-pages`

The same push `gh-deploy` does, reachable from the unified command too - no
`deployments/` entry needed either.

```bash
bxSites deploy --target=github-pages [--branch=gh-pages] [--remote=origin] [--message="..."]
```

### `s3`

Real AWS S3, or any S3-compatible service. Set `endpoint` for non-AWS, and
`forcePathStyle: true` for most non-AWS providers.

```json title="deployments/production.json"
{
  "target": "s3",
  "bucket": "my-docs-site",
  "region": "us-east-1",
  "prefix": "",
  "accessKeyIdEnvVar": "AWS_ACCESS_KEY_ID",
  "secretAccessKeyEnvVar": "AWS_SECRET_ACCESS_KEY"
}
```

```json title="deployments/spaces.json (DigitalOcean Spaces)"
{
  "target": "s3",
  "bucket": "my-docs-site",
  "endpoint": "https://nyc3.digitaloceanspaces.com",
  "forcePathStyle": true,
  "accessKeyIdEnvVar": "SPACES_KEY",
  "secretAccessKeyEnvVar": "SPACES_SECRET"
}
```

Same shape (custom `endpoint` + `forcePathStyle: true`) also covers
Cloudflare R2 (`https://<accountid>.r2.cloudflarestorage.com`), Backblaze
B2, and MinIO/Wasabi.

### `azure`

```json title="deployments/production.json"
{ "target": "azure", "account": "mystorageaccount", "container": "site", "accountKeyEnvVar": "AZURE_STORAGE_KEY" }
```

Authenticate with exactly one of a SAS token, an account key, or a full
connection string.

### `gcs`

```json title="deployments/production.json"
{ "target": "gcs", "bucket": "my-docs-site", "serviceAccountKeyPath": "/path/to/service-account.json" }
```

Downloaded service-account JSON key (Google Cloud Console → IAM & Admin →
Service Accounts → Keys).

### `firebase`

```json title="deployments/production.json"
{ "target": "firebase", "siteId": "my-firebase-site", "serviceAccountKeyPath": "/path/to/service-account.json" }
```

### `ftp` / `sftp`

```json title="deployments/production.json"
{ "target": "sftp", "host": "example.com", "username": "deploy", "remotePath": "/var/www/html", "key": "/home/me/.ssh/id_rsa" }
```

SFTP accepts a password or an SSH key. Preserves the site's folder structure.

### `rsync`

```json title="deployments/production.json"
{ "target": "rsync", "host": "example.com", "username": "deploy", "remotePath": "/var/www/html", "identityFile": "/home/me/.ssh/id_rsa" }
```

Real `rsync` binary over SSH - only transfers what changed, faster than
FTP/SFTP for a full rebuild. Requires `rsync` and `ssh` on the machine
running `bxSites`.

### `netlify`

```json title="deployments/production.json"
{ "target": "netlify", "siteId": "my-site-id-or-name.netlify.app", "authTokenEnvVar": "NETLIFY_AUTH_TOKEN" }
```

### `vercel`

```json title="deployments/production.json"
{ "target": "vercel", "projectId": "my-project", "authTokenEnvVar": "VERCEL_TOKEN" }
```

### `cloudflare-pages`

```json title="deployments/production.json"
{ "target": "cloudflare-pages", "accountId": "your-account-id", "projectName": "my-project", "apiTokenEnvVar": "CLOUDFLARE_API_TOKEN" }
```

Cloudflare has no documented REST API for direct-upload deploys (only
`wrangler`) - this target reverse-engineers Wrangler's own upload flow and
needs a BLAKE3 hash implementation most default JVMs don't ship. Treat as
the roughest-edged target; verify a real deploy before relying on it.

## `package` - a plain archive instead

```bash
bxSites package
bxSites package --output=dist/my-site.zip
```

Builds, then zips `site/`'s own contents (not a wrapping `site/` folder)
into one file - for attaching to a GitHub release, a host that only accepts
a zip upload, or any target the pluggable list above doesn't reach.
`--output` defaults to `<projectRoot>/site.zip`; parent directories are
created automatically.

## GitHub Actions: multi-version publishing

bx-sites ships a ready-to-use `.github/workflows/pages.yml` pattern
publishing `main` and `development` as two independently-live versions of
the same site to GitHub Pages:

1. Installs BoxLang + bx-markdown (and whatever else the project needs)
2. Registers the repo as a module so `boxlang bxSites build` resolves
3. On any branch but `main`, points `baseURL` at `.../<branch-name>/` for
   just that build
4. Runs `boxlang bxSites build`
5. Pushes `site/` to `gh-pages` - `main` to the site root, `development` to
   `/development/`, each with `keep_files: true` and its own
   `destination_dir` so neither overwrites the other

Also triggerable manually (`workflow_dispatch`) for a one-off republish.

**One-time setup** (after the workflow has run at least once, since the
first run creates the `gh-pages` branch): Settings → Pages → Build and
deployment → Source: "Deploy from a branch" → Branch: `gh-pages` / `(root)`.

**Adding a third branch** (e.g. `release/2.0`): add it to
`on.push.branches` and give it its own `if: github.ref_name == '...'` deploy
step with `destination_dir: release-2.0`.

**Project Pages sub-path** - `https://<user>.github.io/<repo>/` (as opposed
to a `<user>.github.io` user site) needs `baseURL` set to that full URL so
every internal link/asset/nav entry gets the `/<repo>/` prefix and a real
`sitemap.xml` is generated (see `bx-sites-configuration`). A user site or a
custom domain mapped to the root can leave `baseURL` at its default (`/`).

## Restricting who can reach a deployed site

No built-in access control - a plain static `site/` has no concept of "who's
asking." `robots: false` tells well-behaved crawlers not to index it, but
it's a polite request, not a lock - the URL still works for anyone who has
it. Real access restriction has to happen in front of the static files, at
whichever host serves them:

- **Cloudflare Pages/Access** - a Cloudflare Access policy (email allowlist,
  SSO, one-time PIN), no application code.
- **Netlify** - built-in password protection per site/deploy, from site
  settings alone.
- **A tiny reverse-proxy** (any host) - HTTP Basic Auth in front of the
  static files - keeps out search engines and casual visitors, not real
  per-user identity.

None of these are bx-sites features - they're host-level settings.
