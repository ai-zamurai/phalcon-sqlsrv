---
id: patterns
title: Implementation Patterns
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [patterns, coding, psr-4]
references:
  - ../02-design/ARCHITECTURE.md
  - ../01-context/CONSTRAINTS.md
changeImpact: medium
---

# Implementation Patterns

Conventions every contributor (human or AI) must follow when editing this package.

## Directory Layout

```
Phalcon/
├── Db/
│   ├── Adapter/Pdo/Sqlsrv.php     # Adapter
│   ├── Dialect/Sqlsrv.php         # Dialect
│   ├── Result/PdoSqlsrv.php       # Result
│   └── DbListener.php             # Debug listener
└── Logger/
    └── Adapter/Database.php       # Logger adapter
```

PSR-4 roots (see `composer.json`):
- `Phalcon\Db\`     → `Phalcon/Db/`
- `Phalcon\Logger\` → `Phalcon/Logger/`

Never rename namespaces. Users of this fork expect the same class paths as upstream.

## Method Signature Rules (Phalcon 4 compatibility)

All overridden methods **must** match the parent interface including return type hints. Phalcon 4 introduced strict signatures; missing or mismatched hints cause fatal errors.

| Good | Why |
|------|-----|
| `public function connect(?array $descriptor = null): bool` | Matches `AbstractPdo::connect()` |
| `public function describeColumns(string $table, ?string $schema = null): array` | Matches interface |
| `public function numRows(): int` | Matches `Result\Pdo::numRows()` |

> **PHP version note**: the current source still uses the implicit-nullable form `array $descriptor = null` which PHP 8.4+ deprecates. When bumping the minimum PHP version, migrate to the explicit `?array` form shown above. Track in a follow-up issue before bumping.

When in doubt, run the `phalcon-compat-check` skill (see `.claude/skills/phalcon-compat-check/`).

## PDO Usage

- Always go through `$this->pdo->prepare()`. If `$bindParams` is an array, call `executePrepared($statement, $bindParams, $bindTypes)`; otherwise fall back to `$statement->execute()` directly. Both branches exist in `Sqlsrv::query()` by design — do not "clean up" the fallback.
- Never use `PDO::query()` directly in the adapter.
- Keep `PDO::ATTR_ERRMODE = ERRMODE_EXCEPTION` — rely on exceptions, not boolean returns.
- Choose cursor type based on statement shape: `CURSOR_FWDONLY` when the SQL contains the lowercase substring `exec`, `CURSOR_SCROLL` otherwise. The match is case-sensitive — see ADR-004.

## Event Emission

`Sqlsrv::query()` fires `db:beforeQuery` and `db:afterQuery` via Phalcon's `eventsManager` if one is attached. When adding new execution paths (e.g., `execute()`), emit the same events to keep listener contracts stable.

## SQL Generation (Dialect)

- Keep the dialect **pure**: no PDO, no I/O, no event firing.
- Identifiers must be wrapped in square brackets (`[col_name]`) to handle SQL Server reserved words and spaces.
- Use `prepareTable($table, $schema)` for fully-qualified names.
- Throw `Phalcon\Db\Exception` for unrecognised column types — never silently coerce.

## Exception Handling

- Do not swallow PDO exceptions in the adapter; let them propagate. Phalcon's upper layers translate them.
- The logger adapter (`Phalcon\Logger\Adapter\Database`) also lets exceptions propagate — a broken log table must surface loudly, not disappear.

## Code Style

- PHP tags: always `<?php`, never short tags.
- Indent: 4 spaces, no tabs.
- Docblocks: keep existing `@param` / `@return` lines when editing methods; AI agents must not strip them even if redundant with type hints.
- Comments: English only (see documentation language policy in `CLAUDE.md`).

## Adding a New Dialect Method

1. Add the method to `Dialect/Sqlsrv.php` with the signature from `Phalcon\Db\Dialect`.
2. Return a string. No side effects.
3. If the adapter needs to call it, add a thin wrapper in `Adapter/Pdo/Sqlsrv.php`.
4. Update `DOMAIN.md` if the change affects column types or SQL shapes.
5. Log the decision in `06-reference/DECISIONS.md` if it changes observable behavior.
