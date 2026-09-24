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

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill <name> -g
```

Or copy `<name>/SKILL.md` to `~/.claude/skills/<name>/SKILL.md`.
