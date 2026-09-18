---
name: bx-sites-api-docs
metadata:
  version: "1.0"
description: Generate API reference pages inside a bx-sites (ortus-boxlang/bx-sites) site from real source code - bxSites docbox (BoxLang/CFML classes, via DocBox) and bxSites coldbox (a ColdBox app's routes, handlers, models, modules, interceptors, scheduled tasks, read straight from disk with no boot required). Use this whenever a user wants generated API/class docs, or wants a ColdBox application's routes/handlers/models documented, inside their bx-sites site. For a Java/Spring Boot project's Javadoc/OpenAPI/controller-scan docs, use bx-sites-java-integration; for the ::: openapi ::: viewer widget itself, use bx-sites-content-blocks.
---

# BxSites API Docs (DocBox & ColdBox)

Two verbs turn real source code into ordinary Markdown pages under
`docs/api/` - themed, searchable, and built/served exactly like any
hand-written page, since that's literally what they become.

## `bxSites docbox` - BoxLang/CFML class reference

Runs [DocBox](https://docbox.ortusbooks.com)'s own JSON strategy over your
classes and converts the result into Markdown. Requires the `bx-docbox`
module in the BoxLang runtime first:

```bash
install-bx-module bx-docbox   # OS binary
box install bx-docbox         # CommandBox
```

```bash
bxSites docbox
bxSites build
```

With no config, it documents whichever of `models`/`handlers`/`bifs`/
`components`/`interceptors` your project actually has, writing pages under
`docs/api/docbox/` - an overview index, one index per package, one page per
class (docblock, properties, functions grouped constructor/public/package/
private, each with signature/hint/parameter table/`@return`).

```yaml title="bxsites.yaml"
docbox:
  projectTitle: "My API"
  mappings: { models: models, bifs: bifs }
  excludes: "tests|build"
  pagePathPrefix: api/docbox
  tags: [api, docbox]
```

Every key also has a one-run CLI override flag (`--mappings:models=models
--projectTitle="My API" --pagePathPrefix=api/classes --tags=api,classes
--excludes=tests`); `--jsonDir=<path>` keeps DocBox's raw JSON output too.
`bxsites.toml` works the same as `.yaml`/`.json` for this key (see
`bx-sites-configuration`).

**Deliberately doesn't**: link between classes (an `extends`/type name
renders as plain inline code even when that class has its own page), show
inherited members, or wire pages into `nav` for you - reference the
generated `docs/api/docbox/index.md` from your own `nav`/`docs/nav.json`
(see `bx-sites-configuration`).

DocBox's own JSON output skips `property` blocks, implemented interfaces,
class-level annotations, and per-function `@return` text - bx-sites reads
those back from the class's own metadata and merges them in, so nothing
about a class is documented twice or missing without cause.

Also available from a Java build: see `bx-sites-java-integration`'s BoxLang
doc generation section - one implementation on the BoxLang side, wrapped
identically by both `bxSites docbox` and the Gradle/Maven plugins.

## `bxSites coldbox` - ColdBox application reference

Documents a [ColdBox](https://coldbox.ortusbooks.com) app purely from its
conventions on disk - routes, handlers, models/WireBox mappings, modules,
interceptors, scheduled tasks. **Never boots the app**: nothing compiles, no
datasource needs to be reachable, no env var needs to be set, so it runs
identically in CI and on a laptop.

```bash
bxSites coldbox            # from the app root
bxSites coldbox --appRoot=app   # app lives elsewhere
bxSites build
```

A resolved root with no `handlers/`, `config/ColdBox`, `config/Router`, or
`modules_app/` fails with `BxSites.NotAColdBoxApp` rather than producing
empty pages - check `--appRoot`/`coldbox.appRoot` first when that's the
error.

```yaml title="bxsites.yaml"
coldbox:
  appRoot: "."
  pagePathPrefix: api/coldbox
  tags: [api, coldbox]
  include: [routes, handlers, models, modules, interceptors, scheduler]
```

`include` decides which page sets get generated at all - drop a token to
skip that set entirely. Same one-run override pattern as `docbox`
(`--appRoot=app --include=routes,handlers --pagePathPrefix=reference
--tags=reference,api`).

```text
docs/api/coldbox/
├── index.md               # app overview and counts
├── routes.md              # every route, in declaration order
├── handlers/               # one page per handler; module handlers nest under their module
├── models/                  # one page per model, plus the binder's own mappings
├── modules/                  # one page per module (author/version/entry point/dependencies)
├── interceptors.md
└── scheduled-tasks.md
```

- **Routes** preserve declaration order (ColdBox matches the first
  matching pattern), with `resources()`/`apiResources()` expanded into their
  individual generated routes and a module route showing its actual mount
  point.
- **Handlers** list the routes reaching each action plus its doc
  comment/args; lifecycle hooks (`preHandler`, `aroundHandler`, `onError`,
  ...) get their own section rather than being listed as reachable actions;
  `init` and private methods are left out.
- **Models** separate WireBox-injected properties (`inject="..."`) from
  plain data properties; the index also lists the binder's own `map()`/
  `mapPath()`/`mapDirectory()` mappings.
- **Interceptors** cover both `config/ColdBox`-registered and
  `interceptors/`-folder-declared ones, each public method listed as the
  interception point it is. **Scheduled tasks** read `config/Scheduler`
  plus every module's own.

**Installing `bx-docbox` makes handler/model pages substantially richer**
(per-method arguments and doc comments come from DocBox) - without it the
verb still runs and lists everything, the pages just say what's missing.

**What it can't see** (purely static reading, stated plainly rather than
silently guessed): a route/mapping/task whose pattern, target, or name is
built from a variable or registered in a loop; a module installed at boot
instead of committed under `modules_app/`; anything `mapDirectory()`
resolves only once the app actually boots. Every one of these is skipped,
not guessed at - a generated page under-reports rather than lying, and
anything declared literally still parses correctly even when the rest of
that file doesn't.

**Not available from Gradle/Maven, deliberately** - a ColdBox app is built
and run through CommandBox, never a JVM build tool, so there is no
`bxSitesColdBoxDoc` task/`bxsites:coldbox` goal (they do expose `docbox`,
since BoxLang/CFML classes genuinely sit inside a JVM project - see
`bx-sites-java-integration`).

## Wiring generated pages into nav

Both verbs write ordinary content, so pages already appear in the automatic
directory nav. To place them deliberately, name the index page in your own
`nav`:

```yaml title="bxsites.yaml"
nav:
  - title: Reference
    children:
      - api/docbox/index.md
      - api/coldbox/index.md
```
