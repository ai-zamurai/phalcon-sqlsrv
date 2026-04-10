---
name: phalcon-compat-check
description: Verify that Phalcon\Db\Adapter\Pdo\Sqlsrv and related classes are compatible with the currently installed Phalcon version. Use when bumping ext-phalcon, upgrading PHP, investigating "signature mismatch" fatal errors, or before releasing.
---

# Phalcon Compatibility Check

Detects breakage between this package and the installed Phalcon extension. Phalcon changes parent signatures across major versions (notably 3 → 4 → 5), so overriding methods must be re-audited whenever users upgrade.

## Step 1: Identify the Installed Phalcon Version

```bash
php -r 'echo phpversion("phalcon") . PHP_EOL;'
```

If the output is empty, `ext-phalcon` is not installed. Stop and tell the user to install it.

## Step 2: Enumerate Overridden Methods

The classes that override Phalcon parents:

| File | Parent | Methods to check |
|------|--------|------------------|
| `Phalcon/Db/Adapter/Pdo/Sqlsrv.php` | `Phalcon\Db\Adapter\Pdo\AbstractPdo` | `connect`, `describeColumns`, `query`, `getDsnDefaults` |
| `Phalcon/Db/Dialect/Sqlsrv.php` | `Phalcon\Db\Dialect` | `limit`, `forUpdate`, `sharedLock`, `getColumnDefinition`, `addColumn`, `modifyColumn`, `dropColumn`, `addIndex`, `dropIndex`, `addPrimaryKey`, `dropPrimaryKey`, `addForeignKey`, `dropForeignKey`, `createTable`, `dropTable`, `createView`, `dropView`, `tableExists`, `viewExists`, `describeColumns`, `listTables`, `listViews`, `describeIndexes`, `describeReferences`, `tableOptions`, `getPrimaryKey` |
| `Phalcon/Db/Result/PdoSqlsrv.php` | `Phalcon\Db\Result\Pdo` | `numRows` |

## Step 3: Diff Each Override Against the Parent

For each method, run:

```bash
php -r '
$rc = new ReflectionClass("Phalcon\\Db\\Adapter\\Pdo\\AbstractPdo");
$m = $rc->getMethod("connect");
echo $m->__toString() . PHP_EOL;
'
```

Compare the output to the override in `Phalcon/Db/Adapter/Pdo/Sqlsrv.php`. Check:

- Parameter list (type, nullability, default, count)
- Return type hint (present and matching)
- `static` / `final` modifiers

Repeat for every method in the table above. Mismatches cause fatal errors at class load time in Phalcon 4+.

## Step 4: Syntax Sanity Check

```bash
php -l Phalcon/Db/Adapter/Pdo/Sqlsrv.php
php -l Phalcon/Db/Dialect/Sqlsrv.php
php -l Phalcon/Db/Result/PdoSqlsrv.php
```

All three must report `No syntax errors detected`.

## Step 5: Load Test

```bash
php -r '
require "vendor/autoload.php";
new ReflectionClass("Phalcon\\Db\\Adapter\\Pdo\\Sqlsrv");
new ReflectionClass("Phalcon\\Db\\Dialect\\Sqlsrv");
new ReflectionClass("Phalcon\\Db\\Result\\PdoSqlsrv");
echo "OK" . PHP_EOL;
'
```

If any class fails to load with "must be compatible with" or "must return", that method is out of sync.

## Step 6: Report

Summarise findings as:

- ✅ **Compatible** — all signatures match, load test passes
- ⚠️ **Drift** — list each mismatched method with current and expected signature
- ❌ **Broken** — load test failed; the package will not boot against the installed Phalcon

If drift is detected, do not auto-fix — flag it to the user and, if asked, open a `fix/#N-phalcon-<version>-compat` branch per `docs/05-operations/GIT_WORKFLOW.md`.

## References

- `docs/03-implementation/PATTERNS.md` — method signature rules
- `docs/06-reference/DECISIONS.md` ADR-001 — Phalcon 4 fork rationale
