# NWDAF Priority 11 NRF, OAuth, Discovery, And Metrics Alignment Plan

Date: 2026-07-15

Status: Phase 0 implemented and committed in `NWDAF` commit `2d34594` on
2026-07-15; Phase 1 OAuth and NRF-certificate support is complete in `NWDAF`
commits `3c545d0` and `74b608b` and `nwdaf-resources` commit `d70c4e1`; live
OAuth-disabled and OAuth-enabled HTTP/H2C gates and the complete focused
automated-test matrix passed; current NRF v1.4.5 dropping registered
`nwdafInfo` remains an accepted NRF-side limitation; discovery, metrics
runtime, and heartbeat remain later work

Historical remediation item:

- `Priority 11 — Decide The Intended free5GC Integration Level`

Current remediation item:

- `Priority 11 — Implement free5GC NRF, OAuth, Discovery, And Metrics Alignment`

Related issue record:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

---

## 1. Purpose

This plan turns the resolved Priority 11 architecture decision into bounded
implementation phases.

NWDAF will no longer treat NRF integration as an undecided optional direction.
The target is a free5GC-aligned, NRF-managed NF with:

1. NRF registration and deregistration
2. OAuth and NRF trust-certificate support
3. NRF-backed NF discovery
4. the normal free5GC SBI metrics middleware, collectors, outbound hooks, and
   separately owned metrics server

The NRF path is the implementation priority because registration establishes
NWDAF as a managed NF and discovery unlocks later standardized peer-facing
interfaces. Metrics remains required for full free5GC alignment, but it is
optional and disabled by default at deployment time and is not a technical
prerequisite for registration, OAuth, or discovery. It therefore proceeds as
an independently gated supporting workstream, either in parallel with or
after the NRF baseline. Registration and OAuth may be developed in the same
round if the dependency path remains small, but they retain separate commits,
tests, and completion gates.

TS 29.510 NFUpdate heartbeat remains a known standards requirement, but it is
deliberately deferred from this implementation line after checking free5GC
`main` commit `f64135d` and NRF v1.4.5. The current NRF provides partial PATCH
support, while the surveyed free5GC NFs do not implement the periodic sender
and the current NRF does not provide a complete timer/expiry/suspension
lifecycle. This gap remains tracked and must not be described as implemented.

---

## 2. Fixed Decisions And Constraints

The following decisions are fixed for this implementation line:

1. NWDAF will align upward as an NRF-managed NF.
2. The main SBI server's existing HTTP/HTTPS selection and TLS support remain.
3. OAuth and certificate support are part of the target, not a permanent
   omission.
4. The SBI TLS certificate and the NRF OAuth trust certificate are different
   concerns:
   - SBI TLS uses the configured NWDAF server certificate and private key
   - NRF OAuth verification uses an NRF certificate such as `nrfCertPem`
5. The complete free5GC metrics pattern is part of the target:
   - keep `metrics.InboundMetrics()` on the main SBI router
   - initialize SBI Prometheus collectors through `metrics.NewServer(...)`
   - expose them from a separately configured `/metrics` listener
   - attach `sbi_metrics.SbiMetricHook` to generated outbound SBI clients
   - start and stop the metrics server under the app lifecycle
6. Metrics configuration should follow the common optional
   `configuration.metrics` shape with `enable`, `scheme`, `bindingIPv4`,
   `port`, `namespace`, and optional TLS material. Support is required, while
   the free5GC-aligned default remains disabled until explicitly enabled.
7. The metrics binding must not duplicate the main SBI address and port, and
   enabled HTTP/HTTPS modes must both have focused configuration coverage.
8. The NRF NF profile must advertise only services that NWDAF actually exposes
   as standardized producer services.
9. For the current implementation, the advertised standardized service is
   `nnwdaf-eventssubscription` version `v1`.
10. `NwdafInfo.nwdafEvents` must reflect current runtime truth. The current
   supported event is `UE_COMMUNICATION`.
11. Project-local `AnLF` and `MTLF` auxiliary endpoints are not NRF services and
    must not be added to the NF profile.
12. Analytics Information, ML Model Provision, and Data Management services
    must not be advertised until their standardized producer APIs are actually
    implemented and verified.
13. Registration and OAuth may share one development round, but must remain
    separately reviewable and independently verifiable.
14. NRF discovery remains a separate phase and must not silently replace fixed
    peer configuration without an explicit migration rule.
15. Generated free5GC OpenAPI clients and models from the currently pinned
    `github.com/free5gc/openapi v1.2.3` dependency remain the default contract
    mechanism. Any mismatch between that dependency and the local Release 18
    specifications must be recorded under Priority 10 rather than bypassed with
    ad hoc duplicate standardized structs.
16. Metrics completion does not block the NRF, OAuth, or discovery phase gates.
    Generated NRF clients must use centralized constructors so the outbound
    hook can be installed without redesigning those procedures.
17. NFUpdate heartbeat is deferred as a non-blocking standards gap pending
    separate free5GC coordination or clearer upstream end-to-end behavior. It
    does not block Phase 0 registration/deregistration, Phase 1 OAuth, Phase 2
    discovery, or the metrics supporting workstream.

---

## 3. Planning Basis And Reference Selection

This plan follows:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/nf-registration-discovery.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/nf-specific-routing.md`
- `free5gc-dev-skill/references/testing.md`

Primary free5GC implementation exemplar:

- `resources/references/free5gc-main/NFs/udr/internal/context/context.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/nrf_service.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`

UDR is the primary exemplar because it shows the complete relationship among
context-owned NF identity, NRF consumer behavior, service startup, OAuth
handling, deregistration, and shutdown.

Supporting references:

- `resources/references/free5gc-main/NFs/udm/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/pcf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/smf/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/amf/pkg/service/init.go`
  for cross-NF metrics construction, run, and stop behavior
- `resources/references/free5gc-main/NFs/pcf/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/smf/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/server.go`
  for inbound metrics and service-group authorization placement
- `resources/references/free5gc-main/NFs/udm/internal/context/context.go`
- `resources/references/free5gc-main/NFs/pcf/internal/context/context.go`
- `resources/references/free5gc-main/NFs/smf/internal/context/context.go`
  for the common OAuth context contract
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nf_management_test.go`
  for focused NRF consumer test shape
- `resources/references/free5gc-main/NFs/nrf/internal/sbi/server.go` for NRF
  route and authorization placement
- `resources/references/free5gc-main/NFs/nrf/internal/sbi/processor/access_token.go`
  for access-token behavior and validation expectations

Normative local references:

- Release 18 TS 29.510 text and OpenAPI YAML for NFManagement, NFDiscovery,
  AccessToken, NFProfile, and NWDAF-specific profile information
- Release 18 TS 29.520 text and OpenAPI YAML for the currently implemented
  Events Subscription producer service
- Release 18 TS 29.500 and TS 29.571 material for common SBI security and data
  types

Reference code is implementation evidence, not normative proof. Where the
free5GC exemplar and the pinned generated models differ from the included
Release 18 contract, the plan must record the difference and select the target
deliberately.

### 3.1 Confirmed Cross-NF Conventions

The local free5GC reference snapshot was checked across AMF, AUSF, CHF, NEF,
NRF, NSSF, PCF, SMF, UDM, and UDR. The following are recurring conventions,
not conclusions drawn from only one exemplar:

| Area | Confirmed recurring shape | Allowed or observed variation |
| --- | --- | --- |
| Metrics ownership | The app holds `metricsServer *metrics.Server`, constructs it conditionally with `metrics.NewServer(...)`, runs it with app-owned lifecycle tracking, and stops it during termination. | Metrics are optional and default to disabled in the surveyed factory code; sample YAML exposure is not uniform. |
| Metrics collection | Main SBI routers use `metrics.InboundMetrics()`. Generated outbound client configurations commonly use `sbi_metrics.SbiMetricHook`. `metrics.NewServer(...)` initializes both inbound and outbound SBI collectors and exposes `/metrics`. | NF-specific business, NAS, or NGAP collectors differ; NWDAF initially needs the common SBI feature set only. |
| Metrics config | `configuration.metrics` commonly carries `enable`, `scheme`, `bindingIPv4`, `port`, `namespace`, and optional TLS `pem`/`key`, with factory getters and SBI/metrics bind-conflict validation. | Default values and whether the sample YAML spells out the optional block vary. |
| Runtime context | NF identity, advertised address, NRF URI, NF services/profile state, `OAuth2Required`, and `NrfCertPem` live in the NF context after config initialization. | Exact context field names and whether the complete profile or its components are stored differ by NF. |
| NF profile | Profiles use generated models, NF type/status, advertised rather than binding address, registered service entries, API version, scheme, API prefix, IP endpoint, and NF-specific info. | `ServiceInstanceId` generation and the amount of optional PLMN/slice/locality data are NF-specific. |
| NRF clients | `internal/sbi/consumer` owns generated NFManagement and NFDiscovery clients, commonly cached by NRF base URI behind a mutex. Generated client configurations install the outbound SBI metrics hook. | Some older/lightweight NFs use package-level clients or weaker context handling; those are not the preferred target. |
| Registration | Registration is an app-start lifecycle operation, uses a context-owned NF ID/profile, handles created versus updated results, and usually retries until success or cancellation. | The call is placed in `pkg/service` in some NFs and `sbi.Server.Run` in others. Retry timing and startup failure policy are not uniform. |
| OAuth bootstrap | Successful NRF registration reads `customInfo["oauth2"]` into `OAuth2Required`; contexts expose `GetTokenCtx(...)` and `AuthorizationCheck(...)`; deregistration and discovery request `nnrf-nfm` or `nnrf-disc` token context when required. | Surveyed NFs log a missing `nrfCertPem` and continue; NWDAF follows that behavior with a strong log rather than introducing a stricter startup policy. |
| Inbound authorization | Standardized producer route groups commonly attach a service-name-specific `RouterAuthorizationCheck`. Callback groups are often kept separate from producer-service authorization. | Coverage is not perfectly uniform across every NF and route. NWDAF applies the rule to its implemented producer service; the Phase 1 detailed plan resolves current collector callbacks separately from TS 29.500 notify-operation evidence. |
| Discovery | Consumers create a generated NFDiscovery request with requester/target NF types, add service or domain filters, obtain `nnrf-disc` token context, and select a `REGISTERED` service URI from returned profiles. | Cache policy, multi-instance selection, and configured-endpoint fallback are procedure-specific rather than a single free5GC-wide rule. |
| Tests | Consumer tests use generated models with `gock`/H2C interception where applicable; authorization helpers have focused valid/invalid tests; config changes use table-driven validation. | Surveyed lifecycle and metrics integration coverage is uneven, so NWDAF should add stronger focused lifecycle and `/metrics` proof rather than copy the gap. |

### 3.2 Narrow NWDAF Compatibility Adjustments

The default is to preserve the recurring free5GC shape and behavior, including
known limitations that remain functional. NWDAF makes only the following
bounded adjustments where an already approved local lifecycle or response
boundary requires them:

1. replace raw retry `time.Sleep(...)` loops with cancellation-aware timers
2. propagate caller context instead of introducing `context.TODO()` in new
   consumer paths
3. log OAuth-required operation without `nrfCertPem` strongly and continue,
   matching other NFs
4. avoid logging bearer tokens, complete profiles, or certificate material
5. retain the existing NWDAF `ProblemDetails` error boundary instead of a
   generic Gin `{"error": ...}` authorization response
6. inspect `customInfo.oauth2` for every successful registration response shape
   supported by the pinned client, not only one branch by accident
7. keep unknown-route metric labels bounded; use Gin route templates when
   available and review the raw-path fallback before exposing it operationally

Phase 1 does not add stricter expiry, issuer, subject, or audience validation
than `oauth.VerifyOAuth(...)`. Those Release 18 differences are documented as
known gaps rather than used to justify an NWDAF-private JWT implementation.

---

## 4. Current NWDAF State

The starting point on 2026-07-14 is:

1. `configuration.nrfUri` exists but is not validated as an active runtime
   dependency and is not copied into the runtime context.
2. `internal/context` owns an NF instance ID and NWDAF name, but no NRF URI,
   NF profile, OAuth requirement, or NRF trust certificate.
3. `internal/sbi/consumer` has no NFManagement, AccessToken, or NFDiscovery
   client ownership.
4. `pkg/service` starts and stops the three owned NWDAF listeners but performs
   no NRF registration or deregistration.
5. The main SBI server already uses the free5GC HTTP/2 server helper, inbound
   metrics middleware, and configured HTTP/HTTPS transport, but NWDAF does not
   call `metrics.Init(...)` or `metrics.NewServer(...)`. SBI metrics therefore
   remain disabled and no `/metrics` endpoint exists.
6. The implementation currently supports the
   `nnwdaf-eventssubscription/v1` producer surface and the
   `UE_COMMUNICATION` analytics event.
7. No NRF integration path is currently available, including in OAuth-disabled
   mode.

This means Phase 0 starts from no NRF runtime integration. The metrics
workstream separately starts from an inert middleware hook, but that gap does
not prevent NWDAF from registering with NRF.

---

## 5. Supporting Workstream M — Complete The free5GC Metrics Runtime

Metrics remains required alignment work because the inbound middleware is
already mounted and generated clients should eventually expose outbound
request metrics. It is not an NRF readiness gate: this workstream may run in
parallel with or after Phase 0, and its default-disabled runtime does not delay
registration, OAuth, discovery, or later producer-interface development.

Detailed metrics workstream plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 Metrics Runtime Supporting Workstream Plan.md`

### 5.1 Workstream M1 — Configuration And Validation

Add the common optional `configuration.metrics` shape:

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

The exact sample binding may be adjusted to avoid collisions in the local
deployment, but the field shape should remain aligned with the surveyed NFs.
The surveyed factory defaults are consistently `enable: false`, port `9091`,
scheme `https`, namespace `free5gc`, and wildcard binding when no explicit
metrics binding is present. NWDAF should use those defaults unless a documented
local deployment constraint requires a narrower binding default.

Validation and getters must cover:

1. optional omission with free5GC-aligned defaults
2. explicit enable/disable
3. `http` and `https` schemes
4. valid binding address and port
5. non-empty namespace or the `free5gc` default
6. TLS certificate/key requirements when metrics HTTPS is selected
7. rejection of a binding collision with the main SBI, AnLF, or MTLF listener,
   accounting for wildcard bindings

Metrics has its own listener identity. Its TLS configuration may reuse the
NWDAF development certificate paths when that is the deployment policy, but it
must not be confused with `nrfCertPem`, which is NRF OAuth trust material.

### 5.2 Workstream M2 — App Ownership And Lifecycle

Follow the recurring free5GC app shape:

1. add `metricsServer *metrics.Server` to the root NWDAF app/service owner
2. build `metrics.InitMetrics` with NF name `nwdaf`
3. enable the common `utils.SBI` feature set
4. construct `metrics.NewServer(...)` only when metrics are enabled
5. start the server through the existing app-owned goroutine/WaitGroup model
6. stop it explicitly during termination
7. preserve partial-startup cleanup if a later owned server fails

The current NWDAF lifecycle is stricter than some surveyed NFs about
synchronous startup and cleanup. Metrics integration must preserve that local
contract instead of starting an untracked fire-and-forget goroutine.

The pinned `github.com/free5gc/util v1.3.1` metrics server does not directly
satisfy this contract: `Run(...)` returns no startup result, and its HTTPS path
passes private key and certificate paths to `ListenAndServeTLS(...)` in the
wrong order. The detailed metrics plan records the dependency-versus-local-
adapter decision gate; HTTP-only completion is not an implicit fallback.

### 5.3 Workstream M3 — Inbound And Outbound Collection

Keep the existing main-SBI middleware:

```go
router.Use(metrics.InboundMetrics())
```

Once `metrics.NewServer(...)` initializes SBI metrics, it records:

1. inbound request count by method, route path, status, and problem cause
2. inbound request duration by method, route path, and status

Every newly created generated NRF or peer-NF client should also install:

```go
configuration.SetMetrics(sbi_metrics.SbiMetricHook)
```

This enables the common outbound request count and duration metrics by target
service, method, and status. The hook should be applied in the shared client
constructor/cache path so individual calls cannot silently bypass it.

The same audit applies to existing generated clients, including the current ML
Model Provision client. Existing raw-`http.Client` SMF and ADRF consumers are
not automatically observed by the free5GC generated-client hook. Before Phase
M is declared complete, record one of these outcomes for each standardized raw
client path:

1. migrate it to the available generated client as a coordinated Priority 10
   prerequisite and install `SbiMetricHook`, which is the preferred
   free5GC-aligned outcome
2. document that the specific path remains outside outbound SBI metric coverage
   because the pinned generated contract cannot represent it, with a concrete
   Priority 10 blocker

Do not invent an unrelated custom metrics transport merely to hide a generated
client/model gap.

The pinned generated NWDAF ML Model Provision configuration currently stores a
hook in `SetMetrics(...)` but returns `nil` from `Metrics()`, which is the
accessor used by `openapi.CallAPI(...)`. Workstream M must resolve or explicitly
block on that generated-contract defect; merely adding the setter is not proof
of outbound instrumentation.

NWDAF's `ProblemDetails` response helper should populate the Gin context value
expected by the middleware when a stable, non-sensitive cause is available.
This makes the `cause` label useful without placing full error details or
subscriber data into metrics.

### 5.4 Workstream M Verification Gate

Metrics alignment is complete only when:

1. disabled mode starts no listener and leaves normal SBI behavior unchanged
2. enabled HTTP mode serves `/metrics` on the configured separate binding
3. enabled HTTPS mode uses the configured metrics certificate and key
4. SBI/metrics binding conflicts fail validation before startup
5. one inbound request changes the expected inbound counter and histogram
6. one generated-client request changes the expected outbound counter and
   histogram
7. all generated client constructors install the outbound hook, and every
   remaining standardized raw HTTP consumer has an explicit Priority 10
   disposition
8. route labels use stable templates for registered routes
9. problem responses expose only the intended bounded cause label
10. startup failure and termination tests prove metrics server cleanup

---

## 6. Phase 0 Target NF Profile

The initial registered profile should be small and truthful.

Required profile content:

1. stable process-lifetime NF instance ID from the NWDAF runtime context
2. NF type `NWDAF`
3. configured NWDAF instance name where supported by the generated model
4. NF status appropriate for a successfully starting/registered instance
5. configured advertised SBI address rather than a wildcard binding address
6. one registered `NfService` for `nnwdaf-eventssubscription`
7. service version `v1`, with its full version defined by service-specific
   constants rather than the configuration document version
8. configured SBI scheme and advertised endpoint/port
9. service status consistent with the running producer
10. `NwdafInfo.nwdafEvents` containing `UE_COMMUNICATION`
11. the current free5GC-compatible `nfServices` array, without also emitting
    `nfServiceList`
12. the full reachable SBI base URI as `apiPrefix`, plus the advertised IP
    endpoint and port

Phase 0 omits root `plmnList` and is scoped to the local same-PLMN deployment.
The current free5GC NRF applies its configured `defaultPlmnId`; PLMN-filtered,
inter-PLMN, roaming, and SNPN behavior require a later profile/config extension.

Profile construction must reuse validated config getters and context-owned
state. It must not read mutable package-global config directly from the
consumer.

The following must not be advertised in the initial profile:

- Analytics Information
- ML Model Provision
- Data Management
- `AnLF` auxiliary HTTP endpoints
- `MTLF` auxiliary HTTP endpoints
- events other than those accepted by the current standardized producer path

Profile tests should assert the exact generated-model output, including API
prefix and endpoint serialization, before relying on an integration smoke test.

---

## 7. Phase 0 — NRF NFManagement Baseline

Detailed Phase 0 plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 Phase 0 NRF NFManagement Detailed Plan.md`

### 7.1 Configuration And Context

Phase 0 should:

1. normalize and validate `nrfUri`
2. expose active NRF configuration through explicit config getters
3. extend the runtime context with the NRF URI and constructed NF profile
4. keep NF identity and advertised service metadata context-owned
5. add only the configuration needed for registration; OAuth-only certificate
   validation belongs to Phase 1

`nrfUri` is required and must be a valid URL. Phase 0 has no standalone switch
and does not infer standalone operation from an empty value. Runtime NRF
unreachability is handled by the registration retry policy rather than by
weakening configuration validation.

### 7.2 Consumer Ownership

Add an NRF service under `internal/sbi/consumer` that owns:

1. the generated NFManagement client
2. client caching keyed by NRF base URI where appropriate
3. registration request construction and response handling
4. deregistration request construction
5. later AccessToken and NFDiscovery clients without exposing generated client
   details through the root app interface unnecessarily

The consumer should accept context-owned dependencies. It should not construct
its own independent root context or read runtime globals.

### 7.3 Registration Semantics

Registration must:

1. use the context-owned NF instance ID in the NF instance resource URI
2. send the constructed, truthful profile
3. handle both successful update and successful creation response shapes used
   by the generated client
4. absorb server-returned profile data that changes relevant Phase 0 runtime
   state
5. retain any returned resource location needed by the client contract
6. classify retryable and terminal failures explicitly
7. respect application cancellation and avoid unbounded `time.Sleep` retry
   loops
8. retry transport and retryable 5xx failures until success or cancellation,
   with cancellation-aware backoff from two seconds up to thirty seconds
9. log the first failure strongly, include bounded retry context on later
   attempts, and log recovery without producing a two-second log storm
10. report `307`/`308` as unsupported Phase 0 deployment errors rather than
    following them or blindly retrying the original URI
11. treat `customInfo.oauth2=true` as a terminal error that explicitly requires
    Phase 1, leave local listeners unstarted, and report that the already-created
    remote profile may require NRF-side cleanup until authorized deregistration
    is implemented
12. treat response statuses outside the generated `200`/`201` success contract,
    including other `2xx` values, as terminal contract incompatibilities
13. avoid logging complete token, certificate, or sensitive payload data

The pinned generated registration response does not expose the underlying HTTP
status. It therefore cannot distinguish a valid `200` response without
`Location` from a non-conforming `201` response that omitted the required
header. Phase 0 records this validation boundary and relies on the local
free5GC NRF live gate, which returned a conforming `201` with `Location`, rather
than adding a custom status-capturing transport.

Retryable failure keeps the process alive with owned listeners unstarted. It
does not fall back to standalone operation. Terminal contract or deployment
incompatibilities return a startup error without starting local listeners.

### 7.4 Startup And Shutdown Ordering

The intended lifecycle is:

1. validate configuration
2. initialize context and construct the NF profile
3. initialize consumer and processors
4. register with NRF, retrying retryable failures until success or cancellation
5. after registration succeeds, start the advertised SBI listener and then the
   required auxiliary listeners
6. complete application startup and start workers only after registration and
   all required listeners succeed
7. on shutdown, stop expansion of new internal work
8. deregister through a bounded cleanup context while SBI remains reachable
9. stop SBI and the remaining owned servers after deregistration or its timeout

This order follows the surveyed free5GC NF lifecycle convention and must be
proven with lifecycle tests. A terminal registration condition leaves local
listeners unstarted. A listener failure after registration must stop partial
local startup and attempt bounded deregistration.

### 7.5 Phase 0 Verification Gate

Phase 0 is complete only when:

1. config validation accepts a valid NRF URI and rejects missing, empty, and
   malformed values
2. profile construction tests prove the advertised service/event truth,
   `nfServices` representation, full `apiPrefix`, and PLMN omission
3. consumer tests cover registration creation, replacement, retryable failure,
   indefinite cancellation-aware retry, terminal failure, unexpected `2xx`,
   unsupported redirect, OAuth-required failure, cancellation, and
   deregistration
4. service lifecycle tests prove registration-before-listener ordering,
   post-registration listener-failure rollback, startup-completion gating,
   recovery after temporary NRF unavailability, and deregistration-before-SBI
   shutdown order
5. a local NRF smoke test passes with OAuth disabled
6. the generated request contains only the intended NWDAF service/event and
   the NRF-applied default PLMN is verified; NRF v1.4.5 dropping stored
   `nwdafInfo` is accepted for Phase 0 and remains a future NRF-side correction
7. existing HTTP and HTTPS SBI tests remain green

### 7.6 Deferred Workstream H — NFUpdate Heartbeat

Heartbeat is not part of the Phase 0 implementation or completion gate. The
detailed Phase 0 plan records the Release 18 requirement, the current free5GC
partial-support evidence, and the reasons for deferral.

A later heartbeat plan must be created only after the expected free5GC behavior
has been clarified. It must separately decide timer source/negotiation, PATCH
response handling, lost-registration recovery, OAuth behavior, cancellation,
and NRF expiry/suspension semantics. Deferral does not authorize Phase 0
documentation to claim complete TS 29.510 NFManagement conformance.

---

## 8. Phase 1 — OAuth And NRF Certificate Support

Phase 1 may follow immediately after Phase 0 or be implemented in the same
development round. It remains a separate completion gate because an
OAuth-disabled registration success does not prove secured operation.

Detailed Phase 1 plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 Phase 1 OAuth And NRF Certificate Detailed Plan.md`

### 8.1 Registration-Driven OAuth State

The registration response must be inspected for the free5GC OAuth indication,
including the relevant `customInfo.oauth2` representation used by the pinned
generated model and local NRF.

The resolved value should be stored in context-owned runtime state. The code
must not assume OAuth is enabled merely because a certificate path is present,
and it must not assume OAuth is disabled merely because registration itself was
accepted without a token.

NRF NFManagement registration is the bootstrap operation and should remain
compatible with the reference NRF's unauthenticated registration route.
Subsequent protected operations, including deregistration and discovery, must
follow the OAuth indication.

### 8.2 Certificate Configuration

Add NRF trust-certificate configuration with clear naming such as
`nrfCertPem`.

Rules:

1. do not reuse the NWDAF SBI server certificate as an implicit NRF trust
   certificate
2. keep `nrfCertPem` optional in config, like the surveyed free5GC NFs
3. if NRF reports OAuth required while the certificate is missing or unusable,
   emit a strong log and continue startup
4. allow protected inbound authorization to fail normally until deployment
   supplies usable verification material
5. place development certificate/key assets and preparation workflow in
   `nwdaf-resources`, while NWDAF config uses conventional runtime-relative
   paths such as `cert/nrf.pem`
6. keep production certificate/private-key replacement deployment owned

### 8.3 Outbound Token Acquisition

The consumer should provide the free5GC-style token context path for protected
outbound NRF calls.

It must:

1. request the correct target NF type and service scope
2. preserve caller cancellation/deadlines
3. avoid token values in logs and errors
4. handle OAuth-disabled mode without a redundant token request
5. use a valid token for protected deregistration
6. later support protected NF discovery without duplicating token logic

Preserve the free5GC `GetTokenCtx(...)` and generated-client boundary. Because
the pinned shared helper discards caller deadlines, the NWDAF NRF consumer will
make the same generated Access Token request with the caller context and return
an authorization context for the immediately following operation. Phase 1 adds
no token cache; repeated discovery can revisit cache/refresh behavior in its
own phase.

### 8.4 Inbound Authorization

OAuth-enabled inbound authorization should be attached to the implemented
standardized producer route group, beginning with
`nnwdaf-eventssubscription`.

Follow the recurring free5GC boundary: context-owned
`AuthorizationCheck(...)` delegates to `oauth.VerifyOAuth(...)`, and a
service-name-specific router check protects the producer group. Keep NWDAF's
existing `ProblemDetails` response shape and never log bearer-token content.

The pinned helper verifies RSA/RS512 signature and service scope but does not
fully enforce Release 18 expiry, issuer, subject, or audience semantics. Phase
1 accepts this free5GC compatibility behavior and records the standards gap; it
does not introduce an NWDAF-private strict claims validator.

The main `collector` callback routes remain unchanged and outside the
producer-service scope check in Phase 1. Their future authorization mechanism
is not planned in this phase.

The `AnLF` and `MTLF` auxiliary servers remain outside this NRF producer-service
authorization phase.

### 8.5 Phase 1 Verification Gate

Phase 1 is complete only when:

1. OAuth-disabled registration, producer requests, and deregistration still
   work
2. OAuth-enabled registration updates runtime OAuth state
3. protected outbound deregistration succeeds with a valid access token
4. the producer accepts a valid token with the correct service/operation scope
5. missing, malformed, incorrectly signed, and wrong-scope tokens are rejected
   through `oauth.VerifyOAuth(...)` with NWDAF's expected SBI error shape
6. missing NRF trust material produces a strong log without forcing startup
   failure, and protected authorization fails safely
7. logs and errors do not expose bearer tokens or certificate contents
8. a local NRF smoke test passes in both OAuth-disabled and OAuth-enabled
   HTTP/H2C modes
9. expiry, issuer, subject, audience, and OAuth+HTTPS/mTLS limitations remain
   documented and are not represented as full standards closure

Current result on 2026-07-15: Phase 1 is complete. Functional implementation
landed in `NWDAF` commit `3c545d0`, focused regression completion landed in
`74b608b`, and development certificate assets landed in `nwdaf-resources`
commit `d70c4e1`. The completed matrix covers Access Token
transport/pre-dispatch cancellation, invalid PEM content, OAuth-enabled
listener rollback through token-protected deregistration, and production
`NewServer(...)` route wiring in addition to the OAuth-disabled and
OAuth-enabled live NRF gates.

---

## 9. Phase 2 — NRF Discovery

Discovery should be implemented only after registration and OAuth operation are
stable.

The first target should be SMF discovery because the current NWDAF data
collection flow already depends on SMF endpoints.

Phase 2 should:

1. add generated NFDiscovery client ownership to the NRF consumer service
2. query using the correct requester NF type, target NF type, and service name
3. include available PLMN, slice, UE, or service filters only when supported by
   current runtime input and the applicable discovery contract
4. select endpoints deterministically from the discovery response
5. define caching, refresh, empty-result, and stale-result behavior
6. reuse OAuth token acquisition when NRF requires it
7. preserve caller deadlines and application cancellation

Before implementation, decide how discovery interacts with fixed peer
configuration:

1. discovery replaces configured SMF endpoints
2. configuration overrides discovery
3. discovery is primary with an explicit configured fallback
4. a transition mode selects behavior explicitly

The choice must be visible in configuration and tests. Silent fallback can hide
NRF or profile errors and should not be introduced accidentally.

Additional NF types should be added only after the SMF path has a complete
query, selection, caching, and failure policy.

### 9.1 Phase 2 Verification Gate

Phase 2 is complete for SMF only when:

1. focused tests cover query construction and endpoint selection
2. empty, malformed, multi-instance, unavailable-NRF, and cancellation cases
   have explicit outcomes
3. the configured endpoint migration/fallback rule is tested
4. OAuth-enabled and OAuth-disabled discovery both pass
5. a local NRF/SMF integration test demonstrates discovery of the service
   actually consumed by NWDAF

---

## 10. Expected Implementation Surface

The likely implementation surface includes:

- `NWDAF/pkg/factory/config.go`
- `NWDAF/pkg/factory/config_test.go`
- `NWDAF/config/nwdafcfg.yaml`
- `NWDAF/internal/context/context.go`
- `NWDAF/internal/sbi/consumer/consumer.go`
- a new NRF-focused file under `NWDAF/internal/sbi/consumer/`
- focused NRF consumer tests under `NWDAF/internal/sbi/consumer/`
- `NWDAF/internal/sbi/server.go`
- focused authorization middleware tests under `NWDAF/internal/sbi/`
- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/service/init_test.go`
- `NWDAF/README.md`

The final file list should remain right-sized. In particular, the work should
not introduce a generic security framework or a second app boundary merely to
wrap the generated free5GC clients.

---

## 11. Verification Strategy

Each implementation phase should run focused tests first, followed by the
repository-wide development-policy gates.

Expected focused commands, adjusted to the final package split:

```bash
go test ./pkg/factory ./internal/context ./internal/sbi/consumer ./internal/sbi ./pkg/service
```

Repository gates:

```bash
go test ./...
make build
make lint
```

Runtime integration evidence should include:

1. metrics-disabled startup with no metrics listener
2. metrics-enabled `/metrics` output with proven inbound and outbound changes
3. local NRF with OAuth disabled
4. local NRF with OAuth enabled and the configured NRF certificate
5. inspection of the registered NWDAF profile
6. graceful deregistration evidence
7. positive and negative inbound token cases
8. SMF discovery evidence in Phase 2

An OAuth-disabled unit test alone is not sufficient proof for Phase 1, and a
mocked discovery response alone is not sufficient proof for the Phase 2 runtime
gate.

---

## 12. Resolved Phase 1 And Future Decision Gates

Phase 0 decisions are resolved in its detailed plan. Phase 1 decisions are
also resolved in the Phase 1 detailed plan:

1. preserve the free5GC token-context boundary but use caller context for the
   generated Access Token request
2. use `oauth.VerifyOAuth(...)` inbound acceptance semantics
3. strongly log missing `nrfCertPem` and continue startup
4. validate against the common OAuth-enabled HTTP/H2C NRF deployment and defer
   OAuth+HTTPS mutual TLS
5. keep development certificate/key assets in `nwdaf-resources` while runtime
   config uses conventional `cert/...` paths
6. leave collector callback authorization unchanged and unplanned in Phase 1

No additional Phase 1 decision is required before implementation. Remaining
future decisions are:

1. the Priority 10 disposition for existing standardized raw HTTP SMF and ADRF
   consumers that cannot use `SbiMetricHook` directly
2. Phase 2 fixed-endpoint replacement, override, or fallback behavior
3. initial discovery filters and endpoint selection policy for multiple SMFs

---

## 13. Explicitly Out Of Scope

The following are outside Priority 11 unless a later plan adds them explicitly:

1. custom NWDAF business metrics beyond the common SBI inbound/outbound set
2. NAS, NGAP, PFCP, or UPF metrics that do not apply to this NF
3. exposing `/metrics` on the main SBI listener instead of the free5GC-style
   separately owned metrics server
4. advertising project-local `AnLF` or `MTLF` endpoints as NRF services
5. claiming unsupported NWDAF standardized services in the NF profile
6. SCP routing integration
7. unrelated analytics behavior changes
8. broad OpenAPI regeneration or handwritten contract replacement owned by
   Priority 10
9. automatic generation or repository storage of production certificates or
   private keys; development assets may be tracked in `nwdaf-resources`
10. immediate discovery integration for every peer NF in one change
11. refactoring auxiliary-server authentication without applicable contract
    evidence
12. strict NWDAF-private JWT claim validation beyond the selected free5GC
    helper semantics
13. OAuth-enabled HTTPS mutual TLS in Phase 1

---

## 14. Overall Completion Criteria

Priority 11 is complete only when:

1. NWDAF registers a truthful NF profile with NRF and deregisters cleanly
2. startup, retry, cancellation, and shutdown behavior are explicit and tested
3. both OAuth-disabled and OAuth-enabled modes work
4. protected outbound NRF calls obtain and use tokens correctly
5. implemented standardized producer routes enforce the intended token and
   scope checks when OAuth is enabled
6. the NRF trust certificate is configured separately from SBI TLS server
   identity, and a missing certificate is reported strongly without changing
   the selected free5GC startup behavior
7. SMF discovery has an explicit endpoint migration/fallback policy and passes
   its separate integration gate
8. the NF profile does not advertise unsupported services, events, or
   auxiliary endpoints
9. NWDAF provides the free5GC-style optional metrics server, `/metrics`, SBI
   collectors, inbound middleware, outbound client hooks, and owned lifecycle
10. metrics-disabled, metrics-enabled HTTP, and metrics-enabled HTTPS modes are
    validated
11. focused tests, `go test ./...`, `make build`, and `make lint` pass
12. current README, config comments, and remediation status match the landed
    behavior
13. the deferred NFUpdate heartbeat gap remains explicitly documented and is
    not represented as completed standards conformance
14. the selected free5GC OAuth helper limitations and deferred OAuth+HTTPS/mTLS
    support remain explicit and are not represented as full Release 18
    security conformance

Until these conditions are met, the direction may be considered decided, but
Priority 11 implementation must remain open.
