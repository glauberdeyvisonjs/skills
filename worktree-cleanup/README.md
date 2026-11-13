# worktree-cleanup

**English** · [Português](README.pt-BR.md) · [SKILL.md](SKILL.md)

Tears down what a git worktree left behind: its own Docker containers, images, volumes and networks and, if you confirm, its own databases and cache keys. It can also delete the worktree folder. Shared resources always stay, for example a dev database every worktree uses.

Usage:
- `/worktree-cleanup <path|branch|name>...`: clean the given worktrees.
- `/worktree-cleanup --all --keep a,b`: sweep every worktree except the main checkout and the kept ones.

## Prerequisites

You need Docker and git. The GitHub CLI (`gh`) is optional and only used to check open PRs.

## Flow

1. Enters plan mode and asks three yes/no questions: clean the Docker resources? the databases and cache keys? the folder?
2. Learns the repo's conventions from its setup scripts and compose files: which resources are created per worktree and which are shared.
3. Inventories each target, read-only.
4. Asks you to tick the exact items to delete, one multi-select per category, with nothing pre-selected. It refuses to delete a folder that has uncommitted or unpushed work.
5. Removes only the approved items, verifies, and reports. The branch is kept.

## Install

```bash
npx skills add glauberdeyvisonjs/skills --skill worktree-cleanup -g
```

Or copy the `worktree-cleanup/` folder to `~/.claude/skills/` (Claude Code) and `~/.agents/skills/` (Codex).
