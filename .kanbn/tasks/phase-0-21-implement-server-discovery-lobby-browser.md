---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:network'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T04:09:41.635Z
updated: 2026-11-24T04:56:44.256Z
---

# Phase 0.21: Implement server discovery & lobby browser

**Task ID:** 0.21
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.18 (Relay server), Phase 0.19 (Steam SDK)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §11 (Network/Multiplayer — server discovery, lobby browser UI)
- Consult `design/implementation_plan.md` §4.3 task 0.21 — Server discovery and lobby browser
- Consult `design/gap_analysis.md` §1.1 (Network Provider Architecture — Steam lobby browser + UDP broadcast for local servers)

---

**Objective:**
Implement server discovery system that finds available multiplayer servers. Uses Steam lobby list for Steam players, UDP broadcast for local network discovery, and manual IP entry for custom servers.

**Deliverables:**
- `src/network/ServerBrowser.ts` — server discovery and lobby browser
- `tests/network/server-browser.test.ts` — discovery, filtering tests

**Subtasks:**
- [ ] Implement `ServerBrowser` class: scan, filter, select, connect
- [ ] Steam lobby list: query Steam for available lobbies
- [ ] UDP broadcast: discover local servers on network (port 7777)
- [ ] Manual IP entry: allow user to type server address
- [ ] Server info display: player count, map name, ping, password status
- [ ] Filter servers: by player count, region, password protection
- [ ] Sort servers: by ping, player count, or name
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Steam lobby scan: returns list of available lobbies with player counts
  - UDP broadcast: discovers local servers within network
  - Manual IP entry: connects to specified server address
  - Filter servers: only servers matching criteria shown
  - Sort servers: ordered by ping (lowest first) or player count
- **Sad paths:**
  - No servers found: returns empty list, no crash
  - Server timeout: marked as unavailable, removed from list
  - Password-protected server: flagged, cannot join without password
  - Invalid IP address: rejected with validation error

**Completion Criteria:** All tests pass with ≥90% coverage. Server browser discovers and displays available servers correctly.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (server discovery works) and sad path (no servers, invalid IP) tests must pass.**

## Sub-tasks

- [ ] Implement ServerBrowser class: scan, filter, select, connect
- [ ] Steam lobby list: query Steam for available lobbies
- [ ] UDP broadcast: discover local servers on network
- [ ] Manual IP entry: allow user to type server address
- [ ] Server info display: player count, map name, ping, password status
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-18-implement-relay-server-udp-player-tracking](phase-0-18-implement-relay-server-udp-player-tracking.md)
- [depends-on phase-0-19-integrate-steam-sdk-steamworks-js](phase-0-19-integrate-steam-sdk-steamworks-js.md)

## History

- type: created
  date: 2026-11-24T04:09:41.635Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
