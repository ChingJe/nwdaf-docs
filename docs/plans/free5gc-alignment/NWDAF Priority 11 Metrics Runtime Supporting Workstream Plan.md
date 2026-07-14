# NWDAF Priority 11 Metrics Runtime Supporting Workstream Plan

Date: 2026-07-14

Status: Planned and required, but non-blocking for the NRF implementation
phases; source inspection complete, implementation not started

Parent plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 NRF OAuth Discovery And Metrics Alignment Plan.md`

Remediation item:

- `Priority 11 — Implement free5GC NRF, OAuth, Discovery, And Metrics Alignment`

---

## 1. Purpose

This document is the implementation-level plan for Priority 11 supporting
Metrics Workstream M. It completes the common free5GC SBI metrics runtime, but
does not precede or block Phase 0 NRF registration, Phase 1 OAuth, or Phase 2
discovery. It may be implemented in parallel with or after those phases.

Workstream M must produce a real, separately owned `/metrics` service and working
inbound and outbound collectors. The existing
`metrics.InboundMetrics()` call alone is not completion: the pinned free5GC
metrics package does not record SBI metrics until a registry has been
initialized.

The intended outcome is:

1. optional, default-disabled free5GC-style metrics configuration
2. a service-owned metrics listener with HTTP and HTTPS support
3. inbound SBI request count and duration collection
4. outbound SBI request count and duration collection for generated clients
5. bounded problem-cause labels
6. startup, partial-startup cleanup, and shutdown behavior consistent with the
   current NWDAF lifecycle contract

Workstream M does not implement NRF behavior. The NRF phases must centralize
generated client construction so this workstream can install outbound hooks
later without redesigning registration, OAuth, or discovery procedures.

---

## 2. Governing Rules And References

This plan follows:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `free5gc-dev-skill/references/testing.md`

Primary free5GC reference paths:

- `resources/references/free5gc-main/NFs/udr/pkg/factory/config.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/server.go`

Supporting cross-NF evidence:

- AMF, AUSF, CHF, NEF, NRF, NSSF, PCF, SMF, UDM, and UDR app lifecycle,
  factory, SBI router, and consumer code under
  `resources/references/free5gc-main/NFs/`
- the BSF sample configuration for an explicitly represented optional metrics
  block
- `github.com/free5gc/util v1.3.1`, which is the metrics implementation used by
  the current NWDAF dependency graph
- `github.com/free5gc/openapi v1.2.3`, which provides the generated-client
  metrics hook contract used by NWDAF

The UDR pattern is the primary structural exemplar. NWDAF's existing stricter
listener readiness and cleanup behavior remains the lifecycle authority where
the reference NFs are weaker.

---

## 3. Confirmed Starting State

### 3.1 Configuration

`NWDAF/pkg/factory/config.go` currently has no metrics configuration, defaults,
validation, or getters. `NWDAF/config/nwdafcfg.yaml` has no metrics block.

NWDAF owns four possible listener identities after Workstream M:

1. main standardized SBI listener
2. AnLF auxiliary listener
3. MTLF auxiliary listener
4. metrics listener

Metrics validation therefore must consider all three existing owned listeners,
not only the main SBI listener used by the simpler free5GC exemplars.

### 3.2 Lifecycle

`NWDAF/pkg/service/init.go` currently owns the SBI, AnLF, and MTLF servers. The
startup path provides stronger behavior than the surveyed reference NFs:

- listener startup failure is returned synchronously
- successful return means the listener has reached readiness
- partial startup is cleaned up before returning an error
- shutdown is bounded and tracked through app-owned wait groups

Metrics integration must preserve these properties.

### 3.3 Inbound Metrics

`NWDAF/internal/sbi/server.go` already mounts:

```go
router.Use(metrics.InboundMetrics())
```

This is the correct free5GC middleware placement on the main SBI router.
However, no code currently calls `metrics.Init(...)` or
`metrics.NewServer(...)`, so the package-level SBI metric switch remains off
and the middleware is inert.

The AnLF and MTLF auxiliary servers are project-local interfaces, not SBI
producer services. They are outside the common SBI collector feature set in
this phase.

### 3.4 Outbound Metrics

The ML Model Provision consumer uses a generated free5GC OpenAPI client. The
SMF and ADRF consumers use raw `http.Client` requests.

The current generated-client snapshot contains a contract defect specific to
several NWDAF packages: for example,
`openapi/nwdaf/MLModelProvision.Configuration.SetMetrics(...)` stores the hook,
but `Configuration.Metrics()` always returns `nil`. The common
`openapi.CallAPI(...)` path calls `cfg.Metrics()`. Consequently, adding only
`SetMetrics(sbi_metrics.SbiMetricHook)` would compile but would not emit
outbound metrics.

The raw SMF and ADRF consumers cannot use the generated-client hook without a
client migration. Their disposition is a separate contract decision, not a
reason to invent a second ad hoc metrics implementation.

### 3.5 Problem Cause

`NWDAF/internal/util/problem_json.go` serializes `ProblemDetails`, but it does
not populate the Gin context value consumed by the inbound middleware for the
`cause` label. The result is an empty cause label even when the response has a
stable 3GPP cause.

---

## 4. Fixed Workstream M Decisions

The following decisions do not require another architecture vote:

1. Metrics support is required, but remains optional at deployment time and
   disabled by default.
2. The configuration shape follows the recurring free5GC
   `configuration.metrics` convention.
3. `/metrics` uses a separate listener owned by `NwdafApp`.
4. Both HTTP and HTTPS are required completion modes.
5. The existing main-SBI `metrics.InboundMetrics()` placement remains.
6. The initial collector feature set is common SBI metrics only; no speculative
   NWDAF business collectors are added.
7. Generated clients must install `sbi_metrics.SbiMetricHook` through a shared
   constructor or cache path.
8. Stable `ProblemDetails.Cause` values may be exposed as the inbound `cause`
   label. Title, detail, instance, subscriber identity, and arbitrary response
   content must not become labels.
9. Metrics listener startup must satisfy the current NWDAF synchronous
   readiness contract.
10. No metrics code may weaken partial-startup cleanup or bounded shutdown.
11. NRF generated clients must use centralized constructors so this workstream
    can install the outbound hook without changing procedure logic.

---

## 5. Pre-Implementation Decision Gates

Workstream M has two dependency gates. They are recorded now so implementation does
not conceal a dependency limitation with a partial local workaround.

### 5.1 Gate M-A — Metrics Server Transport And Readiness

Original assumption:

- construct and run `metrics.NewServer(...)` exactly as the surveyed NFs do

Contradicting evidence in `github.com/free5gc/util v1.3.1/metrics/Server.go`:

1. `Server.Run(...)` launches an internal goroutine and returns no startup
   error or readiness signal.
2. the HTTPS path passes `Tls.Key` as the first argument and `Tls.Pem` as the
   second argument to `http.Server.ListenAndServeTLS(...)`; the standard API
   requires certificate file first and private-key file second.

Therefore the pinned helper cannot directly meet both NWDAF's readiness
contract and the Workstream M HTTPS gate.

Resolution order:

1. preferred: move to or produce a compatible `free5gc/util` version that
   fixes certificate/key ordering and exposes usable startup behavior
2. acceptable if dependency work cannot land in the same round: implement a
   small NWDAF-owned adapter/server that reuses `metrics.Init(...)`, the
   returned Prometheus registry, `promhttp.HandlerFor(...)`, and
   `httpwrapper.NewHttp2Server(...)`, while applying the correct certificate
   order and the existing NWDAF readiness contract
3. not acceptable without revising the parent plan: silently limit Workstream M to
   HTTP or accept fire-and-forget startup

The implementation change must record which resolution was selected and why.
If option 2 is selected, the adapter must remain narrow: listener ownership and
readiness only, with no duplicate collector definitions.

### 5.2 Gate M-B — Generated NWDAF Client Metrics Contract

Original assumption:

- calling `SetMetrics(...)` on the existing generated ML Model Provision client
  is sufficient

Contradicting evidence:

- the pinned generated NWDAF configuration accessor returns `nil`, while
  `openapi.CallAPI(...)` obtains the hook only through that accessor

Resolution order:

1. preferred: correct or update the generated OpenAPI package and verify all
   NWDAF generated-client `Metrics()` accessors return `MetricsHook`
2. if a dependency update is not available, record a concrete Priority 10
   dependency blocker; do not claim outbound coverage from a no-op
   `SetMetrics(...)` call
3. do not fork an individual generated API call or wrap raw transport merely to
   make one metric appear

This gate applies to the current ML Model Provision client and to later NWDAF
generated NRF clients. The NRF packages in the current snapshot return the
stored hook correctly, but the shared dependency correction should still be
verified before relying on mixed generated snapshots.

### 5.3 Raw SMF And ADRF Client Disposition

Before the Workstream M completion claim, inspect the pinned generated SMF Event
Exposure and NWDAF Data Management contracts against the current requests.

For each raw standardized client path, record one result:

1. compatible: migrate it under a coordinated Priority 10 change and install
   the standard outbound hook
2. incompatible: keep the raw client temporarily and create a concrete
   Priority 10 blocker describing the missing generated model or operation

The metrics work must not silently change peer-selection, callback semantics,
payload shape, or response handling while performing this audit.

---

## 6. Target Configuration Contract

Add the following optional shape under `configuration`:

```yaml
metrics:
  enable: false
  scheme: http
  bindingIPv4: 127.0.0.1
  port: 9091
  namespace: free5gc
  tls:
    pem: cert/nwdaf.pem
    key: cert/nwdaf.key
```

The committed sample should explicitly show disabled HTTP on loopback so the
feature is discoverable without creating a new listener in the default local
deployment. Factory fallback values for an omitted or partially specified
block should follow the surveyed free5GC convention:

| Field | Factory fallback |
| --- | --- |
| `enable` | `false` |
| `scheme` | `https` |
| `bindingIPv4` | `0.0.0.0` |
| `port` | `9091` |
| `namespace` | `free5gc` |

The explicit sample and the omitted-block fallback intentionally serve
different purposes: safe local discoverability versus free5GC convention.

### 6.1 Factory Types And Getters

In `NWDAF/pkg/factory/config.go`:

1. add `Metrics *Metrics` to `Configuration`
2. add `Metrics` fields for enable, scheme, binding IPv4, port, namespace, and
   optional TLS
3. reuse the existing TLS value shape where its YAML contract is compatible
4. add constants for the factory fallbacks
5. add explicit getters for enabled state, scheme, binding address, port,
   namespace, PEM, and key
6. do not register the metrics address as an NRF-advertised SBI address

Omitting the metrics block must remain valid and must not start a listener.

### 6.2 Validation

Validation must cover:

1. supported schemes are only normalized `http` and `https`
2. binding host is valid and port is in range
3. namespace is non-empty after defaults
4. enabled HTTPS requires both PEM and key
5. a partially supplied TLS pair is invalid even if metrics are disabled or
   currently use HTTP, because it is an internally inconsistent config
6. listener conflicts are rejected when the metrics port matches an SBI,
   AnLF, or MTLF port and the IPs are equal or either address is wildcard
7. the same port on two distinct concrete local addresses may remain valid

Conflict errors should identify both listener roles and the colliding address,
so startup failures are actionable.

---

## 7. Target Runtime And Lifecycle

### 7.1 Construction

`NWDAF/pkg/service/init.go` should:

1. add a metrics server or narrow metrics-server adapter to `NwdafApp`
2. construct it only when metrics are enabled
3. use NF name `nwdaf`
4. use the configured namespace and binding
5. initialize
   `map[utils.MetricTypeEnabled]bool{utils.SBI: true}` only
6. use the normal NWDAF logger entry and TLS key-log path policy
7. return construction errors from `NewApp(...)`

Construction must occur before any owned listener accepts requests so inbound
hooks never observe a partially initialized metrics package.

### 7.2 Startup

Start order should be:

1. AnLF auxiliary listener
2. MTLF auxiliary listener
3. main SBI listener
4. metrics listener

Metrics starts last because it observes the complete running service and is
not required to bootstrap the earlier listeners. Successful app startup still
requires the enabled metrics listener to accept requests on its configured
address.

If metrics startup fails:

1. stop any partially started metrics listener
2. stop SBI, MTLF, and AnLF through the existing cleanup path
3. wait for owned goroutines
4. return the original startup error with listener context

### 7.3 Shutdown

Normal shutdown should stop request-producing and request-accepting work before
the observability endpoint:

1. stop auxiliary and SBI ingress using the established NWDAF order
2. stop the metrics listener
3. wait for all owned server goroutines

This preserves final counter updates while ingress drains. Metrics shutdown
must remain bounded and idempotent, including disabled mode and partial
startup.

---

## 8. Inbound And Outbound Instrumentation

### 8.1 Inbound

Keep `metrics.InboundMetrics()` on the main SBI engine. Do not mount it on
AnLF, MTLF, or `/metrics` itself.

Update the shared problem response helper to set the middleware context value
only when `ProblemDetails.Cause` is non-empty after trimming. The helper must
set the value before writing the response.

Verification should prove:

1. a successful registered SBI route increments request count and duration
2. a stable failure cause appears in the bounded `cause` label
3. title/detail/subscriber data do not appear in scraped labels
4. the route label uses Gin's registered route template, not a test-specific
   concrete identifier

Unknown-route behavior must be inspected for label cardinality. If the pinned
middleware falls back to raw paths, Workstream M tests must document that behavior
and either normalize the fallback safely or record an operational limitation.

### 8.2 Outbound

After Gate M-B is resolved:

1. install `sbi_metrics.SbiMetricHook` in the shared ML Model Provision
   generated-client construction path
2. add a focused test proving the configured accessor returns a non-nil hook
3. perform one generated request against a controlled peer and verify the
   outbound count and duration families change
4. apply the same constructor rule to generated NRF clients whether they landed
   before or after this workstream

A compile-only test of `SetMetrics(...)` is insufficient. The test must prove
that `openapi.CallAPI(...)` invokes the hook.

---

## 9. Implementation Slices

### Slice M1 — Configuration Contract

Primary files:

- `NWDAF/pkg/factory/config.go`
- `NWDAF/pkg/factory/config_test.go`
- `NWDAF/config/nwdafcfg.yaml`

Work:

1. add types, constants, defaults, validation, and getters
2. add all-owned-listener collision detection
3. add the explicit disabled sample block
4. cover omitted, disabled, HTTP, HTTPS, invalid TLS, invalid host/port, and
   collision cases

Gate:

- factory tests pass and existing configurations without a metrics block keep
  their prior runtime behavior

Suggested commit subject:

```text
feat(factory): add metrics runtime configuration
```

### Slice M2 — Metrics Server Ownership

Primary files:

- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/service/init_test.go`
- a narrow service-owned metrics adapter file only if Gate M-A selects that
  resolution

Work:

1. resolve Gate M-A
2. initialize the common SBI registry
3. construct, start, probe, stop, and wait for the metrics listener
4. integrate startup failure cleanup
5. cover disabled, HTTP, HTTPS, bind failure, and clean shutdown

Gate:

- enabled `/metrics` is reachable before `Start()` reports success and every
  tested termination path releases its port

Suggested commit subject:

```text
feat(service): own the metrics server lifecycle
```

### Slice M3 — SBI Labels And Generated Client Hook

Primary files:

- `NWDAF/internal/util/problem_json.go`
- focused utility tests
- `NWDAF/internal/sbi/consumer/mtlf_service.go`
- focused generated-client metrics tests
- the selected OpenAPI dependency change or blocker record for Gate M-B

Work:

1. expose bounded problem cause to the inbound middleware
2. resolve Gate M-B and install the ML Model Provision hook
3. prove inbound and outbound metric deltas through actual requests
4. record SMF and ADRF generated-client compatibility results

Gate:

- scraped data proves both inbound and outbound common SBI collectors work;
  raw standardized paths have a named Priority 10 outcome

Suggested commit subject:

```text
feat(sbi): complete common request metrics hooks
```

Do not combine these slices into one commit merely because they are developed
in one session. Each slice has an independently meaningful rollback and review
boundary.

---

## 10. Test And Verification Matrix

### 10.1 Factory Tests

Required cases:

1. omitted block uses defaults and remains disabled
2. explicit disabled sample parses
3. enabled HTTP parses and validates
4. enabled HTTPS with complete TLS parses and validates
5. HTTPS missing PEM or key fails
6. unsupported scheme fails
7. invalid host, invalid port, and empty explicit namespace fail or default as
   designed
8. conflicts with SBI, AnLF, and MTLF wildcard/equal bindings fail
9. same port on distinct concrete IP addresses follows the documented rule

### 10.2 Lifecycle Tests

Required cases:

1. disabled mode opens no metrics port
2. enabled HTTP serves `/metrics`
3. enabled HTTPS completes a TLS scrape with the configured certificate
4. occupied metrics port makes startup fail synchronously
5. metrics startup failure closes already started owned listeners
6. normal termination closes the metrics port and returns its goroutine
7. repeated or partial cleanup is safe

Use real ephemeral listeners, as the existing service lifecycle tests do.

### 10.3 Collector Tests

Required cases:

1. inbound success changes the expected counter and histogram
2. inbound problem response exposes only the intended cause
3. generated outbound success changes the expected counter and histogram
4. generated outbound error records the intended status behavior without a
   panic on a nil response
5. route and service labels remain bounded

The free5GC metrics package stores collector state in package globals. Tests
that enable or reinitialize metrics must be serialized and must not use
`t.Parallel()`. Assertions should compare before/after deltas or scrape the
test registry rather than assume process-global zero state.

### 10.4 Verification Commands

Run focused checks first:

```bash
go test ./pkg/factory
go test ./pkg/service
go test ./internal/util ./internal/sbi/consumer
go test ./internal/sbi
```

Then run the repository verification required by policy:

```bash
go test ./...
make build
make lint
```

Also perform one HTTP and one HTTPS runtime scrape against the configured
`/metrics` endpoint and inspect the resulting family names and labels.

---

## 11. Out Of Scope

Workstream M does not include:

- NRF NFManagement registration or deregistration
- NF profile construction
- OAuth token acquisition or inbound authorization middleware
- NRF trust-certificate loading
- NRF NFDiscovery
- changing fixed peer endpoint selection
- custom NWDAF business, model-quality, retraining, or inference metrics
- instrumenting the project-local AnLF and MTLF auxiliary protocols as SBI
- advertising the metrics listener in the NRF NF profile

Dependency corrections required by Gates M-A or M-B are in scope only to the
minimum extent needed to make the common metrics contract true.

---

## 12. Completion Criteria

Workstream M is complete only when all of the following are true:

1. optional metrics configuration, defaults, getters, and validation are
   implemented and documented
2. disabled mode preserves current startup behavior and opens no listener
3. enabled HTTP and HTTPS modes both serve `/metrics` separately from SBI
4. listener collisions with SBI, AnLF, and MTLF are rejected or fail with
   correct cleanup
5. metrics startup participates in synchronous readiness and partial-startup
   rollback
6. termination releases the metrics listener and all tracked goroutines
7. a real inbound SBI request changes the common inbound counter and histogram
8. a real generated-client request changes the common outbound counter and
   histogram
9. stable problem cause is visible without sensitive or high-cardinality
   labels
10. the ML Model Provision metrics accessor defect is fixed or the outbound
    completion claim remains explicitly blocked
11. every raw standardized SMF and ADRF client has a recorded Priority 10
    migration or dependency-blocker outcome
12. focused tests, `go test ./...`, build, and lint pass
13. Phase 0 generated NRF client constructors provide the centralized seam
    needed to install the verified outbound-hook path

HTTP-only service, a no-op generated-client hook, or a fire-and-forget metrics
goroutine is not Workstream M completion.

---

## 13. Implementation Handoff Checklist

Before editing code:

- [ ] select and record the Gate M-A resolution
- [ ] select and record the Gate M-B resolution
- [ ] confirm the generated-client compatibility of the raw SMF and ADRF paths
- [ ] re-check working-tree ownership in `NWDAF/` and `nwdaf-docs/`

During implementation:

- [ ] keep configuration, lifecycle, and SBI hook changes in separate commits
- [ ] preserve existing uncommitted user changes
- [ ] serialize tests that mutate free5GC metrics package globals
- [ ] avoid sensitive and unbounded metric labels
- [ ] keep metrics disabled in the committed default deployment

Before declaring completion:

- [ ] prove both HTTP and HTTPS scrapes
- [ ] prove inbound and outbound metric deltas
- [ ] prove synchronous bind-failure cleanup
- [ ] record raw client dispositions under Priority 10
- [ ] update the parent Workstream M status and evidence links
