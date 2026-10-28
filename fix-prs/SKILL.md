---
name: fix-prs
description: Use when the user invokes /fix-prs or asks to list their pull requests in the current repository awaiting fixes after review feedback, including fixes missing a review re-request.
---

# Fix PRs

Use GitHub CLI (`gh`) to find open PRs in the current local repository (its GitHub remote) authored by or assigned to the authenticated user that still need their response to review feedback.

## Collect evidence

- Identify the authenticated GitHub user.
- Search authored PRs and assigned PRs separately, then deduplicate by repository and PR number. Retrieve all result pages.
- Read reviews, review threads, inline and general comments, subsequent commits and review-request timeline events.
- A historical Request changes or Comment review is not sufficient by itself: determine whether actionable feedback is still pending.
- Commits after a review do not prove that all feedback was fixed. Check the changes and thread discussion before treating an item as addressed. Disclose uncertainty when the evidence is insufficient.

## Classify

| Situation | Result |
|-----------|--------|
| Authored by or assigned to the user; outstanding requested changes or actionable comments | Include in the fixes table |
| Feedback addressed by the user's subsequent commits and a fresh review requested | Exclude; awaiting another review |
| Feedback addressed by the user's subsequent commits, but no fresh review requested | Report separately as missing a re-request |
| Some feedback addressed, but other actionable feedback remains | Include in the fixes table |
| Outside the current repository, closed, merged, or neither authored by nor assigned to the user | Exclude |

Do not count resolved feedback, approvals, informational comments or the user's own replies as new pending fixes without supporting evidence.

## Output

Sort each list by PR creation date, oldest first. Use this Portuguese table for pending fixes:

| Código | Título | Link da Issue no Linear | Branch |
|--------|--------|------------------------|--------|

Use the linked issue key for Código when known, otherwise `repo#number`. Link the title to the PR. Extract the Linear link from the PR or verify it through Linear when available; never invent it. Use "Sem issue vinculada" when none is linked.

Under "Re-request pendente", list corrected PRs that still need the user to request another review, with a short explanation of the evidence. Keep uncertain cases separate and explain what could not be verified.

Say when no PRs qualify and disclose incomplete access. This skill reports work to do; it does not edit code, commit, push, post comments or send re-requests.
