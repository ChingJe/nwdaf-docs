# Implementation And Verification

Use this module for authorized code changes, defect remediation, and code
verification. Substantial slices use [Planning](planning.md); ownership or
contract changes use [Architecture](architecture.md). Test selection and design
use [Testing](testing.md); independent review and finding closure use
[Review](review.md). Reading remediation rules does not authorize edits during
a review-only or diagnostic task.

## Code Quality And Change Safety

- Use English for code, identifiers, comments, logs, API-facing strings, and
  configuration-facing text. Implementation-facing documents in production
  repositories default to English unless their existing convention differs;
  document edits use [Documentation](documentation.md).
- Keep code explicit and readable; add comments for otherwise unclear intent,
  rather than speculative abstractions.
- Align config names with existing YAML layout and project terminology.
- Follow free5GC patterns where the repository and active boundary use them.
- Preserve characterized behavior unless explicitly replaced or obsolete.
- Preserve validation, required real dependencies, and agreed architecture;
  inconvenience does not justify weakening or replacing them.

## Defect Remediation

Confirm the defect and its cause, make the smallest production fix, and verify
the affected behavior and direct dependencies using [Testing](testing.md).
Use [follow-up review](review.md#follow-up-review) for the fix and its consequences.

When remediation preserves the approved plan, architecture, owners, contracts,
and verification scope, continue without a new confirmation. Progress updates
are not approval gates. Pause for missing authority/permissions, a user change
of objective, or a [decision gate](../development_policy.md#decision-gates).

Keep remediation within direct dependencies of the admitted finding. Classify
adjacent cleanup and speculative resilience using
[Planning](planning.md#out-of-scope-work). Broader ownership, flow, contract, or
verification changes require a decision before editing that boundary.

## Hash-level Validation

Treat new checksum, hash, or digest validation as overdesign unless explicitly
required by an existing standard/external contract or a user decision.

- Trusted locally provisioned files, datasets, and configuration need the
  loading, parsing, and semantic handling required by supported behavior, not
  additional hash-based proof of unchanged deployment input.
- Avoid operator-maintained expected hashes, duplicate derived configuration,
  sidecar integrity manifests, and multi-stage digest checks as general
  hardening.
- A deployed process needs no preparation/certification step solely to create
  or check hashes; prefer one deployment input and normal consumption.
- Tests should cover the production contract, rather than introducing
  hash-mismatch cases for an unrequired mechanism.
- Keep a contract-required hash at its defining boundary, without copying it
  into unrelated configuration/state. Existing checks are not precedent for
  new checks elsewhere.
- Hypothetical corruption or tampering alone does not justify adding a check.

This governs new plans and implementation. Removing a supported wire field or
externally relied-upon check remains a contract change subject to decision gates.

## Repository-native Verification

Distinguish focused, full, race, build, and integration verification. Use
repository-native commands and the plan's required verification matrix.

For `NWDAF/`, default to Makefile targets:

```bash
make build
make test
make lint
make run
```

NWDAF implementation completion requires `make build` and `make lint` results
for the final code. Run `make test` for behavior changes when feasible. Use
focused tests during remediation and meet the active plan's required checks.
These NWDAF-specific commands do not apply automatically elsewhere.

Python repositories use their declared package/lockfile workflow, normally
repository lint and the full pytest suite before completion.

Verification belongs to implementation and acceptance. Git delivery uses
[Delivery](delivery.md); a commit request does not create a new verification
stage or imply that open acceptance criteria have been satisfied.

Documentation-only work uses [document verification](documentation.md#verification).
Code, scripts, tests, and services follow the workspace's
[execution permissions](../development_policy.md#workspace-boundaries-and-execution).
