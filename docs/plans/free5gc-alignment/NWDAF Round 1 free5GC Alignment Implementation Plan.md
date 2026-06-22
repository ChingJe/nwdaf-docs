# NWDAF Round 1 free5GC Alignment Implementation Plan

Date: 2026-06-22

Status: Completed on 2026-06-22 and merged to `NWDAF/master`

---

## 1. Purpose

This plan defines the first implementation round that follows the free5GC
alignment scan findings for the main `NWDAF/` repository.

The goal of this round is narrow on purpose:

1. fix the confirmed subscription update correctness problem
2. bring long-running notifier work under the application lifecycle
3. add the minimum targeted test coverage needed to change those paths safely

This plan does not attempt to solve the entire scan in one pass.

Round-1 implementation and verification are complete.
The code changes were carried on `NWDAF/` branch `fix/free5gc-alignment-round1`,
then merged locally into `NWDAF/master` and pushed to `origin/master`.

---

## 2. Classification And Storage Location

This plan is stored under:

```text
nwdaf-docs/docs/plans/free5gc-alignment/
```

Reasoning:

- The work is implementation planning derived from the free5GC alignment scan.
- The scope is broader than one feature area such as `mtlf`, `adrf`, or
  `daisy`.
- The plan is not another scan artifact; it is the execution plan for the
  first remediation round.

---

## 3. Branch Strategy

Repository handling for this plan is intentionally split:

- `NWDAF/`
  - use dedicated implementation branch: `fix/free5gc-alignment-round1`
  - this branch was created for the round-1 work
- `nwdaf-docs/`
  - keep planning and issue updates on the current main branch

This preserves the repository boundary defined by the workspace rules while
still giving the implementation round its own code branch.

---

## 4. Fixed Decisions For Round 1

The following decisions are treated as settled for this round:

1. `PUT /subscriptions/{subscriptionId}` is treated as full replacement of the
   subscription resource, not a constrained merge update.
2. Optional fields omitted from the `PUT` request are not silently inherited
   from the previous stored subscription representation.
3. Asynchronous downstream data-collection activation after subscription
   acceptance is not treated as a spec violation by itself.
4. Redesign of post-acceptance activation signaling is deferred to a later
   round unless a change in this round must touch it incidentally.
5. Daisy-specific callbacks, ADRF retraining callbacks, and the current
   incomplete ML Model Provision workflow are out of scope for this round.

---

## 5. Scope

### 5.1 In Scope

- subscription update correctness for `PUT /subscriptions/{subscriptionId}`
- notifier scheduler ownership and shutdown behavior
- focused processor, lifecycle, and handler-adjacent tests required to prove
  the changed behavior

### 5.2 Expected Code Areas

- `NWDAF/internal/sbi/api_eventssubscription.go`
- `NWDAF/internal/sbi/processor/eventssubscription.go`
- `NWDAF/internal/notifier/`
- `NWDAF/pkg/service/`
- small supporting changes in `NWDAF/internal/context/` if lifecycle ownership
  or stored subscription state requires them

### 5.3 Explicit Non-Goals

- rebuilding the shared `pkg/app` boundary
- repository-wide SBI error contract normalization
- redesign of post-subscription activation and late-failure signaling
- logging cleanup outside the touched paths
- config hardening and config-scope separation
- OpenAPI/model governance work
- free5GC integration-level decisions such as NRF registration or metrics
  server alignment
- repo/package ownership cleanup
- Daisy/MTLF/ADRF callback redesign

---

## 6. Workstreams

### Workstream A — Repair Subscription Update Correctness

Objectives:

- make `PUT` apply the same effective request semantics as `POST`
- reconcile external collection state when the effective subscription shape
  changes
- stop treating update as a local-record-only overwrite

Required outcomes:

- defaults and notification-method resolution are re-applied on update
- stale SMF/data-collection/model side effects do not survive a changed update
- the returned subscription representation matches the effective runtime state
  that NWDAF now intends to maintain

### Workstream B — Put Notifier Work Under App Lifecycle Control

Objectives:

- tie scheduler ownership to the application lifecycle
- ensure shutdown stops active periodic notification work
- remove long-lived `context.Background()` ownership where app-owned context is
  required

Required outcomes:

- active subscription schedulers stop during app termination
- notifier shutdown does not depend on subscription update/delete alone
- goroutine ownership is visible through existing lifecycle control paths

### Workstream C — Add Targeted Safety-Net Tests

Objectives:

- prove update-time reconcile behavior
- prove notifier cancellation and shutdown behavior
- avoid large framework migration during the same round

Required outcomes:

- focused tests cover update cleanup and re-subscription paths
- focused tests cover notifier stop/cancel behavior
- tests added in this round stay close to the changed packages instead of
  attempting a repo-wide mock strategy overhaul

---

## 7. Planned Sequence

1. Stabilize `PUT` semantics and define the effective replacement invariants.
2. Implement update-path cleanup and re-subscription behavior.
3. Move notifier scheduler ownership under app/service lifecycle control.
4. Add targeted tests for update reconciliation and notifier shutdown.
5. Run the minimum verification set for the touched code.

Recommended commit grouping:

- `fix(processor): reconcile subscription update side effects`
- `fix(service): own notifier lifecycle from app shutdown`
- `test(processor): add update and notifier lifecycle coverage`

These subjects are examples, not mandatory final commit messages.

---

## 8. Acceptance Criteria

Round 1 is considered complete when all of the following are true:

1. `PUT /subscriptions/{subscriptionId}` behaves as full replacement.
2. Update no longer leaves stale external collection or model side effects from
   the pre-update subscription.
3. Periodic notification schedulers are stopped through app termination, not
   only through per-subscription mutation paths.
4. Targeted tests prove the changed behavior at the affected seams.
5. Verification notes clearly distinguish what was run from what remains out of
   scope.

---

## 9. Deferred Follow-Up

The following items remain intentionally outside round 1 and should be planned
later:

- shared app-boundary reconstruction
- broader handler/error-contract normalization
- post-subscription activation and late-failure signaling design
- logging cleanup
- config/model/repo-scope cleanup
- free5GC integration-level decisions
- Daisy/MTLF-local workflow redesign

---

## 10. Completion Record

Round 1 was completed with the following outcomes:

1. `PUT /subscriptions/{subscriptionId}` now behaves as full replacement in the
   local implementation path.
2. Update reconciles existing scheduler, SMF/data-collection resources, and the
   current local ML-model state before recreating the effective subscription.
3. Periodic notification schedulers are now rooted in the app cancellation
   context and are stopped during application termination.
4. Focused tests were added for:
   - update-time default validation re-application
   - update-time external state reconciliation
   - notifier cancellation through parent context
   - application-side scheduler shutdown fan-out

Verification completed on 2026-06-22:

- `make build`
- `go test ./...`
- `make lint`

Verification result:

- build passed
- full Go test suite passed
- `golangci-lint` reported `0 issues`
- the verified round-1 code was merged to `NWDAF/master` and pushed to `origin/master`

Round-1 intentionally did not include:

- shared app-boundary reconstruction
- broader SBI error-contract normalization
- post-subscription activation and late-failure signaling design
- Daisy/MTLF-local callback redesign
