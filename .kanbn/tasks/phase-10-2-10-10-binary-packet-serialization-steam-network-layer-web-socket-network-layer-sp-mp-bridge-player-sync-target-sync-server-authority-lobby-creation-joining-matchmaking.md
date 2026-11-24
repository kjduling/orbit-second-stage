---
tags:
  - 'type:feature'
  - 'phase:10'
  - 'area:network'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:29:57.532Z
updated: 2026-11-24T04:59:49.381Z
---

# Phase 10.2-10.10: Binary packet serialization, Steam network layer, WebSocket network layer, SP-MP bridge, player sync, target sync, server authority, lobby creation/joining, matchmaking

**Task ID:** 10.2–10.10 (grouped multiplayer layer tasks)
**Phase:** 10 — Multiplayer Layer
**Dependencies:** Phase 10.1 (INetworkLayer interface), Phase 0.18-0.21 (Networking infrastructure)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §11 (Network/Multiplayer — packet formats, architecture, Steam SDR)
- Consult `design/implementation_plan.md` §14.1 tasks 10.2–10.10 — Multiplayer components
- Consult `design/gap_analysis.md` §1.1 (Network Provider Architecture — Steam SDR + WebSocket hybrid)

---

**Objective:**
Implement the complete multiplayer layer: binary packet serialization, Steam network layer, WebSocket network layer, single-player/multiplayer bridge, player state synchronization, target state synchronization, server authority for AI, lobby creation/joining, and matchmaking.

**Deliverables:**
- `src/network/PacketSerializer.ts` — binary packet serialization/deserialization
- `src/network/SteamNetworkLayer.ts` — Steam SDR network implementation
- `src/network/WebSocketNetworkLayer.ts` — WebSocket network implementation
- `src/network/SPMPBridge.ts` — single-player to multiplayer bridge
- `src/network/PlayerSync.ts` — player state synchronization
- `src/network/TargetSync.ts` — target state synchronization
- `src/network/ServerAuthority.ts` — server-side AI authority
- `src/network/LobbyManager.ts` — lobby creation and joining
- `src/network/Matchmaking.ts` — matchmaking system
- `tests/network/multiplayer.test.ts` — all multiplayer components tested

**Subtasks:**
- [ ] 10.2: Implement PacketSerializer: binary packet serialization (magic byte 0x56)
- [ ] 10.3: Implement SteamNetworkLayer: Steam SDR networking
- [ ] 10.4: Implement WebSocketNetworkLayer: WebSocket networking for web builds
- [ ] 10.5: Implement SPMPBridge: bridge between single-player and multiplayer modes
- [ ] 10.6: Implement PlayerSync: synchronize player position/velocity/state
- [ ] 10.7: Implement TargetSync: synchronize target positions/states
- [ ] 10.8: Implement ServerAuthority: server-side AI decision making
- [ ] 10.9: Implement LobbyManager: create/join/manage lobbies
- [ ] 10.10: Implement Matchmaking: find suitable servers for players
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Packet serialized/deserialized correctly with magic byte 0x56
  - Steam network connects and sends/receives packets
  - WebSocket network connects and sends/receives packets
  - SP-MP bridge: single-player game transitions to multiplayer seamlessly
  - Player sync: player position updated on all clients within 1 frame
  - Target sync: target positions/states synchronized across clients
  - Server authority: server makes AI decisions, clients follow
  - Lobby created: other players can join by lobby ID
  - Matchmaking: finds server with appropriate player count and ping
- **Sad paths:**
  - Packet too short: deserialization throws error
  - Steam network disconnected: reconnects automatically (up to 3 times)
  - WebSocket connection failed: falls back to single-player mode
  - SP-MP bridge during active mission: throws error (must be in menu)
  - Player sync lag: client interpolates position, doesn't snap

**Completion Criteria:** All tests pass with ≥90% coverage. Multiplayer layer fully functional: packets serialize, networks connect, state synchronized.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (multiplayer works end-to-end) and sad path (network errors, disconnections) tests must pass.**

## Sub-tasks

- [ ] 10.2: Implement PacketSerializer: binary packet serialization (magic byte 0x56)
- [ ] 10.3: Implement SteamNetworkLayer: Steam SDR networking
- [ ] 10.4: Implement WebSocketNetworkLayer: WebSocket networking for web builds
- [ ] 10.5: Implement SPMPBridge: bridge between single-player and multiplayer modes
- [ ] 10.6: Implement PlayerSync: synchronize player position/velocity/state
- [ ] 10.7: Implement TargetSync: synchronize target positions/states
- [ ] 10.8: Implement ServerAuthority: server-side AI decision making
- [ ] 10.9: Implement LobbyManager: create/join/manage lobbies
- [ ] 10.10: Implement Matchmaking: find suitable servers for players
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-10-1-define-inetwork-layer-interface-spnetwork-layer-no-op](phase-10-1-define-inetwork-layer-interface-spnetwork-layer-no-op.md)
- [depends-on phase-0-18-implement-relay-server-udp-player-tracking](phase-0-18-implement-relay-server-udp-player-tracking.md)
- [depends-on phase-0-19-integrate-steam-sdk-steamworks-js](phase-0-19-integrate-steam-sdk-steamworks-js.md)

## History

- type: created
  date: 2026-11-24T04:29:57.532Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
