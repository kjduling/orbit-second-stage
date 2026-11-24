---
tags:
  - 'type:feature'
  - 'phase:1'
  - 'area:assets'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:31:22.636Z
updated: 2026-11-24T04:56:44.505Z
---

# Phase 1.4: Load and render original ship model (GLTF/GLB)

**Task ID:** 1.4
**Phase:** 1 — Eye Candy (Starfield + Ship)
**Dependencies:** Phase 0.14 (assets created), Phase 1.1 (BabylonJS scene)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §2.1 (Player Ship — attributes, state machine)
- Consult `design/implementation_plan.md` §5.1 task 1.4, §5.2 (IShipModel interface), §9.1 (Model Formats)
- Consult `design/gap_analysis.md` §2.4 (Original Asset Pipeline — GLTF/GLB format)

---

**Objective:**
Load and render an original ship model (GLTF/GLB) in the BabylonJS scene. The ship is "Sparky" — humanity's top space fighter pilot. Supports multiple ship types: light1, light2, medium1, heavy1, fight1.

**Deliverables:**
- `src/player/ShipModel.ts` — GLTF/GLB model loader for ships
- `tests/core/model-loader.test.ts` — GLTF parsing, bounding box, material setup tests

**Subtasks:**
- [ ] Implement `IShipModel` interface (load, getBoundingInfo, setOrientation)
- [ ] Use BabylonJS `ImportMeshAsync` to load GLTF/GLB ship model
- [ ] Compute bounding box for collision detection (LODbound/hibound)
- [ ] Set up materials from GLTF mesh (preserve original colors from .tri hex values)
- [ ] Implement `setOrientation(viewDir, upDir)` — apply rotation via BabylonJS meshes
- [ ] Support multiple ship types: light1, light2, medium1, heavy1, fight1
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - GLTF model loads successfully with all meshes and materials
  - Bounding box computed correctly (lobound/hibound match model extents)
  - Orientation set via BabylonJS mesh rotation matches expected transform
  - Multiple ship types load without error
- **Sad paths:**
  - Missing GLTF file: throws descriptive error with available models listed
  - Corrupt GLTF file: caught by BabylonJS loader, error propagated
  - Invalid model path: graceful fallback to default ship

**Completion Criteria:** All tests pass. Ship model loads and renders in scene with correct orientation.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (model loads correctly) and sad path (missing/corrupt files handled) tests must pass.**

## Sub-tasks

- [ ] Define IShipModel interface with load, getBoundingInfo, setOrientation
- [ ] Implement GLTF/GLB model loading via ImportMeshAsync
- [ ] Compute bounding box for collision detection
- [ ] Set up materials preserving original colors
- [ ] Implement setOrientation with BabylonJS mesh rotation
- [ ] Support multiple ship types (light1, light2, medium1, heavy1, fight1)
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-14-0-17-source-create-all-original-game-assets-models-textures-sounds-music](phase-0-14-0-17-source-create-all-original-game-assets-models-textures-sounds-music.md)
- [depends-on phase-1-1-initialize-babylonjs-scene](phase-1-1-initialize-babylonjs-scene.md)

## History

- type: created
  date: 2026-11-24T03:31:22.636Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
