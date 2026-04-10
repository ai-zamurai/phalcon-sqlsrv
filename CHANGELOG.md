# Changelog

All notable changes to `ai-zamurai/phalcon-sqlsrv` are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-04-10

First official release of the `ai-zamurai/phalcon-sqlsrv` fork. As a project-policy choice — not a SemVer requirement — versioning restarts from `1.0.0` because the Packagist package name is distinct from the upstream `bakaphp/phalcon-sqlsrv` and the fork's defining guarantee is Phalcon 4 compatibility. Upstream `bakaphp/phalcon-sqlsrv` last tagged `2.0.1` (2023-08-01); this fork diverges from that tag. Historical entries prior to `1.0.0` are not re-listed here — refer to the upstream repository for that history.

### Added

- Spec-driven documentation framework under `docs/`, organized following [feel-flow/ai-spec-driven-development](https://github.com/feel-flow/ai-spec-driven-development). Entry point: `docs/MASTER.md`. (#3)
- Multi-AI tool instruction files kept in sync: `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.github/copilot-instructions.md`. (#3)
- Project-specific Claude Code skills in `.claude/skills/`: `phalcon-release`, `phalcon-compat-check`, `sqlsrv-smoke-test`. (#3)
- Git workflow documentation: `docs/05-operations/GIT_WORKFLOW.md`. (#3)
- Expanded `README.md` with component overview, quick start, and project structure. (#1)

### Changed

- Package renamed from `bakaphp/phalcon-sqlsrv` to `ai-zamurai/phalcon-sqlsrv` on Packagist.
- DSN defaults now include `TrustServerCertificate=true` and `LoginTimeout=3`, tuned for internal-network SQL Server deployments. See `docs/06-reference/DECISIONS.md` for the trade-off rationale.
- Public method signatures across `Phalcon\Db\Adapter\Pdo\Sqlsrv`, `Phalcon\Db\Dialect\Sqlsrv`, and `Phalcon\Db\Result\PdoSqlsrv` aligned with Phalcon 4's `AbstractPdo` / `AdapterInterface`, including return type hints (e.g. `connect(): bool`). This is the fork's reason for existing and is **breaking relative to Phalcon 3** users of upstream.

### Fixed

- `Phalcon\Db\Adapter\Pdo\Sqlsrv::query()` cursor-type heuristic is now case-insensitive and word-boundary matched. `EXEC`, `Execute`, and other case variants correctly select `CURSOR_FWDONLY`, while identifiers like `executor_name` no longer false-match. (#5)
- `Phalcon\Db\Result\PdoSqlsrv::numRows()` fallback branch no longer discards the return value of `parent::numRows()`. Previously, when `PDOStatement::rowCount()` returned `false`, the method could violate its declared `int` return contract. (#7)
