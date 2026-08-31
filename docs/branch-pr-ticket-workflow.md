# Branch, pull request, and ticket workflow

This is the default workflow for Gradely implementation changes. A repository may add stricter local rules, but it must not silently weaken these rules.

## Branch naming

Gradely's existing branches use a mixture of underscore and hyphen separators. Preserve the established style of the relevant repository or the exact branch name supplied for a task; do not rename an existing convention merely for consistency.

- **Ticket-linked work:** start with the exact ticket key, followed by a concise task description.
  - Common examples: `GS-1005_student_reschedule_tutoring_class`, `GA-84_provide_vdl_callback_url`
  - Other existing forms, such as `729-use_websocket_to_imporove_tutoring_experience_2`, are valid historical conventions and must be preserved when referenced.
- **Ticketless work:** use a concise task description.
  - Example: `add_homework_indexes`

A ticket key is optional and is not limited to `GS`; preserve the exact key supplied for the work. Add a numeric suffix only when it is necessary to distinguish a follow-up branch.

Do not invent a ticket key, alter an existing key, or create a branch from an assumed ticket. If a task arrives without a key and it has not been stated to be a ticketless quick fix, ask whether it should be linked to Jira before creating the branch.

## Base branches

Confirm the target base branch for every task; do not infer it from the currently checked-out local branch.

For `gradely-2.1`, the remote default branch is `dev`. Release branches are explicit targets and must only be used when the task calls for that release line. Apply the same check in each repository before opening a PR.

## Pull requests

- Create a focused branch and pull request for each independent change.
- When a ticket is supplied, include the exact ticket key in the branch name and PR title/body, and link the ticket when the Jira integration is available.
- State: problem, implementation summary, affected repositories/contracts, validation performed, and rollout/rollback considerations.
- Do not claim tests passed unless they were run and their result was observed.
- Do not create or modify Jira tickets, comments, or status without the requested Jira connection and explicit user direction.

## Ticketless quick fixes

A ticket is not mandatory. When a task is explicitly ticketless, use the ticketless branch form and explain the reason/scope in the PR. Keep these changes small; work that changes a contract, data model, security boundary, or several repositories should normally have a ticket.

## Before starting implementation

1. Read the central architecture reference and the owning repository’s local guidance.
2. Confirm the intended target branch.
3. Determine whether a ticket key is supplied.
4. If no key is supplied, ask whether the change is ticketless before creating a branch.
5. Trace the affected behavior in source and identify tests before editing.

## Jira integration

Once Jira is connected, use it to read the ticket’s context and acceptance criteria before implementation. Ticket creation, updates, assignment, and transition remain explicit actions: confirm the requested operation rather than assuming it.
