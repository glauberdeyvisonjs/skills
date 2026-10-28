# tasks-pendentes

**English** · [Português](README.pt-BR.md) · [SKILL.md](SKILL.md)

List your Linear issues available for development or already in progress, filtered by blockers and sorted by priority.

Usage: `/tasks-pendentes`.

## Requirements

Access to Linear, including assigned issues, blocking relations and status history.

## Flow

1. Select your assigned issues in Ready for Dev, Doing or To Fix.
2. Exclude pending blockers at In Review or earlier; Preparing for Test or later no longer blocks development.
3. Sort by priority, due date, creation date and time in status, in that order.

## Output and boundaries

Portuguese table: Código/Link, Título. Unknown blockers are disclosed rather than guessed. Does not change issues or begin implementation.

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill tasks-pendentes -g
```

Or copy `tasks-pendentes/SKILL.md` to `~/.claude/skills/tasks-pendentes/SKILL.md`.
