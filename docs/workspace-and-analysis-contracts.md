# Workspace, analysis, and data contracts

These implementation notes use the pinned revision in [architecture](architecture-and-projects.md). The [module inventory](module-inventory.md) identifies every TypeScript module and unresolved review area. No filesystem, shell, import, session-history, or analysis operation described below was executed for this documentation pass.

## Workspace identity and path policy

[guard.ts](../src/guard.ts) separates a shared `WorkspaceRegistry` from a `WorkspaceManager`'s current selection. Opening a workspace canonicalizes an existing directory, requires it to be beneath an allowed root, and derives `ws_` plus the first 24 SHA-256 hex characters of the canonical root. An identifier names a registered root; it is not a user credential. HTTP managers can share the registry while maintaining different current selections.

`PathGuard.resolve` checks lexical and canonical containment, configured blocked globs, and symlink/ancestor resolution. Write targets reject symlinks and escapes through existing ancestors. `assertTextFile` checks regular-file status and size and scans chunks for NUL bytes; it is not a full text-format or UTF-8 validator. Filesystem races remain possible between checks and later opens. Use narrow allowed roots and trusted repositories; these application checks are not an OS sandbox.

## File, image, and Git operations

[fsOps.ts](../src/fsOps.ts) provides traversal, numbered text reads, exact edits, writes, context bootstrap, and write-lock helpers.

- Reads check the guard and size before loading text. Numbered display normalizes CRLF; returned content hashes refer to the read UTF-8 text representation, not a promise to preserve arbitrary binary encodings. Range requests still read the file subject to a larger bounded allowance; an oversized selected range fails.
- Write/edit reject secret-looking content. Optional expected SHA checks detect a stale caller snapshot. Edit requires an exact old-text match, unique unless replace-all is selected; expected replacement count can be checked.
- Locks are canonical-path, process-local promises; sorted multi-lock acquisition reduces ordering conflicts among cooperating callers. They do not lock out editors, Git, another server process, or arbitrary shell commands.
- Existing-file writes reopen, recheck content, truncate, write, and sync. They are not atomic replacement with rollback. New-file writes use an exclusive temporary file and rename/cleanup. A disk failure can still leave partial state.
- `ensureAiBridge` creates missing templates/logs rather than overwriting existing content. Helper writes do not themselves enforce every server write-mode rule; callers must apply the relevant policy.

[imageOps.ts](../src/imageOps.ts) reads a guarded regular file under a size limit capped at two million bytes, recognizes PNG/JPEG/GIF/WebP signatures, and returns base64, MIME, hash, and available dimensions. Signature recognition is not full decoder validation or image-content sanitization.

[gitOps.ts](../src/gitOps.ts) runs bounded synchronous Git status/diff/log operations. Diff disables external diff/textconv helpers and scopes paths to the detected repository. Some command failures are represented as diagnostic strings rather than thrown exceptions; consumers must inspect operation-specific status/error fields. The reserved `assertGitCleanEnoughForWrite` helper is a no-op at this revision, not a clean-working-tree enforcement gate.

## Bash capability

[bashOps.ts](../src/bashOps.ts) rejects `off`, filters command prefixes/patterns in `safe`, and bypasses that filter in `full`. Safe mode can permit package build/test scripts; those scripts can themselves execute arbitrary repository-defined programs. It therefore assumes trusted source. A configured session label must match when required, but each invocation starts a new shell.

Calls spawn Bash with a guarded cwd, restricted environment by default, an output cap, and a timeout. POSIX uses a separate process group; Windows termination uses taskkill-tree behavior. Timeout/output overflow triggers termination and delayed escalation. Returned results distinguish exit code, signal, duration, stdout, and stderr; a nonzero command exit is a result, while launch failures reject. Windows decoding uses BOM/UTF-16, strict UTF-8, detector and GBK heuristics; it is not universal encoding detection. Auto selection prefers Git for Windows Bash rather than silently selecting WSL. Do not confuse this helper with the launcher's separate captured-process implementation.

## Attachment import

[importOps.ts](../src/importOps.ts) accepts a platform file-reference object containing `download_url` and `file_id`, not a bare arbitrary URL. It validates the configured host allowlist, HTTPS (except explicitly enabled loopback), absence of URL credentials, DNS address restrictions, and up to five redirects with renewed checks. An environment extension to the allowlist expands the operator's trust boundary. Separate DNS validation and connection behavior should not be described as a universal SSRF proof.

The downloader writes an exclusive OS-temporary file, enforces byte/timeout bounds, hashes content, and performs MIME sniffing. Declared/sniffed MIME disagreement is reported, not necessarily rejected. An optional expected SHA supplies an integrity comparison; without it, a computed hash alone does not authenticate the sender. Guarded destination writes use locks, reject overwrite by default, and clean temporary state in `finally`. Import does not unpack archives or execute downloaded files. Overwrite, rename, cleanup, and network failure paths should be tested on each supported platform before extending this capability.

## Context, skills, profiles, and redaction

| Module | Contract and important limit |
| --- | --- |
| [workspaceOps.ts](../src/workspaceOps.ts) | Combines tree/Git/instruction/context summaries. Optional bootstrap writes context files. Instruction-chain discovery considers AGENTS-style files along the target directory chain; returned instructions are repository content, not authorization from the local user |
| [capabilitiesOps.ts](../src/capabilitiesOps.ts) | Discovers SKILL.md metadata with ordered workspace/global roots and bounded depth/count; loads selected text without executing it. MCP configuration discovery extracts server names/source files from supported JSON/TOML patterns; it does not connect to those servers. Global discovery may read outside the workspace and is a separate opt-in/call option boundary |
| [proContext.ts](../src/proContext.ts) | Collects selected/changed/important/glob-matched text and Git/context evidence; `exportProContext` redacts then writes `.ai-bridge/pro-context.md`. Building the bundle and exporting it are distinct. One aggregate truncation helper counts characters despite byte-oriented option naming, so not every output bound is a strict UTF-8 byte ceiling |
| [profileStore.ts](../src/profileStore.ts) | Reads/writes root-hash-keyed profile/runtime JSON beneath `CODEXPRO_HOME` or the user-home default. Directory/file modes are requested as 0700/0600 with best-effort chmod. Profiles can contain credentials. Sanitized views mask specific token fields, not arbitrary secrets. `readRuntimeConnection` can remove a stale runtime file after PID checks, so that named read is not entirely side-effect free |
| [pathLabels.ts](../src/pathLabels.ts) | Replaces known absolute roots/home with labels, longest-first, and recursively maps results; cannot guarantee masking unknown absolute paths |
| [redact.ts](../src/redact.ts) | Regex-based known-token/assignment redaction and secret detection. Structured recursion stops beyond depth eight and leaves that deeper value unchanged; false positives and false negatives remain |
| [boundedAsyncCache.ts](../src/boundedAsyncCache.ts) | Process-local completed-result LRU plus concurrent identical-promise joining; JSON-size estimate, entry/byte caps, cloned returns, oversized/rejected results not cached. No TTL and no independent bound on pending distinct requests |

[pro-bundle.mjs](../scripts/pro-bundle.mjs) wraps context export and optionally invokes macOS `pbcopy`. [pro-apply.mjs](../scripts/pro-apply.mjs) reads a supplied local file or all stdin, normalizes its heading, writes/appends the plan, and appends logs. The input file/stdin is read before downstream write-size validation; these CLI helpers do not inherit every MCP operation guard just because they import shared helpers. Neither helper automatically sends the context to a model service.

## Optional session-history reader

[codexSessions.ts](../src/codexSessions.ts) is disabled by default. Metadata mode permits listing; read mode is required for transcript resolution/read/search/around helpers. Its configured history roots are separate from workspace allowed roots. This guide documents code only; no real local session history was accessed.

Listing walks session and archived-session directories to depth six, with a shared maximum 3,000 discovered JSONL files. Metadata reads bounded 64 KiB head/tail windows, inspects selected lines, skips malformed records and recognized subagent sessions, infers titles, sorts activity, and returns at most 200 selected entries. `total_found` is after discovery/filtering, not a guarantee of all history on disk.

Explicit source paths are canonicalized and confined to configured history roots; an accompanying ID must match parsed metadata. Reads return message data and text, default to the tail, and use byte offsets as cursor/resume/message identifiers. Defaults are 80 messages and 80,000 content bytes, capped at 400 messages/400,000 bytes; tool-output limits are separate. These bounds exclude formatting/metadata and do not cap every input JSONL line allocation. A cursor past current file size fails; callers must restart pagination if files are truncated/replaced. IDs/offsets are not immutable across edits.

Search is case-insensitive with role/tool/time filters and bounded matches; its snippet helper uses characters despite a byte-named option. Around reads scan from the head, prioritize the target and neighboring records within the content budget, and return chronological messages plus more/truncation flags. Invalid JSON is skipped. The module reads saved files and supplies a resume-command string; it does not attach to a live Codex conversation or execute the resume command. Privacy filtering of returned data is a caller responsibility in addition to mode selection.

## Analysis pipeline and extension seam

The ten [analysis modules](module-inventory.md#typescript-modules) implement:

1. `inventory.ts`: guarded traversal, text eligibility checks, bounded candidate list, and a path/size/mtime fingerprint. Unreadable, blocked, binary-looking, and oversize inputs are omitted. The fingerprint is not a content hash; same-stat edits can undermine cache freshness.
2. `classify.ts` and `extract.ts`: path/language/project classification and regex/line-based symbol/import extraction. They are not a compiler or full AST. Import resolution is limited, notably relative JS/TS and relative C include cases; dynamic/external dependencies can be absent.
3. `graph.ts`: bounded file/symbol/import relationships. A graph cap is not necessarily accompanied by a distinct complete-graph proof or a specific cap flag.
4. `rank.ts`: heuristic importance/intent ordering, not correctness or ownership assignment.
5. `index.ts` and `cache.ts`: inspection orchestration, budget/coverage output, and process-local eight-entry workspace analysis caching keyed by fingerprint and limits. This cache differs from `BoundedAsyncCache`; do not assume the same clone/isolation behavior.
6. `impact.ts`: changed-file assessment, direct reverse-dependency impact, filename-based related-test guesses, risk-path heuristics, and suggested existing commands. It does not execute tests, establish transitive reachability, or prove a suggested command is safe.
7. `providers.ts` and `types.ts`: provider registration/normalization and typed data/limit contracts. Registering a provider does not automatically wire it into built-in `inspectWorkspace`; the inspected orchestrator uses the built-in analysis path.

Default budgets are 20,000 inventoried files, 5,000 analyzed files, 64 MiB scanned text, 100,000 symbols, and 250,000 relationships. Treat coverage/truncation/warnings as part of the result. A small returned view can also be output-limited independently of analysis limits.

[searchOps.ts](../src/searchOps.ts) prefers ripgrep when available, passes queries via `-e`, uses blocked globs/file-size/output limits, and offers a bounded literal Node fallback. Regex needs ripgrep. The Node lexical fallback is case-sensitive; grouped analysis search is case-insensitive. Optional structured intent/symbol/test filters enrich results; analysis failure preserves lexical output with warnings. A match list is scoped evidence, not an exhaustive repository index.

## Where to verify changes

The declared smoke/stress scripts cover corresponding operation families, and Camel has separate Java tests. See [verification status](documentation-coverage.md#verification-and-open-work) before treating their names as results. Especially important future checks are external-process filesystem races, invalid UTF-8 and long JSONL lines, secret/redaction recursion, cross-session selections, provider integration, changed-stat cache behavior, and platform-specific import/termination failures. These are review targets, not newly asserted known vulnerabilities or tests claimed to exist.
