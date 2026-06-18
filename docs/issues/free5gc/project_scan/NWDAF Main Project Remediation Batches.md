# NWDAF Main Project Remediation Batches

Date: 2026-06-18

This document groups the first scan findings into execution batches. The order
is chosen to reduce behavior risk before tackling broader structural cleanup.

## Batch 1 — Fix Subscription Correctness First

Target findings:

- update path does not reconcile SMF/ML side effects
- create/update notification-method behavior is inconsistent

Concrete goals:

- decide whether `PUT` is full replacement or constrained update
- make `PUT` re-run the same effective notification-method/default logic as
  `POST`
- explicitly reconcile existing SMF subscriptions, correlation mappings, traffic
  buckets, ADRF state, and ML-model provisioning when the request changes the
  effective collection shape
- add processor tests that prove update-time cleanup and re-subscription

Why first:

- This is the clearest confirmed correctness bug in the current codebase.

## Batch 2 — Put All Long-Running Work Under App Lifecycle Control

Target findings:

- periodic notifier scheduler is detached from app shutdown
- server run/shutdown signatures ignore passed context

Concrete goals:

- give periodic notification work an app-owned cancellation path
- make scheduler goroutines observable by the app waitgroup or an equivalent
  owned shutdown mechanism
- stop all active subscription schedulers during `terminateProcedure()`
- remove or repurpose unused lifecycle parameters such as the ignored `traceCtx`

Why second:

- This batch reduces shutdown ambiguity before larger refactors change package
  boundaries.

## Batch 3 — Rebuild The App Boundary Around One Real Contract

Target findings:

- `pkg/app` is under-specified
- internal packages re-declare overlapping app interfaces
- consumer construction is global-config/global-context oriented

Concrete goals:

- expand `pkg/app` so it represents the actual app boundary used by service,
  server, processor, consumer, and domain packages
- reduce the number of local `NwdafApp` interface variants
- move consumer construction toward `NewConsumer(app.App)` or a similarly
  explicit app-driven form
- keep the change mechanical and avoid mixing unrelated business rewrites

Why third:

- After subscription correctness and lifecycle ownership are fixed, the app
  contract can be tightened with lower regression risk.

## Batch 4 — Harden Factory, Config, And Documented Runtime Truth

Target findings:

- config parsing lacks strong validation
- `GetSbiBindingAddr()` is broken
- advertised supported analytics defaults contradict runtime support
- README Go version is stale

Concrete goals:

- add explicit config validation similar to free5GC NF factories
- fix `GetSbiBindingAddr()` before it becomes a used dependency
- align `supportedAnalytics` defaults with actual supported runtime events
- align README/runtime docs with `go.mod`
- add focused config tests for invalid/missing combinations
- decide which config content is true NF runtime config versus local lab or
  external-system workflow data, then split those boundaries intentionally

Why fourth:

- This batch is important, but it is safer after the runtime behavior bugs are
  contained.

## Batch 5 — Decide The Intended free5GC Integration Level

Target findings:

- missing NRF registration/deregistration path
- missing metrics server wiring
- lightweight SBI server path compared with free5GC baselines

Decision point:

- either explicitly keep NWDAF as a standalone SBI service and document that
  scope
- or close the lifecycle gap and implement the normal free5GC NF service hooks

If aligning upward:

- wire `nrfUri` into consumer/service startup
- implement register/deregister behavior
- add metrics config and metrics server lifecycle
- evaluate whether TLS/auth middleware should follow nearby free5GC NF patterns

Why fifth:

- This is a bigger architectural decision than the earlier corrective batches.

## Batch 6 — Contract Cleanup, Test Strategy, And Schema Traceability

Target findings:

- outbound SBI tests do not follow the preferred free5GC mock pattern
- handwritten standardized or release-extended payload models need a clearer
  contract strategy

Concrete goals:

- add `gock`-based consumer tests where the code is mocking peer NF behavior
- use `openapi.InterceptH2CClient()` when tests need the free5GC client path
- keep Python or multi-process fixtures classified as explicit integration
  helpers rather than substitutes for unit coverage
- replace local standardized structs with generated models where the workspace
  already has them
- document any places where local schemas intentionally extend the current
  dependency snapshot
- separate “dependency snapshot limitation” from “local ad hoc model”

## Batch 7 — Clean Repo And Package Ownership Boundaries

Target findings:

- main repo carries auxiliary docs and Python fake peers
- `internal/sbi` owns Daisy callback HTTP glue that is really MTLF workflow
  behavior
- config boundary is overloaded with lab-specific and external-workflow state

Concrete goals:

- decide which auxiliary docs, fake peers, and workflow helpers stay in the
  main repo versus move to a dedicated resource/support repo
- move non-3GPP Daisy-specific ownership behind `internal/mtlf` while keeping
  only necessary HTTP binding glue at the SBI edge
- keep NF runtime config minimal and explicitly separate deployable config from
  local experiment data
- avoid mixing this batch with runtime-correctness changes

## Suggested Execution Rule

Do not combine Batches 1 to 3 in one code change.

Recommended order:

1. Batch 1
2. Batch 2
3. Batch 3
4. Batch 4
5. Batch 5
6. Batch 6
7. Batch 7

This ordering keeps the early work focused on correctness and lifecycle, then
uses that stabilized base for structural cleanup.
