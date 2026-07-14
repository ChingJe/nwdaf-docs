# NWDAF Priority 11 Phase 0 NRF NFManagement Detailed Plan

Date: 2026-07-15

Status: Phase 0 behavior decisions confirmed; heartbeat deliberately deferred,
implementation not started

Parent plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 NRF OAuth Discovery And Metrics Alignment Plan.md`

Implementation target:

- `NWDAF/`

---

## 1. Purpose And Required Outcome

Phase 0 establishes the first usable NRF NFManagement baseline for NWDAF in an
OAuth-disabled free5GC deployment:

1. construct a truthful NWDAF NF profile from validated configuration and
   context-owned runtime state
2. register the NF instance through the generated `Nnrf_NFManagement` client
3. deregister with a bounded cleanup context during normal shutdown
4. make registration, cancellation, failure, and shutdown behavior explicit
   and testable

The main goal is NRF integration. OAuth and NRF certificate support remain the
next phase, NF discovery remains a later phase, and the default-disabled
metrics server remains an independent supporting workstream. None of those
three areas may become an accidental prerequisite for Phase 0.

Periodic NFUpdate heartbeat is also excluded from this phase by an explicit
project decision. This makes Phase 0 a free5GC interoperability baseline, not
a claim of complete TS 29.510 NFManagement conformance. The standards gap is
retained as a separately tracked follow-up rather than hidden or treated as
implemented.

---

## 2. Planning Method And Evidence Boundaries

This plan deliberately separates two kinds of evidence:

1. **Normative behavior** comes from the local Release 18 3GPP corpus,
   especially TS 23.502, TS 29.510, and the exact Release 18 NFManagement
   OpenAPI attachment.
2. **Implementation shape** comes from the local free5GC NF and NRF source
   trees, with UDR as the primary NF exemplar and UDM/PCF as supporting
   exemplars.

The free5GC source shows package ownership, generated-client construction,
profile assembly, and lifecycle placement. It does not override a normative
requirement, and a recurring free5GC omission is not automatically a design
decision for NWDAF.

### 2.1 Normative References

The primary local references are:

- `nwdaf-docs/specs/TS 23.502/`:
  - clause 4.17.1, NF registration
  - clause 4.17.3, NF deregistration
  - clause 5.2.7.2, NRF service operations and NWDAF registration information
- `nwdaf-docs/specs/TS 29.510/`:
  - clause 5.2.2.2, NFRegister
  - clause 5.2.2.3, NFUpdate and heartbeat
  - clause 5.2.2.4, NFDeregister
  - clause 6.1.8, NFManagement authorization
  - Annex B, NF profile change reporting
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_NFManagement.yaml`
- `nwdaf-docs/specs/openapi/TS29571_CommonData.yaml`
- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`

The included TS 29.510 text and NFManagement YAML are version 18.11.0. They are
the normative Stage 3 baseline for this plan.

### 2.2 free5GC Convention References

Primary implementation references:

- `resources/references/free5gc-main/NFs/udr/internal/context/context.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/nrf_service.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udr/pkg/factory/config.go`

Supporting comparisons:

- corresponding NFManagement/context/lifecycle code under UDM and PCF
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nf_management_test.go`
- NRF NFManagement routes and processors under
  `resources/references/free5gc-main/NFs/nrf/internal/sbi/`

The reference tree was updated on 2026-07-14 to free5GC `main` commit
`f64135d`, two commits after v4.2.3, with NRF v1.4.5. Across the current AMF,
AUSF, BSF, CHF, NEF, NSSF, PCF, SMF, UDM, and UDR revisions, the NFManagement
consumers implement registration and deregistration but do not periodically
call NFUpdate. The current NRF accepts NF instance PATCH requests, but the
current free5GC stack does not provide a complete timer-selection, NF heartbeat
sender, expiry watchdog, and automatic suspension lifecycle.

These references establish the preferred free5GC implementation shape:

1. runtime context owns the NF instance ID, NRF URI, advertised SBI identity,
   service information, and registration-derived state
2. `internal/sbi/consumer` owns generated NRF clients and request procedures
3. generated clients are configured through a centralized constructor/cache
4. registration is an application lifecycle operation
5. shutdown attempts deregistration
6. advertised addresses use the registration address, not a wildcard binding
   address
7. service entries use generated models and include scheme, version, endpoint,
   status, and a service instance ID

Raw retry sleeps, ignored registration failures, and partial OAuth response
handling are observed behaviors, not conventions that Phase 0 must copy.
Phase 0 does adopt the recurring free5GC ordering in which NRF registration
precedes listener startup, but strengthens it with cancellation-aware retry,
immediate listener startup after registration, and explicit deregistration plus
partial-listener cleanup when local startup subsequently fails.

### 2.3 Pinned Generated-Contract Check

`NWDAF/go.mod` currently pins `github.com/free5gc/openapi v1.2.3`. Its generated
NFManagement client is older than the included Release 18 specification, so
the implementation must use the generated types while checking each Phase 0
operation against the Release 18 YAML.

The required Phase 0 surface is present in the pinned dependency:

- generated `NFProfile`, `NFService`, and `NwdafInfo` models
- register and deregister NF instance operations
- `200`/`201` registration response handling
- `204` deregistration response handling
- redirect response metadata

The pinned profile model also contains both `nfServices` and `nfServiceList`.
No confirmed Phase 0 contract gap currently requires a handwritten
standardized model. If implementation reveals a real generated-client mismatch,
it must be recorded under Priority 10 instead of being hidden behind duplicate
request structs.

---

## 3. Confirmed Standards Contract

### 3.1 NFRegister

TS 23.502 requires an NF instance becoming operative to register its NF
profile with NRF. TS 29.510 realizes this as:

```text
PUT {apiRoot}/nnrf-nfm/v1/nf-instances/{nfInstanceID}
```

The request body is an `NFProfile`. The NF selects its own globally unique NF
instance ID; TS 29.510 recommends a lowercase UUID version 4 representation.
The current NWDAF context already creates a process-lifetime UUID, so Phase 0
should retain that free5GC-aligned ownership.

Both registration success forms are valid:

- `201 Created` when the resource is created, with the created profile and a
  `Location` header
- `200 OK` when the existing NF instance resource is replaced

TS 29.510 expects the returned profile to carry an NRF-selected
`heartBeatTimer`. Current free5GC NF profiles do not request that timer and the
current NRF does not reliably assign a positive value when it is omitted.
Phase 0 records but does not operationalize that response field because
heartbeat has been explicitly deferred.

TS 29.510 says the NF should use OPTIONS before registration to determine NRF
capabilities unless those capabilities were obtained through bootstrap. The
local free5GC NRF snapshot does not expose the corresponding OPTIONS route.
Phase 0 therefore does not issue OPTIONS and uses the confirmed `nfServices`
compatibility representation directly.

### 3.2 Deferred NFUpdate Heartbeat Standards Gap

TS 29.510 requires every registered NF to periodically invoke NFUpdate as a
heartbeat. A missing heartbeat allows NRF to mark the NF `SUSPENDED`, after
which it is not discoverable. Deferring it is therefore a deliberate standards
gap, not evidence that the procedure is optional in the specification.

The updated free5GC reference establishes only partial support:

1. NRF exposes and processes PATCH on the NF instance resource.
2. NRF validates an explicitly supplied `heartBeatTimer` in the range 1 to
   3600 seconds.
3. The surveyed NF consumers do not call `UpdateNFInstance` periodically.
4. NRF does not reliably choose a positive timer when the request omits one.
5. No complete NRF expiry watchdog and automatic `SUSPENDED` transition was
   found in the current snapshot.

Phase 0 therefore does not add heartbeat configuration, a timer owner, JSON
Patch requests, heartbeat recovery, or heartbeat-specific readiness behavior.
Those items require a separate plan after coordination with free5GC maintainers
or after upstream establishes the intended end-to-end behavior.

The deferred plan must later revisit timer source and negotiation, PATCH
response handling, lost-registration recovery, OAuth protection, cancellation,
and NRF expiry semantics. This follow-up does not block Phase 0, Phase 1 OAuth,
Phase 2 discovery, or the metrics supporting workstream.

### 3.3 NFDeregister

TS 29.510 realizes deregistration as:

```text
DELETE {apiRoot}/nnrf-nfm/v1/nf-instances/{nfInstanceID}
```

The Stage 3 request has no body and succeeds with `204 No Content`. TS 23.502
describes a deregistration reason at the service-operation level, but the
Release 18 Stage 3 API and the pinned generated client provide no parameter for
that reason. Phase 0 must follow the Stage 3 contract and must not invent a
non-standard DELETE body.

### 3.4 OAuth Boundary

TS 29.510 makes OAuth authorization applicable when static authorization is
not used. Phase 0 is intentionally limited to NRF operation with OAuth
disabled. Registration itself is the free5GC bootstrap point that returns the
non-standard `customInfo.oauth2` indication used by free5GC NFs.

Phase 0 may parse and retain that indication as a handoff to Phase 1, but it
must not pretend that protected deregistration can continue without Phase 1
token and certificate support.

---

## 4. Target NF Profile

### 4.1 Mandatory NF Identity And Reachability

The initial profile must use the generated NFManagement models and include:

| Field | Phase 0 source | Required behavior |
| --- | --- | --- |
| `nfInstanceId` | runtime context | Equal the ID in the resource URI for the complete process lifetime. |
| `nfType` | constant | `NWDAF`. |
| `nfStatus` | lifecycle state | `REGISTERED` for registration. |
| `nfInstanceName` | validated NWDAF config, if populated | Advertise the configured instance name without making it an identity key. |
| `ipv4Addresses` | advertised SBI registration address | Never substitute a wildcard binding address. |
| service collection | constructed profile state | Contain only the implemented producer service described below. |
| `nwdafInfo` | supported analytics configuration | Explicitly constrain the advertised event set. |

The Release 18 schema requires at least one of FQDN, IPv4 address, or IPv6
address. The current NWDAF configuration has an advertised IPv4 address, so
Phase 0 does not need to invent FQDN or IPv6 configuration.

### 4.2 Advertised Producer Service

The profile must advertise exactly one standardized producer service:

| NF service field | Phase 0 value/source |
| --- | --- |
| service name | `nnwdaf-eventssubscription` |
| service instance ID | stable within the NF instance lifetime |
| URI scheme | validated SBI scheme |
| API version in URI | `v1` |
| API full version | a service-specific declared version, not an unrelated config-schema version by accident |
| service status | `REGISTERED` |
| endpoint protocol | TCP |
| endpoint address | advertised SBI IPv4 address |
| endpoint port | advertised SBI port |
| API prefix | full reachable SBI base URI following current free5GC convention |

The following must not appear in the Phase 0 profile:

- Analytics Information producer service
- ML Model Provision producer service
- Data Management producer service
- project-local AnLF or MTLF endpoints
- a consumer relationship such as the existing MTLF client represented as an
  NWDAF-produced service

### 4.3 NWDAF-Specific Information

`NwdafInfo.nwdafEvents` must explicitly contain only `UE_COMMUNICATION`, which
is the currently implemented standardized Events Subscription analytics event.
This field cannot simply be omitted: in the Release 18 schema, absence means
the NWDAF can serve any analytics event, which would over-advertise current
behavior.

The initial profile must not claim analytics aggregation, roaming, accuracy,
federated-learning, MTLF, data-source, or serving-area capabilities unless a
separate implementation change proves that runtime behavior.

### 4.4 Release 18 Service Map Versus free5GC Compatibility

Release 18 marks the array-form `nfServices` as deprecated and prefers the
map-form `nfServiceList`; Service-Map is a defined NFManagement feature. The
local free5GC NRF snapshot and the surveyed NF profile builders still primarily
read and write `nfServices`.

The pinned `github.com/free5gc/openapi v1.2.3` model can represent both forms,
but local free5GC NRF interoperability is not proven for the Release 18 map.
Phase 0 therefore emits only `nfServices`. It does not emit both forms or
attempt OPTIONS-based Service-Map negotiation.

---

## 5. Current NWDAF Baseline And Required Ownership Changes

The current implementation has the following usable foundations:

1. `internal/context` already creates an NF instance UUID.
2. factory configuration already contains a reserved `nrfUri` field.
3. SBI configuration already distinguishes advertised and binding addresses
   and owns HTTP/HTTPS scheme, port, and TLS data.
4. `internal/sbi/consumer.Consumer` is already the central outbound service
   aggregate.
5. `pkg/service.App` already owns server startup, partial-start cleanup,
   cancellation, waitgroups, and bounded shutdown paths.
6. `nnwdaf-eventssubscription/v1` currently accepts
   `UE_COMMUNICATION` subscriptions.

The missing Phase 0 ownership is:

| Layer | New responsibility |
| --- | --- |
| `pkg/factory` | Validate and normalize the active NRF URI and expose explicit getters. |
| `internal/context` | Own NRF base URI, advertised NF profile inputs, and registration state. |
| `internal/sbi/consumer` | Own the generated NFManagement client and register/deregister procedures. |
| `pkg/service` | Place registration, failure rollback, and deregistration in the app lifecycle. |

No consumer procedure may read mutable package-global configuration or create
an independent root context.

---

## 6. Confirmed Runtime Flow

The confirmed Phase 0 flow must preserve these invariants:

| Transition | Required invariant |
| --- | --- |
| config to context | Missing or invalid `nrfUri` fails validation; there is no implicit standalone mode. |
| initial registration | Registration is attempted before any owned listener starts, following the surveyed free5GC NF lifecycle convention. |
| retryable register failure | NWDAF remains alive with no owned listener started and retries until success or app cancellation. |
| terminal register failure | Unsupported redirect, OAuth-required response, and other classified terminal incompatibilities fail startup without starting listeners. |
| register success | Runtime state records that deregistration is required; the advertised SBI listener is started immediately, followed by the required auxiliary listeners. |
| listener startup after registration | Any listener failure stops partial local startup and triggers bounded deregistration so NRF does not retain an intentionally abandoned profile. |
| startup completion | Application startup does not return success and workers do not start until registration and all required listeners have succeeded. |
| app cancellation | Registration retry wakes immediately rather than waiting for raw sleeps. |
| shutdown | Deregistration is attempted only for a successfully registered instance and while SBI is still reachable. |
| deregister timeout | Shutdown remains bounded even when NRF is unreachable. |

Retryable transport and NRF server failures use a cancellation-aware backoff
starting at two seconds and capped at thirty seconds. They do not consume a
fixed attempt budget. The first failure is logged at error level with the NRF
URI and classified cause; subsequent failures include attempt count and next
delay without logging the full profile. Recovery is logged once with elapsed
time and attempt count. This preserves the recurring free5GC retry behavior
without copying raw `time.Sleep(...)` or producing an unbounded two-second log
storm.

The generated client should be obtained through one centralized constructor or
URI-keyed cache protected by a mutex. That constructor is also the future seam
for `sbi_metrics.SbiMetricHook`; installing the no-op-capable hook must not make
the metrics server a Phase 0 dependency.

Registration-derived mutable state should be kept behind a narrow service or
context API. At minimum it includes:

- whether the instance is currently considered registered
- any returned resource URI needed for diagnostics or validation
- the free5GC OAuth-required indication, if present

The NF instance ID must not be silently replaced merely because a `Location`
header is returned. If the response location encodes a different instance ID,
the client rejects the response as a malformed registration success. A missing
`Location` remains valid for the `200 OK` replacement response.

---

## 7. Implementation Work Packages

### P0.1 — Activate NRF Configuration And Build The Profile

Expected implementation areas:

- `NWDAF/pkg/factory/config.go`
- `NWDAF/pkg/factory/config_test.go`
- `NWDAF/config/nwdafcfg.yaml`
- `NWDAF/internal/context/`
- focused profile-construction tests

Tasks:

1. promote `nrfUri` from a reserved field to a required, explicitly validated
   runtime input; do not add an implicit or explicit standalone mode
2. normalize trailing-slash and URL handling so the generated client's
   `BasePath` produces the intended `/nnrf-nfm/v1` URI exactly once
3. copy validated NRF and advertised SBI inputs into the runtime context
4. construct the NF profile with generated models
5. use a stable service instance ID for the process lifetime
6. advertise only Events Subscription `v1` and `UE_COMMUNICATION`
7. emit the current free5GC-compatible `nfServices` array and do not also emit
   `nfServiceList`
8. omit `plmnList` for the Phase 0 same-PLMN scope and rely on the local NRF
   `defaultPlmnId`
9. follow free5GC convention by advertising the full reachable SBI base URI as
   `apiPrefix`, plus the advertised IP endpoint and port
10. define service-specific `v1` and full API version constants instead of
    deriving the API version from the configuration document version
11. add table-driven config and exact-profile tests

Completion gate:

- profile serialization proves the exact service/event truth and no runtime
  consumer needs to read factory globals

### P0.2 — Add The Generated NFManagement Consumer

Expected implementation areas:

- `NWDAF/internal/sbi/consumer/consumer.go`
- new `NWDAF/internal/sbi/consumer/nrf_service.go`
- new focused consumer tests

Tasks:

1. add consumer-owned NFManagement client construction/cache
2. set the normalized NRF base path in the generated configuration
3. keep the generated-client metrics-hook seam centralized without requiring
   metrics enablement
4. implement registration using the context-owned ID and generated profile
5. accept both `200 OK` and `201 Created`
6. validate relevant response profile fields and `Location` behavior
7. parse the free5GC `customInfo.oauth2` extension for the Phase 1 handoff
8. retry transport and retryable server failures indefinitely with the
   cancellation-aware two-to-thirty-second backoff
9. treat `307`/`308` as a clear unsupported Phase 0 deployment error rather
   than following or blindly retrying the original URI
10. treat `customInfo.oauth2=true` as a terminal Phase 0 startup error that
    explicitly requires Phase 1; record that the already-created remote
    profile may require NRF-side cleanup because Phase 0 cannot authenticate
    the protected DELETE
11. classify malformed success, terminal 4xx, and cancellation distinctly
12. implement Stage 3 DELETE deregistration with no invented reason body
13. avoid logs containing full profiles, certificate material, or future
    bearer tokens

Completion gate:

- mocked HTTP tests prove the exact method, URI, generated request body, success
  variants, error classification, and cancellation behavior

### P0.3 — Integrate Startup, Rollback, And Shutdown

Expected implementation areas:

- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/service/init_test.go`
- any narrow app interface additions required by ownership

Tasks:

1. attempt NRF registration before starting any owned listener, matching the
   surveyed free5GC NF lifecycle convention
2. keep the process alive with no owned listener started while a retryable
   registration failure is being retried
3. after registration succeeds, immediately start the advertised SBI listener
   and then the required AnLF and MTLF auxiliary listeners
4. if any listener fails after registration, stop partial local startup and
   attempt bounded deregistration before returning the startup error
5. report startup success and start workers only after registration and all
   required listeners succeed
6. stop expansion of new internal work, then deregister while SBI is still
   reachable through an owner-created bounded cleanup context
   rather than the already-canceled app context
7. stop SBI and the remaining owned servers after deregistration completes or
   reaches its timeout
8. preserve shutdown convergence when deregistration fails or times out
9. ensure cancellation or terminal failure before registration success never
   emits a false deregistration

Completion gate:

- lifecycle tests prove event ordering, failure rollback, bounded cleanup, and
  the absence of work after termination

### P0.4 — Integration Proof And Behavior Documentation

Expected documentation areas after the behavior lands:

- `NWDAF/README.md`
- `NWDAF/config/nwdafcfg.yaml` comments
- the parent Priority 11 plan and remediation status

Tasks:

1. run a local free5GC NRF with OAuth disabled
2. prove create registration and existing-resource replacement registration
3. inspect the NRF-stored profile and confirm only the intended service/event
4. confirm the request uses `nfServices`, omits `nfServiceList` and `plmnList`,
   and the local NRF applies its configured default PLMN
5. prove that temporary NRF unavailability leaves listeners unstarted and that
   NRF recovery allows registration followed by listener startup
6. force a post-registration listener startup failure and prove partial local
   cleanup plus bounded deregistration
7. terminate NWDAF and prove deregistration occurs before SBI shutdown and
   removes the NF instance resource
8. document active `nrfUri`, startup dependency, the Phase 0 OAuth limitation,
   and the explicitly deferred heartbeat gap
8. document unsupported Phase 0 redirect behavior and the same-PLMN scope

Completion gate:

- the documented configuration reproduces the register and deregister baseline
  against the local free5GC NRF snapshot without claiming heartbeat support

---

## 8. Verification Matrix

### 8.1 Factory And Profile Tests

Cover at least:

- omitted, empty, and malformed NRF URI rejection plus valid NRF URI acceptance
- HTTP and HTTPS NRF base URIs
- URI normalization and rejection of ambiguous path/query fragments
- advertised IP versus wildcard binding behavior
- exact NF type, status, service name, versions, scheme, endpoint, and service
  status
- explicit `UE_COMMUNICATION` and absence of unsupported analytics/events
- exact `nfServices` representation and absence of `nfServiceList`
- absence of request `plmnList` under the confirmed same-PLMN policy

### 8.2 Consumer Contract Tests

Use the repository's existing generated-client test pattern, including `gock`
and `InterceptH2CClient` where applicable. Cover at least:

- register `201` with profile and `Location`
- register `200` for replacement, including absence of `Location`
- malformed registration success handling
- generated request body and exact NF instance resource URI
- indefinite retry for transport/5xx responses with the two-to-thirty-second
  cancellation-aware backoff
- terminal 4xx response handling
- explicit unsupported errors for `307` and `308`
- cancellation during registration request and retry wait
- deregister `204`, unknown-resource `404`, timeout, and cancellation behavior
- terminal Phase 0 failure for an OAuth-required registration response
- explicit reporting of the possible remote-profile cleanup requirement after
  an OAuth-required registration response

### 8.3 Lifecycle Tests

Use controlled fake consumers/servers and the existing app lifecycle test seam
to prove:

- the first registration request occurs before any owned listener starts
- repeated retryable registration failure keeps the process alive with no
  owned listener started
- NRF recovery allows the same startup attempt to register and then start the
  advertised SBI listener before the auxiliary listeners
- terminal registration failure returns without starting local listeners
- post-registration listener failure stops partial listeners and attempts
  bounded deregistration
- application startup does not return success and workers do not start before
  registration and all required listeners succeed
- shutdown makes at most the intended deregistration call before SBI shutdown
- deregistration uses bounded cleanup after app cancellation
- deregistration failure does not block server/dependency cleanup indefinitely
- cancellation before registration success does not emit a false deregistration

### 8.4 Local Integration Gate

Run with OAuth disabled and verify:

1. the NF resource exists at the expected NFManagement URI
2. the stored ID and profile equal the NWDAF runtime truth
3. only `nnwdaf-eventssubscription/v1` and `UE_COMMUNICATION` are advertised
4. `nfServices` is accepted by the current NRF and `nfServiceList` is absent
5. the request omits `plmnList` and the stored profile uses the NRF default PLMN
6. the registered endpoint and advertised `apiPrefix` are reachable
7. temporary NRF unavailability is retried while owned listeners remain
   unstarted
8. successful registration is followed by reachable SBI and auxiliary
   listeners before application startup is considered complete
9. a forced post-registration listener failure causes bounded deregistration
   and partial-listener cleanup
10. normal termination causes successful DELETE before SBI becomes unreachable
11. the resource is no longer discoverable after deregistration

No heartbeat request is expected in this gate. The absence is the documented
Phase 0 scope decision, not evidence of complete TS 29.510 conformance.

### 8.5 Repository Verification

After each implementation package, run focused tests first. Before Phase 0 is
declared complete, run the repository's standard verification set:

- formatting for changed Go files
- focused factory, consumer, and service lifecycle tests
- `go test ./...`
- `make build`
- `make lint`

Any command requiring script or code execution follows the workspace elevation
rule at implementation time.

---

## 9. Commit And Review Boundaries

Keep the implementation reviewable in this order unless a dependency forces a
smaller split:

1. activate NRF config and add exact generated-model profile construction
2. add generated NFManagement register/deregister consumer behavior
3. wire registration/startup/rollback/shutdown lifecycle
4. add local NRF proof and update behavior documentation

Each commit must keep its own focused tests green. OAuth/certificate support,
NF discovery, and metrics-server completion remain separate later commits even
if the centralized generated-client constructor creates a shared seam.

---

## 10. Explicit Non-Goals

Phase 0 does not include:

1. access-token acquisition or NRF certificate verification
2. OAuth authorization middleware on inbound NWDAF routes
3. NF discovery or replacement of fixed SMF/ADRF/MTLF endpoints
4. the separately owned `/metrics` listener or metrics configuration
5. advertising additional standardized NWDAF producer services
6. registering AnLF or MTLF auxiliary endpoints
7. SCP routing
8. broad OpenAPI regeneration or locally duplicated standardized models
9. production certificate generation or secret provisioning
10. NFUpdate heartbeat, heartbeat timer configuration, and recovery behavior
11. custom NF profile update procedures

---

## 11. Confirmed Phase 0 Decisions

The following decisions were confirmed on 2026-07-14 and D4 was revised on
2026-07-15 after comparing the updated free5GC `main` reference, NRF v1.4.5,
the local Release 18 contract, and the current NWDAF lifecycle.

### D1 — NRF Configuration Is Required

`nrfUri` is required and must pass URL validation. Phase 0 does not add a
standalone switch and does not infer standalone operation from an empty value.
This configuration decision is separate from runtime reachability: temporary
NRF connection failure does not immediately terminate NWDAF.

### D2 — Emit `nfServices`

The Phase 0 profile emits only the deprecated but current-free5GC-compatible
`nfServices` array. It does not emit `nfServiceList` and does not attempt
OPTIONS-based Service-Map negotiation. The Release 18 migration remains a
documented future compatibility improvement.

### D3 — Retry Until Success Or Cancellation

Retryable transport and NRF 5xx failures do not consume a fixed attempt budget.
NWDAF remains alive without starting owned listeners and retries with
cancellation-aware backoff starting at two seconds and capped at thirty
seconds. The first failure is logged at error level, subsequent attempts
identify the retry count and next delay, and recovery is logged once. Terminal
contract or deployment incompatibilities are not retried indefinitely.

### D4 — Register Before Starting Owned Listeners

Phase 0 follows the surveyed free5GC NF lifecycle convention and completes NRF
registration before starting any owned listener. Retryable registration failure
keeps the process alive but leaves SBI, AnLF, and MTLF listeners unstarted.
After registration succeeds, NWDAF starts the advertised SBI listener first and
then the auxiliary listeners, minimizing the interval in which NRF advertises
an endpoint that is not yet reachable. Startup completion and worker startup
remain gated on all required listeners. A post-registration listener failure
stops partial local startup and attempts bounded deregistration before startup
fails.

### D5 — Do Not Follow Redirects In Phase 0

Phase 0 does not implement `307`/`308` redirect following because no explicit
redirect policy exists in the surveyed free5GC NF consumers. A redirect is
reported as a clear unsupported deployment error and is not blindly retried
against the original URI. Multi-NRF redirect support requires a later plan.

### D6 — OAuth-Required NRF Needs Phase 1

If registration reports `customInfo.oauth2=true`, Phase 0 returns a terminal
startup error and identifies Phase 1 as required without starting local
listeners. Because the indication arrives after registration and DELETE may
already require authorization, Phase 0 cannot guarantee removal of the remote
profile; it logs the instance ID and required NRF-side cleanup without logging
sensitive material. Phase 1 remains responsible for tokens, `nrfCertPem`,
authorized deregistration, and closing this cleanup gap.

### D7 — Omit PLMN From The Phase 0 Request

Phase 0 is scoped to the local same-PLMN deployment. NWDAF does not add PLMN
configuration or send root `plmnList`; the local free5GC NRF applies its
configured `defaultPlmnId`. Inter-PLMN, roaming, SNPN, or PLMN-filtered
discovery requires a later profile/config extension.

### D8 — Follow The free5GC API Prefix Convention

The NF service advertises the full reachable SBI base URI as `apiPrefix` and
also includes the advertised IP endpoint and port. `v1` and the full API
version come from service-specific constants, not from the configuration
document version even if both full versions currently read `1.0.0`.

### D9 — Deregister Before Stopping SBI

Shutdown first prevents expansion of new internal work, then attempts
deregistration with an owner-created bounded cleanup context while SBI remains
reachable. SBI and the remaining owned servers stop after deregistration
completes or reaches its timeout. Deregistration failure never makes shutdown
unbounded.
