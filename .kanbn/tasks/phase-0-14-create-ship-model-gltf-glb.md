---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:assets'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:06:19.089Z
updated: 2026-11-24T04:55:58.061Z
---

# Phase 0.14: Create ship model (GLTF/GLB)

**Task ID:** 0.14
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (Project scaffolding)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §3 (Weapons System — ship model reference), §2.1 (Player Ship — visual description)
- Consult `design/implementation_plan.md` §4.2 task 0.14 — Create ship model deliverable
- Consult `design/gap_analysis.md` §9.2 (Asset Pipeline — GLTF/GLB format for BabylonJS)

---

**Objective:**
Create the player ship model in GLTF/GLB format. This is the visual representation of the player's spacecraft — a sleek, angular fighter design. Model will be loaded by Phase 1.4.

**Deliverables:**
- `assets/models/ship.glb` — player ship model in GLTF binary format
- `assets/textures/ship_*.png` — ship textures (if PBR workflow used)

**Subtasks:**
- [ ] Model ship geometry: angular fighter design, ~500 triangles
- [ ] UV unwrap for texture mapping
- [ ] Create PBR materials: hull, cockpit, engines, weapons mounts
- [ ] Export as GLB (binary GLTF) for optimal loading
- [ ] Verify model loads correctly in BabylonJS sandbox
- [ ] Optimize triangle count for web performance

**Unit Tests Required:** N/A (asset creation verified manually in Phase 1.4)

**Completion Criteria:** Ship model exports as valid GLB file, loads in BabylonJS without errors.

---

⚠️ **This task is NOT complete without verification. Model must load correctly in BabylonJS sandbox before moving to Done.**

## Sub-tasks

- [ ] Model ship geometry: angular fighter design, ~500 triangles
- [ ] UV unwrap for texture mapping
- [ ] Create PBR materials: hull, cockpit, engines, weapons
- [ ] Export as GLB (binary GLTF) format
- [ ] Verify model loads in BabylonJS sandbox

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T04:06:19.089Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
