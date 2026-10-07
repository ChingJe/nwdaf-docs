# Review Handoff And Git Delivery

This module owns review confirmation, document-status updates, commit messages,
and Git authorization across the workspace repositories.

## User-review Handoff

When implementation is ready for user review, summarize the changes, relevant
verification, and remaining work. Keep changes unstaged and uncommitted, and
the active plan `Ready for User Review` or equivalent, unless the user has
already requested a commit or another delivery form.

## Review Confirmation And Document Status

An explicit request to commit the current work confirms the user's review and
authorizes staging and committing that work. Prepare the commit message and
complete the commit without a separate proposal or message-approval step.
Ask only when the intended scope cannot reasonably be determined from context.

Before committing, update the related existing plans, implementation/review
records, and indexes to reflect user review and actual progress. Include these
status changes in the commit. Preserve historical review and verification facts;
remaining required implementation or external acceptance keeps the plan open.
A commit request does not make unfinished work complete. Ordinary changes do
not need a new status document.

## Commit

Commit the task's changes, preserve unrelated edits, and keep independent
repositories in separate commits. Choose a coherent split when the work spans
distinct changes. Report the resulting commit hashes and any remaining work.

## Push And History Operations

A request for commit and push authorizes both. A commit request alone does not
authorize push. Use the requested or established remote and branch; ask when
the target is ambiguous. Follow the workspace's
[execution permissions](../development_policy.md#workspace-boundaries-and-execution).

Amend, rebase, reset, cherry-pick, and other history operations require explicit
authorization for the intended operation.

## Commit Message Format

Every commit in a workspace repository has an English title and a non-empty
description, separated by a blank line:

```text
<type>(<scope>): <summary>

<description>
```

The scope is optional: `<type>: <summary>` is also valid. Types are `feat`,
`fix`, `docs`, `refactor`, `test`, and `chore`. Use an imperative, concise,
concrete summary. The description explains the changes and their purpose or
result, using a short paragraph or bullets appropriate to the change.

Implementation messages describe the technical delta and rationale. Internal
project-management/review labels (`phase`, `batch`, `round`, `priority`, finding
IDs, or remediation iterations) do not belong in them. Documentation messages
may name such an identifier when it is the document being changed.
