# review-prs

**English** · [Português](README.pt-BR.md) · [SKILL.md](SKILL.md)

List the current repository's pull requests awaiting your review or re-review, respecting which developer has already taken responsibility.

Usage: `/review-prs`.

## Requirements

Authenticated GitHub CLI (`gh`) with access to the current repository. Linear is optional for verifying issue links.

## Flow

1. Find open PRs in the current repository requesting your review.
2. Check reviewers, comments, reviews and re-request history.
3. Include sole-reviewer requests, shared requests nobody has taken, and your re-requests after fixes.
4. Exclude shared requests already taken by another developer; flag ambiguous histories separately.

## Output and boundaries

Portuguese table: Código, Título, Link da Issue no Linear, Branch, Autor. Oldest PRs first. Lists the queue without performing or publishing a review.

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill review-prs -g
```

Or copy `review-prs/SKILL.md` to `~/.claude/skills/review-prs/SKILL.md`.
