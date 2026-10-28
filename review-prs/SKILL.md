---
name: review-prs
description: Use when the user invokes /review-prs or asks to list the current repository's pull requests awaiting their review or re-review. This skill lists the queue; it does not perform code reviews.
---

# Review PRs

Use GitHub CLI (`gh`) to list open PRs in the current local repository (its GitHub remote) that still need the authenticated user's review.

## Collect evidence

- Identify the authenticated GitHub user.
- Search only the current repository. Retrieve all result pages.
- Inspect current requested reviewers, submitted reviews, inline comments and review-request timeline events. A re-request after fixes must be distinguished from a request already satisfied by a review.
- Inspect review-related discussion to determine which developer has taken responsibility. Ignore bots, author replies and unrelated comments for this purpose.

## Include

- The user is the only requested developer reviewer and the review is pending, including a re-request after fixes.
- The user and another developer are requested, the user started the review, and the author has re-requested the user's review after fixes.
- The user and another developer are requested, and neither has started reviewing or made review-related comments.

## Exclude

- PRs outside the current repository, or PRs already closed or merged.
- Shared review requests where another developer has already started reviewing or made review-related comments. Only one developer needs to participate.
- Requests already satisfied by the user without a subsequent pending re-request.

If both developers appear to have taken the review or the timeline is incomplete, flag the ambiguity separately instead of presenting it as a confirmed pending item.

## Output

Sort by PR creation date, oldest first. Return a table in Portuguese:

| Código | Título | Link da Issue no Linear | Branch | Autor |
|--------|--------|------------------------|--------|-------|

Use the linked issue key for Código when known, otherwise `repo#number`. Link the title to the PR. Extract the Linear link from the PR or verify it through Linear when available; never invent it. Use "Sem issue vinculada" when none is linked.

Say when no PRs qualify. Disclose incomplete access or history. This is a read-only queue report: do not review the diff, submit reviews, post comments, request reviews or change assignments.
