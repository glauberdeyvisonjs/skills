# skills

My personal [Claude Code](https://claude.com/claude-code) skills. Each skill lives in its own folder with:

- `SKILL.md` — the skill itself (English, written for the model)
- `README.md` — summary in English
- `README.pt-BR.md` — resumo em português

Minhas skills pessoais do Claude Code. Cada skill fica numa pasta com a skill em si (`SKILL.md`), um resumo em inglês (`README.md`) e um resumo em português (`README.pt-BR.md`).

## Skills

| Skill | Description | Descrição |
|-------|-------------|-----------|
| [custom-workflow](custom-workflow/) | Plan → approval → one multi-agent Workflow per task set, with per-phase and final Codex review | Plan → aprovação → Workflow multi-agent, com revisão do Codex por fase e geral |
| [custom-code-review](custom-code-review/) | Read-only review of the current branch's PR against its issue spec, with tests, optional parallel Codex review and a recommendation | Revisão só de leitura da PR da branch atual contra a spec da issue, com testes, revisão paralela opcional do Codex e recomendação |
| [review-prs](review-prs/) | List the current repository's PRs awaiting your review or re-review | Lista PRs do repositório atual pendentes da sua revisão ou re-review |
| [fix-prs](fix-prs/) | List PRs needing your fixes and missing review re-requests | Lista PRs pendentes de ajustes e re-requests esquecidos |
| [tasks-pendentes](tasks-pendentes/) | List actionable Linear issues by priority and blockers | Lista issues disponíveis no Linear por prioridade e bloqueios |
| [before-pr-protocol](before-pr-protocol/) | Run the current repository's test suites before a PR | Roda as suítes de testes do repositório atual antes da PR |

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill <name> -g
```

Or copy `<name>/SKILL.md` to `~/.claude/skills/<name>/SKILL.md`.
