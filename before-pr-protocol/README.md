# before-pr-protocol

**English** · [Português](README.pt-BR.md) · [SKILL.md](SKILL.md)

Run the current repository's automated test suites before opening a PR and wait for the results.

Usage: `/before-pr-protocol`.

## Requirements

A local checkout with its test dependencies and documented commands. Database-backed or service-backed tests require confirmed isolated test resources.

## Flow

1. Discover the current repository's actual test commands.
2. Cover every configured suite: unit, integration, end-to-end and browser tests.
3. Inspect setup and hooks for database or external-service side effects before execution.
4. Run available safe suites, wait for completion and report missing or blocked coverage.

## Output and boundaries

Portuguese table: Suíte, Comando, Resultado, Detalhes. Only the current repository and branch are tested. No migrations, resets or seeding without an explicit request for the exact command, including indirect script calls. Does not fix code, create a PR or push.

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill before-pr-protocol -g
```

Or copy `before-pr-protocol/SKILL.md` to `~/.claude/skills/before-pr-protocol/SKILL.md`.
