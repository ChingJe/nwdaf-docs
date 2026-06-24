# NWDAF Priority 8 Factory And Runtime Config Hardening Plan

Date: 2026-06-23

Status: Completed on 2026-06-24

Historical remediation item:

- `Priority 8 - Harden Factory And Runtime Config Behavior`

Completion note:

- this round was implemented in `NWDAF/` on 2026-06-24
- `pkg/factory/config.go` now applies defaults, runs explicit validation, and
  rejects contradictory startup config
- SBI helper behavior is normalized around binding/register getters and the
  `GetSbiBindingAddr()` multi-digit port bug is fixed
- sample config and `README.md` now match the current runtime truth for
  supported analytics, Go version, HTTP-only SBI, and inactive `nrfUri`
- focused table-driven config tests were added in `pkg/factory`
- broader shared app-boundary reconstruction remains intentionally deferred to
  the next round

---

## 1. Purpose

This plan defines the next free5GC-alignment refactor round for the main
`NWDAF/` repository, limited to configuration parsing, validation, runtime
truth, and the factory-to-service boundary that already exists today.

The purpose of this round is to align NWDAF more carefully with the patterns
used by current free5GC control-plane NFs while keeping the change set narrow:

1. add an explicit config validation phase in `pkg/factory`
2. fix known SBI helper bugs and normalize getter behavior
3. align sample config and README/runtime guidance with the code that actually
   runs today
4. add table-driven config tests that protect later app-boundary work

This round is intentionally not a full lifecycle or integration uplift.

It is about:

- config defaults and validation
- startup-time rejection of contradictory runtime settings
- helper/getter correctness for SBI binding and register addresses
- narrowing the gap between advertised config surface and real runtime support
- documentation and sample-config accuracy

It is not about:

- adding NRF registration or deregistration
- adding a metrics server
- deciding the final free5GC integration level for NWDAF
- rebuilding the shared `pkg/app` boundary
- separating lab/workflow config from runtime config

---

## 2. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `resources/references/free5gc-main/NFs/udr/pkg/factory/factory.go`
- `resources/references/free5gc-main/NFs/udr/pkg/factory/config.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udm/pkg/factory/factory.go`
- `resources/references/free5gc-main/NFs/udm/pkg/factory/config.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/factory/factory.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/factory/config.go`
- `resources/references/free5gc-main/NFs/nef/pkg/factory/factory.go`
- `resources/references/free5gc-main/NFs/nef/pkg/factory/config.go`
- `resources/references/free5gc-main/NFs/nef/pkg/service/init.go`
- the current `NWDAF/` tree on 2026-06-23

---

## 3. free5GC Baseline Survey

### 3.1 ReadConfig Pattern In Other NFs

The surveyed free5GC NFs consistently split config loading into two steps:

1. YAML load through `InitConfigFactory(...)`
2. explicit `cfg.Validate()` inside `ReadConfig(...)`

This pattern is present in the reference UDR, UDM, NRF, and NEF factories.

Implication for NWDAF:

- `ReadConfig(...)` should stop being a "best effort plus defaults" loader
- invalid config should fail at startup instead of surviving into processor,
  consumer, or server code

### 3.2 Validation Style In Other NFs

The surveyed reference factories perform two kinds of validation:

1. structural validation through config structs and field tags
2. semantic validation through explicit `validate()` helpers

Common examples:

- allowed schemes restricted to `http` or `https`
- host and port validation for SBI bindings
- metrics/SBI binding collision rejection
- service-name allowlists
- version checks where the NF treats config version as part of the contract

Implication for NWDAF:

- NWDAF does not need to adopt every free5GC field or tag immediately
- it does need one explicit semantic validation pass for the config surface it
  already exposes and depends on

### 3.3 Getter Pattern In Other NFs

The surveyed factories do not build address strings by hand from rune math.
They commonly:

1. expose `GetSbiBindingIP()` and `GetSbiPort()`
2. build addresses with `strconv.Itoa(...)`
3. separate binding address and register address responsibilities

Implication for NWDAF:

- `GetSbiBindingAddr()` should be corrected and normalized to the same getter
  style
- if NWDAF keeps both binding and register addresses, helper naming should make
  the distinction explicit

### 3.4 Service Truth In Other NFs

UDR, UDM, and NEF show the same broad discipline:

- metrics are only wired when the runtime actually supports a metrics server
- NRF registration is only attempted when consumer and lifecycle paths exist
- config getters and service wiring describe the same runtime truth

Implication for NWDAF:

- this round should not pretend that `nrfUri` or metrics are already active
  runtime features when the current service path does not use them
- the safer alignment is to harden and document the current runtime truth first,
  then decide later whether to align upward toward full free5GC lifecycle
  behavior

---

## 4. Current NWDAF State

### 4.1 Current Factory Shape

`NWDAF/pkg/factory/config.go` currently:

- defines a broad `Configuration` struct with SBI, SMF, MTLF, ML service, ADRF,
  group mapping, analytics, and workflow-oriented settings
- applies some defaults directly inside `ReadConfig(...)`
- does not run an explicit validation phase before returning the config

This differs from the surveyed free5GC baseline, where `ReadConfig(...)`
returns an error after validation failure instead of relying on later runtime
paths to discover contradictions.

### 4.2 Confirmed Current Problems

The current code has the following confirmed config-layer problems:

1. `GetSbiBindingAddr()` is wrong for multi-digit ports because it uses rune
   conversion instead of decimal formatting.
2. `supportedAnalytics` defaults to `ABNORMAL_BEHAVIOUR` even though the
   current runtime support list is limited to `UE_COMMUNICATION`.
3. README and module/runtime truth have drifted:
   - `README.md` still says `Go 1.21+`
   - `go.mod` declares `go 1.25.5`
4. the config surface still advertises fields such as `nrfUri` without a
   matching runtime path in `pkg/service`
5. current tests in `pkg/factory` cover helper defaults well, but they do not
   yet provide a validation matrix for complete config documents

### 4.3 Constraints From Current NWDAF Architecture

This round should respect the current repository shape:

- `pkg/service` still starts a standalone SBI server without NRF registration
- `internal/sbi/server.go` binds directly from `cfg.Configuration.Sbi`
- `pkg/app` is still too narrow to serve as the shared boundary for a larger
  config/lifecycle redesign

These constraints mean the factory round should harden the current path without
quietly expanding the app or service architecture.

---

## 5. Fixed Decisions For This Round

The following decisions are treated as settled for this plan:

1. Do not add NRF registration or deregistration in this round.
   `nrfUri` is a later integration-level decision, not a prerequisite for
   fixing current config correctness.
2. Do not add a metrics server in this round.
   The reference NF metrics pattern is useful as a validation baseline, but
   introducing runtime metrics ownership is out of scope here.
3. Do not rebuild `pkg/app` in this round.
   Shared app-boundary reconstruction remains the next architectural step after
   config hardening.
4. Treat sample config, README runtime guidance, and `go.mod` as one contract.
   They should stop disagreeing about the environment and runtime shape.
5. Validate only what NWDAF already claims to support today, plus clearly
   reject contradictory settings that the current runtime cannot honor safely.

---

## 6. Scope

### 6.1 In Scope

- `NWDAF/pkg/factory/config.go`
- `NWDAF/pkg/factory/*_test.go`
- `NWDAF/config/nwdafcfg.yaml`
- `NWDAF/README.md`
- small service/server call-site adjustments if the new getters or validation
  path require them

### 6.2 Explicit Non-Goals

- `pkg/app` expansion
- `consumer.NewConsumer(app.App)` refactor
- metrics server wiring
- NRF registration/deregistration wiring
- TLS/H2C server-helper migration
- runtime config split between NF core config and lab/workflow payloads

---

## 7. Proposed Workstreams

### Workstream A - Add Explicit Factory Validation

Objectives:

- give NWDAF a real `Validate()` entrypoint in `pkg/factory`
- move obvious config contradictions from runtime paths into startup failure

Planned outcomes:

1. introduce `Config.Validate()` plus targeted sub-validators for the current
   config surface
2. run validation from `ReadConfig(...)` after defaults are applied
3. return concrete startup errors instead of only logging warnings for invalid
   structural input

Minimum validation set for this round:

- required `configuration` section after defaulting
- allowed SBI scheme values
- non-empty or defaultable SBI binding values
- valid positive SBI port
- `supportedAnalytics` values restricted to what NWDAF actually supports today
- `analytics.ueCommunication.inputWindow <= ringBufferSize`

Optional-but-desirable validation in the same round if it stays small:

- `smf.enabled` requires at least one endpoint and callback URIs
- `mlService.enabled` requires an endpoint
- `externalMtlf.enabled` requires at least one endpoint and `notifUri`
- `adrf.url`-driven settings reject impossible values such as non-positive batch
  size where the runtime depends on them

Why first:

- this gives later boundary work a stable startup contract instead of a
  partially-defaulted config blob

### Workstream B - Normalize SBI Getter Behavior

Objectives:

- fix the known address-construction bug
- align NWDAF helper structure with the surveyed free5GC getter pattern

Planned outcomes:

1. fix `GetSbiBindingAddr()` to use decimal port formatting
2. add or normalize helper separation between:
   - binding IP/address
   - register IP/address
   - scheme
3. update direct call sites to consume the normalized helpers where that keeps
   the code smaller and clearer

Why here:

- this is a confirmed bug-level issue
- it is also the lowest-risk place to make future service wiring less ad hoc

### Workstream C - Align Runtime Truth And Advertised Config

Objectives:

- stop the config and docs from claiming behavior the runtime does not actually
  provide
- align defaults with the current supported analytics path

Planned outcomes:

1. resolve `supportedAnalytics` drift by aligning defaults and validation with
   the current `UE_COMMUNICATION` runtime support
2. document `nrfUri` as reserved or currently inactive rather than implying
   active registration support
3. align `README.md`, `go.mod`, and sample config comments with current startup
   truth

Why here:

- this is the smallest safe response to the current integration-level mismatch
- it avoids pretending that full free5GC lifecycle parity already exists

### Workstream D - Add Table-Driven Config Tests

Objectives:

- follow the free5GC skill guidance for config/factory tests
- protect the new validation and defaulting rules from regression

Planned outcomes:

1. add table-driven tests in `pkg/factory` for:
   - valid minimal config
   - omitted optional fields that should default
   - unsupported analytics value
   - bad scheme
   - bad or missing SBI port/binding combination
   - `inputWindow > ringBufferSize`
2. add direct tests for normalized SBI getter behavior
3. keep helper-default tests that already exist

Why here:

- later app-boundary and service wiring work should not need to rediscover the
  factory contract by reading implementation details

---

## 8. Sequencing

Recommended implementation order:

1. define the intended validated contract for the current config surface
2. add `Validate()` and wire it into `ReadConfig(...)`
3. fix and normalize SBI getter helpers
4. align defaults and documentation with current runtime truth
5. add the config test matrix
6. run focused factory tests plus full module verification

Recommended commit grouping:

- `fix(factory): validate nwdaf runtime config`
- `fix(factory): normalize sbi address helpers`
- `test(factory): add config validation matrix`
- `docs: align nwdaf runtime guidance`

These subjects are examples, not mandatory final commit messages.

---

## 9. Acceptance Criteria

This round is considered complete when all of the following are true:

1. `ReadConfig(...)` rejects invalid or contradictory config instead of
   returning a partially-usable object silently.
2. `GetSbiBindingAddr()` is correct for normal multi-digit ports.
3. `supportedAnalytics` no longer defaults to a runtime-unsupported value.
4. README, sample config, and module version guidance match the actual current
   runtime contract.
5. `pkg/factory` has focused table-driven validation coverage for the new
   contract.
6. verification notes clearly distinguish unit/build checks from any unrun
   integration-level checks.

---

## 10. Verification Plan

Minimum verification for this round:

```bash
go test ./pkg/factory
go test ./...
make build
```

Optional if touched code spreads into startup wiring:

```bash
go test ./pkg/service ./internal/sbi/...
```

Expected reporting standard:

- list exact commands run
- state whether verification was factory-only, full unit suite, or build-only
- explicitly say that NRF registration, metrics server behavior, MongoDB-backed
  integration, and multi-NF runtime validation were not proven by this round

---

## 11. Follow-Up After This Round

This round completed on 2026-06-24.

Verification executed in `NWDAF/`:

- `go test ./pkg/factory`
- `go test ./...`
- `make build`
- `make lint`

Validation scope proven by this round:

- startup-time config validation and defaulting
- normalized SBI getter behavior at the current server/callback call sites
- README/sample-config alignment with current runtime truth

Not proven by this round:

- NRF registration or deregistration behavior
- metrics server runtime behavior
- MongoDB-backed integration behavior
- multi-NF end-to-end integration behavior

The next implementation target should be
the shared app-boundary reconstruction:

- expand `pkg/app` into the real shared contract
- collapse local `NwdafApp` interface variants
- move consumer construction toward `NewConsumer(app.App)` or an equivalent
  app-owned form

That follow-up should start from a hardened factory contract rather than from
today's loosely-validated config surface.
