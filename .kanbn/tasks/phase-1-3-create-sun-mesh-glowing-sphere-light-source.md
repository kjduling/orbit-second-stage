---
tags:
  - 'type:feature'
  - 'phase:1'
  - 'area:scene'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:29:16.075Z
updated: 2026-11-24T04:56:44.457Z
---

# Phase 1.3: Create Sun mesh (glowing sphere, light source)

**Task ID:** 1.3
**Phase:** 1 — Eye Candy (Starfield + Ship)
**Dependencies:** Phase 1.1 (BabylonJS scene initialized)
**Type:** Feature | **Priority:** Medium

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System — Sun as center, light source)
- Consult `design/implementation_plan.md` §5.1 task 1.3 — Sun mesh deliverable
- Consult `design/gap_analysis.md` §2.4 (Original Asset Pipeline — BabylonJS native formats)

---

**Objective:**
Create a glowing sun sphere at the origin of the solar system. The sun serves as both a visual landmark and the primary light source for the scene. Uses BabylonJS standard material with emissive glow.

**Deliverables:**
- Sun mesh: glowing sphere at (0, 0, 0)
- Point light attached to sun for scene illumination

**Subtasks:**
- [ ] Create sphere mesh at origin with radius ~500 game units
- [ ] Apply StandardMaterial with emissive color (yellow-orange glow)
- [ ] Add emissive texture for realistic sun appearance
- [ ] Create BabylonJS PointLight attached to sun position
- [ ] Set light intensity and range appropriately
- [ ] Verify sun visible from any camera position

**Unit Tests Required:** N/A (visual element tested via manual inspection in Phase 1)

**Completion Criteria:** Glowing sun sphere visible at origin with point light illuminating the scene.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (sun renders and lights scene) and sad path (light not attached, wrong position) tests must pass.**

## Sub-tasks

- [ ] Create sphere mesh at origin with appropriate radius
- [ ] Apply StandardMaterial with emissive yellow-orange glow
- [ ] Add emissive texture for realistic sun appearance
- [ ] Create PointLight attached to sun position
- [ ] Verify sun visible from any camera position

## Relations

- [depends-on phase-1-1-initialize-babylonjs-scene](phase-1-1-initialize-babylonjs-scene.md)

## History

- type: created
  date: 2026-11-24T03:29:16.075Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
