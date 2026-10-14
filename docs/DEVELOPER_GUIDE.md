# CodexPro developer field guide

CodexPro connects an MCP client to explicitly allowed local workspaces. The
[README](../README.md) covers user setup; this guide explains the source,
development checks, and how to interpret repository-intelligence results.
Read [SECURITY.md](../SECURITY.md) before exposing any workspace or tunnel.

## Understand the layers

| Layer | Source entry point | Responsibility |
| --- | --- | --- |
| User-facing CLI | [scripts/codexpro.mjs](../scripts/codexpro.mjs) | Setup, start, diagnostics, profiles, and connection workflow |
| MCP transports | [src/stdio.ts](../src/stdio.ts), [src/http.ts](../src/http.ts) | Transport startup; HTTP-specific exposure/auth concerns |
| Tool surface | [src/server.ts](../src/server.ts) | Tool registration and dispatch |
| Configuration and boundaries | [src/config.ts](../src/config.ts), [src/guard.ts](../src/guard.ts) | Runtime policy, allowed roots, blocked paths, path resolution |
| Workspace operations | [src/fsOps.ts](../src/fsOps.ts), [src/searchOps.ts](../src/searchOps.ts), [src/gitOps.ts](../src/gitOps.ts) | Bounded file, search, and Git operations |
| Repository intelligence | [src/analysis](../src/analysis) | Inventory, extraction, relationships, ranking, impact, caching |
| Optional integrations | [CPU-DAG contract](camel-kie-cpu-dag.md), [Chrome2api contract](chrome2api.md) | Separate bounded execution/inference contracts |

For example, `stdio.ts` loads configuration, creates the shared server, and
connects a `StdioServerTransport`. A transport is not an authorization model by
itself: follow the configuration and operation guards too.

The public fork's source can differ from the upstream npm package. An
`npm install -g codexpro` installation is not evidence that every feature present
in this fork is available in the installed release. Check version and origin
before diagnosing a missing command.

## Build and test prerequisites

[package.json](../package.json) requires Node.js **20 or newer**. Use npm and
the checked-in lockfile. The TypeScript build and the optional Camel/KIE Java
sidecar are separate build steps, but the aggregate test/smoke path exercises the
sidecar. Building it needs Java 17 and Maven, as described in the README.

From a trusted checkout, these commands are source-checked examples; **none were
executed** for this documentation:

```bash
# Dependency installation can run package lifecycle scripts.
npm ci

# TypeScript build and a focused analysis check.
npm run build
npm run analysis:smoke

# CLI-facing analysis coverage.
npm run analysis:cli-smoke

# Full test path builds the sidecar and TypeScript before the smoke suite.
npm test

# Additional stress and installation/package checks.
npm run stress
npm run package:smoke
```

`analysis:smoke` and `analysis:cli-smoke` each build TypeScript before their
script. `npm run smoke` itself does not perform those prerequisite builds;
`npm test` is the aggregate build-and-smoke route. A focused analysis success is
not a pass for HTTP auth, filesystem writes, handoffs, packaging, or real local
inference. Inspect individual scripts for fixture/service requirements.

Do not use `release:publish` as a verification command: it publishes a package.
The [contributing guide](../CONTRIBUTING.md) asks for scoped changes and explicit
security impact when auth, file access, shell execution, or tunnels change.

## Worked source exploration: what does workspace analysis prove?

Start at `inspectWorkspace()` in [src/analysis/index.ts](../src/analysis/index.ts).

1. The function first checks whether analysis is enabled, then calls
   `inventoryWorkspace`. Inventory establishes the file set and fingerprint.
2. The cache key includes workspace ID, inventory fingerprint, and serialized
   analysis limits. A hit reports `cache.hit: true` rather than rescanning the
   same extracted structure.
3. On a miss, `extractWorkspaceFiles` produces symbols and imports. Symbol and
   relationship counts are bounded by configuration.
4. [graph.ts](../src/analysis/graph.ts) builds relationships from extracted
   imports. An import from a file classified as a test becomes a `tests` edge;
   other imports become `imports` edges. This is not a complete runtime call graph.
5. The result includes `coverage`, `warnings`, `fingerprint`, and `cache`.
   Inventory/extraction truncation is propagated rather than hidden.
6. [cache.ts](../src/analysis/cache.ts) holds at most eight entries and refreshes
   recency on access. It is process-local and has workspace-prefix invalidation.

Now read `searchWorkspaceStructured()` in the same index file. Text matches,
definitions, related tests, and import-derived dependencies contribute to ranked
groups. Its file, byte, candidate, and result budgets are separate concerns.
For regex searches it returns a warning that grouped results are unavailable;
lexical regex handling is delegated to ripgrep elsewhere.

### Practical interpretation exercise

Suppose a result lists a definition and two dependent modules but reports
`coverage.truncated: true`. A useful handoff says: “These are the dependencies
found within this scan's bounds; inspect warnings and the remaining scope.” It
must not say: “There are exactly two callers.” Even without truncation,
import-derived edges do not prove dynamic dispatch, generated code, or all
language semantics. Follow the references and run relevant tests.

When investigating cache behavior, compare fingerprint, cache hit, limits, and
warnings between requests. A fast result alone does not establish cache reuse.

## Daily-use cheat sheet and risk boundaries

| Goal | Command or tool | What to check |
| --- | --- | --- |
| Diagnose setup | `codexpro doctor` | Installed version and local dependencies |
| Diagnose connectivity | `codexpro connection-test` | Whether requests reach the intended local server |
| Inspect configured projects | `codexpro settings show` | Only intended repositories are allowed |
| Explore a workspace | `inspect_workspace`, targeted `search` | Coverage, warnings, intent, and scope |
| Review a proposed change | `show_changes` | Actual diff, affected areas, related tests, and verification suggestions |
| Disable shell tools | `codexpro start --no-bash` | Writes are a separate policy; disabling bash does not disable every write tool |
| Use only local connectivity | `codexpro start --tunnel none` | This does not provide remote ChatGPT web connectivity |

Before launching, read the active profile and choose a disposable, trusted test
repository. Starting a connection can expose files to the client. Keep tokens,
token-bearing URLs, private paths, and `.env` values out of issue reports.

The guards in `guard.ts` enforce workspace and blocked-path boundaries, including
resolved symlink paths. They are **not an OS sandbox**. Safe bash can still run
repository package scripts; for untrusted repositories use `--no-bash` and do
not run their install/build scripts without reviewing them. Handoff mode limits
generic writes but still permits its bounded planning-file workflow.

The optional Chrome2api and CPU-DAG features have their own contracts and test
boundaries. Read those linked specifications rather than inferring permission to
execute arbitrary endpoints, commands, or model-proposed plans from the existence
of a `fabric` tool. Keep [upstream notices](../THIRD_PARTY_NOTICES.md) intact.

## Analysis result contract: missing, truncated, and cached are different

The result assembled by `inspectWorkspace()` includes:

| Field | Use it to answer | Do not infer |
| --- | --- | --- |
| `files`, `symbols`, `relationships` | What was actually inventoried/extracted? | Every file or runtime relationship was covered |
| `coverage.inventoryFiles` / `analyzedFiles` | How much inventory and extraction occurred? | The two counts must be equal |
| `coverage.truncated` and `warnings` | Which reported budgets or conditions limited the result? | False means inaccessible files do not exist |
| `fingerprint` | Which inventory metadata snapshot keyed this analysis? | A cryptographic digest of every file's contents |
| `cache.hit` / `cache.key` | Was a cached result used for this key? | Persistent cache state shared across server restarts |

[inventory.ts](../src/analysis/inventory.ts) sorts files and hashes the joined
`path:bytes:modifiedMs` strings. It does **not** hash file contents. Edits that
preserve both size and modification time can therefore escape this metadata-based
fingerprint. The inventory also deliberately omits blocked, escaping, unreadable,
binary, and oversized candidates in its catch path; those omissions do not each
produce a warning. Treat completeness separately from `truncated: false`.

### End-to-end investigation: a source file seems absent

1. Confirm the intended workspace/root in the tool result and active settings.
   `loadConfig()` canonicalizes existing root directories; a stale profile can
   point to another valid checkout without producing a missing-directory error.
2. Check [config.ts](../src/config.ts)'s blocked globs and `guard.ts` resolution.
   Build directories, dependencies, credentials, and paths escaping through a
   symlink are deliberately excluded. Do not weaken those controls just to make
   a search result larger.
3. Check whether the file is binary or exceeds the text-scan budget used by
   inventory. Hidden-file inclusion does not override path/security exclusions.
4. Inspect inventory/extraction counts and warnings. Inventory, analyzed-file,
   byte, symbol, and relationship caps are distinct settings; increasing one
   does not establish exhaustive coverage of all others.
5. For structured search, check `includeTests`, `root`, and `regex`. Tests are
   skipped unless included; generated files are skipped; regex uses a separate
   lexical path rather than the grouped result engine.
6. Compare fingerprint and cache state. If a stale-result concern remains,
   inspect invalidation callers and reproduce it in a disposable fixture with
   the analysis smoke scripts. Do not describe a cache hit alone as proof of
   correctness or a performance improvement.

For a bug report, preserve the command/tool parameters, sanitized coverage and
warning fields, expected file role, and installed version. Exclude source secrets,
absolute private paths, connector URLs, and tokens. A compact reproduction with
ordinary fixture names is more actionable than an unbounded workspace dump.
