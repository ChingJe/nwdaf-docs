# Implementation And Verification

Use this module for authorized code changes, defect remediation, and code
verification. Substantial slices use [Planning](planning.md); ownership or
contract changes use [Architecture](architecture.md). Findings and review
closure use [Review](review.md). Reading remediation rules does not authorize
edits during a review-only or diagnostic task.

## Code Quality And Change Safety

- Use English for code, identifiers, comments, logs, API-facing strings, and
  configuration-facing text. Implementation-facing documents in production
  repositories default to English unless their existing convention differs;
  document edits use [Documentation](documentation.md).
- Keep code explicit and readable; add comments for otherwise unclear intent,
  rather than speculative abstractions.
- Align config names with existing YAML layout and project terminology.
- Follow free5GC patterns where the repository and active boundary use them.
- Update or add tests in the same slice when supported behavior changes.
- Preserve characterized behavior unless explicitly replaced or obsolete.
- Preserve validation, required real dependencies, and agreed architecture;
  inconvenience does not justify weakening or replacing them.

## Test-first Remediation

For each admitted defect in an authorized implementation task:

1. Add or identify a deterministic failing test and confirm the expected cause.
2. Make the smallest production change closing the failure and its direct
   behavioral dependency.
3. Run focused regression verification.
4. Immediately perform the [targeted follow-up review](review.md#follow-up-review).
5. Continue until the finding closes or a shared decision gate is reached.

When remediation preserves the approved plan, architecture, owners, contracts,
and verification scope, continue without a new confirmation. Progress updates
are not approval gates. Pause for missing authority/permissions, a user change
of objective, or a [decision gate](../development_policy.md#decision-gates).

Capture structural defects in deterministic tests where possible before
editing. For concurrency tests, use events, barriers, controllable clocks, or
fault injection; fixed sleeps alone do not establish the behavior.

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

Before an NWDAF code commit, the verified final code must have `make build` and
`make lint` results. Run `make test` for behavior changes when feasible. Use
focused tests during remediation and ensure the required full suite covers the
final state. These NWDAF-specific commands do not apply automatically elsewhere.

Python repositories use their declared package/lockfile workflow, normally
repository lint and the full pytest suite before completion.

Verification results remain usable under the
[evidence-reuse conditions](../development_policy.md#loading-and-evidence-reuse).
Approval alone does not trigger another run. A relevant change or uncertain
coverage requires affected focused checks and any required full verification;
a focused result is not a substitute for a required full command.

Documentation-only work uses [document verification](documentation.md#verification)
and may skip code tests, stating the skip. Follow workspace escalation rules
before executing code, scripts, tests, or services.

Unit/mock tests do not prove real NRF, SMF, UPF, ADRF, MongoDB, OAuth, TLS, UE,
or data-plane integration. Record unavailable environment checks as gaps,
not confirmed bugs unless a current acceptance requirement is violated.
