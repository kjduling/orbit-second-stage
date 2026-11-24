---
tags:
  - 'type:feature'
  - 'phase:10'
  - 'area:network'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:45:17.285Z
updated: 2026-11-24T04:59:49.312Z
---

# Phase 10.1: Define INetworkLayer interface + SPNetworkLayer (no-op)

**Task ID:** 10.1
**Phase:** 10 — Multiplayer Layer
**Dependencies:** Phase 8 (Events & Game Flow), Phase 0.18-0.21 (Networking infrastructure)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §11 (Network/Multiplayer — packet formats, architecture)
- Consult `design/implementation_plan.md` §14 (Phase 10), §14.1 task 10.1, §14.2 (Packet Format)
- Consult `design/gap_analysis.md` §1.1 (Network Provider Architecture — Steam SDR + WebSocket hybrid)

---

**Objective:**
Define the INetworkLayer interface that abstracts all network operations. Implement SPNetworkLayer as a no-op for single-player mode (zero overhead when multiplayer not used). This is the foundation for all multiplayer functionality.

**Deliverables:**
- `src/network/INetworkLayer.ts` — network interface
- `src/network/SPNetworkLayer.ts` — single-player no-op implementation
- `tests/network/packet.test.ts` — packet format, serialization tests

**Subtasks:**
- [ ] Define `INetworkLayer` interface: connect, disconnect, sendPacket, onPacket, isConnected
- [ ] Implement `SPNetworkLayer` as no-op (all methods do nothing, isConnected = false)
- [ ] Define binary packet format: magic byte 0x56, size-prefixed packets
- [ ] Define PacketType enum: POSS(0x80), POSL(0x81), POSU(0x82), POSN(0x83), PLAN(0x84), VCNT(0x85), PING(0x86)
- [ ] Define ASCIICommand types: FIRE, CHAT, CMSG, DROP, NAME
- [ ] Implement packet serialization/deserialization utilities
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - SPNetworkLayer.connect(): does nothing, returns immediately
  - SPNetworkLayer.isConnected(): returns false
  - Packet serialization: IPacket → Uint8Array with correct magic byte and type
  - Packet deserialization: Uint8Array → IPacket with correct fields
  - All packet types have unique type values (no collisions)
- **Sad paths:**
  - Packet too short (< 3 bytes): deserialization throws error
  - Invalid magic byte: deserialization throws error
  - Packet size exceeds buffer: truncated to buffer size, warning logged

**Completion Criteria:** All tests pass with ≥90% coverage. INetworkLayer interface defined, SPNetworkLayer provides zero-overhead no-op for single-player.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (packet serialization correct) and sad path (invalid packets rejected) tests must pass.**

## Sub-tasks

- [ ] Define INetworkLayer interface: connect, disconnect, sendPacket, onPacket, isConnected
- [ ] Implement SPNetworkLayer no-op (all methods do nothing)
- [ ] Define binary packet format with magic byte 0x56
- [ ] Define PacketType enum: POSS, POSL, POSU, POSN, PLAN, VCNT, PING
- [ ] Define ASCIICommand types: FIRE, CHAT, CMSG, DROP, NAME
- [ ] Implement packet serialization/deserialization utilities
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-8-1-implement-event-evaluator-all-trigger-types](phase-8-1-implement-event-evaluator-all-trigger-types.md)
- [depends-on phase-0-18-0-21-networking-infrastructure-relay-server-steam-sdk-open-id-auth-server-discovery](phase-0-18-0-21-networking-infrastructure-relay-server-steam-sdk-open-id-auth-server-discovery.md)

## History

- type: created
  date: 2026-11-24T03:45:17.285Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
