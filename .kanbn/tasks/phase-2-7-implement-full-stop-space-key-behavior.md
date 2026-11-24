---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:physics'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T04:14:13.447Z
updated: 2026-11-24T04:57:25.365Z
---

# Phase 2.7: Implement full stop (SPACE key) behavior

**Task ID:** 2.7
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 2.4 (Input axis mapping), Phase 2.6 (Gravity)
**Type:** Feature | **Priority:** Medium

---

**References:**
- Consult `design/Orbit.md` §2.1 (Player Ship — SPACE key full stop, velocity = 0)
- Consult `design/implementation_plan.md` §6.1 task 2.7 — Full stop behavior
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — full stop must be deterministic)

---

**Objective:**
Implement the SPACE key full stop behavior: when pressed, ship velocity is instantly set to zero. This simulates firing retro-rockets in all directions. Only works in Newtonian flight model (not arcade — arcade has its own full stop via throttle = 0).

**Deliverables:**
- `src/player/FullStop.ts` — full stop logic integrated into Ship.update()
- `tests/player/full-stop.test.ts` — velocity zeroing, gravity interaction tests

**Subtasks:**
- [ ] Implement full stop trigger: SPACE key pressed in NORMAL state
- [ ] Set ship velocity to zero instantly (both magnitude and direction)
- [ ] Apply gravity after full stop (ship begins falling toward nearest planet)
- [ ] Prevent full stop during DEAD1/DEAD2 states
- [ ] Visual feedback: brief engine flare when stopping
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - SPACE pressed: velocity becomes exactly zero (all components)
  - After full stop: gravity begins pulling ship toward nearest planet
  - Full stop in NORMAL state: works correctly
- **Sad paths:**
  - Full stop in DEAD1/DEAD2 state: ignored, no crash
  - Full stop while already at zero velocity: no-op, no error
  - Full stop during warp: allowed (warp velocity also zeroed)

**Completion Criteria:** All tests pass with ≥90% coverage. Full stop correctly zeros ship velocity and allows gravity to resume.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (velocity zeroed) and sad path (invalid states prevented) tests must pass.**

## Sub-tasks

- [ ] Implement full stop trigger: SPACE key in NORMAL state
- [ ] Set ship velocity to zero instantly
- [ ] Apply gravity after full stop (ship begins falling)
- [ ] Prevent full stop during DEAD1/DEAD2 states
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-2-4-implement-input-axis-mapping-throttle-pitch-yaw-roll](phase-2-4-implement-input-axis-mapping-throttle-pitch-yaw-roll.md)
- [depends-on phase-2-6-implement-gravity-from-planets-on-ship](phase-2-6-implement-gravity-from-planets-on-ship.md)

## History

- type: created
  date: 2026-11-24T04:14:13.447Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
