# NWDAF Priority 4 Rebuild One Real App Boundary Plan

Date: 2026-06-24

Status: Completed across Phase 1 and Phase 2

Historical remediation items:

- `Priority 4 — Rebuild One Real App Boundary`
- residual cleanup from `Priority 2 — Put Long-Running Work Under App Lifecycle Control`

This document is now the canonical record of the completed Priority 4 Phase 1
round.

Phase 1 implementation status:

- implemented in `NWDAF/` on 2026-06-24
- committed as `bbf42c3` in the local `NWDAF/` repository
- completed the boundary reconstruction and adjacent lifecycle cleanup defined
  below

The stricter free5GC follow-up that remains after this completed round is now
tracked separately in:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan.md`

Phase 2 completion status:

- implemented in `NWDAF/` on 2026-06-24
- committed as `6569e02` in the local `NWDAF/` repository
- removed `CancelContext()` from the root `pkg/app.App` contract
- narrowed processor-side consumer ownership to a mockable consumer seam
- removed the exported consumer test-assembly path from the public package API

---

## 1. Purpose

This plan defines the next free5GC-alignment refactor round for the main
`NWDAF/` repository, limited to rebuilding the app boundary that should connect
`pkg/service`, `pkg/app`, `internal/sbi`, `internal/anlf`, and `internal/mtlf`.

The purpose of this round is to stop NWDAF from carrying multiple overlapping
"app-like" contracts and one global-config-driven consumer construction path.

This round is about:

1. defining one canonical `pkg/app` contract that reflects the runtime boundary
   NWDAF actually uses
2. converting consumer construction to an app-driven form instead of relying on
   `factory.NwdafConfig`
3. collapsing duplicated local app interfaces so packages extend one shared
   contract instead of re-declaring common methods
4. absorbing the remaining small lifecycle cleanup that naturally falls out of
   this refactor, especially the unused `traceCtx` path and server shutdown
   ownership drift
5. updating tests and mocks immediately after the contract is stabilized

This round is intentionally not a broader architectural expansion.

It is not about:

- adding NRF registration or deregistration
- adding a metrics server
- adding TLS or HTTP/2 helper migration
- redesigning late activation or `failEventReports` semantics
- OpenAPI/model governance for standardized versus handwritten payloads
- splitting runtime config from lab or workflow config
- moving Daisy/MTLF callback ownership out of `internal/sbi`

---

## 2. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `resources/references/free5gc-main/NFs/udr/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udm/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udm/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/nrf/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/nef/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/nef/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/consumer/consumer.go`
- the current `NWDAF/` tree on 2026-06-24

---

## 3. free5GC Baseline Survey

### 3.1 Shared `pkg/app` Contract Pattern In Other NFs

The surveyed control-plane NFs converge on the same broad shape:

1. `pkg/app.App` is the root service-owned contract.
2. It includes log toggles plus `Start()`, `Terminate()`, `Context()`, and
   `Config()`.
3. The concrete app struct in `pkg/service` implements that interface and owns
   runtime lifecycle, server construction, consumer construction, and shutdown.

This pattern is present in UDR, UDM, NRF, and NEF.

Implication for NWDAF:

- NWDAF should stop treating `pkg/app` as a shallow log/config shell.
- The root app contract should describe the runtime boundary that service,
  server, processor, and consumer actually share.

### 3.2 Consumer Construction Pattern In Other NFs

The surveyed NFs consistently construct consumers from the app boundary:

1. `consumer.NewConsumer(app)` is called from `pkg/service.NewApp(...)`.
2. Consumer packages define a small alias such as `type ConsumerUdm interface {
   app.App }`.
3. Consumer services then read config, context, or peer-NF details through the
   embedded app boundary rather than global package state.

This pattern is visible in UDR, UDM, NRF, and NEF.

Implication for NWDAF:

- `internal/sbi/consumer.NewConsumer()` should stop discovering ADRF and other
  configuration from `factory.NwdafConfig`.
- NWDAF should pass the owning app into consumer construction the same way the
  reference NFs do.

### 3.3 Layer-Specific Extension Pattern

The surveyed NFs do not force one giant app interface for every package.
Instead, they use:

1. one canonical `pkg/app.App`
2. small layer-specific extension interfaces only where an extra typed
   dependency is needed

Typical examples:

- server boundary: `app.App` plus `Processor()`
- consumer boundary: `app.App`
- processor boundary: a local seam that can depend on `app.App` plus NF-local
  helpers such as `Consumer()`

Implication for NWDAF:

- Priority 4 should not introduce a "mega interface" that exposes every method
  to every package.
- The correct target is one canonical root contract plus a small number of
  extension seams that embed it.

### 3.4 Lifecycle Ownership Pattern

The surveyed service packages keep runtime ownership in `pkg/service`:

1. the app struct owns `context.Context`, `cancel`, and `sync.WaitGroup`
2. startup calls server run methods from service
3. termination stops servers and other long-running work from service
4. goroutine ownership is explicit at the app boundary

Some reference code still uses timeout-scoped `context.Background()` for
shutdown internals, but it does not expose unused lifecycle parameters that are
ignored by the callee.

Implication for NWDAF:

- the remaining lifecycle cleanup for this round is narrow
- the server API should either use a passed context meaningfully or stop
  pretending it accepts one
- service should remain the single owner of startup and shutdown orchestration

---

## 4. Current NWDAF State

### 4.1 Boundary Fragmentation Confirmed In Code

The current `NWDAF/` tree still has the exact fragmentation called out by the
project scan:

1. `pkg/app.App` exposes only `Config()`, log setters, and `Terminate()`.
2. `pkg/service.NwdafApp` implements additional methods that the official app
   interface does not acknowledge:
   - `Start()`
   - `Context()`
   - `CancelContext()`
   - `Consumer()`
   - `Processor()`
3. `internal/sbi/server.go`, `internal/sbi/processor/processor.go`,
   `internal/anlf/anlf.go`, and `internal/mtlf/mtlf.go` all define their own
   local `NwdafApp` variants.
4. `internal/sbi/consumer.NewConsumer()` still reads runtime configuration from
   `factory.NwdafConfig` instead of the owning app.

This means NWDAF has one concrete runtime object but several competing ideas of
what that object is.

### 4.2 Adjacent Cleanup That Fits This Same Round

Two small remaining lifecycle issues naturally belong in the same refactor:

1. `internal/sbi/server.Run(traceCtx, wg)` ignores `traceCtx`.
2. `internal/sbi/server.Shutdown(traceCtx)` ignores `traceCtx` and creates its
   own timeout context anyway.

These are no longer standalone Priority 2 blockers, but they are exactly the
kind of lifecycle drift that should be cleaned up while rebuilding the service
to server boundary.

### 4.3 Additional Global-State Drift Near The App Boundary

The global-config problem is not limited to `consumer.NewConsumer()`:

1. `internal/sbi/consumer` enables ADRF by reading `factory.NwdafConfig`.
2. `internal/sbi/processor.NewProcessor()` derives the ADRF threshold from
   `factory.NwdafConfig` instead of the owning app config.

This is important because the app-boundary refactor should not stop halfway at
method signatures while leaving the same runtime truth split between service and
package-level globals.

### 4.4 Constraints From Current NWDAF Architecture

This round should respect the current NWDAF scope:

- NWDAF still runs as a standalone HTTP SBI service
- `nrfUri` remains intentionally inactive
- metrics server wiring is still deferred
- MTLF and AnLF already depend on app cancellation and should keep doing so
- current tests already cover the main handler, processor, consumer, and
  notifier seams and should be updated rather than replaced

These constraints mean the correct change is a boundary reconstruction, not a
whole-service redesign.

---

## 5. Fixed Decisions For This Round

The following decisions are treated as settled for this plan:

1. Keep one canonical root contract in `pkg/app`.
   Priority 4 is successful only if common runtime methods stop being
   re-declared across multiple packages.
2. Do not put `Consumer()` and `Processor()` into the root `pkg/app.App`
   unless a later code-reading pass proves it is unavoidable.
   Reference NFs normally keep the root app contract smaller and use local
   extension interfaces where needed.
3. Add `CancelContext()` to the root NWDAF app contract.
   This is a real runtime dependency already used by AnLF, MTLF, notifier
   ownership, and processor paths, so hiding it outside the shared contract is
   no longer defensible.
4. Convert consumer construction to app-driven ownership.
   New consumer code should derive config and context from the app boundary
   rather than from `factory.NwdafConfig`.
5. Remove or simplify lifecycle parameters that are currently ignored.
   Do not preserve `traceCtx` merely for appearance if the implementation does
   not use it.
6. Absorb only the narrow Priority 2 leftovers that are adjacent to the app
   boundary.
   Do not expand this round into a generic shutdown audit of all network calls.

---

## 6. Target End State For This Round

### 6.1 Canonical Root App Contract

At the end of this round, `pkg/app.App` should be the single canonical root
contract for NWDAF runtime ownership.

Minimum target methods:

1. `SetLogEnable(bool)`
2. `SetLogLevel(string)`
3. `SetReportCaller(bool)`
4. `Start()`
5. `Terminate()`
6. `Config() *factory.Config`
7. `Context() *nwdaf_context.NWDAFContext`
8. `CancelContext() context.Context`

Rationale:

- this matches the actual NWDAF runtime methods already implemented in
  `pkg/service`
- it aligns more closely with free5GC app baselines
- it gives AnLF, MTLF, server, and consumer code one stable shared root

### 6.2 Narrow Extension Interfaces Where Needed

Packages that need extra typed dependencies should extend `app.App` instead of
re-declaring common methods.

Planned examples:

1. server seam:
   - `app.App`
   - `Processor() *processor.Processor`
2. processor seam:
   - `app.App`
   - `Consumer() *consumer.Consumer`
3. consumer seam:
   - usually `app.App` only

This keeps package seams explicit without re-fragmenting the common contract.

### 6.3 App-Driven Consumer Construction

At the end of this round:

1. `pkg/service.NewApp(...)` should construct the consumer by passing the app
   boundary in.
2. `internal/sbi/consumer.NewConsumer(...)` should derive ADRF enablement and
   URLs from the passed app config.
3. processor-side ADRF threshold decisions should also use app-owned config
   instead of `factory.NwdafConfig`.

This change is required for the app boundary to become real rather than merely
better named.

### 6.4 Lifecycle Contract Cleanup

At the end of this round:

1. server run and shutdown methods should no longer expose ignored context
   parameters
2. service should remain the only owner of server startup and termination
3. goroutine ownership should still be visible through the existing app
   waitgroup path

This is intentionally a narrow cleanup, not a full runtime redesign.

---

## 7. Scope

### 7.1 In Scope

- `NWDAF/pkg/app/app.go`
- `NWDAF/pkg/service/init.go`
- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/sbi/consumer/*.go`
- `NWDAF/internal/sbi/processor/processor.go`
- `NWDAF/internal/anlf/*.go` where the local app seam is declared
- `NWDAF/internal/mtlf/*.go` where the local app seam is declared
- affected test files and generated mocks that describe these seams

### 7.2 Explicit Non-Goals

- NRF registration or deregistration
- metrics server introduction
- TLS/H2C migration to free5GC utility server helpers
- callback contract redesign for Daisy or ADRF
- model-governance cleanup for handwritten standardized payloads
- config split between deployable NF settings and local workflow settings
- repository asset relocation or package ownership cleanup outside the app seam

---

## 8. Proposed Workstreams

### Workstream A - Define The Canonical App Contract

Objectives:

- expand `pkg/app.App` to match the real NWDAF runtime boundary
- replace re-declared common methods with one canonical contract

Planned outcomes:

1. add `Start()`, `Context()`, and `CancelContext()` to `pkg/app.App`
2. import the NWDAF internal context type into `pkg/app` the same way
   reference NFs do
3. keep log setters, `Terminate()`, and `Config()` on the root interface
4. leave `Consumer()` and `Processor()` off the root interface for now

Acceptance check for this workstream:

- no package should still re-declare `Config()`, `Context()`, or
  `CancelContext()` if it already depends on `app.App`

### Workstream B - Replace Fragmented Local App Interfaces With Extensions

Objectives:

- stop each package from inventing its own root app contract
- keep extra typed dependencies explicit and local

Planned outcomes:

1. `internal/sbi/server.go` uses a local interface that embeds `app.App` and
   adds `Processor()`
2. `internal/sbi/processor/processor.go` uses a local interface that embeds
   `app.App` and adds `Consumer()`
3. `internal/anlf` uses `app.App` directly or a trivial alias of it
4. `internal/mtlf` uses an interface that embeds `app.App` and adds
   `Consumer()`

Important rule:

- this round should remove duplicated common-method declarations, not merely
  rename them

### Workstream C - Convert Consumer Construction To App Ownership

Objectives:

- make the consumer lifecycle and config ownership match free5GC NF patterns
- remove `factory.NwdafConfig` as the hidden consumer constructor input

Planned outcomes:

1. change `consumer.NewConsumer()` to `consumer.NewConsumer(app)` or an
   equivalent small app-derived interface
2. derive ADRF enablement and URL from `app.Config()`
3. keep `NewConsumerWithServices(...)` available for tests where it still
   provides value
4. re-check whether `Consumer.Context()` remains useful once app ownership is
   explicit

Important follow-through:

- this workstream must also remove adjacent processor constructor reads of
  `factory.NwdafConfig` where the owning app config is already available

### Workstream D - Absorb The Remaining Narrow Lifecycle Cleanup

Objectives:

- clean up the misleading server lifecycle API while touching the same boundary

Planned outcomes:

1. remove unused `traceCtx` parameters from `Run(...)` and `Shutdown(...)`, or
   make them meaningfully control server lifecycle
2. keep service as the single owner of server start and stop
3. ensure app termination still stops active subscription schedulers before SBI
   shutdown
4. avoid introducing any new long-lived goroutine outside the existing
   waitgroup pattern

This workstream is intentionally small.
It exists to finish the nearby Priority 2 drift that the refactor will already
touch.

### Workstream E - Update Tests And Mocks Immediately

Objectives:

- keep the newly stabilized boundary mechanically protected
- align mock seams with the reconstructed contract in the same change

Planned outcomes:

1. regenerate or rewrite gomock seams that depend on the old fragmented
   interfaces
2. update consumer tests for the new constructor shape
3. update processor tests if constructor or app mock methods change
4. add focused lifecycle coverage if server run or shutdown signatures change
5. keep `go test ./...` green without relying on package-global factory setup
   in tests that should now be app-driven

---

## 9. Proposed Implementation Order

1. expand `pkg/app.App` and lock the target interface shape first
2. convert `pkg/service.NwdafApp` and all compile-time interface assertions
3. change `internal/sbi/consumer.NewConsumer(...)` to app-driven construction
4. remove `factory.NwdafConfig` reads from adjacent consumer and processor
   constructor paths
5. collapse local app interfaces in `server`, `processor`, `anlf`, and `mtlf`
   onto `app.App`-based extensions
6. simplify or correct server run and shutdown lifecycle signatures
7. update mocks and tests immediately after interface stabilization
8. rerun focused and full verification

Reason for this order:

- the root contract must stabilize before mocks or constructors are rewritten
- consumer construction should move before processor and server seams are fully
  cleaned up, because it removes the largest hidden global dependency
- lifecycle signature cleanup is safest after the service and server boundary is
  already explicit

---

## 10. Acceptance Criteria

This round is complete only if all of the following are true:

1. `pkg/app.App` becomes the canonical NWDAF root contract and includes the
   runtime methods that are actually shared today.
2. No production package still re-declares `Config()`, `Context()`, and
   `CancelContext()` as part of a fake root app contract when it could embed
   `app.App` instead.
3. `internal/sbi/consumer.NewConsumer(...)` no longer depends on
   `factory.NwdafConfig`.
4. Adjacent processor constructor logic no longer depends on
   `factory.NwdafConfig` where the owning app config is already available.
5. Server lifecycle methods no longer expose ignored context parameters.
6. Existing notifier and MTLF lifecycle behavior remains under app-owned
   cancellation and waitgroup control.
7. Updated tests cover the new constructor and interface seams.
8. Verification passes with:
   - `go test ./...`
   - `make build`
   - `make lint`

---

## 11. Verification Plan

Focused checks during implementation:

- `go test ./internal/sbi/consumer`
- `go test ./internal/sbi/processor`
- `go test ./internal/anlf`
- `go test ./internal/mtlf`
- `go test ./internal/sbi`

Full repository checks after the refactor:

- `go test ./...`
- `make build`
- `make lint`

Verification notes to report:

1. whether any tests needed mock regeneration or manual rewrite
2. whether any app-seam tests still require package-global config setup
3. whether lifecycle validation remained unit-level only or included any manual
   startup or shutdown check

---

## 12. Follow-Up After This Round

This plan intentionally leaves the following items for later rounds:

1. `Priority 6 — Clarify Post-Subscription Activation And Late-Failure Signaling`
2. `Priority 7 — Tighten Logging Boundaries`
3. `Priority 10 — Establish OpenAPI / Model Governance`
4. `Priority 11 — Decide The Intended free5GC Integration Level`
5. `Priority 9 — Separate Runtime Config From Lab / Workflow Config`
6. `Priority 12 — Clean Repo And Package Ownership Boundaries`

The main benefit of completing Priority 4 first is that each of these later
rounds can then build on one real, testable, app-owned runtime boundary instead
of several partially overlapping local contracts.
