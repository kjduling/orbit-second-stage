---
tags:
  - 'type:feature'
  - 'phase:3'
  - 'area:planets'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:17:27.044Z
updated: 2026-11-24T04:58:01.932Z
---

# Phase 3.4: Implement planetary orbits (angular velocity, theta)

**Task ID:** 3.4
**Phase:** 3 — Planets & Solar System
**Dependencies:** Phase 3.2 (Planet rendering), Phase 3.3 (Rings)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System — orbital mechanics, angular velocity, theta)
- Consult `design/implementation_plan.md` §7.1 task 3.4 — Planetary orbits
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — orbits must be deterministic)

---

**Objective:**
Implement planetary orbital motion: each planet revolves around its parent body at a constant angular velocity (angvel). Position computed from orbital radius and theta (current angle). Moons orbit their parent planets.

**Deliverables:**
- `src/planets/OrbitalMechanics.ts` — orbital position computation
- `tests/planets/orbital-mechanics.test.ts` — orbit computation, period tests

**Subtasks:**
- [ ] Implement `OrbitalMechanics` class computing planet positions
- [ ] Compute orbital position: x = r × cos(theta), z = r × sin(theta) (Z-up)
- [ ] Update theta each frame: theta += angvel × deltaT
- [ ] Handle parent body movement: moon position relative to moving parent
- [ ] Implement Kepler's third law: angvel proportional to 1/√(r³) for realistic orbits
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Planet orbit: position computed correctly from radius and theta
  - Moon orbit: position relative to parent planet's current position
  - Theta update: increases by angvel × deltaT each frame
  - Full orbit: after 2π/angvel seconds, planet returns to starting position
- **Sad paths:**
  - Planet without parent (Sun): stays at origin, no orbit
  - Zero angular velocity: theta doesn't change, planet stationary
  - Negative radius: clamped to positive value, logs warning

**Completion Criteria:** All tests pass with ≥90% coverage. Planetary orbits computed correctly with proper theta updates.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (orbits correct) and sad path (zero angvel, negative radius) tests must pass.**

## Sub-tasks

- [ ] Implement OrbitalMechanics class computing planet positions
- [ ] Compute orbital position: x = r × cos(theta), z = r × sin(theta)
- [ ] Update theta each frame: theta += angvel × deltaT
- [ ] Handle parent body movement for moons
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-3-2-implement-planet-rendering-in-babylon-js](phase-3-2-implement-planet-rendering-in-babylon-js.md)
- [depends-on phase-3-3-implement-planet-rings-saturn-like](phase-3-3-implement-planet-rings-saturn-like.md)

## History

- type: created
  date: 2026-11-24T04:17:27.044Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
