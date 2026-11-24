---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:tooling'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:14:38.535Z
updated: 2026-11-24T04:55:06.861Z
---

# Phase 0.2: Create vitest.config.ts with coverage thresholds

**Task ID:** 0.2
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)

---

**References:**
- Consult `design/Orbit.md` for game overview and architecture recommendations
- Consult `design/implementation_plan.md` §3 (Build & Test Infrastructure) for Vitest config, coverage thresholds (90/85/90/90)
- Consult `design/gap_analysis.md` §7.1 for Phase 0 additions

---

**Objective:**
Configure Vitest with coverage thresholds matching CharGen standards. Deliverable: `npm run test:coverage` passes with green thresholds.

**Deliverables:**
- `vitest.config.ts` with coverage enabled (text + lcov reporters)
- Coverage thresholds: lines 90%, branches 85%, functions 90%, statements 90%
- Test environment set to 'node', globals enabled, testMatch for `tests/**/*.test.ts`

**Subtasks:**
- [ ] Create `vitest.config.ts` with coverage configuration (text + lcov reporters)
- [ ] Set coverage thresholds: lines 90%, branches 85%, functions 90%, statements 90%
- [ ] Configure test environment to 'node', globals true, testMatch pattern
- [ ] Create a dummy test file to verify Vitest runs correctly
- [ ] Run `npm run test:coverage` and verify coverage report generates

**Unit Tests Required:**
- **Happy path:** Dummy test passes, coverage report generates with all metrics
- **Sad path:** Verify coverage threshold failures are reported when thresholds not met (temporarily lower threshold on a failing test)

**Completion Criteria:** `npm run test:coverage` runs successfully and produces coverage reports.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all scripts work as expected) and sad path (invalid config rejected) tests must pass.**

## Sub-tasks

- [ ] Create vitest.config.ts with coverage configuration
- [ ] Set coverage thresholds: lines 90%, branches 85%, functions 90%, statements 90%
- [ ] Configure test environment to 'node', globals true, testMatch pattern
- [ ] Create a dummy test file to verify Vitest runs correctly
- [ ] Run npm run test:coverage and verify coverage report generates

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:14:38.535Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
