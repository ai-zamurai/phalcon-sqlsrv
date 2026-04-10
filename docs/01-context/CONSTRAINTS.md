---
id: constraints
title: Constraints
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [constraints, runtime, compatibility]
references:
  - PROJECT.md
  - ../06-reference/DECISIONS.md
changeImpact: high
---

# Constraints

Hard boundaries that every change must respect. Breaking any of these requires a `DECISIONS.md` entry.

## Runtime

| Item | Constraint | Source |
|------|------------|--------|
| PHP | `>= 7.4` | `composer.json` |
| Phalcon | `ext-phalcon >= 4.0` | `composer.json` |
| PDO extension | `ext-pdo` | `composer.json` |
| PDO SQLSRV driver | `pdo_sqlsrv` (Windows) or `sqlsrv` via MS ODBC Driver 17+ (Linux/macOS) | implicit — tested via `sqlsrv:` DSN |
| SQL Server | `2012+` | `Dialect::limit()` uses `OFFSET/FETCH NEXT` |

## API Compatibility

- Must extend `Phalcon\Db\Adapter\Pdo\AbstractPdo` and implement `Phalcon\Db\Adapter\AdapterInterface`.
- Method signatures must match the Phalcon 4 interface including return type hints (`: bool`, `: array`, `: string`).
- `Phalcon\Db\Result\PdoSqlsrv` must extend `Phalcon\Db\Result\Pdo`.
- Dialect class must extend `Phalcon\Db\Dialect`.

## Upstream Compatibility

- The package is a fork of `bakaphp/phalcon-sqlsrv`. Existing users porting from upstream should only need to change the Composer package name and autoload path.
- Keep namespaces `Phalcon\Db\…` and `Phalcon\Logger\…` (do not rename to `AiZamurai\…`).

## Connection Defaults

These defaults are baked into `Sqlsrv::connect()` and cannot be disabled without a breaking change:

- `LoginTimeout=3` — fail fast on unreachable SQL Server instances.
- `TrustServerCertificate=true` — required for internal corporate TLS setups.
- `PDO::ATTR_ERRMODE = ERRMODE_EXCEPTION`
- `PDO::ATTR_STRINGIFY_FETCHES = true`

See `06-reference/DECISIONS.md` for rationale.

## Coding Constraints

- PSR-4 autoload only (`Phalcon\Db\` → `Phalcon/Db/`, `Phalcon\Logger\` → `Phalcon/Logger/`).
- No new runtime dependencies in `composer.json` without an ADR entry.
- Test-only dependencies may be added to `require-dev`.

## License

MIT (inherited from upstream). All contributions must be MIT-compatible.
