# NWDAF Priority 3 Test Refactor Detailed Plan

Date: 2026-06-22

Status: In Progress

Phase status update:

- Phase 1 completed on 2026-06-23
- remaining items are tracked as Phase 2 follow-up
- see `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Test Refactor Phase 1 Summary.md`
  for the implementation snapshot, boundary matrix, coverage summary, and
  explicit remaining gaps

---

## 1. Purpose

This plan defines the next free5GC-alignment remediation round for the main
`NWDAF/` repository after round 1 was completed.

The objective of this round is to rebuild the test safety net around the real
free5GC-style boundaries already present in the repository:

1. handler boundary
2. processor procedure boundary
3. outbound consumer boundary
4. app/context seam
5. lifecycle and scheduler behavior

This round is not only about adding more tests. It is also about correcting the
current test strategy where some unit tests cross too many layers, rely on
transport tricks, or validate implementation accidents rather than intended
behavior.

---

## 2. Planning Basis

This plan is based on the following sources:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/testing.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- the current `NWDAF/` repository state on 2026-06-22

The scan already classifies this work as Priority 3:

- add `gock` for outbound peer-NF tests
- add `gomock` at the app/interface seam
- add handler-level tests
- expand lifecycle-oriented tests
- classify Python fixtures as integration helpers

This plan keeps that ordering, but adds stricter rules for test rationality and
coverage interpretation.

---

## 3. Current Project State

### 3.1 Repository Truth

The current repository truth is:

- build and test entrypoints are from `NWDAF/Makefile`
- module and dependency truth are from `NWDAF/go.mod`
- user-facing testing guidance is in `NWDAF/docs/testing.md`
- there is currently no local `.github/workflows/` tree in `NWDAF/`

Implications:

- there is no CI workflow in-repo today that enforces one stable unit-test
  contract or coverage gate
- local test/documentation quality matters more because contributors currently
  learn the expected test workflow from `Makefile`, `README.md`, and
  `docs/testing.md`

### 3.2 Current Test Inventory

The current repository already has useful package-level tests in:

- `internal/context/`
- `internal/anlf/`
- `internal/mtlf/`
- `internal/notifier/`
- `internal/sbi/processor/`
- `internal/sbi/consumer/`
- `pkg/factory/`

Round-1 also added targeted tests for:

- update-time default re-validation
- update-time external-state reconciliation
- notifier parent-context cancellation
- application-side scheduler shutdown fan-out

Those additions are valid and should be preserved.

### 3.3 Current Structural Gaps

The current test layout still has the following confirmed gaps:

1. There are no handler-focused `api_*_test.go` files under `internal/sbi/`.
2. `go.mod` does not currently include `gock` or `gomock`.
3. Outbound HTTP tests still primarily use `httptest.NewServer`.
4. Processor tests still use handwritten app stubs and sometimes instantiate
   real consumers.
5. Python fixtures are still prominent in the repo-level testing guide, which
   blurs the line between unit tests and integration helpers.

### 3.4 Reference NF Survey: Consumer Pattern In free5GC Main

A local survey of `resources/references/free5gc-main/NFs/` shows a consistent
free5GC pattern for standardized SBI consumers:

- consumer construction is app-driven, for example `NewConsumer(app.App)`
- consumers cache generated OpenAPI API clients keyed by peer URI
- generated client configuration typically uses:
  - `NewConfiguration()`
  - `SetBasePath(uri)`
  - `SetMetrics(...)` where the NF already wires metrics
  - sometimes `SetHTTPClient(http.DefaultClient)` when the generated client path
    needs an injected transport
- consumer tests commonly use:
  - `gock`
  - `openapi.InterceptH2CClient()`
  - `gomock` app mocks

Representative local reference examples:

- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nrf_service.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nf_management_test.go`
- `resources/references/free5gc-main/NFs/chf/internal/sbi/consumer/nrf_service_test.go`
- `resources/references/free5gc-main/NFs/smf/internal/sbi/consumer/consumer.go`

The survey also shows that raw `http.Client` is still used in a few free5GC
reference paths, but mostly as an exception:

- when the integration is not covered by an existing generated OpenAPI client
- when the logic is effectively a custom REST integration layer, such as some
  BSF-oriented helper paths

This means raw `http.Client` is not categorically forbidden, but it is not the
default style for standardized SBI consumers.

### 3.5 Standardized Versus Custom Consumer Classification In NWDAF

Based on the current NWDAF code and the local OpenAPI snapshot under
`resources/openapi/openapi/`, the current outbound integrations should be split
into two classes.

Class A: standardized 3GPP/free5GC-style SBI consumers that should move toward
generated OpenAPI clients where practical:

- SMF Event Exposure
- MTLF / NWDAF ML Model Provision

Class B: local or external integration clients that may legitimately remain
raw-HTTP-based until a stronger contract source exists:

- Daisy integration
- local ML service integration
- ADRF integration, unless and until a stable generated client path is adopted

This distinction is important for both refactor scope and test design:

- Class A should align with the stronger free5GC expectations in the skill
- Class B should still improve tests and interfaces, but should not be forced
  into fake generated-client structure when the contract is not actually managed
  that way today

### 3.6 Current Test-Design Problems

This round should explicitly address not only missing coverage, but also weak
test design.

Representative issues in the current tree:

1. `internal/sbi/processor/data_collection_test.go` exercises processor logic
   by creating a real consumer plus local HTTP server.
   This proves some behavior, but it couples processor tests to consumer
   transport and makes the seam less clear.
2. `internal/sbi/consumer/ml_service_test.go` also contains
   `TestMtlfService_SubscribeToMtlf`, which mixes unrelated consumer ownership
   in one file and weakens test discoverability.
3. `internal/sbi/consumer/consumer_test.go` contains
   `TestEmbeddedMethodPromotion`, which only proves that promoted methods exist
   at compile time rather than validating runtime behavior.
4. `internal/notifier/notifier_test.go` now covers parent cancellation, but the
   cancellation test still uses real time waits and coarse sleep windows.
5. `internal/sbi/processor/data_collection_test.go` includes assertions that are
   partially justified by comments about current synchronous versus asynchronous
   implementation details, which risks locking tests to incidental behavior.

These are not all equally severe, but together they show that test quality
needs a design pass, not only a count increase.

---

## 4. free5GC-Aligned Testing Principles For This Round

This round should follow the skill guidance, with the following local
interpretation.

### 4.1 Test the Smallest Useful Boundary

Use the narrowest seam that still proves the intended behavior:

- handler tests: HTTP boundary only
- processor tests: procedure/state logic only
- consumer tests: outbound request/response logic only
- lifecycle tests: ownership, cancellation, and shutdown only

Avoid using a lower layer's transport setup when the target boundary can be
tested directly.

### 4.2 Prefer Behavioral Assertions Over Implementation Accidents

Tests should prove observable behavior, not incidental mechanics such as:

- exact goroutine timing
- comments about current synchronous setup order
- compile-only existence of promoted methods
- network indirection used only because a seam was not modeled cleanly

### 4.3 Avoid Tricky Unit Tests

For this project, a unit test is considered too tricky if it does any of the
following without a strong reason:

- spins up a local HTTP server when request interception would be enough
- instantiates a real consumer from a processor test only to reach one branch
- relies on sleeps longer than a poll interval when a deterministic hook or
  bounded wait condition could be used
- passes only because it mirrors the current implementation path rather than the
  intended contract
- validates a symbol exists while skipping real behavior assertions

### 4.4 Coverage Must Be Seam-Based, Not Vanity-Based

This round should not introduce a misleading global line-coverage target such
as "reach X percent overall" without context.

Instead, coverage must be described by seam:

- every handler under `internal/sbi/api_*.go` should have success and failure
  tests
- every touched processor path should have success and meaningful failure-path
  tests
- every outbound consumer should have request-shape and status-handling tests
- every lifecycle change should have cancellation or shutdown proof tests

Coverage numbers may still be collected, but they are support signals, not the
primary acceptance rule.

### 4.5 Apply Skill Tools Only Where They Fit

The skill mentions `gock`, `gomock`, `httptest`, `gin.CreateTestContext`, and
`openapi.InterceptH2CClient()`.

This repository should interpret them as follows:

- `httptest` plus `gin.CreateTestContext`:
  required for HTTP handler tests
- `gock`:
  preferred for outbound HTTP mocking in consumer tests and in processor tests
  that intentionally verify outbound request construction through consumer seams
- `gomock`:
  preferred at app/interface seams once the seam being exercised is stable
- `openapi.InterceptH2CClient()`:
  use only when a test actually touches a free5GC H2C client path

Important current-state note:

- current NWDAF consumers are mixed:
  - several standardized SBI paths still use raw `http.Client`
  - local workflow integrations also use raw `http.Client`
- after Class A consumers are migrated toward generated OpenAPI clients,
  `InterceptH2CClient()` should become a normal part of those consumer tests
- for Class B consumers, `InterceptH2CClient()` remains optional unless the
  implementation itself switches onto an OpenAPI H2C path

### 4.6 Generated Client First For Standardized SBI Paths

For standardized 3GPP/free5GC-style SBI consumers, this round should treat
generated OpenAPI clients as the target shape whenever the local dependency
snapshot already provides them.

Concretely:

- do not preserve a handwritten raw-HTTP consumer for a standardized API only
  because tests already exist around it
- prefer refactoring the consumer toward generated client usage first, then
  normalize tests around that path
- only keep raw HTTP for a standardized SBI path when the generated client is
  missing, materially unusable, or would require a separate contract/regeneration
  decision that this round is not prepared to make

This principle is directly supported by:

- the skill guidance in `openapi-contract.md`
- the reference NF survey under `resources/references/free5gc-main/NFs/`
- the local OpenAPI snapshot already available in `resources/openapi/openapi/`

---

## 5. Desired End State For This Round

At the end of this round, the repository should have a test strategy with these
properties.

### 5.1 Handler Coverage Exists Where the HTTP Contract Lives

Add dedicated handler tests for:

- `internal/sbi/api_eventssubscription.go`
- `internal/sbi/api_collector.go`
- `internal/sbi/api_mlmodelprovision.go`
- `internal/sbi/api_daisy_callback.go`
- `internal/sbi/api_adrf_notify.go`

Each handler test suite should verify:

- request parsing success
- invalid JSON or invalid required input
- processor failure mapping
- response status code
- response body shape, especially `ProblemDetails` versus ad hoc callback errors

### 5.2 Processor Tests Stop Carrying Transport Responsibilities

Processor tests should focus on:

- procedure branching
- context/state mutation
- reconciliation rules
- lifecycle ownership behavior
- downstream-invocation decisions

Processor tests should stop relying on real consumer construction unless the
test is explicitly validating consumer cooperation as part of a boundary test.

### 5.3 Consumer Tests Use One Deliberate Mocking Strategy

Consumer tests should move toward a stable outbound-mocking pattern:

- use `gock` to assert request method, path, body, and status handling
- intercept the specific HTTP client used by the consumer when needed
- avoid unnecessary local HTTP servers where transport interception is enough

This should cover at least:

- `NsmfService`
- `NmtlfService`
- `AdrfClient`
- `MlServiceClient`
- `DaisyClient`

Additional alignment requirement:

- the standardized SBI consumers in `NsmfService` and `NmtlfService` should no
  longer be treated as permanent raw-HTTP wrappers if generated client packages
  are already available locally
- their tests should converge toward the same generated-client plus `gock`
  pattern used by reference NFs

### 5.4 App-Seam Tests Become Easier To Refactor

The project still has an under-specified shared app boundary, so this round
should not block on Priority 4.

Instead:

- introduce `gomock` incrementally for the current local interfaces that
  processor, AnLF, MTLF, and server tests actually depend on
- keep generated mocks close to the packages that use them
- avoid turning Priority 3 into a forced `pkg/app` redesign

### 5.5 Lifecycle Tests Become More Deterministic

Lifecycle tests should prefer:

- bounded polling helpers
- explicit stop/cancel signals
- assertion on state transitions or request counts

They should reduce:

- coarse `time.Sleep(...)` waits
- timing-sensitive assumptions about how soon a goroutine starts or exits

### 5.6 Unit And Integration Guidance Are Explicitly Separated

The repo should clearly distinguish:

- pure Go unit tests
- package/module verification
- race-detection runs
- manual API tests
- Python-based fake peer integration helpers

Python helpers may remain in the repository for now, but they should no longer
be presented as part of the default unit-test story.

---

## 6. Refactor Workstreams

### Workstream A — Establish the Test Boundary Matrix

Objectives:

- map each production package to its intended test seam
- stop accidental overlap between handler, processor, and consumer tests

Required outcomes:

- one explicit list of which files own handler tests
- one explicit list of which files own consumer transport tests
- one explicit list of which tests remain integration-oriented

Planned changes:

- update the plan-derived issue notes if needed
- prepare a package-to-test-seam matrix for implementation use
- use this matrix to prevent duplicate or misplaced new tests

### Workstream B — Add Handler-Focused Test Suites

Objectives:

- lock down HTTP contract behavior before broader error-contract cleanup

Required outcomes:

- new `api_*_test.go` files under `internal/sbi/`
- handler tests built with `httptest.NewRecorder()` and
  `gin.CreateTestContext(...)`
- clear assertions on status codes and response bodies

Implementation notes:

- start with `api_eventssubscription.go` and `api_collector.go`
- then cover callback-style handlers
- keep processor internals mocked or stubbed at the narrowest possible seam

Why this comes first:

- Priority 5, SBI error-contract normalization, should not proceed before the
  HTTP boundary has tests that will catch regressions

### Workstream C — Normalize Consumer Tests Around `gock`

Objectives:

- make outbound consumer tests more consistent and less transport-heavy

Required outcomes:

- `gock` added to test dependencies
- current HTTP-server-based consumer tests migrated where appropriate
- new tests added for currently uncovered consumers such as ADRF-related paths

Implementation notes:

- for consumers that expose an `HTTPClient()`, intercept the actual client used
  by the implementation rather than relying on `http.DefaultClient`
- do not force-migrate every current `httptest.NewServer` case if the server is
  still the clearest boundary, but require justification for exceptions
- split mixed-ownership files so MTLF tests do not remain inside
  `ml_service_test.go`

### Workstream D — Align Standardized Consumers With Reference NF Structure

Objectives:

- reduce divergence from free5GC reference NF consumer design
- migrate standardized SBI paths away from ad hoc raw-HTTP request building
  where generated clients are already available

Required outcomes:

- classify each NWDAF consumer as Class A or Class B
- migrate Class A consumers toward generated OpenAPI client ownership where the
  local dependency snapshot already supports it
- keep Class B consumers explicit as external/custom integrations rather than
  pretending they follow the same contract lifecycle

Priority target paths:

- `internal/sbi/consumer/smf_service.go`
- `internal/sbi/consumer/mtlf_service.go`

Conditional targets:

- `internal/sbi/consumer/adrf_service.go` only if a stable generated client or
  approved contract-generation path is adopted during this work

Explicit raw-HTTP holdouts allowed in this round:

- `internal/sbi/consumer/daisy_service.go`
- `internal/sbi/consumer/ml_service.go`

Implementation notes:

- follow the reference NF pattern of app-driven consumer ownership and cached
  generated API clients keyed by peer URI
- use generated request/response models where the local `openapi` snapshot
  already exposes them
- avoid mixing this with a full `pkg/app` redesign; the target is consumer
  structure first, not global boundary cleanup

### Workstream E — Introduce `gomock` At Real App Seams

Objectives:

- replace handwritten stubs where they currently weaken test clarity

Required outcomes:

- `gomock` added to test dependencies
- package-local mocks for the currently used app interfaces
- processor and lifecycle tests no longer depend on ad hoc handwritten app
  stubs where mocks express the contract more clearly

Implementation notes:

- do this incrementally
- do not wait for a full `pkg/app` consolidation
- keep mock generation scope narrow to avoid repo-wide mock churn

### Workstream F — Improve Test Rationality And Determinism

Objectives:

- remove low-value or fragile test patterns

Required outcomes:

- replace compile-surface-only tests with behavior tests or delete them
- reduce implementation-timing assumptions in async/lifecycle tests
- explicitly mark the few remaining timing-based tests that cannot yet be
  simplified

Examples to address:

- `TestEmbeddedMethodPromotion`
- time-based waits in notifier cancellation tests
- tests whose comments explain why an implementation detail currently makes the
  assertion safe

### Workstream G — Refresh Test Documentation And Coverage Reporting

Objectives:

- align repository guidance with the actual test strategy after refactor

Required outcomes:

- `NWDAF/docs/testing.md` updated to distinguish unit, race, and integration
  paths
- `README.md` testing section updated if commands or expectations change
- coverage reporting guidance added without overstating line coverage as proof

Coverage reporting rule for this round:

- report package-level coverage snapshots for touched packages
- describe gaps in words
- do not present a single overall percentage as sufficient evidence of quality

---

## 7. Proposed Execution Order

The following order minimizes dependency risk and matches the scan sequence.

1. Build the boundary matrix and identify test ownership.
2. Classify current consumers into standardized versus custom integration paths.
3. Refactor standardized consumers toward generated OpenAPI clients.
4. Add handler tests for subscription and collector endpoints.
5. Add handler tests for callback-style endpoints.
6. Introduce `gock` and normalize consumer tests.
7. Add missing consumer tests for SMF, MTLF, and ADRF paths.
8. Introduce `gomock` for the most unstable handwritten app stubs.
9. Simplify or remove fragile timing-dependent and compile-surface-only tests.
10. Refresh `docs/testing.md` and related testing instructions.

Recommended implementation split:

- change set 1: standardized consumer classification and first client migration
- change set 2: handler test scaffolding plus first handler suites
- change set 3: consumer test normalization plus dependency additions
- change set 4: app-seam mocks plus processor/lifecycle cleanup
- change set 5: testing documentation and coverage reporting updates

This should remain separate from Priority 4 app-boundary reconstruction.

---

## 8. Coverage Policy For This Round

### 8.1 Coverage Goal

The goal is to improve confidence at the architecture-sensitive seams, not to
optimize one aggregate number.

### 8.2 Minimum Coverage Expectations By Layer

Handler layer:

- success path
- malformed JSON or missing required field
- processor error mapping
- response-body contract

Processor layer:

- success path
- state mutation or reconciliation effect
- meaningful failure path
- no accidental transport coupling

Consumer layer:

- request path and method
- request body shape where applicable
- success status handling
- non-2xx handling
- decode failure or empty-body edge case where relevant
- for Class A consumers, proof that the generated-client path is the one under
  test, not a legacy handwritten fallback

Lifecycle layer:

- cancellation
- stop/shutdown
- repeated start/stop safety where applicable

### 8.3 Coverage Measurement Guidance

Recommended reporting commands during implementation:

```bash
go test ./internal/sbi/... -cover
go test ./internal/notifier ./internal/context -cover
go test ./internal/... -race
go test ./... 
```

If more precise package reporting is needed, use focused coverprofiles for the
touched packages and include them in implementation notes.

### 8.4 What Coverage Must Not Mean

The following are not acceptable substitutes for real confidence:

- high coverage produced by broad helper invocation with weak assertions
- tests that pass only because a server or goroutine happened to respond quickly
- compile-only tests counted as meaningful seam coverage
- integration-harness coverage counted as unit-level proof

---

## 9. Non-Goals

This round should not include:

- rebuilding the shared `pkg/app` contract itself
- NRF registration or metrics-server integration decisions
- repo/package ownership cleanup beyond test-file placement
- callback contract redesign beyond what handler tests need to lock behavior
- replacing every Python helper immediately
- adding a hard CI coverage gate before the repository has an agreed baseline

---

## 10. Acceptance Criteria

This round is complete when all of the following are true:

1. Each `internal/sbi/api_*.go` file has direct handler coverage or is
   explicitly documented as deferred with a concrete reason.
2. Standardized NWDAF SBI consumers are classified and, where local generated
   clients already exist, are migrated away from handwritten raw-HTTP request
   paths.
3. Consumer tests use a deliberate outbound-mocking strategy instead of an
   ad hoc mix of local servers and real client construction.
4. The highest-value handwritten app stubs are replaced by `gomock`-backed
   seams without forcing a Priority 4 boundary rewrite.
5. Low-value tests that only prove symbol existence or rely on fragile timing
   assumptions are removed, replaced, or explicitly justified.
6. `docs/testing.md` clearly separates unit tests from integration helpers.
7. Verification notes report both commands run and the remaining test gaps by
   layer.

---

## 11. Follow-Up After This Round

Once this round is complete, the repository should be in a safer position to
take the next scan priorities:

- Priority 4: rebuild one real app boundary
- Priority 5: normalize SBI error contracts
- Priority 6: clarify post-subscription activation and late-failure signaling

The main reason for doing this round first is that those later changes need a
more trustworthy test seam than the repository has today.
