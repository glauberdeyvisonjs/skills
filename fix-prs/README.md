# fix-prs

**English** · [Português](README.pt-BR.md) · [SKILL.md](SKILL.md)

List your pull requests in the current repository with outstanding review feedback and identify corrected PRs missing a review re-request.

Usage: `/fix-prs`.

## Requirements

Authenticated GitHub CLI (`gh`) with access to the current repository. Linear is optional for verifying issue links.

## Flow

1. Find open PRs in the current repository authored by or assigned to you, deduplicating both searches.
2. Inspect requested changes, actionable comments, commits and review-request history.
3. List feedback still requiring fixes; exclude corrected PRs already awaiting another review.
4. Report corrected PRs without a new review request separately; do not assume any later commit fixes all feedback.

## Output and boundaries

Portuguese table: Código, Título, Link da Issue no Linear, Branch. Oldest PRs first, plus a separate Re-request pendente section. Does not edit code or send re-requests.

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill fix-prs -g
```

Or copy `fix-prs/SKILL.md` to `~/.claude/skills/fix-prs/SKILL.md`.
