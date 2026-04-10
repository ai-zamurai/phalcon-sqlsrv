---
id: claude-code-instructions
title: Claude Code Instructions
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [claude, ai-agent, instructions]
references:
  - docs/MASTER.md
  - docs/05-operations/GIT_WORKFLOW.md
  - docs/06-reference/DECISIONS.md
changeImpact: high
---

# Claude Code Instructions — phalcon-sqlsrv

Read this file first. It is the entry point for every Claude Code session on this repository.

## What This Repo Is

`ai-zamurai/phalcon-sqlsrv` — a Composer library that adds a MS SQL Server PDO adapter to the Phalcon framework (Phalcon 4+). Fork of `bakaphp/phalcon-sqlsrv` with updated method signatures and safer internal-network defaults.

## Start Here

Before doing anything, load context from:

1. [`docs/MASTER.md`](docs/MASTER.md) — documentation navigation hub
2. [`docs/01-context/PROJECT.md`](docs/01-context/PROJECT.md) — what we build and why
3. [`docs/02-design/ARCHITECTURE.md`](docs/02-design/ARCHITECTURE.md) — Adapter / Dialect / Result layering
4. [`docs/06-reference/DECISIONS.md`](docs/06-reference/DECISIONS.md) — accepted trade-offs (TrustServerCertificate, LoginTimeout, etc.)

If a user request conflicts with a decision in `DECISIONS.md`, flag the conflict — do not silently override.

## Tech Stack

| Layer | Technology | Version constraint |
|-------|------------|-------------------|
| Language | PHP | `>= 7.4` |
| Framework extension | Phalcon (`ext-phalcon`) | `>= 4.0` |
| PDO driver | `pdo_sqlsrv` (see [`docs/01-context/CONSTRAINTS.md`](docs/01-context/CONSTRAINTS.md) for OS-specific install notes) | required at runtime |
| Package manager | Composer | PSR-4 autoload |
| Target DB | Microsoft SQL Server | `2012+` (for OFFSET/FETCH) |

No build step. The library is loaded directly by Composer.

## Directory Map

```
Phalcon/
├── Db/Adapter/Pdo/Sqlsrv.php  # main adapter
├── Db/Dialect/Sqlsrv.php      # SQL generation
├── Db/Result/PdoSqlsrv.php    # result wrapper
├── Db/DbListener.php          # debug event listener
└── Logger/Adapter/Database.php
docs/                          # spec-driven documentation (start at MASTER.md)
.claude/skills/                # project-specific Claude Code skills
```

## Key Commands

```bash
# Validate composer metadata
composer validate

# Static syntax check on a file
php -l Phalcon/Db/Adapter/Pdo/Sqlsrv.php

# Install for local dev
composer install
```

There is no test runner configured yet. Smoke tests are manual — see `docs/04-quality/TESTING.md`.

## Critical Rules (Do Not Violate)

These eight rules are mirrored verbatim in [`AGENTS.md`](AGENTS.md), [`.cursorrules`](.cursorrules), and [`.github/copilot-instructions.md`](.github/copilot-instructions.md). Keep all four lists in sync when editing.

1. **Never change namespaces.** Classes stay under `Phalcon\Db\…` and `Phalcon\Logger\…`. Users depend on this for drop-in upgrades from upstream.
2. **Phalcon 4 signatures are mandatory.** Every public method must match `AbstractPdo` / `AdapterInterface` including return type hints.
3. **Dialect is pure.** No PDO, no I/O, no events inside `Phalcon\Db\Dialect\Sqlsrv`.
4. **Wrap SQL Server identifiers in `[brackets]`** inside dialect output — `[column_name]`, `[schema].[table]`.
5. **Throw `Phalcon\Db\Exception`** for unknown column types in the dialect. Never silently coerce.
6. **Do not remove `TrustServerCertificate=true` or `LoginTimeout=3`** without first updating `docs/06-reference/DECISIONS.md` and opening a discussion.
7. **No new Composer dependencies without an ADR entry** in `docs/06-reference/DECISIONS.md`.
8. **English-only documentation.** All files under `docs/`, plus `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, and `.github/copilot-instructions.md`, are written in English. GitHub Issues and Pull Requests remain in Japanese.
9. **No secrets in commits.** SQL Server credentials, connection strings to real servers, and `.env` files never enter git.

## Git Workflow (Summary)

Full details in `docs/05-operations/GIT_WORKFLOW.md`. Quick summary:

- Branch from `develop`, never from `master`.
- Branch name: `<type>/#<issue>-<slug>` (types: `feature`, `fix`, `docs`, `chore`).
- Commit header: `<type>: #<issue> <summary>` (Conventional Commits).
- PR target: `develop`. Never open PRs against `master`.
- Create an issue before starting work. Issue text in Japanese is fine.

## Releases

Tagged from `master` after merging `develop`. Packagist auto-picks up the tag. See `docs/05-operations/DEPLOYMENT.md` and the `phalcon-release` skill.

## Available Skills (`.claude/skills/`)

- **`phalcon-release`** — semver bump, git tag, push, verify Packagist pickup.
- **`phalcon-compat-check`** — diff method signatures against current Phalcon version.
- **`sqlsrv-smoke-test`** — minimal connect + `SELECT 1` + `describeColumns()` round trip.

Invoke via the Skill tool when the situation matches.

## When You Are Unsure

- Prefer reading the existing source in `Phalcon/` over guessing.
- Ask the user before touching anything under `docs/06-reference/DECISIONS.md` — those are committed trade-offs.
- Never invent dependencies. If something is missing, report it; do not add a Composer package to "fix" the report.
