---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:network'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:26:33.632Z
updated: 2026-11-24T04:55:58.022Z
---

# Phase 0.18-0.21: Networking infrastructure (relay server, Steam SDK, OpenID auth, server discovery)

**Task ID:** 0.18–0.21 (grouped networking infrastructure tasks)
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.10 (Electron), Phase 0.11 (Tauri)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §11 (Network/Multiplayer) for packet formats and architecture
- Consult `design/implementation_plan.md` §1.3 (Hybrid Network Architecture), §4.1 tasks 0.18–0.21
- Consult `design/gap_analysis.md` §1.1, §2.1, §9 (Steam + Browser Architecture Research)

---

**Objective:**
Set up the networking infrastructure: relay server scaffolding, Steamworks.js integration for Electron/Tauri, Steam OpenID auth flow for browser clients, and server discovery via Steam Web API.

**Deliverables:**
- `src/network/RelayServer.ts` — dual transport (Steam SDR + WebSocket) scaffolding
- `src/network/steamworks-integration.ts` — Steam SDK bridge for Electron/Tauri main process
- `src/network/openid-auth.ts` — browser OpenID 2.0 login flow
- `src/network/server-discovery.ts` — IGameServersService/GetServerList integration

**Subtasks:**
- [ ] **0.18 Relay Server:** Scaffold `RelayServer.ts` with dual transport listeners (Steam SDR + WebSocket)
- [ ] **0.19 Steamworks.js:** Integrate `steamworks.js` for Electron/Tauri — full Steam SDK access (lobby, SDR, overlay, auth tickets)
- [ ] **0.20 OpenID Auth:** Implement Steam OpenID 2.0 flow — browser → OpenID login → SteamID64 verification via Web API
- [ ] **0.21 Server Discovery:** Implement `IGameServersService/GetServerList` HTTP API → JSON server list for browser matchmaking

**Unit Tests Required:**
- **Happy paths (0.20):** OpenID login returns valid SteamID64, CheckAppOwnership confirms game ownership
- **Sad paths (0.20):** Invalid OpenID token rejected, expired token rejected, app not owned → access denied
- **Happy paths (0.21):** Server list endpoint returns valid JSON array of servers
- **Sad paths (0.21):** Steam Web API rate limited → retry with exponential backoff, no servers found → empty array

**Completion Criteria:** Networking infrastructure scaffolding in place. Steamworks.js loads in Electron/Tauri. OpenID auth flow functional. Server discovery returns JSON list.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (auth works, servers found) and sad path (invalid tokens, rate limits) tests must pass.**

## Sub-tasks

- [ ] 0.18: Scaffold RelayServer.ts with dual transport listeners
- [ ] 0.19: Integrate steamworks.js for Electron/Tauri (lobby, SDR, overlay)
- [ ] 0.20: Implement Steam OpenID 2.0 auth flow for browser clients
- [ ] 0.21: Implement server discovery via IGameServersService/GetServerList

## Relations

- [depends-on phase-0-10-initialize-electron-app](phase-0-10-initialize-electron-app.md)
- [depends-on phase-0-11-initialize-tauri-app](phase-0-11-initialize-tauri-app.md)

## History

- type: created
  date: 2026-11-24T03:26:33.632Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
