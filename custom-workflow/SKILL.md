---
name: custom-workflow
description: Use when the user invokes /custom-workflow, or asks to develop one or more tasks (issue-tracker tickets or pasted text) through a planned, multi-agent workflow with developer agents and Codex review.
---

# Custom Workflow

Prereq check → grill task source, tasks and models → plan → approval → one Workflow (one step per plan phase + final review) → report → commit only on approval.

**This is the user's standing process. Follow every rule below unless the user emphatically asks to change a specific part in the current conversation.** Invoking this skill is the explicit opt-in for the Workflow tool.

## Step 0 — Prerequisite: grill skills

Check that both `~/.claude/skills/grill-me/SKILL.md` and `~/.claude/skills/grilling/SKILL.md` exist (`ls -L`).

- Present → continue.
- Missing → AskUserQuestion: install now or abort. Install = `npx skills add mattpocock/skills --skill grill-me --skill grilling -g`, then re-check. Declined or install failed → stop: "custom-workflow requires grill-me; aborting." Do not continue without it and do not ask the model questions any other way.

`grill-me` cannot be invoked by the model; invoke skill `grilling` (grill-me delegates to it).

## Step 1 — Grill the task source and tasks

Never assume a tracker. Invoke skill `grilling` (same rounds as Step 2 — ask T1 in round 1 together with Q1/Q2):

- **T1** Task source. Offer only what this session can read:
  - `Linear` — only if Linear MCP tools are available.
  - `Jira` — only if a Jira/Atlassian MCP is available (authenticate it first if it requires authentication).
  - `Text` — the user pastes the task description. Always offered.

  Recommend the source the invoking prompt points to (issue keys, tracker URLs, pasted spec); otherwise `Text`. A source the user picks that has no working MCP → say so and re-ask T1.
- **T2** (round 2, depends on T1) The task(s): issue keys/URLs for Linear/Jira, or the pasted text for `Text`. Skip T2 when the invoking prompt already gives tasks that match T1.

Record `TASK_SOURCE` and the task list.

## Step 2 — Grill the model choices

Same `grilling` session as Step 1, scoped to exactly these decisions, in its round format with a recommended answer each:

- Round 1 — **Q1** Developer agents' model: `haiku` | `sonnet` | `opus` | `fable` (➡️ haiku). **Q2** Review with Codex? yes | no (➡️ yes).
- Round 2 — **Q3** Reviewer agents' model: `haiku` | `sonnet` | `opus` | `fable` (➡️ haiku). Frame it with the Q2 answer: yes → "this agent only relays to Codex"; no → "this agent reviews the diff itself".
- Round 2, only if Q2 = yes — **Q4** Codex model + reasoning effort for **per-phase** reviews. **Q5** Codex model + reasoning effort for the **final general** review. Options: the `model` in `~/.codex/config.toml` or any model the user types; effort `low` | `medium` | `high` (➡️ Q4: `gpt-6-astra` · `high`. Q5: `gpt-6-astra` · `medium` — the final diff is larger and `high` risks the 10-min Bash timeout).

Record `DEV_MODEL`, `USE_CODEX`, `REVIEW_MODEL`, and when `USE_CODEX`: `PHASE_CODEX = {model, effort}`, `FINAL_CODEX = {model, effort}`. They hold for the whole run. If the user later asks to change one, re-grill only that one.

## Step 3 — Build the plan

Load each task from `TASK_SOURCE`: the issue plus its comments through the Linear or Jira MCP, or the pasted text as-is. Explore only the code needed to plan. Pick the planning mode, first match wins:

1. A system reminder says plan mode is active → plan in plan mode; ExitPlanMode is the approval gate.
2. Skill `superpowers:writing-plans` is available → use it.
3. Neither → tell the user to switch to plan mode (Shift+Tab) and stop.

The plan contains:

- **Implementation plan** per task: phases → steps, files per step, acceptance criteria mapped (use the task's own numbering when it has one).
- **Multi-task analysis** (2+ tasks): dependency graph, suggested order, what runs in parallel and why. Parallel only with disjoint file sets; overlapping tracks go sequential. Worktree isolation only if the user approves it in this plan.
- **Workflow plan** table, final row always the general review:

| Step | Track | Dev (model) | Review | Files | Depends on | Parallel with |
|------|-------|-------------|--------|-------|------------|---------------|
| F1 — Backend | TASK-A/api | 1 × DEV_MODEL | REVIEW_MODEL → Codex PHASE_CODEX (or REVIEW_MODEL self-review) | src/services/x.ts | — | TASK-B/api |
| Final review | all | fixes: DEV_MODEL | REVIEW_MODEL → Codex FINAL_CODEX (or self-review), full diff | all touched files | all phases | — |

- Project rules devs must obey (from CLAUDE.md and memory: test policy, no commits, no migrations, etc.) — inject these into dev prompts; subagents do not see your memory.

## Step 4 — Approval gate

Present both plans. Ask for approval (ExitPlanMode in plan mode, AskUserQuestion otherwise). No workflow script and no code edits before explicit approval. Changes requested → revise, ask again.

## Step 5 — Run one Workflow

Load skill `workflow-authoring`, then write ONE script:

- One `phase()` step per plan phase, in plan order; tracks inside a step run in parallel. Final step `Final review` over the whole diff.
- `meta.phases` = one entry per plan phase + `{ title: 'Final review' }`. Agent `phase` options prefixed `F<n>` / `Final` so each step's devs and reviewers group separately.
- End of each step → `log()` one line per track. Any track not `approved` → return right after that step.
- Review loop at both levels: max 3 review rounds, fixes between them (2 max).

```js
export const meta = {
  name: 'custom-workflow',
  description: 'Dev agents + review per plan phase, then final general review',
  phases: [{ title: 'F1 — <name>' }, { title: 'F2 — <name>' }, { title: 'Final review' }], // pure literal
}
const MAX_REVIEWS = 3
const { devModel, reviewModel, useCodex, phaseCodex, finalCodex } = args  // *Codex = {model, effort}, null when !useCodex
const DEV = { type: 'object', required: ['status', 'files_changed', 'summary'], properties: {
  status: { enum: ['done', 'blocked'] }, files_changed: { type: 'array', items: { type: 'string' } },
  summary: { type: 'string' }, blocker: { type: 'string' } } }
const REVIEW = { type: 'object', required: ['review_ran'], properties: {
  review_ran: { type: 'boolean' }, command: { type: 'string' }, error: { type: 'string' },
  verdict: { enum: ['APPROVED', 'CHANGES_REQUESTED'] },
  findings: { type: 'array', items: { type: 'object', properties: {
    severity: { enum: ['P0', 'P1', 'P2', 'P3'] }, file: { type: 'string' }, line: { type: 'integer' }, text: { type: 'string' } } } },
  raw_output_path: { type: 'string' } } }

// Shared loop: review → fix → review ... (max 3 reviews). `dev` = result of the initial dev run, or null for the final review.
// `codex` = phaseCodex for plan phases, finalCodex for the final review.
async function reviewLoop(key, t, dev, codex) {
  for (let round = 1; round <= MAX_REVIEWS; round++) {
    const rev = await agent(reviewPrompt(t, round, codex), { model: reviewModel, phase: `${key} Review · ${t.name}`, label: `review:${t.id}#${round}`, schema: REVIEW })
    if (!rev || !rev.review_ran) { log(`${key} ${t.id}: review failed to run — stopping`); return { id: t.id, status: 'review_unavailable', rev } }
    if (rev.verdict === 'APPROVED') return { id: t.id, status: 'approved', rounds: round, dev, rev }
    if (round === MAX_REVIEWS) return { id: t.id, status: 'review_exhausted', findings: rev.findings, dev }
    dev = await agent(fixPrompt(t, rev.findings), { model: devModel, phase: `${key} Dev · ${t.name}`, label: `fix:${t.id}#${round}`, schema: DEV })
    if (!dev || dev.status !== 'done') return { id: t.id, status: 'dev_blocked', dev }
  }
}

async function runTrack(p, t) {
  const dev = await agent(t.devPrompt, { model: devModel, phase: `${p.key} Dev · ${t.name}`, label: `dev:${t.id}`, schema: DEV })
  if (!dev || dev.status !== 'done') return { id: t.id, status: 'dev_blocked', dev }
  return reviewLoop(p.key, t, dev, phaseCodex)
}

const report = []
for (const p of args.phases) {
  phase(p.title)
  const results = (await parallel(p.tracks.map(t => () => runTrack(p, t))))
    .map((r, i) => r || { id: p.tracks[i].id, status: 'agent_died' })
  results.forEach(r => log(`${p.key} ${r.id}: ${r.status}${r.rounds ? ` (${r.rounds} review rounds)` : ''}`))
  report.push({ phase: p.title, results })
  if (results.some(r => r.status !== 'approved')) return { stoppedAt: p.title, report }
}

phase('Final review')
const allFiles = [...new Set(report.flatMap(s => s.results.flatMap(r => r.dev?.files_changed || [])))]
const final = (await reviewLoop('Final', { ...args.final, files: allFiles }, null, finalCodex)) || { id: 'final', status: 'agent_died' }
log(`Final review: ${final.status}${final.rounds ? ` (${final.rounds} rounds)` : ''}`)
report.push({ phase: 'Final review', results: [final] })
return { stoppedAt: final.status === 'approved' ? null : 'Final review', report }
```

`args`: `{ devModel, reviewModel, useCodex, phaseCodex: { model, effort } | null, finalCodex: { model, effort } | null, phases: [{ key: 'F1', title: 'F1 — <name>', tracks: [{ id, name, files, devPrompt, summary, repoPath, scratchDir }] }], final: { id: 'final', name: 'general', summary: '<all tasks + acceptance criteria>', repoPath, scratchDir } }`. Titles in `args` must match `meta.phases` exactly. `reviewPrompt(t, round, codex)` builds the Codex-relay prompt (with `codex.model` / `codex.effort`) when `useCodex`, else the self-review prompt. `fixPrompt(t, findings)` lists findings and allowed files.

### Dev prompt

Plan steps for this track verbatim, allowed file list, acceptance criteria, injected project rules, and: "Do not commit. Do not run DB migrations. Do not edit files outside the list — if needed, return status=blocked with the reason." Fix prompts add the review findings (P0–P2 must be fixed; P3 optional).

### Review request (used by both reviewer modes)

"Run `git diff` and `git status`. Review ONLY these files: <files>. Task: <summary + acceptance criteria>. List findings as `P0|P1|P2|P3 file:line — text`. Request changes only for P0–P2. End with exactly one line: `VERDICT: APPROVED` or `VERDICT: CHANGES_REQUESTED`." Final review adds: "Focus on cross-phase and cross-task integration: contracts, naming consistency, duplication, acceptance criteria that only close with everything together."

### Reviewer prompt — `useCodex = true` (relay; copy this shape)

> You are a relay, not a reviewer. Do not read the diff, do not judge code, do not add opinions.
> 1. Write the review request to `<scratchDir>/review-<id>-<round>.txt` with a heredoc.
> 2. Run with Bash (timeout 600000):
>    `codex exec -m "<codex.model>" -c model_reasoning_effort="<codex.effort>" -s read-only -C "<repoPath>" --ephemeral -o "<scratchDir>/review-<id>-<round>.md" - < "<scratchDir>/review-<id>-<round>.txt"`
> 3. Failure (not found, auth, non-zero exit, timeout, empty output) → retry once; still failing → return `review_ran=false` with the exact error. Never substitute your own review.
> 4. Success → `review_ran=true`, verdict and findings copied from the Codex output, `raw_output_path`.

Codex rejects the chosen model or effort → that is a failure (`review_ran=false` with the exact error). Never fall back to another model silently; the main agent asks the user.

### Reviewer prompt — `useCodex = false`

The review request itself, plus: "Return `review_ran=true`, your verdict and findings." Use this mode only when the user answered Q2 = no.

## Step 6 — Report and finish

When the Workflow returns:

1. Report per step and track: status, files changed, review rounds, open P3s. Keep it short.
2. `stoppedAt` set → explain the failure, ask how to proceed, apply the decision, relaunch with `resumeFromRunId` (completed steps return from cache).
3. All approved → final summary (acceptance criteria covered, remaining P3s), then ask whether to commit / open a PR. Commit only after explicit approval.

## Red flags — stop

| Thought | Reality |
|---------|---------|
| "grill-me is missing, I'll just ask with AskUserQuestion" | Ask to install; declined → abort. |
| "Prompt has an issue key, it must be Linear" | Ask T1. Recommend, never assume. |
| "User used haiku last time, skip the grill" | Grill every run. |
| "Final review can reuse the per-phase Codex model" | Ask Q5 separately. |
| "Chosen Codex model failed, use the config default" | Fail and ask. |
| "Codex failed, the reviewer agent can review this small diff" | Only when Q2 = no. Otherwise return review_unavailable and ask. |
| "Plan is obvious, start the workflow" | No script before explicit approval. |
| "Every phase passed, skip the final review" | Final review always runs. |
| "4th review round will surely pass" | Max 3. Ask the user. |
| "One phase failed but the next doesn't depend on it" | Stop after the failed step. Ask. |
| "These tracks touch the same file but can run in parallel" | Sequential. |
| "Separate Workflow per phase is cleaner" | One Workflow, one step per phase + final review. |
