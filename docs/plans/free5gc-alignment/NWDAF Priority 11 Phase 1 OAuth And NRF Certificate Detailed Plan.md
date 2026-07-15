# NWDAF Priority 11 Phase 1 OAuth And NRF Certificate Detailed Plan

Date: 2026-07-15

Status: Complete; Phase 0 is implemented in `NWDAF` commit `2d34594`; Phase 1
functional implementation is committed as `3c545d0`, focused regression
completion as `74b608b`, and development certificate assets as
`nwdaf-resources` commit `d70c4e1`; the known free5GC helper and
OAuth+HTTPS/mTLS gaps remain documented

Parent plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 NRF OAuth Discovery And Metrics Alignment Plan.md`

Implemented prerequisite:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 11 Phase 0 NRF NFManagement Detailed Plan.md`

Remediation item:

- `Priority 11 — Implement free5GC NRF, OAuth, Discovery, And Metrics Alignment`

---

## 1. Purpose

This document defines the implementation-level plan for Priority 11 Phase 1.
It extends the completed Phase 0 NRF registration lifecycle so NWDAF can remain
running when the NRF reports `customInfo.oauth2=true`, use an access token for
protected NRF NFManagement operations, and authorize requests to the currently
implemented `nnwdaf-eventssubscription` producer API.

The intended Phase 1 outcome is:

1. registration remains the bootstrap operation and determines whether OAuth
   is required
2. `nrfCertPem` is configured separately from the NWDAF SBI TLS certificate
3. protected NRF deregistration obtains and sends a token with the `nnrf-nfm`
   scope
4. the Events Subscription producer routes enforce the
   `nnwdaf-eventssubscription` scope when OAuth is required
5. inbound authorization follows the free5GC `AuthorizationCheck(...)` and
   `oauth.VerifyOAuth(...)` acceptance model while keeping NWDAF's existing
   `ProblemDetails` response boundary and secret-safe logging
6. caller cancellation, bounded shutdown, and Phase 0 registration ordering
   remain intact
7. both OAuth-disabled and OAuth-enabled local NRF modes are verified

Phase 1 is not a replacement for transport security. The NRF signing
certificate used to validate access tokens, the TLS trust used to connect to an
HTTPS NRF, and the NWDAF server certificate/private key are separate security
concerns. The common free5GC OAuth-over-HTTP/H2C deployment is the Phase 1
integration target; OAuth-enabled HTTPS/mTLS is explicitly deferred.

---

## 2. Governing Rules And References

This plan follows:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/nf-registration-discovery.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `free5gc-dev-skill/references/testing.md`

Local normative sources:

- Release 18 TS 29.500 clause 6.7 for SBI security and OAuth behavior
- Release 18 TS 29.510 clauses 6.1.8 and 6.3 for NFManagement authorization
  and the Access Token service
- Release 18 TS 29.520 clause 5.1.9 for Events Subscription scopes
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_AccessToken.yaml`
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_NFManagement.yaml`
- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`

Generated contract baseline:

- `github.com/free5gc/openapi v1.2.3`
- `github.com/free5gc/util v1.3.1`

The generated dependency remains the implementation contract. Phase 1 keeps
its Access Token and NFManagement request/response types instead of introducing
duplicate standardized payloads. Differences from Release 18 are documented;
the only local adaptation is preserving the caller context around the generated
Access Token request for the already approved bounded shutdown lifecycle.

---

## 3. Exemplar Selection And Evidence Boundary

Primary free5GC exemplar:

- UDR
  - `NFs/udr/internal/context/context.go`
  - `NFs/udr/internal/sbi/consumer/nrf_service.go`
  - `NFs/udr/internal/sbi/server.go`
  - `NFs/udr/internal/util/router_auth_check.go`
  - `NFs/udr/internal/util/router_auth_check_test.go`

UDR is the primary exemplar because it shows the smallest complete relationship
among context-owned OAuth state, NRF token acquisition, token-protected
deregistration, producer-route authorization, and focused middleware tests.

Supporting exemplars:

- UDM, PCF, and SMF context methods for the recurring `GetTokenCtx(...)` and
  `AuthorizationCheck(...)` boundary
- PCF and SMF SBI server construction for service-group authorization and
  callback grouping
- NRF v1.4.5 server and processor code for registration bootstrap exclusion,
  protected NFManagement routes, Access Token request validation, signing, and
  certificate handling

Reference snapshot:

- free5GC main commit `f64135d`
- NRF submodule commit `8e567d4` (`v1.4.5`)
- UDR submodule commit `b649f90` (`v1.4.4`)
- UDM submodule commit `bdcc8bc` (`v1.4.5`)
- PCF submodule commit `90d88f1`
- SMF submodule commit `796a2de`

Directly reusable free5GC conventions are:

1. registration is excluded from NRF authorization because it bootstraps OAuth
   state
2. successful registration supplies `customInfo.oauth2`
3. OAuth state and NRF certificate configuration belong to the NF runtime
   context
4. protected NRF calls use a token scoped to the target NRF service
5. standardized producer route groups use service-specific authorization
   middleware
6. generated Access Token and NFManagement clients remain consumer-owned

Section 6 records limitations in the pinned helpers and current reference NRF.
They do not make the free5GC approach unusable. Phase 1 follows the established
free5GC boundaries and behavior, with one narrow outbound adaptation required
to preserve NWDAF's already approved bounded shutdown context.

---

## 4. Normative Security Baseline

### 4.1 OAuth Is A Local-Policy Mode

TS 29.500 and TS 29.520 allow OAuth authorization based on local
configuration. The Release 18 Events Subscription OpenAPI contract represents
this with both an empty security alternative and the
`oAuth2ClientCredentials` alternative.

For this NWDAF/free5GC integration line, the NRF registration response is the
runtime authority for the local mode:

- `customInfo.oauth2=false` keeps the OAuth-disabled behavior
- `customInfo.oauth2=true` enables protected outbound NRF calls and inbound
  producer authorization

Phase 1 does not add a second NWDAF boolean that can silently disagree with the
NRF. Configuration supplies certificate material and, if later approved,
transport credentials; it does not override the registration result.

### 4.2 Access Token Request

The Access Token request uses the client-credentials grant and the generated
contract. For protected deregistration, the required request values are:

| Field | Phase 1 value |
| --- | --- |
| `grant_type` | `client_credentials` |
| `nfInstanceId` | the process-lifetime NWDAF NF instance ID |
| `nfType` | `NWDAF` |
| `targetNfType` | `NRF` |
| `scope` | `nnrf-nfm` |

PLMN, S-NSSAI, target NF set, source NF, inter-PLMN fields, and
`targetNfInstanceId` remain unset in the same way as the surveyed free5GC NF
`GetTokenCtx(...)` calls. Phase 0 does not learn the NRF's NF instance ID, and
Phase 1 does not add a separate identity-discovery mechanism solely to fill
that optional request field.

The generated response must contain a non-empty access token. Phase 1 uses it
for the immediately following protected operation and does not add a token
cache or independent refresh policy. This deliberately avoids depending on the
pinned helper's `expires_in` cache interpretation without expanding the phase
into a different token-management design.

### 4.3 Inbound Producer Authorization

When OAuth is required, requests to the implemented Events Subscription API
must carry a Bearer token authorized for:

```text
nnwdaf-eventssubscription
```

Phase 1 implements only the service-level scope because transfer resources are
not implemented. It follows the recurring free5GC producer pattern:

1. `NWDAFContext.AuthorizationCheck(...)` bypasses validation when OAuth is
   disabled
2. otherwise it calls `oauth.VerifyOAuth(...)` with the configured
   `nrfCertPem` and the required service name
3. route middleware is attached to the Events Subscription service group and
   aborts the request on failure
4. NWDAF keeps its existing `ProblemDetails` response boundary instead of
   copying the generic Gin error body

The helper verifies the NRF RSA/RS512 signature and service scope. It does not
fully validate every Release 18 access-token claim or distinguish every
standards-defined authorization failure. Phase 1 records that as a known
standards gap rather than replacing the common free5GC acceptance behavior
with an NWDAF-private validator. OAuth-disabled behavior remains unchanged.

### 4.4 Notify And Callback Operations

TS 29.500 states that OAuth access tokens are not used for notify operations in
the current release. The current NWDAF `/collector` routes are notification or
project-local callback paths, not the advertised Events Subscription producer
API.

Therefore Phase 1 keeps these routes outside the Events Subscription
authorization group:

- `/collector/notify`
- `/collector/upf-notify`
- `/collector/retrieval-notify`

This is the resolved Phase 1 boundary. The routes remain unchanged and are not
protected using the unrelated `nnwdaf-eventssubscription` producer scope. Their
future authorization design is not planned here.

---

## 5. Confirmed Phase 0 Starting State

The Phase 0 baseline in `NWDAF` commit `2d34594` already provides:

1. a required normalized `nrfUri`
2. context-owned NF identity, NF profile, registration resource URI, OAuth
   indication, and heartbeat value
3. a generated NFManagement client behind `internal/sbi/consumer`
4. HTTP/2 H2C and HTTPS client transport construction with redirect rejection
5. registration before all owned listeners
6. cancellation-aware registration retry
7. deregistration before SBI shutdown through a bounded cleanup context
8. exact lifecycle, generated-client, H2C, TLS, and live OAuth-disabled NRF
   proof

The intentional Phase 0 OAuth behavior is:

1. inspect `customInfo.oauth2` in every supported successful registration
   response shape
2. return `ErrOAuth2Required` when the value is true
3. leave local listeners unstarted
4. record that the remote profile may require NRF-side cleanup

Phase 1 replaces only this terminal OAuth branch. It must preserve the rest of
the Phase 0 registration and lifecycle behavior.

The current NWDAF does not yet provide:

- `nrfCertPem` configuration or parsed NRF verification key ownership
- a generated Access Token client
- an outbound token context for deregistration
- inbound token authorization middleware
- standards-aligned OAuth error responses
- OAuth-enabled integration proof

---

## 6. Confirmed free5GC And Dependency Compatibility Notes

### 6.1 Outbound Helper And NWDAF Lifecycle Context

`github.com/free5gc/openapi v1.2.3/oauth.GetTokenCtx(...)` is the normal
free5GC helper and is sufficient for common NF calls. Its implementation uses
`context.Background()` for the Access Token request and returned authorization
context. That conflicts with one already approved NWDAF behavior: Phase 0
deregistration must finish inside its bounded five-second cleanup context.

Phase 1 therefore preserves the free5GC `GetTokenCtx(...)` boundary and the
generated Access Token client, but implements the request inside the NWDAF NRF
consumer with the caller's context. No private OAuth protocol, alternate token
format, or custom cache is introduced. The helper's cache and `expires_in`
behavior are not copied because Phase 1 only needs a fresh token immediately
before protected deregistration.

### 6.2 Inbound Helper Has A Known Standards Gap

The pinned `oauth.VerifyOAuth(...)` helper verifies the RSA `RS512` signature
and checks the requested service scope. This is the recurring implementation
used by the surveyed UDR, UDM, PCF, and SMF producer routes.

The helper does not fully enforce Release 18 expiry, issuer, subject, and
audience semantics, and it does not classify all invalid-token versus
insufficient-scope outcomes. Phase 1 intentionally accepts that free5GC
compatibility limitation. Tests and completion claims must cover what the
helper actually guarantees, and the documentation must not claim complete TS
29.500/29.510 access-token validation.

### 6.3 Current NRF Audience Behavior

The surveyed NRF v1.4.5 populates `aud` from `targetNfInstanceId`, while the
common free5GC NF token request supplies target NF type and leaves that
instance field empty. Phase 1 does not add stricter audience validation than
other free5GC NFs and therefore does not need a special empty-audience
exception in NWDAF code. This remains part of the documented standards gap in
Section 6.2.

### 6.4 Current NRF Certificate Behavior

When OAuth is enabled, NRF v1.4.5 generates an instance-specific NF
certificate during first registration. It obtains or creates an NF-type public
key under the NRF certificate directory, signs a certificate containing the
NF type and registered NF instance ID, and later reads that certificate while
authorizing an Access Token request.

This is deployment-side free5GC behavior. It does not deliver a private key or
certificate to NWDAF. `nrfCertPem` on NWDAF is instead the NRF public
certificate used to verify JWT signatures.

### 6.5 OAuth And HTTPS/mTLS Are Separate

The current free5GC sample enables NRF OAuth while using an HTTP SBI. In that
mode, the access token and NFManagement procedures run over H2C and
`nrfCertPem` validates signed tokens.

NRF v1.4.5 additionally requires and verifies a client certificate when both
OAuth and the NRF SBI `https` scheme are enabled. Supporting that deployment
requires a matching NWDAF private key, client certificate, trust chain, and a
stable relationship to the registered NF instance ID. The current NWDAF has a
random process-lifetime NF ID and no NRF OAuth client-certificate
configuration. Reusing the SBI server certificate implicitly would conflate
roles and may not satisfy the NRF identity checks.

Phase 1 targets OAuth token authorization using the common free5GC HTTP/H2C
deployment. Existing Phase 0 HTTPS server verification remains supported when
the server chain is trusted, but OAuth-enabled HTTPS with NRF-required mutual
TLS client authentication is deferred because it is not a uniformly wired
cross-NF convention in the surveyed implementations.

---

## 7. Target Phase 1 Design

### 7.1 Configuration

Add `configuration.nrfCertPem` following the recurring free5GC NF naming:

```yaml
configuration:
  nrfUri: http://127.0.0.10:8000
  nrfCertPem: cert/nrf.pem
```

Rules:

1. normalize whitespace and expose an explicit getter
2. keep the field optional, matching the surveyed NF config structs
3. copy the path into the runtime context during normal context initialization
4. when registration reports OAuth required and the path is empty or unusable,
   emit a strong configuration/security log but do not force process shutdown
5. let subsequent protected inbound authorization fail through the same helper
   boundary as other free5GC NFs when verification material is unavailable
6. do not treat `nrfCertPem` as the SBI server certificate or private key

Development certificate and key assets belong in `nwdaf-resources`, following
the workspace repository boundary. NWDAF sample configuration uses the normal
free5GC runtime-relative path `cert/nrf.pem`; deployment tooling must arrange
the corresponding runtime `cert/` directory. Development keys may be tracked
in `nwdaf-resources` in the same manner as the free5GC main development assets,
but production certificate and private-key replacement remains deployment
owned. Tests should use temporary material rather than depend on those assets.

### 7.2 Context-Owned OAuth State

Extend `NWDAFContext` without introducing package-global security state.

The context should own or expose:

1. the resolved OAuth-required flag from registration
2. the configured NRF certificate path
3. the existing NF instance ID, NRF URI, and registered resource state needed
   for token requests

The registration state must represent remote truth. An OAuth-required
registration is a successful remote registration even when `nrfCertPem` is
missing, so it must not be converted into a local startup failure.

Context methods must remain safe under the existing registration-state mutex.
Sensitive token data and private keys must not be placed in the registration
snapshot or logs.

### 7.3 Consumer-Owned Access Token Path

Extend the existing NRF consumer service instead of creating a separate root
client owner.

The target shape is:

1. cache a generated `AccessToken.APIClient` by normalized NRF base URI using
   the existing consumer mutex discipline
2. use the same Phase 0 HTTP/2/TLS transport construction and redirect policy
3. build the generated Access Token request with the values in Section 4.2
4. call it with the caller's context
5. require a usable access token response
6. return an authorization context derived from the caller context for the
   immediately following protected operation

Phase 1 needs a token for deregistration only. It requests a fresh token for
each protected Phase 1 operation and defers cache/refresh behavior until a
repeated caller such as discovery actually requires it. This is the only
intentional call-level difference from the common helper, and exists solely to
preserve the caller's bounded context.

Access Token request redirects follow the Phase 0 policy: `307` and `308` are
clear terminal unsupported-deployment errors until a separate multi-NRF
redirect design exists. Transport and NRF 5xx behavior must be classified
explicitly; shutdown deregistration remains bounded by its existing cleanup
context and must not enter an independent infinite retry loop.

### 7.4 Registration And Startup Transition

Revise the Phase 0 OAuth branch as follows:

1. register without an access token, as required for bootstrap and as routed
   by the current NRF
2. validate the successful registration response exactly as Phase 0 does
3. record the returned resource URI, heartbeat value, and OAuth indication
4. mark the remote registration active even when OAuth is required
5. if OAuth is required and `nrfCertPem` is empty or unusable, emit a strong
   log that inbound OAuth verification will fail
6. continue with Phase 0 listener startup ordering
7. start workers only after registration and all listeners succeed

OAuth-disabled registration must not request a token or require an NRF
certificate.

The old `ErrOAuth2Required` terminal behavior and its test are replaced by
successful OAuth state transition tests. Missing certificate material is a
logged degraded security configuration, matching other free5GC NFs; it is not
a registration rollback condition. Other malformed response, redirect, retry,
cancellation, and listener rollback semantics remain unchanged.

### 7.5 Protected Deregistration

`DeregisterNFInstance(...)` should:

1. inspect context-owned OAuth state
2. use the caller context directly when OAuth is disabled
3. obtain a token context for target `NRF` and scope `nnrf-nfm` when OAuth is
   enabled
4. call the existing generated NFManagement deregistration operation with that
   context
5. preserve Phase 0 handling for `204`, `404`, redirect, cancellation, and
   terminal errors

Token acquisition failure is a deregistration failure, not permission to send
the same protected request without a token. Shutdown still proceeds after the
bounded cleanup attempt and stops owned listeners.

### 7.6 Inbound Authorization Check

Follow the existing free5GC context method directly:

1. add `AuthorizationCheck(token, serviceName)` to the NWDAF runtime context
2. return success without parsing when registration reports OAuth disabled
3. otherwise call `oauth.VerifyOAuth(token, string(serviceName), nrfCertPem)`
4. rely on the helper's RSA/RS512 signature and service-scope checks
5. do not add an NWDAF-private claims type or additional expiry, issuer,
   subject, or audience policy in Phase 1
6. do not log the token or raw claims, even though some surveyed NF context
   methods currently log the token at debug level

This retains the common free5GC authorization semantics while avoiding an
unnecessary local security framework. The narrower verification guarantee is
documented, not hidden.

### 7.7 Router Authorization Middleware

Follow the free5GC route-group convention while preserving NWDAF's existing
error helper:

1. define one reusable authorization check for
   `models.ServiceName_NNWDAF_EVENTSSUBSCRIPTION`
2. attach it only to the `factory.NwdafEventsSubResUriPrefix` group
3. short-circuit with no token parsing when context says OAuth is disabled
4. abort the Gin request after an authorization failure
5. follow the surveyed free5GC single-status behavior by returning `401` for
   any `VerifyOAuth(...)` failure, while using NWDAF's existing
   `ProblemDetails` response body
6. keep `/collector` outside that group

The middleware should depend on a narrow authorization interface so focused
tests do not require constructing the full application. It should remain under
`internal` and must not create a new general-purpose exported security
framework.

### 7.8 Error And Log Hygiene

Phase 1 logs may include:

- whether OAuth mode is enabled after registration
- the NF instance ID at lifecycle boundaries
- requested service scope
- classified failure type without token content
- cleanup success or failure

Phase 1 logs must not include:

- Authorization header values
- bearer tokens
- complete JWT claims
- certificate contents
- private keys
- complete NF profiles merely to diagnose OAuth

The surveyed UDR/UDM/PCF context helpers log complete tokens at debug level.
Avoiding that leak is a local hygiene correction and does not change token
acceptance semantics.

---

## 8. Implementation Work Packages

### Package A — Configuration And Context

1. add `nrfCertPem` config shape, normalization, getter, and focused tests
2. copy it into context using the same optional-config pattern as other NFs
3. add a strong log for OAuth-required operation without usable certificate
4. revise registration-state methods so OAuth-required registration remains
   represented as remotely registered
5. add context tests for OAuth-disabled, OAuth-enabled, and deregistered state

### Package B — Generated Access Token Consumer

1. add cached generated Access Token client ownership to `NrfService`
2. share Phase 0 HTTP client construction
3. construct the exact generated Access Token request
4. preserve caller context
5. require a usable token response
6. produce a context suitable for a generated protected NFManagement call
7. add generated-client/H2C consumer tests

### Package C — Protected Lifecycle

1. replace the Phase 0 OAuth terminal branch with a successful state transition
2. make deregistration acquire `nnrf-nfm` authorization only when required
3. prove missing certificate produces a strong log without closing listeners
4. preserve listener startup, partial-startup cleanup, cancellation, and
   bounded shutdown behavior
5. keep OAuth-disabled behavior regression-tested

### Package D — Producer Authorization

1. add the free5GC-style context `AuthorizationCheck(...)` using
   `oauth.VerifyOAuth(...)`
2. keep NWDAF's existing `ProblemDetails` error boundary
3. attach authorization only to the Events Subscription group
4. add focused context-helper, middleware, and route-level tests
5. prove collector routes remain outside the producer scope middleware

### Package E — Runtime Proof And Documentation

1. run an OAuth-disabled NRF regression test
2. run an OAuth-enabled NRF integration test under the approved transport scope
3. obtain a token for `nnwdaf-eventssubscription` and test one valid producer
   request
4. test valid, missing, incorrectly signed, and wrong-scope producer requests
5. prove token-protected graceful deregistration removes the remote profile
6. add the agreed development certificate assets/workflow to
   `nwdaf-resources` and prepare the runtime-relative `cert/` layout
7. update README, sample config comments, parent plan, and remediation status

Implementation commits should keep these packages separately reviewable where
practical. A single package must not mix OAuth work with discovery, metrics
server completion, heartbeat, or unrelated authorization cleanup.

---

## 9. Detailed Test Plan

### 9.1 Factory Tests

Cover:

1. omitted `nrfCertPem` with otherwise valid OAuth-disabled configuration
2. normalized non-empty certificate path
3. optional field remains accepted when omitted
4. no regression to HTTP/HTTPS SBI config validation

### 9.2 Context Tests

Cover:

1. OAuth-disabled registered state
2. OAuth-enabled registered state
3. certificate path initialization
4. concurrent registration-state snapshot safety
5. state after successful deregistration
6. state retained after failed deregistration

### 9.3 Access Token Consumer Tests

Use generated-client interception with `gock`/H2C where applicable. Cover:

1. exact form values for `client_credentials`, NWDAF identity, NRF target, and
   `nnrf-nfm` scope
2. caller cancellation before and during the request
3. valid access-token response
4. empty token
5. AccessToken error response without leaking payload secrets
6. transport failure
7. NRF 5xx
8. unsupported `307`/`308`
9. generated client uses the Phase 0 H2C/HTTPS transport constructor

### 9.4 Authorization Check Tests

Follow the UDR/PCF/SMF router authorization test shape and exercise the actual
`oauth.VerifyOAuth(...)` boundary with temporary RSA material. Cover:

1. OAuth-disabled bypass
2. valid RSA/RS512 token with the required scope
3. malformed or incorrectly signed token
4. missing scope
5. wrong scope
6. multiple scopes including the required service scope
7. missing or unreadable `nrfCertPem` returns authorization failure rather
   than crashing the process

Expiry, issuer, subject, and audience rejection tests are not Phase 1
requirements because the selected free5GC helper does not guarantee them.

### 9.5 Middleware And Route Tests

Cover:

1. OAuth-disabled request without a token reaches the producer handler
2. OAuth-enabled valid token reaches the producer handler
3. missing token returns `401` with NWDAF `ProblemDetails`
4. invalid token returns the same `401` authorization error shape
5. wrong scope is rejected through the same helper error boundary and `401`
6. failed authorization aborts before the producer handler
7. each implemented Events Subscription method is protected
8. collector callback routes are not passed through the Events Subscription
   authorization check

### 9.6 Lifecycle Tests

Cover:

1. OAuth-disabled registration still starts listeners without a token request
2. OAuth-enabled registration records a registered state and starts listeners
3. missing certificate logs strongly and does not convert registration into a
   startup failure
4. unreadable or invalid configured certificate logs strongly and does not
   convert registration into a startup failure
5. post-registration listener failure uses token-protected deregistration
6. normal termination uses token-protected deregistration before SBI shutdown
7. cancellation after registration still performs bounded cleanup

### 9.7 Local Integration Gate

OAuth-disabled gate:

1. start the current local NRF with OAuth disabled
2. confirm registration, producer reachability, graceful deregistration, and
   profile removal remain unchanged from Phase 0

OAuth-enabled gate:

1. start the current local NRF with OAuth enabled under the approved transport
   scope
2. provision the NRF signing certificate outside the NWDAF repository
3. confirm registration returns `customInfo.oauth2=true`
4. confirm NWDAF remains running and listeners start
5. obtain a valid token scoped to `nnwdaf-eventssubscription`
6. confirm a protected producer request succeeds
7. confirm missing, incorrectly signed, and wrong-scope requests fail through
   the selected free5GC authorization path
8. terminate NWDAF and confirm token-protected DELETE succeeds before SBI
   shutdown
9. confirm the registered resource is no longer present

A mock-only access token test is not sufficient to complete Phase 1. The live
gate must record the free5GC NRF mode, transport scheme, certificate source,
and the known helper-validation limitations.

---

## 10. Expected Implementation Surface

Likely `NWDAF` changes:

- `pkg/factory/config.go`
- `pkg/factory/config_test.go`
- `config/nwdafcfg.yaml`
- `internal/context/context.go`
- `internal/context/nrf_profile_test.go`
- `internal/sbi/consumer/nrf_service.go`
- `internal/sbi/consumer/nrf_service_test.go`
- `internal/sbi/server.go`
- a narrow authorization implementation and tests under `internal/sbi` or
  `internal/util`
- `pkg/service/init.go`
- `pkg/service/init_test.go`
- `README.md`

The final location of the authorization helper should follow the narrowest
existing NWDAF boundary. It must not be exported through `pkg` merely for tests
and must not force the root app to expose generated clients.

Likely `nwdaf-docs` changes at completion:

- this detailed plan status and implementation record
- the Priority 11 parent plan
- project scan findings/remediation status if their wording still says OAuth
  is absent

Likely `nwdaf-resources` changes at completion:

- a development-only certificate asset directory or preparation workflow
- the NRF public certificate required by `nrfCertPem`
- the NWDAF development certificate/key pair already referenced by the SBI TLS
  config, plus a development root certificate when the chosen workflow needs it
- instructions for preparing or mounting those assets as runtime `cert/...`
  paths without coupling NWDAF config to a sibling-repository path

---

## 11. Verification Commands

Run focused tests after each package, adjusted to the final file split:

```bash
go test ./pkg/factory ./internal/context ./internal/sbi/consumer ./internal/sbi ./pkg/service
```

Run race coverage for the stateful consumer/context/lifecycle packages:

```bash
go test -race ./internal/context ./internal/sbi/consumer ./internal/sbi ./pkg/service
```

Before commit, run:

```bash
go test ./...
make build
make lint
```

Also run `git diff --check` in `NWDAF/` and `nwdaf-docs/`. Script, test,
service, and integration execution must follow the workspace elevation rule.

---

## 12. Explicit Non-Goals

Phase 1 does not include:

1. NRF NFDiscovery or replacement of fixed SMF/ADRF/MTLF endpoints
2. the metrics server or completion of existing raw-client metric coverage
3. NFUpdate heartbeat
4. SCP routing, delegated discovery, or indirect-communication headers
5. CCA or source-NF client credentials assertion flows
6. inter-PLMN, SNPN, roaming, or SEPP behavior
7. Events Subscription transfer resources or transfer-specific scope
8. OAuth protection for current notify/callback routes
9. authorization changes to project-local AnLF and MTLF servers
10. production certificate/private-key generation or distribution
11. a broad generated OpenAPI regeneration
12. a permanent token cache
13. OAuth-enabled HTTPS mutual TLS
14. an NWDAF-private strict JWT claims validator

These exclusions must not be described as full SBI security conformance.

---

## 13. Phase 1 Completion Criteria

Phase 1 is complete only when:

1. OAuth-disabled Phase 0 registration and deregistration remain green
2. OAuth-required registration no longer terminates merely because OAuth is
   required
3. missing NRF verification material produces a strong log without forcing
   startup failure
4. protected deregistration obtains and sends an `nnrf-nfm` token
5. Events Subscription routes enforce the service-level scope when OAuth is
   enabled
6. missing, invalid, and wrong-scope tokens are rejected through
   `oauth.VerifyOAuth(...)` and the NWDAF `ProblemDetails` response boundary
7. helper-backed signature and scope behavior is covered by tests, while the
   unverified expiry/issuer/subject/audience gap remains documented
8. callback routes retain their separately justified treatment
9. token values and certificate contents never appear in logs or errors
10. caller cancellation and bounded shutdown are preserved
11. live OAuth-disabled and OAuth-enabled NRF gates pass under the documented
    HTTP/H2C transport mode
12. focused tests, full tests, race tests, build, lint, and diff checks pass
13. README, sample configuration, and parent/remediation status match the
    landed behavior
14. development assets are placed in `nwdaf-resources`, runtime config uses
    conventional `cert/...` paths, and production replacement remains explicit
15. the free5GC claim-validation and OAuth+HTTPS/mTLS gaps are not represented
    as full standards closure

---

## 14. Resolved Implementation Decisions

All Phase 1 implementation-direction decisions are resolved:

| ID | Decision | Consequence |
| --- | --- | --- |
| D1 | Preserve the free5GC `GetTokenCtx(...)`/consumer boundary, but issue the generated Access Token request with the caller context and no Phase 1 token cache. | The existing five-second cleanup deadline remains effective without inventing a different OAuth protocol. |
| D2 | Use `oauth.VerifyOAuth(...)` acceptance semantics and the free5GC route-group authorization shape; retain NWDAF `ProblemDetails` and secret-safe logging. | Signature and service scope are enforced; expiry, issuer, subject, audience, and fine-grained error classification remain documented gaps. |
| D3 | Treat missing or unusable `nrfCertPem` like the surveyed NFs: emit a strong log and continue startup. | Registration remains active; OAuth-protected inbound calls fail until deployment supplies the certificate. |
| D4 | Validate Phase 1 against the common OAuth-enabled HTTP/H2C NRF deployment. | OAuth+HTTPS NRF mutual TLS is deferred and existing SBI HTTPS support is not removed. |
| D5 | Store development certificate/key assets and preparation workflow in `nwdaf-resources`; use runtime-relative `cert/...` paths in NWDAF config. | Repository roles stay separate while local setup follows free5GC configuration conventions. |
| D6 | Leave current `/collector` callback routes unchanged and do not plan their future authorization mechanism in this phase. | Only the standardized Events Subscription producer group receives the Phase 1 middleware. |

Other fixed values are:

1. registration remains the unauthenticated OAuth bootstrap
2. protected deregistration targets `NRF` with scope `nnrf-nfm`
3. the implemented producer group uses scope
   `nnwdaf-eventssubscription`
4. Events Subscription transfer scope remains out of scope
5. OAuth-disabled mode requires neither a certificate nor a token
6. bearer tokens, certificate contents, and private keys are never logged

No further user decision is required before implementation unless source or
runtime evidence contradicts one of these assumptions or expands the needed
architecture, scope, or verification environment.

---

## 15. Implementation And Verification Record

Implemented and committed on 2026-07-15 across `NWDAF` commits `3c545d0` and
`74b608b` and `nwdaf-resources` commit `d70c4e1`:

1. `configuration.nrfCertPem` normalization, getter, sample configuration, and
   runtime-context ownership
2. OAuth-required registration as a successful remotely registered state,
   including strong missing/unusable certificate logging without startup
   termination
3. consumer-owned generated Access Token client caching using the same NRF
   HTTP/2/TLS transport and metrics-hook construction as NFManagement
4. caller-context-preserving `client_credentials` token acquisition for target
   `NRF` and scope `nnrf-nfm`
5. token-protected deregistration without an unauthenticated fallback
6. context-owned `AuthorizationCheck(...)` delegating to the pinned
   `oauth.VerifyOAuth(...)`
7. Events Subscription route-group authorization for
   `nnwdaf-eventssubscription`, returning `401 application/problem+json` on all
   helper failures
8. unchanged `/collector` callback routing outside that producer scope
9. development NRF public-certificate material and an isolated runtime
   certificate preparation workflow under `nwdaf-resources/cert/`, including
   validation that an existing NWDAF certificate and private key are usable and
   contain a matching public key
10. README and runtime configuration documentation distinguishing NRF token
    verification from NWDAF SBI TLS identity

Repository verification passed:

```text
go test -count=1 ./pkg/factory ./internal/context ./internal/sbi/consumer ./internal/sbi ./pkg/service
go test -race -count=1 ./internal/context ./internal/sbi/consumer ./internal/sbi ./pkg/service
go test -count=1 ./...
make build
make lint
git diff --check
```

The development-certificate preparation script was executed with output under
`/tmp`; the resulting NRF verification certificate and NWDAF SBI certificate
were parsed successfully, and the NWDAF certificate contained the expected
loopback IP and `nwdaf` DNS subject alternative names. The workflow also passed
fresh-generation and valid-pair reuse checks and rejected a deliberately
mismatched private key with a nonzero exit status.

Live free5GC NRF v1.4.5 verification used HTTP/H2C and the local MongoDB:

1. OAuth enabled:
   - registration returned `customInfo.oauth2=true` and NWDAF started all owned
     listeners
   - NRF issued an RS512 token with scope
     `nnwdaf-eventssubscription`
   - a valid token passed authorization and reached the producer handler
   - missing, malformed, and valid-but-wrong-scope tokens returned
     `401 application/problem+json`
   - `/collector/notify` remained outside the producer middleware
   - graceful shutdown obtained a fresh `nnrf-nfm` token, completed DELETE,
     stopped SBI afterward, and the NRF profile lookup returned `404`
2. OAuth disabled:
   - registration reported `oauth2Required=false`
   - a producer request without a bearer token reached the handler
   - graceful deregistration completed without a token request and the NRF
     profile lookup returned `404`

The valid-token live request deliberately targeted a nonexistent subscription;
the resulting `SUBSCRIPTION_NOT_FOUND` response proves that authorization
passed and the producer handler ran without requiring unrelated AnLF/SMF
integration to be added to this security gate.

The focused automated-test completion in `NWDAF` commit `74b608b` closed the
remaining Section 9 matrix with:

1. an Access Token transport-failure case
2. an Access Token request whose caller context is canceled before dispatch
3. an invalid-PEM-content case in addition to missing/unreadable material
4. an OAuth-enabled listener-startup rollback that exercises token-protected
   deregistration in the same lifecycle test
5. a route-wiring test that constructs the production `NewServer(...)` path
   instead of reproducing the route groups in a test-only router

The new cases were repeated ten times without failure, and the final focused,
race, full-module, build, lint, and diff gates passed after review cleanup. The
production `NewServer(...)` route test retains the existing `MockApp` seam
instead of shadowing its cancel-context behavior. Together with the already
completed OAuth-disabled and OAuth-enabled live NRF gates, the Section 13
completion criteria are satisfied and Phase 1 is complete.

Phase 1 does not close the documented expiry, issuer, subject, audience,
OAuth+HTTPS/mTLS, discovery, metrics-server, or heartbeat gaps. Those remain
exactly within the boundaries stated in Sections 6 and 12.
