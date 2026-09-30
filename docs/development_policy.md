# NWDAF Development Policy

This is the workspace's development-policy entry and loading guide. Detailed
rules have one owner in the modules below; use the applicable ones for the
current task rather than treating the corpus as a single required reading.

## Scope And Common Principles

Use this policy for implementation in `NWDAF/`, `PyAnLF/`, `PyMTLF/`, `nrf/`,
`smf-nwdaf-ext/`, `udm/`, `udr/`, and `adrf/`, and for implementation-oriented
plans under `nwdaf-docs/docs/plans/`. Its common evidence, documentation, review,
and delivery rules also apply to workspace analysis and documentation work.
Repository-specific commands and message rules retain their explicit scope.

The workspace guide establishes repository boundaries, authorization, safety,
and free5GC skill triggers. This entry routes detailed policy; technical skill
references are selected only for the relevant Go/standardized boundary.

- Make a right-sized, complete approved slice; preserve behavior unless
  explicitly replaced. Classify unrelated refactoring, future work, and
  speculative hardening instead of expanding scope.
- Prefer existing owners and direct behavior over speculative abstractions and
  nonessential helpers. New hash-level validation or integrity manifests need
  a contract requirement or explicit user decision.
- Match claims to direct evidence. Separate implemented behavior, verification,
  external acceptance, user review, and Git authorization.

## Task Routing

Read each applicable module in full when first needed. Combine modules for a
task spanning concerns; links inside a module apply when their stated condition
arises, not as an unconditional chain loading the whole corpus.

| Current concern/action | Applicable rules |
| --- | --- |
| Local specification/code questions | Evidence principles below; inspect relevant sources directly |
| Substantial implementation plan | [Planning](development-policy/planning.md) + [Documentation](development-policy/documentation.md) |
| Small approved internal code change | [Implementation](development-policy/implementation.md); add Planning for substantial slice/flow changes |
| Existing production-flow extension | Planning + Implementation; add Architecture for ownership/contract decisions |
| New package, ownership, cross-boundary design or feasibility advice | [Architecture](development-policy/architecture.md); add Planning for plans, Implementation for code edits |
| Requested code review | [Review](development-policy/review.md); add Architecture for boundary decisions |
| Authorized implementation-finding remediation | Review + Implementation; use decision gates for broader changes |
| Documentation edits/review | Documentation; add Review for findings or completion conformance |
| Completion and user-review handoff | Review + [Delivery](development-policy/delivery.md) |
| Commit proposal, approved stage/commit, push/history operation | Delivery; add Documentation when editing record statuses |
| Context-compaction recovery | Reload applicable instructions/modules and relevant active-plan contents as below |

## Loading And Evidence Reuse

A continuous work unit retains its objective, phase, repository boundaries, and
architecture through follow-ups such as “continue”, remediation, or targeted
review. Use already-read applicable rules and the established evidence map in
that context. A new concern/action loads any missing module, not a new full
repository orientation.

Read or supplement sources when entering the concern for the first time,
recovering from context compaction, changing task/repository/technical boundary,
learning that a rule or plan changed, or lacking context for a reliable decision.
Read technical evidence directly when needed for the current conclusion.
Higher-priority runtime/skill instructions requiring a fresh read still apply.

### Recovery After Context Compaction

Use the latest user goal and summary to locate the task, repository, slice, and
stage, then reestablish the basis from original sources:

1. Reread applicable workspace instructions, this entry, and the modules for
   the current task/stage.
2. Reread relevant active-plan goals, scope, decisions, acceptance criteria,
   required commands, progress, and open items; include parent commitments
   needed by this slice.
3. Reconcile summarized work with those sources, actual authorization, and
   evidence before continuing.

The summary navigates; original rules and plans establish requirements. Expand
reading as needed for recovery. With no active plan, restore the user request
and applicable rules without inventing a plan or recovery record. Recovery
reading does not automatically invalidate prior tests or reviews.

### Evidence Applicability

Review, focused/full verification, and full-document language checks apply to
the content, dependencies, tools/environment, and acceptance requirements they
covered. Reuse them for unchanged relevant conditions, including proposal and
approval follow-ups.

When relevant content/conditions materially change or applicability is uncertain,
check the affected scope and any required full verification. A plan-status-only
edit needs document checks, not rerunning unrelated production tests. Required
full commands still need evidence for the final state; focused checks do not
replace them.

Keep evidence in working context, necessary plan records, and concise handoffs;
no new registry is needed. Final conformance reconciles all current commitments
using the working map and valid results, returning to original passages for
changes or gaps. Worktree and staged-content checks at Git operations establish
actual scope; output full diffs only as differences and risk warrant.

## Evidence And Reference Order

Use the minimum sufficient local evidence, including constraints and
counterevidence, in this order unless current upstream information is required:

1. target repository production paths and direct tests;
2. active plan and confirmed decisions;
3. Release 18 OpenAPI YAML under `../specs/Rel-18/openapi/`;
4. relevant Release 18 TS text under `../specs/Rel-18/`;
5. applicable free5GC skill references;
6. generated free5GC OpenAPI code under workspace `resources/openapi/openapi/`;
7. exemplars under workspace `resources/references/free5gc-main/`.

Release 18 is the implementation baseline. Use Release 19/20 material only for
tasks needing that release and keep release distinctions explicit. Read the
selected release's OpenAPI README before assuming external dependencies exist.

OpenAPI defines paths, methods, fields, status codes, headers, and schemas;
TS defines procedure intent and role boundaries; exemplars guide implementation
shape without overriding contracts. Distinguish these from provenance records
and generated code. Local mirrors are not evidence of latest upstream behavior.

Label specification-defined behavior, observed implementation, inference, and
proposed design. For decisive field semantics or procedure legality, cite the
exact clause and shortest necessary quotation. Answer the exact question
concisely with necessary caveats; brevity does not reduce investigation depth.
Cross-boundary design advice uses Architecture even without edits.

## Decision Gates

Continue ordinary local choices preserving approved scope, owners, contracts,
and acceptance. Stop and request a decision when completion requires:

- changing agreed ownership, architecture, data flow, or state flow;
- changing external or explicitly standard-shaped contracts;
- adding an external dependency, service, or persistence mechanism;
- weakening/dropping acceptance or required verification;
- implementing another phase's behavior;
- choosing meaningful product behaviors with different outcomes;
- proceeding without required specifications, dependencies, permissions,
  tooling, or environment;
- replacing the strategy because a core assumption is false.

Optional cleanup/hardening or future work alone is not a blocker. For a genuine
gate, report the original plan/assumption, exact contradiction, realistic
options, recommendation/tradeoffs, and whether the plan must change first.

## Working Sequence

Define the slice and baseline when needed, establish conformance, implement and
focus verification, then perform the initial review. Close admitted findings
through authorized remediation and targeted review. Reconcile final conformance
and required verification, then hand off for user review with changes unstaged.
Delivery separately handles review confirmation, status synchronization, commit
proposal/approval, and authorized Git operations. Each stage adds its own
responsibility while reusing applicable evidence from earlier stages.
