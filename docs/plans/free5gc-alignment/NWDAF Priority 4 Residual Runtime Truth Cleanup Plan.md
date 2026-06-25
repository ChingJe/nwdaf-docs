# NWDAF Priority 4 Residual Runtime Truth Cleanup Plan

Date: 2026-06-24

Status: Completed

Historical remediation items:

- residual strict-alignment follow-up after the completed
  `Priority 4 — Rebuild One Real App Boundary`
- narrower follow-up after the completed
  `NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan`

This plan exists because the 2026-06-24 strict reassessment confirmed that the
main Priority 4 work is materially complete, but not yet fully strict in one
important remaining area:

- runtime truth is still split between the app boundary and package-global
  `factory.NwdafConfig` reads in several already-touched runtime paths

This plan is the implementation plan for that residual cleanup.

Related issue record:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

Implementation status:

- implemented in `NWDAF/` on 2026-06-24
- committed as `a912581` in the local `NWDAF/` repository
- removed direct `factory.NwdafConfig` reads from the covered app-owned
  runtime paths in `internal/anlf`, `internal/mtlf`, and
  `internal/sbi/processor`
- tightened the adjacent notifier-owned UE analytics path so it also derives
  runtime config through the app seam
- updated affected tests so the covered app-owned runtime paths receive config
  through test app seams instead of package-global config setup

Verification rerun after implementation:

- `go test ./internal/anlf`
- `go test ./internal/mtlf`
- `go test ./internal/notifier`
- `go test ./internal/sbi/processor`
- `go test ./...`
- `make build`
- `make lint`

Result:

- all focused package tests passed
- full repository test suite passed
- build passed
- lint passed with `0 issues`

Completion judgment:

- this residual Priority 4 strictness gap should now be treated as completed
  for the originally defined scope of this plan
- the remaining repository uses of `factory.NwdafConfig` after this round are
  outside this plan's target and do not keep this specific issue open

---

## 1. Purpose

This plan defines the next implementation round for the already-touched
Priority 4 area in `NWDAF/`.

The purpose of this round is narrow:

1. remove the remaining package-global `factory.NwdafConfig` reads from
   runtime paths that already have access to the owning app or app-derived seam
2. restore one consistent app-owned source of runtime truth across:
   - `internal/anlf`
   - `internal/mtlf`
   - `internal/sbi/processor`
3. tighten adjacent context ownership only where it is required to make the
   runtime-truth cleanup coherent

This round is intentionally not a replay of the whole Priority 4 refactor.

It is about:

- residual app-boundary strictness
- eliminating package-global config drift in already-touched code paths
- making constructor and runtime ownership claims in the current Priority 4
  documentation true in practice

It is not about:

- full repository-wide lifecycle cancellation redesign
- Daisy / ML service consumer ownership redesign as a separate architecture item
- runtime config versus lab/workflow config separation
- OpenAPI/model governance
- NRF registration, metrics, TLS, or broader free5GC integration-level uplift

---

## 2. Why This Round Exists

The completed Priority 4 rounds already achieved the large structural win:

1. `pkg/app` is the shared runtime boundary
2. consumer construction is app-driven for the main `internal/sbi/consumer`
   path
3. the old `CancelContext()` root-contract overreach was removed
4. tests and mocks were updated to the new seam

However, the strict reassessment found that runtime truth is still not fully
owned by the app seam.

The current remaining problem is not "the app boundary is fake again".

The current remaining problem is narrower:

1. some runtime code still reads config through `factory.NwdafConfig`
2. those code paths are already inside parts of the system that now depend on
   the app boundary
3. that leaves the repository with two overlapping runtime-truth channels:
   - the owning app and its local extension seams
   - a package-global config singleton

This residual split is exactly the kind of strictness gap that should be fixed
before moving on to lower-priority or broader-scope architecture work.

---

## 3. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Rebuild One Real App Boundary Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `resources/references/free5gc-main/NFs/udr/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udm/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/smf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/smf/internal/sbi/processor/processor.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/processor/processor.go`
- the current `NWDAF/` tree on 2026-06-24

---

## 4. Confirmed Current Residual Problem

The current NWDAF tree still has package-global config reads in runtime paths
that already have the owning app available either directly or through their
local extension seam.

Confirmed current locations:

1. `internal/anlf/model.go`
   - ML service configuration is read from `factory.NwdafConfig`
2. `internal/anlf/monitor.go`
   - monitor enablement and analytics sampling settings are read from
     `factory.NwdafConfig`
3. `internal/mtlf/training.go`
   - NWDAF callback URL construction, startup scheduling, retrain dispatch, and
     hot-swap guard still read `factory.NwdafConfig`
4. `internal/sbi/processor/data_collection.go`
   - SMF collection and model-provisioning paths still read
     `factory.NwdafConfig`
5. `internal/sbi/processor/upf_notify.go`
   - MongoDB persistence and ring-buffer size still read `factory.NwdafConfig`

This section records the starting state that motivated the plan.
That starting gap has now been removed by the implementation status recorded
above.

---

## 5. free5GC Baseline And Design Direction

The relevant free5GC baseline for this round is not "every repo must have the
same packages".

The relevant baseline is:

1. service constructs the app
2. app is the runtime owner
3. processor and adjacent runtime packages depend on the owning app or a local
   app-derived seam
4. config is not rediscovered from a package-global singleton when the owning
   runtime object is already present

Reference control-plane NFs such as UDR, UDM, and SMF all keep runtime truth
flowing through service/app/context/processor ownership rather than through a
parallel global config channel once the app is already in play.

Implication for NWDAF:

- the remaining `factory.NwdafConfig` runtime reads in app-owned code should be
  treated as residual drift, not as a legitimate long-term boundary

---

## 6. Fixed Decisions For This Round

The following decisions are treated as fixed for this plan:

1. Remove remaining `factory.NwdafConfig` reads from the already-identified
   app-owned runtime paths listed in this plan.
2. Prefer passing or exposing config through the existing app-derived seam
   instead of adding a new package-global helper layer.
3. Do not widen the root `pkg/app.App` again.
   If a runtime path needs more than `Config()` and `Context()`, use a local
   extension seam.
4. Keep the round narrow.
   Do not fold in the broader Class C issue of redesigning Daisy / ML service
   ownership unless it is required to remove a concrete global-config read.
5. Only touch lifecycle context propagation where it is adjacent to and needed
   for the runtime-truth cleanup.
   This is not a full replay of Priority 2.

---

## 7. Target End State

At the end of this round:

1. the already-touched app-owned runtime paths no longer read
   `factory.NwdafConfig` directly
2. AnLF, MTLF, and processor code derive config through the owning app seam or
   a smaller app-derived interface
3. runtime behavior in those paths remains unchanged
4. tests no longer need to seed package-global config merely to exercise those
   app-owned paths
5. the Priority 4 documentation can truthfully describe the app boundary as the
   effective runtime-truth owner for the touched scope

---

## 8. Scope

### 8.1 In Scope

- `NWDAF/internal/anlf/model.go`
- `NWDAF/internal/anlf/monitor.go`
- `NWDAF/internal/anlf/anlf.go`
- `NWDAF/internal/mtlf/training.go`
- `NWDAF/internal/mtlf/mtlf.go`
- `NWDAF/internal/sbi/processor/data_collection.go`
- `NWDAF/internal/sbi/processor/upf_notify.go`
- `NWDAF/internal/sbi/processor/processor.go`
- adjacent test files and mocks that currently depend on package-global config
  for these paths

### 8.2 Conditional In Scope

The following may be touched only if needed to support the runtime-truth
cleanup cleanly:

- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/app/app.go`
- `NWDAF/internal/sbi/consumer/*.go`

### 8.3 Explicit Non-Goals

- `internal/notifier` lifecycle redesign beyond current ownership
- full consumerization of Daisy or ML service as a separate architecture round
- config file splitting
- metrics or NRF lifecycle wiring
- callback contract redesign
- repository/package relocation

---

## 9. Proposed Workstreams

### Workstream A — Define The Minimum Config-Carrying App Seams

Objectives:

- identify the smallest app-derived interfaces needed by AnLF, MTLF, and
  processor code to access config without using `factory.NwdafConfig`

Planned outcomes:

1. review current local interfaces in:
   - `internal/anlf`
   - `internal/mtlf`
   - `internal/sbi/processor`
2. ensure each of those seams exposes `Config()` through `app.App`
3. add only the minimum extra methods beyond `app.App` where the local runtime
   path truly needs them

Acceptance check:

- the runtime path can obtain all needed config through the passed app seam

### Workstream B — Remove Global Config Reads From AnLF

Objectives:

- make AnLF model and monitor behavior depend on app-owned config

Planned outcomes:

1. replace `factory.NwdafConfig` reads in `internal/anlf/model.go`
2. replace `factory.NwdafConfig` reads in `internal/anlf/monitor.go`
3. keep monitoring and model-init behavior unchanged
4. update affected tests so they stop relying on global config setup when the
   app seam is available

Acceptance check:

- AnLF no longer needs package-global config to access ML service, MTLF monitor,
  or analytics settings in the covered paths

### Workstream C — Remove Global Config Reads From MTLF

Objectives:

- make MTLF callback URL construction and retrain startup behavior depend on
  app-owned config

Planned outcomes:

1. replace `factory.NwdafConfig` reads in `internal/mtlf/training.go`
2. keep callback URL, trigger scheduling, and retrain dispatch behavior
   unchanged
3. update tests so they prove the same behavior through app-provided config
   instead of package-global config

Acceptance check:

- the covered MTLF runtime path no longer consults package-global config

### Workstream D — Remove Global Config Reads From Processor Runtime Paths

Objectives:

- finish the same cleanup in already-touched processor-owned runtime behavior

Planned outcomes:

1. replace `factory.NwdafConfig` reads in
   `internal/sbi/processor/data_collection.go`
2. replace `factory.NwdafConfig` reads in
   `internal/sbi/processor/upf_notify.go`
3. preserve current SMF, MTLF, ADRF, MongoDB, and ring-buffer behavior
4. keep the processor seam explicit rather than introducing new hidden globals

Acceptance check:

- processor-owned runtime paths derive config from the owning app seam or a
  direct app-owned dependency

### Workstream E — Tighten Adjacent Test Ownership

Objectives:

- ensure the residual cleanup actually reduces test dependence on package-global
  config

Planned outcomes:

1. rewrite affected tests that currently seed `factory.NwdafConfig` only to
   support app-owned runtime code
2. keep package-global setup only where the production boundary truly remains
   package-global after this round
3. update gomock seams if local interfaces change

Acceptance check:

- app-owned runtime tests should primarily express config through the app seam

---

## 10. Proposed Implementation Order

1. define or tighten the minimum app-derived interfaces needed by AnLF, MTLF,
   and processor
2. remove the remaining global reads from AnLF
3. remove the remaining global reads from MTLF
4. remove the remaining global reads from processor runtime paths
5. rewrite or tighten affected tests and mocks
6. rerun focused verification
7. rerun full repository verification

Reason for this order:

- interface tightening should happen before the internal runtime paths are
  rewritten
- AnLF and MTLF are narrower and easier to stabilize before the processor
  runtime paths
- processor cleanup is safest after the shared config-carrying seam is already
  proven in the smaller domains

---

## 11. Acceptance Criteria

This round is complete only if all of the following are true:

1. the covered AnLF, MTLF, and processor runtime paths no longer read
   `factory.NwdafConfig` directly
2. those paths instead derive runtime settings from the owning app seam or a
   small app-derived local extension
3. the production behavior of the covered paths remains unchanged
4. affected tests are updated to reflect app-owned runtime truth where
   practical
5. verification passes with:
   - `go test ./...`
   - `make build`
   - `make lint`

---

## 12. Verification Plan

Focused checks during implementation:

- `go test ./internal/anlf`
- `go test ./internal/mtlf`
- `go test ./internal/sbi/processor`
- `go test ./internal/sbi/consumer`

Full repository checks after the round:

- `go test ./...`
- `make build`
- `make lint`

Verification notes to report:

1. which tests stopped requiring package-global config setup
2. whether any residual `factory.NwdafConfig` reads still remain intentionally
   after this round, and why
3. whether any adjacent lifecycle-context cleanup was included or intentionally
   deferred

---

## 13. Follow-Up After This Round

With this residual Priority 4 cleanup now completed, the next choices become
clearer:

1. continue with existing not-started priorities:
   - Priority 6
   - Priority 7
   - Priority 9
   - Priority 10
   - Priority 11
   - Priority 12
2. or continue with the now-recorded adjacent follow-up for:
   - external-client ownership policy for Daisy / ML service style integrations

The main benefit of completing this round first is that it closes the most
important remaining gap in the already-touched boundary work before the project
moves on to unrelated later cleanups.
