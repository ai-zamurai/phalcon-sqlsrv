# AGENTS.md — phalcon-sqlsrv

Tool-agnostic instructions for any AI agent (Claude Code, GitHub Copilot, Cursor, Aider, Devin, etc.) working in this repository.

## Project Summary

`ai-zamurai/phalcon-sqlsrv` is a Composer library that provides a Microsoft SQL Server PDO adapter, SQL dialect, and result wrapper for the Phalcon PHP framework (version 4 and later). It is a maintained fork of `bakaphp/phalcon-sqlsrv`.

## Entry Points

- `docs/MASTER.md` — documentation hub. **Read first.**
- `docs/01-context/PROJECT.md` — vision and scope.
- `docs/02-design/ARCHITECTURE.md` — component layering.
- `docs/06-reference/DECISIONS.md` — hard constraints and trade-offs.

## Source Layout

```
Phalcon/
├── Db/Adapter/Pdo/Sqlsrv.php   — PDO adapter
├── Db/Dialect/Sqlsrv.php       — SQL generator (pure, no I/O)
├── Db/Result/PdoSqlsrv.php     — result set wrapper
├── Db/DbListener.php           — debug event listener
└── Logger/Adapter/Database.php — DB-backed logger adapter
composer.json                   — PHP >= 7.4, ext-phalcon >= 4.0
```

## Tech Stack Contract

- Language: PHP 7.4+
- Framework: Phalcon 4+ (`ext-phalcon`)
- PDO driver: `pdo_sqlsrv` or `sqlsrv` (MS ODBC Driver 17+)
- Target DB: SQL Server 2012+
- Autoload: PSR-4 (`Phalcon\Db\` → `Phalcon/Db/`, `Phalcon\Logger\` → `Phalcon/Logger/`)

## Non-Negotiable Rules

1. **Namespaces are frozen.** Keep `Phalcon\Db\…` and `Phalcon\Logger\…`.
2. **Method signatures must match Phalcon 4 parents** including return type hints.
3. **Dialect is pure.** No PDO, no filesystem, no events in `Phalcon\Db\Dialect\Sqlsrv`.
4. **DSN defaults `TrustServerCertificate=true` and `LoginTimeout=3` are intentional** — see `docs/06-reference/DECISIONS.md` ADR-002 and ADR-003.
5. **Documentation language: English.** All files under `docs/` and every AI agent config file are English. GitHub Issues and PRs stay Japanese.
6. **No secrets in git.** Ever.

## Coding Conventions

- 4-space indentation, no tabs.
- Opening tag always `<?php`, never short tags.
- Preserve existing docblocks even when redundant with type hints.
- English comments only.
- Wrap SQL Server identifiers in square brackets: `[column_name]`.
- Throw `Phalcon\Db\Exception` for unknown dialect types — do not silently coerce.

## Git Workflow

- Create an issue first (Japanese text allowed).
- Branch from `develop`: `<type>/#<issue>-<slug>` (`feature` / `fix` / `docs` / `chore`).
- Conventional Commits with issue number: `feat: #42 ...`.
- Open PRs against `develop`. Never target `master` directly.
- Full details: `docs/05-operations/GIT_WORKFLOW.md`.

## Release

Merge `develop` → `master`, tag `vMAJOR.MINOR.PATCH`, push tags. Packagist updates via webhook. Full steps: `docs/05-operations/DEPLOYMENT.md`.

## Verification Checklist (before opening a PR)

- [ ] `composer validate` passes
- [ ] `php -l` on every modified PHP file
- [ ] Manual smoke test described in `docs/04-quality/TESTING.md` if the change touches the adapter or dialect
- [ ] `docs/06-reference/DECISIONS.md` updated if observable behavior changes
- [ ] Frontmatter `version` bumped on any edited doc
- [ ] Issue number is in branch name, commit header, and PR body

## When in Doubt

Read the source in `Phalcon/` before writing code. When a request conflicts with `docs/06-reference/DECISIONS.md`, stop and ask the maintainer rather than silently overriding.
