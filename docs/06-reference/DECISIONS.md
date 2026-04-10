---
id: decisions
title: Architecture Decision Records
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [adr, decisions, history]
references:
  - ../02-design/ARCHITECTURE.md
  - ../01-context/CONSTRAINTS.md
changeImpact: high
---

# Architecture Decision Records

ADR-lite log. Each entry captures *what* was decided, *why*, and *what we would reconsider later*.

## ADR-001: Fork to `ai-zamurai/phalcon-sqlsrv`

- **Date**: 2026-04 (approximate, see git history)
- **Status**: Accepted
- **Context**: Upstream `bakaphp/phalcon-sqlsrv` had not been updated for Phalcon 4 API changes (return type hints, method signatures). Internal projects could not upgrade Phalcon without a working adapter.
- **Decision**: Create the `ai-zamurai` fork, keep the `Phalcon\Db\…` / `Phalcon\Logger\…` namespaces, update method signatures to Phalcon 4.
- **Consequences**:
  - Users switching from upstream only change the Composer package name.
  - Maintenance burden for keeping pace with future Phalcon releases sits with `ai-zamurai`.
- **Reconsider if**: Upstream resumes active maintenance and catches up.

## ADR-002: Set `TrustServerCertificate=true` by default

- **Date**: 2026-04
- **Status**: Accepted
- **Context**: Internal corporate SQL Server instances use self-signed or internally-issued TLS certificates. Without trusting the cert, connections fail with opaque errors.
- **Decision**: Hard-code `TrustServerCertificate=true` into the DSN string in `Sqlsrv::connect()`.
- **Consequences**:
  - Connections succeed out of the box in trusted internal networks.
  - **Security trade-off**: the adapter does not validate the server certificate, so it is vulnerable to MITM on untrusted networks. Callers who need strict TLS must not use this adapter as-is.
- **Reconsider if**: A configurable `trustServerCertificate` descriptor key is added to let callers opt in/out.

## ADR-003: Set `LoginTimeout=3` by default

- **Date**: 2026-04
- **Status**: Accepted
- **Context**: Default SQLSRV login timeout is ~15 seconds. Failing-fast is important in our workloads (web requests, job workers); a stuck login blocks everything.
- **Decision**: Hard-code `LoginTimeout=3` in the DSN.
- **Consequences**: Unreachable servers fail within 3 seconds. Intermittent network hiccups may cause more visible errors than the default.
- **Reconsider if**: Slow-WAN deployments complain. Could be made configurable.

## ADR-004: Cursor selection heuristic in `query()`

- **Date**: Inherited from upstream, documented here.
- **Status**: Accepted (known limitation)
- **Context**: SQL Server stored-procedure execution (`exec`) does not play well with scrollable cursors. Regular SELECTs benefit from `CURSOR_SCROLL` because it lets `numRows()` return early.
- **Decision**: If `strpos($sqlStatement, 'exec') !== false`, use `PDO::CURSOR_FWDONLY`. Otherwise `PDO::CURSOR_SCROLL`.
- **Consequences**:
  - **Case-sensitive match**: only lowercase `exec` triggers `CURSOR_FWDONLY`. `EXEC`, `Exec`, and `EXECUTE` — all valid T-SQL — fall through to `CURSOR_SCROLL`. Calling a stored procedure with uppercase `EXEC` can therefore produce unexpected cursor behavior. Callers in this fork's codebase currently rely on lowercase `exec`.
  - **False positives**: a SELECT that contains the lowercase substring `exec` in a string literal still gets `CURSOR_FWDONLY`.
- **Reconsider if**: The case-sensitivity bites a production caller, or upstream Phalcon changes the cursor contract. Any fix should be case-insensitive and tokenise rather than substring-match.

## ADR-005: `Dialect::limit()` appends `ORDER BY 1` when absent

- **Date**: Inherited.
- **Status**: Accepted
- **Context**: SQL Server requires `ORDER BY` whenever `OFFSET/FETCH` is used. Phalcon callers do not always add one.
- **Decision**: If the query has no `ORDER BY`, append `ORDER BY 1` before the `OFFSET/FETCH` clause.
- **Consequences**:
  - Queries run, but the ordering is implementation-defined. Callers who need deterministic paging must provide their own `ORDER BY`.
- **Reconsider if**: We can detect this and surface a warning instead.

## ADR-006: English-first documentation

- **Date**: 2026-04-10
- **Status**: Accepted
- **Context**: The project is public OSS on GitHub/Packagist and attracts non-Japanese users. Internal AI agent instructions (Claude/Copilot/Cursor) also benefit from a single language.
- **Decision**: All files under `docs/`, plus `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, and `.github/copilot-instructions.md`, are written in **English**. GitHub Issues, Pull Requests, and team chat remain in **Japanese** per the maintainer's preference.
- **Consequences**: Contributors from outside Japan can read the spec without translation. Maintainer internal communication stays efficient.
- **Reconsider if**: A Japanese-only contributor base grows significantly and needs translated reference material.
