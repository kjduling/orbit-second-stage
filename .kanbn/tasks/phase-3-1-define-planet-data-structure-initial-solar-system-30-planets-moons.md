---
tags:
  - 'type:feature'
  - 'phase:3'
  - 'area:planets'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:38:24.905Z
updated: 2026-11-24T04:58:01.655Z
---

# Phase 3.1: Define Planet data structure + initial solar system (30+ planets/moons)

**Task ID:** 3.1
**Phase:** 3 — Planets & Solar System
**Dependencies:** Phase 2 (Flight Physics), Phase 0.4 (Vector3)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System — full planet structure, NASA-based positions)
- Consult `design/implementation_plan.md` §7 (Phase 3), §7.1 task 3.1, §7.2 (IPlanet interface)
- Consult `design/gap_analysis.md` §3.5 (Coordinate System — Z-up internal for planets)

---

**Objective:**
Define the Planet data structure and initialize the full solar system with 30+ planets and moons based on NASA data. The Sun is at the center, 8 major planets orbit it, plus ~20 moons orbiting their parent planets.

**Deliverables:**
- `src/planets/Planet.ts` — Planet entity class
- `src/planets/SolarSystem.ts` — SolarSystem with all planets/moons
- `tests/planets/solar-system.test.ts` — planet count, positions, masses tests

**Subtasks:**
- [ ] Define `IPlanet` interface: name, pos, vel, radius, mass, color, texturePath, isMoon, primary, rings, hidden, theta
- [ ] Implement Planet class with all properties and update/render methods
- [ ] Initialize Sun at origin (radius ~696,000 km, mass = radius³)
- [ ] Add 8 major planets: Mercury through Neptune with NASA-based positions
- [ ] Add ~20 moons: Phobos, Deimos, Io, Europa, Titan, Triton, etc.
- [ ] Compute gravitational mass for each body (mass = radius³)
- [ ] Set orbital parameters: angvel, theta (current orbital angle), primary reference
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Solar system contains exactly 30+ bodies (1 sun + 8 planets + ~20 moons)
  - Planet positions match NASA data within tolerance
  - Moon primary references point to correct parent planet
  - Mass computed correctly: mass = radius³
  - Orbital angle theta starts at 0
- **Sad paths:**
  - Duplicate planet name: throws error
  - Invalid radius (negative): clamped to positive value
  - Moon without primary: throws descriptive error

**Completion Criteria:** All tests pass with ≥90% coverage. Solar system initialized with all planets and moons.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct planet count/positions) and sad path (duplicate names, invalid data) tests must pass.**

## Sub-tasks

- [ ] Define IPlanet interface with all required properties
- [ ] Implement Planet class with update/render methods
- [ ] Initialize Sun at origin with correct mass
- [ ] Add 8 major planets with NASA-based positions
- [ ] Add ~20 moons with parent planet references
- [ ] Compute gravitational mass (radius³) for all bodies
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-2-3-implement-ship-state-machine-with-shields-death-timer](phase-2-3-implement-ship-state-machine-with-shields-death-timer.md)
- [depends-on phase-0-4-implement-vector3-z-up-internal-coordinate-system](phase-0-4-implement-vector3-z-up-internal-coordinate-system.md)

## History

- type: created
  date: 2026-11-24T03:38:24.905Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
