# NWDAF Priority 2 Remaining Lifecycle Completion Plan

Date: 2026-06-25

Status: Proposed after the 2026-06-25 Priority 4 follow-up implementation

Related issue records:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

Related completed implementation records:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan.md`

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
   shutdown-only cleanup

This plan therefore narrows the remaining Priority 2 work into explicit
implementation categories.

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

1. primary exemplar: `udm`
   - `pkg/service/init.go`
   - `internal/sbi/consumer/consumer.go`
   - evidence used:
     - app-owned waitgroup lifecycle entry
     - clear service shutdown path
     - consumer-owned client assembly from the app boundary
2. secondary exemplar: `pcf`
   - `pkg/service/init.go`
   - evidence used:
     - owned service goroutines
     - explicit server stop during termination
     - deliberate use of service-owned shutdown calls rather than detached
       cleanup
3. secondary exemplar: `nrf`
   - `pkg/service/init.go`
   - evidence used:
     - termination path may still use a fresh bounded timeout context for a
       shutdown-only wait
     - this supports the distinction between runtime parent-context ownership
       and shutdown-only exceptions
4. supporting exemplar: `udr`
   - `pkg/service/init.go`
   - evidence used:
     - app context is passed into service-owned registration work
     - persistence-oriented control-plane NF still keeps lifecycle decisions at
       the service/app boundary

Direct evidence from these exemplars supports the following design rules:

1. long-lived owned work should be visible to the owning service lifecycle
2. shutdown should stop owned work through service-controlled cancellation and
   stop calls
3. peer-facing clients should sit on the app/service/consumer path
4. bounded fresh timeout contexts may still exist for shutdown-only cleanup, but
   they should be explicit owner exceptions rather than the default runtime path

There is no exact free5GC one-to-one exemplar for:

1. NWDAF's external ML/Daisy integrations
2. NWDAF's MongoDB query helper layering in `internal/context`
3. NWDAF's exact async subscription side-effect pattern

Those parts of this plan are therefore inference from the exemplar lifecycle
discipline plus the existing NWDAF package boundaries, not direct one-to-one
feature matching.

---

## 5. Current Remaining Priority-2 State

### 5.1 What Is Already Closed

The following Priority 2 portions are already closed and should not be reopened
by this plan:

1. notifier scheduler cancellation and termination ownership
2. the covered external integration parent-context hardening from the
   2026-06-25 ownership line
3. ADRF teardown unsubscribe cleanup context ownership
4. retrain-workflow waitgroup visibility
5. shutdown convergence for the ADRF retrain flow

### 5.2 Remaining Category A — Consumer Methods Still Hide Parent Context

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

### 5.3 Remaining Category B — Async Work Still Needs Explicit Ownership Classification

Current representative paths:

- `NWDAF/internal/sbi/processor/data_collection.go`
- `NWDAF/internal/sbi/processor/eventssubscription.go`
- `NWDAF/internal/sbi/api_mlmodelprovision.go`
- `NWDAF/internal/anlf/monitor.go`
- `NWDAF/internal/notifier/notifier.go`

Current problem shape:

1. some async work already has acceptable stop behavior but still lacks a clear
   ownership classification in the code and plan
2. some request-triggered goroutines are still launched as ad hoc `go ...`
   dispatch without one shared rule for whether they are:
   - app-owned long-lived work
   - request-owned short-lived work
   - shutdown-only cleanup work
3. this makes later lifecycle review harder because the remaining exceptions are
   not yet categorized explicitly

Why it remains Priority 2:

1. the original Priority 2 scope explicitly covered background worker ownership
   consistency across notifier, server, AnLF, and MTLF-related paths
2. the completed MTLF retrain fix solved one important lineage, not the entire
   repository-wide classification problem

### 5.4 Remaining Category C — Persistence / Runtime Helpers Still Root Themselves

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

### 5.5 Remaining Category D — Lifecycle Exceptions Need One Explicit Rule

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

## 6. Target Lifecycle Rules

This remaining Priority 2 round should establish the following ownership rules
explicitly:

### 6.1 Rule A — Long-Lived Owned Work

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

### 6.2 Rule B — Request-Owned Outbound I/O

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

### 6.3 Rule C — Shutdown-Only Cleanup

Definition:

- work that must still run briefly after the main app cancellation path has
  already begun

Required behavior:

1. owner explicitly creates a fresh bounded cleanup context
2. exception is local and visible at the owner boundary
3. exception is not used as the default runtime path

Examples:

- server shutdown timeout context
- ADRF unsubscribe cleanup timeout
- bounded close/cleanup operations where the parent app context is already done

### 6.4 Rule D — Request-Triggered Short Async Dispatch

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

## 7. Proposed Workstreams

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
   normal operation rather than shutdown-only cleanup
4. update mocks and tests to reflect explicit parent-context ownership

Expected outcome:

- runtime request ownership becomes visible at the call site instead of hidden
  inside the consumer

### Workstream B — Classify Remaining Async Dispatch Paths

Goal:

- make the remaining ad hoc async work either explicitly app-owned or
  explicitly one-shot request-owned

Primary files:

- `NWDAF/internal/sbi/processor/data_collection.go`
- `NWDAF/internal/sbi/processor/eventssubscription.go`
- `NWDAF/internal/sbi/api_mlmodelprovision.go`
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

### Workstream D — Make Shutdown Exceptions Explicit And Narrow

Goal:

- keep legitimate shutdown-only fresh contexts while preventing them from being
  copied into normal runtime paths

Primary files:

- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/sbi/consumer/adrf_service.go`
- `NWDAF/internal/context/db_query.go`
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

## 8. Recommended Implementation Order

1. lock the lifecycle classification rules first
2. normalize consumer parent-context contracts
3. normalize DB/runtime helper parent-context contracts
4. classify and adjust remaining async dispatch paths
5. review and preserve only deliberate shutdown exceptions
6. add focused tests alongside each touched boundary
7. rerun focused package tests
8. rerun full repository verification

This order keeps the highest fan-out API changes first, then converges on
worker ownership and exception cleanup.

---

## 9. Acceptance Criteria

This remaining Priority 2 round should be considered complete only when all of
the following are true:

1. remaining normal-operation consumer calls do not hide their lifetime behind
   internal `context.Background()` ownership
2. remaining DB/runtime helper calls used in analytics or notification flows
   accept a parent context and derive bounded query timeouts from it
3. every remaining async path touched by this round is explicitly classified as
   app-owned long-lived work or request-owned short-lived work
4. any app-owned long-lived work touched by this round is visible to the owning
   lifecycle boundary
5. remaining fresh background-derived timeout contexts in touched files are only
   shutdown-only cleanup exceptions or otherwise explicitly justified
6. touched lifecycle paths have focused cancellation / stop behavior tests where
   practical
7. full repository verification passes after the implementation
8. the project scan status can honestly mark Priority 2 as completed for the
   current intended standalone-NWDAF scope, or the implementation must document
   exactly why one still-open remainder survives

---

## 10. Verification Plan

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
