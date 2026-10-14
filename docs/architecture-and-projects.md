# Architecture and project boundaries

This guide describes public source at [`cd53d0e525352a228ef6cdec73ff4d30872b5920`](https://github.com/hsoliwal/codexpro/tree/cd53d0e525352a228ef6cdec73ff4d30872b5920). It is a source-reading guide, not a claim that the examples, integrations, or tests were run. See [coverage and verification](documentation-coverage.md) for the remaining work.

## Start here

- [Runtime, transports, configuration, and local execution](runtime-contracts.md)
- [All 33 MCP operation registration sites](mcp-api-reference.md)
- [Workspace operations, analysis, and data boundaries](workspace-and-analysis-contracts.md)
- [Camel, local-model, document, and corpus contracts](compute-contracts.md)
- [Every source module and coverage status](module-inventory.md)
- Existing [contributor workflow](../CONTRIBUTING.md), [security policy](../SECURITY.md), [Camel guide](camel-kie-cpu-dag.md), and [Chrome2api guide](chrome2api.md)

## Projects and packages

There is one npm distribution, one separate Maven application, one static documentation site, and two separately configured repository-tooling areas. There is no npm workspace declaration, nested application package.json, root Maven reactor, or Java module-info.java in the pinned tree. A TypeScript directory is not automatically a separately published package.

| Project | Configuration and outputs | Responsibility and lifecycle |
| --- | --- | --- |
| Node application | [package.json](../package.json), [tsconfig.json](../tsconfig.json); `src/**/*.ts` → `dist/**/*.js` plus source maps, ES2022/NodeNext/strict; Node ≥20 | npm `codexpro@0.30.2`; launcher and stdio/HTTP MCP entrypoints. Dependencies provide MCP, HTTP, validation, globs, and text decoding. Runtime access depends on explicit workspace configuration |
| Java CPU sidecar | [camel-cpu-dag/pom.xml](../camel-cpu-dag/pom.xml); Java 17, `com.synexia.codexpro:camel-cpu-dag:0.1.0`; shaded `target/codexpro-camel-cpu-dag.jar` | Camel routes plus KIE/Drools admission execute a bounded list of registered text transformations. Node launches a separate JVM per request; it does not embed a JVM or maintain a sidecar pool |
| Static site | [index.html](index.html), [zh.html](zh.html), [styles.css](styles.css), [script.js](script.js), SVG/PNG assets | Two static language pages, clipboard buttons, and smooth in-page navigation. No additional package/install/server lifecycle is declared. Clipboard denial changes button feedback; the script has no service/API backend |
| Hierarchy intake overlay | [.m3/atom-pattern/pom.xml](../.m3/atom-pattern/pom.xml), [policy.properties](../.m3/atom-pattern/policy.properties) | Independent POM-packaging `com.synexia.intake:repository-hierarchy-intake:1`; invokes externally installed `m3-hierarchy-recipe:0.1.0-hierarchy1` at Maven verify. It does not reparent the application or ship that external implementation |
| Atomize/patternize runner | [.m3/atomize-patternize/bootstrap.sh](../.m3/atomize-patternize/bootstrap.sh), [run.sh](../.m3/atomize-patternize/run.sh) | Bash tools establish and compare Java/POM content manifests, then invoke an installed OpenRewrite recipe in dry-run mode. Their generated inventory is absent from the pinned tree; they are not a completed analysis receipt |

The five package identities used by the coverage ledger are the npm distribution, the sidecar Maven artifact, its `com.synexia.codexpro.dag` main-source namespace, the same namespace in the test source set, and the intake Maven artifact. The namespace/source-set distinction records production and test contracts without inventing a second Java namespace.

### npm delivery boundary

The three bins are `codexpro` → `scripts/codexpro.mjs`, `codexpro-mcp` → `dist/stdio.js`, and `codexpro-mcp-http` → `dist/http.js`. There is no package `exports` map declaring these TypeScript helpers a stable public library API. The server's MCP descriptors are the client-facing operation contracts; internal exported functions have their own assumptions.

The manifest's `files` list includes compiled output, scripts, the sidecar sources/POM/JAR, selected guides, and site assets. It is an allowlist, not all repository files. Newly added Markdown guides in this documentation expansion are repository documentation; they are not added to npm's allowlist. `prepack` builds TypeScript and runs the sidecar's Maven verify, so ordinary packing is not a read-only listing operation. `release:pack` explicitly uses `--dry-run --ignore-scripts`, checks name/version/filename, and rejects `docs/superpowers/` entries. [Release scripts](module-inventory.md#script-modules) do not confer permission to publish the fork under upstream coordinates.

### Java delivery boundary

The sidecar uses Camel 4.22.1, KIE/Drools 10.1.0, Jackson 2.20.0, and JUnit 5.13.4 in the inspected POM. Shading selects `CamelCpuDagMain`, merges service resources, excludes signature files, and disables dependency-reduced-POM generation. Eleven main classes and the DRL resource share one namespace; three test-source classes include two JUnit classes and a compile probe. There are no additional Java source packages in this snapshot. See [the class contracts](compute-contracts.md#java-package-contracts).

## Dependency and ownership flow

1. `scripts/codexpro.mjs` resolves launcher options/profile/environment, optionally provisions or starts a tunnel, and starts `dist/http.js`. `dist/stdio.js` is a separate direct entrypoint.
2. Both transports create a server through `createCodexProServer`. A server owns its workspace selection and review-checkpoint map. HTTP sessions share a workspace registry but have separate server instances and selected-workspace state.
3. `server.ts` validates operation arguments, selects capabilities by mode, invokes workspace/analysis/compute helpers, and formats/redacts results. Helper functions are not interchangeable with this policy boundary.
4. Workspace helpers perform guarded filesystem access, Git or Bash processes, attachment downloads, context-file writes, and optional session-history reads. Analysis is bounded, heuristic source inspection, not compilation.
5. `fabric` routes to declarative fleet contracts, a Java subprocess, or the literal-loopback Chrome2api adapter. Document/corpus calls read guarded UTF-8 files before passing text to summarizers. A hash receipt is not model correctness, authorization, or a signature.
6. The widget consumes result data; the static site is separate from the authenticated local onboarding page generated by `http.ts`.

## Repository tooling and automation

The overlay policy requests all-tracked-file hierarchy analysis, but the shell runner's executable content digest is specifically Java/POM-oriented. Its `find` traversal prunes build/vendor/dependency folders, hashes files twice to reject concurrent changes, compares content count/root/mode with an existing inventory, and requires `SYNEXIA_RECIPE_VERSION` for an installed external recipe. `bootstrap.sh` writes the inventory and target artifacts; it refuses an existing inventory unless explicitly rebound. `run.sh` creates manifests/receipts and invokes Maven for each POM with `-N`. These are filesystem/process side effects even when a rewrite goal is called dry-run. Neither script was run for this guide.

[CI](../.github/workflows/ci.yml) declares Ubuntu/Windows × Node 20/24, Java 17, install/build, Camel verify, smoke, stress, package checks, and dependency audit. The [public control-plane self-test](../.github/workflows/m3-reusable-control-plane-selftest.yml) calls a [reusable workflow](../.github/workflows/m3-reusable-control-plane-v1.yml) with inventory profile, no build/test/runtime, on its own workflow/doc paths or manual dispatch. That inventory configuration does not verify application behavior. The reusable workflow's full execution semantics remain outside this source-contract pass.

## Compatibility and unresolved ownership

Preserve [LICENSE](../LICENSE), [THIRD_PARTY_NOTICES.md](../THIRD_PARTY_NOTICES.md), contributor credits, upstream package/repository metadata, and existing fork notices. This guide makes no new authorship, licensing, namespace-ownership, upstream-maintainer, or release-rights claim. Extension points should be documented at their actual boundaries: registered Java capabilities, analysis-provider registration, and MCP descriptors. Registration alone does not prove every extension is integrated into every runtime path.
