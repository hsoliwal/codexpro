# Runtime, transport, and execution contracts

Source baseline and completeness limits are in [architecture](architecture-and-projects.md) and [coverage](documentation-coverage.md). These contracts describe inspected code, not a penetration test or a supported third-party service guarantee.

## Configuration is layered

[config.ts](../src/config.ts) exports `loadConfig(argv)`. Root selection is `--root`, `CODEXPRO_ROOT`, legacy `CODEBASE_BRIDGE_REPO_ROOT`, then cwd. Roots must exist as directories and are canonicalized. The allowed set includes the default root, repeated `--allow-root`, platform-delimited environment roots, and home only when explicitly enabled. Invalid roots throw before server creation.

The easy launcher has an additional saved-profile layer and maps its selected settings into environment variables for the HTTP child. Do not assume raw `loadConfig` reads profiles or every launcher flag. Invalid mode strings fall back in several config helpers, whereas launcher choice validation throws. Boolean/numeric conversion is field-specific; numeric configuration uses finite-number checks, flooring, and clamping rather than rejecting every out-of-range value.

| Setting | Inspected default / boundary |
| --- | --- |
| host / port | `127.0.0.1` / 8787; port clamp 1–65535 |
| Bash / writes / tool surface | `safe` / `workspace` / `standard`; session-history access `off` |
| analysis / tool cards | enabled / disabled |
| read / write / output limits | 180,000 / 1,000,000 / 120,000 bytes; configurable with separate caps |
| Bash maximum timeout | 600,000 ms default; absolute config maximum 900,000 ms; individual calls also have their own timeout |
| attachment import | 5,000,000 bytes default; maximum configured limit 50,000,000 |
| search results | 200 default, clamped 5–2,000 |
| HTTP sessions / idle TTL | 64 / 30 minutes; maximums 512 / 24 hours |
| context directory | `.ai-bridge`; relative hidden directory required, with escape/source/build/credential components rejected |
| inherited environment / absolute-path disclosure | disabled / disabled |

These limits apply to the relevant helper, not every allocation in the process. For example, a transcript reader can accumulate a long JSONL line before applying returned-message limits, and Java's standalone CLI reads all stdin before admission.

HTTP tokens, when supplied, must be at least 24 UTF-8 bytes. Absence is permitted only through the explicit local-only exemption; non-loopback, tunnel, and explicit require-token modes fail closed at HTTP startup. The stdio entrypoint sets the local exemption default before loading configuration because its transport is not HTTP. A Bash session label is a matching-call guard, not a persistent shell, MCP session, or Codex conversation identity.

## Server construction and operation contracts

See the [MCP operation reference](mcp-api-reference.md) for inputs, outputs, mode gates, and side effects of all 33 registration sites.

[server.ts](../src/server.ts) creates `McpServer`, `WorkspaceManager`, `PathGuard`, and a server-local review checkpoint map. Argument schemas are Zod shapes; `registerToolCompat` catches thrown failures and returns `isError: true` text/structured results, with timing logs only when enabled. Successful operation-specific results can also carry an error flag. Consumers should check that flag and operation fields, not merely transport success.

`minimal`, `standard`, and `full` select different tool surfaces. Bash disappears when off; generic `write`, `edit`, `apply_patch`, and `import_file` disappear unless write mode is `workspace`; `handoff_to_agent` is made available in handoff mode. Analysis and connection-test settings further filter tools. The `codexpro` action dispatcher uses remembered registered handlers and aliases. Actual discovery is authoritative: the separate name-list helper and registration filter are not identical for every optional session-history operation, so a configured mode should not be treated as proof that every advertised name is callable.

Write mode `off` does not mean no filesystem side effects anywhere: handoff/context-export workflows have separate policy and can create `.ai-bridge` files. `fabric` carries read-only MCP hints while its compute lanes launch a JVM or make loopback HTTP requests. Descriptor hints describe an interface, not an OS sandbox or a blanket ban on processes/network access.

`textResult` redacts known secret-shaped text and structured values. Result tagging adds the tool name/title and optionally compacts long structured strings for widgets. Path redaction replaces recognized workspace/home roots unless disclosure is enabled. These are heuristic output filters; they cannot prove absence of secrets or unknown sensitive paths.

### Patch and handoff write boundaries

`applyWorkspacePatch` rejects empty/oversized/secret-looking patches and symlink-mode changes; derives paths from patch headers; checks guard/write policy; acquires process-local locks; then runs `git apply --check` followed by `git apply`. Errors, nonzero exits, and skipped patches fail. This is not a transaction against outside processes; the check/apply interval and Git execution remain observable side effects. The returned diff is derived from the submitted patch, not an independent post-application verification of all disk content.

`writeAgentHandoff` bootstraps context files, writes/replaces or appends `current-plan.md`, and appends session/execution JSONL events. It returns plan/status/diff/log paths and a local-agent instruction, rather than executing that agent. These multiple writes are not an all-or-nothing transaction: a later log failure can leave an already-written plan. Keep a plan's status separate from actual implementation or test completion.

## HTTP lifecycle

[http.ts](../src/http.ts) loads configuration once, checks token requirements, constructs Express, installs request/security/auth middleware, then listens. Profile changes are saved for restart; they do not replace the active process's policy.

| Route | Contract |
| --- | --- |
| `GET /favicon.ico` | Public static SVG, registered before auth/security middleware, with a one-day cache header |
| `GET /`, `GET /setup` | Local onboarding/profile HTML; behind configured token authentication |
| `GET /healthz` | Process/configuration and request counters, with path labeling; not model readiness or end-to-end tool verification |
| `GET /admin/profile` | Current profile/runtime view |
| `POST /admin/profile` | Zod validation, 32 KiB parser/body bound, same-host Origin check when Origin exists, 30 attempts/minute per address; saves profile; other methods return 405 |
| `POST /mcp` | 20 MB JSON parser; existing session dispatch or initialization without a session header; invalid/missing session handling below |
| `GET /mcp`, `DELETE /mcp` | Require an existing valid session; SDK transport handles streaming/session deletion |

Authentication accepts a Bearer token or compatibility query token (`codexpro_token`, then `token`) using equal-length timing-safe comparison. Prefer headers because URL credentials can leak outside application control. Failed authentication returns 401 for the first ten failures in a minute per observed client address, then 429 with Retry-After. Successful tokens are checked before the failed-attempt counter. This is a single-token local connector, not a user/tenant authorization system. Reverse-proxy address aggregation can affect rate limits.

Each initialization creates a transport and server with a shared registry. Session IDs are UUID-shaped and stored in memory; missing/malformed IDs return HTTP 400 and unknown IDs 404. Activity refreshes last-seen time. Pruning runs periodically and on lookup/initialization, closing expired transports and evicting least-recently-seen entries beyond the configured count. Close is invoked without awaiting completion; this code is not a durable session store or proof that every in-flight operation is cancelled on eviction. Restart loses sessions. The direct HTTP entrypoint has no explicit graceful listener-drain sequence here.

JSON parse/size errors on MCP become JSON-RPC error envelopes with HTTP 400/413; caught dispatch failures become 500 with a generic client message while the local terminal receives details. The middleware adds no-store, no-referrer, nosniff, frame denial, and disabled camera/microphone/geolocation headers after the favicon route. Those headers do not make arbitrary clients trustworthy.

## Launcher and local commands

[scripts/codexpro.mjs](../scripts/codexpro.mjs) dispatches inspection/review, setup/settings, direct execution/watch/loop, context helpers, doctor, and launch modes. It can read/write profiles and runtime records, spawn HTTP and tunnel children, copy connector URLs, open a browser, and download a pinned tunnel executable. Its default launch tunnel is Cloudflare; raw HTTP defaults are a distinct configuration layer. `--headless` disables interactive/UI conveniences, not all network or process activity.

The inspected launch body checks roots/options/token exclusivity and port availability, requires built `dist/http.js`, starts HTTP, waits for local health, then chooses no tunnel, ngrok, Tailscale, quick Cloudflare, or named Cloudflare. Signals invoke child/runtime cleanup. The top-level failure handler also cleans up children, but this is not a transaction rolling back every profile/temp/runtime artifact. Tunnel availability depends on external programs and service setup, and URL discovery is not equivalent to a successful MCP tool call.

[cloudflared-release.mjs](../scripts/cloudflared-release.mjs) pins 2026.7.2 and eight platform/architecture asset hashes. It validates supported asset selection, declared/streamed download size (128 MiB default cap), and SHA-256 before the installer consumes the bytes. This is the repository's pinned selection, not a claim it is the latest version. See the existing [security policy](../SECURITY.md) for download and tunnel cautions.

### Local handoff execution is a separate trust boundary

`execute-handoff` reads a bounded plan, selects an executable/argument vector, confirms interactively unless `--yes`, and writes running/completed/failed/timed-out/interrupted state plus bounded output/status/diff artifacts. `--dry-run` prints the prepared command without executing it or writing result files. Custom templates must contain `{{plan_file}}` or `{{plan_text}}`; Windows batch adapters require the file form. Built-in adapters differ: inspect their exact argument arrays rather than assuming all use the same sandbox or approval policy.

The captured-process helper launches with `shell: false`, retains bounded excerpts, and uses a timeout. Exceeding the retained output size truncates output rather than stopping the child. Its timeout path kills the direct child; do not transfer the separate MCP Bash process-group guarantee to this CLI helper. Selected execution helpers were reviewed, but all platform-specific cleanup/watch/settings branches have not been audited.

`loop-handoff` defaults to three iterations (clamped 1–25), runs executor → optional test → reviewer, and requires an explicit `CODEXPRO_REVIEW=PASS|FAIL` assignment by default. Reviewer failure cannot be masked by PASS; executor/test failures also reject PASS unless the supported override is explicitly selected. FAIL requires a changed, usable follow-up plan to continue. Optional stop-on-unchanged/same-diff and human-confirmation settings constrain progression. Verdict text and file fingerprints are workflow signals, not proof of correctness. Commands remain trusted local software with their own permissions.

## UI and release modules

[toolCardWidget.ts](../src/toolCardWidget.ts) exports current v10 and legacy v9/v8 resource URIs, MCP-app MIME type, and self-contained HTML. It extracts tagged structured results with a depth/cycle guard, escapes displayed values, shows bounded previews, and can copy a displayed terminal excerpt. It consumes host globals and tool-result messages, with a 1.2-second unavailable fallback. Its message listener does not authenticate an origin itself; embedding/host isolation matters. It does not issue tool calls or network fetches. Resource registration exists even when optional tool-card metadata is disabled, except in connection-test mode.

The release guard reads the canonical package identity, current directory/INIT_CWD, npm package binding, and version. `release:publish` runs the release-check chain and invokes npm publish, and rejects `--ignore-scripts`. Those are consequential release operations, not validation commands to run casually while writing documentation. No installation, launcher, execution, publishing, or test command was run in this documentation pass.
