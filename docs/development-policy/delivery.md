# Review Handoff And Git Delivery

Use this module for completion handoff, document-status transitions, commit
proposals, approved staging/commits, pushes, and history operations. Preparing
completion also uses [Review](review.md); actual document edits use
[Documentation](documentation.md). Verification evidence follows the
[reuse rules](../development_policy.md#loading-and-evidence-reuse).

## User-review Handoff

Implementation, review, verification, and conformance do not themselves confirm
user review or authorize committing. Until user confirmation:

- keep intended changes unstaged/uncommitted and visible in the IDE, unless
  the user explicitly requests a different review form;
- keep the active plan `Ready for User Review`, `Review Pending`, or equivalent;
- report affected repositories, intended diff summary, verification, and gaps;
- stop for the user's review after this handoff.

### Review Confirmation And Document Status

An explicit request to commit the current delivered work, such as “進行 commit”
or “將計畫進行 commit”, confirms the user's review of that work. A question about
commit mechanics, hypothetical scenario, or future submission does not.
Explicit confirmation of the review result also advances this gate.

Before proposing a commit, synchronize the current active plan, implementation/
review records, and related index statuses to reflect that confirmation, such
as `User Review Confirmed` and `Commit Pending Approval`. Include these status
edits in the proposed change set; preserve historical review/verification facts.

Track plan approval, implementation, verification, external acceptance, and Git
delivery separately. Plan approval does not mean implementation occurred.
Confirmation covers the current deliverable; remaining required follow-up or
external evidence keeps the plan open. It does not imply all phases are
`Completed`, nor authorize stage/commit or push.

## Commit Proposal And Approval

Present a read-only proposal containing:

1. every repository to be committed;
2. the change summary and included files;
3. verification results and remaining gaps;
4. proposed repository-separated commit split and complete messages;
5. unrelated/pre-existing changes excluded from the commits.

Then stop for explicit approval of the current proposal. An unambiguous
affirmative answer to a direct proposal-approval question qualifies. Approval
before the proposal, a plan requirement for commits, checkpoint wording, or
“continue”, “finish”, and “implement the plan” does not.

Approval covers only the proposed repositories, changes, split, and complete
messages. Material changes to any of these require a revised proposal and new
approval. Check actual current worktree/scope before staging; stage only approved
changes, inspect actual staged content, and create only approved commits.
Preserve unrelated edits and keep repositories separate.

Use established review/conformance/verification evidence if it still covers
the content and conditions. Proposal and approval do not restart tests,
language passes, full reviews, or full-diff reporting. New relevant differences
require corresponding review/verification and any necessary reapproval.

At Git delivery, check only newly relevant state: the actual staged scope,
the commit result, and the push result, plus any required branch or outgoing
commit information not already established. Reuse unchanged content validation.
Do not repeatedly print the same status, log, or full diff between steps unless
an intervening change or unresolved question makes another inspection necessary.

Pass simple commit messages directly to Git with correctly quoted arguments.
Use a temporary message file only when complex content or tool constraints
justify it. Do not add a separate verification step solely to check that the
message file was created.

A required Git checkpoint is tracked as `pending user approval`, separately
from non-Git conformance; it is neither implementation evidence nor permission.
Report resulting commit hashes after committing.

## Push And History Operations

Commit approval does not authorize push, amend, rebase, reset, cherry-pick, or
other history rewriting. Obtain separate explicit approval for the intended
operation. For an approved push, confirm target branch/remote and outgoing
commits, execute with elevated permissions, and report the result. Reuse the
commits' valid evidence rather than treating the push as another implementation
checkpoint. Follow workspace safety and permission rules for all operations.

## Commit Message Format

The following format applies only to implementation commits in `NWDAF/`,
`PyAnLF/`, and `PyMTLF/`:

```text
<type>: <summary>
<type>(<scope>): <summary>
```

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`. Use an imperative,
concise, concrete subject and meaningful optional scope such as `anlf`, `mtlf`,
`processor`, `context`, or `consumer`.

The complete implementation message, subject and body, describes the technical
delta and rationale. Internal project-management/review labels (`phase`,
`batch`, `round`, `priority`, review iterations, finding IDs, remediation rounds)
do not belong anywhere in it. Bodies may explain behavior, migration, rationale,
and follow-up constraints.

`nwdaf-docs/` documentation commits may name a phase, plan, ledger, or finding
when that identifier is itself the documentation changed. The format restriction
above does not extend automatically to other repositories.
