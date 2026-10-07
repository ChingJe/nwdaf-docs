# Review And Conformance

Use this module for code/document review, finding closure, and completion
reconciliation. Review-only work produces findings, not automatic edits.
Authorized code remediation additionally uses [Implementation](implementation.md);
ownership or boundary decisions use [Architecture](architecture.md).
User-review handoff uses [Delivery](delivery.md).

## Finding Admission

A finding belongs to the current task when direct evidence confirms a defect
in the behavior or documents under review. Relate it to the user requirement,
supported behavior, applicable contract, or acceptance criterion it violates.
Work explicitly assigned to a future phase remains outside the current slice.

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

The implementer is unrestricted. After completing code changes and necessary
verification, the implementer launches an independent subagent to review the
production code, test code, and acceptance evidence. Self-checks do not replace
this review. Test-only code changes also require independent review; prose-only
changes do not require a subagent unless the user requests one.

Give the reviewer the task scope, requirement/source locations, changed files,
and available verification results and gaps. The reviewer inspects primary
evidence directly and forms its own conclusions rather than approving the
implementer's summary. Review is read-only; the implementer handles fixes.
The reviewer does not delegate another reviewer. If independent review is
unavailable, report the gap rather than claiming self-review as its replacement.

Inspect the complete slice diff, agreed plan and acceptance criteria, baseline
stage dispositions, direct call paths/lifecycle dependencies, relevant standard
and free5GC evidence, tests, and skipped verification as applicable.

Use existing valid results. Additional execution needs a concrete question or
gap; review does not automatically trigger another test run.

## Test Code Review

Review tests added or modified by the task and existing tests directly relied
on for its acceptance, using [Testing](testing.md). Assess:

- whether each test protects a requirement, contract, or confirmed regression;
- whether its assertions would catch the target failure and check the required
  outcome rather than only an incidental call or implementation detail;
- whether expected values have an independent basis and mocks, fixtures, or
  test-only paths bypass the behavior claimed as verified;
- whether coverage duplicates existing protection or introduces unnecessary
  fixtures, abstractions, or test infrastructure.

Read the test code and available evidence first. Additional experiments need a
specific doubt; proving every test by deliberately breaking production code is
not required. Repair evidence gaps affecting current acceptance, and remove or
merge current-change tests that add no useful protection. Do not expand review
into unrelated repository-wide test cleanup or require a per-test review ledger.

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

Repository-wide test success does not establish criterion coverage. Assess
whether evidence directly establishes each claimed behavior using
[test evidence design](testing.md#design-effective-evidence). Indirect evidence
does not close a requirement for a specific production path or outcome.

Missing, unverified, silently deferred, or inference-only requirements remain
open and block a claim that those requirements are complete.
Focused commands cannot replace plan-required full commands without an approved
plan change; report an omitted command as an open verification gap.

## Follow-up Review

For code review findings, the implementer returns fixes needing re-review to
the same independent reviewer. Review the fix's diff, direct dependencies,
regression coverage, and affected behavior. Close the finding when the fix and
required verification establish that the defect is addressed. If the original
reviewer is unavailable, a replacement uses the existing findings and evidence
for the same focused scope.

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
For documentation changes, apply the relevant
[documentation requirements](documentation.md).

If behavior is implemented but required evidence is open, report
implementation-complete and verification-incomplete. Green test suites do not
override that distinction. Git authorization and status updates follow
[Delivery](delivery.md).

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
