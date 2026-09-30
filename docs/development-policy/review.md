# Review And Conformance

Use this module for code/document review, finding closure, and completion
reconciliation. Review-only work produces findings, not automatic edits.
Authorized code remediation additionally uses [Implementation](implementation.md);
ownership or boundary decisions use [Architecture](architecture.md).
User-review handoff uses [Delivery](delivery.md).

## Finding Admission

A finding belongs to the current slice when all four conditions hold:

1. Code evidence, deterministic reproduction, or direct specification
   contradiction confirms the behavior.
2. It occurs on a currently supported path.
3. It violates an explicit current-slice acceptance criterion.
4. It is not assigned to a future phase.

Otherwise use the [out-of-scope classifications](planning.md#out-of-scope-work).
For an admitted finding, remediate if the work fits the authorized slice;
broader ownership, architecture, contract, dependency, or verification changes
use the [decision gates](../development_policy.md#decision-gates). A confirmed
defect is not downgraded merely because its remedy needs a decision.

### Evidence And Severity

- Passing tests do not prove absence of an untested production defect.
- A plausible concern without confirmation is an unconfirmed risk.
- Missing real-environment tests are integration verification gaps unless
  current acceptance explicitly requires that environment.
- A peer's mandatory standard-contract violation first receives the specified
  peer-error handling. Extra recovery is optional unless explicitly planned.
- Dead code without a production caller is legacy cleanup unless it creates a
  current build, behavior, ownership, or maintenance contradiction explicitly
  covered by this slice.

Reserve P0/P1 for confirmed critical current-slice failures, such as unsafe
state corruption, unbounded supported lifecycles, lost required resource
ownership, or a broken primary flow. Severity does not bring future work into
the current phase.

## Initial Review

After implementation and focused verification, perform one initial review as
the uninterrupted next step, before final full verification or the implementation
commit checkpoint. No separate user request is needed within an authorized
implementation task.

Inspect the complete slice diff, agreed plan and acceptance criteria, baseline
stage dispositions, direct call paths/lifecycle dependencies, relevant standard
and free5GC evidence, tests, and skipped verification as applicable.

## Plan Conformance

Maintain a working conformance map for a substantial slice with plan-defined
review, acceptance/completion criteria, or required commands. Every plan
commitment to production behavior, a deliverable, acceptance/completion, or
verification is normative, including narrative prose, unless explicitly marked
background, non-goal, optional, or approved deferral.

For each item, identify its production path and direct verification evidence
(deterministic test or command/result), or its explicitly approved deferral or
plan change. Inspect all applicable directions:

1. implementation to plan: changes fit the approved slice;
2. plan to implementation: every requirement has implementation and verification;
3. baseline to plan: stages and shared semantic representations have explicit
   dispositions and retain their claimed invariants.

Repository-wide test success does not establish criterion coverage. Tests must
exercise the exact production owner, entry point, lifecycle state, and outcome
named by the requirement. A mock replacing the claimed behavior, an adjacent
operation/state/phase, a call-only assertion without the required downstream
effect/fencing, or unverified composition of tests is indirect evidence.

Mocks outside the behavior under review remain valid. Shared mechanisms count
when tests explicitly run the required entry point and state through them.
Required connecting contracts need explicit verification.

Missing, unverified, silently deferred, or inference-only requirements remain
open and block a completion claim or completed implementation-commit checkpoint.
Focused commands cannot replace plan-required full commands without an approved
plan change; report an omitted command as an open verification gap.

## Follow-up Review

After each authorized in-scope remediation and focused verification, review its
diff, the finding's direct dependencies, regression tests, and directly affected
behavior. Continue this targeted loop until closure or a genuine blocker.

A new full repository review requires an explicit user request or concrete
evidence from remediation of a broader P0/P1 regression. Adjacent issues go to
the backlog. A passing targeted review closes the finding, rather than starting
open-ended reviews to prove the absence of unrelated defects.

## Final Conformance Check

After remediation and targeted checks, reconcile all current requirements,
baseline dispositions, narrative commitments, acceptance/review/completion
items, required commands, and approved deferrals against the final diff and
exact test paths. Use the working map and still-valid evidence; source changes
or gaps prompt targeted reading and verification under the
[loading and reuse rules](../development_policy.md#loading-and-evidence-reuse).

Ensure required final full verification covers the final state. Keep missing
or indirect evidence open until directly verified or explicitly deferred.
For documentation changes, use the full-content
[language check](documentation.md#language) on the final changed content.

If behavior is implemented but required evidence is open, report
implementation-complete and verification-incomplete. Green test suites do not
override that distinction. A pending Git approval is tracked separately from
non-Git conformance; it grants no operation authority.

## Review Output

Lead with confirmed current-slice defects and explain behavior/consequence.
Separate blockers, deferred work, cleanup, hardening, verification gaps, and
unconfirmed risks. State the actual scope, commands run, skips, and remaining
gaps. For free5GC alignment, name the exemplar and boundary used.

For substantial slices, summarize each required criterion as satisfied, open,
explicitly deferred, or changed with its evidence. Distinguish implemented
behavior from closure of the required verification matrix and state whether
the slice is complete, partial, or intentionally deferred. Link existing
evidence rather than repeating full diffs at each handoff.

Durable review records follow [Documentation](documentation.md#document-ownership)
when requested by the user or required by the plan; otherwise report in the
conversation.
