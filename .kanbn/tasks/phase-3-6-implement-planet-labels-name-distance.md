---
tags:
  - 'type:feature'
  - 'phase:3'
  - 'area:ui'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T04:19:02.167Z
updated: 2026-11-24T04:58:01.994Z
---

# Phase 3.6: Implement planet labels (name, distance)

**Task ID:** 3.6
**Phase:** 3 — Planets & Solar System
**Dependencies:** Phase 3.2 (Planet rendering), Phase 3.4 (Orbital mechanics)
**Type:** Feature | **Priority:** Medium

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System — planet labels, distance display)
- Consult `design/implementation_plan.md` §7.1 task 3.6 — Planet labels
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — label positions must update deterministically)

---

**Objective:**
Implement planet labels showing planet name and distance from player. Labels rendered as HTML overlays or BabylonJS sprite text, positioned at planet screen coordinates.

**Deliverables:**
- `src/planets/PlanetLabel.ts` — planet label rendering
- `tests/planets/planet-label.test.ts` — label positioning, visibility tests

**Subtasks:**
- [ ] Implement `PlanetLabel` class managing label sprites
- [ ] Render planet name: "Earth", "Mars", etc. in readable font
- [ ] Render distance: "1.2 AU" or "45000 km" from player
- [ ] Position labels at planet screen coordinates (project 3D to 2D)
- [ ] Hide label if planet behind camera or too far away
- [ ] Scale label size based on distance (closer = larger)
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Label text shows correct planet name
  - Distance computed correctly: player position to planet position
  - Label positioned at correct screen coordinates
  - Label visible when planet in front of camera
- **Sad paths:**
  - Planet behind camera: label hidden, no crash
  - Planet too far away: label hidden (distance threshold)
  - Planet off-screen: label hidden

**Completion Criteria:** All tests pass with ≥90% coverage. Planet labels rendered correctly with name and distance.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (labels visible, positioned correctly) and sad path (behind camera, too far) tests must pass.**

## Sub-tasks

- [ ] Implement PlanetLabel class managing label sprites
- [ ] Render planet name: 'Earth', 'Mars', etc.
- [ ] Render distance: '1.2 AU' or '45000 km' from player
- [ ] Position labels at planet screen coordinates (project 3D to 2D)
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-3-2-implement-planet-rendering-in-babylon-js](phase-3-2-implement-planet-rendering-in-babylon-js.md)
- [depends-on phase-3-4-implement-planetary-orbits-angular-velocity-theta](phase-3-4-implement-planetary-orbits-angular-velocity-theta.md)

## History

- type: created
  date: 2026-11-24T04:19:02.167Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
