# NWDAF Development Policy

This document defines development, planning, remediation, and review rules for
the NWDAF workspace.

Use it for implementation in `NWDAF/`, `PyAnLF/`, or `PyMTLF/`, and for
implementation-oriented plans under `nwdaf-docs/docs/plans/`.

Read `AGENTS.md` first for repository boundaries, continuous-work-unit
behavior, and skill routing. Read `free5gc-dev-skill/SKILL.md` only when the
active work meets the free5GC skill scope defined there.

---

## 1. Scope And Implementation Slices

- Make a right-sized change that fully resolves the approved implementation
  slice.
- Do not mix feature work with unrelated refactors.
- Preserve existing behavior unless the approved plan explicitly replaces or
  removes it.
- Treat reference trees as read-only unless the user explicitly requests a
  reference change.
- Keep commits separated by repository.

### 1.1 Complete Means Complete Within The Slice

Prefer complete fixes within the currently approved implementation slice.

“Complete” means closing the behavior and acceptance criteria owned by that
slice. It does not mean automatically implementing:

- an adjacent or future phase
- speculative hardening
- legacy cleanup with no active production caller
- an unconfirmed integration risk
- a broader architecture refactor

When an adjacent issue is found, classify and record it. Do not add it to the
current implementation unless it directly prevents the current slice from
satisfying its acceptance criteria.

### 1.2 Slice Definition

Before a substantial implementation begins, identify:

1. the behavior or vertical flow being changed
2. the repositories and owners involved
3. the external and private contracts involved
4. the acceptance tests for this slice
5. explicitly deferred behavior

A named project phase may contain several implementation slices. Finish,
verify, and prepare a checkpoint for one slice before expanding into another
cross-cutting flow. Do not intentionally accumulate an entire multi-repository
phase as one unreviewable working-tree change.

### 1.3 Existing-Flow Extension Gate

When a new mode, topology, role, version, or procedure extends an existing
production flow, name the canonical existing flow as its baseline before the
plan is implementation-ready. Map the baseline end to end, including trigger,
preparation, execution, validation, publication or handoff, successful
completion, failure, timeout, restart, and cleanup where applicable.

For every baseline stage, the active plan must record one of:

- reused without semantic change
- adapted, with the changed owner, data flow, or invariant identified
- explicitly replaced
- approved for deferral
- not applicable, with evidence and rationale

A baseline stage must not be silently omitted. A deferral is not valid when
the omitted stage is required for a shared lifecycle claim, completion
criterion, or other meaning asserted by the current slice.

When an extension reuses any existing name, value, or representation that
carries behavioral, lifecycle, or contract meaning, it must preserve the
established preconditions, postconditions, transition invariants, and
externally or internally relied-upon interpretation, regardless of where or
how that meaning is encoded. If the meaning differs, use a distinct
representation or treat the change as an explicit semantic contract decision.
This applies to any semantic representation; states, events, artifact roles,
result labels, operation names, status values, schema discriminators, and
lifecycle milestones are non-exhaustive examples.

The acceptance criteria and tests must cover both the intended differences and
the shared semantics retained from the baseline.

### 1.4 Out-of-scope Classification

Use one of these labels for discovered work that does not block the active
slice:

- `future-phase handoff`: owned by a named later phase
- `legacy cleanup`: obsolete or unused code without a current functional
  failure
- `optional hardening`: resilience beyond the required current contract
- `integration verification gap`: behavior not yet exercised in the required
  external environment
- `unconfirmed risk`: plausible concern without deterministic evidence

These labels do not become current blockers merely because they were found
during review.

---

## 2. Architecture And Ownership

- Follow established repository and package boundaries.
- In `NWDAF/`, preserve the existing `context`, `consumer`, `processor`,
  `factory`, `service`, and config-driven structure where applicable.
- Keep shared Go runtime state in `internal/context/` unless the approved
  ownership design says otherwise.
- Prefer extending an existing service flow over creating a parallel command
  path.
- Keep backend-owned Python business state in the backend that owns the
  behavior; do not apply Go NF package structure to Python internals.
- When introducing a new internal contract or state machine, document its
  owner, lifecycle, failure boundary, and restart behavior.

### 2.1 New Go Package Gate

Creating a new Go package or directory is an architecture decision, not a
default response to shared code.

Before adding one:

1. Identify the behavior owner, callers, transport boundary, state/lifecycle
   owner, and intended dependency direction.
2. Inspect the target repository for an existing owner package that can
   reasonably contain the behavior.
3. Inspect the specifically applicable free5GC exemplar and record which
   package shape and boundary it demonstrates.
4. Prefer adding a file to the existing owner package. A shared parser,
   validator, schema name, protocol name, or desire to avoid duplication is
   not by itself sufficient reason for a new package.
5. Create a package only when it has a distinct, stable responsibility that
   cannot be placed in an existing owner without reversing dependencies,
   creating an import cycle, or mixing unrelated lifecycles.
6. Classify by the actual owner and transport boundary, not by payload shape.
   A private Go-backend API that reuses standard field names or
   `ProblemDetails` is still a private backend boundary; it does not become
   external SBI and must not be placed under `internal/sbi` for that reason
   alone.
7. If the proposed package has no clear repository precedent or applicable
   free5GC exemplar, record the proposed ownership in the active plan and
   discuss it before implementation.

Review must enumerate every newly added package and verify its necessity,
dependency direction, and exemplar. Passing tests demonstrate behavior, not
package-placement correctness.

### 2.2 Cross-Boundary Data-Flow Gate

A proposed change that crosses a process, repository, API, callback, storage,
or trust boundary is not implementation-ready until its end-to-end data flow
is established.

The active plan or implementation analysis must identify:

1. the behavior or acceptance criterion that requires the change;
2. the authoritative producer of each required value;
3. how each value crosses every boundary;
4. which component owns and stores it at each stage;
5. whether the receiving component can obtain it independently of the data
   being validated;
6. lifecycle, restart, timeout, and stale-data behavior where applicable;
7. every affected repository and public, standard-shaped, or private
   contract;
8. focused boundary tests and the required end-to-end verification.

A downstream schema field, validation parameter, configuration entry, or
helper function is not a complete design unless its upstream source and
propagation are also defined.

If the required data is unavailable at the consuming boundary, classify the
work as an architecture or contract change. Do not silently assume the data
exists, derive an expected value from the same untrusted value being checked,
or describe the change as implementation-local. Estimate complexity from the
complete production data flow, not only from the size of the final local code
change.

## 3. Standard Boundary Levels

### 3.1 External SBI

External SBI operations must follow the applicable OpenAPI contract for:

- method and path
- request parameters and body
- success status and representation
- required headers such as Location
- operation-specific declared error status
- applicable ProblemDetails behavior

Use generated models where available. If generation is blocked by a missing
external schema, record the exact gap before introducing an isolated
compatibility type.

### 3.2 Standard-shaped Private Boundary

A Go-to-backend private route described as “standard-shaped” preserves only
the standard properties explicitly listed by the active plan, normally:

- HTTP method semantics
- the standard request or response model
- required identifiers and correlations
- success status, required headers, and representation
- operation-specific peer error forwarding

It does not automatically inherit TLS, OAuth, NRF registration, every shared
3GPP transport feature, or additional recovery mechanisms unless the plan
explicitly includes them.

Private paths may differ from standard public paths. Plain HTTP is sufficient
when that is the confirmed internal-security scope.

### 3.3 Python Backends

PyAnLF and PyMTLF are not independent standardized NFs. Use local 3GPP/OpenAPI
material to define their Go-facing payload semantics, but use normal Python
service structure for internal runtime, repository, queue, scheduling,
training, and model logic.

---

## 4. Language And Code Quality

- Use English for code, identifiers, comments, logs, API-facing strings,
  configuration-facing text, and committed implementation documents inside
  implementation repositories.
- Documentation in `nwdaf-docs/` may use English or Traditional Chinese, but
  each document should remain internally consistent.
- Keep implementations explicit and readable; avoid speculative abstractions.
- Add comments only when intent is not apparent from the code.
- Keep config naming aligned with the existing YAML layout and project terms.
- Follow free5GC patterns only where the repository and active boundary
  actually use them.

---

## 5. Change Safety And Remediation

- If supported behavior changes, update or add tests in the same slice.
- Preserve existing characterized behavior unless it is explicitly classified
  as replaced or obsolete.
- Do not silently weaken validation, substitute a fake dependency for a
  required real one, or change architecture because the planned approach is
  inconvenient.

### 5.1 Test-first Remediation

For each confirmed defect:

1. Add or identify a deterministic failing test.
2. Confirm that it fails for the expected reason.
3. Make the smallest production change that closes the failure and its direct
   behavioral dependency.
4. Run the focused regression suite.
5. Perform a targeted follow-up review of the remediation diff and its direct
   dependencies.
6. Continue the remediation and targeted-review loop until the admitted
   finding is closed or a decision gate is reached.

Once an initial review admits a finding whose remediation stays within the
approved plan, architecture, ownership, contracts, and verification scope,
continue directly into test-first remediation without waiting for additional
user confirmation. Progress updates do not replace this continuation and are
not approval gates. Pause only when section 6 requires a decision, required
authority or permissions are missing, or the user changes the objective.

Do not begin a structural remediation from prose alone when the defect can be
captured in a deterministic test. Use Event, barrier, controllable clock, or
fault-injection dependencies for concurrency tests instead of fixed sleep as
the sole proof.

### 5.2 No Scope Expansion During Remediation

A remediation may expand only into code directly required to close the
admitted finding. Adjacent cleanup, future work, or speculative resilience goes
to the appropriate out-of-scope classification.

If closing the finding requires changing ownership, data flow, an external
contract, or the approved verification level, stop and request a decision
before editing that broader boundary.

---

## 6. Decision Gates And Blocking Policy

Do not stop for ordinary implementation-local choices that preserve the
approved architecture, contract, and acceptance criteria.

Stop and request a decision only when completion requires one of the following:

- changing an agreed ownership, architecture, data-flow, or state-flow boundary
- changing an external or explicitly standard-shaped contract
- adding a new external dependency, service, or persistence mechanism
- dropping or weakening an acceptance criterion or required verification
- implementing behavior already assigned to another phase
- choosing between meaningful product behaviors with different outcomes
- proceeding despite missing required specifications, dependencies,
  permissions, tooling, or environment
- replacing the approved implementation strategy because a core assumption is
  false

Discovering optional cleanup, hardening, or future work is not itself a
blocker. Record it and continue the active slice.

### 6.1 Required Blocker Report

When a real decision gate is reached, report:

1. the original plan or assumption
2. the exact blocker or contradiction
3. realistic options
4. the recommended option and tradeoffs
5. whether the plan must change before implementation continues

---

## 7. Finding Admission Gate

A review finding belongs to the current implementation only when conditions
1 through 4 are true:

1. The behavior is confirmed by code evidence, a deterministic reproduction,
   or a direct specification contradiction.
2. It occurs on a path supported by the current implementation slice.
3. It violates an explicit current-slice acceptance criterion.
4. It is not already assigned to a future phase.

If any of conditions 1 through 4 is false, use an out-of-scope classification
from section 1.4 instead of treating it as a current blocker.

After admission, determine whether its remediation fits the approved slice:

- If it fits, fix it through section 5.
- If it requires an ownership, architecture, contract, dependency, or
  verification expansion, use the decision gate in section 6.
- Do not silently downgrade a confirmed current-slice defect merely because
  its correct remediation needs a decision.

### 7.1 Evidence And Severity

- A passing test does not prove the absence of an untested production defect.
- A plausible concern without reproduction is an unconfirmed risk, not a
  confirmed finding.
- A missing real-environment test is an integration verification gap unless
  the current acceptance criteria explicitly require that environment.
- A peer violating a mandatory standard contract should first receive the
  specified peer-error handling. Extra recovery from the peer violation is
  optional hardening unless the active plan explicitly requires it.
- Dead code without a production caller is legacy cleanup unless it creates a
  current build, behavior, ownership, or maintenance contradiction explicitly
  covered by the slice.

Use P0/P1 only for confirmed current-slice failures with critical impact, such
as unsafe state corruption, an unbounded supported lifecycle, loss of required
resource ownership, or a broken primary flow. Do not use severity to pull
future work into the current phase.

---

## 8. Review Closure Protocol

### 8.1 Initial Review

After implementation and its focused verification complete, perform one
initial review as the uninterrupted next step and before the final full
verification or implementation commit checkpoint. Do not wait for a separate
review request.

The initial review must inspect, as applicable:

- the complete slice diff
- the agreed plan and acceptance criteria
- the canonical baseline flow and its recorded stage-by-stage disposition
- direct call paths and lifecycle dependencies
- relevant standard/free5GC evidence
- tests and skipped verification

### 8.1.1 Plan-Conformance Gate

For a substantial implementation slice whose active plan defines review
checklists, acceptance criteria, completion criteria, or required verification
commands, maintain a working conformance map throughout implementation and
review.

Any active-plan statement that commits to production behavior, a deliverable,
an acceptance or completion criterion, or required verification is a
normative plan item unless it is explicitly identified as background context,
a non-goal, optional work, or an approved deferral. Writing a requirement in
narrative prose instead of a checklist does not exempt it from the conformance
map.

For every normative plan item, record one of:

- the production path that implements it
- the deterministic test that verifies it
- the verification command and result that prove it
- an explicitly approved deferral or plan change

The initial review must inspect in all applicable directions:

1. implementation to plan: every production change remains within the approved
   slice
2. plan to implementation: every required plan item has implementation and
   verification evidence
3. baseline to plan: every stage and shared semantic representation from an
   extended production flow has an explicit disposition, and the plan does not
   reuse an established meaning with weaker or different invariants

Passing the existing test suite does not prove that an acceptance criterion is
covered. A required criterion without direct evidence remains open and blocks
a completion claim.

A test satisfies a planned criterion only when it exercises the exact
production owner, entry point, lifecycle state, and outcome named by that
criterion. A test does not directly prove a criterion when:

- the behavior being claimed is replaced or bypassed by a mock
- only an adjacent phase, state, operation, or shared implementation is tested
- the assertion proves only that a call occurred while the required downstream
  effect or fencing behavior is not exercised
- multiple tests are assumed to compose into an end-to-end guarantee without
  explicitly verifying the connecting contract

Mocks remain valid outside the behavior under review. Shared mechanisms may be
reused only when the test explicitly runs the required entry point and state
through that mechanism.

Do not mark a slice complete, prepare its implementation commit, or report it
as complete while any required plan item is missing, unverified, silently
deferred, or satisfied only by inference.

A focused test or command does not satisfy a plan-required full verification
command unless the active plan is explicitly changed or the omission is
reported as an open verification gap.

### 8.2 Follow-up Review

After each in-scope remediation and its focused verification, immediately
perform a targeted follow-up review. Inspect only:

- the remediation diff
- the original finding's direct dependencies
- existing and newly added regression tests
- behavior that the remediation can directly affect

Do not perform another full repository review unless:

- the user explicitly requests a new full review, or
- the remediation supplies concrete evidence of a broader P0/P1 regression.

New issues outside the remediation boundary go to the backlog and do not
reopen the active slice. A passing targeted review closes the finding; do not
require repeated open-ended reviews to prove that no unrelated defect exists
anywhere in the repository.

Continue the remediation and targeted-review loop without pausing while the
remaining work stays within the approved plan. Stop only for a section 6
decision gate or another genuine blocker.

### 8.2.1 Final Fresh-read Conformance Gate

After implementation, review remediation, and targeted verification are
finished, but before the final full verification and completion claim:

1. Re-open and read the current development policy and active plan from disk.
2. Rebuild the final conformance check from the current text, including every
   normative statement, verification-matrix item, review-checklist item,
   completion criterion, required command, and approved deferral.
3. Reconcile each item against the final production diff and exact test path.
4. Treat any missing or indirect evidence as open. Do not claim completion
   until it is implemented, directly verified, or explicitly deferred.

Earlier summaries, remembered requirements, existing conformance maps, and
previously passing test results do not replace this fresh read and
reconciliation.

If production behavior appears implemented but any required direct evidence
remains open, report the phase as implementation-complete but
verification-incomplete. A green focused or full test suite must not override
that classification.

### 8.3 Review Output

- Lead with confirmed current-slice defects.
- Separate blockers, deferred work, legacy cleanup, hardening, verification
  gaps, and unconfirmed risks.
- Explain behavior and consequence in plain language; do not rely only on file
  and line references.
- State exactly what was run and what remains unverified.
- When claiming free5GC alignment, name the exemplar and boundary used.
- State whether the slice is complete, partially complete, or intentionally
  deferred.
- For a substantial slice, include a plan-conformance summary that identifies
  each required criterion as satisfied, open, explicitly deferred, or changed,
  and names the evidence for each completed or open conclusion.
- Never summarize a slice as complete solely from repository-wide test success.
  State separately whether the required behavior is implemented and whether
  the plan-required verification matrix is closed.

### 8.4 Review Documentation

Create or update durable review documentation only when the user requests it
or the active plan requires it. When a durable record is needed, maintain one
review ledger per implementation phase. Each finding should record:

- ID and status
- owner phase
- confirmed evidence
- remediation
- verification
- closing commit

Append a short iteration record instead of creating a new full review document
for every remediation pass. Create a separate document only when architecture,
product scope, or the canonical plan changes. Do not rewrite existing review
history solely to consolidate old files.

---

## 9. Build And Verification

Use repository-native commands and distinguish focused, full, race, build, and
integration verification.

For `NWDAF/`, use Makefile targets by default:

```bash
make build
make test
make lint
make run
```

Minimum before an NWDAF code commit:

```bash
make build
make lint
```

Run `make test` for behavior changes when feasible. Use focused tests during
remediation, then the required full suite before declaring the slice complete.

For Python repositories, use their declared package and lockfile workflow,
normally repository lint plus the full pytest suite before completion.

Documentation-only changes may skip code tests, but run a document diff check
and state that code tests were not run.

Never claim real NRF, SMF, UPF, ADRF, MongoDB, OAuth, TLS, UE, or data-plane
integration from unit or mock tests alone. Record unavailable environment
checks without turning them into confirmed bugs.

---

## 10. Reference Order

When implementation meaning is unclear:

1. target repository code and tests
2. active plan and confirmed decisions
3. local Release 18 OpenAPI YAML
4. relevant local 3GPP TS text
5. specifically applicable free5GC skill guidance
6. free5GC-generated OpenAPI code
7. free5GC exemplar implementation

Use OpenAPI for paths, methods, fields, status codes, headers, and schema. Use
TS text for procedure intent and role boundaries. Use free5GC exemplars for
implementation shape, not to override the target contract.

### 10.1 Evidence Before Technical Conclusions

Use the reference order to gather the minimum sufficient evidence, not merely
the first evidence that supports an expected answer.

- For implementation behavior, trace the relevant production path and its
  direct tests.
- For standardized behavior, distinguish TS procedure text, OpenAPI schema,
  and implementation behavior.
- When field semantics or procedure legality is central to the decision, cite
  the exact clause and the shortest necessary quotation.
- Explicitly label conclusions as specification-defined behavior,
  implementation behavior, inference, or proposed design.
- Check for contradictory evidence or semantic tension before recommending a
  design.
- Keep the final explanation proportional to the user's question; evidence
  depth does not require a long response.

---

## 11. Commit Format

This format applies to implementation commits in `NWDAF/`, `PyAnLF/`, and
`PyMTLF/`. The repositories remain independent and must be committed
separately.

Use:

```text
<type>: <summary>
<type>(<scope>): <summary>
```

Allowed types:

- `feat`
- `fix`
- `docs`
- `refactor`
- `test`
- `chore`

Use imperative mood and a concise, concrete subject. Prefer a meaningful scope
such as `anlf`, `mtlf`, `processor`, `context`, or `consumer`.

The complete implementation commit message, including both subject and body,
must describe the concrete technical delta and its rationale. Do not use
internal project-management or review-tracking labels such as `phase`, `batch`,
`round`, `priority`, review iteration, finding ID, or remediation round anywhere
in an implementation commit message.

Use the body for concise rationale, behavior changes, migration notes, and
follow-up constraints.

Documentation commits in `nwdaf-docs/` may name a phase, plan, review ledger,
or finding when that identifier is itself the documentation being changed.

---

## 12. Common Workflow

### 12.1 Implementation

1. Confirm the active slice, owner, contract, acceptance tests, and deferred
   work.
2. Determine whether the work extends an existing production flow. If it does,
   identify the canonical baseline and complete the stage and semantic-delta
   map required by section 1.3.
3. Convert every normative commitment in the active plan, including narrative
   requirements, checklists, acceptance criteria, completion criteria, and
   required verification commands, into a working conformance map before
   implementation begins.
4. Trace the current behavior and direct dependencies.
5. Read only the evidence required for the active technical boundary.
6. Add characterization or failing tests where practical.
7. Implement the smallest complete slice.
8. Run focused verification.
9. Continue directly into one mandatory initial review without waiting for a
   separate request.
10. Classify findings through the finding admission gate.
11. Fix admitted in-scope findings test-first without pausing when remediation
    preserves the approved plan, architecture, ownership, contracts, and
    verification scope.
12. Run focused verification and perform a targeted follow-up review after
    each remediation; repeat this loop until the findings close or a decision
    gate is reached.
13. Record and defer optional hardening, future-phase work, legacy cleanup,
    integration gaps, and unconfirmed risks without letting them block the
    current slice.
14. Re-read the current development policy and active plan from disk, then
    rebuild the final plan-conformance map from their current text.
15. Reconcile the baseline dispositions and every required plan item against
    the final production diff, exact deterministic test paths, and required
    verification commands. Keep missing or indirect evidence open.
16. Run the required final full verification.
17. Record the final verification results and close the plan-conformance map.
18. Prepare repository-separated commit checkpoints only after the
    plan-conformance map is closed.

### 12.2 Documentation

1. Update the canonical plan or policy first.
2. Keep workspace routing in `AGENTS.md`.
3. Keep stable development and review rules in this document.
4. Keep phase-specific decisions in the phase plan.
5. Keep review iterations in one phase ledger where practical.
6. Link existing evidence instead of restating entire parent documents.
