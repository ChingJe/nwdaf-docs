# NWDAF Main Project Remediation Batches

Date: 2026-06-18
Last focused reassessment against current `NWDAF/` code: 2026-07-15
(`Priority 11`; other work items retain their recorded status dates)

This document reorganizes the scan findings into a stricter execution order.
The new ordering is based on three criteria:

- direct user-visible correctness risk
- runtime/lifecycle risk during operation or shutdown
- dependency order for safer later refactors

The larger original batches are split below into smaller work items so the next
implementation phase can progress with narrower, reviewable changes.

## 2026-06-23 Reassessment

This document was rechecked against the current `NWDAF/` tree after the
documented round-1 work and the later Priority-3 test refactor completion.

Current conclusions:

- Priority 3 is materially closed in code, not only in planning notes. The
  repository now carries direct `api_*_test.go` handler coverage plus active
  `gock` and `gomock` usage at the intended seams.
- Priority 2 no longer blocks the next refactor round. The main scheduler
  ownership issue is fixed; the remaining lifecycle cleanup is narrower and can
  be absorbed while touching adjacent service/server paths.
- The previously-next SBI error-contract round is now closed for the covered
  standards-facing scope and no longer blocks narrower factory work.
- The factory/config hardening round is now complete in code, including the
  explicit validation pass, SBI helper normalization, runtime-truth cleanup,
  and focused config test matrix.
- The next confirmed architectural cleanup target is now the fragmented
  app-boundary/interface ownership and global-config-driven consumer wiring.

The refreshed ordering below therefore favors smaller contract/config cleanups
before the broader shared app-boundary reconstruction.
Historical priority numbers are retained in the section headings below; the
table and execution rule are the current ordering.

## 2026-06-24 Strict Reassessment Absorption

The later same-day strict review is now absorbed into this remediation file and
into:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

This document is now the canonical status and ownership map for those stricter
judgments.

The absorbed strict review distinguished three cases:

1. issues that were already written down and simply still open
2. issues in already-touched areas that were still not strict enough
3. issues that needed a clearer formal home in the current document set

The table below replaces the old standalone reassessment file as the primary
classification map.

| Reassessment ID | Topic | Disposition | Canonical Home | Current Status |
| --- | --- | --- | --- | --- |
| 1 | broader lifecycle cancellation and app-owned I/O context | existing open work | Priority 2 | still open as residual lifecycle hardening |
| 2 | remaining package-global config reads after Priority 4 | residual in completed line | Priority 4 residual follow-up | closed on 2026-06-24 by `NWDAF/` commit `a912581` |
| 3 | non-3GPP external clients bypass shared consumer/app ownership | newly formalized work item | Priority 4 follow-up in this file | not started |
| 4 | runtime config mixed with lab/workflow config | existing open work | Priority 9 | not started |
| 5 | standalone SBI service vs fuller free5GC NF lifecycle level | Phase 0 complete; Phase 1 functionally implemented with focused test follow-up; later phases remain | Priority 11 | NRF lifecycle plus OAuth/certificate support are implemented and functionally verified; OAuth-disabled and OAuth-enabled HTTP/H2C live gates passed; the Phase 1 detailed automated-test matrix still has focused follow-up; current NRF v1.4.5 dropping registered `nwdafInfo` remains an accepted NRF-side limitation; discovery and metrics remain later work |
| 6 | OpenAPI/model governance still incomplete | existing open work | Priority 10 | not started |
| 7 | logging boundary and payload hygiene | existing open work | Priority 7 | completed for current intended scope on 2026-06-29 by `NWDAF/` commit `2ff07af`; local verification passed and Priority 6 late-failure semantics remain separate by design |
| 8 | test ownership was still more package-local than surveyed free5GC baseline | newly formalized same-lineage follow-up | Priority 3 strict ownership follow-up | closed on 2026-06-24 by `NWDAF/` commit `0768839` |

## Progress Tracking

Use this table as the single progress snapshot for the remediation plan.
Current status reflects the latest documented implementation and verification
state in this document set.

| Current Rank | Tier | Work Item | Status | Notes |
| --- | --- | --- | --- | --- |
| 1 | A | Repair Subscription Update Correctness | Completed | Closed in round 1 on 2026-06-22; merged to `NWDAF/master` and pushed |
| 2 | A | Put Long-Running Work Under App Lifecycle Control | Completed for bounded lifecycle / ownership scope | 2026-06-26 implementation closed the remaining bounded Priority 2 inventory: runtime consumer parent-context propagation, DB/runtime helper ownership, async ownership classification within scope, and explicit cleanup-exception narrowing. Adjacent Priority 5, 6, 7, 10, 11, and 12 work remains separate by design |
| 3 | B | Build The Test Safety Net Around The Real Boundaries | Completed | Round 1 closed update/lifecycle basics; the broader test refactor completed on 2026-06-23, and the stricter shared mock-ownership follow-up in the same Priority 3 lineage landed on 2026-06-24 as `NWDAF/` commit `0768839` |
| 4 | B | Normalize SBI Error Contracts | Completed for covered scope | 2026-06-23 implementation landed for the reviewed standards-facing handlers; intentionally excluded callbacks remain future contract-governance work and no longer block the next round |
| 5 | C | Harden Factory And Runtime Config Behavior | Completed | Implemented on 2026-06-24; explicit config validation, SBI getter normalization, runtime-truth doc alignment, and config test matrix are now in code |
| 6 | B | Rebuild One Real App Boundary | Completed | 2026-06-24 Phase 1 and Phase 2 both landed in `NWDAF/`; shared app boundary, root-contract scope, and consumer test-seam ownership are now materially aligned with the intended free5GC-style shape |
| 7 | B | Normalize Non-3GPP External Client Ownership | Completed for current intended scope | Phase 1 `637235b` and Phase 2 `f026e77` both landed on 2026-06-25 in `NWDAF/`; ML/Daisy ownership is now consumer-driven, processor consumes already-owned clients, and the same implementation line also closed ADRF teardown/shutdown convergence follow-up findings |
| 8 | B | Clarify Post-Subscription Activation And Late-Failure Signaling | Not started | Design completeness and observability work, not an immediate correctness bug |
| 9 | B | Tighten Logging Boundaries | Completed for current intended scope | 2026-06-29 workspace implementation closed the planned boundary, payload-hygiene, message-shape, and hot-path verbosity cleanup in `NWDAF/` commit `2ff07af`; later Priority 6 signaling semantics remain intentionally separate |
| 10 | C | Establish OpenAPI / Model Governance | Not started | Generated/reference models already exist for some locally redefined payloads; governance should follow handler/config cleanup |
| 11 | C | Implement free5GC NRF, OAuth, Discovery, And Metrics Alignment | In progress; Phase 0 complete and Phase 1 functionally verified with focused test follow-up | Phase 0 provides the NRF registration lifecycle; Phase 1 adds registration-driven OAuth state, separate `nrfCertPem`, generated Access Token client ownership, token-protected deregistration, Events Subscription scope authorization, certificate workflow, focused/race/full repository checks, and OAuth-disabled/OAuth-enabled HTTP/H2C live proof; the remaining detailed automated regression cases are recorded in the Phase 1 plan and changes are pending their completion and commit; NRF v1.4.5 `nwdafInfo` persistence remains an accepted NRF-side limitation; discovery, heartbeat, and metrics remain later work |
| 12 | C | Separate Runtime Config From Lab / Workflow Config | Not started | Structural config-scope cleanup after factory hardening and boundary decisions |
| 13 | C | Clean Repo And Package Ownership Boundaries | Completed for the current intended scope | Phase 1 landed in `NWDAF/` baseline `9b343ef` on 2026-07-01; Phase 2 then landed the separate `sbi` / `anlf` / `mtlf` server-topology split in `NWDAF/` commits `0ddbf3c` and `b547727`, including owned auxiliary listeners, callback-URI ownership cleanup, and focused lifecycle/config regression coverage |
| 14 | B | Align HTTP Edge Shape And Flow Ownership | Completed for the current active tranche and current-stage auxiliary follow-up | Phase 1 landed in `NWDAF/` on 2026-07-06 as commit `e8e249a`; Phase 1.5 then landed in `NWDAF/` on 2026-07-06 across commits `8762b35` and `a0fff93`, completing the main-`SBI` HTTPS uplift and collector callback-ownership correction; the currently selected Phase 2 auxiliary structural-alignment scope then landed in `NWDAF/` on 2026-07-06 as commit `1b06411`, while deeper processor/business-logic consolidation remains future work |

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
- Completed portions:
  1. notifier schedulers now inherit the app cancellation context and are
     stopped explicitly during application termination
  2. the 2026-06-25 external-client ownership implementation line also landed
     adjacent parent-context hardening for covered external integration calls
  3. shutdown-time ADRF unsubscribe cleanup now uses an owner-created bounded
     cleanup context instead of depending on the already-canceled app context
  4. ADRF retrain workflow follow-up work is now waitgroup-visible rather than
     detached from the app lifecycle boundary
  5. shutdown no longer waits for the ADRF runtime watchdog before retrain
     cleanup can converge
- Remaining-plan implementation completed on 2026-06-26.
- Newly completed portions from the bounded remaining-work plan:
  1. SMF / MTLF / ML / Daisy / ADRF runtime consumer paths now expose
     caller-owned parent-context contracts instead of silently rooting runtime
     I/O at `context.Background()`
  2. MongoDB query helpers now derive bounded query timeouts from caller-owned
     contexts, and AnLF ground-truth lookup propagates monitor-owned
     cancellation into that path
  3. UPF persistence writes now derive from app-owned cancellation rather than
     a root background context
  4. request-triggered async follow-up in subscription data collection and ML
     model provisioning is now explicitly treated as bounded request-triggered
     work, and shutdown no longer expands new work through those paths
  5. the remaining production `context.Background()` usage in the touched
     Priority 2 scope is now limited to explicit cleanup / shutdown exceptions
  6. the implementation-completion review against `development_policy.md`
     found no new confirmed issue inside this bounded Priority 2 scope
- Boundary clarification for the still-open remainder:
  1. Priority 2 owns lifecycle/ownership cleanup, parent-context propagation,
     async ownership classification, and cleanup-exception classification
  2. post-acceptance activation semantics and late-failure signaling remain
     Priority 6 even where they touch the same async subscription paths
  3. callback contract/model redesign remains Priority 5 or Priority 10
  4. logging hygiene remains Priority 7
  5. fuller free5GC integration-level uplift such as completing the metrics
     collector/server lifecycle, NRF registration, OAuth/certificate support,
     discovery, and broader NF-lifecycle alignment remains Priority 11
  6. Daisy / MTLF callback relocation and broader package-boundary cleanup
     remain Priority 12
- The completed notifier-lifecycle portion, the later covered residual
  implementation, and the final bounded remaining-work implementation are all
  now landed in `NWDAF/`.
- The current detailed remaining-work plan now lives in:
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 2 Remaining Lifecycle Completion Plan.md`

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
- the narrower same-lineage strictness follow-up for shared app-boundary mock
  ownership and local mock placement was then implemented on 2026-06-24
- that follow-up landed as `NWDAF/` commit `0768839`
- `pkg/mockapp` now owns the shared generated app-boundary mock, while
  `internal/sbi` and `internal/sbi/processor` no longer own separate app mock
  shapes
- this means Priority 3 is now closed both for its primary safety-net purpose
  and for the stricter same-lineage mock-ownership cleanup, while the
  documented SMF raw-HTTP exception still remains later contract/model-
  governance work

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

Status update:

- Phase 1 completed in `NWDAF/` on 2026-06-24.
- `pkg/app` is now the canonical shared runtime contract.
- `consumer.NewConsumer()` is now app-driven and no longer uses
  `factory.NwdafConfig` as hidden constructor input.
- overlapping local app interfaces were collapsed onto the shared root seam.
- adjacent server lifecycle signature drift was cleaned up in the same round.
- Phase 2 also completed in `NWDAF/` on 2026-06-24.
- `CancelContext()` was removed from the root app contract and moved back to
  the smaller local seams that actually require cancellation.
- processor-side consumer ownership now depends on a narrower mockable seam
  instead of the concrete consumer type.
- the exported consumer test-assembly path was removed from the public package
  API.

### Priority 4 Follow-Up — Normalize Non-3GPP External Client Ownership

Original gap at formalization time:

- Daisy and ML service clients were still instantiated directly inside
  `internal/mtlf` and `internal/anlf`
- non-3GPP external integrations still bypassed the shared app/service/consumer
  ownership model that now exists for the main SBI consumer path

Sub-items:

1. define the intended ownership rule for non-3GPP external clients after the
   completed Priority 4 boundary reconstruction
2. move Daisy and ML service client construction behind an app-owned or
   consumer-owned boundary rather than direct domain-local instantiation
3. keep the distinction explicit between:
   - standardized or semi-standard NF interaction under `internal/sbi/consumer`
   - project-local external integration clients
4. update tests and mocks so the resulting ownership path follows the same
   app-boundary injection story used by the surveyed free5GC control-plane NFs

Why here:

- the surveyed free5GC control-plane NFs construct peer-facing clients from
  the app/service/consumer boundary rather than instantiating them ad hoc
  inside domain logic
- after Priority 4, NWDAF now has the shared app boundary needed to clean this
  up without reopening the whole boundary reconstruction
- this is narrower and more directly free5GC-alignment-relevant than the later
  repository/package cleanup in Priority 12

Status update:

- Phase 1 implemented in `NWDAF/` on 2026-06-25 as local commit `637235b`
- Phase 2 then implemented in `NWDAF/` on 2026-06-25 as local commit `f026e77`
- AnLF and MTLF no longer instantiate ML and Daisy clients directly inside
  runtime logic
- ML and Daisy client assembly now lives in
  `internal/sbi/consumer.NewConsumer(...)`
- processor now consumes already-owned external clients instead of deciding
  client construction itself
- the same implementation line also closed the narrower same-round findings on:
  1. shutdown-time ADRF unsubscribe cleanup context ownership
  2. retrain-workflow shutdown convergence previously tied to the runtime
     watchdog
- focused and full repository verification passed after the covered
  implementation rerun
- this work item should now be treated as completed for the current intended
  standalone-NWDAF alignment scope
- any broader lifecycle cancellation or app-owned I/O context cleanup that
  remains open stays under Priority 2 rather than reopening this Priority 4
  follow-up
- the canonical implementation and reassessment record now lives in:
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan.md`

### Priority 5 — Normalize SBI Error Contracts

Scope:

- SBI handlers mix `ProblemDetails` and ad hoc `gin.H` error bodies
- standards-facing handlers do not yet follow one verified free5GC-style
  problem-response path
- parse/model alignment and response-shape alignment are currently entangled
- Daisy local callbacks should not define the official-alignment target

Sub-items:

1. Re-baseline against actual free5GC NF implementations before further code
   changes.
   Confirm the real role of:
   - shared `openapi.ProblemDetails...` constructors
   - NF-local `internal/util/GinProblemJson(...)` writers
   - `GetRawData() + openapi.Deserialize(...)` versus Gin binding
2. Execute Priority 5A first.
   Normalize the outward `ProblemDetails` contract and media type for
   standards-facing handlers without introducing new `internal/sbi`-local
   abstractions.
3. Execute Priority 5B second.
   Revisit parse/model alignment endpoint by endpoint only where the current
   dependency snapshot has an exact or clearly intended generated model.
4. Keep Daisy callback cleanup secondary.
   It may still need server-wide consistency, but it should not drive the
   free5GC alignment baseline.
5. Update tests after the verified target contract is fixed.

Why here:

- This cleanup is easier once handler tests exist and before larger contract
  governance work starts.
- After the re-baseline, the immediate next round is narrower than originally
  framed: fix the outward `ProblemDetails` path first, then decide which parse
  boundaries should move closer to `openapi.Deserialize(...)`.

Status update:

- The covered standards-facing Phase A and Phase B work is implemented in the
  current `NWDAF/` tree on 2026-06-23.
- Handler-generated error bodies across the reviewed `internal/sbi/api_*.go`
  endpoints now converge on `models.ProblemDetails`.
- problem responses are now written through an NF-local
  `internal/util/GinProblemJson(...)` path that sets
  `application/problem+json`.
- the earlier custom malformed-JSON cause drift was removed so malformed
  request failures align more closely with the shared
  `openapi.ProblemDetailsMalformedReqSyntax(...)` pattern.
- create/update subscription handlers and the covered callback handlers now use
  `GetRawData() + openapi.Deserialize(...)` where the current dependency
  snapshot provides an exact or clearly intended model path.
- direct handler tests were updated and focused verification passed.
- intentionally excluded callbacks, such as `upf-notify`, Daisy local callback
  cleanup, and an exact top-level ADRF retrieval callback model, remain future
  contract/model-governance work rather than unfinished Priority 5 work.

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

Current status:

- completed for the current intended Priority 7 scope on 2026-06-29 by
  `NWDAF/` commit `2ff07af`
- implemented cleanup covered the main touched handler, processor, consumer,
  notifier, context, AnLF, MTLF, and ADRF-retrieval paths
- local verification passed with `make build`, `make lint`, and
  `go test ./...`
- this closure does not claim that Priority 6 post-acceptance signaling
  semantics were resolved here

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

Status update:

- Completed on 2026-06-24.
- `pkg/factory/config.go` now applies defaults, runs explicit validation, and
  rejects contradictory config at startup.
- `GetSbiBindingAddr()` was fixed and SBI helper usage was normalized through
  binding/register URI getters at the current server and callback call sites.
- `supportedAnalytics`, sample config, and `README.md` now reflect the current
  `UE_COMMUNICATION` runtime truth and documented environment requirements.
- focused config validation tests were added and verification passed with:
  `go test ./pkg/factory`, `go test ./...`, `make build`, and `make lint`.

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

### Priority 11 — Implement free5GC NRF, OAuth, Discovery, And Metrics Alignment

Scope:

- the architectural direction is now decided: NWDAF should align upward as an
  NRF-managed free5GC-style NF
- Phase 0 NRF registration/deregistration is complete; Phase 1
  OAuth/certificate support is implemented and functionally verified in the
  current working trees, with focused automated-test follow-up still recorded;
  NRF-backed NF discovery is not yet wired
- live verification against free5GC `main` commit `f64135d`/NRF v1.4.5 proves
  create/delete lifecycle and default PLMN handling, but the NRF data-model
  copy path drops registered `nwdafInfo`; this is accepted as a Phase 0
  deployment limitation while NWDAF continues sending the correct field and a
  future NRF-side correction is pursued
- existing-resource replacement returned `200`; unavailable-NRF recovery kept
  all owned listeners closed through four failed attempts and opened them after
  attempt 5 succeeded; forced post-registration AnLF bind failure completed
  DELETE, closed the partial SBI listener, exited nonzero, and left the NRF
  resource absent
- the existing main SBI HTTP/HTTPS and `metrics.InboundMetrics()` support from
  Priority 14 should be preserved and completed with collector initialization,
  outbound client hooks, a separately owned `/metrics` listener, configuration,
  and lifecycle management

Sub-items:

1. Implement an NRF NFManagement baseline:
   - build the NWDAF NF profile from validated runtime config and context
   - register at startup and deregister during bounded shutdown
   - advertise only implemented standardized services and analytics events
   - require `nrfUri` but keep retryable NRF unavailability alive with owned
     listeners unstarted, retrying with cancellation-aware backoff until
     success or cancellation
   - use `nfServices`, omit request `plmnList` for the same-PLMN baseline, rely
     on NRF `defaultPlmnId`, and follow the current free5GC `apiPrefix` shape
   - register before starting listeners, roll back post-registration listener
     failures with bounded deregistration, reject Phase 0 redirects and
     OAuth-required operation explicitly, and deregister before stopping SBI
2. Add OAuth and NRF-certificate support:
   - consume the registration response's OAuth indication
   - obtain tokens for protected outbound NRF operations
   - protect the appropriate inbound standardized producer routes
   - keep OAuth-disabled operation covered explicitly
   - implemented and functionally verified on 2026-07-15; focused automated
     regression follow-up and commit remain pending
3. Add NRF discovery as a separately gated phase, beginning with SMF lookup,
   and decide how discovery interacts with currently configured peer endpoints.
4. Complete the common free5GC metrics runtime as an independent workstream:
   - add optional `configuration.metrics` with the usual enable, scheme,
     binding, port, namespace, and TLS fields
   - initialize inbound/outbound SBI collectors and attach outbound generated
     client hooks
   - start and stop the separate metrics server under app lifecycle ownership
   - do not make this default-disabled runtime a prerequisite for the NRF,
     OAuth, or discovery phase gates
5. Keep registration, OAuth, discovery, and metrics as separate commits and
   verification gates even if some are developed in one round.
6. Keep TS 29.510 NFUpdate heartbeat as an explicit, non-blocking deferred
   standards gap. Current free5GC `main`/NRF v1.4.5 has partial NRF PATCH
   support but no surveyed NF heartbeat sender or complete NRF
   timer/expiry/suspension lifecycle; revisit it through a separate plan after
   upstream coordination.

Why here:

- The direction is no longer an open architecture vote, but the implementation
  changes startup, shutdown, trust, authorization, and peer-selection behavior
  and therefore needs its own staged completion gates.
- NRF is the main implementation objective because registration and discovery
  establish the runtime basis for adding later standardized interfaces;
  metrics remains required alignment work but is not an NRF dependency.

Detailed implementation plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 NRF OAuth Discovery And Metrics Alignment Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 Phase 0 NRF NFManagement Detailed Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 Phase 1 OAuth And NRF Certificate Detailed Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 Metrics Runtime Supporting Workstream Plan.md`

### Priority 12 — Clean Repo And Package Ownership Boundaries

Scope:

- historical Phase 1:
  - retrain workflow support tooling left `NWDAF/` for the sibling
    `nwdaf-resources/` repo
  - inference-engine and Daisy-specific production ownership left
    `internal/sbi` for `internal/anlf` and `internal/mtlf`
- implemented Phase 2:
  - the main SBI server no longer carries non-SBI ingress semantics through
    shared Gin route injection
  - `AnLF` and `MTLF` now each own their own auxiliary inbound HTTP server
  - callback URIs for the touched flows now derive from owned auxiliary-server
    config instead of the main SBI listener

Detailed implementation plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 12 Domain Boundary And Support Asset Cleanup Plan.md`

Sub-items:

1. Preserve the landed Phase 1 baseline as the starting point for the server-
   topology round.
2. Split the shared inbound HTTP surface into:
   - a main NWDAF SBI server
   - an `AnLF` auxiliary server
   - an `MTLF` auxiliary server
3. Remove shared-Gin late route injection for Daisy and inference-related
   ingress.
4. Derive callback URIs from owned `AnLF` / `MTLF` server config rather than
   assuming the main SBI listener.
5. Add focused lifecycle and config tests for listener startup, listener
   cleanup, and callback-URI derivation.

Why here:

- This remained valuable and invasive, but the codebase had a cleaner baseline
  from which the server-topology phase could proceed deliberately instead of
  mixing it with the earlier ownership move.

Status update:

- Phase 2 implemented in `NWDAF/` on 2026-07-01 as commits `0ddbf3c` and
  `b547727`.
- The implementation now gives `pkg/service` explicit lifecycle ownership of
  the main SBI listener plus the `AnLF` and `MTLF` auxiliary listeners.
- `internal/sbi` no longer exposes `Router()`-style late route injection for
  these non-SBI ingress paths.
- The follow-up cleanup commit removed the stale `externalMtlf.notifUri`
  override path so external MTLF callbacks now derive only from
  `configuration.anlf.server`.
- Verification reran `go test ./...`, `make build`, and `make lint`; all
  passed.

### Priority 14 — Align HTTP Edge Shape And Flow Ownership

Scope:

- the completed Priority 12 split fixed HTTP transport meaning, but the current
  server implementations still do not present one stable post-split code shape
- `internal/sbi` should align more closely with the chosen local `pcf`
  free5GC server skeleton without reopening the semantic server-boundary split
- `internal/anlf` and `internal/mtlf` should align to one consistent auxiliary
  flow shape:
  - `api -> processor -> client`
- `collector` remains part of the main `internal/sbi` surface because it is an
  outward NWDAF callback / collection edge for peer NF traffic

Detailed implementation plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan.md`

Sub-items:

1. Align `internal/sbi` router, middleware, and HTTP-server assembly toward the
   chosen local `pcf` shape.
2. Preserve the current `api -> processor -> consumer` story in `internal/sbi`
   and keep `collector` on the main server.
3. Introduce explicit auxiliary HTTP-edge ownership in `internal/anlf` and
   `internal/mtlf` so callback parsing and transport response shaping are no
   longer owned directly by service structs.
4. Normalize auxiliary outbound integration naming to `client` while keeping
   `consumer` for standards-facing service consumption.
5. Keep all three servers similar in engineering form, but do not introduce a
   shared generic base HTTP server abstraction.
6. Preserve current `startOwnedServers()` rollback semantics during the Phase 1
   `internal/sbi` skeleton alignment by using a thin synchronous preflight
   bind check before the normal `ListenAndServe()` path.

Why here:

- Priority 12 fixed the semantic boundary question first. The next narrower
  cleanup is to make the resulting three-server design read as one deliberate
  repository style instead of three independently evolved HTTP-edge shapes.

Status update:

- Phase 1 implemented in `NWDAF/` on 2026-07-06 as commit `e8e249a`
- landed Phase 1 aligned `internal/sbi` router/middleware/server assembly to a
  `pcf`-style shape using `logger_util.NewGinWithLogrus(...)`,
  `metrics.InboundMetrics()`, and `httpwrapper.NewHttp2Server(...)`
- the final landed startup contract preserves a synchronous preflight bind
  check and adds a short startup readiness handshake so
  `startOwnedServers()` again means the SBI listener is actually accepting
  connections
- verification reran `go test ./...`, `make build`, and `make lint`; all
  passed
- remaining open scope is Phase 2 auxiliary-server alignment for `internal/anlf`
  and `internal/mtlf`

## Suggested Execution Rule

Do not combine Priorities 1 to 5 in one code change.

Recommended sequence:

1. Priority 1
2. Priority 2
3. Priority 3
4. Priority 5
5. Priority 8
6. Priority 4
7. Priority 4 follow-up for non-3GPP external client ownership
8. Priority 6
9. Priority 7
10. Priority 10
11. Priority 11
12. Priority 9
13. Priority 12
14. Priority 14

Compressed rule of thumb:

- first stop the system from doing the wrong thing
- then make runtime behavior stoppable and owned
- then build the test seam
- then normalize the externally visible HTTP contract
- then remove obvious factory/config drift and latent helper bugs
- then consolidate the real app boundary
- only after that tackle later-failure semantics, logging, governance, repo
  scope, and strategic free5GC alignment
