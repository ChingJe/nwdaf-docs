# NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan

Date: 2026-06-25

Status: Phase 1 implemented; Phase 2 implementation committed locally

Historical remediation items:

- `Priority 4 Follow-Up — Normalize Non-3GPP External Client Ownership`
- residual follow-up after `Priority 2 — Put Long-Running Work Under App Lifecycle Control`

This plan defines one combined implementation round because the remaining
external-client ownership gap and the remaining lifecycle-context gap are now
the same boundary problem in practice.

Related issue record:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

Implementation status:

- Phase 1 implementation landed in `NWDAF/` on 2026-06-25
- committed as `637235b` in the local `NWDAF/` repository
- moved ML, Daisy, and ADRF runtime clients behind app-owned runtime seams
- changed notifier and covered external integration paths to use owned HTTP
  clients and parent-context-derived request timeouts
- added focused owned-client tests for AnLF and MTLF

Phase 2 finding status:

- after Phase 1 implementation and verification, a stricter review found that
  the covered ownership direction is materially improved but not yet fully
  closed
- the remaining issues are now treated as Phase 2 findings of this same
  implementation line, not as a separate unrelated round
- the originally recorded Phase 2 findings were:
  1. shutdown-time ADRF retrieval cleanup now inherits the already-canceled app
     context and can skip remote unsubscribe work
  2. client assembly is still processor-owned rather than being fully lifted to
     the outer app/service assembly boundary used by the surveyed free5GC NFs
- after the later local implementation review of those two findings, one more
  same-round lifecycle-closure residual was identified and is intentionally
  absorbed into this plan rather than deferred:
  3. detached ADRF retrain workflow goroutines can outlive the app waitgroup
     boundary and make shutdown truth less strict than the surveyed free5GC NF
     lifecycle shape
- after implementing that lifecycle-closure direction, one further same-round
  shutdown-convergence issue was identified and is also absorbed into this plan:
  4. once the retrain workflow becomes waitgroup-visible, `runFetchLoop(...)`
     can hold shutdown until the runtime ADRF watchdog closes `fetchCh`, which
     incorrectly ties graceful termination to a business-timeout path

Phase 2 implementation status:

- Phase 2 implementation landed in `NWDAF/` on 2026-06-25
- committed as `f026e77` in the local `NWDAF/` repository
- moved ML and Daisy client assembly into `internal/sbi/consumer.NewConsumer(...)`
- changed processor construction to consume already-owned external clients
- kept ADRF unsubscribe teardown on an owner-created bounded cleanup context
- made the ADRF retrain fetch loop stop on app cancel instead of waiting for
  the runtime watchdog
- prevented shutdown from dispatching a new Daisy training task after retrain
  cleanup

Verification rerun after Phase 2 implementation:

- `go test ./internal/anlf ./internal/mtlf ./internal/notifier ./internal/sbi/consumer ./internal/sbi/processor`
- `go test ./...`
- `make build`
- `make lint`

Result:

- focused package tests passed
- full repository test suite passed
- build passed
- lint passed with `0 issues`

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
- `resources/references/free5gc-main/NFs/udm/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/pcf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/service/init.go`
- the current `NWDAF/` tree on 2026-06-25

Exemplar choice for this round:

1. primary exemplar: `udr`
   - clean control-plane `service -> app -> consumer` ownership
   - clear consumer construction from the app boundary
2. secondary exemplar: `udm`
   - consumer-heavy control-plane shape with multiple service clients
   - explicit app waitgroup ownership around service lifecycle work
3. additional lifecycle evidence: `pcf` and `nrf`
   - explicit `wg.Add(...)` before owned goroutines
   - termination path waits on owned service work instead of letting detached
     runtime work silently outlive the app boundary

These exemplars do not provide a literal Daisy or ML service equivalent.
Their value here is the ownership rule:

- peer-facing clients are assembled from the app/service boundary
- domain logic does not rediscover or reconstruct runtime clients ad hoc
- app-owned lifecycle work is made visible to the owning service shutdown shape

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

The surveyed free5GC control-plane NFs such as UDR, UDM, and PCF follow one
stable pattern:

1. `pkg/service.NewApp(...)` constructs the concrete app
2. that app constructs `internal/sbi/consumer.NewConsumer(...)`
3. the consumer owns concrete outbound client assembly
4. runtime logic uses those owned clients through the app or consumer seam

Implication for NWDAF:

- Daisy and ML service clients should stop being created inside AnLF and MTLF
- they should stop being created inside `internal/sbi/processor.NewProcessor(...)`
- they should be assembled from the existing app-to-consumer boundary and then
  consumed through a local seam

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
- project-local external integrations may also remain under
  `internal/sbi/consumer` for this round when that is the closest fit to the
  surveyed free5GC ownership shape
- the important rule is that app/service owns consumer construction and
  consumer owns concrete client assembly, even if the target service is not a
  standardized peer NF

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
4. Move Daisy and ML service client construction behind the existing
   app-to-consumer boundary rather than direct domain-local
   `New...Client(...)` calls.
5. Do not leave ML or Daisy client assembly in
   `internal/sbi/processor.NewProcessor(...)` as the final target shape.
6. Preserve the explicit distinction between:
   - shared NF consumer ownership
   - project-local external integration ownership
7. Prefer local extension seams over new package-global helpers.
8. For runtime-managed outbound I/O, prefer a passed parent context plus a
   local timeout rather than a brand-new root background context.
9. Keep dedicated background-derived timeout contexts where they are
   intentionally required for shutdown or cleanup after app cancellation.
10. Update tests together with the ownership changes.

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
2. app-owned construction builds `internal/sbi/consumer.NewConsumer(...)`
3. the consumer owns the concrete assembly of:
   - ML service runtime clients
   - Daisy training workflow clients
   - existing ADRF clients as needed by the current workflow
4. `internal/sbi/processor.NewProcessor(...)` receives those owned clients from
   `nwdaf.Consumer()` instead of assembling them itself
5. AnLF and MTLF receive those clients through narrower app-derived seams or
   injected local interfaces

This keeps the root app contract stable while still matching the surveyed
free5GC `service -> consumer -> processor` ownership direction more closely.

For the lifecycle side of this same round, the preferred design direction is:

1. retrain workflow goroutines should have an explicit owner
2. app-visible long-lived or multi-step workflow goroutines should not silently
   outlive the service waitgroup boundary
3. shutdown cleanup may still use bounded fresh timeout contexts, but the
   surrounding workflow should remain visible to the owner that is declaring the
   app terminated

### 9.2 Package Placement Rule For This Round

This plan does not require immediate package relocation of concrete client
implementations.

Acceptable and preferred for this round:

- keep the concrete Daisy, ML, and ADRF client types in
  `internal/sbi/consumer`
- assemble them from `consumer.NewConsumer(...)`
- stop allowing AnLF, MTLF, or processor to instantiate them directly

The key requirement is ownership and assembly placement, not package movement.

### 9.3 Context Rule

The preferred context rule for this round is:

1. methods that perform normal runtime-managed outbound I/O should accept a
   parent context
2. those methods may derive a bounded timeout from that parent context
3. app-managed background paths should use the app cancellation context as the
   parent
4. shutdown-only cleanup may still use a fresh timeout context when the parent
   app context is already canceled

Stricter clarification after the free5GC re-review:

- the shutdown-cleanup exception should be chosen by the owner of the cleanup
  workflow, not hidden inside the consumer method
- in NWDAF that means ADRF retrieval unsubscribe cleanup should receive a
  dedicated owner-created timeout context from MTLF or the surrounding
  shutdown path, while the consumer continues to honor the context it is given
- the same owner should also decide whether the surrounding retrain workflow is
  waitgroup-visible lifecycle work rather than leaving it as a detached
  goroutine lineage

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

- align the ownership seam with the surveyed free5GC
  `service -> consumer -> processor` shape while keeping the root app contract
  stable

Tasks:

1. keep the root `pkg/app.App` unchanged
2. extend `internal/sbi/consumer.Consumer` ownership to cover ML and Daisy
   runtime clients alongside ADRF
3. expose those owned clients through the existing `Consumer()` seam or a
   narrow local extension seam without re-widening `pkg/app.App`
4. keep test seams compatible with the shared `pkg/mockapp` pattern

Expected outcome:

- domain code no longer needs to call `consumer.NewMlServiceClient(...)` or
  `consumer.NewDaisyClient(...)` directly
- processor no longer needs to assemble those clients directly

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
5. make the shutdown-cleanup exception explicit at the owner boundary, with
   ADRF unsubscribe as the immediate target case

Expected outcome:

- runtime-managed outbound I/O has explicit parent ownership and bounded
  timeout behavior

### Workstream E — Close Retrain Workflow Lifecycle Ownership

Goal:

- ensure the ADRF retrain workflow does not remain a detached lineage that can
  outlive the app's declared shutdown boundary

Tasks:

1. inventory the current detached goroutine lineage for:
   - `runAdrfRetrainWorkflow(...)`
   - `runFetchLoop(...)`
   - post-fetch Daisy task dispatch
2. define which parts are app-owned lifecycle work and therefore should be
   visible to the owning waitgroup
3. preserve bounded shutdown cleanup for unsubscribe work without turning
   termination into an unbounded wait
4. prevent shutdown from silently expanding the runtime work set by dispatching
   new Daisy training work after termination has begun
5. make shutdown convergence independent from the runtime ADRF watchdog by
   giving `runFetchLoop(...)` an app-cancel exit path

Expected outcome:

- the current round closes both the cleanup-context gap and the adjacent
  retrain-workflow ownership gap instead of deferring the latter
- shutdown waits only on bounded owned cleanup, not on runtime watchdog expiry

### Workstream F — Update Tests And Shared Mocks

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
2. move ML and Daisy client assembly into `internal/sbi/consumer`
3. move ML service ownership out of AnLF
4. move Daisy ownership out of MTLF
5. tighten notifier and adjacent runtime-I/O context ownership
6. make shutdown cleanup context explicit for ADRF unsubscribe
7. close retrain workflow lifecycle ownership so shutdown truth remains strict
8. update tests and mocks alongside each seam change
9. rerun focused package tests
10. rerun full repository verification

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
3. processor no longer directly assembles ML or Daisy clients from config.
4. the chosen external-client ownership seam is app-owned, consumer-owned at
   concrete assembly, and mockable.
5. notifier delivery no longer uses `http.DefaultClient`.
6. covered runtime-managed outbound request paths no longer create isolated
   root background ownership for normal operation.
7. ADRF unsubscribe cleanup no longer depends on the already-canceled app
   context during teardown.
8. ADRF retrain workflow goroutines that materially belong to app lifecycle no
   longer outlive the app waitgroup boundary as detached work.
9. any remaining background-derived shutdown timeout contexts are deliberate
   and documented by code shape, not accidental leftovers.
10. shutdown does not wait for the ADRF runtime watchdog once app termination
    has begun.
11. focused tests cover the new ownership seams and the most relevant lifecycle
    behavior.
12. full repository verification passes.

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
4. focused lifecycle verification should explicitly cover retrain-workflow
   shutdown behavior, including app-cancel convergence before watchdog expiry,
   not only per-request timeout behavior

---

## 14. Risks And Guardrails

Main risks:

1. accidentally widening the root app contract again
2. silently turning this ownership cleanup into a package-relocation round
3. breaking graceful shutdown by replacing an intentional shutdown-local timeout
   context with an already-canceled app context
4. over-coupling Daisy and ML client ownership to the main NF consumer API in a
   way that obscures their non-3GPP role
5. declaring NWDAF terminated while detached retrain cleanup or dispatch work is
   still running in the background
6. making shutdown depend on business watchdog expiry instead of an explicit
   owner-controlled exit path

Guardrails:

1. keep root `pkg/app.App` unchanged
2. prefer small local extension seams
3. keep behavior-preserving ownership moves separate from later repository
   cleanup
4. distinguish normal runtime I/O from termination-only cleanup contexts
5. treat app-declared termination and app-owned workflow completion as one
   lifecycle contract, unless a narrower documented exception is deliberate

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

---

## 16. Phase 2 Findings After Phase 1 Implementation

This section records the stricter reassessment after the covered Phase 1 code
was implemented and verified.

The implementation did solve the original highest-value part of this round:

1. AnLF and MTLF no longer construct Daisy or ML service clients directly
2. notifier no longer uses `http.DefaultClient`
3. covered external integration calls now accept a parent context instead of
   always rooting themselves at `context.Background()`

However, the original two narrower findings were followed by one same-round
lifecycle-closure residual, and all three are now treated as part of this
round rather than as deferred later work.

### 16.1 Phase 2 Finding A — Shutdown Cleanup Context Is Too Narrow

Confirmed issue:

- ADRF retrain cleanup now uses the app cancellation context even for remote
  unsubscribe work during teardown
- this can cause cleanup to stop immediately once app termination begins,
  leaving remote ADRF retrieval subscriptions behind

Confirmed current locations:

- `NWDAF/internal/mtlf/adrf_retrieval.go`
- `NWDAF/internal/sbi/consumer/adrf_service.go`

Why this remains open:

1. normal runtime I/O should inherit app-owned cancellation
2. shutdown cleanup is a narrower special case
3. unsubscribe and similar teardown operations may still need a short-lived
   cleanup timeout context after the main app context has already been canceled

Phase 2 direction:

- keep normal runtime request ownership as implemented in Phase 1
- introduce an explicit shutdown-cleanup context rule for remote ADRF
  unsubscribe work instead of reusing the already-canceled app context
- keep that exception at the owner boundary rather than hiding it inside the
  consumer implementation

### 16.2 Phase 2 Finding B — Assembly Point Still Sits In Processor

Confirmed structural residual:

- external client ownership is improved, but ML and Daisy client construction
  is still initiated inside `internal/sbi/processor.NewProcessor(...)`
- the surveyed free5GC control-plane NFs assemble comparable outbound clients
  from the outer app/service-to-consumer boundary rather than from the
  processor constructor

Evidence basis:

- direct exemplar evidence from UDR and UDM shows app/service-owned consumer
  assembly
- no exact free5GC Daisy or external ML service exemplar exists, so this
  residual judgment is partly inference from the broader control-plane
  ownership pattern rather than from a one-to-one implementation twin

Why this remains open:

1. Phase 1 removed the domain-local ad hoc construction problem
2. Phase 1 did not yet fully lift the assembly point to the outer app/service
   boundary
3. this means the code is materially better aligned than before, but still not
   at the strictest free5GC-style assembly level

Phase 2 direction:

- preserve the now-established owned-client seams
- move ML and Daisy client assembly into `internal/sbi/consumer.NewConsumer(...)`
- have processor consume those already-owned clients through `nwdaf.Consumer()`
  rather than deciding client construction itself

### 16.3 Same-Round Lifecycle Closure Residual — Detached Retrain Workflow

Confirmed issue:

- the local review after implementing the original two Phase 2 findings showed
  that ADRF retrain workflow goroutines can still outlive the app waitgroup
  boundary
- this means shutdown cleanup may now be more correct for remote unsubscribe,
  but the app can still declare termination before all owned retrain work has
  actually converged

Confirmed current locations:

- `NWDAF/internal/mtlf/training.go`
- `NWDAF/internal/mtlf/adrf_retrieval.go`
- `NWDAF/pkg/service/init.go`

Evidence basis:

- direct exemplar evidence from UDM, PCF, and NRF shows service-owned goroutine
  startup being registered through the app waitgroup before shutdown waits
- no exact free5GC ADRF-retrain equivalent exists, so the judgment about this
  specific workflow shape remains inference from the surveyed ownership and
  lifecycle discipline rather than from a one-to-one implementation twin

Why this must stay in this round:

1. the new cleanup-context fix intentionally preserved remote unsubscribe work
   after app cancellation
2. that improvement makes the surrounding workflow ownership truth more
   important, not less
3. deferring this would leave the current round with a stricter cleanup rule but
   a still-detached workflow boundary

Same-round direction:

- keep the owner-created bounded cleanup context rule introduced for remote
  unsubscribe
- also make the surrounding retrain workflow visible to app lifecycle
  ownership where it is materially long-lived work
- avoid dispatching new Daisy work from a teardown path once termination has
  already begun

### 16.4 Same-Round Shutdown Convergence Residual — Owned Workflow Still Waits On Runtime Watchdog

Confirmed issue:

- after making the retrain workflow waitgroup-visible, shutdown can still block
  on `runFetchLoop(...)` until the ADRF runtime watchdog fires
- this means lifecycle ownership is stricter than before, but shutdown is still
  coupled to a business-timeout path rather than an owner-controlled cancel path

Confirmed current locations:

- `NWDAF/internal/mtlf/adrf_retrieval.go`
- `NWDAF/pkg/factory/config.go`
- `NWDAF/pkg/service/init.go`

Why this remains in-scope for this same round:

1. the detached-work residual and this watchdog-bound wait are two halves of the
   same lifecycle closure problem
2. once the workflow is correctly brought under the app waitgroup, its stop
   condition must also become shutdown-aware
3. changing the runtime watchdog value would alter normal business behavior and
   is therefore not the right fix for a termination-only problem

Same-round direction:

- keep the ADRF watchdog as a runtime convergence timeout for normal operation
- add an app-cancel path so `runFetchLoop(...)` can stop waiting immediately
  once shutdown begins
- preserve bounded unsubscribe cleanup after cancellation
- do not dispatch new Daisy training work from the shutdown path
