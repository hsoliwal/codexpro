# M3 Public Reusable Control Plane

The public reusable workflow in this repository is the distribution surface for repository-local
M3 inventory and verification callers.

Canonical implementation and policy remain in the private Synexia repository:

- Java convergence engine: `hsoliwal/com.synexia/synexia-code-convergence`
- Canonical reusable workflow source: `hsoliwal/com.synexia/.github/workflows/m3-reusable-control-plane-v1.yml`
- Public distribution workflow: `.github/workflows/m3-reusable-control-plane-v1.yml`

The public workflow is independently authored under this repository's MIT license. It does not
contain or expose private repository content.

## Safety contract

The workflow is evidence-only. It never:

- merges pull requests;
- updates default branches;
- rebases;
- squashes;
- force-pushes;
- authorizes promotion.

It accepts bounded declarative inputs and emits tracked-file inventory plus optional deterministic
tool-candidate discovery. `fast` and `full` profiles additionally run repository-local
diff/lint/compile/test/runtime gates when the caller explicitly selects compatible adapters.

## Recommended caller

Pin callers to an immutable commit SHA:

```yaml
jobs:
  m3:
    uses: hsoliwal/codexpro/.github/workflows/m3-reusable-control-plane-v1.yml@<commit-sha>
    with:
      profile: inventory
      target_root: '.'
      build_system: none
      lint_profile: none
      run_tests: false
      run_runtime: false
      tool_discovery: true
```

Start a new repository in `inventory` mode. Upgrade to `fast` or `full` only after the
repository's actual build and lint contracts are inventoried.

## Relationship to additive convergence

Repository-wide additive convergence remains a separate mutation boundary implemented in Java in
`com.synexia`. The control-plane workflow only produces evidence used by review and verification.
A conflict whose head is merely reachable in Git ancestry is not considered content-materialized;
verified BOTH-first content reconciliation is recorded separately from ancestry-only custody.
