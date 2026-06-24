# NWDAF Priority 3 Strict Test Ownership Alignment Plan

Date: 2026-06-24

Status: Planned

Historical remediation items:

- stricter structural follow-up after the completed
  `Priority 3 — Build The Test Safety Net Around The Real Boundaries`
- narrower ownership-cleanup round after the completed
  `NWDAF Priority 3 Test Refactor Completion Summary`

This plan exists because the 2026-06-24 strict reassessment confirmed that
Priority 3 achieved its main purpose, but still left one narrower strictness
gap inside the same test-alignment lineage:

- shared app-boundary test ownership is still more package-local and ad hoc
  than the surveyed free5GC baseline

This plan is the implementation plan for that narrower cleanup.

Related issue record:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Strict free5GC Alignment Reassessment 2026-06-24.md`

---

## 1. Purpose

This plan defines the next strict free5GC-alignment round for test ownership in
the main `NWDAF/` repository.

The purpose of this round is narrow:

1. move app-boundary test ownership closer to the surveyed free5GC baseline
2. stop making `internal/sbi` and `internal/sbi/processor` own their own
   app-level mock shape
3. preserve package-local mocks only where the seam is truly SBI-specific or
   processor-specific

This round is not a replay of Priority 3 core coverage work.

It is about:

- shared mock ownership
- seam placement
- generator/source placement
- tighter free5GC-style test conventions

It is not about:

- rebuilding the runtime app boundary again
- changing runtime lifecycle behavior
- redesigning Daisy, ML service, or ADRF ownership
- switching consumer transport strategy
- broad repository-wide test-style cleanup outside the affected seams

---

## 2. Why This Round Exists

Priority 3 was the correct earlier move.

It added the missing safety net:

1. direct handler tests under `internal/sbi/api_*_test.go`
2. `gomock` at important internal seams
3. `gock` for outbound consumer tests
4. stronger processor and notifier coverage

That work should still be treated as completed.

However, the stricter same-day reassessment found a narrower structural gap:

1. handler tests still depend on a package-local fake app
2. `internal/sbi` still owns a generated local app mock
3. `internal/sbi/processor` still owns its own local app mock shape
4. the repository therefore still describes the canonical app boundary
   differently at multiple test locations

This does not mean Priority 3 was wrong.

It means Priority 3 solved the coverage problem first, and left a later
ownership cleanup that is more about strictness than about missing safety.

---

## 3. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Strict free5GC Alignment Reassessment 2026-06-24.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Test Refactor Detailed Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Rebuild One Real App Boundary Plan.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/testing.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `resources/references/free5gc-main/NFs/smf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/smf/pkg/service/mock.go`
- `resources/references/free5gc-main/NFs/amf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/amf/pkg/service/mock.go`
- `resources/references/free5gc-main/NFs/udm/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udm/pkg/app/mock.go`
- `resources/references/free5gc-main/NFs/udm/pkg/mockapp/mock.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/processor/generate_auth_data_test.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nf_management_test.go`
- `resources/references/free5gc-main/NFs/chf/internal/sbi/consumer/nrf_service_test.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/server_mock.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/api_sanity_test.go`
- the current `NWDAF/` tree on 2026-06-24

---

## 4. Confirmed Current Problem

The current NWDAF tree still keeps app-level test ownership too local.

Confirmed current locations:

1. `NWDAF/internal/sbi/api_test_helpers_test.go`
   - handler tests use a package-local fake app
2. `NWDAF/internal/sbi/mock_interfaces_test.go`
   - handler-side generated mocks still include a local app mock shape
3. `NWDAF/internal/sbi/processor/mock_interfaces_test.go`
   - processor tests still own a separate local app mock shape

This is the exact issue described in the strict reassessment.

Important clarification:

1. the mere existence of `internal/sbi/mock_interfaces_test.go` is not, by
   itself, proof of bad alignment
2. free5GC reference NFs also sometimes keep SBI-local mocks
3. the real issue is that app-boundary mock ownership has not yet been
   consolidated behind the now-canonical NWDAF app/service seam

---

## 5. free5GC Baseline And Design Direction

### 5.1 Shared App Mock Ownership In Surveyed NFs

The surveyed control-plane NFs do not use one universal placement rule, but
they do share one important pattern:

1. reusable app-boundary mocks are owned by a shared package
2. that shared package is usually `pkg/app` or `pkg/service`
3. package-local SBI mocks are kept only for SBI-specific seams

Representative local references:

1. shared app ownership under `pkg/service`
   - `resources/references/free5gc-main/NFs/smf/pkg/service/mock.go`
   - `resources/references/free5gc-main/NFs/amf/pkg/service/mock.go`
2. shared app ownership under `pkg/app`
   - `resources/references/free5gc-main/NFs/udm/pkg/app/mock.go`
3. separate richer shared mock package when tests need a wider seam
   - `resources/references/free5gc-main/NFs/udm/pkg/mockapp/mock.go`
4. local SBI mock kept for an SBI-specific seam
   - `resources/references/free5gc-main/NFs/udr/internal/sbi/server_mock.go`

### 5.2 Best-Fit Baseline For NWDAF

For NWDAF, the closest free5GC-style target is not `pkg/app`.

The best fit is a shared service-owned test seam under `pkg/service` because:

1. the real runtime owner is already `pkg/service.NwdafApp`
2. the affected tests need more than the narrow root `pkg/app.App`
3. the affected seams currently depend on:
   - `Config()`
   - `Context()`
   - `CancelContext()`
   - `Consumer()`
   - `Processor()`
4. this is closer to the SMF and AMF pattern than to widening `pkg/app` again
5. it avoids introducing a second shared package such as `pkg/mockapp` unless
   the simpler `pkg/service` baseline proves insufficient

### 5.3 Testing Tool Expectations From `free5gc-dev-skill`

The implementation in this round should continue to follow the local skill
guidance:

1. internal dependency mocks should use `go.uber.org/mock/gomock`
2. handler-level HTTP boundary tests should use:
   - `httptest.NewRecorder()`
   - `gin.CreateTestContext(...)`
3. outbound consumer tests should keep using `gock`
4. `openapi.InterceptH2CClient()` and `openapi.RestoreH2CClient()` should only
   be used where the free5GC/OpenAPI client path actually needs H2C
   interception
5. this round should not introduce a new mocking framework
6. this round should not switch assertion style merely for aesthetics

---

## 6. Fixed Decisions For This Round

The following decisions are fixed for this plan:

1. treat this as a strictness follow-up after completed Priority 3, not as a
   reopened failure of Priority 3 core boundary coverage
2. introduce one shared richer app test seam under `NWDAF/pkg/service`
3. keep the root `pkg/app.App` narrow
   - do not widen it again solely to make tests easier
4. use `pkg/service` rather than a new `pkg/mockapp` package unless the
   implementation proves that `pkg/service` cannot cleanly host the seam
5. preserve local mocks only where the seam is truly local
   - example: `processorAPI` under `internal/sbi`
   - example: processor-local consumer/client mocks under
     `internal/sbi/processor`
6. remove package-local fake app ownership from handler tests
7. keep the production runtime boundary stable unless a minimal supporting
   signature or interface extraction is required to support the shared mock
8. follow existing free5GC-style `mockgen` usage
   - source the shared mock from a real shared interface
   - do not maintain multiple hand-diverged app mock shapes

---

## 7. Target End State

At the end of this round:

1. `pkg/service` owns the canonical shared app-boundary test mock for the
   richer NWDAF seam
2. `internal/sbi` handler tests no longer define or own a package-local fake
   app
3. `internal/sbi/processor` tests no longer own their own app mock shape
4. handler and processor tests use the same shared app-level test seam
5. true SBI-local mocks remain local
6. the repository no longer has multiple separate app-level mock owners for the
   same runtime boundary

---

## 8. Scope

### 8.1 In Scope

- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/service/mock.go`
- `NWDAF/internal/sbi/api_test_helpers_test.go`
- `NWDAF/internal/sbi/mock_interfaces_test.go`
- `NWDAF/internal/sbi/api_*_test.go` files that currently rely on the handler
  helper app seam
- `NWDAF/internal/sbi/processor/mock_interfaces_test.go`
- `NWDAF/internal/sbi/processor/*_test.go` files that currently use the local
  app mock
- minimal generator/source-file adjustments required to keep only the correct
  local mocks local

### 8.2 Conditionally In Scope

The following may be touched only if needed to support the ownership cleanup
cleanly:

- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/sbi/processor/processor.go`
- small new files that isolate local test-seam interfaces for `mockgen`

### 8.3 Out Of Scope

- redesigning the production app boundary again
- moving consumer tests to a different transport strategy
- replacing all local mocks in the repository
- refactoring `internal/anlf`, `internal/mtlf`, or notifier tests unrelated to
  app-mock ownership
- changing Daisy / ML service / ADRF ownership policy
- rewriting tests to a new assertion framework

---

## 9. Proposed Design

### 9.1 Add A Shared Service-Owned Test Seam

Add a shared richer interface in `pkg/service` that matches the current real
test ownership needs without widening `pkg/app.App`.

Planned direction:

1. define a shared service-level interface close to the real runtime shape
2. keep `NwdafApp` as the concrete owner
3. generate the mock from that shared interface using `mockgen`

The intended interface shape is:

1. embed `app.App`
2. expose `CancelContext() context.Context`
3. expose `Consumer() consumer.ConsumerAPI`
4. expose `Processor() *processor.Processor`

This is intentionally close to the AMF/SMF service-level mock ownership model.

### 9.2 Keep `pkg/app.App` Narrow

Do not reverse the earlier Priority 4 correction by widening `pkg/app.App`
again.

Rationale:

1. Priority 4 intentionally narrowed the root contract
2. this plan is about test ownership, not about re-expanding runtime
   dependencies
3. free5GC references commonly use a narrower root app contract plus a richer
   service-owned or test-owned seam where needed

### 9.3 Move Handler Tests To The Shared Service Mock

Handler tests under `internal/sbi` should stop owning a package-local fake app.

Planned change:

1. replace `handlerTestApp` in `internal/sbi/api_test_helpers_test.go`
2. build handler test servers from the shared `pkg/service` mock
3. keep `processorAPI` as the handler-local mock seam

This preserves the correct handler boundary:

- app ownership is shared
- processor ownership is local to the handler package

### 9.4 Move Processor Tests To The Shared Service Mock

Processor tests should stop owning a separate local app mock shape.

Planned change:

1. replace `MockNwdafApp` usage in `internal/sbi/processor/*_test.go`
2. build processor tests with the shared `pkg/service` mock
3. keep processor-local mocks only for:
   - `consumer.ConsumerAPI`
   - local service-client seams such as SMF-specific client helpers

This preserves the correct processor boundary:

- app ownership is shared
- consumer/client helper ownership remains local where appropriate

### 9.5 Separate Shared Ownership From Truly Local Mock Ownership

This round should not try to delete every local mock.

The intended split is:

1. shared app-boundary mock
   - owned by `pkg/service`
2. local SBI-specific seam mock
   - owned by `internal/sbi`
3. local processor-side consumer/client seam mocks
   - owned by `internal/sbi/processor`

This is the closest match to the surveyed free5GC baseline.

### 9.6 Clean Up `mockgen` Source Placement

The current generator/source layout should be cleaned up enough that app-level
mock ownership is not accidentally reintroduced under internal packages.

Planned generator direction:

1. generate the shared app mock from `pkg/service`
   - expected command shape:
     `mockgen -package=service -source=pkg/service/init.go -destination=pkg/service/mock.go`
2. stop sourcing handler-local generated mocks from a file that forces the
   app-level mock to be emitted again
3. if needed, isolate truly local seam interfaces into smaller source files so
   generated output only contains the local seam that should remain local

This part is important.

Without it, the repository can easily migrate test usage while still preserving
the same structural ownership drift in generated output.

---

## 10. Implementation Sequence

### Phase 1: Shared Service Mock Introduction

1. add the shared richer test seam in `pkg/service`
2. generate `pkg/service/mock.go`
3. confirm the concrete `NwdafApp` still satisfies the intended service-level
   interface

### Phase 2: Handler Test Ownership Migration

1. update `internal/sbi/api_test_helpers_test.go`
2. switch all handler tests to the shared service mock
3. preserve `processorAPI` as the handler-local seam

### Phase 3: Processor Test Ownership Migration

1. update processor test helpers to use the shared service mock
2. preserve local consumer/client mocks only where they are genuinely local
3. remove the local processor-side app mock owner

### Phase 4: Generator And Residual Cleanup

1. prune or split generated/local mock files so app-level mock ownership no
   longer lives under `internal/sbi` or `internal/sbi/processor`
2. rerun focused package tests
3. rerun full repository verification

---

## 11. Verification Plan

This round should follow the verification shape required by
`free5gc-dev-skill/references/testing.md`.

Minimum focused verification:

- `go test ./internal/sbi`
- `go test ./internal/sbi/processor`

Recommended repository verification:

- `go test ./internal/sbi/consumer`
- `go test ./...`
- `make build`
- `make lint`

Verification expectations:

1. handler tests should still exercise the HTTP boundary with
   `httptest.NewRecorder()` and `gin.CreateTestContext(...)`
2. processor tests should still use `gomock` for the app seam
3. consumer tests touched in the same round, if any, should keep their current
   `gock` pattern
4. `openapi.InterceptH2CClient()` should only be used where the touched path
   actually goes through the free5GC/OpenAPI H2C client flow

This round remains unit-level and module-level verification only.

It is not integration proof.

---

## 12. Review Checks

Before considering this round complete, re-check the following:

1. does `pkg/service` now own the shared richer app mock?
2. does any handler test still define its own fake app?
3. does `internal/sbi/processor` still own a separate app mock shape?
4. are remaining local mocks clearly local in responsibility?
5. does any generator source still accidentally re-emit an internal app-level
   mock owner?
6. were any production interfaces widened only for test convenience?
7. did the round keep the actual runtime boundary unchanged?

---

## 13. Completion Criteria

This plan is complete only when all of the following are true:

1. a shared richer app-level test seam exists in `pkg/service`
2. handler tests use that shared seam instead of a package-local fake app
3. processor tests use that shared seam instead of a local app mock owner
4. remaining internal local mocks are limited to genuinely local seams
5. verification passes with at least:
   - `go test ./internal/sbi`
   - `go test ./internal/sbi/processor`
   - `go test ./...`
   - `make build`
   - `make lint`
6. the repository is materially closer to the surveyed free5GC pattern of:
   - shared app-boundary mock ownership in `pkg/service` or another shared
     package
   - local SBI mocks only where the seam is actually local

---

## 14. Expected Outcome

If this round completes successfully, NWDAF will keep the actual gains from
Priority 3 while removing the remaining structural drift in how the project
owns app-boundary test seams.

That is the intended next step if the project wants to move from
"tests now exist at the right boundaries" to
"those boundaries are also owned in a shape that looks like free5GC rather
than like local ad hoc test assembly".
