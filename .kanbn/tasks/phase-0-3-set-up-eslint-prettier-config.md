---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:tooling'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T03:14:38.662Z
updated: 2026-11-24T04:55:06.890Z
---

# Phase 0.3: Set up ESLint + Prettier config

**Task ID:** 0.3
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)

---

**References:**
- Consult `design/Orbit.md` for game overview and architecture recommendations
- Consult `design/implementation_plan.md` §3 (Build & Test Infrastructure) for lint/format scripts
- Consult `design/gap_analysis.md` §7.1 for Phase 0 additions

---

**Objective:**
Configure ESLint + Prettier for consistent code quality. Deliverable: `npm run lint` and `npm run format` run clean.

**Deliverables:**
- `eslint.config.js` with TypeScript support, strict rules
- `.prettierrc.js` with consistent formatting config
- `npm run lint` passes on empty project (no errors)
- `npm run format` formats all files consistently

**Subtasks:**
- [ ] Install ESLint + TypeScript parser + BabylonJS globals
- [ ] Create `eslint.config.js` with strict rules, no-unused-vars, no-explicit-any
- [ ] Install Prettier + ESLint Prettier plugin
- [ ] Create `.prettierrc.js` with 2-space indent, semicolons, single quotes
- [ ] Verify `npm run lint` runs clean on empty project
- [ ] Verify `npm run format` formats files correctly

**Unit Tests Required:** N/A (tooling task)

**Completion Criteria:** `npm run lint` and `npm run format` both run without errors.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all scripts work as expected) and sad path (invalid config rejected) tests must pass.**

## Sub-tasks

- [ ] Install ESLint + TypeScript parser + BabylonJS globals
- [ ] Create eslint.config.js with strict rules
- [ ] Install Prettier + ESLint Prettier plugin
- [ ] Create .prettierrc.js with 2-space indent, semicolons, single quotes
- [ ] Verify npm run lint runs clean on empty project
- [ ] Verify npm run format formats files correctly

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:14:38.662Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
