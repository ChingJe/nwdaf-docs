# NWDAF Main Project Remediation Batches

Date: 2026-06-18

This document reorganizes the scan findings into a stricter execution order.
The new ordering is based on three criteria:

- direct user-visible correctness risk
- runtime/lifecycle risk during operation or shutdown
- dependency order for safer later refactors

The larger original batches are split below into smaller work items so the next
implementation phase can progress with narrower, reviewable changes.

## Progress Tracking

Use this table as the single progress snapshot for the remediation plan.
Current status reflects the latest documented implementation and verification
state in this document set.

| Priority | Tier | Work Item | Status | Notes |
| --- | --- | --- | --- | --- |
| 1 | A | Repair Subscription Update Correctness | Completed | Closed in round 1 on 2026-06-22; merged to `NWDAF/master` and pushed |
| 2 | A | Put Long-Running Work Under App Lifecycle Control | Partially completed | Notifier lifecycle ownership closed in round 1 and pushed; broader lifecycle cleanup still remains |
| 3 | B | Build The Test Safety Net Around The Real Boundaries | Completed | Round 1 closed update/lifecycle basics; the broader test refactor completed on 2026-06-23 with direct handler coverage, normalized consumer test seams, expanded gomock-based processor seams, and one documented SMF raw-HTTP exception that remains deferred to later contract/model-governance work |
| 4 | B | Rebuild One Real App Boundary | Not started | Consolidate shared app contract after initial behavior stabilizes |
| 5 | B | Normalize SBI Error Contracts | Not started | Align error bodies and parse handling across endpoints |
| 6 | B | Clarify Post-Subscription Activation And Late-Failure Signaling | Not started | Design completeness and observability work, not an immediate correctness bug |
| 7 | B | Tighten Logging Boundaries | Not started | Follow error-contract and late-failure signaling cleanup |
| 8 | C | Harden Factory And Runtime Config Behavior | Not started | Includes config validation and `GetSbiBindingAddr()` fix |
| 9 | C | Separate Runtime Config From Lab / Workflow Config | Not started | Structural config-scope cleanup |
| 10 | C | Establish OpenAPI / Model Governance | Not started | Clarify generated vs handwritten model ownership |
| 11 | C | Decide The Intended free5GC Integration Level | Not started | Architectural scope decision |
| 12 | C | Clean Repo And Package Ownership Boundaries | Not started | Most invasive cleanup; scheduled last |

## Tier A — Immediate Correctness And Runtime Risk

### Priority 1 — Repair Subscription Update Correctness

Scope:

- `PUT /subscriptions/{id}` does not reconcile SMF, ADRF, traffic-bucket, and
  ML-model side effects
- create/update notification-method/default handling is inconsistent

Sub-items:

1. Define update semantics explicitly.
   Treat `PUT` as full replacement of the subscription resource, not a
   constrained merge update.
2. Align effective request semantics.
   Re-run the same defaulting and notification-method resolution used by
   `POST`.
3. Reconcile external collection state.
   Clean up and recreate SMF subscriptions, correlation mappings, traffic
   buckets, ADRF state, and MTLF/ML state when the effective shape changes.
4. Add update-path proof tests.
   Add focused processor tests that prove cleanup, re-subscription, and failure
   handling.

Why here:

- This is the clearest confirmed correctness bug in the current codebase.

Status update:

- Completed in round 1 on 2026-06-22.
- `PUT` now re-applies default/notification validation, reconciles prior
  scheduler and external collection state, and re-triggers downstream
  collection for the replacement subscription shape.
- Focused update-path tests were added and full verification passed.
- The round-1 implementation was merged locally to `NWDAF/master` and pushed to
  `origin/master`.

### Priority 2 — Put Long-Running Work Under App Lifecycle Control

Scope:

- periodic notifier scheduler is detached from app shutdown
- some run/shutdown signatures ignore passed context
- background worker ownership is inconsistent across notifier, SBI server, AnLF,
  and MTLF-related paths

Sub-items:

1. Give notifier work an app-owned cancellation path.
2. Stop active subscription schedulers during termination.
3. Remove `context.Background()` ownership from runtime-managed long-lived work
   where app context should apply.
4. Make goroutine ownership observable.
   Register scheduler/background work under the app waitgroup or an equivalent
   owned mechanism.
5. Clean up ignored lifecycle parameters such as unused `traceCtx`.

Why here:

- This stabilizes shutdown behavior before test expansion and app-boundary
  refactors.

Status update:

- Partially completed in round 1 on 2026-06-22.
- Notifier schedulers now inherit the app cancellation context and are stopped
  explicitly during application termination.
- Broader lifecycle cleanup listed in this batch, such as remaining unused
  lifecycle parameters and cross-component ownership normalization, is still
  pending.
- The completed notifier-lifecycle portion is already merged to
  `NWDAF/master` and pushed.

## Tier B — Safety Net And Boundary Consolidation

### Priority 3 — Build The Test Safety Net Around The Real Boundaries

Scope:

- outbound SBI tests do not follow the preferred free5GC mock pattern
- internal dependency mocking is weak
- handler and lifecycle seams are under-tested

Sub-items:

1. Add `gock` for outbound peer-NF consumer tests.
2. Add `gomock` at the app/interface seam instead of handwritten stubs.
3. Add handler-level tests for parsing, status codes, and error bodies.
4. Add notifier/lifecycle tests for cancellation, timeout, and stop behavior.
5. Add H2C interception/restoration where the free5GC client path needs it.
6. Keep Python fixtures explicitly classified as integration helpers.

Why here:

- Later refactors should not proceed without first improving the test seam that
  protects them.

Status update:

- Completed across round 1 plus the final follow-up implementation on
  2026-06-23.
- Focused tests were added for update-time default validation, update-time
  state reconciliation, notifier parent-context cancellation, and
  application-side scheduler shutdown.
- Direct handler coverage now exists across all `internal/sbi/api_*.go` files.
- `gock` and `gomock` are now in active use across the intended high-value
  consumer, handler, and processor seams.
- direct ADRF consumer coverage was added
- ML service and Daisy consumer tests were normalized to the same deliberate
  outbound-mocking strategy
- remaining high-value handwritten processor/app stubs were replaced by
  `gomock` seams
- SMF remains a documented raw-HTTP exception because the current local
  `openapi` dependency snapshot still does not safely model the required
  request fields for the existing UPF event workflow
- this exception is treated as follow-up contract/model-governance work, not as
  unfinished Priority 3 test work

### Priority 4 — Rebuild One Real App Boundary

Scope:

- `pkg/app` is under-specified
- internal packages re-declare overlapping app interfaces
- consumer construction is detached from the shared boundary

Sub-items:

1. Expand `pkg/app` into the actual contract used by service, server,
   processor, consumer, AnLF, and MTLF.
2. Collapse overlapping local `NwdafApp` interface variants.
3. Move consumer construction toward an app-driven form such as
   `NewConsumer(app.App)`.
4. Align mock/test setup with the new boundary immediately after the interface
   is stabilized.

Why here:

- This is the smallest point at which the repo can stop fighting itself on
  dependency injection and mocking.

### Priority 5 — Normalize SBI Error Contracts

Scope:

- SBI handlers mix `ProblemDetails` and ad hoc `gin.H` error bodies
- `BindJSON` and `ShouldBindJSON` usage is inconsistent
- callback/API failure semantics are not uniform

Sub-items:

1. Standardize parse/procedure error response shape.
2. Replace `BindJSON` where explicit response control is required.
3. Align callback endpoints with the chosen synchronous/asynchronous contract.
4. Add tests that lock the response shape and status code behavior.

Why here:

- This cleanup is easier once handler tests exist and before larger contract
  governance work starts.

### Priority 6 — Clarify Post-Subscription Activation And Late-Failure Signaling

Scope:

- accepted analytics subscriptions may trigger downstream data collection after
  the subscription resource is already created
- late activation failures are not expressed clearly through local state,
  response wording, or spec-aligned failure signaling
- local Daisy/MTLF workflow callbacks are currently non-standard integration
  paths and should not drive early remediation sequencing

Sub-items:

1. Define resource-acceptance versus downstream-activation semantics for
   `POST /subscriptions` and the later `PUT` reconcile path.
2. Decide when `failEventReports` is sufficient and when later
   notification/status signaling should be used for post-acceptance failures.
3. Add a minimal local observability rule for “accepted but activation degraded
   or failed later” paths.
4. Keep Daisy/MTLF-local callback contract redesign explicitly out of this
   batch unless the local workflow itself is being reworked.

Why here:

- This is a design completeness and operability issue, not the same class as
  the broken `PUT` path or shutdown-ownership bugs.
- It should follow the earlier correctness, lifecycle, and test-stabilization
  work.

### Priority 7 — Tighten Logging Boundaries

Scope:

- logger categories exist, but payload hygiene and fatal/worker boundaries are
  loose

Sub-items:

1. Remove or downgrade raw outbound payload dumps unless clearly justified.
2. Review fatal process-exit logging in worker/background control paths.
3. Keep one meaningful log boundary per failure path where possible.
4. Re-check sensitive or high-cardinality data in logs after error contracts and
   late-failure signaling are stabilized.

Why here:

- This depends partly on the error-contract and late-failure signaling
  decisions above, so it should follow them rather than precede them.

## Tier C — Configuration, Contract Governance, And Scope Decisions

### Priority 8 — Harden Factory And Runtime Config Behavior

Scope:

- config parsing lacks strong validation
- `GetSbiBindingAddr()` is broken
- runtime defaults and documented runtime truth drift apart

Sub-items:

1. Add explicit config validation similar to free5GC NF factories.
2. Fix concrete helper bugs such as `GetSbiBindingAddr()`.
3. Align runtime defaults with actual supported analytics/event behavior.
4. Align README/runtime guidance with `go.mod` and current startup truth.
5. Add focused config test matrices for valid, missing, and conflicting cases.

Why here:

- These are important and partly bug-level, but they are safer after the
  correctness/runtime path and test seams are stabilized.

### Priority 9 — Separate Runtime Config From Lab / Workflow Config

Scope:

- main config currently mixes deployable NF config with local experiment,
  fallback, Daisy, and ADRF workflow data

Sub-items:

1. Define which fields are true runtime NF config.
2. Define which fields are local lab data, test fixture data, or workflow
   payload templates.
3. Split or re-home those boundaries intentionally.
4. Keep migration impact explicit for existing local setups.

Why here:

- This is structurally large and should not be mixed with the earlier
  bug-fixing work.

### Priority 10 — Establish OpenAPI / Model Governance

Scope:

- handwritten standardized models and release-extended payloads have no clean
  governance rule

Sub-items:

1. Replace locally duplicated standardized payloads with generated models where
   already available.
2. Mark which handwritten models are temporary dependency-snapshot extensions.
3. Define regeneration inputs/version/options for future Release 18/19 work.
4. Separate local extension layers from standardized contract layers.

Why here:

- This becomes much easier once handler contracts and tests are already in
  place.

### Priority 11 — Decide The Intended free5GC Integration Level

Scope:

- missing NRF registration/deregistration
- missing metrics server wiring
- lightweight SBI server path versus reference NF baselines

Sub-items:

1. Decide whether NWDAF remains a standalone SBI service or aligns upward as a
   normal free5GC-style NF.
2. If aligning upward, wire `nrfUri`, register/deregister flow, metrics server,
   and closer HTTP/TLS/auth patterns.
3. If staying standalone, document the divergence explicitly and stop treating
   the gap as accidental.

Why here:

- This is a deliberate architectural scope decision and should not be hidden
  inside lower-level cleanups.

### Priority 12 — Clean Repo And Package Ownership Boundaries

Scope:

- main repo carries auxiliary docs and Python fake peers
- `internal/sbi` owns Daisy-specific workflow glue that should likely sit
  behind `internal/mtlf`

Sub-items:

1. Decide what remains in the main repo versus a dedicated support/resource
   repository.
2. Move Daisy/MTLF-specific ownership behind the correct domain package while
   keeping only the minimal HTTP binding at the SBI edge.
3. Clean up leftover package placement that exists only because earlier
   boundaries were blurred.

Why here:

- This is valuable, but it is the most invasive and least urgent once the
  runtime, contract, and test risks above are addressed.

## Suggested Execution Rule

Do not combine Priorities 1 to 5 in one code change.

Recommended sequence:

1. Priority 1
2. Priority 2
3. Priority 3
4. Priority 4
5. Priority 5
6. Priority 6
7. Priority 7
8. Priority 8
9. Priority 9
10. Priority 10
11. Priority 11
12. Priority 12

Compressed rule of thumb:

- first stop the system from doing the wrong thing
- then make runtime behavior stoppable and owned
- then build the test seam
- then clean up contracts and later-failure signaling
- then consolidate boundaries
- only after that clean up config, contracts, repo scope, and strategic
  free5GC alignment
