---
progress: 0
created: 2026-11-24T17:11:06.010Z
updated: 2026-11-24T17:19:10.136Z
---

# Phase 1.0: Quick visual prototype — sun, planets, ship wireframe on screen

**Task ID:** 1.0
**Phase:** 1 — Eye Candy (Early Visual Feedback)
**Dependencies:** Phase 1.1 (BabylonJS scene initialized)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §6 (Planets & Solar System), §2.1 (Player Ship)
- Consult `design/implementation_plan.md` §5 (Phase 1 — Eye Candy)

---

**Objective:**
Get something visually interesting on screen *fast* — not just a black screen. Create a quick prototype scene with:
- A glowing yellow sphere for the Sun (emissive material, no light yet)
- Colored spheres for Mercury through Neptune at approximate relative distances
- A simple wireframe or basic mesh for the player ship at origin
- Stars as small white points (no density levels yet)

This is NOT production quality — it's "can I see the solar system in the browser" level. The goal is visual gratification and iteration speed.

**Deliverables:**
- `src/prototype/QuickScene.ts` — prototype scene builder
- Browser shows recognizable solar system layout with ship at center

**Subtasks:**
- [ ] Create sun: yellow sphere with emissive material (`MeshBuilder.CreateSphere`)
- [ ] Create 8 planet spheres with distinct colors (no textures yet)
- [ ] Place planets at scaled distances (compress real distances for readability)
- [ ] Create basic ship mesh: box or wireframe at origin, oriented forward
- [ ] Add point light at sun position
- [ ] Add starfield: 500 white points (not 65K yet — just enough to feel like space)
- [ ] Add ArcRotateCamera positioned so player can see the scene
- [ ] Verify in browser: solar system visible, ship at center

**Unit Tests Required:** N/A (prototype — visual inspection only; formal tests come in Phase 1.2+)

**Completion Criteria:** Browser shows a recognizable solar system with sun, planets, stars, and ship. Player can rotate camera around the scene.

---

⚠️ **This is a prototype task — visual gratification over correctness. Get something on screen fast.**

## Sub-tasks

- [ ] Create sun: yellow sphere with emissive material
- [ ] Create 8 planet spheres with distinct colors at scaled distances
- [ ] Create basic ship mesh (box/wireframe) at origin
- [ ] Add point light at sun position
- [ ] Add starfield: 500 white points
- [ ] Add ArcRotateCamera for scene exploration
- [ ] Verify in browser — solar system visible and navigable

## Relations

- [depends-on phase-1-1-initialize-babylon-js-scene](phase-1-1-initialize-babylon-js-scene.md)

## History

- type: created
  date: 2026-11-24T17:11:06.010Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
