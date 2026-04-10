# GitHub Copilot Instructions — phalcon-sqlsrv

Use these instructions for all code suggestions in this repository. They take precedence over general knowledge when they conflict.

## Repository Purpose

`ai-zamurai/phalcon-sqlsrv` is a Composer library that ships a Microsoft SQL Server PDO adapter for the Phalcon framework (Phalcon 4+). It is a fork of `bakaphp/phalcon-sqlsrv`. Full context: `docs/MASTER.md`.

## Stack Constraints

- PHP `>= 7.4`
- `ext-phalcon >= 4.0`
- `ext-pdo`, plus `pdo_sqlsrv` / MS ODBC Driver 17+ at runtime
- Target SQL Server 2012+ (required for `OFFSET/FETCH` paging)
- PSR-4 autoload via Composer. No build step.

## File Layout

```
Phalcon/
├── Db/Adapter/Pdo/Sqlsrv.php   — PDO adapter
├── Db/Dialect/Sqlsrv.php       — SQL dialect (stateless)
├── Db/Result/PdoSqlsrv.php     — result wrapper
├── Db/DbListener.php           — debug event listener
└── Logger/Adapter/Database.php — DB logger adapter
```

## Code Generation Rules

1. **Never rename namespaces.** Classes live under `Phalcon\Db\…` or `Phalcon\Logger\…`.
2. **Match Phalcon 4 method signatures exactly**, including return type hints:
   - `public function connect(array $descriptor = null): bool`
   - `public function describeColumns(string $table, ?string $schema = null): array`
   - `public function numRows(): int`
3. **Dialect must remain pure.** Do not introduce PDO calls, file I/O, or event firing inside `Phalcon\Db\Dialect\Sqlsrv`. It only returns SQL strings.
4. **Wrap identifiers in square brackets**: `[column_name]`, `[schema].[table]`.
5. **Throw `Phalcon\Db\Exception`** for unrecognised column types in the dialect — do not silently fall back.
6. **Preserve DSN defaults**: `LoginTimeout=3` and `TrustServerCertificate=true`. These are documented decisions.
7. **Never add new Composer dependencies** without checking `docs/06-reference/DECISIONS.md` first.

## Code Style

- 4-space indentation, no tabs.
- Always open with `<?php` (no short tags).
- Keep existing `@param` / `@return` docblocks even when redundant with PHP type hints.
- English comments only.
- No trailing whitespace.

## Documentation Changes

All files under `docs/`, `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, and this file must be written in English. GitHub Issues and Pull Requests remain in Japanese per the maintainer's global policy.

Every doc file carries YAML frontmatter with `id`, `title`, `version`, `status`, `created`, `updated`, `owner`, `phase`, `tags`, `references`, `changeImpact`. When editing a doc, bump `version` (semver) and update `updated`.

## Git & PR Conventions

- Branch from `develop`: `<type>/#<issue>-<slug>` where type is `feature`, `fix`, `docs`, or `chore`.
- Commit header: `<type>: #<issue> <short summary>` (Conventional Commits).
- PR target branch: `develop`. Never `master`.
- Full workflow: `docs/05-operations/GIT_WORKFLOW.md`.

## What Not to Suggest

- Changes that rename classes, drop namespaces, or break PSR-4.
- Type coercion in the dialect's default branch instead of `throw Exception`.
- Removing or weakening the DSN defaults.
- Adding a test framework other than `codeception/verify` without an ADR entry.
- Refactors that mix SQL generation and PDO execution in one class.

## When Unsure

Read the existing source in `Phalcon/` and the relevant doc under `docs/` before generating code. If a user request contradicts `docs/06-reference/DECISIONS.md`, surface the conflict.
