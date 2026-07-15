# NWDAF Main Project free5GC Alignment Scan Findings

Date: 2026-06-18

## Scope

This scan reviewed the main `NWDAF/` repository against:

- `free5gc-dev-skill/SKILL.md` and its routed review references
- local free5GC reference NFs under `resources/references/free5gc-main/NFs/`
- the workspace NWDAF development policy

The goal of this pass was to identify confirmed implementation issues, structural
alignment gaps, and verification gaps before any refactor work.

## Verification Run

Executed in `NWDAF/`:

- `go test ./...`
- `make build`
- `make lint`

Result:

- All current Go tests passed.
- Build passed.
- `golangci-lint` reported `0 issues`.

Interpretation:

- The repository is currently internally consistent enough to compile and pass
  its present unit tests.
- Several findings below are therefore test-coverage or lifecycle-alignment
  problems rather than compile-time failures.

## Round-1 Status Update

Date: 2026-06-22

Post-implementation verification rerun in `NWDAF/`:

- `make build`
- `go test ./...`
- `make lint`

Result:

- Build passed.
- Full Go test suite passed.
- `golangci-lint` reported `0 issues`.

Round-1 implementation status:

- Finding 1 was addressed in the subscription update path.
- Finding 2 was addressed for notifier scheduler ownership and shutdown.
- Targeted tests were added for update reconciliation and notifier lifecycle.
- The round-1 code was merged locally to `NWDAF/master` and pushed to
  `origin/master`.
- Findings 3 and later remain relevant unless noted otherwise below.

## 2026-06-23 Reassessment Note

This scan file remains the historical finding list from the original pass.
After rechecking the current `NWDAF/` tree on 2026-06-23:

- the test-gap findings that were later grouped into Priority 3 should now be
  treated as materially closed by implementation, not as current blockers
- the covered standards-facing SBI error-contract work is now implemented and
  should no longer be treated as the next blocking refactor round
- the most actionable remaining confirmed issues in live code are now
  factory/config drift and the fragmented shared app boundary
- the updated execution order lives in
  `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- the next intended implementation round after the completed factory work is
  now documented in
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Rebuild One Real App Boundary Plan.md`

## 2026-06-24 Config-Hardening Update

After implementing the planned Priority 8 round in `NWDAF/`:

- the config/factory finding below should now be treated as materially closed
  for the current repository shape
- `ReadConfig(...)` now runs explicit validation after defaults
- the `GetSbiBindingAddr()` helper bug is fixed and normalized getter usage is
  in place at the current server and callback call sites
- runtime truth drift around `supportedAnalytics`, `README.md`, and sample
  config is now reduced for the currently supported standalone SBI runtime
- the next remaining structural target is the fragmented shared app boundary

## 2026-06-24 App-Boundary Phase-1 Update

After implementing the original Priority 4 round in `NWDAF/`:

- the main fragmented shared app-boundary finding should now be treated as
  materially improved in code
- `pkg/app` is now the shared runtime root contract used by service, server,
  processor, AnLF, MTLF, and consumer paths
- consumer construction is now app-driven and no longer derives ADRF setup from
  `factory.NwdafConfig`
- the adjacent unused server lifecycle parameter drift was also cleaned up
- the stricter remaining follow-up is now Phase 2 rather than a full replay of
  the original Priority 4 round
- the remaining Phase 2 design is documented in
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan.md`

## 2026-06-24 App-Boundary Phase-2 Update

After implementing the planned Phase 2 follow-up in `NWDAF/`:

- the stricter remaining Priority 4 exceptions should now be treated as
  materially closed in code
- `CancelContext()` was removed from the root `pkg/app.App` contract and kept
  only on the smaller local seams that require cancellation
- processor-side consumer ownership now depends on a narrower mockable seam
  instead of the concrete consumer type
- the exported consumer test-assembly constructor is no longer part of the
  public consumer package API
- Priority 4 should now be treated as completed for the current intended
  standalone-NWDAF alignment level

## 2026-06-25 External-Client Ownership Follow-Up Update

After implementing the later Priority 4 follow-up in `NWDAF/`:

- Completed in that implementation line:
  1. ML and Daisy client assembly moved to the app-owned
     `internal/sbi/consumer.NewConsumer(...)` boundary
  2. processor construction now consumes already-owned external clients instead
     of deciding client construction itself
  3. the narrower teardown/shutdown findings that appeared during stricter
     review of that ownership work were also closed
  4. the non-3GPP external-client ownership issue recorded in this scan file
     should now be treated as closed for the current intended standalone-NWDAF
     alignment scope
- Still open, but under Priority 2 rather than this closed Priority 4
  follow-up:
  1. broader lifecycle cancellation cleanup
  2. broader app-owned I/O context cleanup across the wider repository

## 2026-07-01 Priority-12 Completion Update

After implementing the separate Priority 12 server-topology phase in `NWDAF/`:

- the remaining shared-Gin and package-boundary portion of the repository /
  package-ownership finding should now be treated as materially closed for the
  current intended scope
- `pkg/service` now explicitly owns three inbound listeners:
  - the main SBI listener
  - the `AnLF` auxiliary listener
  - the `MTLF` auxiliary listener
- `internal/sbi` no longer exposes route injection for the `AnLF` / `MTLF`
  ingress paths
- `internal/anlf/server.go` and `internal/mtlf/server.go` now own their own
  transport implementation
- touched callback URIs now derive from `configuration.anlf.server` and
  `configuration.mtlf.server` rather than from the main SBI listener
- the follow-up cleanup commit removed the stale `externalMtlf.notifUri`
  override path and completed the touched inference-engine rename cleanup
- post-implementation verification reran `go test ./...`, `make build`, and
  `make lint`; all passed

## 2026-06-24 Strict Reassessment Absorption Note

The later same-day strict reassessment is now absorbed into this scan file and
into:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`

Role split after absorption:

1. this scan file is the evidence ledger
2. `NWDAF Main Project Remediation Batches.md` is the canonical status,
   prioritization, and work-item ownership map
3. the later Priority 3 and Priority 4 strict follow-up plans remain the
   detailed implementation records for their own completed lineages

What the absorbed strict reassessment added:

1. residual judgments on already-scanned areas after later Priority 3 and
   Priority 4 work
2. one narrower completed Priority 3 follow-up for stricter test/mock
   ownership
3. one narrower Priority 4 follow-up that was still open at reassessment time
   and was later closed in the same plan lineage

Current canonical homes after absorption:

- residual lifecycle hardening remains under Priority 2 in
  `NWDAF Main Project Remediation Batches.md`
- the Priority 4 runtime-truth residual was later closed on 2026-06-24 by
  `NWDAF/` commit `a912581`
- the stricter Priority 3 test-ownership follow-up was later closed on
  2026-06-24 by `NWDAF/` commit `0768839`
- the Daisy / ML service external-client ownership follow-up was later closed
  on 2026-06-25 by `NWDAF/` commit `f026e77`
- the latest status and remaining open-work split live in:
  `NWDAF Main Project Remediation Batches.md`
- within that split, the completed portions and still-open portions of Priority
  2 are now called out explicitly
- the current detailed implementation plan for that still-open Priority 2 work
  lives in:
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 2 Remaining Lifecycle Completion Plan.md`
- that remaining-work plan is now explicitly bounded so Priority 2 closes the
  lifecycle/ownership inventory without absorbing adjacent Priority 5, 6, 7,
  10, 11, or 12 issue lines just because some files overlap
- that bounded remaining-work plan was then implemented on 2026-06-26
- post-implementation verification reran `make build`, `go test ./...`, and
  `make lint`, all of which passed
- the post-implementation review against the workspace development policy found
  no new confirmed issue inside that bounded Priority 2 scope
- the canonical current status therefore moves to:
  - Priority 2 completed for its bounded lifecycle / ownership scope
  - adjacent still-open work remains under Priority 5, 6, 7, 10, 11, and 12

## Findings

### 1. High — `PUT /subscriptions/{id}` updates only the local record and leaves upstream collection state stale

Type: confirmed issue

Evidence:

- Create flow validates defaults and triggers external side effects:
  - `internal/sbi/processor/eventssubscription.go:25-29`
  - `internal/sbi/processor/eventssubscription.go:99-109`
- Update flow:
  - validates only the request body: `internal/sbi/processor/eventssubscription.go:147-153`
  - stops and recreates only the local notifier: `internal/sbi/processor/eventssubscription.go:164-215`
  - never calls `applyAndValidateDefaults`
  - never calls `cleanupDataCollection`
  - never re-runs `TriggerDataCollection`
  - never reconciles ML-model provisioning state
- Delete flow does have explicit SMF and ML cleanup:
  - `internal/sbi/processor/eventssubscription.go:237-281`

Why this is a problem:

- A subscription update can change `tgtUe`, notification URI, event set, or
  reporting controls, but the implementation only mutates the stored NWDAF
  object and local scheduler.
- Existing SMF subscriptions, correlation mappings, traffic buckets, ADRF state,
  and ML-model state can remain bound to the pre-update subscription shape.
- Create and update therefore do not implement the same notification-method
  semantics. Create enforces `applyAndValidateDefaults`; update does not.

Impact:

- The NWDAF response body after `PUT` can claim one subscription state while the
  actual upstream SMF collection graph still reflects an older one.
- This is a correctness issue, not just a style issue.

Coverage gap:

- Current tests cover validation helpers, but the scan did not find a test that
  proves update-time resource reconciliation.

### 2. High — periodic notification schedulers are outside application lifecycle control

Type: confirmed issue

Evidence:

- Scheduler roots its control context at `context.Background()`:
  - `internal/notifier/notifier.go:69`
- Notification send path also uses background-derived request timeouts and
  `http.DefaultClient`:
  - `internal/notifier/notifier.go:183-193`
- The scheduler owns its own private waitgroup:
  - `internal/notifier/notifier.go:32-34`
  - `internal/notifier/notifier.go:72-73`
  - `internal/notifier/notifier.go:95`
- Service shutdown only stops the SBI server:
  - `pkg/service/init.go:200-205`
- Scheduler stop is only wired from per-subscription update/delete:
  - `internal/sbi/processor/eventssubscription.go:164-166`
  - `internal/sbi/processor/eventssubscription.go:253-255`

Why this is a problem:

- free5GC lifecycle guidance expects goroutines to be owned by app/service
  shutdown paths or to be tied to an app cancellation context.
- These schedulers are not tied to `NwdafApp` shutdown and are not registered in
  the app waitgroup.

Impact:

- Graceful shutdown is incomplete.
- In-flight periodic notification work can continue independently of service
  termination until process exit forcibly stops it.
- This is especially risky because this repository already has multiple long-run
  background paths for AnLF and MTLF that *are* lifecycle-aware.

### 3. Medium — the `pkg/app` boundary is under-specified, so internal packages re-declare local app interfaces and bypass the shared app contract

Type: structural issue

Evidence:

- NWDAF `pkg/app/app.go` only exposes config/log toggles and `Terminate()`:
  - `pkg/app/app.go:7-13`
- Reference free5GC app interfaces expose at least `Start()`, `Terminate()`,
  `Context()`, and `Config()`:
  - `resources/references/free5gc-main/NFs/udr/pkg/app/app.go:8-17`
- Because the app boundary is too narrow, NWDAF internal packages define their
  own ad hoc app contracts:
  - `internal/sbi/server.go:45-50`
  - `internal/sbi/processor/processor.go:15-18`
  - `internal/anlf/anlf.go:15-18`
  - `internal/mtlf/mtlf.go:14-18`
- Consumer construction is also detached from the shared app interface:
  - `pkg/service/init.go:64-69`
  - `internal/sbi/consumer/consumer.go:22-45`
- Reference consumer construction is app-driven:
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/consumer.go:8-24`

Why this is a problem:

- The repository keeps a `pkg/app` package, but it is not the real app boundary
  used by server, processor, AnLF, MTLF, and consumer code.
- That defeats the normal free5GC pattern where app, service, consumer, and
  processor share one recognizable contract.

Impact:

- Dependency injection becomes inconsistent.
- Future mocking, testing, and lifecycle refactors have to chase several
  partially-overlapping interfaces instead of one stable boundary.

### 4. Medium — free5GC control-plane lifecycle features are largely absent even though the repository presents itself as a free5GC-style NF

Type: structural alignment gap

Current reassessment on 2026-07-14, last updated 2026-07-15 after Phase 2
implementation and live verification:

- the original evidence below is retained as historical scan evidence
- Priority 14 has since moved the main SBI edge to the shared HTTP/2 server
  helper, attached the existing inbound metrics middleware, and completed
  HTTP/HTTPS selection with SBI TLS configuration
- NRF registration/deregistration and OAuth/certificate support are complete
  in `NWDAF` commits `2d34594`, `3c545d0`, and `74b608b` and
  `nwdaf-resources` commit `d70c4e1`; NRF-backed SMF discovery is implemented
  and verified in the current `NWDAF` working tree, with its commit pending
- the integration direction is now decided: NWDAF will align upward as an
  NRF-managed NF through staged implementation
- NRF registration/deregistration, OAuth/certificate support, and SMF discovery
  are the first three implemented phases; this discovery path establishes the
  pattern for later standardized peer-facing interfaces
- Phase 0 behavior is now fixed: `nrfUri` is required; retryable registration
  failure keeps NWDAF alive with owned listeners unstarted until success or
  cancellation; `nfServices`, registration-before-listener startup, same-PLMN
  NRF-default handling, the free5GC API-prefix shape, and
  deregistration-before-SBI shutdown are used;
  redirect and OAuth-required operation are explicit Phase 0 terminal cases
- cross-NF comparison confirmed that the inbound middleware is normally paired
  with initialized SBI collectors, outbound generated-client hooks, and a
  separately owned metrics server exposing `/metrics`; completing that pattern
  is therefore restored to Priority 11 scope
- the common free5GC factory implementation keeps metrics optional and disabled
  by default, while still providing full configuration and lifecycle support
- metrics runtime completion remains required but is tracked as a non-blocking
  supporting workstream rather than a prerequisite for NRF integration
- TS 29.510 NFUpdate heartbeat is an explicit non-blocking deferred standards
  gap: the updated free5GC `main`/NRF v1.4.5 reference accepts NF profile PATCH,
  but the surveyed NF consumers do not send periodic heartbeat and NRF does not
  yet show a complete timer/expiry/suspension lifecycle
- live Phase 0 verification proved registration, reachable listeners, NRF
  default-PLMN insertion, deregistration before SBI shutdown, and resource
  removal; it also confirmed that NRF v1.4.5's
  `NnrfNFManagementDataModel` does not copy the request's `NwdafInfo`, so the
  stored profile loses the `UE_COMMUNICATION` constraint even though NWDAF
  sends it; this NRF-side persistence gap is accepted as a Phase 0 deployment
  limitation and remains future NRF-side correction work rather than an NWDAF
  compatibility PATCH
- extended live P0.4 verification also proved `200` existing-resource
  replacement, listener gating through unavailable-NRF retry and same-process
  recovery, and bounded deregistration plus partial-listener cleanup after a
  forced post-registration AnLF bind failure
- live Phase 1 verification proved both OAuth-disabled and OAuth-enabled
  HTTP/H2C registration, signature/scope enforcement on the Events Subscription
  producer group, separate collector routing, token-protected deregistration,
  and post-shutdown NRF profile removal
- focused Phase 1 regression completion in `NWDAF` commit `74b608b` covers
  Access Token transport/pre-dispatch cancellation, invalid PEM content,
  OAuth-enabled listener rollback through token-protected deregistration, and
  production `NewServer(...)` route wiring; the complete focused, repeated,
  race, full-module, build, lint, and diff gates passed
- Phase 2 adds generated NRF SMF discovery, explicit
  `endpointSource: nrf|configured`, positive-validity caching, all-root fan-out,
  one endpoint set per reconciliation, rollback-preserving discovery failure,
  outbound `nsmf-event-exposure` bearer binding, and bounded app-owned cleanup
  for committed stale/delete work; focused, race, repeated, full-module, build,
  lint, and diff gates passed, including post-review cache, discovery-error,
  context-ownership, partial-fan-out cancellation, and OAuth regressions
- layered OAuth-disabled and OAuth-enabled live gates used the real free5GC NRF
  and SMF plus a temporary protected producer registered in the same NRF;
  discovery returned both endpoints, NWDAF create/delete returned `201`/`204`,
  missing and wrong-scope producer tokens returned `401`, and graceful protected
  deregistration completed
- the live gate also confirmed that the current SMF advertises
  `nsmf-event-exposure` at a standard API root but routes it internally as
  `/nsmf_event-exposure/v1`; the standard request therefore returns `404`, and
  its private underscore-prefixed CRUD handlers return `501`. This remains an
  explicit SMF-side limitation rather than an NWDAF compatibility typo.
- the current implementation plan is:
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 NRF OAuth Discovery And Metrics Alignment Plan.md`

Current evidence in NWDAF:

- `pkg/factory` requires and normalizes the NRF URI
- `internal/context` owns the generated NF profile and registration state
- `internal/sbi/consumer/nrf_service.go` owns generated NFManagement clients,
  generated NFDiscovery and Access Token clients, retry, response validation,
  discovery caching, protected deregistration, and caller-context propagation
- `internal/context` delegates producer authorization to
  `oauth.VerifyOAuth(...)` using context-owned OAuth state and `nrfCertPem`
- `internal/sbi` attaches service-specific authorization to the Events
  Subscription route group while leaving collector callbacks separate
- `internal/sbi/processor` resolves one SMF endpoint set per reconciliation and
  preserves configured/discovered source separation, fan-out resource identity,
  rollback, and cleanup behavior
- `pkg/service/init.go` registers before starting listeners and deregisters
  before stopping SBI
- exact-profile, HTTP-contract, retry/cancellation, and lifecycle tests cover
  the Phase 0 behavior

Reference free5GC baselines:

- UDR wires consumer, metrics server, NRF registration, deregistration, and
  coordinated shutdown:
  - `resources/references/free5gc-main/NFs/udr/pkg/service/init.go:61-76`
  - `resources/references/free5gc-main/NFs/udr/pkg/service/init.go:152-166`
  - `resources/references/free5gc-main/NFs/udr/pkg/service/init.go:186-259`
- NRF and UDR servers are built around shared HTTP server helpers and inbound
  metrics middleware:
  - `resources/references/free5gc-main/NFs/nrf/internal/sbi/server.go:39-55`
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/server.go:95-120`

Why this matters:

- This is not merely “feature not implemented yet”; it changes how close the
  project is to a recognizable free5GC NF lifecycle.
- Missing metrics, NRF registration/deregistration, and HTTP/TLS/auth wiring
  make the repository harder to compare, review, and integrate alongside the
  existing free5GC NFs.

Impact:

- The project behaves more like a standalone SBI service than a normal
  free5GC-managed NF.
- This should be treated as a deliberate gap to either close or document, not as
  an accidental omission to leave unclassified.

### 5. Medium — config/factory is much weaker than the free5GC baseline and already contains at least one latent bug

Type: confirmed issue plus structural gap

Evidence:

- `ReadConfig` loads YAML and sets a few defaults, but does not run a real
  config validation pass:
  - `pkg/factory/config.go:490-541`
- `GetSbiBindingAddr()` is incorrect for multi-digit ports:
  - `pkg/factory/config.go:544-550`
  - it converts the port with `string(rune(c.Configuration.Sbi.Port+'0'))`
    rather than decimal formatting
- Default advertised analytics contradict the actual supported event list:
  - defaults: `pkg/factory/config.go:521-523`
  - runtime support list: `internal/sbi/processor/eventssubscription.go:516-518`
- Reference free5GC config factories expose explicit validation and richer
  getters/setters:
  - `resources/references/free5gc-main/NFs/udr/pkg/factory/config.go:38-96`

Why this is a problem:

- Config is a core free5GC boundary. Weak validation moves failures from startup
  into runtime behavior.
- The bad address helper is a concrete bug even if current code paths do not
  call it today.

Impact:

- Invalid or contradictory configuration can survive parsing too easily.
- Future refactors that start using `GetSbiBindingAddr()` will inherit a broken
  helper.

### 6. Medium — cross-NF unit tests do not follow the free5GC-style mock boundary and are supplemented by external Python harnesses

Type: structural test gap

Evidence:

- The local free5GC testing guidance explicitly separates:
  - processor procedure tests with `httptest`: `free5gc-dev-skill/references/testing.md:10`
  - outbound SBI consumer tests with `gock`: `free5gc-dev-skill/references/testing.md:11`
  - H2C client-path tests with `openapi.InterceptH2CClient()`: `free5gc-dev-skill/references/testing.md:12,20-35`
- Current NWDAF outbound/client-facing tests still use `httptest.NewServer`:
  - `internal/sbi/consumer/daisy_service_test.go:4-7,33-45,72-97`
  - `internal/sbi/consumer/ml_service_test.go:4-10,62-76,147-160,220-239`
  - `internal/sbi/processor/data_collection_test.go:4-11,27-40,204-216,277-288`
- The module does not currently carry `gock` in `go.mod`:
  - `go.mod:5-57`
- Repository testing guidance still depends on a separate Python callback setup:
  - `docs/testing.md:12-23`
- The repo also carries standalone Python fake peers and callback fixtures:
  - `test/fake_daisy/fake_daisy_server.py:1-17,50-109`
  - `test/fake_smf_upf/README.md:13-39`
  - `test/callback/README.md:7-25`

Why this is a problem:

- The current tests prove local request/response logic, but they do not follow
  the free5GC-oriented test split that keeps peer-NF mocking inside Go tests
  and reserves separate scripts/harnesses for explicit integration checks.
- The boundary between unit-level validation and ad hoc multi-process testing is
  therefore blurred.

Impact:

- Consumer behavior is not exercised through the preferred free5GC mock path.
- Cross-NF API regression coverage is weaker than the scan result initially
  suggested.
- The main NF repository is carrying auxiliary test fixtures that compensate for
  missing mock-based coverage.

### 7. Medium — contract governance for handwritten Release 18/19 and standardized payload models is not defined

Type: structural issue

Evidence:

- The skill guidance says standardized SBI payloads should use generated models
  where available and contract changes should prefer regeneration over ad hoc
  structs:
  - `free5gc-dev-skill/references/openapi-contract.md:5-32`
  - `free5gc-dev-skill/references/openapi-generation.md:3-18,35-47`
- NWDAF redefines a standardized ML model provision notification locally:
  - `internal/sbi/api_mlmodelprovision.go:12-29`
- The workspace already contains the generated counterpart:
  - `resources/openapi/openapi/models/model_nwdaf_ml_model_prov_notif.go:15-20`
- NWDAF also carries handwritten SMF/UPF exposure extension models directly in
  the main codebase:
  - `internal/sbi/consumer/models.go:5-100`

Why this is a problem:

- Some local extensions may be justified because the pinned OpenAPI dependency
  is still tied to an earlier 3GPP release, but the project currently has no
  explicit rule for when to regenerate, when to isolate an extension layer, and
  when a handwritten struct is acceptable.
- Without that governance, standardized payloads and local compatibility
  payloads are mixed together in the same implementation boundary.

Impact:

- Schema provenance becomes hard to audit.
- Future alignment to newer Release 18/19 features will be harder to review
  because there is no clean distinction between “dependency snapshot gap” and
  “project-local contract invention”.

### 8. Medium — main-repo and package boundaries are overloaded with auxiliary assets and MTLF-specific ownership
Type: structural alignment gap

Current status:

- The originally recorded finding is now split by scope.
- The support-asset move and the `internal/sbi` / `internal/anlf` /
  `internal/mtlf` ownership split are completed for the current intended scope.
- The server-topology portion of this finding was completed on 2026-07-01 by
  `NWDAF/` commits `0ddbf3c` and `b547727`.
- Any remaining later cleanup under this finding is now about broader
  repository/config-scope discipline rather than the already-implemented
  Daisy / inference-engine transport ownership split.

Evidence:

- The main implementation repo contains project-internal documentation and
  standalone Python peer fixtures:
  - `docs/testing.md:1-23`
  - `test/callback/README.md:1-25`
  - `test/fake_smf_upf/README.md:1-39`
  - `test/fake_daisy/fake_daisy_server.py:1-17`
- free5GC reference NF trees do not show an equivalent per-NF Python fixture
  layout; the broader project keeps runtime config centralized under a root
  `config/` tree:
  - `resources/references/free5gc-main/config/udrcfg.yaml:1-24`
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/api_sanity_test.go`
- Daisy async callback HTTP handling is defined under generic SBI files even
  though the behavior itself belongs to the MTLF workflow:
  - `internal/sbi/api_daisy_callback.go:11-52`
  - `internal/sbi/server.go:83-90`
  - `internal/sbi/processor/processor.go:84-86`
  - `internal/mtlf/training.go:41-58,110-171`
- The sample config and config structs currently bundle NF runtime settings,
  group-resolution fallback data, ML service settings, Daisy task payloads, and
  ADRF retrieval workflow configuration in one main-repo boundary:
  - `config/nwdafcfg.yaml:17-158`
  - `pkg/factory/config.go:34-46,62-67,151-221`

Why this is a problem:

- This is larger than a style issue. The repo boundary currently mixes:
  - core NF runtime code
  - external-system test harnesses
  - workflow-specific callback HTTP glue
  - config that partly acts as deployment/runtime data and partly as local lab
    fixture state
- That makes it harder to preserve a clean free5GC-style separation between NF
  runtime, auxiliary resources, and domain-specific ownership such as MTLF.

Impact:

- `internal/sbi` ends up carrying non-3GPP workflow details that would more
  naturally live behind `internal/mtlf`.
- Future cleanup of config, tests, or external integration assets will be more
  invasive because the repository boundary is already blurred.

### 9. Low — repository guidance is inconsistent about supported Go version

Type: documentation/process issue

Evidence:

- README says `Go 1.21+`:
  - `README.md:5-8`
- `go.mod` declares `go 1.25.5`:
  - `go.mod:1-4`

Why this matters:

- The skill explicitly treats `go.mod` as the source of truth.
- A stale README increases environment mismatch risk for future contributors and
  reviewers.

### 10. Medium — internal dependency mocking, handler coverage, and lifecycle-oriented tests do not meet the local free5GC testing baseline

Type: structural test gap

Evidence:

- The local testing guidance expects:
  - `gomock` for app/context dependencies: `free5gc-dev-skill/references/testing.md:13,51-63`
  - `httptest` plus `gin.CreateTestContext` for SBI procedure/handler work:
    `free5gc-dev-skill/references/testing.md:10,20-35`
  - lifecycle/concurrency tests for cancellation or timeout when the local app
    interface allows it: `free5gc-dev-skill/references/testing.md:14`
- Current processor tests use handwritten stubs instead of interface mocks:
  - `internal/sbi/processor/upf_notify_test.go:18-39`
- Some tests instantiate real consumers or concrete services rather than
  mocking internal dependencies through the app boundary:
  - `internal/sbi/processor/data_collection_test.go:36-40,213-216,285-288`
  - `internal/sbi/consumer/ml_service_test.go:223-228`
- The current `internal/sbi` test set covers consumer and processor packages,
  but the scan did not find handler-focused `api_*_test.go` coverage for:
  - `internal/sbi/api_eventssubscription.go`
  - `internal/sbi/api_collector.go`
  - `internal/sbi/api_mlmodelprovision.go`
  - `internal/sbi/api_daisy_callback.go`
  - `internal/sbi/api_adrf_notify.go`
- The notifier test set currently checks helper/control logic only:
  - `internal/notifier/notifier_test.go:11-240`
  - the scan did not find tests for `sendNotification()`, HTTP failure handling,
    or shutdown/cancellation interaction

Why this is a problem:

- The current unit tests verify a useful subset of package logic, but they do
  not yet align with the skill's intended free5GC-style split between:
  - internal dependency mocking with `gomock`
  - handler/procedure verification at the HTTP boundary
  - lifecycle-aware cancellation and timeout tests
- That leaves the most architecture-sensitive seams under-tested.

Impact:

- App-boundary refactors remain risky because there is no established mock
  contract protecting those seams.
- Handler error semantics and callback parsing behavior are largely unverified.
- Concurrency and shutdown fixes will be harder to validate mechanically.

### 11. Medium — SBI error response handling is inconsistent across endpoints and partly bypasses the normal `ProblemDetails` pattern

Type: confirmed issue plus structural gap

Evidence:

- Subscription and collector handlers return `models.ProblemDetails` on parse or
  procedure errors:
  - `internal/sbi/api_eventssubscription.go:19-32,54-67,78-81`
  - `internal/sbi/api_collector.go:42-60,73-93`
- Callback-oriented handlers instead return ad hoc `gin.H{"error": ...}` bodies:
  - `internal/sbi/api_daisy_callback.go:25-32`
  - `internal/sbi/api_adrf_notify.go:28-32`
  - `internal/sbi/api_mlmodelprovision.go:39-42`
- `HandleCollectorNotify` and `HandleUpfNotify` use `c.BindJSON(...)` rather
  than `c.ShouldBindJSON(...)`:
  - `internal/sbi/api_collector.go:42`
  - `internal/sbi/api_collector.go:73`

Why this is a problem:

- The review guidance asks whether responses and error bodies are aligned with
  local helpers and generated models:
  - `free5gc-dev-skill/references/review-checklist.md:22-27,60-64`
- Right now the same SBI server exposes multiple incompatible error-body styles.
- Using `BindJSON` also weakens control over when Gin writes status and body
  state compared with the explicit `ShouldBindJSON` path already used elsewhere
  in the same server.

Impact:

- Callback consumers cannot rely on one stable error contract.
- Handler behavior is harder to reason about and harder to test consistently.
- Future refactors may accidentally diverge further because there is no single
  enforced response pattern.

### 12. Low — post-subscription activation and late-failure signaling are underspecified once downstream data collection starts asynchronously

Type: design completeness gap

Evidence:

- The analytics subscription procedure in local TS 23.288 separates receiving
  the consumer subscription from the later decision to trigger new data
  collection:
  - `nwdaf-docs/specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.1 Procedures for analytics exposure/6.1.1 Analytics Subscribe and Unsubscribe.md:27-31`
- The NF-side data collection setup is a later procedure in which NWDAF
  subscribes to source NFs and then receives notifications:
  - `nwdaf-docs/specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2 Procedures for Data Collection/6.2.2 Data Collection from NFs/6.2.2.2 Procedure for Data Collection from NFs.md:18-32`
- TS 29.520 provides failure-signaling structures at both the subscription
  resource and notification layers:
  - `failEventReports`: `nwdaf-docs/specs/TS 29.520/5 API Definitions/5.1 Nnwdaf_EventsSubscription Service API/5.1.6 Data Model/5.1.6.2 Structured data types/5.1.6.2.2 Type NnwdafEventsSubscription.md:82-86`
  - `failNotifyCode` and `rvWaitTime`:
    `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml:763-766`
- Current create processing stores the subscription and returns success while
  downstream data collection is started in a background goroutine:
  - `internal/sbi/processor/eventssubscription.go:67-125`
  - `internal/sbi/processor/eventssubscription.go:99-109`

Why this is a problem:

- The spec does not require NWDAF to synchronously complete all downstream
  data-source subscriptions before accepting the analytics subscription.
- The local implementation is therefore not wrong simply because it starts
  data collection asynchronously.
- The actual gap is that later activation failures are not modeled or signaled
  clearly enough once the consumer-visible subscription resource has already
  been created.

Impact:

- Operators and tests cannot clearly distinguish:
  - accepted subscription resource
  - downstream activation in progress
  - activation degraded or failed later
- The current design leaves late failure handling under-specified compared with
  the spec-supported signaling fields.

### 13. Historical low finding — logging used dedicated NF loggers, but message boundaries and payload hygiene were still loose

Type: structural logging gap

Current status on 2026-06-29:

- completed for the current intended Priority 7 scope in `NWDAF/` commit
  `2ff07af`
- historical evidence below is preserved as scan-time basis for the later
  implementation plan, not as the current post-implementation state
- the implemented cleanup removed the main touched duplicate entry logs,
  reduced default-level identifier exposure, demoted high-frequency
  observability detail, and normalized message shape across the main runtime
  paths
- the final implementation also removed an intermediate local consumer HTTP
  error helper so the consumer layer stayed closer to the simpler per-call-site
  free5GC style seen in local exemplar NFs
- local verification passed with `make build`, `make lint`, and
  `go test ./...`
- the current accepted lifecycle `Fatal` boundaries in `pkg/service/init.go`
  and `internal/sbi/server.go` were not widened by this round and are no
  longer treated here as a separate open Priority 7 implementation blocker

Historical evidence at scan time:

- The code does attach logs to purpose-specific logger groups such as
  `SBILog`, `ProcLog`, `InitLog`, `NotifierLog`, and `AnlfLog`, which is a good
  baseline:
  - `internal/sbi/api_eventssubscription.go:16,51,76`
  - `internal/sbi/processor/eventssubscription.go:18,71,133,239`
  - `pkg/service/init.go:128,134,142,201`
- At the same time, several consumer paths log full outbound request payloads:
  - `internal/sbi/consumer/smf_service.go:170`
  - `internal/sbi/consumer/mtlf_service.go:149`
  - `internal/sbi/consumer/daisy_service.go:92`
- Entry/background paths also still use fatal process-exit logging inside
  goroutine-driven control flow:
  - `pkg/service/init.go:177-179,185-190`
  - `internal/sbi/server.go:151-157`

Why this is a problem:

- The review guidance asks whether sensitive values stay out of logs and whether
  warning/error/fatal levels are appropriate:
  - `free5gc-dev-skill/references/review-checklist.md:58-64`
- Logging raw request bodies is convenient during development, but it is a weak
  long-term boundary for an NF that already carries SUPIs, notification URIs,
  model URLs, and task payloads.
- `Fatalf` in background control paths also makes failure handling more abrupt
  than the rest of the repository's return-and-log style.

Impact:

- Log output is less ready for harder operational environments than the current
  structure suggests.
- Future cleanup will need to distinguish observability logs from debug payload
  dumps and process-exit paths.

## 2026-06-24 Strict Reassessment Addendum

This addendum absorbs the evidence-bearing parts of the later same-day strict
reassessment so that a separate reassessment file is no longer required to
understand the remaining project-level gaps.

### A. Residual judgments on already-scanned areas

#### A1. Priority 2 still has broader app-owned I/O context cleanup left

After the notifier scheduler ownership fix, the repository still had several
runtime network and shutdown paths deriving timeout contexts from
`context.Background()` rather than from the app cancellation tree.

Representative evidence at reassessment time:

- `NWDAF/pkg/service/init.go:137`
- `NWDAF/pkg/service/init.go:151`
- `NWDAF/internal/notifier/notifier.go:196`
- `NWDAF/internal/sbi/consumer/smf_service.go:124`
- `NWDAF/internal/sbi/consumer/mtlf_service.go:75`
- `NWDAF/internal/sbi/consumer/daisy_service.go:94`
- `NWDAF/internal/sbi/consumer/ml_service.go:120`
- `NWDAF/internal/sbi/consumer/adrf_service.go:171`
- `NWDAF/internal/sbi/server.go:163`

Judgment after absorption:

- this is not a new issue family
- it remains residual lifecycle hardening under Priority 2 in
  `NWDAF Main Project Remediation Batches.md`

#### A2. Priority 4 had one same-day runtime-truth residual that was later closed

After the original Phase 1 and Phase 2 app-boundary work, stricter re-review
found that some already-touched app-owned runtime paths still read
`factory.NwdafConfig` directly.

Representative evidence at reassessment time:

- `NWDAF/internal/anlf/model.go:16`
- `NWDAF/internal/anlf/monitor.go:22`
- `NWDAF/internal/mtlf/training.go:28`
- `NWDAF/internal/sbi/processor/data_collection.go:52`
- `NWDAF/internal/sbi/processor/upf_notify.go:299`

Judgment after absorption:

- this was a real residual in a completed line, not proof that Priority 4 was
  wrong
- it was later closed on 2026-06-24 by `NWDAF/` commit `a912581`
- the implementation record now lives in:
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Residual Runtime Truth Cleanup Plan.md`

#### A3. Several later findings were stricter scope clarifications, not new issue families

The strict reassessment also confirmed that the following topics were still
already-known open work rather than new problems created by the completed
rounds:

- runtime config mixed with lab/workflow config
- full free5GC lifecycle direction, which was still open at reassessment time
  and was later resolved on 2026-07-14 in favor of staged NRF-managed NF
  alignment plus a required but non-blocking common metrics workstream
- OpenAPI/model governance
- logging boundary and payload hygiene

Their canonical execution home is now the current remediation table in:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`

### B. New issues identified after the original scan

#### B1. Non-3GPP external clients originally bypassed the shared consumer/app ownership model

After Priority 4, NWDAF had a recognizable shared app boundary and an
app-driven `internal/sbi/consumer` path, but some project-local external
clients still bypassed that ownership model.

Evidence in `NWDAF/` at reassessment time:

- AnLF creates ML service clients directly:
  - `NWDAF/internal/anlf/model.go:23`
  - `NWDAF/internal/anlf/model.go:88`
- MTLF creates Daisy clients directly:
  - `NWDAF/internal/mtlf/training.go:129`

free5GC baseline comparison used in this reassessment:

- reference control-plane NFs such as UDR keep peer-facing client ownership on
  the app/service/consumer path:
  - `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- consumer packages such as NEF construct outbound client groups from an
  app-owned seam rather than ad hoc domain-local instantiation:
  - `resources/references/free5gc-main/NFs/nef/internal/sbi/consumer/consumer.go`
- root app contracts such as UDM's remain the shared dependency boundary for
  those owned client paths:
  - `resources/references/free5gc-main/NFs/udm/pkg/app/app.go`

Judgment after absorption:

- this is now a formal work item under
  `NWDAF Main Project Remediation Batches.md`
- it is intentionally tracked as a narrower Priority 4 follow-up rather than
  as late repository/package cleanup under Priority 12

Later status:

- this gap was later closed on 2026-06-25 by `NWDAF/` commit `f026e77`
- ML and Daisy client assembly now lives on the app-owned
  `internal/sbi/consumer.NewConsumer(...)` path
- processor now consumes already-owned external clients through the narrower
  consumer seam instead of constructing them directly
- the remaining adjacent lifecycle cleanup after that implementation is tracked
  under Priority 2, not as a still-open Priority 4 ownership gap

#### B2. Test ownership was still more package-local than the surveyed free5GC baseline

The same strict re-review also found one narrower ownership gap in the already
completed Priority 3 lineage.

Evidence in `NWDAF/` at reassessment time:

- `internal/sbi` kept generated mocks locally:
  - `NWDAF/internal/sbi/mock_interfaces_test.go:1`
- `internal/sbi/processor` kept its own local app mock seam:
  - `NWDAF/internal/sbi/processor/mock_interfaces_test.go:15`
- handler tests used a package-local fake app:
  - `NWDAF/internal/sbi/api_test_helpers_test.go:20`

Reference comparison used in this reassessment:

- shared app-boundary mocks under app/service packages:
  - `resources/references/free5gc-main/NFs/smf/pkg/service/mock.go`
  - `resources/references/free5gc-main/NFs/udm/pkg/mockapp/mock.go`
- SBI-local mocks retained only where the seam is truly local:
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/api_sanity_test.go`

Judgment after absorption:

- this gap was later closed on 2026-06-24 by `NWDAF/` commit `0768839`
- the implementation record now lives in:
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Strict Test Ownership Alignment Plan.md`

## Open Questions And Deferred Risks

### SMF event exposure model strategy

`internal/sbi/consumer/models.go` extends the SMF event exposure schema for
`UPF_EVENT`. This may be justified, because the local TS 29.508 YAML contains
`upfEvents`, `bundlingAllowed`, and `bundledEventNotifyUri`, while the current
generated dependency snapshot does not expose that shape cleanly enough.

This should be treated as:

- not an immediate defect by itself
- but a place where contract-regeneration or generated-client review should be a
  later remediation batch

### Local workspace test artifacts

The workspace currently contains local `.venv` and cache directories under
`NWDAF/test/` and `NWDAF/tools/`. The scan did not classify this as a committed
repository issue because the current git state was clean, but it remains worth
checking in a follow-up hygiene pass.

### Post-subscription activation signaling

The local spec material supports a model in which NWDAF accepts an analytics
subscription and then decides whether and how to start downstream data
collection. That means asynchronous activation is not, by itself, a defect.

What remains open is how the implementation should expose later activation
problems once the subscription resource already exists, especially if the first
round of remediation explicitly defers local Daisy/MTLF workflow redesign.

## Summary

## Revised Priority View

This scan file now preserves the reasoning behind the revised priority view,
but the canonical current execution order lives in:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`

After absorbing the same-day strict reassessment, that current execution order
now also includes the narrower Priority 4 follow-up for non-3GPP external
client ownership.

Interpretation after round-1 completion:

- Priorities 1 to 3 were the first execution slice because they covered the
  clearest confirmed correctness bug, the notifier lifecycle gap, and the
  minimum test seam needed to change those paths safely.
- Round 1 closed the implementation target for priority 1 and the notifier-owned
  portion of priority 2, and it added focused tests that partially satisfy
  priority 3.
- Later rounds then closed the main Priority 4, Priority 5, Priority 8, and
  stricter same-lineage Priority 3/4 follow-up work recorded elsewhere in this
  document set.

The most important open problems after the later implemented follow-up rounds are:

1. broader lifecycle cancellation and app-owned I/O context cleanup still
   remains
2. post-subscription activation and late-failure signaling are still
   underspecified as a design/operability issue
3. OpenAPI/model governance and runtime config scope remain outstanding; the
   free5GC integration direction is decided, NRF and OAuth phases are
   implemented, while discovery and the independent metrics workstream remain
   open

The repository now passes `make build`, `go test ./...`, and `make lint` after
the implemented rounds recorded above. That verifies the current repository at
the unit/build/lint level, but it does not close the later structural,
contract-oriented, and scope-decision findings that remain open.
