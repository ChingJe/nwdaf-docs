# NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan

Date: 2026-06-25

Status: Planned

Historical remediation items:

- `Priority 4 Follow-Up — Normalize Non-3GPP External Client Ownership`
- residual follow-up after `Priority 2 — Put Long-Running Work Under App Lifecycle Control`

This plan defines one combined implementation round because the remaining
external-client ownership gap and the remaining lifecycle-context gap are now
the same boundary problem in practice.

Related issue record:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

---

## 1. Purpose

This plan defines the next implementation round for the main `NWDAF/`
repository after the completed Priority 4 boundary reconstruction and the
already-completed core Priority 2 notifier-lifecycle fix.

The purpose of this round is:

1. remove the remaining ad hoc construction of non-3GPP external clients from
   app-owned domain logic
2. make runtime-managed outbound I/O derive ownership from the already-built
   app/service boundary instead of from package-local `New...Client(...)` calls
   plus `context.Background()` timeouts
3. absorb the remaining narrow Priority 2 lifecycle hardening that naturally
   falls out of that ownership cleanup

This round is intentionally not a broad repository cleanup.

It is about:

- external client ownership for Daisy and ML service paths
- adjacent runtime I/O context ownership in notifier and related external-call
  paths
- preserving the current shared `pkg/app` boundary while making the remaining
  ownership claims true in practice

It is not about:

- reopening the completed Priority 4 root-contract redesign
- package relocation of Daisy-specific workflow glue out of `internal/sbi`
- OpenAPI or model-governance redesign
- changing the intended late-failure semantics of accepted subscriptions
- NRF registration, metrics, TLS, or broader free5GC integration-level uplift
- full repository-wide elimination of every `context.Background()` call without
  regard to shutdown semantics

---

## 2. Why This Combined Round Exists

The current project state no longer matches the earlier remediation ordering in
which Priority 2 and Priority 4 were separate large rounds.

Those larger rounds are materially complete for their original scope:

1. the main app boundary was rebuilt
2. package-global runtime config drift in already-touched paths was cleaned up
3. notifier scheduler ownership now ties into application shutdown
4. handler, processor, and consumer test seams were strengthened around the
   new boundary

However, one narrower ownership split still remains:

1. AnLF and MTLF still construct Daisy and ML service clients directly inside
   domain logic
2. several outbound runtime-managed calls still manufacture their own timeout
   contexts from `context.Background()`
3. notifier runtime delivery still uses `http.DefaultClient`

This means NWDAF currently has one established shared app boundary, but not all
of the long-running or app-owned outbound I/O actually flows through it.

That remaining gap is both:

- the still-open `Priority 4 Follow-Up — Normalize Non-3GPP External Client Ownership`
- the most concrete residual portion of Priority 2 that still matters for
  app-owned runtime I/O

Treating them as one round keeps the change narrow and avoids splitting one
ownership problem into artificial documentation phases.

---

## 3. Planning Basis

This plan is based on the following local sources:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Rebuild One Real App Boundary Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Residual Runtime Truth Cleanup Plan.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/source-orientation.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `resources/references/free5gc-main/NFs/udr/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- the current `NWDAF/` tree on 2026-06-25

Exemplar choice for this round:

1. primary exemplar: `udr`
   - clean control-plane `service -> app -> consumer` ownership
   - clear consumer construction from the app boundary
2. secondary exemplar: `udm`
   - consumer-heavy control-plane shape with multiple service clients

These exemplars do not provide a literal Daisy or ML service equivalent.
Their value here is the ownership rule:

- peer-facing clients are assembled from the app/service boundary
- domain logic does not rediscover or reconstruct runtime clients ad hoc

---

## 4. Current NWDAF State

### 4.1 What Is Already Completed

The current repository already reflects the intended structural progress from
the completed earlier rounds:

1. `pkg/app.App` is now the shared root runtime contract
2. `pkg/service.NwdafApp` owns app construction, processor construction,
   consumer construction, server construction, and shutdown orchestration
3. the main `internal/sbi/consumer.NewConsumer(app)` path is app-driven
4. app-owned runtime code that previously depended on `factory.NwdafConfig`
   has already been cleaned up in the covered Priority 4 residual round

This plan therefore does not need to revisit the completed root app-boundary
reconstruction.

### 4.2 Confirmed Remaining Ownership Gap

The current NWDAF tree still has direct external-client instantiation in
app-owned domain logic:

1. `internal/anlf/analytics.go`
   - derives ML service endpoint from config
   - constructs a fresh `consumer.NewMlServiceClient(...)`
2. `internal/anlf/model.go`
   - constructs a fresh ML service client for model load and swap paths
3. `internal/mtlf/training.go`
   - constructs a fresh `consumer.NewDaisyClient(...)` for async task
     submission
4. `internal/mtlf/adrf_retrieval.go`
   - constructs a fresh Daisy client inside the retrain fetch loop
   - constructs ADRF retrieval client directly for workflow-local use

This is the remaining Priority 4 ownership inconsistency:

- the shared app boundary exists
- but these external runtime clients still bypass it

### 4.3 Confirmed Residual Lifecycle Hardening Gap

The current tree also still has narrow residual runtime-I/O ownership drift:

1. `internal/notifier/notifier.go`
   - still falls back to `context.Background()` when no base context is passed
   - still uses `http.DefaultClient` for outbound notification delivery
2. `internal/sbi/consumer/ml_service.go`
   - per-call timeouts still derive from `context.Background()`
3. `internal/sbi/consumer/daisy_service.go`
   - per-call timeouts still derive from `context.Background()`
4. `internal/sbi/consumer/adrf_service.go`
   - retrieval and unsubscribe paths still derive timeout contexts from
     `context.Background()`
5. `pkg/service/init.go`
   - MongoDB ping and collection creation still use background-derived timeout
     contexts

These are no longer the same class of bug as the earlier detached notifier
scheduler issue.

They are now narrower app-owned I/O-context questions:

- which runtime calls should inherit app cancellation
- which calls should intentionally use a shutdown-local timeout context
- which HTTP clients should be injected instead of using package defaults

### 4.4 Important Current Nuance

Not every `context.Background()` call is automatically wrong.

For example, graceful shutdown code sometimes needs a fresh timeout context
because the app cancellation context is already done.

This plan therefore does not treat "remove every background context" as the
target.

The target is narrower:

- remove accidental background ownership for normal runtime-managed outbound I/O
- keep intentional shutdown-local timeout contexts where using the canceled app
  context would break graceful cleanup

---

## 5. free5GC Baseline And Design Direction

### 5.1 App-Owned Client Construction

The surveyed free5GC control-plane NFs such as UDR and UDM follow one stable
pattern:

1. `pkg/service.NewApp(...)` constructs the concrete app
2. that app constructs consumer-owned clients
3. runtime logic uses those owned clients through the app or consumer seam

Implication for NWDAF:

- Daisy and ML service clients should stop being created inside AnLF and MTLF
- they should be assembled from app-owned wiring and then consumed through a
  local seam

### 5.2 Keep The Root App Contract Narrow

The completed Priority 4 Phase 2 work already established an important
boundary rule:

1. root `pkg/app.App` should stay narrow
2. layer-specific extension seams may embed it when extra behavior is needed

Implication for this round:

- do not widen `pkg/app.App` again
- if AnLF, MTLF, notifier, or processor need access to owned external clients,
  use a local extension seam or a smaller app-owned helper seam

### 5.3 Distinguish Standardized NF Consumers From Local Integrations

The free5GC exemplar baseline mainly covers peer NF consumers.
NWDAF now also has project-local integrations such as Daisy and the external ML
service.

Implication for this round:

1. keep the ownership rule aligned with free5GC
2. keep the classification explicit

That means:

- standardized or semi-standard NF interaction may remain under
  `internal/sbi/consumer`
- project-local external integrations should still be owned from the app
  boundary, even if their concrete client code temporarily remains in the same
  package for this round

This avoids forcing premature package relocation into a plan that is supposed
to stay narrower than Priority 12.

### 5.4 Runtime I/O Must Have A Clear Owner

The lifecycle guidance from the local skill and the completed earlier rounds
supports a consistent rule:

1. long-lived or app-managed work should inherit app-owned cancellation
2. network calls should have explicit timeout bounds
3. shutdown internals may use a dedicated timeout context when that is required
   for graceful termination

Implication for NWDAF:

- external client request methods should stop creating isolated ownership from
  `context.Background()` by default
- notifier delivery should stop relying on `http.DefaultClient`
- service startup helper I/O should prefer the app startup context where safe

---

## 6. Fixed Decisions For This Round

The following decisions are treated as fixed for this plan:

1. Keep the round narrow and behavior-preserving.
2. Do not widen the root `pkg/app.App` contract.
3. Do not perform broad repository/package relocation in this round.
4. Move Daisy and ML service client construction behind app-owned wiring rather
   than direct domain-local `New...Client(...)` calls.
5. Preserve the explicit distinction between:
   - shared NF consumer ownership
   - project-local external integration ownership
6. Prefer local extension seams or small app-owned helper seams over new
   package-global helpers.
7. For runtime-managed outbound I/O, prefer a passed parent context plus a
   local timeout rather than a brand-new root background context.
8. Keep dedicated background-derived timeout contexts only where they are
   intentionally required for shutdown or cleanup after app cancellation.
9. Update tests together with the ownership changes.

These decisions follow the workspace development policy:

- smallest coherent round
- no silent architecture pivot beyond the documented plan
- keep verification scope explicit

---

## 7. Target End State

At the end of this round:

1. AnLF no longer constructs ML service clients directly inside runtime logic.
2. MTLF no longer constructs Daisy clients directly inside runtime logic.
3. the external-client ownership path for these integrations is app-owned and
   mockable.
4. notifier outbound delivery no longer relies on `http.DefaultClient`.
5. normal runtime-managed outbound I/O no longer derives its ownership from
   implicit `context.Background()` roots.
6. intentional shutdown-local timeout contexts remain explicit where needed.
7. tests can exercise AnLF and MTLF ownership paths without reconstructing
   hidden external clients inside the code under test.

---

## 8. Scope

### 8.1 In Scope

- `NWDAF/internal/anlf/anlf.go`
- `NWDAF/internal/anlf/analytics.go`
- `NWDAF/internal/anlf/model.go`
- `NWDAF/internal/mtlf/mtlf.go`
- `NWDAF/internal/mtlf/training.go`
- `NWDAF/internal/mtlf/adrf_retrieval.go`
- `NWDAF/internal/notifier/notifier.go`
- `NWDAF/internal/sbi/consumer/ml_service.go`
- `NWDAF/internal/sbi/consumer/daisy_service.go`
- `NWDAF/internal/sbi/consumer/adrf_service.go`
- `NWDAF/internal/sbi/consumer/consumer.go`
- `NWDAF/internal/sbi/processor/processor.go`
- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/mockapp/app.go`
- focused tests for AnLF, MTLF, notifier, consumer, and processor seams

### 8.2 Conditional In Scope

The following may be touched only if needed to keep the ownership cleanup
coherent:

- `NWDAF/internal/sbi/server.go`
  - only if the final implementation needs a clarified shutdown-context rule
- small local helper interfaces in `internal/anlf`, `internal/mtlf`, or
  `internal/notifier`
- adjacent generated mocks if the chosen seam requires them

### 8.3 Explicit Non-Goals

- moving Daisy callback HTTP routes out of `internal/sbi`
- redesigning `failEventReports` or late activation semantics
- repository-level split of lab/workflow config
- replacing handwritten standardized payloads with generated models
- adding NRF registration, deregistration, metrics, OAuth, or TLS support
- broad cleanup of unrelated `context.Background()` use in database/query code
  that is outside this round's app-owned outbound-I/O target

---

## 9. Proposed Design Direction

### 9.1 Ownership Shape

The preferred design direction for this round is:

1. `pkg/service.NewApp(...)` remains the top-level runtime owner
2. app-owned construction prepares the external clients needed for:
   - ML service runtime calls
   - Daisy training workflow calls
   - existing ADRF calls as needed by the current workflow
3. AnLF and MTLF receive those clients through narrower app-derived seams or
   injected local interfaces

This keeps the root app contract stable while still making external ownership
explicit.

### 9.2 Package Placement Rule For This Round

This plan does not require immediate package relocation of concrete client
implementations.

Acceptable for this round:

- leave the concrete Daisy, ML, and ADRF client types in
  `internal/sbi/consumer` if that minimizes change
- but stop allowing AnLF and MTLF to instantiate them directly

The key requirement is ownership, not package movement.

### 9.3 Context Rule

The preferred context rule for this round is:

1. methods that perform normal runtime-managed outbound I/O should accept a
   parent context
2. those methods may derive a bounded timeout from that parent context
3. app-managed background paths should use the app cancellation context as the
   parent
4. shutdown-only cleanup may still use a fresh timeout context when the parent
   app context is already canceled

### 9.4 HTTP Client Rule

The preferred HTTP client rule for this round is:

1. each external integration client owns its own reusable `http.Client`
2. notifier should use an injected or owned client, not `http.DefaultClient`
3. tests should continue to intercept these owned clients directly with `gock`
   where appropriate

---

## 10. Proposed Workstreams

### Workstream A — Define The External-Client Ownership Seam

Goal:

- choose the minimum app-derived seam through which AnLF and MTLF access owned
  external clients

Tasks:

1. decide whether the owned clients are exposed through:
   - a narrowed extension of the existing app seam
   - a smaller helper seam owned by processor/service and passed into AnLF and
     MTLF
2. keep the root `pkg/app.App` unchanged
3. keep test seams compatible with the shared `pkg/mockapp` pattern

Expected outcome:

- domain code no longer needs to call `consumer.NewMlServiceClient(...)` or
  `consumer.NewDaisyClient(...)` directly

### Workstream B — Move ML Service Ownership Out Of AnLF

Goal:

- remove domain-local ML service client construction from AnLF runtime paths

Tasks:

1. replace direct ML client construction in:
   - `internal/anlf/analytics.go`
   - `internal/anlf/model.go`
2. wire the owned ML client through the chosen seam
3. preserve current model-load, swap, and prediction behavior

Expected outcome:

- AnLF runtime logic depends on an owned ML client interface rather than on a
  constructor call plus endpoint lookup

### Workstream C — Move Daisy Ownership Out Of MTLF

Goal:

- remove domain-local Daisy client construction from MTLF runtime paths

Tasks:

1. replace direct Daisy client construction in:
   - `internal/mtlf/training.go`
   - `internal/mtlf/adrf_retrieval.go`
2. keep the current async training callback workflow unchanged
3. preserve the current ADRF-assisted retrain flow shape

Expected outcome:

- MTLF workflow dispatch uses an owned Daisy client seam instead of ad hoc
  client creation

### Workstream D — Tighten Residual Runtime I/O Context Ownership

Goal:

- absorb the remaining narrow Priority 2 lifecycle hardening adjacent to the
  ownership cleanup

Tasks:

1. update ML, Daisy, and ADRF client request paths to accept or derive from a
   parent context instead of creating an independent root background context
2. stop notifier delivery from using `http.DefaultClient`
3. review `pkg/service/init.go` startup helper I/O and derive from app startup
   context where safe
4. keep shutdown-local timeout contexts only where they are intentionally
   needed for graceful termination

Expected outcome:

- runtime-managed outbound I/O has explicit parent ownership and bounded
  timeout behavior

### Workstream E — Update Tests And Shared Mocks

Goal:

- keep the new ownership path testable at the same quality level as the
  completed Priority 3 and Priority 4 rounds

Tasks:

1. extend or adjust `pkg/mockapp` only as needed for the chosen seam
2. add focused tests for:
   - AnLF owned ML client usage
   - MTLF owned Daisy client usage
   - notifier owned HTTP client usage or cancellation behavior
   - context-derived timeout behavior where practical
3. keep consumer HTTP behavior tested with `gock`
4. keep tests close to the smallest useful boundary

Expected outcome:

- the ownership cleanup is protected by focused seam-based tests rather than by
  indirect repository-wide coverage only

---

## 11. Recommended Implementation Order

1. lock the ownership seam first
2. move ML service ownership out of AnLF
3. move Daisy ownership out of MTLF
4. tighten notifier and adjacent runtime-I/O context ownership
5. update tests and mocks alongside each seam change
6. rerun focused package tests
7. rerun full repository verification

This order keeps the most cross-cutting boundary decision first and defers the
smaller residual lifecycle cleanup until the ownership path is explicit.

---

## 12. Acceptance Criteria

This round should be considered complete only when all of the following are
true:

1. AnLF no longer directly calls `consumer.NewMlServiceClient(...)` in runtime
   logic.
2. MTLF no longer directly calls `consumer.NewDaisyClient(...)` in runtime
   logic.
3. the chosen external-client ownership seam is app-owned and mockable.
4. notifier delivery no longer uses `http.DefaultClient`.
5. covered runtime-managed outbound request paths no longer create isolated
   root background ownership for normal operation.
6. any remaining background-derived shutdown timeout contexts are deliberate
   and documented by code shape, not accidental leftovers.
7. focused tests cover the new ownership seams and the most relevant lifecycle
   behavior.
8. full repository verification passes.

---

## 13. Verification Plan

Focused package verification:

```bash
go test ./internal/anlf
go test ./internal/mtlf
go test ./internal/notifier
go test ./internal/sbi/consumer
go test ./internal/sbi/processor
```

Full repository verification:

```bash
go test ./...
make build
make lint
```

Verification notes:

1. this round is expected to remain unit-level and build-level verification
2. no MongoDB-, NRF-, Daisy-, or external-ML-service-backed integration run is
   required as a completion gate unless implementation reveals a gap that unit
   seams cannot cover
3. if context ownership changes force a test-design decision that current seams
   cannot express cleanly, stop and re-evaluate before widening architecture
   beyond this plan

---

## 14. Risks And Guardrails

Main risks:

1. accidentally widening the root app contract again
2. silently turning this ownership cleanup into a package-relocation round
3. breaking graceful shutdown by replacing an intentional shutdown-local timeout
   context with an already-canceled app context
4. over-coupling Daisy and ML client ownership to the main NF consumer API in a
   way that obscures their non-3GPP role

Guardrails:

1. keep root `pkg/app.App` unchanged
2. prefer small local extension seams
3. keep behavior-preserving ownership moves separate from later repository
   cleanup
4. distinguish normal runtime I/O from termination-only cleanup contexts

---

## 15. Follow-Up After This Round

If this round completes successfully, the most natural next candidates remain:

1. `Priority 6 — Clarify Post-Subscription Activation And Late-Failure Signaling`
2. `Priority 7 — Tighten Logging Boundaries`
3. `Priority 10 — Establish OpenAPI / Model Governance`
4. `Priority 11 — Decide The Intended free5GC Integration Level`
5. `Priority 9 — Separate Runtime Config From Lab / Workflow Config`
6. `Priority 12 — Clean Repo And Package Ownership Boundaries`

The main value of this round is that those later topics would then build on a
stricter and more truthful app-owned runtime boundary, rather than on a mixed
boundary where several important outbound integrations still bypass the owning
application seam.
