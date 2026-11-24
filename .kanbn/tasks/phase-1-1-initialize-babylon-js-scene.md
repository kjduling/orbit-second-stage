---
tags:
  - 'type:feature'
  - 'phase:1'
  - 'area:scene'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:29:16.003Z
updated: 2026-11-24T04:56:44.349Z
---

# Phase 1.1: Initialize BabylonJS scene

**Task ID:** 1.1
**Phase:** 1 — Eye Candy (Starfield + Ship)
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §16.1 (Architecture Recommendations — BabylonJS adaptations)
- Consult `design/implementation_plan.md` §5 (Phase 1 — Eye Candy), §5.1 task 1.1
- Consult `design/gap_analysis.md` §3.4 (Original Asset Pipeline — BabylonJS native formats)

---

**Objective:**
Initialize a BabylonJS scene that renders a black screen. This is the canvas upon which everything else will be drawn. Minimal but essential — no stars, no ship, just the scene infrastructure.

**Deliverables:**
- `src/index.ts` — BabylonJS scene initialization entry point
- Scene renders a black/clear screen in the browser

**Subtasks:**
- [ ] Import BabylonJS core engine and scene modules
- [ ] Create `Engine` instance with antialiasing enabled
- [ ] Create `Scene` instance attached to the engine
- [ ] Set clear color to black (or dark space color)
- [ ] Implement render loop: `engine.runRenderLoop()`
- [ ] Handle window resize: update engine canvas size
- [ ] Verify scene renders in browser (black screen = success!)

**Unit Tests Required:** N/A (BabylonJS rendering tested via visual inspection; integration test in Phase 5)

**Completion Criteria:** Browser displays a black/dark screen with BabylonJS canvas. No errors in console.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (scene renders) and sad path (canvas not found, WebGL unavailable) tests must pass.**

## Sub-tasks

- [ ] Import BabylonJS core engine and scene modules
- [ ] Create Engine instance with antialiasing enabled
- [ ] Create Scene instance with dark clear color
- [ ] Implement render loop with engine.runRenderLoop()
- [ ] Handle window resize to update canvas size
- [ ] Verify scene renders black screen in browser

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:29:16.003Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
