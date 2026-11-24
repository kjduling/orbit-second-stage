---
tags:
  - 'type:feature'
  - 'phase:1'
  - 'area:camera'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:33:47.496Z
updated: 2026-11-24T04:56:44.648Z
---

# Phase 1.6-1.8: Camera wrapper, ship orientation wiring, space junk polygons

**Task ID:** 1.6–1.8 (grouped camera and visual polish tasks)
**Phase:** 1 — Eye Candy (Starfield + Ship)
**Dependencies:** Phase 1.4 (ship model loaded), Phase 1.5 (input manager implemented)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §2.2 (Flight Models — rotation, theta rate 1.6 rad/s), §2.4 (View Locking — three lock modes)
- Consult `design/implementation_plan.md` §5.1 tasks 1.6–1.8, §5.2 (IStarfield, IShipModel interfaces)
- Consult `design/gap_analysis.md` §3.5 (Coordinate System — Y-up for BabylonJS)

---

**Objective:**
Implement camera control around the ship, wire camera to ship orientation with view lock modes, and add subtle space junk reference polygons for motion cues.

**Deliverables:**
- `src/player/Camera.ts` — BabylonJS ArcRotateCamera wrapper
- Camera responds to input (pitch/yaw/roll)
- Space junk: subtle rotating polygons in background

**Subtasks:**
- [ ] **1.6 Camera Wrapper:** Implement camera wrapper around BabylonJS `ArcRotateCamera`
  - Camera orbits ship with pitch, yaw controlled by input
  - Support all three view lock modes (LOCK_ENEMY, LOCK_FRIENDLY, LOCK_PLANET)
  - Implement 'l' key to cycle lock modes
- [ ] **1.7 Wire Camera to Ship:** Connect camera to ship orientation
  - Camera follows ship's view direction
  - Pitch/yaw input rotates camera around ship
  - Roll input banks the camera (rotate around view vector)
- [ ] **1.8 Space Junk:** Add subtle rotating reference polygons in background
  - Subtle rotating reference quads for motion cues
  - Controlled by `junk` preference flag (default enabled)

**Unit Tests Required:**
- **Happy paths (1.6):** Camera rotates with input, lock modes cycle correctly, distance to target maintained
- **Sad paths (1.6):** Invalid lock mode index wraps around, target disappears → camera reverts to free look
- **Happy paths (1.7):** Ship rotation updates camera orientation smoothly
- **Sad paths (1.8):** Space junk disabled via preference → no polygons rendered

**Completion Criteria:** Camera responds to input, orbits ship correctly, space junk visible when enabled.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (camera follows ship) and sad path (invalid lock modes handled) tests must pass.**

## Sub-tasks

- [ ] 1.6: Implement ArcRotateCamera wrapper with pitch/yaw/roll input
- [ ] 1.6: Implement three view lock modes (ENEMY, FRIENDLY, PLANET) with 'l' key cycling
- [ ] 1.7: Wire camera to ship orientation — follow view direction
- [ ] 1.8: Add space junk reference polygons (rotating quads)

## Relations

- [depends-on phase-1-4-load-and-render-original-ship-model-gltf-glb](phase-1-4-load-and-render-original-ship-model-gltf-glb.md)
- [depends-on phase-1-5-implement-input-manager-keyboard-joystick-abstraction](phase-1-5-implement-input-manager-keyboard-joystick-abstraction.md)

## History

- type: created
  date: 2026-11-24T03:33:47.496Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
