---
tags:
  - 'type:feature'
  - 'phase:3'
  - 'area:planets'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T04:15:48.166Z
updated: 2026-11-24T04:58:01.728Z
---

# Phase 3.2: Implement planet rendering in BabylonJS

**Task ID:** 3.2
**Phase:** 3 — Planets & Solar System
**Dependencies:** Phase 3.1 (Planet data structure), Phase 1.1 (BabylonJS scene)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System — planet rendering, sphere meshes with textures)
- Consult `design/implementation_plan.md` §7.1 task 3.2 — Planet rendering
- Consult `design/gap_analysis.md` §3.5 (Coordinate System — Y-up for BabylonJS, Z-up internal)

---

**Objective:**
Render all planets and moons as textured spheres in BabylonJS. Each planet uses a sphere mesh with appropriate texture, scaled to correct relative size. Sun uses emissive material with point light.

**Deliverables:**
- `src/planets/PlanetRenderer.ts` — BabylonJS planet rendering
- `tests/planets/planet-renderer.test.ts` — mesh creation, texture loading tests

**Subtasks:**
- [ ] Implement `PlanetRenderer` class managing planet meshes
- [ ] Create sphere mesh for each planet: radius scaled to game units
- [ ] Apply planet texture from assets/textures/planets/
- [ ] Sun uses emissive material + point light (BabylonJS HemisphericLight)
- [ ] Other planets use standard material with ambient light
- [ ] Implement planet rotation: spin on own axis (theta rate)
- [ ] Implement planet orbit: revolve around parent body
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Planet mesh created with correct radius and segments
  - Texture loaded and applied to planet surface
  - Sun has emissive material and point light
  - Planet rotation updates each frame
  - Planet orbit updates position relative to parent
- **Sad paths:**
  - Missing texture file: falls back to solid color, logs warning
  - Invalid radius (zero): uses minimum radius, logs warning
  - Parent planet not found: planet stays at origin, logs warning

**Completion Criteria:** All tests pass with ≥90% coverage. Planets rendered correctly with textures and lighting.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (meshes created, textures applied) and sad path (missing textures, invalid radii) tests must pass.**

## Sub-tasks

- [ ] Implement PlanetRenderer class managing planet meshes
- [ ] Create sphere mesh for each planet with correct radius
- [ ] Apply planet textures from assets/textures/planets/
- [ ] Sun uses emissive material + point light
- [ ] Implement planet rotation and orbit updates
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-3-1-define-planet-data-structure-initial-solar-system-30-planets-moons](phase-3-1-define-planet-data-structure-initial-solar-system-30-planets-moons.md)
- [depends-on phase-1-1-initialize-babylonjs-scene](phase-1-1-initialize-babylonjs-scene.md)

## History

- type: created
  date: 2026-11-24T04:15:48.166Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
