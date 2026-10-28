---
name: tasks-pendentes
description: Use when the user invokes /tasks-pendentes or asks to list their Linear issues ready for development or already in progress, filtered by blockers and ordered by priority.
---

# Tasks Pendentes

Use Linear to list the authenticated user's assigned issues that can be worked on now.

## Eligibility

Include only issues that:

- Belong to Linear and are assigned to the user.
- Are in Ready for Dev, Doing or To Fix.
- Have no pending blocking issues.

Read the actual blocking relations and each blocker's current status. A related issue or a child issue is not automatically a blocker.

A blocker at In Review or an earlier development stage is pending. At Preparing for Test or a later stage, it no longer prevents development: the implementation should already be on develop or a release branch.

Resolve equivalent status names using the team's workflow rather than alphabetic order. Unknown or inaccessible blockers must be flagged separately; do not silently classify the issue as ready.

## Collection and ordering

- Identify the user in Linear and retrieve all pages of matching issues.
- Read issue details, blocking relations, priority, due date, creation date and status history.
- Sort lexicographically by:
  1. Priority, highest first.
  2. Due date, earliest first.
  3. Creation date, oldest first.
  4. Time in the current status, longest first.
- Treat unset priority as unspecified, after explicitly prioritized issues; do not interpret Linear's numeric "no priority" value as highest priority. Put missing due dates after known dates.
- Derive time in status from the most recent transition into the current status, not the last generic update. If history is unavailable, disclose it and use the available preceding sort keys.

## Output

Return a Portuguese table in the sorted order:

| Código/Link | Título |
|-------------|--------|

Link each issue key to its Linear issue.

Say when no issues qualify. Mention any incomplete access or unresolved blocker states separately. Listing work does not authorize changing issue status, assigning issues, editing code or posting comments.
