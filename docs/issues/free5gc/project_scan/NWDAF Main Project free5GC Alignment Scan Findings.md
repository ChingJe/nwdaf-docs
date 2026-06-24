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

Evidence in NWDAF:

- Service startup only initializes context, consumer, processor, MongoDB, and
  the SBI server:
  - `pkg/service/init.go:37-80`
  - `pkg/service/init.go:127-181`
- Shutdown only stops the SBI server:
  - `pkg/service/init.go:200-205`
- `nrfUri` exists in config shape but is not wired into any registration flow:
  - `pkg/factory/config.go:38`
  - sample config leaves it commented out: `config/nwdafcfg.yaml:15`
- The server uses plain `gin.New()` and does not integrate the inbound metrics
  middleware or HTTP/2/TLS server helper used by reference NFs:
  - `internal/sbi/server.go:59-105`

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
  - `nwdaf-docs/specs/TS 23.288/6.1 Procedures for analytics exposure.md:16-26`
- The NF-side data collection setup is a later procedure in which NWDAF
  subscribes to source NFs and then receives notifications:
  - `nwdaf-docs/specs/TS 23.288/6.2.2 Data Collection from NFs.md:130-140`
- TS 29.520 provides failure-signaling structures at both the subscription
  resource and notification layers:
  - `failEventReports`: `nwdaf-docs/specs/TS 29.520/5.1 Nnwdaf_EventsSubscription Service API.md:731-742`
  - `failNotifyCode` and `rvWaitTime`:
    `nwdaf-docs/specs/yaml/TS29520_Nnwdaf_EventsSubscription.yaml:780-789`
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

### 13. Low — logging uses dedicated NF loggers, but message boundaries and payload hygiene are still loose

Type: structural logging gap

Evidence:

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

The revised execution order after re-review is:

1. repair subscription update correctness
2. put long-running work under app lifecycle control
3. build the test safety net around the real boundaries
4. rebuild one real app boundary
5. normalize SBI error contracts
6. clarify post-subscription activation and late-failure signaling
7. tighten logging boundaries
8. harden factory and runtime config behavior
9. separate runtime config from lab/workflow config
10. establish OpenAPI/model governance
11. decide the intended free5GC integration level
12. clean repo and package ownership boundaries

Interpretation after round-1 completion:

- Priorities 1 to 3 were the first execution slice because they covered the
  clearest confirmed correctness bug, the notifier lifecycle gap, and the
  minimum test seam needed to change those paths safely.
- Round 1 closed the implementation target for priority 1 and the notifier-owned
  portion of priority 2, and it added focused tests that partially satisfy
  priority 3.
- Priorities 4 to 12 remain the main follow-up backlog after round 1.

The most important open problems after round 1 are:

1. the shared app boundary remains under-specified and internally fragmented
2. SBI error contracts are still inconsistent across handlers and callbacks
3. post-subscription activation and late-failure signaling are still
   underspecified as a design/operability issue
4. config/runtime alignment and broader repository governance issues remain
   outstanding

The repository now passes `make build`, `go test ./...`, and `make lint` after
the round-1 changes. That verifies the updated subscription and notifier paths
at the current repository level, but it does not close the later structural and
contract-oriented findings.
