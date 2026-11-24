---
tags:
  - 'type:feature'
  - 'phase:8'
  - 'area:events'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:43:01.048Z
updated: 2026-11-24T04:59:20.554Z
---

# Phase 8.1: Implement EventEvaluator (all trigger types)

**Task ID:** 8.1
**Phase:** 8 — Events & Game Flow
**Dependencies:** Phase 4 (Mission Parser), Phase 5 (Combat), Phase 6 (AI Targets), Phase 7 (HUD)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §7.3 (Event System — trigger types: Approach, Depart, True, Score, Destroy, Alarm, StopNear, Shields)
- Consult `design/implementation_plan.md` §12 (Phase 8), §12.1 task 8.1, §12.2 (Game State Machine)
- Consult `design/gap_analysis.md` §3.4 (Server authority — events evaluated server-side in multiplayer)

---

**Objective:**
Implement the EventEvaluator — evaluates all trigger types every frame to check if events should fire. Triggers: Approach, Depart, True (always), Score, Destroy, Alarm, StopNear, Shields.

**Deliverables:**
- `src/missions/EventEvaluator.ts` — event trigger evaluation engine
- `tests/missions/event-evaluator.test.ts` — all trigger types tested

**Subtasks:**
- [ ] Define Trigger enum: Approach, Depart, True, Score, Destroy, Alarm, StopNear, Shields
- [ ] Implement EventEvaluator class with per-frame trigger checking
- [ ] Approach trigger: player comes within distance of event position
- [ ] Depart trigger: player moves beyond distance from event position
- [ ] True trigger: always fires (one-shot)
- [ ] Score trigger: player's score reaches or exceeds value
- [ ] Destroy trigger: named target is destroyed
- [ ] Alarm trigger: timer expires after N seconds
- [ ] StopNear trigger: player velocity near zero AND within distance
- [ ] Shields trigger: target's shields drop to or below value
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Approach: player within distance → trigger fires once
  - Depart: player beyond distance after being inside → trigger fires once
  - True: fires every frame until consumed (one-shot)
  - Score: player score reaches threshold → trigger fires
  - Destroy: target destroyed → trigger fires with target name
  - Alarm: timer reaches N seconds → trigger fires
- **Sad paths:**
  - Approach with distance 0: never triggers (player can't be at exact point)
  - Score trigger with score 0: fires immediately (score always ≥ 0)
  - Destroy trigger for non-existent target: never fires, logs warning
  - Multiple events with same trigger: all evaluated independently

**Completion Criteria:** All tests pass with ≥90% coverage. EventEvaluator correctly evaluates all 8 trigger types.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all triggers fire correctly) and sad path (edge cases in distance/score checks) tests must pass.**

## Sub-tasks

- [ ] Define Trigger enum with all 8 trigger types
- [ ] Implement EventEvaluator class with per-frame evaluation
- [ ] Implement Approach and Depart triggers (distance-based)
- [ ] Implement True trigger (always fires, one-shot)
- [ ] Implement Score, Destroy, Alarm triggers
- [ ] Implement StopNear and Shields triggers
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-4-1-implement-mission-file-tokenizer-whitespace-separated-comment-aware](phase-4-1-implement-mission-file-tokenizer-whitespace-separated-comment-aware.md)
- [depends-on phase-5-1-define-weapondef-all-5-weapons-spare](phase-5-1-define-weapondef-all-5-weapons-spare.md)
- [depends-on phase-6-1-implement-target-entity-with-ai-strategy-system](phase-6-1-implement-target-entity-with-ai-strategy-system.md)
- [depends-on phase-7-1-implement-iradar-circular-radar-overlay](phase-7-1-implement-iradar-circular-radar-overlay.md)

## History

- type: created
  date: 2026-11-24T03:43:01.048Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
