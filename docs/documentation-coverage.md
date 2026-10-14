# Documentation coverage and verification ledger

## Baseline and counting rules

This pass uses the complete, non-truncated Git tree at [`cd53d0e525352a228ef6cdec73ff4d30872b5920`](https://github.com/hsoliwal/codexpro/tree/cd53d0e525352a228ef6cdec73ff4d30872b5920): 133 files. It deepens the existing fork navigation without changing source, dependencies, build/workflow configuration, licenses, or runtime behavior. A later Markdown-only commit does not make this implementation baseline a moving branch reference.

The unit of accounting is explicit: repository; configured build/site/tooling project; language/build/entrypoint module; distribution or Java namespace/source-set package. Counts are separate categories, not a percentage of source lines or a guarantee that every method has documentation. [Architecture](architecture-and-projects.md) establishes project/package boundaries and [module inventory](module-inventory.md) enumerates every module. Asset/config/workflow/documentation files remain in file inventory even when they are not packages.

- **reviewed-complete**: purpose, full exposed contract, lifecycle, side effects, configuration, errors, inline/API semantics, and relevant verification accepted for that entity
- **partial**: substantive source-backed coverage exists, with specific remaining work
- **inventory-only**: identity/membership established but semantic review insufficient
- **blocked**: a required source cannot be inspected sufficiently; a verification prerequisite can also be recorded as blocked without relabeling already-read source
- **excluded**: an explicitly justified out-of-scope entity; none is used to hide ordinary repository components here

| Category | Inventoried | Reviewed-complete | Partial | Inventory-only | Blocked entities | Excluded |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Repository | 1 | 0 | 1 | 0 | 0 | 0 |
| Projects | 5 | 0 | 5 | 0 | 0 | 0 |
| Modules | 73 | 0 | 52 | 21 | 0 | 0 |
| Package identities | 5 | 0 | 5 | 0 | 0 | 0 |
| Total distinct category records | 84 | 0 | 63 | 21 | 0 | 0 |

Accounting covers every identified entity under the declared counting rules. Strict completed semantic coverage is **0/84**. This is intentionally not an “all packages complete” claim. Source availability is adequate for the inventory; unexecuted acceptance checks and external recipe/runtime prerequisites are separate limitations.

## Repository and project ledger

| Entity | Status | Evidence / substantive guide | Remaining work |
| --- | --- | --- | --- |
| hsoliwal/codexpro | partial | [Architecture and dependency flow](architecture-and-projects.md); pinned full tree | End-to-end docs consistency, all inline/API semantics, broader verification |
| Node application | partial | [Runtime](runtime-contracts.md), [workspace/analysis](workspace-and-analysis-contracts.md), [compute](compute-contracts.md) | Exhaustive MCP schema/inline/test acceptance beyond the compact reference, entire launcher/UI review, tests/platform failures |
| Java CPU sidecar | partial | [Java admission/routes/class contracts](compute-contracts.md#java-package-contracts) | Inline contract acceptance, concurrency/cleanup and independent receipt checks, actual Maven verification |
| Static documentation site | partial | [Project boundary](architecture-and-projects.md#projects-and-packages), browser script inventory | Full page/content/accessibility/responsive/browser review; images only inventoried |
| Hierarchy intake overlay | partial | [Tooling boundary](architecture-and-projects.md#repository-tooling-and-automation), POM/policy | External recipe implementation/version/runtime verification unavailable in this repository snapshot |
| Atomize/patternize runner | partial | [Tooling boundary](architecture-and-projects.md#repository-tooling-and-automation), full shell bodies | Inventory absent, installed recipe/prerequisites and failure/recovery behavior not executed |

## Package ledger

| Identity | Status | Contract | Remaining work |
| --- | --- | --- | --- |
| npm `codexpro@0.30.2` | partial | Three bins, files allowlist, build/runtime boundaries in [npm delivery](architecture-and-projects.md#npm-delivery-boundary) | Install/package tests and complete command/MCP compatibility contract |
| Maven `com.synexia.codexpro:camel-cpu-dag:0.1.0` | partial | Java 17/shaded executable, Node subprocess boundary in [compute](compute-contracts.md) | Actual artifact/classpath/install/packaging verification |
| `com.synexia.codexpro.dag`, main source set | partial | Eleven classes plus policy resource, [class and lifecycle contracts](compute-contracts.md#java-package-contracts) | Exhaustive inline/Javadoc/API acceptance, concurrency and failure tests |
| `com.synexia.codexpro.dag`, test source set | partial | Three classes; six observed JUnit test methods and DSL compile probe | Test execution, fixture/lifecycle coverage and acceptance adequacy |
| Maven `com.synexia.intake:repository-hierarchy-intake:1` | partial | Independent POM-packaging external-recipe invocation | External dependency source/artifact verification; no runtime receipt |

## Module and source-review ledger

The [module inventory](module-inventory.md) is the per-entity module ledger, with a contract/guide/gap for each of 39 TypeScript files, each of 29 MJS files, and each of five remaining browser/build/shell modules. Its status totals reconcile to the table above: 39 TypeScript + eight operational scripts + five other modules = 52 partial; 21 harnesses remain inventory-only.

Thirty-seven TypeScript helper/entrypoint bodies including all server handlers, eight operational script bodies or specifically identified launcher routines, eleven Java main classes, three Java test classes, one DRL, both shell entrypoints, manifests, and selected HTTP/widget routines informed the guides. The server now has a [33-operation compact API reference](mcp-api-reference.md); exhaustive schema/inline/test acceptance, launcher, and embedded UI review remain partial. Retrieval of all 128 non-image files does not mean all 128 were semantically reviewed. Five image assets have path/blob/size inventory only.

No inline Javadocs, TypeScript signatures/comments, shell comments, or source annotations were changed. Existing public types and helper comments were inspected as context, but a complete symbol-by-symbol inline/API audit has not been accepted. Missing inline completeness cannot be repaired by counting Markdown pages alone.

## Verification and open work

| Check | Evidence in this pass | Status |
| --- | --- | --- |
| Pinned recursive tree and configured boundary enumeration | 133 paths; manifests and module lists reconciled | Observed inventory |
| Source retrieval | 128 text files, five image paths; local blob/hash provenance retained | Observed input evidence, not execution |
| Local Markdown source-link/heading checks | Editorial checks against pinned paths and proposed documents | Editorial validation only; does not run repository code |
| TypeScript build / npm install | Commands declared in package.json; no local invocation | NOT_RUN |
| Java compile/JUnit/shading | POM plus two JUnit classes/six tests and compile probe read | NOT_RUN |
| npm smoke chain | 19 scripts declared sequentially; selected assertions read, full harness review incomplete | NOT_RUN |
| stress / HTTP / import / widget / CLI platform integration | Existing harnesses inventoried; relevant implementation boundaries documented | NOT_RUN |
| package smoke / pack / dependency audit | Lifecycle and CI commands read; no install, packing, audit, or publishing performed here | NOT_RUN |
| Real Chrome2api/model/tunnel integration | Adapter/launcher implementation inspected; external services not started | NOT_RUN |
| Hierarchy/OpenRewrite tooling | External recipe and generated inventory prerequisites not established | NOT_RUN; prerequisite-limited |
| Remote CI | Consult the exact current PR-head checks; historical runs are distinct evidence | Not inferred from this source-reading pass |

The [CI workflow](../.github/workflows/ci.yml) specifies Node 20/24 on Ubuntu/Windows and Java 17, install/build, Camel verify, smoke, stress, packaging, and high-severity dependency audit. A failed audit is not a documentation regression by itself, and a documentation-only green run would not prove untested semantic guarantees. Dependency repairs belong in a separately authorized change.

### Ordered remaining review work

1. Expand the compact 33-operation MCP reference into exhaustive schema/inline/test acceptance, and verify the documented discovery-name/actual-registration differences in each optional mode.
2. Finish the complete CLI/settings/watch/tunnel/UI and platform-specific process cleanup review; test lifecycle differences instead of applying one helper's guarantees to another.
3. Review every test harness body and execute the existing supported verification matrix in an appropriate authorized environment; retain exact revision, command, result, and limitations.
4. Review/complete inline public API documentation and Java package contracts under an explicitly source-editing scope, preserving behavior.
5. Validate site accessibility/rendering/assets and reusable-workflow semantics; obtain external recipe provenance before treating repository-tooling policy as implemented verification.
6. Recheck weak boundaries documented here: analysis fingerprint freshness, full-provider integration, pending-request bounds, large input allocations, output redaction depth, receipt recomputation, and multi-write recovery. These are concrete verification targets, not assertions that each is an exploitable defect.

This ledger remains partial until those obligations are resolved with evidence. Names, file counts, declarative policies, screenshots, hash fields, and historical CI alone do not close them.
