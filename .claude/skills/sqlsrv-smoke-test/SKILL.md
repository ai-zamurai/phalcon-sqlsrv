---
name: sqlsrv-smoke-test
description: Run a minimal end-to-end smoke test against a real SQL Server instance using Phalcon\Db\Adapter\Pdo\Sqlsrv. Use after adapter/dialect changes, before releases, or when investigating "my connection works in raw PDO but fails through this package" reports.
---

# SQL Server Smoke Test

Minimal end-to-end verification of the adapter against a live SQL Server. Run this **before every release** and after any change that touches `Phalcon/Db/Adapter/Pdo/Sqlsrv.php` or `Phalcon/Db/Dialect/Sqlsrv.php`.

## Prerequisites

### Runtime

```bash
php -m | grep -E 'pdo_sqlsrv|sqlsrv|phalcon'
```

Expected lines (at minimum):
- `pdo_sqlsrv`
- `phalcon`

On Linux/macOS, this also requires MS ODBC Driver 17+ from Microsoft.

### Connection Target

Ask the user for a reachable SQL Server instance. **Never** commit credentials. Accept them via environment variables:

```bash
export SQLSRV_HOST='test-sql.example.local'
export SQLSRV_DB='scratch'
export SQLSRV_USER='reader'
export SQLSRV_PASS='<paste interactively>'
```

If the user has no test server, stop — do not fabricate one.

## Step 1: Autoload Check

```bash
php -r '
require "vendor/autoload.php";
new ReflectionClass("Phalcon\\Db\\Adapter\\Pdo\\Sqlsrv");
echo "autoload OK" . PHP_EOL;
'
```

If this fails, run `composer dump-autoload` first.

## Step 2: Connect + Select

Run the probe script (inline, do not commit):

```bash
php -r '
require "vendor/autoload.php";

$conn = new \Phalcon\Db\Adapter\Pdo\Sqlsrv([
    "host"     => getenv("SQLSRV_HOST"),
    "dbname"   => getenv("SQLSRV_DB"),
    "username" => getenv("SQLSRV_USER"),
    "password" => getenv("SQLSRV_PASS"),
]);

$row = $conn->fetchOne("SELECT 1 AS n, @@VERSION AS ver");
assert($row["n"] == 1, "SELECT 1 failed");
echo "connect OK: " . substr($row["ver"], 0, 60) . "..." . PHP_EOL;
'
```

Expected: `connect OK: Microsoft SQL Server 20XX ...`.

## Step 3: `describeColumns` Round Trip

Pick any table known to exist on the instance. Ask the user for a safe candidate (e.g., a reference/lookup table). Then:

```bash
php -r '
require "vendor/autoload.php";
$conn = new \Phalcon\Db\Adapter\Pdo\Sqlsrv([
    "host"     => getenv("SQLSRV_HOST"),
    "dbname"   => getenv("SQLSRV_DB"),
    "username" => getenv("SQLSRV_USER"),
    "password" => getenv("SQLSRV_PASS"),
]);

$cols = $conn->describeColumns("'$TARGET_TABLE'");
echo "columns: " . count($cols) . PHP_EOL;
foreach ($cols as $c) {
    echo sprintf("  %-30s %s%s\n", $c->getName(), $c->getType(), $c->isPrimary() ? " PK" : "");
}
'
```

Verify every column type is a `Phalcon\Db\Column::TYPE_*` constant — none should be `null`. If a column is `null`, update the type switch in `Phalcon/Db/Adapter/Pdo/Sqlsrv.php::describeColumns()`.

## Step 4: Paging Query

```bash
php -r '
require "vendor/autoload.php";
$conn = new \Phalcon\Db\Adapter\Pdo\Sqlsrv([...]);

$sql = $conn->getDialect()->limit("SELECT name FROM sys.tables", [0, 5]);
echo "paged SQL: " . $sql . PHP_EOL;
$rows = $conn->fetchAll($sql);
echo "rows: " . count($rows) . PHP_EOL;
'
```

Expected: SQL ends with `ORDER BY 1 OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY` and returns up to 5 rows.

## Step 5: Report

Summarise findings as:

- ✅ **PASS** — all four steps completed, column types all mapped
- ⚠️ **Type gap** — list column names + SQL Server types that fell through to `TYPE_VARCHAR` unexpectedly
- ❌ **FAIL** — record the step that broke, the error message, and the Phalcon/PHP/SQL Server versions

Paste the report into the PR description when the change touches adapter or dialect code.

## Safety

- Never `INSERT` / `UPDATE` / `DELETE` / `DROP` against a shared server during smoke tests.
- Never commit connection strings.
- If the test server is production-adjacent, run only `SELECT` statements.

## References

- `docs/04-quality/TESTING.md` — overall test strategy
- `docs/02-design/DOMAIN.md` — full column type mapping table
- `docs/06-reference/DECISIONS.md` ADR-002/003 — DSN defaults
