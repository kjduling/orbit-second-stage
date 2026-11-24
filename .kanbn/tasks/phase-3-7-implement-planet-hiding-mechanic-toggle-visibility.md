---
tags:
  - 'type:feature'
  - 'phase:3'
  - 'area:planets'
  - 'priority:low'
assigned: Qwen
progress: 0
created: 2026-11-24T04:19:02.184Z
updated: 2026-11-24T04:58:02.038Z
---

# Phase 3.7: Implement planet hiding mechanic (toggle visibility)

**Task ID:** 3.7
**Phase:** 3 — Planets & Solar System
**Dependencies:** Phase 3.2 (Planet rendering), Phase 3.6 (Planet labels)
**Type:** Feature | **Priority:** Low

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System — planet hiding for performance)
- Consult `design/implementation_plan.md` §7.1 task 3.7 — Planet hiding
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — hidden planets still affect gravity)

---

**Objective:**
Implement planet hiding mechanic: planets can be toggled invisible for performance or visual clarity. Hidden planets don't render but still affect gravity and can be targeted.

**Deliverables:**
- `src/planets/PlanetVisibility.ts` — planet hide/show logic
- `tests/planets/planet-visibility.test.ts` — hide/show, gravity persistence tests

**Subtasks:**
- [ ] Implement `PlanetVisibility` class managing planet visibility
- [ ] Add toggle method: setPlanetVisible(planetName, visible)
- [ ] Hide planet mesh when invisible (setMeshVisibility(false))
- [ ] Hide planet label when invisible
- [ ] Hidden planets still affect gravity (physics not affected)
- [ ] Hidden planets can still be targeted by weapons
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Planet hidden: mesh and label not rendered
  - Planet visible again: mesh and label rendered correctly
  - Hidden planet still affects gravity: ship accelerates toward it
  - Hidden planet can be targeted: weapons can hit it
- **Sad paths:**
  - Hide non-existent planet: ignored, no crash, logs warning
  - Toggle visibility while planet destroyed: ignored, no crash

**Completion Criteria:** All tests pass with ≥90% coverage. Planet hiding works correctly without affecting physics.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (hidden planets not rendered) and sad path (non-existent planets handled) tests must pass.**

## Sub-tasks

- [ ] Implement PlanetVisibility class managing planet visibility
- [ ] Add toggle method: setPlanetVisible(planetName, visible)
- [ ] Hide planet mesh and label when invisible
- [ ] Hidden planets still affect gravity and can be targeted
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-3-2-implement-planet-rendering-in-babylon-js](phase-3-2-implement-planet-rendering-in-babylon-js.md)
- [depends-on phase-3-6-implement-planet-labels-name-distance](phase-3-6-implement-planet-labels-name-distance.md)

## History

- type: created
  date: 2026-11-24T04:19:02.184Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
