# Phase 5 Code Review Findings And Remediation Plan

## 1. Purpose

This is the single consolidated review ledger for the Phase 5 dataset-source,
ADRF retrieval, MongoDB retrieval, and generic NRF-discovery implementation.
It records defects found after the implementation slices were integrated and
defines one remediation pass. It is not a new implementation plan.

The review preserves the decisions in
`Phase 5 Dataset Selection And Direct Retrieval.md`:

- PyAnLF owns ADRF-first storage and MongoDB fallback.
- PyMTLF snapshots one selected training data source for each dataset job.
- Go handles standardized NRF and ADRF control-plane communication but does
  not fetch dataset bytes.
- Source fallback does not merge historical MongoDB and ADRF periods.
- Multiple matching ADRFs use a deliberately simple deterministic first
  candidate; priority, capacity, load, locality, and failover remain deferred.

## 2. Review Scope And Evidence

Reviewed repositories:

- `NWDAF`: generic NRF proxy/cache, ADRF storage and retrieval proxy,
  callback forwarding, backend sync, and lifecycle integration.
- `PyAnLF`: ADRF resolution, ADRF-first ingestion, MongoDB fallback, and
  `trainingDataSource` projection.
- `PyMTLF`: ADRF resolution, dataset job coordination, direct fetch,
  MongoDB query, completeness, cleanup, and shutdown.
- `nwdaf-resources`: process-level dataset retrieval scenario.
- `nwdaf-docs`: Phase 5 normative decisions and acceptance criteria.

Normative behavior remains derived from the Release 18 material cited by the
Phase 5 plan. This review focuses on whether the implementation follows that
recorded behavior; it does not introduce a new standard API.

## 3. Findings

### P5-R01: ADRF candidate ordering was not fully deterministic

**Status:** fixed before this remediation pass.

Both backends sorted candidates using the API root too early. An NRF response
containing multiple services could therefore select a different service than
the documented identity-first order. `nfServiceList` entries also needed their
map key as the service identity when the inline service ID is absent.

**Required result:** filter registered matching services, sort by
`(nfInstanceId, serviceInstanceId, normalizedApiRoot)`, deduplicate API roots,
and select the first result.

### P5-R02: PyAnLF could announce MongoDB before the fallback write succeeded

**Severity:** high.

After an ADRF failure, PyAnLF queued the record for MongoDB and immediately
changed `trainingDataSource` based on the repository's previous availability.
An asynchronous insert could still fail. PyMTLF could then start a MongoDB job
for a period in which PyAnLF had not successfully stored the triggering
record.

**Remediation:** MongoDB sender success/failure callbacks own the source
transition. ADRF failure first marks the source unavailable and queues the
same record. Only a completed MongoDB insert changes the source to `mongodb`.
A delayed MongoDB result must not overwrite a newer ADRF recovery.

### P5-R03: PyAnLF ADRF recovery used fixed or immediate probing instead of capped backoff

**Severity:** medium.

The configured initial and maximum ADRF retry backoffs were not both applied.
In particular, `unavailable` could cause an ADRF attempt for every new
notification, bypassing the intended recovery interval.

**Remediation:** maintain a consecutive ADRF failure count, compute capped
exponential recovery deadlines, reset them on a real successful ADRF write,
and send intervening records only to the currently usable MongoDB fallback.

### P5-R04: PyAnLF accepted ADRF roots that the other components reject

**Severity:** medium.

PyAnLF reused SMF API-root normalization for ADRF. That normalization permits
a path and did not consistently reject invalid ports, while PyMTLF and Go
require an HTTP(S) origin. A configured or discovered ADRF could therefore be
accepted locally but rejected on every proxied write.

**Remediation:** use an ADRF-specific origin normalizer matching the Go and
PyMTLF contract: HTTP(S), host present, valid port, and no userinfo, path,
query, or fragment.

### P5-R05: One malformed MongoDB document aborted the whole dataset job

**Severity:** high.

The MongoDB loop validated documents without a per-document boundary. One
malformed historical record therefore failed a job even when valid records
covered every required scope. The plan explicitly requires malformed
documents to be skipped and counted.

**Remediation:** validate each document independently, count and log skipped
documents, continue scanning, and let the existing per-scope completeness
check decide whether the remaining dataset can become READY.

### P5-R06: Selected-source network/query retries were incomplete

**Severity:** high.

ADRF fetch retried selected HTTP statuses but not transport errors. MongoDB
performed only one connection/query attempt. These behaviors contradict the
bounded retry contract in the Phase 5 plan.

**Remediation:** apply the dataset retry settings to ADRF transport failures
and to MongoDB connection/query failures. Each MongoDB retry builds a fresh
temporary result so a partially consumed cursor cannot duplicate records in
the job.

### P5-R07: ADRF callbacks and records were not bounded by the job safety limit

**Severity:** high.

`maxRecordsPerJob` was applied only while reading MongoDB. ADRF callbacks
could accumulate unlimited fetch IDs, an unused list retained complete raw
callback bodies, and fetched records had no common upper bound.

**Remediation:** remove raw callback retention, track only a callback count,
bound accepted fetch IDs and records using the configured job limit, reject
overflow, and apply the same record limit to both storage adapters.

### P5-R08: ADRF callback validation errors could become a false 503

**Severity:** high.

PyMTLF converted model-monitor/provision validation failures to standard
400 `ProblemDetails`, but excluded the ADRF notification path. A malformed
ADRF callback could produce PyMTLF's private 422 response; Go could not map
that private status and returned 503 to ADRF.

Go's callback content-type check also accepted prefixes such as
`application/jsonx`.

**Remediation:** make ADRF callback validation use standard 400
`INVALID_MSG_FORMAT`, and parse the media type exactly in Go before forwarding.

### P5-R09: Go created a new ADRF transport for every proxied request

**Severity:** medium.

The plan requires a target-keyed shared transport client. The implementation
constructed a new `http.Transport` for every storage, subscribe, and
unsubscribe request, defeating connection reuse and retaining idle transports
until garbage collection under sustained notifications.

**Remediation:** make `Consumer` own and reuse one ADRF client per normalized
target API root. Preserve the legacy configured client only for legacy callers.

### P5-R10: ADRF cleanup and shutdown were not convergent

**Severity:** high.

PyMTLF attempted DELETE once, left the peer subscription ID populated after
terminal cleanup, and called cleanup again during shutdown. A worker waiting
for an ADRF callback did not reliably stop when shutdown began, while the
shared HTTP client could be closed after only a timed join.

**Remediation:** make route cleanup idempotent, retry transient cleanup
failures with a bound, check shutdown while waiting/fetching, and use a fixed
worker pool whose shutdown completes before shared clients close.

### P5-R11: Go trusted an arbitrary ADRF retrieval Location

**Severity:** high.

Go stored the peer-provided `Location` and later issued DELETE to it. An
absolute cross-origin Location could redirect the trusted Go proxy to another
host. A malformed path could also make the local subscription identifier
ambiguous.

**Remediation:** resolve relative Location against the selected ADRF, require
the same origin, require the standard retrieval-subscription resource path
with one non-empty identifier, reject userinfo/query/fragment, and preserve
the validated absolute Location for cleanup.

## 4. Remediation Result

Completed on 2026-07-25:

| Finding | Result | Closing commit |
| --- | --- | --- |
| P5-R01 | Fixed and covered in both backend discovery tests. | PyAnLF `6e87d01`; PyMTLF `613c879` |
| P5-R02 | Fixed; MongoDB source transition now follows the completed insert callback. | PyAnLF `6e87d01` |
| P5-R03 | Fixed; PyAnLF uses capped exponential ADRF recovery deadlines. | PyAnLF `6e87d01` |
| P5-R04 | Fixed; PyAnLF ADRF roots now use origin-only validation. | PyAnLF `6e87d01` |
| P5-R05 | Fixed; malformed MongoDB documents are skipped, counted, and tested. | PyMTLF `613c879` |
| P5-R06 | Fixed; ADRF transport and MongoDB connection/query failures use bounded retries. | PyMTLF `613c879` |
| P5-R07 | Fixed; raw callback retention was removed and fetch IDs/records share the job bound. | PyMTLF `613c879` |
| P5-R08 | Fixed; malformed ADRF callbacks use standard 400 and exact JSON media-type parsing. | NWDAF `3c71957`; PyMTLF `613c879` |
| P5-R09 | Fixed; Go reuses a target-keyed ADRF client and transport. | NWDAF `3c71957` |
| P5-R10 | Fixed; workers are bounded, shutdown waits for them, and cleanup is retried and idempotent. | PyMTLF `613c879` |
| P5-R11 | Fixed; Go accepts only same-origin standard ADRF retrieval resource Locations. | NWDAF `3c71957` |

No additional correctness or safety defect was found in the final remediation
diff inspection.

## 5. Verification After Remediation

The remediation is complete only when all of the following pass:

- focused unit tests for every finding above;
- complete PyAnLF and PyMTLF test suites and Ruff lint;
- formatting checks limited to changed Python files, without rewriting
  unrelated existing files;
- `go test ./...`, Go lint, and Go build;
- the real-process dataset retrieval scenario in `nwdaf-resources`;
- documentation and repository diff checks.

The final review must inspect the resulting diff once against this ledger. It
must not create another speculative review cycle unless that final inspection
finds a new correctness or safety defect.

Verified:

- `NWDAF`: `go test ./...`, `make lint`, and `make build`.
- `PyAnLF`: `uv run ruff check .`; `uv run pytest -q` reported
  `241 passed, 1 skipped`.
- `PyMTLF`: `uv run ruff check .`; all 14 changed Python files passed
  `ruff format --check`; `uv run pytest -q` reported `77 passed` with one
  dependency deprecation warning.
- Focused ADRF, source-cutover, retry, validation, limit, Location, and cleanup
  tests passed in all three implementation repositories.
- `nwdaf-resources/tests/mtlf_dataset_retrieval/run.py` passed with real
  processes, covering NRF registration/R17 compatibility, configured ADRF
  storage/retrieval, MongoDB fallback, and ADRF future-write recovery.
- `git diff --check` passed in every affected repository.

PyAnLF has a pre-existing repository formatter baseline. The new Phase 5
implementation and focused tests are formatted. Four previously touched
integration/test files were not mechanically reformatted because doing so
would create broad unrelated diffs; the complete repository still passes Ruff
lint and tests.
