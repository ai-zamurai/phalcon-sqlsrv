---
id: domain
title: Domain Model
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [domain, types, column-mapping]
references:
  - ARCHITECTURE.md
  - ../06-reference/GLOSSARY.md
changeImpact: medium
---

# Domain Model

The "domain" of this package is **SQL Server schema introspection and SQL generation** expressed in terms that Phalcon's abstract `Db` layer understands.

## Core Concepts

| Concept | Represented by | Where |
|---------|----------------|-------|
| Connection | `Phalcon\Db\Adapter\Pdo\Sqlsrv` | Adapter |
| Column | `Phalcon\Db\Column` | emitted by `describeColumns()` |
| Result set | `Phalcon\Db\Result\PdoSqlsrv` | Result wrapper |
| SQL expression | `string` returned from dialect | Dialect |
| Primary key / Index / FK | `IndexInterface` / `ReferenceInterface` | passed into dialect methods |

## Column Type Mapping (SQL Server → Phalcon)

`Sqlsrv::describeColumns()` walks `EXEC sp_columns` output and maps each `TYPE_NAME` to a `Phalcon\Db\Column` type constant.

| SQL Server type | Phalcon `Column::TYPE_*` | Bind type | Notes |
|-----------------|--------------------------|-----------|-------|
| `int identity`, `tinyint identity`, `smallint identity` | `TYPE_INTEGER` | `BIND_PARAM_INT` | `autoIncrement=true` |
| `bigint` | `TYPE_BIGINTEGER` | `BIND_PARAM_INT` | |
| `int`, `tinyint`, `smallint` | `TYPE_INTEGER` | `BIND_PARAM_INT` | |
| `decimal`, `money`, `smallmoney` | `TYPE_DECIMAL` | `BIND_PARAM_DECIMAL` | |
| `numeric` | `TYPE_DOUBLE` | `BIND_PARAM_DECIMAL` | |
| `float` | `TYPE_FLOAT` | `BIND_PARAM_DECIMAL` | |
| `bit` | `TYPE_BOOLEAN` | `BIND_PARAM_BOOL` | |
| `date` | `TYPE_DATE` | `BIND_PARAM_STR` | |
| `datetime`, `datetime2`, `smalldatetime` | `TYPE_DATETIME` | `BIND_PARAM_STR` | |
| `timestamp` | `TYPE_TIMESTAMP` | `BIND_PARAM_STR` | SQL Server `rowversion`, not a clock |
| `char`, `nchar` | `TYPE_CHAR` | `BIND_PARAM_STR` | |
| `varchar`, `nvarchar` | `TYPE_VARCHAR` | `BIND_PARAM_STR` | Dialect emits `NVARCHAR` on create |
| `text`, `ntext` | `TYPE_TEXT` | `BIND_PARAM_STR` | |
| `varbinary` | `TYPE_BLOB` | `BIND_PARAM_STR` | |
| anything else | `TYPE_VARCHAR` | `BIND_PARAM_STR` | Safe fallback |

## SQL Generation Rules (Phalcon → SQL Server)

Selected mappings from `Dialect\Sqlsrv::getColumnDefinition()`:

| Phalcon `Column::TYPE_*` | Emitted SQL |
|--------------------------|-------------|
| `TYPE_INTEGER` | `INT` |
| `TYPE_BIGINTEGER` | `BIGINT` |
| `TYPE_VARCHAR` | `NVARCHAR(size)` |
| `TYPE_CHAR` | `CHAR(size)` |
| `TYPE_TEXT` | `NTEXT` |
| `TYPE_DECIMAL` | `DECIMAL(size, scale)` |
| `TYPE_DOUBLE` | `NUMERIC(size[, scale])` |
| `TYPE_FLOAT` | `FLOAT[(size)]` |
| `TYPE_DATE` | `DATE` |
| `TYPE_DATETIME` | `DATETIME` |
| `TYPE_TIMESTAMP` | `TIMESTAMP` |
| `TYPE_BOOLEAN` | `BIT` |
| `TYPE_BLOB` / medium / long | `VARBINARY(MAX)` |
| `TYPE_TINYBLOB` | `VARBINARY(255)` |
| unknown | throws `Phalcon\Db\Exception` |

## Paging Contract

`Dialect::limit($sql, $number)` produces:

```sql
<sql> ORDER BY 1 OFFSET <offset> ROWS FETCH NEXT <number> ROWS ONLY
```

If the query has no `ORDER BY`, `ORDER BY 1` is appended because SQL Server **requires** an `ORDER BY` when `OFFSET/FETCH` is used.

## Locking Hints

- `forUpdate($sql)` → appends `WITH (UPDLOCK)`
- `sharedLock($sql)` → appends `WITH (NOLOCK)` (note: NOLOCK ≠ true shared read; accepted by this fork as the historical mapping)
