---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:network'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:08:02.399Z
updated: 2026-11-24T04:55:58.222Z
---

# Phase 0.18: Implement relay server (UDP, player tracking)

**Task ID:** 0.18
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (Project scaffolding)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §11 (Network/Multiplayer — relay server architecture, UDP-based)
- Consult `design/implementation_plan.md` §4.3 task 0.18 — Relay server implementation
- Consult `design/gap_analysis.md` §1.1 (Network Provider Architecture — relay server tracks player positions)

---

**Objective:**
Implement the relay server that tracks player positions and broadcasts them to all connected clients. This is a simple UDP server that receives POS packets and distributes them to other players.

**Deliverables:**
- `server/relay-server.ts` — relay server implementation
- `tests/server/relay-server.test.ts` — position tracking, broadcast tests

**Subtasks:**
- [ ] Implement UDP server on configurable port (default 7777)
- [ ] Implement player registration: client sends NAME packet on connect
- [ ] Track player positions: store last known position per player
- [ ] Broadcast positions: send POS packets to all connected clients
- [ ] Handle client disconnect: remove from tracking, notify others
- [ ] Implement basic rate limiting (max 60 updates/sec per player)
- [ ] Implement server health check (ping/pong)
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Client connects and sends NAME → registered with name
  - Client sends POS packet → position tracked correctly
  - Position broadcast → all other clients receive updated positions
  - Client disconnects → removed from tracking, others notified
- **Sad paths:**
  - Max players reached (32): new connection rejected with error
  - Rate limit exceeded: packets dropped, warning logged
  - Invalid packet format: ignored, no crash
  - Server crash recovery: existing connections lost (no persistence)

**Completion Criteria:** All tests pass with ≥90% coverage. Relay server tracks and broadcasts player positions correctly.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (position tracking works) and sad path (rate limiting, invalid packets) tests must pass.**

## Sub-tasks

- [ ] Implement UDP server on configurable port (default 7777)
- [ ] Implement player registration: NAME packet handling
- [ ] Track player positions: store last known position per player
- [ ] Broadcast positions: POS packets to all connected clients
- [ ] Handle client disconnect: remove from tracking, notify others
- [ ] Implement rate limiting (max 60 updates/sec per player)
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T04:08:02.399Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
