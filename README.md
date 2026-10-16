<!-- fork-homepage-guide:start -->
# Understand a repository, make a bounded change, verify the result

This [hsoliwal CodexPro fork](https://github.com/hsoliwal/codexpro) builds on
[rebel0789/codexpro](https://github.com/rebel0789/codexpro): a local Model Context
Protocol (MCP) server for repositories you explicitly allow. Its core loop is
**select a workspace → inspect/search → propose or make a permitted change →
review the diff → run relevant checks**. The existing end-user installation and
connection guide remains below.

## What exists in this fork?

The source includes filesystem/search/Git tools, path guards, bounded repository
analysis, and optional Camel/KIE CPU-DAG and loopback Chrome2api integrations.
Those integrations have distinct dependencies and contracts; they are not required
to understand the TypeScript analysis code. The M3 programme links describe a
supporting-tooling direction, not M3JDK21 runtime ownership or measured speedups.
This documentation update did not execute the server, tests, sidecar, or inference.

The upstream npm package, website and badges below retain their upstream meaning.
Installing the public package does not prove that this fork's source additions
are in that release. Build this checkout when investigating fork-specific code.

## Concepts and trust boundaries

| Concept | Practical meaning |
| --- | --- |
| MCP tool | A named operation exposed to a client through the shared server |
| Allowed workspace | A configured repository root, checked by path/operation guards |
| Inventory | The bounded file set used for analysis and its fingerprint |
| Symbol / relationship | Extracted declaration / import-derived edge, not a complete runtime call graph |
| Coverage and warnings | Limits and omissions to read before treating analysis as exhaustive |
| Fingerprint cache | Process-local reuse keyed by workspace inventory and analysis limits |
| CPU-DAG | A bounded plan admitted by fixed capabilities/policy; model-proposed data is not arbitrary executable routing |

Start in [`src/server.ts`](src/server.ts) for tool dispatch,
[`src/guard.ts`](src/guard.ts) for path policy, and
[`src/analysis/index.ts`](src/analysis/index.ts) for analysis. Read
[SECURITY.md](SECURITY.md) before exposing a workspace or tunnel. Disabling shell
tools and disabling writes are separate controls.

## Quickstart: build and inspect the fork

Use Node.js 20+ and npm. The lockfile-based install can run package lifecycle
scripts; review dependencies in a trusted development checkout first.

```sh
git clone https://github.com/hsoliwal/codexpro.git
cd codexpro
npm ci
npm run build
npm run analysis:smoke
```

These are source-checked commands, not a test result from this README update.
`npm test` also builds the Camel/KIE sidecar and therefore needs Java 17 and Maven.
Do not interpret a TypeScript or analysis-only pass as validation of tunnels,
authentication, write operations, the Java sidecar, or local inference.

## Compact cheat sheet

| Goal | Command or MCP tool | What to inspect |
| --- | --- | --- |
| Check local prerequisites | `node scripts/codexpro.mjs doctor` | The local checkout, version and dependencies |
| Build TypeScript | `npm run build` | Compiler result |
| Focus on analysis | `npm run analysis:smoke` / `npm run analysis:cli-smoke` | Bounded analysis fixtures |
| Run aggregate smoke path | `npm test` | Sidecar build + TypeScript + smoke requirements |
| Inspect a workspace | `inspect_workspace` | Coverage, warnings, fingerprint and cache |
| Review edits | `show_changes` | Actual diff, related tests and verification suggestions |

### Worked interpretation: two edges do not prove two callers

`inspectWorkspace()` inventories files, extracts symbols/imports, then calls
[`buildRelationships()`](src/analysis/graph.ts). Imports from a file classified
as a test produce `tests` edges; other imports produce `imports` edges, up to the
configured limit. If a result reports two relationships and truncation, report
“two relationships found within this scan,” then inspect warnings and remaining
scope. The relationship cap is separate from the inventory/extraction truncation flag,
so `coverage.truncated: false` does not guarantee that every edge was retained.
Inspect the configured relationship limit as well. These edges do not prove
dynamic dispatch or complete language semantics. Use the relevant compiler/tests
to verify a change.

## Continue with the right guide

- [Developer field guide](docs/DEVELOPER_GUIDE.md): source walkthrough, checks and diagnostics
- [CPU-DAG contract](docs/camel-kie-cpu-dag.md): admission, execution and receipt boundaries
- [Chrome2api integration](docs/chrome2api.md): bounded local inference contract
- [Contribution guide](CONTRIBUTING.md) and [security policy](SECURITY.md): change and exposure review

The original project README, attribution and licensing information follow intact.
<!-- fork-homepage-guide:end -->

---

## Explore this fork

Start with the [developer field guide and command cheat sheet](docs/DEVELOPER_GUIDE.md) for workspace-analysis concepts, a worked source walkthrough, build checks, and security boundaries.

## M3 direction in this fork

M3 is based on the idea that shared immutable structure and indexed metadata can
provide a basis for reusable computation. The programme starts with M3 String,
regex, and precompute in M3JDK21, then collections and SWT/Eclipse integration.

This CodexPro fork is a supporting local tooling surface for repository inspection and repeatable development workflows. Its tool and access contracts remain separate from M3JDK21 runtime ownership.

**Explore the idea:** [M3 technical design paper](https://github.com/hsoliwal/M3jdk21/blob/master/m3/papers/M3_SHARED_STRUCTURE_AND_REUSABLE_COMPUTATION.md) ·
[Programme goals and roadmap](https://github.com/hsoliwal/M3jdk21#project-goals-and-plan).
The paper explains the proposed architecture, cost tradeoffs, and evaluation plan.
Readers are invited to examine the work and contribute representative workloads.

This section describes the direction of the hsoliwal fork. Upstream documentation
follows below; upstream authorship, licenses, and project identity remain intact.

---

<p align="center">
  <img src="docs/favicon.svg" width="72" height="72" alt="CodexPro logo">
</p>

<h1 align="center">CodexPro</h1>

<p align="center">
  Give ChatGPT local coding tools for repos you explicitly allow.
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/codexpro"><img alt="npm" src="https://img.shields.io/npm/v/codexpro?style=flat-square"></a>
  <a href="https://github.com/rebel0789/codexpro/actions"><img alt="CI" src="https://img.shields.io/github/actions/workflow/status/rebel0789/codexpro/ci.yml?branch=main&style=flat-square"></a>
  <a href="https://github.com/rebel0789/codexpro/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/github/license/rebel0789/codexpro?style=flat-square"></a>
  <a href="https://rebel0789.github.io/codexpro/"><img alt="Website" src="https://img.shields.io/badge/site-GitHub%20Pages-67e8f9?style=flat-square"></a>
</p>

## Working with this fork

This is [hsoliwal/codexpro](https://github.com/hsoliwal/codexpro), forked from
[rebel0789/codexpro](https://github.com/rebel0789/codexpro). The npm install,
website, and status badges in this README identify upstream; they do not establish which
fork additions are in a published package.

- [Contributor guide](CONTRIBUTING.md): source map, checkout setup, and test boundaries
- [Security policy](SECURITY.md): workspace, command, authentication, and tunnel controls
- [CPU-DAG contract](docs/camel-kie-cpu-dag.md) and
  [Chrome2api integration](docs/chrome2api.md): fork-specific optional components
- [License](LICENSE) and [third-party provenance](THIRD_PARTY_NOTICES.md)

## What it is

CodexPro is a local MCP server. It connects **your ChatGPT session** to **your machine** and **repos you allow**.

ChatGPT can read, search, edit, review, verify, import attachments, and write handoff plans. It stays inside those roots.

It is not a hosted SaaS product, model proxy, quota bypass, account pool, or remote shell service.

## Install

Needs:

- Node.js 20+
- A ChatGPT account that can create custom MCP plugins
- An HTTPS URL to your machine for ChatGPT web (tunnel or Tailscale Funnel)

```bash
npm install -g codexpro
cd /path/to/your/repo
codexpro setup
```

## Connect in ChatGPT

1. `Settings -> Security and login` → turn **Developer mode** on (keep CSP enforcement on).
2. `Settings -> Plugins` → Plugins tab → **+** beside Search plugins.
3. Create a plugin named `CodexPro`.
4. Connection: **Server URL** → paste the URL CodexPro copied.
5. Authentication: **No Authentication / None** (change this if the form defaults to OAuth).

CodexPro auth is the token already in that URL. Do not share the URL.

| Open Plugins and click `+` | Complete the New Plugin form |
| --- | --- |
| ![Open Plugins and click the plus button](docs/images/chatgpt-plugins-add.png) | ![Complete the New Plugin form](docs/images/chatgpt-plugin-details.png) |

Daily use from the same repo:

```bash
codexpro start
```

If plugin creation fails, run `codexpro connection-test` and check whether ChatGPT requests reach the local server.

## What ChatGPT can do

With workspace write mode (the normal agent setup):

- read, search, and inspect the repo with bounded code intelligence
- edit with `write`, `edit`, or guarded `apply_patch`
- import ChatGPT attachments with `import_file`
- run allowlisted checks with `bash`
- review diffs and likely impact with `show_changes`
- write plans under `.ai-bridge`
- export a context bundle for chats that cannot call tools

### Built-in repository intelligence

CodexPro does more than raw file search:

- `inspect_workspace` maps languages, project types, entrypoints, areas, symbols, and internal relationships.
- `search` supports targeted `symbol`, `references`, and `impact` intents as well as ordinary text and regex search.
- `show_changes` identifies affected areas, likely dependents, related tests, risk signals, and relevant verification commands.
- TypeScript/JavaScript, Python, Go, Rust, Swift, Java, C#, C, and C++ declarations are recognized. Other languages retain safe inventory and lexical search.

Analysis is local, bounded, and cached by a workspace fingerprint. It needs no model API key, language-server daemon, embedding service, or vector database. Coverage and inference strength are reported instead of presented as certainty.

### Product focus

CodexPro is optimized for a narrow loop: connect ChatGPT to explicitly allowed local repositories, make a reviewable change, verify it, and preserve a handoff record. The project prioritizes:

- explicit workspace boundaries and separate controls for reads, writes, commands, sessions, and handoffs
- useful code navigation without sending a repository to a separate indexing service
- cross-platform installation and release checks on supported Node.js versions
- compact, bounded tool results that remain usable in long ChatGPT sessions

See the [roadmap](ROADMAP.md) for the next reliability, code-navigation, and workflow improvements.

## Multiple projects

One CodexPro process can allow more than one repo:

```bash
codexpro settings set --project ~/code/web --project ~/code/api
codexpro settings show
codexpro start
```

Ask ChatGPT to `open_workspace` on an allowed project. `open_current_workspace` returns to the launch repo.

For two ChatGPT accounts or hard isolation, run two CodexPro processes on different ports and Server URLs.

## Commands

```bash
codexpro setup
codexpro start
codexpro start --root /path/to/repo
codexpro doctor
codexpro connection-test
codexpro settings
codexpro inspect
codexpro review
```

Useful modes:

```bash
codexpro start --no-bash
codexpro start --tool-mode minimal
codexpro start --tool-mode full
codexpro start --mode handoff
codexpro start --mode pro
codexpro start --headless
```

Opt-in tool cards:

```bash
CODEXPRO_TOOL_CARDS=1 codexpro start
```

## Public HTTPS options

ChatGPT web needs HTTPS:

```bash
codexpro start --tunnel cloudflare          # quick demo URL (changes)
codexpro ngrok --hostname your.ngrok-free.dev
codexpro stable --hostname codexpro.example.com --tunnel-name codexpro
codexpro tailscale --hostname your-device.your-tailnet.ts.net
codexpro start --tunnel none                # local only
```

Keep a stable token for stable hostnames:

```bash
mkdir -p ~/.codexpro
openssl rand -hex 32 > ~/.codexpro/http-token
chmod 600 ~/.codexpro/http-token
```

Prefer `Authorization: Bearer <token>` when the client supports headers. The `?codexpro_token=` query form is a personal compatibility fallback.

## Safety defaults

- Public tunnels require a CodexPro HTTP token (min 24 bytes)
- Writes stay hidden unless write mode is `workspace`
- Safe bash is the default
- Blocked paths cover `.env`, keys, `.git`, build caches, and similar
- Attachment import only accepts ChatGPT Apps SDK file objects from approved HTTPS hosts

Read [SECURITY.md](SECURITY.md) before exposing a tunnel.

## Update

```bash
npm install -g codexpro@latest
codexpro --version
```

Restart `codexpro start` after updating. Saved profiles under `~/.codexpro` stay in place.

## Development

CodexPro's TypeScript runtime requires Node.js 20 or newer. Building the optional
Camel/KIE CPU-DAG sidecar also requires Java 17 and Maven.

```bash
npm install
npm run build
npm run build:camel
npm run smoke
npm run stress
npm run package:smoke
npm run release:check
```

### AI-planned, rules-admitted CPU DAGs

The `fabric` MCP tool exposes ten bounded actions:

- `invariants` returns the recursive fleet contract.
- `dag_contract` returns registered capabilities, policy root, and hard limits.
- `dag_execute` admits a JSON plan through Drools, executes its fixed capability
  with a bounded Camel SEDA route, performs stable fan-in, and returns a
  SHA-256-bound receipt.
- `chrome_contract` returns the fixed local-inference trust boundary.
- `chrome_status` verifies the loopback Chrome2api service and required model.
- `chrome_complete` executes one bounded text-only request through ChromeML and
  returns request, response, and terminal receipt roots.
- `chrome_summarizer_contract` returns the Chrome Summarizer-compatible option
  and lifecycle contract.
- `chrome_summarize` executes a bounded local summary across every documented
  type, format, length, and preference mode.
- `chrome_summarize_document` reads one workspace-confined UTF-8 document,
  creates exact byte-range chunks, maps them locally, performs a bounded
  summary-of-summaries reduction, and returns a candidate-only receipt.
- `chrome_summarize_corpus` resolves one safe top-level workspace glob,
  content-deduplicates at most 64 documents / 8 MB, summarizes each unique
  source once, and returns per-path alias receipts. Unchanged requests reuse a
  bounded process-local summary cache.

The planner is never the router. A model may propose only data; it cannot supply
Camel endpoint URIs, shell commands, executables, or credentials. See the
[CPU-DAG contract](docs/camel-kie-cpu-dag.md) and
[fleet invariants](docs/fleet-capability-invariants.md). The optional
[Chrome2api provider](docs/chrome2api.md) is loopback-only and never forwards
authorization, cookies, browser state, local file paths, or media.

Publish only from the CodexPro root:

```bash
cd /path/to/codexpro
npm run release:publish
```

## Docs

- [Website](https://rebel0789.github.io/codexpro/)
- [FAQ](FAQ.md)
- [Security](SECURITY.md)
- [Roadmap](ROADMAP.md)
- [Stable URL guide](DOMAIN_SETUP.md)
- [Changelog](CHANGELOG.md)
- [Contributors](CONTRIBUTORS.md)
