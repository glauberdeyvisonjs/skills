# custom-code-review

**English** · [Português](README.pt-BR.md) · [SKILL.md](SKILL.md)

A Claude Code skill that reviews the current branch's pull request against its issue spec (Linear, Jira,
pasted text or another source). It optionally runs the automated tests and separates failures caused by the PR
from failures that already existed. It can run a second reviewer (Codex or another Claude subagent) in parallel.
The result is a table-based report with a recommendation. Read-only: it never edits code, never posts to
GitHub or to the tracker, and never runs migrations.

Usage: `/custom-code-review` on the PR's branch.

## Requirements

- [Claude Code](https://claude.com/claude-code) and the [GitHub CLI](https://cli.github.com/) (`gh`), authenticated.
- Optional: a Linear or Jira MCP to read the issue, [Codex CLI](https://github.com/openai/codex) for the second review, and an installed review skill (`code-review`, `superpowers:requesting-code-review`, `caveman:caveman-review`…).

## Flow

| # | Step | Who |
|---|------|-----|
| 1 | Scan: PR, inline comments, CI checks, local vs remote, task key, re-request mode, review skills, Codex | Main agent |
| 2 | Questions (AskUserQuestion, 2 rounds) | Main agent + user |
| 3 | Load the spec: ACs, business rules, out of scope; compare with the PR description | Main agent |
| 4 | Tests: test database isolation check → run → classify failures | Main agent + tests subagent |
| 5 | In parallel: primary reviewer (subagent, optional review skill) + second reviewer + tests | Subagents / Codex |
| 6 | Consolidate, dedupe, verify every P0–P2 in the code, discard false positives | Main agent |
| 7 | Report + recommendation; saved to `~/.claude/custom-code-review/<owner>-<repo>/pr-<n>/` | Main agent |

## Questions

| Round | Question | Options | Default |
|-------|----------|---------|---------|
| 1 | Q1 — Spec source | Linear / Jira / Text (paste) / Other (specify) | Linear |
| 1 | Q3 — Run automated tests? | yes / no | yes |
| 1 | Q5 — Minimum severity that blocks | P0 / P1 / P2 / P3 | P2 |
| 1 | Q6 — Second review in parallel? | Codex / another Claude subagent / no | Codex |
| 2 | Q2 — Task code/link | key from the PR description or the branch | branch key |
| 2 | Q4 — Reject for test failures? *(if Q3 = yes)* | only failures from the PR / any failure / no | only from the PR |
| 2 | Q7 — Review skill *(if 2+ installed)* | installed skills / none | — |
| 2 | Q8 — Second reviewer model *(Codex: model + effort)* | — | `config.toml` model · high |

## Re-review

Detected automatically (GitHub reviews or comments, or a previous local report). Three passes:

1. Status of each previous comment: fixed / partial / not fixed / regressed.
2. Diff since the last reviewed commit: regressions caused by the fixes.
3. Full fresh pass over the whole diff, looking for what earlier reviews missed.

## Tests

- **Database check before any test:** runs only if the test DB is SQLite or a test-only database different from the dev one. If that cannot be confirmed, the skill does not run the tests and asks you.
- Failure classes: `PR` (test added or changed by the PR, or passes on base and fails on the branch), `pre-existing`, `flaky`, `unclassified`. The base comparison uses a separate `git worktree`; it never uses `stash` or `checkout` in your tree.

## Severity and recommendation

| Sev | Meaning |
|-----|---------|
| P0 | data loss, security hole, production outage, broken build |
| P1 | bug in a main flow, AC not met, business rule violated, regression |
| P2 | edge case, missing validation, new behavior without tests, performance, risky out-of-scope change |
| P3 | readability, naming, style, suggestion |

Fixed rule, first match wins: any blocking item → **Reject**; ambiguous spec or unverifiable AC → **Comment without rejecting**; only items below the threshold → **Approve with suggestions**; nothing → **Approve**. Other suggestions come after the rule's outcome, never instead of it.

## Report

Findings · previous comments (re-review) · acceptance criteria · tests + CI · new business rules · spec questions · discarded findings · recommendation. Written in the PR's language.

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill custom-code-review -g
```

Or copy `custom-code-review/SKILL.md` to `~/.claude/skills/custom-code-review/SKILL.md`.
