---
tags:
  - 'type:feature'
  - 'phase:11'
  - 'area:build'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:47:27.826Z
updated: 2026-11-24T05:00:04.114Z
---

# Phase 11.1: Configure Electron production build

**Task ID:** 11.1
**Phase:** 11 — Electron / Tauri Native Builds
**Dependencies:** Phase 10 (Multiplayer Layer), Phase 0.10 (Electron initialized)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §16 (Implementation Notes — Electron/Tauri support)
- Consult `design/implementation_plan.md` §15 (Phase 11), §15.1 task 11.1
- Consult `design/gap_analysis.md` §9.3 (Electron/Tauri — full Steam SDK access via steamworks.js)

---

**Objective:**
Configure Electron production build that produces distributable for macOS, Windows, and Linux. `npm run build:electron` builds the Vite app and packages it with Electron for distribution.

**Deliverables:**
- Production Electron build pipeline
- `npm run build:electron` produces distributable package

**Subtasks:**
- [ ] Configure Vite build output for Electron (static files to dist/)
- [ ] Update `electron/main.js` for production mode (load from dist/ not dev server)
- [ ] Add electron-builder or electron-packager configuration
- [ ] Configure macOS notarization (codesign, notarytool)
- [ ] Configure Windows NSIS installer
- [ ] Configure Linux AppImage/DEB output
- [ ] Test: `npm run build:electron` produces distributable for current platform
- [ ] Add platform-specific build scripts: build:electron:mac, build:electron:win, build:electron:linux

**Unit Tests Required:** N/A (build pipeline tested manually; integration test in Phase 11)

**Completion Criteria:** `npm run build:electron` produces a working distributable package. All platforms supported.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (build succeeds, package runs) and sad path (missing signing certs handled gracefully) tests must pass.**

## Sub-tasks

- [ ] Configure Vite build output for Electron (static files to dist/)
- [ ] Update electron/main.js for production mode
- [ ] Add electron-builder/packager configuration
- [ ] Configure macOS notarization (codesign, notarytool)
- [ ] Configure Windows NSIS installer
- [ ] Configure Linux AppImage/DEB output
- [ ] Test build:electron produces distributable for current platform

## Relations

- [depends-on phase-10-1-define-inetwork-layer-interface-spnetwork-layer-no-op](phase-10-1-define-inetwork-layer-interface-spnetwork-layer-no-op.md)
- [depends-on phase-0-10-initialize-electron-app](phase-0-10-initialize-electron-app.md)

## History

- type: created
  date: 2026-11-24T03:47:27.826Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
