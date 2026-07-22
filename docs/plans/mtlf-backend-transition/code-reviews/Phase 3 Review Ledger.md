# Phase 3 Review Ledger

Date: 2026-07-22

Status: Closed for the current Phase 3 implementation slice; implementation commits and verification recorded

Parent documents:

- `../MTLF Backend Transition Plan.md`
- `../Phase 3 Analytics Subscription Routing.md`

Historical review records:

- `Phase 3 Code Review Findings And Remediation Plan.md`
- `Phase 3 Follow-up Code Review Findings And Remediation Plan.md`
- `Phase 3 Second Follow-up Code Review Findings And Remediation Plan.md`

Affected implementation repositories:

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`

---

## 1. Purpose

This is the canonical current review and remediation ledger for Phase 3. It
does not represent another full code review. It consolidates the already
recorded review results under the current development policy so that only
confirmed, current-slice problems can block implementation.

The three historical review documents remain evidence of what was found and
changed. Their independent acceptance gates are superseded by this ledger.
They must not be combined to create a larger current work list.

This ledger is updated in place with finding status, verification, and closing
commit. Do not create another full follow-up document unless architecture,
product scope, or the canonical Phase 3 plan changes.

---

## 2. Active Implementation Slice

The remaining Phase 3 slice is limited to:

1. Events Subscription and unified-sync state consistency owned by Phase 3.
2. Bounded reporting lifecycle for behavior currently used by Phase 3.
3. Contract correctness for standard or explicitly standard-shaped endpoints
   added or changed by Phase 3.

The slice does not include:

- model provision, generation ordering, model replacement, or model rollback
- arbitrary third-party model-inference cancellation
- unused Go traffic/Mongo cleanup after production ownership has moved
- extra recovery from a peer response that violates a mandatory standard
  contract
- real-environment MongoDB fault injection when that environment is not
  available

These exclusions do not weaken the current Phase 3 ownership boundary. Go must
not remain an active traffic/Mongo writer, and current endpoints must still
enforce the contract explicitly assigned to them.

---

## 3. Current Gate

| ID | Classification | Owner | Status | Current requirement |
|---|---|---|---|---|
| P3-RL-01 | P1 current blocker | Phase 3 | Closed in PyAnLF `88088c4` | Rejected sync must not leave partial subscription, projection, SMF mirror, or collection-intent state |
| P3-RL-02 | P2 current blocker | Phase 3 | Closed in PyAnLF `88088c4` | Supported delete and shutdown paths must not wait forever for the reporting worker |
| P3-RL-03 | P2 current blocker | Phase 3 | Closed in NWDAF `3ced6d6` and PyAnLF `88088c4` | Phase 3 standard/standard-shaped endpoints must enforce their explicit method, media, status, header, and representation contracts |
| P3-RL-F01 | Future-phase handoff | Phase 6 | Deferred | Resolve sync versus model-provision generation ordering and model replacement transaction |
| P3-RL-H01 | Optional hardening | Future hardening | Deferred | Attempt cleanup recovery when an SMF returns malformed 201 without a usable mandatory Location |
| P3-RL-L01 | Legacy cleanup | Phase 7 | Deferred | Remove unused Go traffic/Mongo types, queries, config, and tests after confirming no production caller |
| P3-RL-V01 | Integration verification gap | Phase 3 integration record | Pending environment | Measure MongoDB post-connect stalled-write and shutdown bounds |

P3-RL-01 through P3-RL-03 are closed. Deferred and environment-dependent
records do not reopen the current implementation slice.

---

## 4. P3-RL-01: Rejected Sync Can Leave Partial Phase 3 State

### Status

Closed in PyAnLF `88088c4`; targeted and full verification passed.

### Confirmed Behavior

The current sync flow prepares several managers and then commits them in
sequence. Subscription/runtime state can be published before sync projection,
SMF restore, or collection reconciliation completes. If a later operation
raises an error, the route returns 409 or 503 even though an earlier manager
may already contain the new snapshot.

The result is a split decision:

- Go treats the snapshot as rejected and may retry it.
- PyAnLF may already use part of the rejected snapshot.

This is the remaining Phase 3 portion of the earlier atomic-sync findings. The
model-generation portion is not part of this ledger item.

### Required Remediation

1. Add deterministic tests that inject failure before and after each current
   Phase 3 commit boundary.
2. Complete all schema checks, duplicate checks, state-token checks, projection
   derivation, SMF-mirror validation, and collection-plan derivation before
   active-state publication.
3. Publish one already validated Phase 3 aggregate while external CRUD and
   sync are serialized by the same transaction boundary.
4. Keep HTTP, NRF/SMF calls, MongoDB operations, scheduler joins, and other
   fallible side effects outside the commit.
5. After commit, enqueue collection reconciliation. A peer failure preserves
   intent and retry state rather than changing an accepted snapshot into a
   rejected partial transaction.
6. If a newer CRUD or sync changes the state token before commit, reject the
   stale snapshot with zero current-phase mutation.
7. Do not add model generation CAS, active-model rollback, or model
   replacement behavior in this remediation.

If the current manager layout cannot publish the Phase 3 aggregate without
changing an agreed ownership or contract, stop at the development-policy
decision gate before restructuring it.

### Required Tests

- invalid or duplicate subscription produces zero mutation
- projection preparation failure produces zero mutation
- SMF restore preparation failure produces zero mutation
- prepare followed by newer create/update/delete rejects stale commit
- post-commit collection failure retains accepted intent for retry
- successful sync publishes all current-phase state together

Use barriers or controllable fakes; do not use fixed sleep as the only
concurrency proof.

### Completion Record

- Production change: PyAnLF now captures a Phase 3 state token before prepare,
  rejects stale sync commits before publication, publishes projection,
  subscription, runtime, and restored SMF mirror state through one service
  commit, and performs runtime cleanup and collection convergence after the
  accepted commit.
- Focused tests: `uv run pytest -q tests/test_sync_api.py` passed, 9 tests.
- Full regression: `uv run ruff check .` passed; `uv run pytest -q` passed,
  212 tests with 1 skipped.
- Closing commit: PyAnLF `88088c4`

---

## 5. P3-RL-02: Reporting Worker Stop Is Unbounded

### Status

Closed in PyAnLF `88088c4`; targeted and full verification passed.

### Confirmed Behavior

The reporting scheduler previously used a finite join. A recent remediation
changed it to an unbounded join so that a worker could not outlive dependency
shutdown. If the active worker does not return, subscription deletion,
replacement, or application shutdown can wait forever.

The current problem is the absence of any upper bound. This ledger does not
require Phase 3 to solve cancellation of arbitrary third-party model
inference.

### Required Remediation

1. Add a documented stop deadline for reporting work supported in Phase 3.
2. Keep HTTP delivery bounded by request timeout, finite retry count, and
   interruptible retry waits.
3. Check stop state and runtime revision before report generation and before
   delivery so a late result cannot notify for a deleted or replaced resource.
4. Preserve dependency shutdown ordering: stop intake, request worker stop,
   drain supported bounded work, and then close dependencies.
5. Record a clear lifecycle failure if the deadline is exceeded; do not hide
   it by silently claiming shutdown completed.
6. Hand model-execution deadlines or forceful model cancellation to Phase 6.

### Required Tests

- stop interrupts retry backoff
- a timed HTTP delivery drains within its configured bound
- a late report is discarded after delete or replacement
- shutdown closes dependencies only after supported reporting work drains
- no test claims that a fake blocked model proves production model
  cancellation

### Completion Record

- Production change: PyAnLF now bounds reporting-worker stop by a configured
  deadline, rejects a deadline shorter than the HTTP request timeout, verifies
  runtime currency before generation and delivery, and releases runtime
  dependencies only after the supported worker has drained. Deadline expiry is
  raised as an explicit lifecycle failure.
- Focused tests: `uv run pytest -q tests/test_reporting.py
  tests/test_runtime_manager.py tests/test_sync_api.py` passed, 60 tests;
  `uv run ruff check .` passed.
- Full regression: `uv run ruff check .` passed; `uv run pytest -q` passed,
  212 tests with 1 skipped.
- Closing commit: PyAnLF `88088c4`

---

## 6. P3-RL-03: Phase 3 HTTP Contract Matrix Is Incomplete

### Status

Closed in NWDAF `3ced6d6` and PyAnLF `88088c4`; targeted and full verification
passed. PyMTLF unified-sync compatibility is recorded in `c0cb13a`.

### Supported Boundary

This finding is limited to endpoints added or changed by Phase 3:

- external NWDAF Events Subscription ingress and notification delivery in Go
- Go-to-PyAnLF standard-shaped Events Subscription CRUD
- PyAnLF direct SMF and UPF notification callbacks
- PyAnLF-to-Go standard-shaped NRF discovery and SMF resource operations
- PyAnLF-to-Go standard-shaped ADRF storage operation
- the explicit private sync contract, without treating sync as a 3GPP service

### Confirmed Behavior

Some routes rely on framework-default JSON parsing or route-specific error
construction. Equivalent invalid requests can therefore produce different
status codes, media types, or error bodies. Earlier remediation corrected
selected callback and peer-response paths but did not lock the complete Phase
3 endpoint matrix.

The applicable Release 18 OpenAPI sources remain:

- `specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`
- `specs/openapi/TS29510_Nnrf_NFDiscovery.yaml`
- `specs/openapi/TS29508_Nsmf_EventExposure.yaml`
- `specs/openapi/TS29564_Nupf_EventExposure.yaml`
- `specs/openapi/TS29575_Nadrf_DataManagement.yaml`

### Required Remediation

1. Add failing contract tests before changing shared handlers or middleware.
2. For standard JSON request bodies, distinguish:
   - accepted `application/json`, including valid parameters
   - unsupported or missing media type
   - malformed JSON or schema
   - oversized request body
3. Use the operation's OpenAPI success status, required Location, and expected
   representation. Do not share an over-broad success or redirect allowlist
   across create, read, replace, and delete.
4. Use the existing typed ProblemDetails path for standard errors instead of
   framework-default validation bodies.
5. Keep the implementation small: one existing or minimal shared media/error
   helper is acceptable; a new API framework is not.
6. The private sync route follows only its documented private contract. It
   does not inherit unrelated 3GPP responses, TLS, OAuth, or NRF behavior.
7. Preserve Go handler/processor/consumer ownership for standard
   communication; do not move peer communication into Python.

### Required Test Matrix

Test only operations in the supported boundary:

- valid JSON and JSON with valid media parameters
- missing or unsupported media type
- malformed and oversized body where the operation has a body
- operation-specific success
- required Location and typed representation for create success
- operation-specific redirect behavior
- typed ProblemDetails for declared failures

Do not broaden this remediation into an audit of unchanged legacy endpoints.

### Completion Record

- Production change: Go and PyAnLF now validate standard JSON request media
  types and bounded bodies on the Phase 3 endpoints, return typed
  `ProblemDetails`, validate operation-specific success representations and
  required `Location`, and preserve declared callback redirects without
  automatic follow. Private sync remains governed only by its private
  contract.
- Focused tests: `go test ./internal/anlf/... ./internal/sbi/...` passed;
  `uv run ruff check .` and `uv run pytest -q
  tests/test_events_subscription_api.py tests/test_callbacks.py
  tests/test_reporting.py tests/test_runtime_manager.py tests/test_sync_api.py`
  passed, 80 tests.
- Full regression: NWDAF `make lint`, `go test ./...`, the planned `go test
  -race` package set, and `make build` passed. PyAnLF `uv run ruff check .`
  and `uv run pytest -q` passed, 212 tests with 1 skipped. PyMTLF `uv run
  ruff check .` and `uv run pytest -q` passed, 32 tests with one dependency
  deprecation warning.
- Closing commits: NWDAF `3ced6d6`; PyAnLF `88088c4`; PyMTLF `c0cb13a`

---

## 7. Deferred And Non-blocking Records

### P3-RL-F01: Model Lifecycle Ordering

Classification: future-phase handoff to Phase 6.

The runtime batch introduced during atomic-sync remediation can race with a
newer model-provision update because subscription revision and object identity
do not form a model-generation CAS. Phase 6 must define:

- a monotonic model lifecycle token
- sync/provision ordering
- active-generation replacement and fallback
- model download/load rollback
- model-backed inference cancellation or deadline
- barrier-based concurrency tests

Phase 3 must not claim this is solved, but it also must not implement it.

### P3-RL-H01: Recovery From Malformed SMF 201

Classification: optional hardening.

`TS29508_Nsmf_EventExposure.yaml` requires SMF create 201 to include a
mandatory Location and representation. The current Phase 3 requirement is to
reject a malformed peer success through the standard peer-error path and make
the violation observable.

Deriving a cleanup URI from a body subId when mandatory Location is absent may
reduce a leak caused by a non-conforming peer, but it is extra recovery. It is
not a Phase 3 blocker unless the product plan explicitly adopts that behavior.

### P3-RL-L01: Unused Go Traffic/Mongo Surface

Classification: Phase 7 legacy cleanup.

Phase 3 still requires that Go no longer actively receive, normalize, or write
the traffic data now owned by PyAnLF. After production-call-graph verification:

- an active Go traffic/Mongo caller is a Phase 3 ownership defect
- types, queries, config, comments, or tests with no production caller are
  Phase 7 cleanup

Do not keep unused code only as a speculative Phase 5 compatibility bridge.

### P3-RL-V01: MongoDB Stalled-write Diagnostic

Classification: integration verification gap.

Unit tests and configured socket timeouts do not prove the post-connect
blackhole behavior of a real MongoDB driver connection. When the environment
is available, connect successfully, blackhole a later write, and measure:

- insert return bound
- repository availability transition
- retry behavior
- shutdown duration

Only create a production remediation if the diagnostic confirms an excessive
or unbounded result. Record the unavailable environment without calling it a
confirmed bug.

---

## 8. Historical Finding Mapping

The old IDs remain in their original documents. They are not separate current
gates:

- atomic-sync portions of P3-FR-01 and P3-SR-01 map to P3-RL-01
- model-generation/fallback portions map to P3-RL-F01
- bounded reporting lifecycle maps to P3-RL-02
- remaining Phase 3 wire-contract portions map to P3-RL-03
- malformed SMF 201 recovery beyond contract rejection maps to P3-RL-H01
- dead Go traffic/Mongo surface maps to P3-RL-L01
- MongoDB stalled-write evidence maps to P3-RL-V01

Other historical findings are not reopened. Their remediation behavior remains
protected by existing regression tests. Reopen one only when a targeted test
or direct current-slice evidence demonstrates a regression.

---

## 9. Implementation And Review Sequence

Execute one finding at a time:

1. P3-RL-01 failing tests, remediation, focused tests, targeted review.
2. Prepare a repository-separated checkpoint for P3-RL-01.
3. P3-RL-02 failing tests, remediation, focused tests, targeted review.
4. Prepare a repository-separated checkpoint for P3-RL-02.
5. P3-RL-03 failing contract matrix, remediation, focused tests, targeted
   review.
6. Run required full NWDAF, PyAnLF, and PyMTLF regression.
7. Update this ledger with verification and closing commits.

Do not combine the three remediations into another large cross-cutting rewrite.
Do not perform a fifth full review. Each follow-up review is limited to the
finding's remediation diff, direct dependencies, and regression tests.

---

## 10. Verification

### NWDAF

```bash
make lint
go test ./...
go test -race ./internal/backend ./internal/context ./internal/anlf/... ./internal/sbi/...
make build
```

### PyAnLF

```bash
uv run ruff check .
uv run pytest -q
```

### PyMTLF

```bash
uv run ruff check .
uv run pytest -q
```

PyMTLF has no current finding. Run its regression only to confirm the shared
sync envelope remains compatible. A schema or ownership change requires a
decision and plan update before implementation.

Real NRF, SMF, UPF, ADRF, MongoDB, OAuth, TLS, or UE checks must be reported
separately from unit and mock tests.

---

## 11. Closure Criteria

The remaining Phase 3 slice closes when:

1. P3-RL-01 through P3-RL-03 have deterministic regression tests and
   production fixes.
2. Their focused targeted reviews pass without a direct new P0/P1 regression.
3. Required repository lint, tests, race checks, and builds pass or have a
   concrete environment blocker record.
4. This ledger records the closing commits and verification level.
5. Deferred and non-blocking records remain assigned to their owner instead of
   being silently implemented in Phase 3.

Closure does not require a fifth full repository review, model lifecycle work,
Phase 7 dead-code removal, optional malformed-peer recovery, or a real MongoDB
diagnostic when the required environment is unavailable.

---

## 12. Iteration Record

### 2026-07-22: Review Scope Reclassification

- Adopted the current development-policy finding admission gate.
- Replaced independent follow-up acceptance gates with this ledger.
- Retained three current Phase 3 blockers.
- Deferred model lifecycle to Phase 6.
- Classified malformed SMF 201 recovery as optional hardening.
- Classified unused Go traffic/Mongo code as Phase 7 cleanup, subject to
  production-caller verification.
- Classified the MongoDB blackhole test as an integration verification gap.
- No new code review or implementation change was performed in this
  reclassification.

### 2026-07-22: P3-RL-01 Remediation

- Added a Phase 3 state token independent of runtime contents so an empty or
  otherwise unchanged stale sync cannot overwrite a newer sync or CRUD result.
- Consolidated accepted projection, subscription, runtime, and SMF mirror
  publication behind the subscription-service transaction boundary.
- Moved fallible runtime cleanup and asynchronous collection convergence after
  the accepted local commit; peer reconciliation failure retains queued retry
  intent instead of changing the sync response to rejection.
- Added deterministic regression coverage for stale-after-CRUD,
  stale-empty-sync, post-commit cleanup failure, and post-commit peer failure.
- Performed only a targeted remediation review; no fifth full review was run.

### 2026-07-22: P3-RL-02 Remediation

- Added a configurable reporting-worker stop deadline that covers the active
  HTTP request timeout and leaves retry backoff interruptible by the stop
  event.
- Added runtime-currency checks before generation and before delivery so a
  report completed after delete or replacement is discarded.
- Reordered runtime release so reporting drains before model, accuracy, and
  observation dependencies are released.
- Added deterministic tests for retry interruption, active HTTP drain,
  late-result discard, dependency ordering, and explicit timeout failure.

### 2026-07-22: P3-RL-03 Remediation

- Added bounded JSON media and body validation to the Phase 3 standard and
  standard-shaped request boundaries in Go and PyAnLF.
- Required the OpenAPI-declared success representation and `Location` where
  applicable for Events Subscription, SMF Event Exposure, and ADRF storage
  operations.
- Disabled automatic redirect following for analytics callback delivery and
  preserved the peer redirect status and `Location` through the Go boundary.
- Added operation-level tests for valid media parameters, missing or
  unsupported media, malformed or oversized bodies, success representations,
  and redirect behavior.
- Performed only a targeted remediation review of the changed endpoints and
  direct transport dependencies; no new full repository review was run.

### 2026-07-22: Phase 3 Closing Verification

- NWDAF `make lint`, all Go tests, the planned race suites, and build passed.
- PyAnLF lint passed; all tests passed with 212 passed and 1 skipped.
- PyMTLF lint passed; all tests passed with 32 passed and one dependency
  deprecation warning.
- No real NRF, SMF, UPF, ADRF, MongoDB, OAuth, TLS, or UE environment test was
  claimed. The pre-existing P3-RL-V01 MongoDB integration gap remains pending
  environment and does not reopen the current implementation slice.

### 2026-07-22: Closing Commits

- NWDAF `3ced6d6` committed the Go standard transport, backend routing,
  target-aware peer mirror, and old Go-side collection ownership removal.
- PyAnLF `88088c4` committed subscription/runtime ownership, collection,
  direct callback ingestion, raw storage, reporting lifecycle, unified sync,
  tests, and current API documentation.
- PyMTLF `c0cb13a` committed the shared unified-sync envelope and removed the
  superseded standalone data-source handshake.
- All three implementation commit messages describe the technical delta and
  contain no internal phase or review-tracking label.
