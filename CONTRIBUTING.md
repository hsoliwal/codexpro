# Contributing

CodexPro is early. Good contributions make it safer, faster, and easier to explain.

## Local Setup

Use a checkout of [hsoliwal/codexpro](https://github.com/hsoliwal/codexpro) when
testing this fork. The npm package, website, and badges in the README refer to
the upstream [rebel0789/codexpro](https://github.com/rebel0789/codexpro) project;
installing the published package is not a way to verify unpublished fork changes.

Prerequisites are Node.js 20+ and npm for the TypeScript runtime, plus Java 17
and Maven for the Camel/KIE sidecar. The sidecar is optional for using the core
bridge, but it is required by this fork's full smoke suite.

Run from the checkout root:

```bash
npm ci
npm run build
npm run build:camel
npm run smoke
```

These steps follow [CI](.github/workflows/ci.yml) and
[package.json](package.json). `npm run build:camel` runs Maven `verify` for
[camel-cpu-dag/pom.xml](camel-cpu-dag/pom.xml).
`npm test` also builds the sidecar and TypeScript before the smoke suite.
Running only `npm run build` checks TypeScript compilation, not runtime behavior.

Run a local connector against a disposable, explicitly chosen test repository:

```bash
npm run connect:local -- --root /path/to/test/repo
```

Read [SECURITY.md](SECURITY.md) before enabling a tunnel or workspace writes.
Local-only transport does not itself disable file writes or commands. Safe bash
can execute repository package scripts, so use `--no-bash` for an untrusted repo.

For a deliberate public-tunnel test, the existing helper supports:

```bash
npm run connect -- --root /path/to/test/repo --bash safe --write handoff
```

Keep the resulting connection URL and token private. Tunnel setup is separate
from build validation and can require an external provider connection.

## Find the implementation

| Area | Start here |
| --- | --- |
| CLI setup, profiles, and tunnel orchestration | [scripts/codexpro.mjs](scripts/codexpro.mjs), [profile store](src/profileStore.ts) |
| MCP transports and tool registration | [stdio](src/stdio.ts), [HTTP](src/http.ts), [server](src/server.ts) |
| Workspace boundaries and policy | [guard](src/guard.ts), [configuration](src/config.ts), [security policy](SECURITY.md) |
| File, shell, and attachment operations | [file operations](src/fsOps.ts), [bash operations](src/bashOps.ts), [imports](src/importOps.ts) |
| Repository analysis | [src/analysis](src/analysis), [search](src/searchOps.ts), [Git operations](src/gitOps.ts) |
| Fork's bounded CPU DAG | [TypeScript adapter](src/camelDagOps.ts), [Java sidecar](camel-cpu-dag/src), [contract](docs/camel-kie-cpu-dag.md) |
| Fork's local-model adapter and summarizers | [Chrome2api adapter](src/chrome2ApiOps.ts), [integration guide](docs/chrome2api.md), [fleet constraints](docs/fleet-capability-invariants.md) |

Use [scripts](scripts) for executable regression examples.
[scripts/chrome2api-smoke.mjs](scripts/chrome2api-smoke.mjs) uses a local mock
server; passing that test does not establish that a real ChromeML installation
or model works. The Camel smoke test, by contrast, requires the built JAR and
an available Java runtime.

## Validation before review

For the wider CI checks, run these after the setup sequence:

```bash
npm run stress
npm pack --dry-run
npm run package:smoke
npm audit --audit-level=high
```

Packaging invokes `prepack`, which rebuilds TypeScript and the Java sidecar.
These commands may install dependencies or create local package artifacts.
Use a development checkout and report commands, Node/Java versions, results,
and any skipped checks in the PR. Dependency audit results also depend on the
registry's current advisory data.

Publishing is a separate maintainer action. Do not run `release:publish` just
to validate a documentation change.

## Useful Areas

- safer tool defaults
- better setup diagnostics
- stable tunnel setup helpers
- smaller/faster context bundles
- clearer ChatGPT tool prompts
- better Apps SDK widgets
- tests for path guards and auth boundaries
- docs that reduce user setup mistakes

## Pull Request Checklist

- Keep the change scoped.
- Do not include local tunnel URLs, auth tokens, `.env` values, or private paths.
- Run the setup and validation steps above that apply to the change.
- Separate mock-based results from any real service or platform testing.
- Update `README.md` or `CHANGELOG.md` when behavior changes.
- Explain security impact for changes touching auth, file access, shell execution, or tunnels.

## Docs Style

- Be concrete.
- Avoid hype.
- Name the exact command, mode, flag, and failure case.
- Make risk boundaries clear.
- Prefer examples that use `/path/to/repo` and `codexpro.example.com`, not local machine paths.

## Fork contributions and attribution

Open fork-specific pull requests against
[hsoliwal/codexpro](https://github.com/hsoliwal/codexpro/pulls), with `main` as the
base. Follow upstream's process separately for changes intended for
[rebel0789/codexpro](https://github.com/rebel0789/codexpro).
A fork PR is not an upstream release or acceptance.

Retain the [MIT license](LICENSE), contributor credit, and
[third-party notices](THIRD_PARTY_NOTICES.md). Dependencies and referenced
projects retain their own licenses. Do not include repository contents, logs,
screenshots, or connection details that expose private data.

## Source-backed architecture and contracts

For the pinned fork source inventory and deeper implementation guidance, start
with [architecture and project/package boundaries](docs/architecture-and-projects.md).
The companion guides cover:

- [runtime, transports, configuration, and local execution](docs/runtime-contracts.md)
- [all 33 MCP operation registration sites](docs/mcp-api-reference.md)
- [workspace operations, analysis, and data boundaries](docs/workspace-and-analysis-contracts.md)
- [Java CPU and local-model contracts](docs/compute-contracts.md)
- [every source module and its review status](docs/module-inventory.md)
- [coverage, verification limits, and remaining work](docs/documentation-coverage.md)

These documents distinguish observed implementation from intended contracts,
record unexecuted checks, and do not claim complete inline/API documentation or
complete semantic coverage. Consult current PR checks for exact-head CI status.
