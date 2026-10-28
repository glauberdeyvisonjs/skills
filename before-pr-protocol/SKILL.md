---
name: before-pr-protocol
description: Use when the user invokes /before-pr-protocol or asks to run the current repository's test suites before opening a pull request.
---

# Before PR Protocol

Run the available automated tests for the current local repository, on the current branch, and wait for their results before reporting PR readiness.

## Identify the repository and commands

Read the current repository's AGENTS.md/CLAUDE.md, test documentation, package manifests and test configuration. Required coverage is every automated suite configured there: unit, integration, end-to-end and browser tests, across every language or framework in the repository.

Run only the current repository's suites, on the current branch. Never switch repositories or branches to run tests.

Discover exact commands from maintained project instructions and scripts; do not assume every framework has a configured test suite. Report a missing suite as unavailable, not as passed.

## Check test side effects

Before running commands, inspect test setup, scripts and hooks for database access, migrations, resets, seeding and external-service mutations. For Playwright, check the target URL and web-server setup too.

- Database-backed tests must use a confirmed isolated test database, different from development/shared/production data. An in-memory database or a dedicated disposable test database is suitable; a test-like name alone is not proof.
- Do not run a suite against a dev/shared/production database or service. If isolation cannot be confirmed, skip that suite, explain the blocker and continue with independent safe suites.
- Never run migration, reset, rollback, seed or raw DDL commands without the user's explicit request for that exact command now. This also applies when a test script invokes them indirectly; invoking this skill does not supply that authorization.
- Do not change environment files or database configuration to make a suite runnable. Report missing prerequisites.

## Execute and wait

Run each available, safe suite using its documented command. Use a complete run that exits rather than watch mode. Wait for every started process and record its actual exit status.

Continue independent suites after a failure so the report covers all possible checks. Do not start suites concurrently when they share mutable test resources. If a process stalls or must be interrupted, report it as incomplete, never passed.

Do not install missing dependencies, repair failing tests or change application code as part of this protocol. Report the prerequisite or failure for follow-up.

## Report

Respond in Portuguese with:

| Suíte | Comando | Resultado | Detalhes |
|-------|---------|-----------|----------|

Distinguish passed, failed, unavailable, blocked and interrupted suites. Include test totals when the runner provides them, and concise failure details.

State whether all applicable suites passed or validation remains incomplete. Do not claim a failure is pre-existing without evidence. Report results to the user; do not create a PR, commit, push or publish comments.
