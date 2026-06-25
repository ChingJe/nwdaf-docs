# NWDAF Priority 2 Remaining Lifecycle Completion Plan

Date: 2026-06-25

Revised: 2026-06-26 after NF-backed overlap review against adjacent issue lines

Status: Implemented on 2026-06-26 and verified against the bounded Priority 2 scope

Related issue records:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

Related completed implementation records:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan.md`

## 0. 2026-06-26 Implementation Outcome

Implementation status for this bounded Priority 2 plan:

1. Workstream 0 inventory and classification were completed before code edits
2. Workstream A was implemented
   - remaining SMF / MTLF / ML / Daisy / ADRF runtime consumer paths now
     accept a parent context and no longer silently fall back to
     `context.Background()`
3. Workstream B was implemented within the bounded Priority 2 scope
   - request-triggered async paths in subscription data collection and ML model
     provisioning remain one-shot request-triggered work
   - shutdown now prevents these paths from expanding new runtime work
   - the implementation did not absorb Priority 6 post-acceptance activation
     redesign
4. Workstream C was implemented
   - MongoDB query helpers now derive bounded timeouts from caller-owned parent
     contexts
   - AnLF ground-truth lookup now propagates monitor-owned cancellation into
     the MongoDB query path
   - UPF persistence writes now derive from the app cancel context instead of a
     root background context
5. Workstream D was implemented
   - remaining production `context.Background()` usage in the touched scope is
     now limited to explicit cleanup / shutdown exceptions such as bounded SBI
     shutdown, cursor close cleanup, and ADRF unsubscribe cleanup
6. Workstream E was implemented
   - touched consumer, processor, handler, and AnLF tests were updated to
     reflect the tighter parent-context contracts and lifecycle behavior

Verification rerun after implementation:

```bash
make build
go test ./...
make lint
```

Result:

1. build passed
2. full Go test suite passed
3. `golangci-lint` reported `0 issues`

Development-policy completion review conclusion:

1. the bounded Priority 2 plan was satisfied for the intended lifecycle /
   ownership scope
2. free5GC-style alignment remained acceptable against the plan's exemplar
   basis:
   - app/service lifecycle truth stays at the owner boundary
   - request-triggered async follow-up was not over-promoted into a separate
     app-owned worker system
   - runtime outbound I/O and DB access now follow caller-owned lifetime
3. no new confirmed issue was found inside this bounded Priority 2 scope during
   the post-implementation review
4. adjacent still-open work remains under Priority 5, 6, 7, 10, 11, and 12
   exactly as bounded by this plan

---

## 1. Purpose

This plan defines the next implementation round for the still-open portions of:

- `Priority 2 — Put Long-Running Work Under App Lifecycle Control`

The 2026-06-25 external-client ownership implementation line already closed the
Priority 2 residuals that were directly adjacent to ML / Daisy / ADRF ownership:

1. covered external integration requests now derive from parent contexts
2. ADRF teardown unsubscribe cleanup uses an owner-created bounded cleanup
   context
3. the ADRF retrain workflow no longer escapes the app lifecycle boundary as
   detached work
4. shutdown no longer waits for the ADRF runtime watchdog before retrain
   cleanup converges

This plan is for the broader remaining Priority 2 work that still spans the
wider repository.

This revised version treats the remaining work as an inventory-driven closure
round rather than another narrow spot fix.

The goal is not to reopen the already completed Priority 4 ownership line.

The goal is to close the remaining lifecycle inconsistency across:

1. outbound runtime-managed I/O that still roots itself at
   `context.Background()`
2. asynchronous work that still lacks explicit ownership classification at the
   app, server, processor, or request boundary
3. database/runtime helper paths that still hide request or app lifetime from
   their callers
4. remaining lifecycle exceptions that are still implicit instead of deliberate
   and documented by code shape

The goal is also to prevent Priority 2 from absorbing adjacent design,
contract, package-placement, or architecture issues just because they touch the
same files.

---

## 2. Why A Separate Remaining-Priority-2 Plan Is Needed

The current documentation now distinguishes between:

1. completed Priority 2 portions already landed in round 1 and in the later
   2026-06-25 ownership line
2. still-open Priority 2 work that is broader than the completed ownership line

That distinction is necessary because the remaining open work is no longer one
single bug.

It is now a repository-level cleanup of lifecycle discipline.

Without a dedicated plan:

1. it is too easy to conflate closed Priority 4 follow-up work with still-open
   Priority 2 work
2. it is too easy to treat every `context.Background()` occurrence as equally
   wrong even when some are legitimate shutdown-only cleanup exceptions
3. it is too easy to change ownership paths piecemeal without one explicit
   classification rule for long-lived work, request-owned I/O, and
   explicit cleanup exceptions

This plan therefore narrows the remaining Priority 2 work into explicit
implementation categories and adds an explicit boundary against adjacent issue
lines before implementation starts.

---

## 3. Planning Basis

This plan is based on the following local sources:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `resources/references/free5gc-main/NFs/udm/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/pcf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- the current `NWDAF/` tree on 2026-06-25

This plan also uses direct inspection of current remaining code paths in:

- `NWDAF/internal/sbi/consumer/`
- `NWDAF/internal/sbi/processor/`
- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/anlf/`
- `NWDAF/internal/context/db_query.go`
- `NWDAF/internal/notifier/notifier.go`

---

## 4. Exemplar Choice And Evidence Basis

This remaining Priority 2 plan uses the following free5GC exemplar set:

1. primary exemplar: `nrf`
   - `pkg/service/init.go`
   - evidence used:
     - app-owned context and waitgroup lifecycle baseline
     - owned shutdown listener and owned server stop path
     - explicit bounded shutdown wait as an owner exception
2. secondary exemplar: `udm`
   - `pkg/service/init.go`
   - `internal/sbi/consumer/consumer.go`
   - evidence used:
     - consumer-owned client assembly from the app boundary
     - request-triggered callback follow-up that does not automatically become
       app-owned long-lived work
3. secondary exemplar: `pcf`
   - `pkg/service/init.go`
   - `internal/sbi/processor/notifier.go`
   - evidence used:
     - request-triggered notification dispatch may still be ad hoc async work
       in free5GC-style control-plane code
     - service shutdown remains separately owned at the app boundary
4. supporting exemplar: `udr`
   - `pkg/service/init.go`
   - `internal/sbi/processor/callback.go`
   - evidence used:
     - app context is passed into service-owned registration work
     - persistence-oriented control-plane NF still keeps lifecycle decisions at
       the service/app boundary
     - callback fan-out can be request-triggered async dispatch without being
       promoted to app-owned long-lived work by default

Direct evidence from these exemplars supports the following design rules:

1. long-lived owned work should be visible to the owning service lifecycle
2. shutdown should stop owned work through service-controlled cancellation and
   stop calls
3. peer-facing clients should sit on the app/service/consumer path
4. bounded fresh timeout contexts may still exist for cleanup exceptions, but
   they should be explicit owner exceptions rather than the default runtime path

There is no exact free5GC one-to-one exemplar for:

1. NWDAF's external ML/Daisy integrations
2. NWDAF's MongoDB query helper layering in `internal/context`
3. NWDAF's exact async subscription side-effect pattern

Those parts of this plan are therefore inference from the exemplar lifecycle
discipline plus the existing NWDAF package boundaries, not direct one-to-one
feature matching.

---

## 5. Boundary Against Adjacent Issue Lines

This plan intentionally closes the remaining lifecycle/ownership scope under
Priority 2 without absorbing adjacent issue lines that already have another
canonical execution home.

### 5.1 Still In Priority 2

The following remain in scope for this plan:

1. runtime outbound I/O that still hides its parent context
2. runtime DB/helper paths that still root themselves locally
3. async dispatch paths whose ownership classification is still unclear
4. lifecycle-sensitive cleanup exceptions that still lack one explicit rule
5. exhaustive inventory and closure of the above within the intended code scope

### 5.2 Explicitly Out Of Scope For This Plan

The following may touch some of the same files, but their canonical homes stay
elsewhere:

1. post-acceptance activation semantics, late-failure signaling, and
   observability once downstream data collection starts asynchronously
   - canonical home: `Priority 6`
2. callback parse/model redesign, `upf-notify` contract normalization, Daisy
   local callback contract cleanup, and exact top-level ADRF retrieval callback
   model governance
   - canonical home: `Priority 5` and `Priority 10`
3. payload-hygiene cleanup, raw outbound payload logging, and broader fatal-log
   boundary review
   - canonical home: `Priority 7`
4. metrics server wiring, NRF registration/deregistration, and deciding whether
   NWDAF should align upward as a fuller free5GC-managed NF
   - canonical home: `Priority 11`
5. moving Daisy / MTLF-specific HTTP glue and other domain-local workflow
   ownership out of generic `internal/sbi`
   - canonical home: `Priority 12`

### 5.3 Scope Rule During Implementation

If a touched file exposes both a Priority 2 gap and an adjacent-issue gap:

1. close the lifecycle/ownership gap that belongs to Priority 2
2. do not silently expand into the adjacent issue line
3. record the surviving adjacent gap explicitly if it remains visible after the
   Priority 2 change

---

## 6. Current Remaining Priority-2 State

### 6.1 What Is Already Closed

The following Priority 2 portions are already closed and should not be reopened
by this plan:

1. notifier scheduler cancellation and termination ownership
2. the covered external integration parent-context hardening from the
   2026-06-25 ownership line
3. ADRF teardown unsubscribe cleanup context ownership
4. retrain-workflow waitgroup visibility
5. shutdown convergence for the ADRF retrain flow

### 6.2 Remaining Category A — Consumer Methods Still Hide Parent Context

Current representative paths:

- `NWDAF/internal/sbi/consumer/ml_service.go`
- `NWDAF/internal/sbi/consumer/mtlf_service.go`
- `NWDAF/internal/sbi/consumer/smf_service.go`
- `NWDAF/internal/sbi/consumer/adrf_service.go`

Current problem shape:

1. some consumer methods still derive request timeout contexts from
   `context.Background()` internally
2. some request helpers still allow `nil` parent context and silently replace it
   with a root background context
3. this hides lifetime control from the caller and weakens app- or
   request-scoped cancellation discipline

Why it remains Priority 2:

1. these are runtime-managed outbound I/O paths
2. they are broader than the already-completed ML/Daisy ownership move
3. they still affect how cancellation propagates across the wider repository

### 6.3 Remaining Category B — Async Work Still Needs Explicit Ownership Classification

Current representative paths:

- `NWDAF/internal/sbi/processor/data_collection.go`
- `NWDAF/internal/sbi/processor/eventssubscription.go`
- `NWDAF/internal/sbi/api_mlmodelprovision.go`

Inventory-only recheck paths:

- `NWDAF/internal/anlf/monitor.go`
- `NWDAF/internal/notifier/notifier.go`

Current problem shape:

1. some async work already has acceptable stop behavior but still lacks a clear
   ownership classification in the code and plan
2. some request-triggered goroutines are still launched as ad hoc `go ...`
   dispatch without one shared rule for whether they are:
   - app-owned long-lived work
   - request-owned short-lived work
   - explicit cleanup-exception work
3. this makes later lifecycle review harder because the remaining exceptions are
   not yet categorized explicitly

Important boundary note:

1. asynchronous activation itself is not treated here as a correctness bug
2. this plan is about ownership, cancellation, and cleanup classification
3. post-acceptance failure signaling remains Priority 6 work

Why it remains Priority 2:

1. the original Priority 2 scope explicitly covered background worker ownership
   consistency across notifier, server, AnLF, and MTLF-related paths
2. the completed MTLF retrain fix solved one important lineage, not the entire
   repository-wide classification problem

### 6.4 Remaining Category C — Persistence / Runtime Helpers Still Root Themselves

Current representative paths:

- `NWDAF/internal/context/db_query.go`
- `NWDAF/internal/sbi/processor/upf_notify.go`

Current problem shape:

1. query helpers still create their own timeout contexts from
   `context.Background()`
2. callers therefore cannot express request or app lifetime through those
   helpers
3. cursor cleanup also still uses background-rooted cleanup calls without one
   explicit documented exception rule

Why it remains Priority 2:

1. these are runtime-managed data access paths, not purely local computation
2. they participate in long-lived analytics and notification flows
3. they are part of the broader app-owned I/O context cleanup still left open
   in the issue record

### 6.5 Remaining Category D — Lifecycle Exceptions Need One Explicit Rule

Current representative paths:

- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/notifier/notifier.go`
- `NWDAF/internal/sbi/consumer/adrf_service.go`
- `NWDAF/internal/context/db_query.go`

Current problem shape:

1. some `context.Background()` usage is acceptable because it is shutdown-only
   cleanup or explicit local fallback
2. some remaining usage is accidental runtime-root ownership and should be
   removed
3. the codebase does not yet expose one explicit classification rule to tell
   those cases apart consistently

This plan therefore needs to define the rule, not only list file edits.

---

## 7. Target Lifecycle Rules

This remaining Priority 2 round should establish the following ownership rules
explicitly:

### 7.1 Rule A — Long-Lived Owned Work

Definition:

- work that can outlive the triggering request and materially belongs to the
  running app

Required behavior:

1. derives from app cancellation
2. is visible to the owning waitgroup or another explicit owned mechanism
3. has a clear stop condition

Examples:

- per-model accuracy monitors
- server lifecycle goroutines
- long-lived retrain orchestration

### 7.2 Rule B — Request-Owned Outbound I/O

Definition:

- outbound network or DB work whose lifetime should be controlled by the caller

Required behavior:

1. caller passes a parent context
2. callee may derive a timeout from that parent
3. callee must not silently replace normal runtime ownership with
   `context.Background()`

Examples:

- SMF / MTLF / ML / ADRF runtime requests
- DB query helpers used during analytics or callback flows

### 7.3 Rule C — Explicit Cleanup Exceptions

Definition:

- cleanup work where the caller's normal runtime parent context is no longer the
  right owner for the close/teardown step

Required behavior:

1. owner explicitly creates a fresh bounded cleanup context
2. exception is local and visible at the owner boundary
3. exception is not used as the default runtime path

Examples:

- server shutdown timeout context
- ADRF unsubscribe cleanup timeout
- bounded cursor/body close operations
- bounded close/cleanup operations where the parent app context is already done

### 7.4 Rule D — Request-Triggered Short Async Dispatch

Definition:

- one-shot async work spawned from a request path to avoid blocking the HTTP
  response

Required behavior:

1. classify explicitly whether it should remain one-shot request-owned work or
   be promoted to app-owned lifecycle work
2. if it stays request-owned, document its stop condition and why app waitgroup
   ownership is not required
3. if it is materially long-lived, move it under Rule A

Examples:

- post-subscription data-collection trigger
- ML model initialization callback follow-up

---

## 8. Proposed Workstreams

### Workstream 0 — Build The Exhaustive Inventory Before Changing Ownership

Goal:

- turn this round into an explicit closure pass instead of another partial
  residual cleanup

Primary files:

- `NWDAF/internal/sbi/`
- `NWDAF/internal/anlf/`
- `NWDAF/internal/notifier/`
- `NWDAF/internal/context/`
- `NWDAF/internal/mtlf/`

Tasks:

1. inventory lifecycle-sensitive occurrences of:
   - `context.Background()`
   - `context.TODO()`
   - helper-local timeout contexts
   - request-triggered `go ...` dispatch
2. classify each occurrence as:
   - Rule A
   - Rule B
   - Rule C
   - Rule D
   - adjacent issue line outside Priority 2
3. do not start code changes until the inventory and classification are complete

Expected outcome:

- every in-scope lifecycle-sensitive path has an explicit closure decision
  before implementation starts

### Workstream A — Normalize Parent-Context Contracts For Remaining Consumers

Goal:

- remove silent root-context ownership from remaining runtime-managed consumer
  paths

Primary files:

- `NWDAF/internal/sbi/consumer/ml_service.go`
- `NWDAF/internal/sbi/consumer/mtlf_service.go`
- `NWDAF/internal/sbi/consumer/smf_service.go`
- selected call sites in `internal/anlf`, `internal/mtlf`, and
  `internal/sbi/processor`

Tasks:

1. change remaining consumer APIs that currently own their entire request
   lifetime to accept `ctx context.Context`
2. keep timeout derivation inside the consumer, but derive it from the caller's
   parent context
3. remove silent runtime fallback to `context.Background()` where the path is
   normal operation rather than an explicit cleanup exception
4. update mocks and tests to reflect explicit parent-context ownership

Expected outcome:

- runtime request ownership becomes visible at the call site instead of hidden
  inside the consumer

### Workstream B — Classify Remaining Async Dispatch Paths Within The Priority 2 Boundary

Goal:

- make the remaining ad hoc async work either explicitly app-owned or
  explicitly one-shot request-owned

Primary files:

- `NWDAF/internal/sbi/processor/data_collection.go`
- `NWDAF/internal/sbi/processor/eventssubscription.go`
- `NWDAF/internal/sbi/api_mlmodelprovision.go`

Verification-only inventory paths:

- `NWDAF/internal/anlf/monitor.go`
- `NWDAF/internal/notifier/notifier.go`

Tasks:

1. inventory each remaining async path and classify it under Rule A or Rule D
2. for any path that is materially long-lived, introduce explicit waitgroup or
   equivalent owner registration
3. for any path that remains request-owned one-shot work, make the stop
   condition and reason explicit in code comments or surrounding shape
4. avoid reopening already-correct notifier and MTLF lifecycle paths unless the
   classification review reveals a concrete inconsistency
5. do not fold post-acceptance activation signaling redesign into this
   workstream; record that separately under Priority 6 if encountered

Expected outcome:

- every remaining async path has one explicit ownership story rather than an
  implicit `go ...` default

### Workstream C — Normalize Persistence And Runtime Helper Context Ownership

Goal:

- propagate parent context into DB/runtime helpers instead of rooting them at
  helper-local background contexts

Primary files:

- `NWDAF/internal/context/db_query.go`
- call sites in `NWDAF/internal/anlf/`
- call sites in `NWDAF/internal/sbi/processor/`
- `NWDAF/internal/sbi/processor/upf_notify.go`

Tasks:

1. add `ctx context.Context` to the exported Mongo query helpers
2. derive bounded query timeouts from the passed parent context instead of from
   `context.Background()`
3. keep cursor/body close behavior explicit and classify any bounded cleanup
   exception under Rule C
4. update all callers and tests

Expected outcome:

- DB helper ownership matches the same caller-driven context model used for
  outbound consumers

### Workstream D — Make Cleanup Exceptions Explicit And Narrow

Goal:

- keep legitimate shutdown-only fresh contexts while preventing them from being
  copied into normal runtime paths

Primary files:

- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/sbi/consumer/adrf_service.go`
- `NWDAF/internal/context/db_query.go`
- `NWDAF/internal/sbi/processor/upf_notify.go`
- any touched cleanup helper discovered during implementation

Tasks:

1. review each remaining `context.Background()` usage in lifecycle-sensitive
   paths
2. classify it as:
   - normal runtime ownership to remove
   - shutdown-only cleanup exception to keep
   - local non-runtime helper case that is out of scope for Priority 2
3. where the exception is kept, make the owner and reason visible in code shape
   and tests

Expected outcome:

- the repository no longer mixes accidental runtime-root ownership with
  legitimate cleanup exceptions

### Workstream E — Extend Tests Along The Ownership Boundaries

Goal:

- protect the remaining lifecycle cleanup with focused seam-level tests

Primary files:

- affected tests in `internal/sbi/consumer/`
- affected tests in `internal/anlf/`
- affected tests in `internal/sbi/processor/`
- any new focused tests for context propagation or cancellation

Tasks:

1. add consumer tests that verify parent context propagation where practical
2. add lifecycle tests for cancellation / stop behavior on touched async paths
3. keep using current local mock seams unless a strictly smaller app seam is
   required
4. do not claim integration-level shutdown validation from unit tests alone

Expected outcome:

- the remaining Priority 2 cleanup is defended by direct tests at the ownership
  boundary that was changed

---

## 9. Recommended Implementation Order

1. build the exhaustive inventory and classify every lifecycle-sensitive path in
   the intended scope
2. lock the Priority 2 boundary against Priority 5, 6, 7, 10, 11, and 12 work
3. classify remaining async dispatch paths and decide which are Rule A versus
   Rule D
4. normalize consumer parent-context contracts
5. normalize DB/runtime helper parent-context contracts, including the
   persistence write path in `upf_notify.go`
6. review and preserve only deliberate cleanup exceptions
7. recheck already-aligned notifier and AnLF paths for closure, without
   reopening them unless an actual inconsistency is found
8. add focused tests alongside each touched boundary
9. rerun focused package tests
10. rerun full repository verification

This order keeps the highest fan-out API changes first, then converges on
worker ownership and exception cleanup after the inventory has fixed the scope.

---

## 10. Acceptance Criteria

This remaining Priority 2 round should be considered complete only when all of
the following are true:

1. the inventory for the intended Priority 2 scope is complete, and every
   lifecycle-sensitive path in that inventory is either:
   - fixed in this round
   - deliberately kept under Rule C
   - explicitly mapped to another issue line outside Priority 2
2. remaining normal-operation consumer calls do not hide their lifetime behind
   internal `context.Background()` ownership
3. remaining DB/runtime helper calls used in analytics or notification flows
   accept a parent context and derive bounded query timeouts from it
4. every remaining async path touched by this round is explicitly classified as
   app-owned long-lived work or request-owned short-lived work
5. any app-owned long-lived work touched by this round is visible to the owning
   lifecycle boundary
6. remaining fresh background-derived timeout contexts in touched files are only
   explicit cleanup exceptions or otherwise explicitly justified
7. touched lifecycle paths have focused cancellation / stop behavior tests where
   practical
8. full repository verification passes after the implementation
9. the project scan status can honestly mark Priority 2 as completed for the
   current intended standalone-NWDAF scope, or the implementation must document
   exactly why one still-open remainder survives
10. no adjacent Priority 5, 6, 7, 10, 11, or 12 design work is silently
    reported as “fixed” just because the same files were touched

---

## 11. Verification Plan

Focused package verification is expected to include some subset of:

```bash
go test ./internal/anlf
go test ./internal/context
go test ./internal/notifier
go test ./internal/sbi
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

1. this round is still expected to close at unit/build/lint level unless a
   touched path reveals an integration-only gap
2. MongoDB-backed behavior may still require local environment support for full
   end-to-end validation and should be reported separately if not run
3. cancellation and stop behavior must be treated as first-class verification
   targets, not as incidental side effects

---

## 11. Risks And Guardrails

Main risks:

1. widening this remaining lifecycle cleanup into a broad architecture rewrite
2. accidentally breaking legitimate shutdown-only cleanup by removing every
   background-derived timeout blindly
3. changing public or internal consumer APIs too widely without staged call-site
   updates
4. turning one-shot request async work into app-owned long-lived work without a
   clear ownership reason
5. claiming Priority 2 closure while leaving one unclassified lifecycle hole in
   live code

Guardrails:

1. preserve current package boundaries
2. preserve the already-completed Priority 4 ownership line instead of
   re-litigating it
3. classify each touched path before editing it
4. keep shutdown-only cleanup exceptions narrow and explicit
5. add or update tests for each ownership boundary that changes

---

## 12. User-Decision Status

No immediate user decision is required to begin this plan.

The default recommended scope is:

1. include remaining consumer parent-context cleanup
2. include remaining DB/runtime helper parent-context cleanup
3. include classification of remaining async ownership paths in the same round

If implementation later reveals that one of those areas requires a larger
architecture change than expected, the plan should be revised before continuing.
