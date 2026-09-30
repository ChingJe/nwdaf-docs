# Documentation

Use this module when drafting, changing, or reviewing documentation, including
plans and durable review records. Substantial implementation planning also uses
[Planning](planning.md); finding/conformance work uses [Review](review.md).
Preparing user-review or Git delivery uses [Delivery](delivery.md).

## Language

Select prose language in this order:

1. explicit user instruction;
2. the existing document's dominant prose language;
3. the established language of current sibling documents in the same series;
4. the repository default.

New documents extending a series inherit its language unless the user requests
otherwise. `nwdaf-docs/` supports English and Traditional Chinese, consistently
within each document. Implementation-facing documents in production repositories
default to English unless their established convention differs.

Code identifiers, API names, paths, configuration keys, protocol terms, status
values, and direct quotations do not change the selected prose language.
Headings, sentences, explanations, table labels, checklists, captions, and
decision/status prose use it consistently.

Before declaring changed documentation ready, reviewed, or complete, reopen
and inspect the entire final changed document for language consistency. Include
headings, paragraphs, tables, captions, checklists, statuses, and decision records;
compare at least one current sibling for an established series. This is separate
from technical review, formatting, and diff checks. Grammatical mixed-language
prose or familiar terminology does not excuse inconsistency.

This completed pass supports later proposal/commit delivery under the
[evidence-reuse conditions](../development_policy.md#loading-and-evidence-reuse).
Subsequent content changes require the corresponding language check; approval
or a new conversation turn alone does not. Report selected language, selection
evidence, and the pass result only in the final user-facing conversation, not
in the target document, implementation record, review ledger, or commit message.

## Document Ownership

Update the canonical plan/policy first. Workspace routing belongs in `AGENTS.md`;
shared policy and loading routes belong in the policy entry; detailed stable
rules belong in their modules; phase-specific decisions belong in the phase
plan. Link evidence instead of copying entire parent documents.

Create/update a durable review record only if requested by the user or required
by the active plan. Maintain one ledger per implementation phase, recording
finding ID/status, owner phase, confirmed evidence, remediation, verification,
and closing commit when available. Append short iteration records rather than
new full documents for each remediation pass.

A separate document is appropriate when architecture, product scope, or the
canonical plan changes. Preserve historical review facts rather than rewriting
history to consolidate old files. Current statuses are synchronized at the
[delivery events](delivery.md#review-confirmation-and-document-status).

## Verification

Inspect intended changes, local navigation targets/headings, and document
requirements. Run `git diff --check`; directly read untracked new documents and
check them with `git diff --no-index --check /dev/null <new-file>`. Interpret
diagnostics, since a difference exit code is not itself a whitespace failure.

Documentation-only changes may skip Go/Python tests; state that they were not
run. Reuse still-applicable checks and supplement affected content when it
changes. Worktree/staged checks at Git operations confirm actual approved
scope rather than restarting document review.
