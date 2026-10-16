# Source module inventory

Baseline: [`cd53d0e525352a228ef6cdec73ff4d30872b5920`](https://github.com/hsoliwal/codexpro/tree/cd53d0e525352a228ef6cdec73ff4d30872b5920). This is an explicit module ledger, not a claim that every module has passed semantic/API review. See [coverage definitions and counts](documentation-coverage.md).

Counting rule: 39 TypeScript source-file modules, 29 Node ESM script-file modules, one browser JavaScript module, one Java build module, one independent intake POM module, and two Bash entrypoint modules = 73. Java classes belong to the sidecar build module and main/test package identities; they are separately documented in [compute contracts](compute-contracts.md#java-package-contracts), not mislabeled as 14 additional Maven modules. Workflow/config/data/assets remain file-level inventory, not invented language packages.

`partial` means substantive source-backed documentation exists but full inline/API/test acceptance is unfinished. `inventory-only` means manifest/path membership is established; no comprehensive test semantics are claimed. Every module has an explicit status. Documentation coverage and test success are independent.

## TypeScript modules

All 39 entries are partial. Full implementation bodies were read for 37 helpers/entrypoints including the complete server registration/handler surface; HTTP and widget reviews covered the specific complete routines described here but not every embedded UI/style branch. Even a full body read is not a completed API/inline/test acceptance review.

| Source module | Primary contract | State, side effects, or limitation | Guide |
| --- | --- | --- | --- |
| [analysis/cache.ts](../src/analysis/cache.ts) | get/set/invalidate workspace-analysis results | Eight-entry process-local cache; fingerprint freshness and shared objects matter | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/classify.ts](../src/analysis/classify.ts) | classify language, file role, project markers | Path/extension heuristics; no compile or executable detection guarantee | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/extract.ts](../src/analysis/extract.ts) | extract symbols and import candidates from text | Regex/line analysis; incomplete syntax and dependency coverage | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/graph.ts](../src/analysis/graph.ts) | construct bounded relationships | Relative resolution and edge limits; not complete transitive semantics | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/impact.ts](../src/analysis/impact.ts) | review changes and recommend checks | Direct dependents/test/risk heuristics; does not execute suggested commands | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/index.ts](../src/analysis/index.ts) | inspectWorkspace and reviewWorkspaceChanges orchestration | Reads guarded source and combines budgets/cache; registered external providers not automatically invoked | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/inventory.ts](../src/analysis/inventory.ts) | collect eligible file inventory/fingerprint | Reads stats/text eligibility; omitted files and path-size-mtime cache identity | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/providers.ts](../src/analysis/providers.ts) | register/list/normalize analysis providers | Named registry/extension seam; distinct from integration into built-in inspector | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/rank.ts](../src/analysis/rank.ts) | rank files/symbols by intent/importance | Heuristic scores, not ownership or correctness | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [analysis/types.ts](../src/analysis/types.ts) | analysis data interfaces and default limits | Compile-time shapes plus declared runtime budgets; no analysis by itself | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [bashOps.ts](../src/bashOps.ts) | runBash and runtime/encoding helpers | Shell process/tree termination, bounded output, safe-mode filter not sandbox | [Contract](workspace-and-analysis-contracts.md#bash-capability) |
| [boundedAsyncCache.ts](../src/boundedAsyncCache.ts) | BoundedAsyncCache completed/pending result lifecycle | Clone/size/LRU handling; no TTL or independent pending-key cap | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [camelDagOps.ts](../src/camelDagOps.ts) | camelDagContract and runCamelDag | File hashes/Java probe/process execution; shape validation not full hash recomputation | [Contract](compute-contracts.md#nodejvm-boundary) |
| [capabilitiesOps.ts](../src/capabilitiesOps.ts) | discover/load skills and discover MCP configuration names | Bounded local/global reads; no skill or discovered-server execution | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [chrome2ApiContract.ts](../src/chrome2ApiContract.ts) | loopback endpoint/limits/canonicalRoot contract | Literal loopback validation and framed identities; no external service installation | [Contract](compute-contracts.md#chrome2api-completion-adapter) |
| [chrome2ApiOps.ts](../src/chrome2ApiOps.ts) | status and completion HTTP operations | Timeout/response bounds; advertised model is not inference proof | [Contract](compute-contracts.md#chrome2api-completion-adapter) |
| [chromeCorpusSummarizer.ts](../src/chromeCorpusSummarizer.ts) | multi-document exact-content work sharing | Unique IDs, totals, aliases and sequential provider work; no disk deduplication | [Contract](compute-contracts.md#document-and-corpus-lifecycle) |
| [chromeDocumentSummarizer.ts](../src/chromeDocumentSummarizer.ts) | UTF-8 splitting and map/reduce summaries | Exact chunk reconstruction, convergence/depth guard; per-call rather than global deadline | [Contract](compute-contracts.md#document-and-corpus-lifecycle) |
| [chromeSummarizerContract.ts](../src/chromeSummarizerContract.ts) | summary enums/options/compatibility metadata | Native-browser API and compatibility implementation remain distinct | [Contract](compute-contracts.md#summarizer-compatibility-and-cache-behavior) |
| [chromeSummarizerOps.ts](../src/chromeSummarizerOps.ts) | validated prompts, summary receipts, cache | Advisory preferences/output instructions; timeout excluded from reuse identity | [Contract](compute-contracts.md#summarizer-compatibility-and-cache-behavior) |
| [codexSessions.ts](../src/codexSessions.ts) | list/read/search/around saved JSONL sessions | Separate opt-in history roots; bounded output/cursors, not live-session control | [Contract](workspace-and-analysis-contracts.md#optional-session-history-reader) |
| [config.ts](../src/config.ts) | loadConfig and root/mode/limit normalization | CLI/environment parsing and canonical filesystem reads can fail startup | [Contract](runtime-contracts.md#configuration-is-layered) |
| [fleetInvariants.ts](../src/fleetInvariants.ts) | fleetCapabilityContract metadata | Declarative invariants, not distributed orchestration enforcement | [Contract](compute-contracts.md#nodejvm-boundary) |
| [fsOps.ts](../src/fsOps.ts) | list/tree/read/write/edit/context bootstrap and locks | Process-local concurrency; existing writes not atomic rollback | [Contract](workspace-and-analysis-contracts.md#file-image-and-git-operations) |
| [gitOps.ts](../src/gitOps.ts) | status/diff/log/runtime helpers | Synchronous subprocesses; diagnostic strings and no-op clean-write hook | [Contract](workspace-and-analysis-contracts.md#file-image-and-git-operations) |
| [guard.ts](../src/guard.ts) | WorkspaceRegistry, WorkspaceManager, PathGuard, CodexProError | Canonical roots and lexical/real path checks; races and OS permissions remain | [Contract](workspace-and-analysis-contracts.md#workspace-identity-and-path-policy) |
| [http.ts](../src/http.ts) | HTTP entrypoint, onboarding/profile UI, sessions | Auth/middleware/session lifecycle reviewed; large inline UI not exhaustively audited | [Contract](runtime-contracts.md#http-lifecycle) |
| [imageOps.ts](../src/imageOps.ts) | viewWorkspaceImage | Bounded guarded bytes, recognized signatures/base64/hash; not full decode validation | [Contract](workspace-and-analysis-contracts.md#file-image-and-git-operations) |
| [importOps.ts](../src/importOps.ts) | importAttachmentFile | Network/temp/destination lifecycle; allowlist/DNS/redirect and optional hash checks | [Contract](workspace-and-analysis-contracts.md#attachment-import) |
| [pathLabels.ts](../src/pathLabels.ts) | pathRedactions and recursive replacements | Known-root presentation filtering; not complete privacy proof | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [proContext.ts](../src/proContext.ts) | buildProContext and exportProContext | Read/aggregate versus context-file write; mixed byte/character limits | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [profileStore.ts](../src/profileStore.ts) | workspace profile/runtime persistence | Credential-bearing JSON and best-effort modes; stale-read cleanup mutation | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [redact.ts](../src/redact.ts) | secret detection and text/structured masking | Heuristic patterns; bounded recursion can leave deeper values untouched | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [searchOps.ts](../src/searchOps.ts) | runtime backend selection and searchWorkspace | Ripgrep versus literal fallback differences; optional analysis warnings | [Contract](workspace-and-analysis-contracts.md#analysis-pipeline-and-extension-seam) |
| [server.ts](../src/server.ts) | createCodexProServer and MCP handlers | All 33 registration/handler bodies read; compact MCP reference added, exhaustive schema/inline/test acceptance remains | [Contract](runtime-contracts.md#server-construction-and-operation-contracts) |
| [stdio.ts](../src/stdio.ts) | stdio process entrypoint | Loads config, creates MCP server, connects StdioServerTransport; fatal startup logs/exits | [Contract](runtime-contracts.md#configuration-is-layered) |
| [toolCardWidget.ts](../src/toolCardWidget.ts) | widget URIs/MIME/self-contained HTML | Escaped bounded renderer and clipboard/host event lifecycle; full CSS/browser compatibility unverified | [Contract](runtime-contracts.md#ui-and-release-modules) |
| [version.ts](../src/version.ts) | CODEXPRO_VERSION from package metadata | Import-time package-name/version validation; malformed metadata can fail startup | [Contract](architecture-and-projects.md#npm-delivery-boundary) |
| [workspaceOps.ts](../src/workspaceOps.ts) | workspaceSummary and instruction/context readers | Combines guard/Git/tree/skills; optional context bootstrap writes | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |

## Script modules

Eight production/operational script modules have partial documentation below. The remaining 21 are test/stress/package-check harnesses: their paths and package-script membership are inventoried, with selected assertions read, but their full fixture/setup/failure/cleanup bodies are not all reviewed. They remain inventory-only rather than receiving credit from their filenames.

| Source module | Role or declared script membership | Coverage / limitation | Guide |
| --- | --- | --- | --- |
| [analysis-cli-smoke.mjs](../scripts/analysis-cli-smoke.mjs) | Declared directly by `smoke`, `analysis:cli-smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [analysis-smoke.mjs](../scripts/analysis-smoke.mjs) | Declared directly by `smoke`, `analysis:smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [bash-encoding-smoke.mjs](../scripts/bash-encoding-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [camel-cpu-dag-smoke.mjs](../scripts/camel-cpu-dag-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [chrome-corpus-summarizer-smoke.mjs](../scripts/chrome-corpus-summarizer-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [chrome-document-corpus-check.mjs](../scripts/chrome-document-corpus-check.mjs) | Top-level local UTF-8 document manifest | partial: Full body read; reads documents but makes no model call | [Contract](compute-contracts.md#document-and-corpus-lifecycle) |
| [chrome-document-summarizer-smoke.mjs](../scripts/chrome-document-summarizer-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [chrome-summarizer-smoke.mjs](../scripts/chrome-summarizer-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [chrome2api-smoke.mjs](../scripts/chrome2api-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [cloudflared-release.mjs](../scripts/cloudflared-release.mjs) | Pinned asset selection, bounded read, SHA validation | partial: Full helper body read; no download/install run | [Contract](runtime-contracts.md#launcher-and-local-commands) |
| [codexpro.mjs](../scripts/codexpro.mjs) | Launcher, profiles/settings, tunnel and local agent execution | partial: Selected full lifecycle/execute/loop bodies; platform/watch/settings branches remain | [Contract](runtime-contracts.md#launcher-and-local-commands) |
| [doctor-smoke.mjs](../scripts/doctor-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [execute-handoff-smoke.mjs](../scripts/execute-handoff-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [fleet-invariants-smoke.mjs](../scripts/fleet-invariants-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [http-smoke.mjs](../scripts/http-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [import-smoke.mjs](../scripts/import-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [package-install-smoke.mjs](../scripts/package-install-smoke.mjs) | Declared directly by `package:smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [pro-apply.mjs](../scripts/pro-apply.mjs) | Plan file/stdin import to context and logs | partial: Full body read; reads input before downstream write bounds | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [pro-bundle.mjs](../scripts/pro-bundle.mjs) | Context export CLI and optional pbcopy | partial: Full body read; output/write/clipboard paths unexecuted | [Contract](workspace-and-analysis-contracts.md#context-skills-profiles-and-redaction) |
| [pro-smoke.mjs](../scripts/pro-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [release-guard-smoke.mjs](../scripts/release-guard-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [release-guard.mjs](../scripts/release-guard.mjs) | Validate release cwd/identity/version/npm binding | partial: Full body read; upstream metadata retained | [Contract](runtime-contracts.md#ui-and-release-modules) |
| [release-pack.mjs](../scripts/release-pack.mjs) | Guarded npm pack dry-run manifest checks | partial: Full body read; spawns npm with scripts disabled | [Contract](architecture-and-projects.md#npm-delivery-boundary) |
| [release-publish.mjs](../scripts/release-publish.mjs) | Run release checks then npm publish | partial: Full body read; publishing not performed | [Contract](runtime-contracts.md#ui-and-release-modules) |
| [settings-smoke.mjs](../scripts/settings-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [skill-precedence-smoke.mjs](../scripts/skill-precedence-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [smoke.mjs](../scripts/smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [stress.mjs](../scripts/stress.mjs) | Declared directly by `stress` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |
| [widget-smoke.mjs](../scripts/widget-smoke.mjs) | Declared directly by `smoke` | inventory-only; no run or full harness review | [Verification](documentation-coverage.md#verification-and-open-work) |

## Remaining modules

| Module | Contract | Status |
| --- | --- | --- |
| [docs/script.js](script.js) | Static clipboard feedback and hash-navigation behavior; no app backend | partial; full body read, browser checks not run |
| [camel-cpu-dag](../camel-cpu-dag/pom.xml) | Java request/admission/routes/receipt/close lifecycle and three test-source classes in one main/test namespace | partial; main/test bodies and DRL read, JVM/build checks not run |
| [.m3/atom-pattern](../.m3/atom-pattern/pom.xml) | Separate verify-phase entrypoint to an external recipe, policy metadata | partial; local declaration read, external recipe unresolved |
| [.m3/atomize-patternize/bootstrap.sh](../.m3/atomize-patternize/bootstrap.sh) | Generates Java/POM content manifest and bound inventory; existing inventory replacement needs explicit rebind setting | partial; body read, no execution |
| [.m3/atomize-patternize/run.sh](../.m3/atomize-patternize/run.sh) | Checks inventory drift, writes receipts, invokes externally installed OpenRewrite dry-run | partial; body read, missing inventory/recipe and unexecuted verification |

## File-level remainder

The complete pinned tree contains 133 files: 39 TypeScript, 29 MJS, 14 Java, one DRL, one browser JS, two shell scripts, two POMs, three workflow YAMLs, two HTML pages, one CSS, three SVGs, two PNGs, and 34 other documentation/configuration/metadata files. All 128 non-image files were retrieved as source text; five image assets were inventoried by path/blob/size without visual or binary-format review. Source retrieval is not semantic review.

The lockfile records resolved dependency versions; no dependency provenance/security review is inferred from fetching it. Existing design/recovery/rights/prose documents are context, not proof that the current runtime implements each statement. The reusable workflow's behavior, screenshot correctness, all historical documentation, and every test harness remain explicit review debt. None is silently excluded from the repository boundary.
