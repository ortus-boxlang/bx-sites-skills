---
name: bx-sites-java-integration
metadata:
  version: "1.0"
description: Add a bx-sites (ortus-boxlang/bx-sites) site to a Java/Spring Boot project via the Gradle (io.boxlang.bxsites) or Maven (io.boxlang:bxsites-maven-plugin) plugin - build/serve/deploy tasks-and-goals, pinning the BoxLang runtime/bx-sites version, and the three Spring Boot doc generators (springdoc OpenAPI wrapper, in-process Javadoc, Spring MVC controller-scan) plus the shared BoxLang/DocBox generator. Use this whenever a user wants to build/serve/deploy a bx-sites site from Gradle or Maven instead of CommandBox, or wants generated Javadoc/OpenAPI/controller docs inside that site. For the bx-sites CLI's own docbox/coldbox verbs, use bx-sites-api-docs; for the ::: openapi ::: viewer widget, use bx-sites-content-blocks.
---

# BxSites Java/Gradle/Maven Integration

Lets a Java/Spring Boot project add a bx-sites site without CommandBox or a
system-wide BoxLang install - either plugin downloads the BoxLang runtime
and bx-sites itself into a local cache on first run. Only prerequisite: a
JDK 21. Both wrap the *same* underlying logic, so task/goal coverage and
behavior are identical between them - pick whichever matches your build tool.

> **Status: pre-1.0.** Neither plugin is published yet (not on the Gradle
> Plugin Portal or Maven Central) - see `gradle-plugin/`/`maven-plugin/` in
> the bx-sites repo for source and current build/test instructions. What's
> documented here is what each plugin does once published; the mechanics are
> already real and verified.

Neither plugin duplicates `bxsites.yaml`/`.toml`/`.json` - your site's
look/theme/nav/everything-else stays controlled by that one config file
(see `bx-sites-configuration`), same as a CommandBox-run project. Both infer
the content directory (`docs/`, falling back to `src/` - except with the
Java plugin applied, where `src/` is real Java source and never used as
content) and always write output to `<projectRoot>/site/` (not settable -
bx-sites itself hardcodes it).

## Gradle

```kotlin title="build.gradle.kts"
plugins {
    id("io.boxlang.bxsites") version "<version>"
}
```

```bash
./gradlew bxSitesNew    # scaffold docs/ + bxsites.yaml
./gradlew bxSitesBuild   # render docs/**.md -> site/, real up-to-date checking
./gradlew bxSitesServe   # build + serve locally with live reload (foreground)
```

Other tasks: `bxSitesClean`, `bxSitesSearchIndex`, `bxSitesLint` (wired into
`check` by default), `bxSitesDeploy`, `bxSitesPublish` (bxSites Cloud),
`bxSitesPackage` (`site.zip`), `bxSitesStats`, `bxSitesDoctor`.
`bxSitesBuild` never runs as part of `assemble` unless you opt in.

```kotlin title="build.gradle.kts"
bxSites {
    projectRoot.set(layout.projectDirectory)
    boxlangMiniserverVersion.set("1.18.0-snapshot")
    bxSitesVersion.set("1.0.0-snapshot")
    boxlangHomeDir.set(layout.buildDirectory.dir("bxsites/boxlang-home"))
    hookIntoAssemble.set(false)   // opt-in: run bxSitesBuild as part of assemble
    hookIntoCheck.set(true)       // wires bxSitesLint into `check` (default)
}
```

## Maven

```xml title="pom.xml"
<build>
  <plugins>
    <plugin>
      <groupId>io.boxlang</groupId>
      <artifactId>bxsites-maven-plugin</artifactId>
      <version>&lt;version&gt;</version>
    </plugin>
  </plugins>
</build>
```

```bash
mvn bxsites:new
mvn bxsites:build
mvn bxsites:serve
```

The short `bxsites:<goal>` form needs the `<plugin>` block under
`<build><plugins>` specifically (not just `<pluginManagement>`); otherwise
use `mvn io.boxlang:bxsites-maven-plugin:build`. Other goals: `bxsites:clean`,
`bxsites:search-index`, `bxsites:lint`, `bxsites:deploy`, `bxsites:publish`,
`bxsites:package`, `bxsites:stats`, `bxsites:doctor`, `bxsites:docbox`. No
goal is bound to any lifecycle phase by default - bind your own in
`<executions>` if you want one to run automatically (e.g. `bxsites:build` to
`pre-site`).

**Build staleness checking** (Maven has no Gradle-style incremental engine):
`bxsites:build` compares the newest last-modified timestamp under the
content dir/config file against the newest one already in `site/`, and
skips the subprocess entirely when nothing is newer. Force it with
`mvn bxsites:build -Dbxsites.build.forceRebuild=true`.

```xml title="pom.xml"
<plugin>
  <groupId>io.boxlang</groupId>
  <artifactId>bxsites-maven-plugin</artifactId>
  <configuration>
    <projectRoot>${project.basedir}</projectRoot>
    <boxlangMiniserverVersion>1.18.0-snapshot</boxlangMiniserverVersion>
    <bxSitesVersion>1.0.0-snapshot</bxSitesVersion>
    <boxlangHomeDir>${project.build.directory}/bxsites/boxlang-home</boxlangHomeDir>
  </configuration>
</plugin>
```

## Spring Boot doc generators

Three generators, none wired into any lifecycle by default - opt in and, for
the ones that depend on another plugin's output, bind them after it runs
(e.g. `tasks.named("bxSitesBuild") { dependsOn("bxSitesOpenApiDoc") }` on
Gradle).

### OpenAPI (springdoc wrapper)

Wires an already-generated springdoc OpenAPI/Swagger spec into the site's
native `::: openapi :::` widget (see `bx-sites-content-blocks`) - no OpenAPI
parsing happens in the plugin itself, it only copies the spec file into
`assets/openapi/` and writes a thin wrapper page. Needs your own springdoc
plugin already producing that spec file, and `bxsites.yaml`'s `openapi:
true` already set (fails with an actionable error otherwise, unless
`autoPatchConfig` is on - YAML/TOML only, never JSON).

```kotlin title="Gradle: bxSitesOpenApiDoc"
bxSites {
    springBoot {
        openApi {
            enabled.set(true)
            specFile.set(layout.buildDirectory.file("openapi/openapi.json"))
            pageTitle.set("Bookshelf API")     // default: "API Reference"
            pagePath.set("api/openapi.md")     // default
            autoPatchConfig.set(false)
        }
    }
}
```

```bash title="Maven: bxsites:openapi"
mvn bxsites:openapi -Dbxsites.openapi.specFile=target/openapi/openapi.json
```

**Known limitation**: Swagger UI renders entirely client-side, so
per-endpoint text never reaches the search index - only the wrapper page's
own title/frontmatter is indexed.

### Javadoc (in-process, JDK doclet SPI)

One Markdown page per public top-level Java type, using the JDK's own
Javadoc doclet SPI in-process (no `javadoc` subprocess).

```kotlin title="Gradle: bxSitesJavadocDoc"
bxSites {
    springBoot {
        javadoc {
            enabled.set(true)
            sourceFiles.from(sourceSets.getByName("main").allJava)
            pagePathPrefix.set("api/javadoc")   // default
            tags.set(listOf("api", "javadoc"))  // default
        }
    }
}
```

```bash title="Maven: bxsites:javadoc"
mvn bxsites:javadoc
```

**Deliberately scoped down for v1** - not a complete Javadoc-to-Markdown
converter. Covers: public/protected constructors and methods of public
top-level types (nested/package-private types and fields are skipped), each
member's own (not inherited) doc comment, and `@param`/`@return`/`@throws`/
`@deprecated`. Doesn't yet: convert inline HTML in doc comments (best-effort
text extraction only); resolve `{@link}`/`{@see}` as real hyperlinks
(renders as inline code); generate an index/nav page (wiring pages into
nav is yours). Runs against source directly, so a record's
compiler-generated accessors/`toString`/`equals`/`hashCode` show up too,
matching the standard `javadoc` tool.

### Controller scan (Spring MVC fallback)

Reflection-scans compiled classes for `@Controller`/`@RestController`
classes and emits one page per controller listing its mapped endpoints -
the fallback for a project without OpenAPI generation on. Runs in a forked
JVM against the project's own runtime classpath (Spring included), never in
the plugin's own JVM.

```kotlin title="Gradle: bxSitesControllerScanDoc"
bxSites {
    springBoot {
        controllerScan {
            // enabled defaults true unless springBoot.openApi.enabled is true
            classesDir.set(layout.buildDirectory.dir("classes/java/main"))
            runtimeClasspath.from(configurations.getByName("runtimeClasspath"))
            pagePathPrefix.set("api/controllers")     // default
            tags.set(listOf("api", "controllers"))    // default
        }
    }
}
```

```bash title="Maven: bxsites:controller-scan"
mvn bxsites:controller-scan
```

Maven's goal requires `requiresDependencyResolution=RUNTIME` - bind it
after `compile` (e.g. to `process-classes`) if wiring it into your own
build. **Deliberately scoped down**: only directly-annotated
`@Controller`/`@RestController` classes are recognized (a custom stereotype
annotation built on one isn't); only directly-annotated
`@RequestMapping`/`@GetMapping`/`@PostMapping`/`@PutMapping`/
`@DeleteMapping`/`@PatchMapping` methods; nested classes skipped; no
per-endpoint description text (reflection has no access to source-level doc
comments).

## BoxLang doc generation (DocBox)

`bxSitesDocBoxDoc`/`bxsites:docbox` generate a BoxLang/CFML API reference
for a JVM project whose sources include `.bx`/`.cfc` classes - a thin
wrapper over the bx-sites `docbox` verb (same implementation both build
tools drive, so it can't drift between them). Requires `bx-docbox` installed
in the provisioned BoxLang runtime. Only options you actually set are
passed through; anything left out falls through to `bxsites.yaml` - the
config file stays the single source of truth.

```kotlin title="Gradle"
bxSites {
    boxlang {
        docbox {
            enabled.set(true)
            mappings.put("models", "models")
            projectTitle.set("Bookshelf API")   // default: site name + " API"
            excludes.set("tests|build")
            pagePathPrefix.set("api/docbox")    // default
            tags.set(listOf("api", "docbox"))   // default
        }
    }
}
```

```xml title="Maven"
<configuration>
  <mappings><models>models</models></mappings>
  <projectTitle>Bookshelf API</projectTitle>
  <excludes>tests|build</excludes>
  <pagePathPrefix>api/docbox</pagePathPrefix>
  <tags><tag>api</tag><tag>docbox</tag></tags>
</configuration>
```

See `bx-sites-api-docs` for what the generated pages look like. **There is
deliberately no ColdBox task/goal** - a ColdBox app is built and run through
CommandBox, never Gradle/Maven, so `bxSites coldbox` (see `bx-sites-api-docs`)
stays a CLI-only concern.

## What's not built yet

`bxSitesServe`/`bxsites:serve`'s live output streaming currently buffers
output with a 30-minute timeout - both wrong for a task meant to run
indefinitely.
