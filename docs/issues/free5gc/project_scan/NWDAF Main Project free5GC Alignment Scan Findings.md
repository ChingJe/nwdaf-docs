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

### 6. Low — the ML model provision callback redefines a standardized model that already exists in the local OpenAPI reference

Type: structural issue

Evidence:

- NWDAF defines local callback structs:
  - `internal/sbi/api_mlmodelprovision.go:12-29`
- The workspace OpenAPI reference already contains the generated model:
  - `resources/openapi/openapi/models/model_nwdaf_ml_model_prov_notif.go:15-20`

Why this is a problem:

- The skill guidance prefers generated models where they exist.
- Keeping a local parallel struct for a standardized payload increases drift
  risk if the dependency or schema changes later.

Qualification:

- This is not as severe as the update-path or lifecycle issues because the code
  may have been written before the local OpenAPI reference was fully available.
- It is still a traceability and maintenance problem.

### 7. Low — repository guidance is inconsistent about supported Go version

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

## Summary

The most important confirmed problems are:

1. subscription update does not reconcile external collection/model side effects
2. periodic notification goroutines are not owned by the app lifecycle
3. app/consumer/config boundaries are weaker than normal free5GC NF structure

The repository currently compiles and its present unit tests pass, but that
mainly shows local consistency. It does not prove the update path, shutdown
path, or free5GC lifecycle alignment are sound.
