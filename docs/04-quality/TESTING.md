---
id: testing
title: Testing Strategy
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [testing, quality, codeception]
references:
  - ../01-context/CONSTRAINTS.md
  - ../03-implementation/PATTERNS.md
changeImpact: medium
---

# Testing Strategy

Strategy: **smoke test what the compiler cannot catch**.

## Current State

- `require-dev` includes `codeception/verify`.
- No unit test suite is committed yet.
- The Phalcon 3 → 4 migration relied on manual signature review.

Adding tests is a welcome contribution. Start with the layers below.

## Test Layers

### 1. Static / Signature Checks (free, run locally)

- `php -l Phalcon/Db/Adapter/Pdo/Sqlsrv.php` on every modified file.
- `composer validate` on `composer.json`.
- Optional but recommended: PHPStan level 5 targeting `Phalcon/` (not wired in CI yet).

### 2. Phalcon Compatibility Check

Use the `phalcon-compat-check` skill (`.claude/skills/phalcon-compat-check/`) after any Phalcon version bump:

- Diff public method signatures against `Phalcon\Db\Adapter\Pdo\AbstractPdo`.
- Verify return type hints match.
- Confirm `Phalcon\Db\Result\PdoSqlsrv` still extends `Phalcon\Db\Result\Pdo`.

### 3. SQL Server Smoke Test (manual)

Use the `sqlsrv-smoke-test` skill. Minimum pass criteria:

```php
$conn = new \Phalcon\Db\Adapter\Pdo\Sqlsrv([
    'host'     => getenv('SQLSRV_HOST'),
    'dbname'   => getenv('SQLSRV_DB'),
    'username' => getenv('SQLSRV_USER'),
    'password' => getenv('SQLSRV_PASS'),
]);

$row = $conn->fetchOne('SELECT 1 AS n');
assert($row['n'] === 1);

$cols = $conn->describeColumns('your_existing_table');
assert(count($cols) > 0);
```

Run against SQL Server 2012+ (required for `OFFSET/FETCH`).

### 4. Dialect Unit Tests (desired, not yet implemented)

Target: `Phalcon\Db\Dialect\Sqlsrv`. It is pure (no PDO), so tests are cheap:

- `limit('SELECT * FROM t', 10)` returns `SELECT * FROM t ORDER BY 1 OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY`.
- `getColumnDefinition(new Column('x', ['type' => Column::TYPE_VARCHAR, 'size' => 50]))` returns `NVARCHAR(50)`.
- Unknown type throws `Phalcon\Db\Exception`.
- `createTable()` emits bracketed identifiers and `IDENTITY(1,1)` for auto-increment columns.

If you add this suite, wire it via `composer test` and document the runner here.

## Environments

| Env | PHP | Phalcon | SQL Server | Notes |
|-----|-----|---------|------------|-------|
| Dev (macOS) | 7.4 / 8.x | 4.x | 2019 (Docker) | `mcr.microsoft.com/mssql/server` |
| Dev (Linux) | 7.4 / 8.x | 4.x | 2019 (Docker) | MS ODBC Driver 17+ |
| CI | not configured | — | — | See `OUT_OF_SCOPE` in `PROJECT.md` |

## What NOT to Test

- Phalcon core behavior (not ours to own).
- SQL Server engine behavior (use existing MS test instances).
- ODBC driver bugs — report upstream instead.
