---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:core'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T03:22:40.307Z
updated: 2026-11-24T04:55:07.115Z
---

# Phase 0.9: Implement Logger (structured, log levels)

**Task ID:** 0.9
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** Medium

---

**References:**
- Consult `design/Orbit.md` §16 (Implementation Notes) for architecture recommendations
- Consult `design/implementation_plan.md` §4.1 task 0.9 — Logger deliverables and tests
- Consult `design/gap_analysis.md` §3.4 (Server authority, packet validation logging needs)

---

**Objective:**
Implement a structured logger with configurable log levels (DEBUG, INFO, WARN, ERROR). Used throughout the codebase for debugging, diagnostics, and mission loading verbosity.

**Deliverables:**
- `src/core/Logger.ts` — structured logger with levels
- `tests/core/logger.test.ts` — ≥90% coverage
- Methods: debug, info, warn, error, setLevel, disable

**Subtasks:**
- [ ] Define LogLevel enum: DEBUG=0, INFO=1, WARN=2, ERROR=3
- [ ] Implement Logger class with configurable minimum log level
- [ ] Implement `debug()`, `info()`, `warn()`, `error()` methods
- [ ] Include timestamp and log level in output
- [ ] Support structured logging: `info('Player hit', { target: name, damage })`
- [ ] Implement static `setLevel(level)` for global config
- [ ] Implement `disable()` to suppress all output (production mode)
- [ ] Write unit tests with ≥90% coverage

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Log level INFO: info, warn, error appear; debug suppressed
  - Log level DEBUG: all levels appear
  - Structured log: output contains both message and data properties
  - Disable logger: no output at any level
- **Sad paths:**
  - Invalid log level string: throws error or defaults to INFO
  - Empty message: handled gracefully (no crash)
  - Log with null/undefined data: serialized as "null"/"undefined"

**Completion Criteria:** All tests pass with ≥90% coverage. Logger supports all log levels and structured output.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all levels work) and sad path (invalid input handled) tests must pass.**

## Sub-tasks

- [ ] Define LogLevel enum: DEBUG, INFO, WARN, ERROR
- [ ] Implement Logger class with configurable minimum log level
- [ ] Implement debug, info, warn, error methods with timestamp and level
- [ ] Support structured logging with data objects
- [ ] Implement setLevel and disable for global config
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:22:40.307Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
