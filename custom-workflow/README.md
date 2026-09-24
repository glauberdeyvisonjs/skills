# custom-workflow

**English** · [Português](README.pt-BR.md) · [SKILL.md](SKILL.md)

A Claude Code skill for developing one or more tasks (issue-tracker tickets or pasted text) through a
planned multi-agent workflow: developer agents, Codex review after every phase and a general
review at the end. The user approves everything before any work starts.

Usage: `/custom-workflow` — optionally followed by the tasks (issue keys, URLs or text).

## Requirements

- [Claude Code](https://claude.com/claude-code) with the Workflow tool.
- The `grill-me` and `grilling` skills ([mattpocock/skills](https://github.com/mattpocock/skills)). If they are missing, the skill offers to install them and aborts if you decline.
- [Codex CLI](https://github.com/openai/codex), if you review with Codex.
- Optional: `superpowers:writing-plans`, used when plan mode is off. Linear or Jira MCP, to read tasks from a tracker.

## Flow

| # | Step | Who | Gate |
|---|------|-----|------|
| 0 | Check that `grill-me` and `grilling` are installed. If missing, offer to install; abort on decline | Main agent | Install |
| 1–2 | Grill: task source, tasks and models (see below) | Main agent + user | Answers |
| 3 | Read the issues and explore the code that is needed | Main agent | — |
| 4 | Plan: plan mode if active → else `superpowers:writing-plans` → else ask the user to switch to plan mode | Main agent | — |
| 5 | With several tasks: suggested order and what runs in parallel | Main agent | — |
| 6 | Present the implementation plan and the workflow plan (phase × tracks × devs × reviewers × models) | Main agent | **Approval** |
| 7 | One Workflow: one step per plan phase + a Final review step | Workflow | — |
| 8 | End of each step: `log()` summary. On failure → stop | Workflow | Stops on failure |
| 9 | Consolidated report. If stopped: ask how to proceed, resume with `resumeFromRunId` | Main agent | — |
| 10 | Ask about commit/PR. Commit only after approval | Main agent | Approval |

## Grill (every run)

| Round | Question | Options | Recommended |
|-------|----------|---------|-------------|
| 1 | T1 — Task source | Linear (if Linear MCP) / Jira (if Jira MCP) / Text (paste) | what the prompt points to, else Text |
| 2 | T2 — The task(s) *(skipped if the prompt already has them)* | keys/URLs, or pasted text | — |
| 1 | Q1 — Developer agents' model | haiku / sonnet / opus / fable | haiku |
| 1 | Q2 — Review with Codex? | yes / no | yes |
| 2 | Q3 — Reviewer agents' model | haiku / sonnet / opus / fable | haiku |
| 2 | Q4 — Codex model + effort for per-phase reviews *(if Q2 = yes)* | `~/.codex/config.toml` model or any · low / medium / high | gpt-6-astra · high |
| 2 | Q5 — Codex model + effort for the final review *(if Q2 = yes)* | same | gpt-6-astra · medium (larger diff, avoids timeout) |

- No tracker is assumed: the source is always asked, and only sources with a working MCP are offered.
- **Q2 = yes** → the reviewer is only a **relay**: it runs Codex and passes on the result. If Codex fails or rejects the model, the review fails and the user is asked. The relay never reviews on its own, and there is no silent fallback to another model.
- **Q2 = no** → the reviewer agent (Q3 model) reviews the diff itself.

## Review: per phase + general

- **Per phase:** catches problems early, before the next phase builds on them.
- **General:** catches what only shows with everything together: integration across phases and tasks, contracts, naming, duplication and acceptance criteria.
- At both levels: at most **3 review rounds, with 2 fixes in between**. If round 3 still requests changes, the user decides how to proceed.
- Only P0–P2 block. P3 is listed but does not block.

```
F1: Dev ─ Review#1 ─(fix)─ Review#2 ─(fix)─ Review#3 ─┐
F2: Dev ─ Review ... (parallel tracks inside a phase)  ├─ all approved
...                                                     ┘
Final review: Review#1 (full diff) ─(fix)─ #2 ─(fix)─ #3 ─ approved → final report
```

## Rules

- Parallel tracks share the working tree only when their file sets are disjoint. Worktree isolation only when approved in the plan.
- Developers do not commit, do not run migrations and do not edit files outside their track.
- A Codex timeout (10 min Bash limit) counts as a failure, and the user is asked.
- In `/workflows`, each phase shows as a step, with the groups `F<n> Dev · <track>` and `F<n> Review · <track>`.

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill custom-workflow -g
```

Or copy `custom-workflow/SKILL.md` to `~/.claude/skills/custom-workflow/SKILL.md`.
