---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:native'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:24:23.245Z
updated: 2026-11-24T04:55:07.162Z
---

# Phase 0.10: Initialize Electron app

**Task ID:** 0.10
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §16 (Implementation Notes for BabylonJS Port)
- Consult `design/implementation_plan.md` §3.4 (Electron Setup — early setup rationale)
- Consult `design/gap_analysis.md` §1.1, §9.3 (Steam Networking Sockets for native builds)

---

**Objective:**
Initialize Electron app with minimal window that loads Vite dev server in dev mode and built files from `dist/` in production. Dev mode connects to Vite HMR. Steam integration loaded in main process via steamworks.js.

**Deliverables:**
- `electron/main.js` — minimal Electron main process
- `electron/preload.js` — context bridge for IPC
- `electron/package.json` — Electron-specific dependencies
- `npm run electron:start` opens a window

**Subtasks:**
- [ ] Create `electron/` directory with `package.json` (electron, @electron/remote, steamworks.js)
- [ ] Implement `electron/main.js`: create BrowserWindow, load Vite dev server URL in dev mode
- [ ] Implement `electron/preload.js`: context bridge for IPC between renderer and main process
- [ ] Configure dev mode: Electron connects to Vite HMR on localhost:5173
- [ ] Configure prod mode: Electron loads built files from `dist/`
- [ ] Add npm script: `electron:start` runs Electron with proper env detection
- [ ] Test: `npm run electron:start` opens a window displaying the app

**Unit Tests Required:** N/A (Electron integration tested manually; unit tests for IPC bridge in Phase 11)

**Completion Criteria:** `npm run electron:start` opens a window. Dev mode uses Vite HMR, prod mode loads from dist/.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (window opens correctly) and sad path (dev/prod mode detection) tests must pass.**

## Sub-tasks

- [ ] Create electron/ directory with package.json including electron + steamworks.js
- [ ] Implement electron/main.js: BrowserWindow with dev/prod mode detection
- [ ] Implement electron/preload.js: context bridge for IPC
- [ ] Configure dev mode to connect to Vite HMR
- [ ] Configure prod mode to load from dist/
- [ ] Add npm script and verify npm run electron:start opens window

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:24:23.245Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
