---
tags:
  - 'type:feature'
  - 'phase:11'
  - 'area:build'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T04:31:33.575Z
updated: 2026-11-24T05:00:04.186Z
---

# Phase 11.2-11.12: Tauri build, file bridges (Electron + Tauri), audio setup, code signing, Steam overlay, native launcher UI, platform-specific builds, app menu, window management, auto-updater, crash reporter

**Task ID:** 11.2–11.12 (grouped native build tasks)
**Phase:** 11 — Electron / Tauri Native Builds
**Dependencies:** Phase 11.1 (Electron production build), Phase 10 (Multiplayer Layer)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §16 (Implementation Notes — Electron/Tauri support, native builds)
- Consult `design/implementation_plan.md` §15.1 tasks 11.2–11.12 — Native build components
- Consult `design/gap_analysis.md` §9.3 (Electron/Tauri — full Steam SDK access via steamworks.js)

---

**Objective:**
Implement complete native build support: Tauri build pipeline, file bridges for saves and missions on both Electron and Tauri, audio setup, code signing for all platforms, Steam overlay integration, native launcher UI, platform-specific builds, app menu, window management, auto-updater, and crash reporter.

**Deliverables:**
- `src/native/TauriBridge.ts` — Tauri file system bridge
- `src/native/ElectronFileBridge.ts` — Electron file system bridge
- `src/native/TauriFileBridge.ts` — Tauri file system bridge
- `src/native/AudioSetup.ts` — native audio setup (WebAudio + native)
- `src/native/CodeSigning.ts` — code signing for macOS/Windows/Linux
- `src/native/SteamOverlay.ts` — Steam overlay integration
- `src/native/LauncherUI.ts` — native launcher UI
- `tests/native/native-builds.test.ts` — all native components tested

**Subtasks:**
- [ ] 11.2: Implement Tauri build pipeline (parallel to Electron)
- [ ] 11.3: Implement ElectronFileBridge: file I/O for saves/missions
- [ ] 11.4: Implement TauriFileBridge: file I/O for saves/missions
- [ ] 11.5: Implement AudioSetup: native audio setup (WebAudio + native backends)
- [ ] 11.6: Implement CodeSigning: codesign, notarytool for macOS; NSIS for Windows
- [ ] 11.7: Implement SteamOverlay: Shift+Tab overlay in Electron builds
- [ ] 11.8: Implement LauncherUI: native window with mission select, settings
- [ ] 11.9: Implement platform-specific builds: build:electron:mac/win/linux, build:tauri:mac/win/linux
- [ ] 11.10: Implement AppMenu: native application menu (File, Edit, View, Help)
- [ ] 11.11: Implement WindowManagement: fullscreen, resizable, minimize
- [ ] 11.12: Implement AutoUpdater: check for updates, download, install
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Electron file bridge: reads/writes save files correctly
  - Tauri file bridge: reads/writes save files correctly
  - Audio setup: native audio backend initialized correctly
  - Code signing: macOS app codesigned and notarized successfully
  - Steam overlay: Shift+Tab opens overlay in Electron builds
  - Launcher UI: mission select, settings displayed correctly
  - Platform-specific builds: produces correct package for each platform
  - Auto-updater: checks for updates, downloads and installs new version
- **Sad paths:**
  - File bridge unavailable (web build): falls back to IndexedDB, logs warning
  - Audio setup fails: falls back to WebAudio default, logs warning
  - Code signing fails: build continues without signing, logs error
  - Steam overlay not available (non-Steam build): no-op, no crash
  - Auto-update check fails: logs warning, continues with current version

**Completion Criteria:** All tests pass with ≥90% coverage. Native builds fully functional: Electron and Tauri both produce distributable packages.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (native builds work) and sad path (fallbacks for unavailable features) tests must pass.**

## Sub-tasks

- [ ] 11.2: Implement Tauri build pipeline (parallel to Electron)
- [ ] 11.3: Implement ElectronFileBridge: file I/O for saves/missions
- [ ] 11.4: Implement TauriFileBridge: file I/O for saves/missions
- [ ] 11.5: Implement AudioSetup: native audio setup (WebAudio + native backends)
- [ ] 11.6: Implement CodeSigning: codesign, notarytool for macOS; NSIS for Windows
- [ ] 11.7: Implement SteamOverlay: Shift+Tab overlay in Electron builds
- [ ] 11.8: Implement LauncherUI: native window with mission select, settings
- [ ] 11.9: Implement platform-specific builds: build:electron:mac/win/linux, build:tauri:mac/win/linux
- [ ] 11.10: Implement AppMenu: native application menu (File, Edit, View, Help)
- [ ] 11.11: Implement WindowManagement: fullscreen, resizable, minimize
- [ ] 11.12: Implement AutoUpdater: check for updates, download, install
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-11-1-configure-electron-production-build](phase-11-1-configure-electron-production-build.md)
- [depends-on phase-10-2-10-10-binary-packet-serialization-steam-network-layer-web-socket-network-layer-sp-mp-bridge-player-sync-target-sync-server-authority-lobby-creation-joining-matchmaking](phase-10-2-10-10-binary-packet-serialization-steam-network-layer-web-socket-network-layer-sp-mp-bridge-player-sync-target-sync-server-authority-lobby-creation-joining-matchmaking.md)

## History

- type: created
  date: 2026-11-24T04:31:33.575Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
