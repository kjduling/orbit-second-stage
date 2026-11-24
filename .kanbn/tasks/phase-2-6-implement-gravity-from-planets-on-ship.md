---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:physics'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:14:13.386Z
updated: 2026-11-24T04:57:25.305Z
---

# Phase 2.6: Implement gravity from planets on ship

**Task ID:** 2.6
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 2.1 (FlightModel base), Phase 3.1 (Planets data structure)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System — gravitational pull from planets)
- Consult `design/implementation_plan.md` §6.1 task 2.6 — Gravity implementation
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — gravity must be computed deterministically)

---

**Objective:**
Implement gravitational attraction between the ship and all planets/moons. Gravity follows inverse-square law: F = G × m₁ × m₂ / r². Each planet's mass is radius³. Ship accelerates toward nearest massive body.

**Deliverables:**
- `src/physics/GravityCalculator.ts` — gravitational force calculation
- `tests/physics/gravity-calculator.test.ts` — gravity computation, edge cases tests

**Subtasks:**
- [ ] Implement `GravityCalculator` class with planet list
- [ ] Compute gravitational force: F = G × m₁ × m₂ / r² for each body
- [ ] Use mass = radius³ for all bodies (simplified from real masses)
- [ ] Sum gravitational forces from all planets/moons
- [ ] Apply net gravitational acceleration to ship velocity
- [ ] Implement gravity culling: ignore bodies beyond 100× ship distance
- [ ] Implement minimum force threshold: ignore gravity below threshold
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Single planet: ship accelerates toward planet at G × m_ship × m_planet / r²
  - Multiple planets: forces sum correctly (vector addition)
  - Ship between two planets: net force points toward stronger body
  - Gravity culling: bodies beyond 100× distance ignored
- **Sad paths:**
  - Ship at planet center (r = 0): force capped to prevent singularity
  - No planets in system: no gravitational acceleration applied
  - Planet with zero mass: no force applied, no division by zero

**Completion Criteria:** All tests pass with ≥90% coverage. Gravity correctly computed and applied to ship velocity.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (gravity toward planets) and sad path (singularity, zero mass) tests must pass.**

## Sub-tasks

- [ ] Implement GravityCalculator class with planet list
- [ ] Compute gravitational force: F = G × m₁ × m₂ / r²
- [ ] Use mass = radius³ for all bodies
- [ ] Sum gravitational forces from all planets/moons
- [ ] Apply net gravitational acceleration to ship velocity
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-2-1-implement-flight-model-base-interface-newtonian-impl](phase-2-1-implement-flight-model-base-interface-newtonian-impl.md)
- [depends-on phase-3-1-define-planet-data-structure-initial-solar-system-30-planets-moons](phase-3-1-define-planet-data-structure-initial-solar-system-30-planets-moons.md)

## History

- type: created
  date: 2026-11-24T04:14:13.386Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
