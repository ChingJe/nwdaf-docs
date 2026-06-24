# NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan

Date: 2026-06-24

Status: Planned

Historical remediation items:

- residual strict-alignment follow-up after `Priority 4 — Rebuild One Real App Boundary`
- narrower architecture and test-seam cleanup after the completed Priority 4
  Phase 1 implementation

---

## 1. Purpose

This plan defines the Phase 2 follow-up after the completed Priority 4 Phase 1
round in `NWDAF/`.

Phase 1 already fixed the largest structural problem:

1. `pkg/app` is now the shared runtime boundary
2. consumer construction is app-driven
3. overlapping local app contracts were collapsed onto one shared root
4. adjacent server lifecycle drift was cleaned up
5. tests and mocks were updated to the new seam

The remaining work is no longer "rebuild one real app boundary". The remaining
work is stricter free5GC alignment of two deliberate Phase 1 exceptions:

1. `CancelContext()` still lives in the root `pkg/app.App` contract even
   though reference free5GC root app interfaces do not expose it
2. `internal/sbi/consumer.NewConsumerWithServices(...)` remains an exported
   production-package constructor used mainly to assemble test fixtures

Phase 2 therefore exists to tighten the boundary further without undoing the
stability gained in Phase 1.

This phase is about:

1. removing `CancelContext()` from the root `pkg/app.App` contract
2. moving cancellation requirements into the smallest layer-specific extension
   interfaces that actually need them
3. converging the consumer package on one exported constructor path,
   `NewConsumer(app)`
4. replacing the exported test-construction seam with stricter free5GC-style
   test patterns based on `gomock`, `gock`, and smaller local interfaces
5. keeping the already completed Phase 1 production behavior unchanged while
   improving structural alignment

This phase is intentionally not about:

- broader lifecycle redesign beyond the `CancelContext()` contract move
- NRF registration or deregistration
- metrics server introduction
- TLS or HTTP/2 helper migration
- runtime config split between core NF config and lab/workflow config
- Daisy or ADRF callback contract redesign
- repository/package ownership relocation for MTLF workflow files

---

## 2. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Rebuild One Real App Boundary Plan.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `resources/references/free5gc-main/NFs/udr/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udm/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udm/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nf_management_test.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/nrf/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/nef/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/consumer/consumer.go`
- the current `NWDAF/` tree after commit `bbf42c3`

---

## 3. free5GC Baseline Survey For The Remaining Gap

### 3.1 Root `pkg/app.App` Usually Stops At `Config()` And `Context()`

The surveyed reference NFs still converge on a small root app contract:

1. log setters
2. `Start()`
3. `Terminate()`
4. `Config()`
5. `Context()`

This is visible in UDR, UDM, and NRF `pkg/app/app.go`.

Implication for NWDAF:

- `CancelContext()` in the root contract is stricter than needed for Phase 1
  stability but broader than the reference baseline
- Phase 2 can now remove it from the root contract without reopening the
  already-solved shared-boundary problem

### 3.2 Concrete App Structs May Still Implement `CancelContext()`

The reference survey also shows that concrete app structs can still own and
expose cancellation context even when root `pkg/app.App` does not require it.

Examples:

1. `UdmApp` implements `CancelContext()` in `pkg/service/init.go`
2. `NefApp` implements `CancelContext()` in `pkg/service/init.go`
3. UDM server code depends on `CancelContext()` through a local extension
   interface rather than through root `pkg/app.App`

Implication for NWDAF:

- Phase 2 should remove `CancelContext()` from root `pkg/app.App`
- Phase 2 should not remove the method from the concrete `NwdafApp`
- instead, packages that truly need cancellation should request it explicitly
  through their local extension interface

### 3.3 Consumer Construction In Reference NFs Uses One Exported Path

The surveyed consumers in UDR, UDM, NRF, and NEF all expose the same
high-level shape:

1. one exported constructor: `NewConsumer(app)`
2. consumer state is initialized from the passed app or internal service setup
3. tests instantiate that same constructor and then control external behavior
   through `gomock`, `gock`, and app/context mocks

No surveyed reference NF exposes an exported production-package constructor
whose primary role is assembling mixed fake service clients for tests.

Implication for NWDAF:

- `NewConsumerWithServices(...)` is structurally useful today, but it is not
  aligned with the stricter free5GC baseline
- Phase 2 should remove or at least de-export that path once tests are migrated
  to more typical free5GC seams

### 3.4 Reference Tests Prefer Mocked App Seams And HTTP Interception

The inspected UDM consumer tests follow the pattern encouraged by the local
skill guidance:

1. instantiate `NewConsumer(mockApp)`
2. mock app methods with `gomock`
3. intercept outbound HTTP with `gock`
4. avoid introducing extra exported test-only constructors when the normal app
   constructor is sufficient

Implication for NWDAF:

- consumer-package tests should increasingly use the standard constructor plus
  app mocks
- processor tests should stop depending on an exported helper that assembles a
  concrete `*consumer.Consumer` from fake sub-services

---

## 4. Current NWDAF State After The Completed Phase 1 Round

### 4.1 The Large Priority 4 Problem Is Solved

After commit `bbf42c3`, NWDAF now has:

1. one canonical `pkg/app.App`
2. app-driven consumer construction via `consumer.NewConsumer(nwdaf)`
3. processor-side ADRF threshold initialization from app-owned config
4. shared root app embedding across server, processor, AnLF, and MTLF seams
5. simplified server lifecycle signatures without unused `traceCtx` arguments

This means Phase 2 does not need to revisit the main Phase 1 direction.

### 4.2 `CancelContext()` Is Still Broader Than The Reference Root Contract

The remaining alignment gap is now narrower:

1. `pkg/app.App` still includes `CancelContext()`
2. AnLF, MTLF, and processor code rely on that root-level inclusion
3. some test helpers and mocks now inherit that larger root interface

This is not a correctness bug. It is a shape mismatch against the stricter
free5GC baseline.

### 4.3 `NewConsumerWithServices(...)` Keeps Processor Tests Coupled To A Concrete Consumer Shape

The current tree still exposes:

1. `internal/sbi/consumer.NewConsumerWithServices(...)`
2. processor tests that call that helper to assemble a concrete
   `*consumer.Consumer` around fake service clients

This leaves Phase 1 with one public constructor path that exists mainly for
test injection rather than for production ownership.

### 4.4 The Remaining Test-Seam Mismatch Is About Ownership, Not Coverage

The current tests are already useful and passing.

The remaining problem is not lack of tests. The remaining problem is that some
tests still prove behavior through a public helper constructor that reference
free5GC NFs normally avoid exposing.

This is why Phase 2 should be handled as a design-tightening round rather than
as a generic test-expansion round.

---

## 5. Fixed Decisions For Phase 2

The following decisions are treated as fixed for this plan:

1. Remove `CancelContext()` from the root `pkg/app.App`.
   This is now safe because Phase 1 already stabilized the shared boundary.
2. Keep `CancelContext()` on the concrete `pkg/service.NwdafApp`.
   Phase 2 is about narrowing the root contract, not eliminating app-owned
   cancellation.
3. Reintroduce cancellation only through local extension interfaces that
   actually need it.
   This should follow the same pattern seen in UDM server code.
4. Converge on a single exported consumer constructor.
   The intended end state is `NewConsumer(app)` as the only public consumer
   construction path.
5. Do not solve the test problem by widening root interfaces again.
   If processor tests need more isolation, add smaller local interfaces instead
   of pushing more methods back into `pkg/app`.
6. Prefer `gomock` and `gock` over exported test-only assembly helpers.
   This matches both the skill guidance and the inspected free5GC NF tests.

---

## 6. Target End State For Phase 2

### 6.1 Root App Contract

At the end of Phase 2, `pkg/app.App` should contain only:

1. `SetLogEnable(bool)`
2. `SetLogLevel(string)`
3. `SetReportCaller(bool)`
4. `Start()`
5. `Terminate()`
6. `Config() *factory.Config`
7. `Context() *nwdaf_context.NWDAFContext`

It should no longer contain `CancelContext()`.

### 6.2 Layer-Specific App Extensions

Packages that need cancellation should request it explicitly through local
extensions such as:

1. `type cancelApp interface { app.App; CancelContext() context.Context }`
2. server boundary: `app.App` plus `Processor()` and, only if still needed,
   `CancelContext()`
3. processor boundary: `cancelApp` plus a consumer-facing seam
4. AnLF boundary: `cancelApp`
5. MTLF boundary: `cancelApp` plus a consumer-facing seam

The exact names can vary, but the structure should keep `CancelContext()` out
of the root contract.

### 6.3 Consumer Construction And Test Seams

At the end of Phase 2:

1. `internal/sbi/consumer` should expose one exported constructor:
   `NewConsumer(app)`
2. `NewConsumerWithServices(...)` should either be removed or reduced to an
   unexported package-local helper that no longer defines the public seam
3. consumer package tests should instantiate the normal constructor with a mock
   app wherever possible
4. processor tests should stop assembling a concrete consumer through that
   exported helper

### 6.4 Narrower Processor-Side Dependency

To make the constructor cleanup practical, processor-side code should move
toward a narrower consumer-facing seam.

Recommended direction:

1. stop depending on the concrete `*consumer.Consumer` type where a smaller
   behavior seam is enough
2. define one or more local interfaces for the operations processor/MTLF
   actually needs, such as SMF subscribe/unsubscribe, MTLF subscribe paths,
   and ADRF access
3. expose explicit consumer methods, not raw fields, when processor needs
   configuration-dependent capabilities such as ADRF presence

This is the cleanest path to removing the exported assembly helper without
making tests more brittle.

---

## 7. Proposed Workstreams

### Workstream A — Remove `CancelContext()` From The Root Contract

1. remove `CancelContext()` from `NWDAF/pkg/app/app.go`
2. keep `NwdafApp.CancelContext()` implemented in `pkg/service`
3. add or restore local extension interfaces in:
   - `internal/anlf`
   - `internal/mtlf`
   - `internal/sbi/processor`
   - `internal/sbi/server` only if it still needs cancellation after review
4. update mocks and test helper types to embed the smaller root app plus local
   cancellation extension where necessary

Expected effect:

- stricter root-app alignment with UDR/UDM/NRF/NEF
- no runtime behavior change

### Workstream B — Redesign The Processor-To-Consumer Test Seam

1. review every production use of `nwdaf.Consumer()` in processor and MTLF code
2. derive the minimum behavior interface those call sites actually need
3. decide whether one shared `ConsumerAPI` or a small set of narrower
   interfaces is cleaner
4. update processor/MTLF app-extension interfaces to return that seam instead
   of a concrete `*consumer.Consumer` where practical
5. add explicit consumer methods for capabilities that are currently accessed
   indirectly through concrete struct shape

Expected effect:

- tests can mock consumer behavior directly
- exported consumer assembly helper is no longer structurally necessary

### Workstream C — Remove Or De-Export `NewConsumerWithServices(...)`

1. migrate consumer-package tests toward `NewConsumer(mockApp)` plus `gock`
   wherever outbound HTTP is the real seam under test
2. migrate processor tests away from helper-assembled concrete consumers
3. once those tests no longer need the public helper, either:
   - delete `NewConsumerWithServices(...)`, or
   - make it unexported and keep it as an internal package helper if truly
     useful for package-local tests

Expected effect:

- public consumer API matches the reference NF baseline
- test injection no longer defines production package shape

### Workstream D — Verification And Safety Nets

1. rerun the full repository verification used in Phase 1
2. add focused tests that prove:
   - root app no longer requires `CancelContext()`
   - local seams still provide cancellation where needed
   - consumer package tests still cover ADRF/config-driven initialization
   - processor tests no longer require the exported assembly constructor

Expected effect:

- structural tightening stays reviewable and mechanically proven

---

## 8. Recommended Implementation Order

1. remove `CancelContext()` from `pkg/app.App`
2. introduce local cancellation extensions where needed
3. update affected mocks and helper types so the tree compiles again
4. derive the minimum processor/MTLF consumer seam
5. migrate processor tests to that seam
6. migrate consumer tests toward `NewConsumer(mockApp)` where possible
7. remove or de-export `NewConsumerWithServices(...)`
8. rerun verification

This order keeps the lower-risk root-contract cleanup separate from the
somewhat broader test-seam redesign.

---

## 9. Acceptance Criteria

Phase 2 should be considered complete only when all of the following are true:

1. `pkg/app.App` no longer includes `CancelContext()`
2. concrete `NwdafApp` still owns cancellation, but local packages request it
   only through explicit extension seams
3. `internal/sbi/consumer` exposes one exported constructor path,
   `NewConsumer(app)`
4. processor/MTLF tests no longer rely on
   `consumer.NewConsumerWithServices(...)`
5. consumer tests continue to cover app-driven ADRF/config initialization
6. all code builds and tests pass after the public helper removal

---

## 10. Verification Expectations

After implementation, run at minimum:

```bash
go test ./...
make build
make lint
```

Recommended focused checks during the round:

```bash
go test ./internal/sbi/consumer ./internal/sbi/processor ./internal/anlf ./internal/mtlf
```

If any future stricter-alignment test depends on live MongoDB, external ML
services, or full callback workflows, that limitation should be reported
explicitly rather than implied away.

---

## 11. Out Of Scope

This phase should not expand into:

- NRF registration/deregistration work
- metrics server ownership
- TLS/H2C server-helper migration
- OpenAPI/model-governance cleanup
- lab/workflow config separation
- Daisy/MTLF callback ownership relocation
- generic package-layout cleanup outside the immediate boundary and test seam

---

## 12. Follow-Up After Phase 2

If Phase 2 completes successfully, the later remaining structural work should
still proceed in the existing broader order:

1. `Priority 6 — Clarify Post-Subscription Activation And Late-Failure Signaling`
2. `Priority 7 — Tighten Logging Boundaries`
3. `Priority 10 — Establish OpenAPI / Model Governance`
4. `Priority 11 — Decide The Intended free5GC Integration Level`
5. `Priority 9 — Separate Runtime Config From Lab / Workflow Config`
6. `Priority 12 — Clean Repo And Package Ownership Boundaries`

The main value of Phase 2 is that these later rounds would then build on a
shared runtime boundary that is not only internally stable, but also shaped
more strictly like the reference free5GC NF baseline.
