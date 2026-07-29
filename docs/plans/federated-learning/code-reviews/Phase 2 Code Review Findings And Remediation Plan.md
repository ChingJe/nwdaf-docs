# Phase 2 Code Review Findings And Remediation Plan

Date: 2026-07-29

Status: Closed; remediation and final verification completed on 2026-07-29.

Parent documents:

- `../Distributed NWDAF Federated Learning Implementation Plan.md`
- `../Phase 2 Cross-NWDAF Model Provision And Monitoring Detailed Plan.md`

Affected repositories:

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`
- `nwdaf-resources/`
- `nwdaf-docs/`

## 1. Purpose

This document is the canonical review and remediation ledger for the
cross-NWDAF Model Provision and Model Monitor implementation. It records the
confirmed defects found after the integrated Phase 2 implementation and defines
one bounded remediation pass.

It does not reopen the Phase 2 architecture. The following architecture remains
unchanged:

- A and B are model consumers and Model Monitor providers.
- C is the Model Provision provider and Model Monitor consumer.
- Python backends select the peer NWDAF; Go validates the private selected-target
  metadata and owns the standard peer HTTP operation.
- Go owns local route identity, peer `Location`, callback routing, and backend
  sync projection.
- Standard request and notification bodies do not contain private routing
  metadata.
- Backend restart is recoverable through Go-owned sync; Go restart remains an
  experiment restart.
- Phase 2 does not perform Model Training or FedAvg.

The findings below were reproduced, remediated, and verified. The closing
evidence is recorded in section 15.

## 2. Confirmed Decisions

### 2.1 Resource identity

Go local route identity remains the stable identity exposed to a local backend
and retained across a backend restart. Backend-generated resource IDs remain
private process-local identities.

Every dependent reference must be translated into the same identity namespace
before it enters the Go-owned snapshot. In particular, a monitor subscription
owner must refer to the Go registration route ID, not a stale PyMTLF-generated
registration ID.

### 2.2 Configured model provider

Configured mode describes one provider NWDAF identity with two independently
identified services:

```yaml
model_provider:
  mode: configured
  configured_target:
    nf_instance_id: "<provider NWDAF UUID>"
    provision:
      nf_service_instance_id: "<Model Provision service instance ID>"
      api_root: "http://..."
    monitor:
      nf_service_instance_id: "<Model Monitor service instance ID>"
      api_root: "http://..."
```

The two services may use the same origin, but they must not be represented as
the same NF service instance. NRF mode continues to use capability-filtered
Model Provision discovery followed by exact Model Monitor discovery for the
selected provider NF.

### 2.3 Existing decisions that require no further product choice

- Failed peer-create compensation enters retryable `PENDING_CLEANUP`.
- Public peer callbacks accept only active outbound routes.
- `307 Temporary Redirect` affects only the current request.
- `308 Permanent Redirect` may update the saved peer resource location.
- The distributed E2E uses deterministic data but drives the report from the
  PyAnLF side of the existing production path; it does not add a test-only API
  to NWDAF, PyAnLF, or PyMTLF.

No additional user decision is required for this remediation pass.

## 3. Review Gate

| ID | Priority | Area | Status | Required outcome |
| --- | --- | --- | --- | --- |
| P2-R01 | High | Registration owner identity | Closed | Registration and monitor owner survive sync/restart in one identity namespace without duplicate peer resources |
| P2-R02 | High | Failed compensation | Closed | A created peer resource remains tracked until compensation succeeds |
| P2-R03 | Medium | Public callback ownership | Closed | Only active outbound routes can ingest peer notifications |
| P2-R04 | Medium | Configured service identity | Closed | Configured Provision and Monitor targets carry their real independent service identities |
| P2-R05 | Medium | Distributed E2E coverage | Closed | WAPE travels through PyAnLF, Go-A/B, Go-C, and PyMTLF-C |
| P2-R06 | Low | Redirect persistence | Closed | Temporary redirects do not overwrite the durable peer Location |
| P2-R07 | Medium | Accuracy report scheduling | Closed | A valid WAPE cannot be starved by an earlier insufficient-data liveness notification |

## 4. P2-R01: Registration Owner Identity Can Split During Sync

### 4.1 Current behavior

When C receives a remote Model Monitor registration:

1. PyMTLF creates a backend registration ID, for example `R-python`.
2. C Go creates a separate local route ID, for example `R-go`.
3. PyMTLF may immediately create a monitor subscription and send
   `R-python` in the private owner header.
4. Go stores that owner value unchanged.
5. A subsequent Go sync sends the registration as `R-go` but sends the monitor
   subscription owner as `R-python`.
6. PyMTLF restore cannot associate the subscription with the registration. It
   classifies the subscription as an orphan, deletes it, and creates another
   peer subscription for `R-go`.

Whether this occurs depends on the ordering of the PyMTLF monitor reconciler and
the Go availability refresh. A passing E2E run therefore does not prove the
race is absent.

### 4.2 Root cause

The Go route already records both the Go local registration ID and the current
backend resource ID, but the private monitor-create path does not translate the
owner ID before storing it. The sync projection then combines one registration
identity namespace with a different owner identity namespace.

### 4.3 Remediation

Go must normalize the owner registration ID when accepting a monitor
subscription from PyMTLF:

1. accept the Go registration route ID directly when it already identifies an
   active local registration;
2. otherwise resolve the supplied backend registration ID through the
   registration route's `BackendResourceID`;
3. store only the corresponding Go registration route ID as
   `OwnerRegistrationID`;
4. reject unknown, ambiguous, non-active, or non-MTLF-owned registration IDs;
5. serialize registration and owner using the same Go identity in every sync
   snapshot.

The regression test must force the problematic order: registration create,
monitor subscription create using the backend ID, Go sync, and PyMTLF restart.
The existing peer subscription and correlation must survive without orphan
cleanup or duplicate create.

## 5. P2-R02: Failed Compensation Loses the Peer Resource

### 5.1 Current behavior

Remote Provision, registration, and monitor-subscription create all attempt a
compensation `DELETE` when:

- the peer returns `201` with an invalid representation;
- the peer returns `201` with an invalid contract response that still exposes a
  usable `Location`; or
- the local route cannot be committed after the peer resource was created.

If the compensation request fails, Go only logs the error and returns the
original failure. The peer resource still exists, but no route retains its
absolute `Location`; future cleanup is impossible.

`PENDING_CLEANUP` is defined in the route lifecycle type but is not entered by
the create paths and has no cleanup reconciler.

### 5.2 Remediation

For every remote resource kind:

1. reserve the Go local route ID before peer create;
2. retain the selected target and resolved peer `Location` as soon as they are
   trustworthy;
3. attempt bounded immediate compensation after a later failure;
4. if compensation fails, commit a `PENDING_CLEANUP` route containing only the
   cleanup information;
5. exclude that route from backend active-resource snapshots and callbacks;
6. retry cleanup with bounded backoff;
7. remove the route only after `204` or terminal `404`;
8. expose no successful resource representation to the initiating backend.

Tests must cover compensation success, transport failure, retryable peer error,
terminal `404`, process shutdown, and the absence of a leaked active snapshot.

## 6. P2-R03: Public Callback Does Not Enforce Outbound Active Ownership

### 6.1 Current behavior

The public callback handlers pass the path route ID into the general
notification processors.

For Model Provision, outbound ownership is checked only in the branch where the
body subscription ID differs from the callback path ID. A peer that copies the
callback route ID into the body bypasses that check.

For Model Monitor, the callback path performs correlation and model validation
but does not verify route direction or lifecycle at all.

An inbound route or a non-active cleanup route can therefore reach a backend if
the caller knows its route ID and valid correlation.

### 6.2 Remediation

Public callbacks must use a common route guard before any body rewrite or
backend delivery:

- the route kind matches the callback kind;
- `Direction == OUTBOUND`;
- `SelectedTarget` is present;
- `LifecycleState == ACTIVE`;
- the backend process generation is currently usable;
- Provision peer resource identity, correlation, and Monitor model IDs match.

Unknown routes return `404`; known but non-active routes return `503`; malformed
identity, correlation, or model references return standard `400
ProblemDetails`. None of these failures may enter AnLF model activation or MTLF
accuracy policy.

## 7. P2-R04: Configured Mode Reuses the Provision Service Identity

### 7.1 Current behavior

PyAnLF configured mode currently stores one NF service instance ID and one API
root. Model Provision uses that target correctly. When PyAnLF later creates the
Model Monitor registration, it copies the same candidate and changes only the
service name to `nnwdaf-mlmodelmonitor`.

The resulting private selected-target metadata claims that a Model Provision
service instance is a Model Monitor service instance. It works only while both
services happen to share a usable root and Go does not strictly validate the
configured service identity.

### 7.2 Remediation

Implement the configuration shape in section 2.2:

- validate the provider `nf_instance_id` as UUIDv4;
- validate both service instance IDs as non-empty stable values;
- validate both API roots as HTTP(S) roots with no userinfo, query, or fragment;
- use the Provision entry only for Model Provision;
- use the Monitor entry only for Model Monitor registration;
- preserve the common provider NF instance identity;
- keep private headers out of the standard peer body.

Configuration, resolver, model-demand, sync-restore, and API tests must cover
same-root and different-root configured providers.

## 8. P2-R05: Distributed E2E Bypasses the A/B Report Path

### 8.1 Current behavior

The distributed runner obtains C's monitor route IDs and correlations from
PyMTLF logs, then posts a synthetic WAPE notification directly to C Go's public
callback.

That verifies:

```text
runner -> C Go callback -> PyMTLF-C
```

It does not verify:

```text
deterministic prediction/actual input
  -> PyAnLF-A/B report gate and WAPE calculation
  -> A/B Go private notification ingress
  -> A/B Go outbound standard callback
  -> C Go public callback validation
  -> PyMTLF-C accuracy policy
```

The current runner therefore does not prove the Phase 2 requirements for sample
gating, stable prediction, WAPE construction, A/B notification delivery, or
the complete cross-NWDAF callback chain.

### 8.2 Remediation

Keep a production SMF, UPF, UDM, ADRF, and live traffic outside this Phase 2
fixture, but inject deterministic inputs through an existing PyAnLF production
ingress or runtime path. A support-only fake SMF may establish the normal
collection binding and callback correlation. The runner must then observe the
resulting report at PyMTLF-C.

The runner must verify:

- insufficient samples produce no report;
- sufficient stable samples produce one report per scope;
- A and B correlations remain distinct;
- the report uses standard `deviation` and the expected model ID;
- restart recovery still uses the complete A/B-to-C route;
- deleting A prevents A reports while B remains functional.

No test-only endpoint is added to a production repository. Support-only
orchestration remains in `nwdaf-resources`.

## 9. P2-R06: A Temporary Redirect Becomes the Saved Peer Location

### 9.1 Current behavior

The shared standard transport correctly follows only `307` and `308` with the
original method and body. After a successful remote replace, however, the
processor always overwrites the saved peer Location with the final effective
request URI.

The response does not retain whether the successful redirect chain was
temporary or permanent. Consequently a `307` endpoint is persisted and used by
future replace and delete operations.

### 9.2 Remediation

The transport result must retain enough redirect metadata for route ownership:

- preserve the original request URI;
- record the final effective URI;
- record whether a permanent redirect authorizes a durable route update;
- do not persist a `307` target;
- update the saved route only after an accepted `308` chain;
- continue rejecting userinfo, invalid Location, excessive hops, and HTTPS to
  HTTP downgrade.

Contract tests must cover direct success, one and multiple `307` hops, `308`,
mixed chains, invalid redirects, and a later DELETE using the expected durable
Location.

## 10. P2-R07: Liveness Delivery Can Starve a Valid WAPE

### 10.1 Behavior exposed by the completed E2E

PyAnLF generates a legal no-`deviation` notification when a report window has
insufficient matched predictions. The monitor subscription service previously
counted that liveness notification as the last periodic delivery. If a valid
WAPE became available before the next `repPeriod`, the valid measurement was
dropped as not due.

With deterministic inference and ground-truth input, insufficient and valid
windows could remain phase-aligned. PyMTLF then received continuous liveness
while every valid WAPE was discarded.

### 10.2 Remediation

PyAnLF now retains the latest valid accuracy measurement per monitor resource
when its periodic delivery is not yet due. At the next eligible delivery:

1. the pending valid measurement takes priority over a no-`deviation`
   liveness measurement;
2. the actual notification interval remains bounded by the subscription
   `repPeriod`;
3. a successful delivery removes only the pending measurement that was sent;
4. replace, delete, and sync removal clear the related pending state.

A focused test proves that a liveness delivery, an early valid WAPE, and the
next due liveness tick produce exactly two notifications, with the second
carrying the retained WAPE.

## 11. Remediation Order

Implement the fixes in this order without per-finding commits:

1. normalize registration owner identity and add the forced-order restart test;
2. implement `PENDING_CLEANUP` state and cleanup reconciliation;
3. add the outbound-active callback guard;
4. preserve redirect permanence and correct Location updates;
5. migrate the configured provider schema and resolver behavior;
6. retain valid WAPE measurements until their monitor period is due;
7. extend focused Go, PyAnLF, and PyMTLF tests;
8. change the distributed runner to exercise the complete report path;
9. run all repository canonical verification;
10. inspect only the remediation diff and direct dependencies against this
   ledger;
11. update the ledger and parent plans with the closing evidence.

## 12. Completion Criteria

This ledger can be closed only when:

1. registration and monitor owner IDs remain consistent across immediate sync,
   periodic sync, and PyMTLF restart;
2. no restart or forced ordering creates a duplicate peer monitor subscription;
3. failed compensation retains a retryable cleanup route until terminal
   cleanup;
4. pending cleanup state is absent from backend active snapshots;
5. public callbacks reject inbound and non-active routes;
6. configured mode uses the real Provision and Monitor service identities;
7. `307` does not alter durable route Location and `308` can;
8. deterministic WAPE traverses PyAnLF-A/B, Go-A/B, Go-C, and PyMTLF-C;
9. A and B scope isolation and delete isolation remain valid;
10. an insufficient-data liveness notification cannot starve the next valid
    WAPE;
11. all existing single-NWDAF and distributed tests remain green;
12. Go test, vet, lint, Python test, Ruff, E2E, and diff checks pass;
13. parent plan and current architecture documentation describe only behavior
    proved by the final implementation.

## 13. Baseline Verification

The pre-remediation implementation passed:

- `NWDAF`: `go test ./...`, `go vet ./...`, and
  `golangci-lint v2.11.4 run ./...`;
- `PyAnLF`: 252 tests passed, 1 skipped, and Ruff passed;
- `PyMTLF`: 143 tests passed and Ruff passed;
- `nwdaf-resources`: the isolated A/B/C process runner passed.

These results are retained as the regression baseline. They do not close the
findings because the current assertions omit the failure scenarios above.

## 14. Non-Goals

This remediation does not add:

- Model Training or FedAvg;
- participant selection;
- a production SMF, UPF, UDM, ADRF, or live traffic to the Phase 2 E2E (the
  runner uses only a support fake SMF for collection-resource setup);
- Go restart persistence;
- OAuth, TLS, or callback authentication;
- a new standardized NF or service;
- test-only production APIs;
- priority or capacity-based NWDAF selection.

## 15. Closure Record

All seven findings are closed in the current implementation worktree:

- Go normalizes monitor owners into the registration route namespace, reserves
  peer routes before create, retains failed compensation as
  `PENDING_CLEANUP`, filters inactive routes from sync, guards public callbacks,
  and persists only `308` redirect targets.
- PyAnLF configured mode carries independent Provision and Monitor service
  identities. Valid WAPE measurements that become available between periodic
  delivery times are retained and take priority over liveness at the next due
  delivery.
- The distributed runner uses a support-only fake SMF to establish normal
  collection bindings, injects UPF-shaped observations through PyAnLF's
  production callback, and proves the complete A/B-to-C WAPE path before and
  after backend restart.

Final verification:

| Repository | Verification | Result |
| --- | --- | --- |
| `NWDAF/` | `go test ./...` | pass |
| `NWDAF/` | `go vet ./...` | pass |
| `NWDAF/` | `golangci-lint run ./...` | pass, 0 issues |
| `PyAnLF/` | full `pytest` | 253 passed, 1 skipped |
| `PyAnLF/` | `ruff check src tests run.py` | pass |
| `PyMTLF/` | full `pytest` | 143 passed |
| `PyMTLF/` | `ruff check src tests` | pass |
| `nwdaf-resources/` | distributed deployment Ruff | pass |
| `nwdaf-resources/` | isolated A/B/C multi-process E2E | pass |
| all affected repositories | `git diff --check` | pass |

No new review document is required for this finding set unless the architecture
or implementation scope changes.
