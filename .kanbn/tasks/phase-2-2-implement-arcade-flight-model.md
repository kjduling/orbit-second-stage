---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:physics'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:36:09.454Z
updated: 2026-11-24T04:57:25.056Z
---

# Phase 2.2: Implement Arcade flight model

**Task ID:** 2.2
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 2.1 (FlightModel base interface)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §2.2.2 (Arcade Physics — velocity set directly by throttle, no momentum)
- Consult `design/implementation_plan.md` §6.1 task 2.2 — ArcadeFlightModel deliverable
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — arcade model also needs fixed timestep)

---

**Objective:**
Implement the Arcade flight model as an alternative to Newtonian. In arcade mode, velocity is set directly by throttle — no momentum, no inertia. Full stop (SPACE key) sets velocity to zero. Throttle range: 0 to MAX_THROTTLE (10,000 km/s). Warp: MAX_WARP_THROTTLE (1,000,000 km/s).

**Deliverables:**
- `src/player/ArcadeFlightModel.ts` — Arcade physics implementation
- Same test suite as Newtonian (polymorphic tests)

**Subtasks:**
- [ ] Implement ArcadeFlightModel implementing IFlightModel interface
- [ ] Velocity set directly by throttle: `vel = direction × throttle × MAX_THROTTLE`
- [ ] Throttle range: 0 to 10,000 km/s (converted to game units)
- [ ] Warp throttle: up to 1,000,000 km/s when warpspeed enabled
- [ ] Full stop (SPACE key): velocity = 0 instantly
- [ ] Rotation same as Newtonian: theta rate 1.6 rad/s
- [ ] Apply gravity same as Newtonian (shared method)
- [ ] Write polymorphic tests (same test suite works for both models)

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Throttle 50%: velocity = 0.5 × MAX_THROTTLE in view direction
  - Full stop: velocity becomes exactly zero
  - Warp enabled: throttle can reach MAX_WARP_THROTTLE
  - Rotation identical to Newtonian at same input
- **Sad paths:**
  - Throttle input clamped to [0, 1] range
  - Gravity applied but doesn't overcome direct velocity setting
  - Reverse thrust (S key): velocity set negative in view direction

**Completion Criteria:** All tests pass with ≥90% coverage. Arcade model runs alongside Newtonian via same IFlightModel interface.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (velocity set correctly) and sad path (clamping, gravity interaction) tests must pass.**

## Sub-tasks

- [ ] Implement ArcadeFlightModel class implementing IFlightModel
- [ ] Velocity set directly by throttle × MAX_THROTTLE
- [ ] Throttle range 0 to MAX_THROTTLE (10,000 km/s)
- [ ] Warp throttle up to MAX_WARP_THROTTLE (1,000,000 km/s)
- [ ] Full stop (SPACE): velocity = 0 instantly
- [ ] Write polymorphic tests shared with Newtonian model

## Relations

- [depends-on phase-2-1-implement-flight-model-base-interface-newtonian-impl](phase-2-1-implement-flight-model-base-interface-newtonian-impl.md)

## History

- type: created
  date: 2026-11-24T03:36:09.454Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
