---
name: worktree-cleanup
description: Tear down what a git worktree left behind — its own Docker containers, images, volumes and networks, and, if confirmed, its own databases and cache keys — after a plan-mode inventory and explicit user confirmation. User-invoked via /worktree-cleanup.
disable-model-invocation: true
---

# Worktree cleanup

Remove the **worktree-specific** footprint of one or more git worktrees. Shared resources stay: shared infra, a dev database every worktree uses, pulled images, external networks, the main checkout and the other worktrees. Nothing is deleted before the user confirms the exact list. Reply in the user's language.

**Arguments**
- `<target>...`: worktree path, branch, or worktree name. With no target, use the current worktree when the session runs inside a linked worktree; otherwise ask for one.
- `--all [--keep <a,b,...>]`: every worktree-scoped resource except those of the main checkout and the kept worktrees.

## 0. Enter plan mode and ask the scope

Switch to plan mode before any other step (Claude Code: `EnterPlanMode`; on agents without one, stay read-only until step 4). Everything up to step 3 is read-only.

Ask the scope with the structured question tool (Claude Code: `AskUserQuestion`; otherwise the agent's own ask-the-user tool, or plain chat as a last resort). Send one call with three yes/no questions:
1. **Docker**: "Clean the containers, volumes, images and networks specific to this worktree, keeping the shared ones?"
2. **Data**: "Clean the databases and cache keys specific to this worktree, keeping the shared ones?"
3. **Folder**: "Also delete the worktree folder (only if it has no uncommitted or unpushed work)?"

Inventory only the categories answered yes.

## 1. Learn the repo's conventions

Resources are named by the repo, not by this skill. Read how the repo creates per-worktree resources:
- worktree setup scripts and hooks (`scripts/**/setup*`, `orca.yaml`, `.superset/config.json`, `Makefile`, `.worktreeinclude`);
- the compose files worktrees use, and how they set the project name and volume names (often an interpolated variable such as `${WORKTREE_NAME}`);
- how the setup derives the **worktree name**: the normalisation rule, and any truncation or hash fallback;
- which databases the setup **creates per worktree** and which ones every worktree **shares**. A common split: one shared dev DB, plus a dedicated test DB created per worktree. Also find which container hosts them and which env keys carry cache or queue prefixes.

Done when you can name, for a given worktree, the exact compose project, volumes, databases and cache prefix the setup creates **for that worktree alone**. Anything you cannot tie to the worktree counts as shared.

## 2. Inventory each target (read-only)

Resolve the main checkout (the parent of `git rev-parse --path-format=absolute --git-common-dir`) and `git worktree list --porcelain`. A target's folder may already be gone; resolve it anyway from the branch or name.

**Docker** (if Q1 = yes):
- Containers: `docker ps -a --filter label=com.docker.compose.project.working_dir=<dir>`. This also works after the folder was deleted. Read the compose project name from these containers.
- Images built for that project (`<project>-<service>`). Pulled images used by other projects are shared.
- Volumes labelled `com.docker.compose.project=<project>`, plus the named volumes from step 1 resolved for this worktree.
- Networks labelled with the project. External networks are shared.

**Data** (if Q2 = yes):
- Databases: the per-worktree names from step 1, plus any database on the shared server whose name contains the worktree name as a whole token (hyphen and underscore variants). A token that is a prefix of another worktree's name (`foo-62` vs `foo-621`) is not a match. A DB the setup shares across worktrees is never a candidate, even if its name matches.
- Cache keys: only under a prefix unique to this worktree (`redis-cli --scan --pattern '<prefix>*'`, count them). If the prefix is a shared default, there are no candidates.

**Folder** (if Q3 = yes): check for uncommitted changes (`git -C <dir> status --porcelain`; untracked but ignored env files are expected), commits not on any remote (`git -C <dir> log --branches --not --remotes --oneline`), and an open PR (`gh pr list --head <branch>`). If the git metadata is broken (`.git` points to a pruned gitdir), compare the tracked files with the branch tip from the main checkout (`git show <branch>:<path> | cmp - <dir>/<path>`). Any work at risk blocks the folder deletion.

With `--all`, the targets are every worktree name found in these resources, minus the main checkout and `--keep`. Mark a target whose containers are **running** and that the user did not name as active.

## 3. Confirm the exact list

Ask with the question tool again, one question per category that has candidates. Use **multiSelect**, one option per resource with its exact name, plus counts and sizes where known (e.g. `teatec_test_foo_62 (database)` or `412 cache keys under foo-62_`). Every option starts unselected, and active worktrees get "(running)" in the label.

The plan lists the selected items with their actions, and the skipped items with the reason: shared, main checkout, kept, running, ambiguous match, or work at risk. Get approval through `ExitPlanMode` in Claude Code, or an explicit "yes" elsewhere. Only the approved items are deleted.

## 4. Execute (approved items only)

1. **Docker**: run `docker compose -f <file> down -v --rmi local` from the folder when its compose file exists and every resource it would remove was approved. Otherwise remove each approved item with `docker rm -f`, `docker rmi`, `docker volume rm` and `docker network rm`.
2. **Databases**: one `DROP DATABASE` per approved name, by exact name. Take credentials from the DB container's own env inside the container, e.g. `docker exec <db> sh -c 'MYSQL_PWD="$MYSQL_ROOT_PASSWORD" mysql -uroot -e "DROP DATABASE \`<name>\`"'`.
3. **Cache keys**: `redis-cli --scan --pattern '<approved prefix>*' | xargs -r redis-cli del` inside the cache container.
4. **Folder**: `git worktree remove --force <dir>` when git still knows it, otherwise `rm -rf <dir>`. Then run `git worktree prune` in the main checkout.
5. Keep the branch, local and remote. Delete it only on an explicit request.

Never print env-file or credential values. Read single keys with `grep '^KEY=' file | cut -d= -f2-` into a variable.

## 5. Verify and report

Re-run the step 2 queries. Done when the approved items are gone and every skipped item is still present. Report what was removed (with the image size reclaimed), what was skipped and why, and what remains for the user to decide.
