# NWDAF Priority 11 Phase 2 NRF SMF Discovery Detailed Plan

Date: 2026-07-15

Status: Implemented and verified in the current `NWDAF` working tree; D1
through D5 and the D2 compatibility choice are resolved in Section 15;
focused, race, repeated, full-repository, build, lint, diff, and layered local
integration gates pass; post-review context ownership, cache, discovery-error,
and OAuth regression hardening also passes; implementation and documentation
commits are pending

Parent plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 NRF OAuth Discovery And Metrics Alignment Plan.md`

Completed prerequisites:

- Phase 0 NRF NFManagement: `NWDAF` commit `2d34594`
- Phase 1 OAuth lifecycle: `NWDAF` commits `3c545d0` and `74b608b`
- Phase 1 development certificate workflow: `nwdaf-resources` commit `d70c4e1`

Current remediation item:

- `Priority 11 — Implement free5GC NRF, OAuth, Discovery, And Metrics Alignment`

---

## 1. Purpose And Required Outcome

Phase 2 introduces the first NRF-backed peer-resolution path in NWDAF. Its
initial target is the SMF service already consumed by the UE communication data
collection flow:

- target NF type: `SMF`
- requester NF type: `NWDAF`
- target service: `nsmf-event-exposure`

The intended bounded outcome is:

1. the NRF consumer owns a generated `NFDiscovery` client in the same
   free5GC-style boundary as its existing `NFManagement` and `AccessToken`
   clients
2. NWDAF can query the configured NRF for registered SMF instances that offer
   `Nsmf_EventExposure`
3. the current SMF data collection procedure can obtain its service API roots
   from that discovery result instead of depending only on
   `smf.endpoints`
4. the endpoint-source migration rule is explicit in configuration and tests
5. discovery honors the Phase 1 OAuth state when NRF requires `nnrf-disc`
   authorization
6. discovery and any cache wait preserve request cancellation and application
   shutdown
7. empty, malformed, multi-instance, unavailable-NRF, expired-result, and
   migration-mode outcomes are explicit
8. a layered live local gate proves that the service advertised by a real
   free5GC SMF is discovered and contacted by NWDAF, while a temporary Event
   Exposure producer registered in the same real NRF proves successful
   subscription and cleanup semantics that the current free5GC SMF stub cannot
   provide

Phase 2 must not claim more than it implements. In particular, direct discovery
of every SMF offering `Nsmf_EventExposure` is not the same procedure as finding
the SMF currently serving a particular SUPI. Section 6 records that standards
boundary. Section 15 fixes the bounded service-catalog scope and records UDM
serving-SMF resolution as later work.

---

## 2. Governing Rules And Reference Order

This plan follows:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/source-orientation.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/nf-registration-discovery.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`

The evidence order for Phase 2 is:

1. current NWDAF consumer and SMF data collection behavior
2. the pinned `github.com/free5gc/openapi v1.2.3` generated contract
3. local Release 18 OpenAPI YAML
4. local Release 18 TS 29.510 and TS 23.288 text
5. local free5GC NF exemplars and current NRF implementation

Standards text determines procedure meaning. free5GC sources determine the
implementation shape where the standard does not prescribe Go ownership,
constructors, client caching, errors, or tests. A free5GC implementation gap is
not silently converted into an NWDAF-private protocol.

---

## 3. Normative References

### 3.1 NRF Discovery Contract

The primary Release 18 references are:

- TS 29.510 clause 5.3.2.2, `NFDiscover`
- TS 29.510 clause 6.2.3.2.3.1, `GET /nf-instances`
- TS 29.510 clause 6.2.6.2.2, `SearchResult`
- TS 29.510 clause 6.2.6.2.3, discovery `NFProfile`
- TS 29.510 clause 6.2.6.2.4, discovery `NFService`
- TS 29.510 clause 6.2.8, NFDiscovery security
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_NFDiscovery.yaml`

The Release 18 API is `Nnrf_NFDiscovery` version `1.3.4`. The basic request used
by this phase is:

```text
GET {nrfApiRoot}/nnrf-disc/v1/nf-instances
    ?target-nf-type=SMF
    &requester-nf-type=NWDAF
    &service-names=nsmf-event-exposure
```

`target-nf-type` and `requester-nf-type` are mandatory. `service-names` is
optional in the generic operation, but it is required by this plan because
NWDAF needs an SMF service API root, not merely an arbitrary SMF profile.

The success response is `200 OK` with a `SearchResult`. A successful search may
contain an empty `nfInstances` array when no registered profile matches. That
case is not a `404` service-resource error and must be handled separately from
transport or NRF failure.

The response contains `validityPeriod`, which defines how long the discovery
result may be cached. TS 29.510 says a consumer should reuse a previous result
when the new query parameters are identical and the validity period has not
expired, and should avoid simultaneous duplicate searches.

The operation supports OAuth scope `nnrf-disc`. Phase 2 reuses the Phase 1
access-token boundary rather than introducing a second OAuth mechanism.

### 3.2 NWDAF Data Collection Semantics

The primary Release 18 data collection references are:

- TS 23.288 clause 6.2.2.1, `Data Collection from NFs — General`
- TS 23.288 clause 6.2.2.4, `Procedure for Data Collection from NRF`

They establish that:

1. NWDAF consumes `Nsmf_EventExposure` for SMF data collection
2. NWDAF may use `Nnrf_NFDiscovery` dynamically or periodically to build and
   maintain a network map
3. for a specific UE without an area-of-interest procedure, NWDAF should use
   UDM `Nudm_UECM` with the SUPI, and optionally DNN and S-NSSAI, to identify
   the serving SMF
4. for an Internal Group ID, NWDAF may need to discover all SMF instances and
   subscribe to all instances of the required event-exposure source
5. for `any UE` and applicable analytics filters or area of interest, direct
   NRF discovery of matching SMFs is part of the specified procedure

The current NWDAF implementation resolves groups into SUPIs and then sends each
Supi subscription to every configured SMF endpoint. Direct NRF discovery of all
SMF Event Exposure producers can replace the endpoint catalog used by that
existing fan-out behavior, but does not by itself implement the specified UDM
serving-SMF lookup for a particular SUPI.

### 3.3 Pinned Generated Contract

NWDAF currently pins:

```text
github.com/free5gc/openapi v1.2.3
```

That module contains a TS 29.510 V17.12.0 generated NFDiscovery client with API
version `1.2.6`. It supports all fields required for the initial query:

- `TargetNfType`
- `RequesterNfType`
- `RequesterNfInstanceId`
- `ServiceNames`
- optional PLMN, S-NSSAI, DNN, SUPI, and locality fields

Its `models.SearchResult` contains `ValidityPeriod` and `NfInstances`. Its
`openapi.GetNFServiceUri(...)` helper follows the current free5GC service URI
selection rule and accepts only a matching `REGISTERED` service.

The pinned model does not expose every Release 18 response addition, including
the Release 18 `ignoredQueryParams` information. Phase 2 therefore must not
depend on those newer response fields. This is a Priority 10 contract-version
gap, not permission to create duplicate Release 18 structs in NWDAF.

### 3.4 OAuth For SMF Service Consumption

The governing references are:

- TS 29.500 clause 6.7.3, OAuth 2.0 authorization between NF service consumers
  and producers
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_AccessToken.yaml`

The standard contract defines NRF-issued bearer tokens through the OAuth 2.0
client-credentials flow. The access-token request identifies the requester NF
instance and type, target NF type, and service scope; a target NF instance ID
is optional. For this path the target is `SMF` and the scope is
`nsmf-event-exposure`.

TS 29.500 permits OAuth authorization for service operations such as
subscription creation and deletion, while the applicable release does not use
this procedure to authorize callback notifications. The standard determines
those semantics; the free5GC NEF exemplar in Section 4.3 supplies the concrete
raw-Go request-binding convention.

---

## 4. free5GC Exemplar Selection

### 4.1 Primary Exemplar — AMF Discovering SMF

Primary files:

- `resources/references/free5gc-main/NFs/amf/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/amf/internal/sbi/consumer/nrf_service.go`
- `resources/references/free5gc-main/NFs/amf/internal/sbi/consumer/smf_service.go`
- `resources/references/free5gc-main/NFs/amf/internal/util/search_nf_service.go`

AMF is the closest procedure exemplar because it discovers an SMF before
consuming an SMF service. Direct source evidence shows this shape:

1. `consumer.NewConsumer(...)` initializes generated NFManagement and
   NFDiscovery client maps
2. the NRF service caches generated clients by NRF URI behind a mutex
3. the generated NFDiscovery configuration uses the NRF base URI and the SBI
   metrics hook
4. `SendSearchNFInstances(...)` supplies target and requester NF types and
   obtains an `nnrf-disc` token context
5. `SelectSmf(...)` includes an SMF service name and only accepts a
   `REGISTERED` matching service URI
6. optional DNN, S-NSSAI, PLMN, and locality filters are added only because the
   AMF procedure owns those values

NWDAF should copy that ownership and request shape. It must not copy AMF's
PDU-session-specific filters when the current NWDAF analytics request does not
provide equivalent validated inputs.

### 4.2 Secondary Exemplar — UDM Consumer Ownership

Secondary files:

- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nrf_service.go`

UDM confirms the recurring free5GC boundary in a less protocol-heavy NF:

- one NRF consumer owns both NFManagement and NFDiscovery clients
- clients are cached by NRF URI
- `GetTokenCtx(...)` supplies `nnrf-disc` authorization
- service URI selection stays in the consumer layer rather than handlers or
  factory configuration

### 4.3 Supporting Exemplar — NEF Generic Discovery

Supporting files:

- `resources/references/free5gc-main/NFs/nef/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/consumer/nrf_service.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/processor/callback.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/processor/callback_test.go`

NEF shows a generic discovery method that sets the service-specific target NF
type, parses a matching registered service URI, and returns a profile and URI
to the calling consumer procedure. It is useful evidence for error ownership
and service-name filtering, but its package-level service map and manual URI
helper are not required in NWDAF because Phase 2 initially targets only one
service and the pinned OpenAPI module already provides the URI helper.

NEF also provides the direct free5GC exemplar needed by the existing raw SMF
client boundary. Its callback processor reads an `oauth2.TokenSource` from
`openapi.ContextOAuth2`, obtains the token, and applies
`oauth2.Token.SetAuthHeader(...)` to a raw `http.Request`; its tests verify the
exact `Authorization: Bearer ...` header. Phase 2 should reuse that narrow
request-binding shape for SMF Event Exposure POST and DELETE rather than
inventing an NWDAF-private bearer mechanism or forcing an incompatible
generated request model.

### 4.4 NRF Compatibility Evidence

The local free5GC `main` snapshot is at commit `f64135d`. The current NRF
processor:

- accepts `target-nf-type`, `requester-nf-type`, and `service-names`
- filters `nfServices` by service name and `REGISTERED` status
- returns `200 OK` with `SearchResult`
- currently sets `validityPeriod` to 100 seconds

The sample SMF configuration advertises both:

- `nsmf-pdusession`
- `nsmf-event-exposure`

This is sufficient for the Phase 2 local HTTP/H2C discovery gate. It is not
evidence for every Release 18 query parameter, enhanced discovery field,
redirect path, or selection policy.

---

## 5. Confirmed Current NWDAF Baseline

### 5.1 NRF Consumer

`NWDAF/internal/sbi/consumer/nrf_service.go` currently owns:

- generated NFManagement clients keyed by NRF URI
- generated AccessToken clients keyed by NRF URI
- a shared free5GC-style HTTP/H2C or HTTPS transport constructor
- SBI metrics hooks on generated clients
- registration retry and response validation
- protected deregistration
- caller-context access-token requests

It does not yet own an NFDiscovery client or discovery-result cache.

### 5.2 SMF Data Collection

`NWDAF/internal/sbi/processor/data_collection.go` currently:

1. requires `smf.enabled=true`
2. requires at least one configured `smf.endpoints` value
3. builds SUPI targets directly or from resolved Internal Group IDs
4. iterates every configured endpoint for every target
5. creates or reuses one SMF subscription per target, endpoint, and collection
   profile
6. reconciles existing resources against the configured endpoint set

Endpoint identity is therefore already part of runtime state:

- SMF subscription state stores `SmfEndpoint`
- target-correlation keys include the SMF endpoint
- cleanup resources store the SMF endpoint
- reconciliation treats an endpoint removed from configuration as stale

Discovery integration must update the endpoint-source boundary consistently.
Changing only the subscription creation loop while leaving reconciliation tied
to configuration would immediately release valid discovered resources.

### 5.3 Current SMF Client Boundary

`NWDAF/internal/sbi/consumer/smf_service.go` deliberately remains a raw HTTP
client because the pinned generated SMF Event Exposure model does not safely
represent all request fields used by the current NWDAF/UPF data collection
extension.

The client accepts an SMF API root and appends:

```text
/nsmf-event-exposure/v1/subscriptions
```

The URI returned from discovery must therefore remain an API root compatible
with this existing boundary. Phase 2 does not silently redefine it as a full
subscription collection URI.

### 5.4 Current Failure Boundary

Subscription creation and update call `TriggerDataCollection(...)`
synchronously before accepting the NWDAF subscription. However,
`triggerUeCommunicationCollection(...)` currently logs SMF endpoint or
subscription failures and returns no error. Phase 2 must decide and test how a
discovery failure reaches the existing analytics-runtime rollback path; it must
not appear as a successful source reconciliation with no SMF resources.

---

## 6. Standards Scope Boundary

There are two different procedures that must not be conflated.

### 6.1 Service-Catalog Discovery

The narrow NRF-first Phase 2 can discover every registered SMF offering
`Nsmf_EventExposure`, then feed those API roots into the current fan-out
collection procedure.

This is compatible with:

- the current configured-endpoint semantics
- Internal Group ID cases where all relevant source NF instances may need a
  subscription
- future `any UE` or area/filter-driven discovery work
- the user's current priority of completing NRF integration before adding
  other peer interfaces

It is not sufficient proof that the selected SMF serves a particular SUPI.

### 6.2 Serving-SMF Resolution

For a specific SUPI, TS 23.288 directs NWDAF to determine the serving SMF
through UDM `Nudm_UECM`, optionally narrowing by DNN and S-NSSAI. Implementing
that complete path requires at least:

1. NRF discovery of a UDM service
2. a generated UDM UECM consumer
3. SUPI/DNN/S-NSSAI input mapping
4. response handling for multiple PDU sessions or SMFs
5. OAuth and error behavior for the UDM call
6. new integration fixtures and live UDM state

That is a real expansion beyond an NRF-only SMF discovery phase. Phase 2 is
fixed to the bounded service-catalog transition. The complete serving-SMF
procedure remains later UDM work.

---

## 7. Target Phase 2 Design

The design below is the approved bounded direction. Section 15 records the
resolved decisions that govern it.

### 7.1 Explicit Endpoint Source

The SMF block uses one explicit source policy. Its configuration shape is:

```yaml
smf:
  enabled: true
  endpointSource: nrf       # nrf or configured
  endpoints:                # required only for configured mode
    - http://127.0.0.1:8081
```

The approved semantics are:

| Mode | Source | Automatic fallback |
| --- | --- | --- |
| `nrf` | `Nnrf_NFDiscovery` result | none |
| `configured` | existing `smf.endpoints` | not applicable |

This preserves a deliberate local/fake-SMF mode without hiding NRF, profile,
or authorization failures behind silent fallback. The sample free5GC-aligned
runtime configuration should select `nrf`; focused tests may continue to use
`configured` where discovery is not under test.

Factory validation should enforce:

1. `endpointSource` is required when SMF data collection is enabled
2. only the approved enum values are accepted
3. `configured` requires at least one valid endpoint
4. `nrf` does not require `smf.endpoints`
5. callback URI validation remains unchanged
6. omitting `endpointSource` while SMF data collection is enabled is invalid

### 7.2 NRF Consumer Ownership

Extend the existing `NrfService` rather than creating another NRF abstraction.
It should own:

- `map[string]*NFDiscovery.APIClient`
- a dedicated mutex or the current client-construction mutex
- the approved discovery-result cache and in-flight request state

The NFDiscovery client constructor should match existing NWDAF and free5GC
generated-client conventions:

1. reject an empty NRF URI
2. use `NFDiscovery.NewConfiguration()`
3. set the NRF base URI
4. install `sbi_metrics.SbiMetricHook`
5. reuse `newNRFHTTPClient(...)` so HTTP/H2C, HTTPS, timeouts, tracing, and the
   no-automatic-redirect policy stay consistent with Phase 0 and Phase 1
6. cache the generated client by NRF URI

The consumer should expose one service-specific operation rather than a
repository-wide generic discovery framework. A likely boundary is equivalent
to:

```go
DiscoverSmfEventExposure(ctx context.Context) ([]DiscoveredSmfService, error)
```

Only API roots and the minimal cache metadata are needed, so the result type
should remain private or narrow. Generated NRF profiles must not be pushed into
the processor merely to duplicate URI parsing there.

### 7.3 Discovery Request Construction

The initial request must contain:

```go
TargetNfType:    SMF
RequesterNfType: NWDAF
ServiceNames:    [NSMF_EVENT_EXPOSURE]
```

The plan recommends not adding PLMN, S-NSSAI, DNN, SUPI, locality, area, or UE
filters in the first implementation because:

- current NWDAF configuration and subscription runtime do not provide a
  complete, validated filter contract for this discovery call
- free5GC exemplars add those filters only in procedures that own the values
- direct SUPI filtering is not a substitute for the TS 23.288 UDM
  serving-SMF procedure
- the pinned response cannot report Release 18 `ignoredQueryParams`

`RequesterNfInstanceId` is optional. The surveyed AMF and UDM discovery paths
do not require it for this shape, so the first implementation should follow
that convention unless the live NRF or OAuth gate demonstrates a concrete need.

### 7.4 OAuth And Context Propagation

When Phase 1 registration state says OAuth is disabled, the generated
NFDiscovery request uses the caller context directly.

When OAuth is required, the NRF consumer obtains a token context using:

- target NF type: `NRF`
- scope: `nnrf-disc`

It then invokes the generated discovery client with that derived context.

The inbound Events Subscription request context should be threaded through the
processor data collection call rather than replaced with
`context.Background()` or `context.TODO()`. Application shutdown must also
cancel the operation. The implementation should use one context whose
cancellation covers both conditions, following the existing NWDAF lifecycle
boundary without adding an unowned goroutine.

That combined caller/application context owns discovery, access-token requests,
and SMF subscription creation while the requested state is still being
established. Once an update has synchronized its replacement observation
bindings, stale-resource cleanup is committed server-side work. Explicit
subscription deletion and rollback cleanup have the same property. Those
cleanup calls use a ten-second child of the app-owned cancellation context, not
the inbound request context, so a disconnected caller cannot strand a remote
SMF subscription after NWDAF has already removed its local state. The bounded
cleanup still stops on application shutdown and cannot outlive the NF.

### 7.5 Response Validation And URI Extraction

The consumer must distinguish:

1. NRF transport failure
2. caller cancellation or deadline
3. OAuth token failure
4. NRF ProblemDetails or unsupported redirect
5. nil or malformed generated success response
6. successful empty result
7. profiles with no registered matching service
8. profiles with a usable matching service API root

For each returned profile, use the pinned free5GC OpenAPI URI helper or its
exact established semantics:

1. require service name `nsmf-event-exposure`
2. require service status `REGISTERED`
3. prefer the generated helper's API-prefix/FQDN/IP endpoint rules
4. do not synthesize an endpoint from unrelated profile fields
5. do not append the Event Exposure version path at discovery time; the current
   SMF client remains responsible for its collection path

The approved behavior is to return every usable SMF Event Exposure API
root, de-duplicated without inventing a load-balancing score. This preserves
the current fan-out semantics. NRF response order should be preserved for
procedure behavior; tests should compare normalized sets where ordering is not
contractual.

A `200 OK` with no usable service root is a discovery result failure for the
current data collection procedure. It should not fall through to configured
endpoints in `nrf` mode.

### 7.6 Discovery Result Cache

The approved cache is deliberately narrow:

- key: complete effective query parameters plus NRF URI and OAuth mode
- value: validated discovered SMF service roots
- expiry: receipt time plus positive `SearchResult.validityPeriod`
- refresh: lazy, on the next data collection request after expiry
- stale-on-error: disabled
- background worker: none
- persistence: none

If `validityPeriod` is zero or invalid, the response is usable for the current
call but is not retained.

Concurrent identical cache misses should not produce multiple simultaneous NRF
requests. Any wait for an in-flight result must select on the caller context so
a canceled request does not wait for the first caller indefinitely. The
in-flight owner must always publish completion, including error and panic-safe
cleanup.

Cache state belongs to the NRF consumer, not factory configuration or global
NWDAF context. It is transport/procedure state and ends with the app-owned
consumer. No goroutine is needed for expiry.

No stale endpoint fallback is allowed. Serving stale profiles after NRF
failure can target deregistered or suspended NF services and would hide the
same NRF failure that the new phase is intended to expose.

### 7.7 Processor Integration

Introduce one endpoint-resolution step before SMF subscription fan-out. It
should:

1. inspect the validated endpoint-source mode
2. return a copy of configured endpoints in `configured` mode
3. invoke NRF discovery in `nrf` mode
4. return an error when the selected source produces no usable endpoints
5. never merge configured and discovered endpoints implicitly

`TriggerDataCollection(...)` should resolve the endpoint set once per
reconciliation operation and use the same set for:

- creating/reusing SMF subscriptions
- determining which existing SMF resources remain current
- releasing resources whose target or endpoint is no longer selected

This avoids a second discovery call and prevents creation and reconciliation
from observing different endpoint sets.

The current `triggerUeCommunicationCollection(...)` boundary should return an
error for discovery failure and for the condition where no SMF endpoint can be
used. That error then reaches the existing create/update rollback path. The
phase should not redesign unrelated analytics runtime error details unless the
current `analyticsRuntimeUnavailableProblem()` contract is proven inadequate.

Per-endpoint SMF subscription failure behavior should remain consistent with
the current fan-out procedure. A
partial success must be logged and represented honestly in resource state;
complete failure must not be reported as successful source reconciliation.

### 7.8 Existing Resource Migration

When an existing NWDAF subscription is updated after switching from configured
to NRF-discovered endpoints:

1. resources whose endpoint remains in the resolved set and whose collection
   profile still matches may be reused
2. resources whose endpoint is absent become stale only after the replacement
   resolution has succeeded
3. newly discovered endpoints receive new SMF subscriptions
4. stale resources are released through the existing cleanup path after
   observation binding synchronization succeeds
5. if discovery fails, the update follows the existing rollback path and must
   not eagerly delete the previous working resources
6. stale, rollback, and explicit-delete cleanup uses the bounded app-owned
   context rather than inheriting caller cancellation

Changing endpoint-source configuration requires restart; live config reload is
outside this phase.

### 7.9 SMF Producer OAuth

OAuth-protected NRF discovery and OAuth-protected consumption of the discovered
SMF are separate calls:

- NRF query scope: `nnrf-disc`, target `NRF`
- SMF subscription scope: `nsmf-event-exposure`, target `SMF`

The current raw SMF Event Exposure client does not attach an access token.
Therefore an OAuth-enabled live gate can prove protected discovery but cannot
prove protected use of the discovered SMF without additional work.

Phase 2 includes the narrow outbound SMF token path:

1. reuse the existing Phase 1 access-token request implementation
2. request `nsmf-event-exposure` scope for target `SMF`
3. at the existing raw SMF request boundary, read the
   `oauth2.TokenSource` from `openapi.ContextOAuth2`, obtain its token, and use
   `SetAuthHeader(...)` for the POST and DELETE requests
4. preserve the supplied parent context and current request timeout; normal
   subscription creation receives the caller/application context, while
   committed cleanup receives the bounded app-owned context
5. never log the token
6. keep the documented raw-client exception; do not force an unsafe generated
   request-model migration into this phase

This follows the free5GC NEF raw HTTP OAuth pattern. It is a bounded
compatibility addition, not a general token cache or a new OAuth
implementation. SMF callback notifications remain outside this outbound bearer
path: TS 29.500 does not apply this OAuth authorization procedure to
notifications in the applicable release.

### 7.10 Error And Log Contract

Logs should identify:

- endpoint source
- NRF URI without credentials or query secrets
- discovery target NF and service name
- cache hit, miss, expiry, or refresh at debug level
- number of matching profiles and usable service roots
- cancellation, transport, OAuth, NRF response, or empty-result category

Logs must not include:

- bearer tokens
- complete NF profiles
- certificate content
- full SUPI lists merely to diagnose discovery

Consumer errors should be wrapped with the operation boundary and remain
inspectable for `context.Canceled` and `context.DeadlineExceeded`. Handlers
should continue using the existing NWDAF `ProblemDetails` boundary rather than
returning raw generated-client errors.

---

## 8. Implementation Work Packages

### P2.1 — Confirm The Approved Planning Package

1. record the resolved D1 through D5 choices
2. align Sections 7, 12, 14, and 16 with those choices
3. record UDM serving-SMF resolution as out of Phase 2
4. fix omitted `endpointSource` as a configuration error
5. require the OAuth-enabled SMF integration claim

This document completes that planning package.

### P2.2 — Configuration And Endpoint Source

1. add the approved endpoint-source field and enum
2. revise `SmfConfig.validate()` for source-specific requirements
3. update sample configuration and README
4. preserve callback URI and subscription-duration behavior
5. add factory compatibility and invalid-mode tests

### P2.3 — Generated NFDiscovery Consumer

1. add generated NFDiscovery client ownership to `NrfService`
2. reuse the current NRF HTTP client factory and metrics hook
3. build the fixed initial SMF Event Exposure query
4. apply `nnrf-disc` OAuth when required
5. validate responses and extract matching registered service roots
6. classify cancellation, redirect, NRF error, malformed response, and empty
   result outcomes

### P2.4 — Cache And Concurrency Policy

1. define the exact query key
2. cache only validated results
3. honor positive `validityPeriod`
4. coalesce identical in-flight discovery requests
5. make waiters cancellation-aware
6. add no-stale-fallback tests
7. keep cache expiry lazy and goroutine-free

### P2.5 — SMF Data Collection Integration

1. resolve one endpoint set for the reconciliation operation
2. replace direct reads of `smf.endpoints` in creation and reconciliation with
   that set
3. return discovery/no-endpoint failure through the existing rollback boundary
4. preserve endpoint-based resource identity and cleanup
5. verify partial and total SMF subscription failure behavior

### P2.6 — SMF OAuth Compatibility

1. request an SMF-targeted `nsmf-event-exposure` token when OAuth is required
2. attach the token to raw POST and DELETE Event Exposure requests
3. preserve OAuth-disabled requests byte-for-byte except for unrelated runtime
   metadata
4. add token-request, bearer-header, cancellation, and secret-hygiene tests

### P2.7 — Runtime Proof And Documentation

1. run OAuth-disabled discovery against a real local NRF and SMF
2. prove NWDAF consumes the Event Exposure API root advertised by the real SMF
   and record the current SMF stub's expected `501 Not Implemented` response
3. register a temporary Event Exposure producer in the same real NRF and prove
   successful subscription and cleanup without changing either production
   repository
4. repeat discovery and both producer paths with OAuth enabled
5. prove `nnrf-disc` and `nsmf-event-exposure` authorization, including safe
   rejection of a wrong or missing producer scope
6. record the accepted NRF, SMF, and generated-contract limitations
7. update the parent plan and remediation records only after the gates pass

---

## 9. Detailed Test Plan

### 9.1 Factory Tests

Cover:

1. SMF disabled with discovery fields omitted
2. valid configured mode with endpoints
3. configured mode without endpoints
4. valid NRF mode without endpoints
5. NRF mode ignores legacy endpoints rather than merging or falling back
6. omitted endpoint source is rejected when SMF is enabled
7. unknown source value
8. invalid configured endpoint URL
9. existing callback URI validation in both modes
10. sample config load

### 9.2 Discovery Client Construction Tests

Cover:

1. empty NRF URI rejection
2. one NFDiscovery client reused for the same NRF URI
3. distinct clients for distinct NRF URIs
4. HTTP/H2C transport path
5. HTTPS transport construction
6. unsupported URI scheme
7. generated client uses the existing HTTP client factory seam

### 9.3 Query Construction Tests

Intercept the generated request and assert:

1. path is `/nnrf-disc/v1/nf-instances`
2. method is `GET`
3. `target-nf-type=SMF`
4. `requester-nf-type=NWDAF`
5. `service-names=nsmf-event-exposure`
6. unapproved PLMN, S-NSSAI, DNN, SUPI, and locality filters are absent
7. caller cancellation before dispatch prevents the request

Use the free5GC consumer-test pattern with generated models, `gock`, and H2C
interception where it matches the final client path.

### 9.4 OAuth Discovery Tests

Cover:

1. OAuth disabled sends discovery without an access-token request
2. OAuth enabled requests scope `nnrf-disc` for target `NRF`
3. the returned token is attached to the discovery request
4. token request failure prevents discovery dispatch
5. token request cancellation preserves the caller error
6. discovery cancellation after token acquisition remains observable
7. logs and errors do not expose the token

### 9.5 Response And Selection Tests

Cover:

1. one matching registered SMF service
2. multiple matching profiles
3. duplicate API roots
4. a profile without `nfServices`
5. wrong service name
6. matching service in `SUSPENDED` or non-registered state
7. usable `apiPrefix`
8. service FQDN fallback
9. profile FQDN fallback
10. IPv4 endpoint fallback
11. no usable address
12. successful empty `nfInstances`
13. nil or malformed generated response
14. NRF `400`, `401`, `403`, `404`, and `500` responses
15. `307` or `308` redirect remains explicitly unsupported
16. transport failure
17. caller deadline and application cancellation

Assertions must cover all usable roots, de-duplication, and preserved NRF
response order.

### 9.6 Cache And Concurrency Tests

1. identical query reuses an unexpired result
2. expiry causes a new NRF request
3. zero validity is not retained
4. failed responses are not cached
5. malformed responses are not cached
6. an expired result is not returned when refresh fails
7. concurrent identical misses produce one NRF request
8. a canceled waiter returns without canceling the in-flight owner
9. owner cancellation or failure releases all waiters
10. distinct query keys do not collide
11. OAuth-mode or NRF-URI changes do not reuse the wrong entry
12. race tests report no cache or client-map data race

### 9.7 Processor And Resource Reconciliation Tests

Cover:

1. configured mode preserves the current endpoint list
2. NRF mode uses discovered endpoints
3. NRF mode does not silently use configured endpoints on failure
4. a discovered endpoint is used for SMF subscribe and stored resource state
5. the same endpoint set is used for creation and reconciliation
6. unchanged discovered resources are reused
7. removed discovered endpoints become stale only after successful resolution
8. newly discovered endpoints receive subscriptions
9. discovery failure preserves the previous resources during update rollback
10. empty discovery fails the create/update reconciliation
11. complete SMF subscription failure is not reported as success
12. partial fan-out success follows the approved D3 policy
13. cleanup uses the original discovered endpoint
14. request and shutdown cancellation stop discovery and subscription creation
15. committed stale/delete cleanup is bounded by the app-owned context and is
    not canceled merely because the inbound request disconnects

### 9.8 SMF OAuth Tests

1. OAuth disabled POST and DELETE do not request or attach a token
2. OAuth enabled requests target `SMF` with scope
   `nsmf-event-exposure`
3. POST subscription carries the bearer token
4. DELETE subscription carries the bearer token
5. token failure prevents the SMF call
6. cancellation from the supplied parent is preserved across token and SMF
   requests, including pre-dispatch cancellation
7. no token appears in logs or returned errors

### 9.9 Local Integration Gate

The current local free5GC SMF advertises `nsmf-event-exposure` with the standard
API prefix, but `pkg/factory/config.go` wires the producer router under the
non-standard `/nsmf_event-exposure/v1` prefix. A request to the advertised
standard path therefore returns `404`. Directly probing the underscore-prefixed
route reaches `internal/sbi/api_eventexposure.go`, whose POST, DELETE, GET, and
PUT handlers all return `501 Not Implemented`. Requiring a route-prefix repair
and successful CRUD would expand this NWDAF phase into a separate SMF feature
or upstream contribution. The accepted gate therefore separates real-SMF
discovery and route evidence from successful producer semantics. This is not a
mocked NRF gate: both producers register in and are discovered through the same
running free5GC NRF.

OAuth-disabled real-NF gate:

1. start the current local NRF
2. start a free5GC SMF advertising `nsmf-event-exposure`
3. register NWDAF and SMF
4. issue the NWDAF flow that triggers UE communication data collection
5. capture the NRF discovery query
6. confirm the returned profile contains the expected registered SMF service
7. confirm NWDAF sends the Event Exposure subscription to the real discovered
   API root rather than a configured endpoint
8. confirm the standard discovered path reaches the real SMF and returns the
   expected `404` caused by the upstream route-prefix mismatch
9. directly probe the real SMF's underscore-prefixed route and confirm its
   handler-level `501 Not Implemented`, without treating that private typo as
   an NWDAF compatibility contract

OAuth-disabled successful-producer gate:

1. register a temporary SMF Event Exposure producer in the running real NRF
2. confirm the producer is returned by the same generated discovery path
3. confirm NWDAF successfully creates a subscription at its discovered root
4. delete or update the NWDAF subscription and confirm cleanup uses the same
   root and subscription resource identifier
5. keep the producer and all generated configuration, binaries, data, and logs
   under `/tmp`; do not modify the free5GC reference tree or NWDAF production
   code

OAuth-enabled gate:

1. start NRF with OAuth enabled under the accepted HTTP/H2C deployment mode
2. confirm NWDAF obtains `nnrf-disc` authorization
3. confirm protected discovery succeeds
4. confirm the protected standard path returns the same upstream-prefix `404`
5. use an `nsmf-event-exposure` bearer for a direct diagnostic probe of the
   real SMF's underscore-prefixed route and confirm authorization reaches its
   handler-level `501 Not Implemented`
6. confirm NWDAF obtains `nsmf-event-exposure` authorization and the temporary
   protected producer accepts subscription and deletion
7. confirm wrong or missing producer scope fails safely

A mocked discovery response or a standalone fake NRF is not sufficient runtime
proof. The temporary producer does not prove free5GC SMF CRUD support; it proves
NWDAF's success and cleanup behavior after real NRF registration and discovery.

### 9.10 Completed Verification Record

Repository verification passed before the live gate:

```text
go test -count=1 ./pkg/factory ./internal/sbi/consumer ./internal/sbi/processor ./internal/sbi
go test -count=1 ./...
go test -race -count=1 ./internal/sbi/consumer ./internal/sbi/processor
go test -count=10 ./internal/sbi/consumer ./internal/sbi/processor -run 'Discovery|SmfEndpoint|Cache|TriggerDataCollection'
make build
make lint
git diff --check
```

The layered gate then used local MongoDB, free5GC NRF v1.4.5, free5GC SMF
v1.4.4-27, the current NWDAF binary, and a temporary producer and AnLF runtime
responder built and run entirely under `/tmp`. No production source or
free5GC reference file was changed by the runtime setup.

OAuth-disabled results:

1. the exact SMF discovery query returned `200` and two matching profiles
2. the response contained both the real SMF root `http://127.0.0.2:8000` and
   temporary producer root `http://127.0.0.22:8000`
3. NWDAF logged two profiles and two endpoints and returned `201` for the
   Events Subscription request after one producer succeeded
4. the standard path on the real SMF returned `404`; the diagnostic
   underscore-prefixed path returned the expected stub `501`
5. NWDAF cleanup returned `204`, the temporary producer recorded one create
   and one delete, and graceful NRF deregistration completed

OAuth-enabled results:

1. registration reported `oauth2Required=true`
2. protected `nnrf-disc` returned `200` for the exact generated query
3. the real SMF again returned `404` on the standard path; a valid
   `nsmf-event-exposure` bearer reached the underscore-prefixed handler and
   returned `501`
4. the temporary producer accepted bearer-protected POST and DELETE, while a
   missing bearer and a valid token with the wrong scope each returned `401`
5. NWDAF returned `201` for create and `204` for delete, then obtained protected
   deregistration and stopped cleanly
6. log scans found no bearer header, access-token field, or JWT-shaped value in
   the retained live logs

These results prove NWDAF's discovery, multi-endpoint partial-success, OAuth,
successful producer, cleanup, and lifecycle behavior without claiming that the
current upstream SMF implements the standard Event Exposure resource.

Post-review hardening then reran the repository gates after adding focused
coverage for:

1. app-owned, ten-second cleanup contexts for explicit deletion and stale
   discovered-resource migration
2. reuse of unchanged discovered resources and preservation of their original
   discovered API roots
3. discovery errors across `400`, `401`, `403`, `404`, `500`, `307`, `308`,
   transport failure, malformed responses, missing NRF URI, cache-key
   separation, and owner cancellation
4. OAuth-disabled raw SMF requests, token-failure dispatch prevention, and
   caller cancellation before dispatch
5. caller and application cancellation after one SMF fan-out target succeeds,
   including immediate stop before the next target and inspectable context
   errors for the existing create/update rollback boundary

Focused, full-repository, race, ten-repeat, build, lint, and diff gates passed
again. The live gate was not repeated because this hardening did not change the
discovery query, transport contract, endpoint extraction, or bearer protocol;
the live evidence above remains the runtime gate for those unchanged paths.

---

## 10. Expected Implementation Surface

The likely Phase 2 files are:

- `NWDAF/pkg/factory/config.go`
- `NWDAF/pkg/factory/config_test.go`
- `NWDAF/config/nwdafcfg.yaml`
- `NWDAF/internal/sbi/consumer/consumer.go`
- `NWDAF/internal/sbi/consumer/nrf_service.go`
- `NWDAF/internal/sbi/consumer/nrf_service_test.go`
- `NWDAF/internal/sbi/processor/data_collection.go`
- `NWDAF/internal/sbi/processor/data_collection_test.go`
- `NWDAF/internal/sbi/processor/eventssubscription.go`
- focused Events Subscription processor or handler tests if request context is
  threaded through those signatures
- `NWDAF/internal/sbi/consumer/smf_service.go` and its tests for outbound SMF
  OAuth
- `NWDAF/README.md`

The implementation should not need:

- a second NRF package
- a generic service registry abstraction
- generated-code edits
- persistent discovery storage
- a background cache-refresh worker
- new global state

The final file list must be based on the landed design rather than treating this
estimate as mandatory scope.

---

## 11. Verification Commands

Run focused tests first, adjusted to the final affected packages:

```bash
go test -count=1 ./pkg/factory ./internal/sbi/consumer ./internal/sbi/processor ./internal/sbi
```

Run race coverage for stateful consumer and reconciliation paths:

```bash
go test -race -count=1 ./internal/sbi/consumer ./internal/sbi/processor
```

Run repository gates:

```bash
go test -count=1 ./...
make build
make lint
```

Run focused new discovery, cache, and migration tests repeatedly where timing
or concurrency is involved:

```bash
go test -count=10 ./internal/sbi/consumer ./internal/sbi/processor -run 'Discovery|SmfEndpoint|Cache|TriggerDataCollection|Cleanup|Cancellation'
```

Also run:

```bash
git diff --check
```

Script, test, build, lint, and live-service execution must follow the workspace
elevation rule. The implementation report must distinguish unit, race, build,
lint, and live integration evidence.

---

## 12. Commit And Review Boundaries

Recommended implementation commits in `NWDAF/` are:

1. configuration plus generated NFDiscovery consumer and focused tests
2. SMF data collection integration, cache policy, and reconciliation tests
3. SMF OAuth compatibility, if it remains independently reviewable from the
   discovery integration diff

The exact split may be reduced when changes are inseparable, but each commit
must build and its behavior must be clear from the subject. Do not mix metrics,
heartbeat, unrelated OpenAPI regeneration, or other peer discoveries into the
Phase 2 commits.

Documentation updates belong to the separate `nwdaf-docs/` repository and
must not be included in an `NWDAF/` commit.

The post-implementation review must compare:

- the fixed Section 15 decisions
- Release 18 request and response semantics
- AMF, UDM, and NEF exemplar boundaries
- actual config and consumer ownership
- cache and cancellation behavior
- resource reconciliation and rollback
- OAuth-disabled and OAuth-enabled claims
- tests and live evidence

---

## 13. Explicit Non-Goals

Phase 2 does not include:

1. UDM `Nudm_UECM` serving-SMF resolution
2. discovery of ADRF, MTLF, AMF, PCF, UDM, another NWDAF, or every future peer
3. `any UE`, area-of-interest, DNN, DNAI, PLMN, S-NSSAI, locality, or
   PDU-session-specific query policy
4. NRF NFStatusSubscribe/NFStatusNotify network-map maintenance
5. proactive periodic discovery refresh
6. discovery-result persistence across restarts
7. stale-profile fallback after NRF failure
8. load balancing, capacity weighting, priority scoring, circuit breaking, or
   endpoint failover beyond the current fan-out behavior
9. SCP routing, delegated discovery, inter-PLMN discovery, SEPP, or automatic
   NRF redirects
10. generated OpenAPI upgrades or handwritten Release 18 replacement models
11. removal of the accepted raw SMF Event Exposure request-model exception
12. metrics-server implementation
13. NFUpdate heartbeat
14. OAuth-enabled HTTPS mutual TLS
15. a general access-token cache
16. callback authorization changes
17. correction or implementation of the current free5GC SMF Event Exposure
    route and subscription CRUD

These gaps must remain visible in the completion record.

---

## 14. Phase 2 Completion Criteria

Phase 2 is complete only when:

1. D1 through D5 are resolved and reflected in the plan
2. the generated NFDiscovery client is owned by the existing NRF consumer
3. the query uses target `SMF`, requester `NWDAF`, and service
   `nsmf-event-exposure`
4. only registered matching service API roots are returned
5. empty, malformed, multi-instance, duplicate, unavailable-NRF, redirect, and
   cancellation outcomes are tested
6. the endpoint source and any compatibility behavior are explicit and tested
7. discovery creation and resource reconciliation use one consistent endpoint
   set
8. discovery failure reaches the existing create/update rollback boundary
9. the approved cache policy and validity behavior are tested, including race
   coverage if shared state is added
10. committed stale/delete cleanup uses a bounded app-owned context and has
    focused cancellation regression coverage
11. OAuth-disabled and OAuth-enabled NRF discovery pass
12. the accepted SMF producer OAuth scope and raw-request bearer binding are
    verified
13. the layered live gate proves that NWDAF discovers and contacts the real
    free5GC SMF Event Exposure route, records the upstream `501` limitation,
    and completes successful POST/DELETE cleanup against a temporary protected
    producer registered in the same real NRF
14. focused tests, full tests, race tests, build, lint, and diff checks pass
15. README, sample config, parent plan, and remediation records match the
    landed behavior
16. direct NRF service-catalog discovery is not misrepresented as UDM-backed
    serving-SMF resolution
17. metrics and heartbeat remain separate workstreams

---

## 15. Resolved Implementation Decisions

The Phase 2 implementation direction is approved as follows:

| Decision | Approved direction |
| --- | --- |
| D1 — standards scope | Discover every registered SMF offering `Nsmf_EventExposure` and feed the resulting roots into the current fan-out procedure. UDM `Nudm_UECM` serving-SMF resolution remains later work. |
| D2 — endpoint migration | Use explicit `endpointSource: nrf\|configured`. `nrf` never falls back to or merges configured endpoints; `configured` preserves the isolated/fake-SMF path. |
| D2 compatibility | Reject an omitted `endpointSource` when SMF data collection is enabled. Do not introduce a compatibility default. |
| D3 — multiple SMFs | Return all usable registered service roots, de-duplicate them, preserve NRF response order, and retain fan-out behavior. |
| D4 — result cache | Cache validated exact-query results for a positive `validityPeriod`, coalesce identical concurrent misses, refresh lazily, and never serve stale results after refresh failure. |
| D5 — SMF OAuth | Request target `SMF` scope `nsmf-event-exposure` and bind the bearer token to raw SMF Event Exposure POST and DELETE requests using the free5GC NEF `TokenSource`/`SetAuthHeader(...)` pattern. |

The D5 decision does not extend bearer authorization to SMF callback
notifications, does not add a token cache, and does not reopen the accepted raw
SMF request-model exception.

No implementation-direction decision remains before Phase 2 starts. Stop and
replan rather than silently expanding scope if implementation evidence shows,
for example, that `targetNfInstanceId` is required by the accepted live NRF/SMF
environment or that bearer binding cannot remain a narrow raw-request change.

---

## 16. Planned Follow-Up After Phase 2

After the bounded SMF discovery phase is complete, later plans may address:

1. UDM-backed serving-SMF resolution for individual SUPIs
2. discovery filters derived from validated PLMN, S-NSSAI, DNN, DNAI,
   locality, or area-of-interest inputs
3. NRF status subscription and network-map maintenance
4. discovery of additional standardized peer services only when NWDAF has a
   real consumer procedure for them
5. the separately downgraded metrics runtime workstream
6. the deferred NFUpdate heartbeat gap

Those items must not be used to weaken the Phase 2 completion gates or to claim
standards behavior that the current phase does not implement.
