---
id: project-overview
title: Project Overview
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [project, vision, scope]
references:
  - ../MASTER.md
  - CONSTRAINTS.md
changeImpact: medium
---

# Project Overview

## Vision

Provide a production-ready **MS SQL Server PDO adapter** for the Phalcon framework so PHP applications running on Phalcon 4+ can use SQL Server as a first-class database — with the same API surface as Phalcon's built-in MySQL/PostgreSQL adapters.

## Why This Package Exists

Phalcon does not ship an official SQL Server adapter. This package is a community fork lineage (ToNict → bakaphp → ai-zamurai) that keeps the adapter working across major Phalcon releases. The `ai-zamurai` fork focuses on:

- Phalcon 4 compatibility (method signatures, return type hints)
- Safe defaults for enterprise SQL Server environments (`TrustServerCertificate=true`, `LoginTimeout=3`)
- Minimal drift from upstream API expectations

## Functional Requirements

- Connect to MS SQL Server via PDO (`sqlsrv:` DSN).
- Implement `Phalcon\Db\Adapter\AdapterInterface` through `AbstractPdo`.
- Translate Phalcon column types to SQL Server native types (INT, NVARCHAR, DATETIME2, etc.).
- Support SQL Server paging via `OFFSET ... FETCH NEXT ... ROWS ONLY`.
- Expose metadata queries (describe columns / indexes / foreign keys) via `INFORMATION_SCHEMA` and `sys.*`.
- Provide a `Phalcon\Logger\Adapter\Database` implementation that writes log rows through this adapter.

## Non-Functional Requirements

- Zero runtime dependencies beyond `ext-pdo` and `ext-phalcon >= 4.0`.
- PSR-4 autoloadable via Composer.
- Library must remain drop-in for users migrating from upstream `bakaphp/phalcon-sqlsrv`.

## In Scope

- PDO adapter (`Phalcon\Db\Adapter\Pdo\Sqlsrv`)
- SQL dialect generator (`Phalcon\Db\Dialect\Sqlsrv`)
- Result set wrapper (`Phalcon\Db\Result\PdoSqlsrv`)
- Debug event listener (`Phalcon\Db\DbListener`)
- Database logger adapter (`Phalcon\Logger\Adapter\Database`)

## Out of Scope

- ORM / Model layer (Phalcon core already provides it)
- Migration tooling
- Multi-server replication handling
- SQL Server 2005/2008 legacy support (we target 2012+ because of `OFFSET/FETCH`)

## Success Criteria

- `composer require ai-zamurai/phalcon-sqlsrv` installs cleanly on PHP 7.4–8.x.
- A minimal integration snippet connects to SQL Server and runs `SELECT 1` with no custom config.
- `describeColumns()` returns correct Phalcon column types for the full SQL Server type matrix documented in `02-design/DOMAIN.md`.
