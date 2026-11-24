---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:tooling'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:13:09.798Z
updated: 2026-11-24T04:55:06.692Z
---

# Phase 0.1: Initialize project scaffolding (package.json, tsconfig, vite.config)

**Task ID:** 0.1
**Phase:** 0 — Foundation
**Duration:** ~1-2 days (shared across Phase 0)

---

**References:**
- Consult `design/Orbit.md` for game overview and architecture recommendations
- Consult `design/implementation_plan.md` §4 (Phase 0) for full task details, key interfaces, and deliverables
- Consult `design/gap_analysis.md` §7.1 for Phase 0 additions (asset creation, relay server, Steam SDK setup)

---

**Objective:**
Initialize the project with working build system. Deliverable: `npm run dev` starts Vite dev server successfully.

**Deliverables:**
- `package.json` with all scripts (dev, build, test, lint, format, etc.)
- `tsconfig.json` with proper module resolution and strict mode
- `vite.config.ts` configured for both browser and Node targets
- Working `npm run dev` that serves the app

---

**Subtasks:**
- [ ] Create `package.json` with dependencies: BabylonJS, Vite, TypeScript, Vitest, ESLint, Prettier
- [ ] Configure `tsconfig.json` with ES2020 target, strict mode, path aliases for `src/` and `tests/`
- [ ] Create `vite.config.ts` with dev server, build config, and HMR support
- [ ] Add npm scripts: dev, build, preview, test, test:run, test:coverage, lint, format, clean
- [ ] Create minimal `src/index.ts` entry point that renders a blank BabylonJS scene
- [ ] Verify `npm run dev` launches Vite dev server and page loads

**Unit Tests Required:** N/A (infrastructure task)

**Completion Criteria:** `npm run dev` works, `npm run build` succeeds, project structure matches implementation plan §2.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all scripts work as expected) and sad path (invalid config rejected) tests must pass.**

## Sub-tasks

- [ ] Create package.json with all dependencies and scripts
- [ ] Configure tsconfig.json with ES2020 target, strict mode, path aliases
- [ ] Create vite.config.ts with dev server, build config, HMR support
- [ ] Add npm scripts: dev, build, preview, test, test:run, test:coverage, lint, format, clean
- [ ] Create minimal src/index.ts entry point that renders a blank BabylonJS scene
- [ ] Verify npm run dev launches Vite dev server and page loads

## Comments

- author: Qwen
  date: 2026-11-23T00:00:00.000Z
  Phase 0 is the foundation — everything else depends on this. No visual gratification yet but critical for all downstream work.

## History

- type: created
  date: 2026-11-24T03:13:09.798Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
