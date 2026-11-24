---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:network'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:08:02.415Z
updated: 2026-11-24T04:56:44.164Z
---

# Phase 0.19: Integrate Steam SDK (steamworks.js)

**Task ID:** 0.19
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (Project scaffolding), Phase 0.18 (Relay server)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §11 (Network/Multiplayer — Steam SDR integration)
- Consult `design/implementation_plan.md` §4.3 task 0.19 — Steam SDK integration
- Consult `design/gap_analysis.md` §1.1 (Network Provider Architecture — Steam SDR for NAT traversal, lobby creation)

---

**Objective:**
Integrate Steamworks.js for Steam SDK access in Electron builds. This provides Steam networking (SDR), lobby creation/joining, authentication, and overlay functionality.

**Deliverables:**
- `src/network/steam/SteamManager.ts` — Steam SDK wrapper
- `tests/network/steam-manager.test.ts` — Steam init, lobby tests

**Subtasks:**
- [ ] Install steamworks.js dependency
- [ ] Implement `SteamManager` class: init, shutdown, createLobby, joinLobby
- [ ] Initialize Steam with app ID (to be assigned)
- [ ] Implement Steam authentication (get user identity)
- [ ] Implement lobby creation: public/private, max players
- [ ] Implement lobby joining: by lobby ID or browser list
- [ ] Handle Steam networking callbacks: join request, data received
- [ ] Implement Steam overlay toggle (Shift+Tab)
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Steam init: initializes successfully with valid app ID
  - Create lobby: returns lobby ID, other players can join
  - Join lobby: connects to existing lobby by ID
  - Get user identity: returns Steam ID and persona name
  - Lobby data sent/received: binary packets transmitted correctly
- **Sad paths:**
  - Steam not running: init fails gracefully with error message
  - Invalid app ID: init fails with descriptive error
  - Join non-existent lobby: fails, logs warning
  - Network error during data send: retries up to 3 times, then fails

**Completion Criteria:** All tests pass with ≥90% coverage. Steam SDK integrated and functional in Electron builds.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (Steam init, lobby creation) and sad path (Steam not running, invalid app ID) tests must pass.**

## Sub-tasks

- [ ] Install steamworks.js dependency
- [ ] Implement SteamManager class: init, shutdown, createLobby, joinLobby
- [ ] Initialize Steam with app ID
- [ ] Implement Steam authentication (get user identity)
- [ ] Implement lobby creation: public/private, max players
- [ ] Implement lobby joining: by ID or browser list
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)
- [depends-on phase-0-18-implement-relay-server-udp-player-tracking](phase-0-18-implement-relay-server-udp-player-tracking.md)

## History

- type: created
  date: 2026-11-24T04:08:02.415Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
