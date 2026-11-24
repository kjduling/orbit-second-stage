---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:native'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T03:24:23.338Z
updated: 2026-11-24T04:55:57.686Z
---

# Phase 0.11: Initialize Tauri app

**Task ID:** 0.11
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §16 (Implementation Notes for BabylonJS Port)
- Consult `design/implementation_plan.md` §3.5 (Tauri Setup — early setup rationale)
- Consult `design/gap_analysis.md` §9.3 (Steamworks.js via Rust N-API wrapper in Tauri)

---

**Objective:**
Initialize Tauri app as alternative native build target. Smaller footprint than Electron, preferred for performance-conscious builds. Same Vite output served by Tauri backend.

**Deliverables:**
- `tauri/tauri.conf.json` — basic config, app ID `orbit.secondstage`
- `tauri/Cargo.toml` — Rust deps: tauri, steamworks.js via N-API
- `npm run tauri:dev` opens a window

**Subtasks:**
- [ ] Initialize Tauri project: `tauri/tauri.conf.json` with app ID `orbit.secondstage`
- [ ] Create `tauri/Cargo.toml` with Rust deps: tauri, steamworks.js (via N-API)
- [ ] Implement Tauri backend to serve Vite-built static files from `dist/`
- [ ] Configure Rust handlers for native file I/O and audio
- [ ] Add npm script: `tauri:dev` runs Tauri dev server
- [ ] Test: `npm run tauri:dev` opens a window displaying the app

**Unit Tests Required:** N/A (Tauri integration tested manually; Rust unit tests in Cargo.toml)

**Completion Criteria:** `npm run tauri:dev` opens a window. Tauri serves Vite output correctly.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (window opens correctly) and sad path (config errors caught) tests must pass.**

## Sub-tasks

- [ ] Initialize Tauri project with tauri.conf.json (app ID: orbit.secondstage)
- [ ] Create Cargo.toml with tauri + steamworks.js Rust dependencies
- [ ] Configure Tauri backend to serve Vite-built static files
- [ ] Configure Rust handlers for native file I/O and audio
- [ ] Add npm script and verify npm run tauri:dev opens window

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:24:23.338Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
