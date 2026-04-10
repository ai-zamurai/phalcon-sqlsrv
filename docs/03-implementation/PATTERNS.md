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
| `public function connect(array $descriptor = null): bool` | Matches `AbstractPdo::connect()` |
| `public function describeColumns(string $table, ?string $schema = null): array` | Matches interface |
| `public function numRows(): int` | Matches `Result\Pdo::numRows()` |

When in doubt, run the `phalcon-compat-check` skill (see `.claude/skills/phalcon-compat-check/`).

## PDO Usage

- Always go through `$this->pdo->prepare()` → `executePrepared()`; never use `PDO::query()` directly in the adapter.
- Keep `PDO::ATTR_ERRMODE = ERRMODE_EXCEPTION` — rely on exceptions, not boolean returns.
- Choose cursor type based on statement shape: `CURSOR_FWDONLY` for `exec`/stored procedures, `CURSOR_SCROLL` otherwise.

## Event Emission

`Sqlsrv::query()` fires `db:beforeQuery` and `db:afterQuery` via Phalcon's `eventsManager` if one is attached. When adding new execution paths (e.g., `execute()`), emit the same events to keep listener contracts stable.

## SQL Generation (Dialect)

- Keep the dialect **pure**: no PDO, no I/O, no event firing.
- Identifiers must be wrapped in square brackets (`[col_name]`) to handle SQL Server reserved words and spaces.
- Use `prepareTable($table, $schema)` for fully-qualified names.
- Throw `Phalcon\Db\Exception` for unrecognised column types — never silently coerce.

## Exception Handling

- Do not swallow PDO exceptions in the adapter; let them propagate. Phalcon's upper layers translate them.
- Logger adapter (`Phalcon\Logger\Adapter\Database`) is the only place that may catch and ignore, and only for non-critical logging failures — currently it does not, which is intentional: a broken log table should surface.

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
