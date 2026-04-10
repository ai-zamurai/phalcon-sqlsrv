---
id: architecture
title: Architecture
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [architecture, design, phalcon]
references:
  - ../01-context/PROJECT.md
  - ../01-context/CONSTRAINTS.md
  - DOMAIN.md
changeImpact: high
---

# Architecture

## Three-Layer Adapter Pattern

The package follows Phalcon's standard database adapter shape: **Adapter → Dialect → Result**. Each layer has a single concern.

```
┌─────────────────────────────────────────────────────────┐
│  Application (Phalcon DI)                               │
│    $di->set('db', fn() => new Sqlsrv([...]));           │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│  Phalcon\Db\Adapter\Pdo\Sqlsrv      (Adapter)           │
│    - connect()         build DSN, open PDO              │
│    - query()           execute + wrap result            │
│    - describeColumns() introspect schema                │
│    extends AbstractPdo, implements AdapterInterface     │
└──────────────────┬────────────────────┬─────────────────┘
                   │                    │
                   ▼                    ▼
┌──────────────────────────┐ ┌──────────────────────────┐
│ Phalcon\Db\Dialect\      │ │ Phalcon\Db\Result\       │
│   Sqlsrv   (Dialect)     │ │   PdoSqlsrv  (Result)    │
│  - limit() OFFSET/FETCH  │ │  - numRows() override    │
│  - createTable()         │ │  extends Result\Pdo      │
│  - describeColumns()     │ └──────────────────────────┘
│  extends Dialect         │
└──────────────────────────┘
```

## Component Responsibilities

| Component | File | Responsibility |
|-----------|------|----------------|
| Adapter | `Phalcon/Db/Adapter/Pdo/Sqlsrv.php` | PDO lifecycle, query execution, event firing, column introspection |
| Dialect | `Phalcon/Db/Dialect/Sqlsrv.php` | Pure SQL string generation for SQL Server syntax |
| Result | `Phalcon/Db/Result/PdoSqlsrv.php` | Result set iteration, row count correction |
| Listener | `Phalcon/Db/DbListener.php` | Optional `beforeQuery` debug hook |
| Logger | `Phalcon/Logger/Adapter/Database.php` | Write log rows into a SQL Server table through the adapter |

## Key Design Choices

- **Dialect is stateless**: it takes strings/columns in, returns SQL out. No PDO access inside the dialect.
- **Adapter owns PDO**: all statement preparation, execution, and cursor selection lives in `Sqlsrv::query()`.
- **Cursor selection**: if the SQL contains `exec` (stored procedure call), use `PDO::CURSOR_FWDONLY`; otherwise `PDO::CURSOR_SCROLL`. This compromise keeps `numRows()` usable for SELECT while allowing stored procs.
- **Type-mapping lives in the Adapter, not the Dialect**. `describeColumns()` walks raw `sp_columns` rows and emits `Phalcon\Db\Column` objects — see `DOMAIN.md` for the mapping table.

## Interaction with Phalcon DI

Registration is standard Phalcon:

```php
$di->set('db', function () use ($config) {
    return new \Phalcon\Db\Adapter\Pdo\Sqlsrv([
        'host'     => $config->database->host,
        'dbname'   => $config->database->name,
        'username' => $config->database->username,
        'password' => $config->database->password,
    ]);
});
```

No extra service provider is required. The class is picked up via Composer PSR-4.

## Logger Adapter (Optional)

`Phalcon\Logger\Adapter\Database` reuses the SQL Server adapter to persist logs. It requires an `options` array with `db` (the adapter instance) and `table` (target table). See `Phalcon/Logger/Adapter/Database.php` for the row schema (`LogType`, `LogProcess`, `LogContent`, `LogUser`, `LogDate`, `LogIP`, `LogBrowser`).
