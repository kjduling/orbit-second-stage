---
tags:
  - 'type:feature'
  - 'phase:1'
  - 'area:scene'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:29:16.050Z
updated: 2026-11-24T04:56:44.377Z
---

# Phase 1.2: Implement Starfield (65K+ stars, two density levels)

**Task ID:** 1.2
**Phase:** 1 — Eye Candy (Starfield + Ship)
**Dependencies:** Phase 1.1 (BabylonJS scene initialized)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §9.4 (Starfield — two levels: dense ~65K stars, sparse ~6.5K stars)
- Consult `design/implementation_plan.md` §5.1 task 1.2, §5.2 (IStarfield interface)
- Consult `design/gap_analysis.md` §3.2 (Starfield data source — Raul Alonso's ssystem stellar database)

---

**Objective:**
Implement a starfield renderer with 65,000+ stars (dense) and 6,500+ stars (sparse). Stars have magnitudes mapped to brightness, rendered as points with depth sorting (brightest last). Based on Raul Alonso's ssystem stellar database.

**Deliverables:**
- `src/planets/Starfield.ts` — starfield implementation with BabylonJS point rendering
- `tests/planets/starfield.test.ts` — star count, brightness mapping, culling tests

**Subtasks:**
- [ ] Define `IStarfield` interface (create, setDensity, updateCamera)
- [ ] Implement star data structure: position (x,y,z), magnitude → brightness mapping
- [ ] Load star data from ssystem stellar database format (or create synthetic dataset)
- [ ] Create BabylonJS point cloud with `CreatePointsCloud` or `InstancedMesh` for performance
- [ ] Map magnitudes to vertex colors (brightness): brighter stars = whiter, dimmer = redder
- [ ] Implement density levels: sparse (~6.5K stars), dense (~65K stars)
- [ ] Depth sorting: brightest stars rendered last (on top)
- [ ] Distance culling: don't render stars behind camera
- [ ] Implement `setDensity(density)` — switch between sparse/dense
- [ ] Implement `updateCamera(camera)` — reposition starfield to follow camera
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Dense mode creates ~65K stars, sparse mode creates ~6.5K stars
  - Brightest star has RGB(1,1,1), dimmest has reddish tint
  - Stars behind camera are culled (not rendered)
  - Density switch updates star count correctly
- **Sad paths:**
  - Empty star data: renders nothing without crashing
  - Invalid magnitude value: clamped to valid range
  - Camera at origin: stars still render correctly

**Completion Criteria:** All tests pass with ≥90% coverage. Starfield visible in scene with depth sorting and density levels.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct star count, brightness) and sad path (empty data, invalid magnitudes) tests must pass.**

## Sub-tasks

- [ ] Define IStarfield interface with create, setDensity, updateCamera
- [ ] Implement star data structure: position + magnitude-to-brightness mapping
- [ ] Load/create star dataset (65K dense, 6.5K sparse)
- [ ] Create BabylonJS point cloud with vertex colors for brightness
- [ ] Implement density levels and depth sorting (brightest last)
- [ ] Implement distance culling and camera follow
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-1-1-initialize-babylonjs-scene](phase-1-1-initialize-babylonjs-scene.md)

## History

- type: created
  date: 2026-11-24T03:29:16.050Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
