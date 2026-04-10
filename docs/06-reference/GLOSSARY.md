---
id: glossary
title: Glossary
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [glossary, terms]
references:
  - ../02-design/ARCHITECTURE.md
  - ../02-design/DOMAIN.md
changeImpact: low
---

# Glossary

Terms specific to this package. When a word in documentation has a capitalised meaning here, check this file first.

| Term | Definition |
|------|------------|
| **Adapter** | The class that owns a live PDO connection and implements `Phalcon\Db\Adapter\AdapterInterface`. Here: `Phalcon\Db\Adapter\Pdo\Sqlsrv`. |
| **Dialect** | The stateless SQL string generator. Converts abstract column/table/index definitions into SQL Server syntax. Here: `Phalcon\Db\Dialect\Sqlsrv`. |
| **Result** | A thin wrapper around `PDOStatement` that Phalcon returns from `query()`. Here: `Phalcon\Db\Result\PdoSqlsrv`. |
| **DSN** | PDO Data Source Name. For this adapter: `sqlsrv:server=<host>;database=<db>;LoginTimeout=3;TrustServerCertificate=true`. |
| **Dialect Class** | The fully-qualified class name Phalcon instantiates for the dialect. Auto-resolved from `dialectType` unless overridden via the `dialectClass` descriptor key. |
| **Descriptor** | The array passed to `Sqlsrv::connect()` — holds host, dbname, credentials, and optional overrides. |
| **`sp_columns`** | SQL Server system stored procedure used by `describeColumns()` to introspect a table's columns. |
| **`sp_pkeys`** | SQL Server system stored procedure used by `getPrimaryKey()`. |
| **`OFFSET / FETCH`** | SQL Server 2012+ paging syntax. Replaces the older `TOP N` / ROW_NUMBER workarounds. |
| **`WITH (UPDLOCK)`** | Query hint emitted by `forUpdate()`. Holds an update lock until the transaction ends. |
| **`WITH (NOLOCK)`** | Query hint emitted by `sharedLock()`. Reads without acquiring shared locks (dirty reads possible). |
| **`IDENTITY(1,1)`** | SQL Server auto-increment clause emitted by the dialect for `isAutoIncrement()` columns. |
| **`ext-phalcon`** | The compiled Phalcon PHP extension. Users install it via PECL or from `phalcon.io`. |
| **`ext-pdo_sqlsrv`** | Microsoft's PDO driver for SQL Server. Required on Linux/macOS alongside MS ODBC Driver 17+. |
| **Upstream** | The original project this repo is forked from: `bakaphp/phalcon-sqlsrv`. |
| **`TrustServerCertificate=true`** | DSN option that tells the SQLSRV driver to skip TLS certificate validation. On by default in this fork — see `DECISIONS.md`. |
| **`LoginTimeout=3`** | DSN option capping the initial login wait to 3 seconds. Fails fast on unreachable servers. |
| **ADR** | Architecture Decision Record. Stored as entries in `DECISIONS.md`. |
