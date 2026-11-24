---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:physics'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:33:47.517Z
updated: 2026-11-24T04:57:24.989Z
---

# Phase 2.1: Implement FlightModel base interface + Newtonian impl

**Task ID:** 2.1
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 1.5 (InputManager), Phase 0.4-0.6 (Vector3, Matrix4, Timer)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §2.2.1 (Newtonian Physics — velocity persistent, delta-v accumulation, warp speed)
- Consult `design/implementation_plan.md` §6 (Phase 2), §6.1 task 2.1, §6.2 (IFlightModel interface)
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics for netcode — fixed timestep)

---

**Objective:**
Implement the Newtonian flight model — the core physics system where velocity is persistent and momentum matters. This is the primary flight model in the original game. Key constants: DELTAV = 0.2 units/s², THETA = 1.6 rad/s, WARP_COEFF = 0.2.

**Deliverables:**
- `src/player/IFlightModel.ts` — base interface for all flight models
- `src/player/NewtonianFlightModel.ts` — Newtonian physics implementation
- `tests/player/flight-model.test.ts` — delta-v accumulation, warp speed, gravity tests

**Subtasks:**
- [ ] Define `IFlightModel` interface: type, update(ship, deltaT, input), applyGravity(ship, gravity)
- [ ] Implement NewtonianFlightModel with persistent velocity
- [ ] Implement thrust acceleration: `deltav = view_direction × DELTAV × deltaT` per second
- [ ] Implement DELTAV constant: 0.2 units/second²
- [ ] Implement warp speed: `deltaT × (move_forward × DELTAV + WARP_COEFF × |velocity|)`
- [ ] Implement superwarp (single-player only): `100.0 × move_forward × DELTAV × DELTAV`
- [ ] Implement rotation: theta rate 1.6 rad/s at full turn rate, pitch/yaw/roll via input axes
- [ ] Implement `applyGravity(ship, gravity)`: add gravitational force to ship velocity
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Thrust: ship accelerates at 0.2 units/s² in view direction
  - Warp: velocity approaches warp speed with WARP_COEFF accumulation
  - Superwarp: single-player mode achieves 100× DELTAV acceleration
  - Rotation: full pitch input rotates view direction by 1.6 rad/s
  - Gravity: ship accelerates toward planet proportional to mass/r²
- **Sad paths:**
  - Zero thrust (no input): velocity unchanged (momentum preserved)
  - Reverse thrust (S key): decelerates in opposite direction
  - Gravity at infinite distance: force approaches zero (no singularity)

**Completion Criteria:** All tests pass with ≥90% coverage. Newtonian flight model correctly implements persistent velocity, delta-v accumulation, warp speed, and gravity.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct physics) and sad path (edge cases handled) tests must pass.**

## Sub-tasks

- [ ] Define IFlightModel interface with type, update, applyGravity
- [ ] Implement NewtonianFlightModel with persistent velocity
- [ ] Implement thrust acceleration: view_direction × DELTAV × deltaT
- [ ] Implement warp speed with WARP_COEFF accumulation
- [ ] Implement superwarp (SP only): 100× DELTAV²
- [ ] Implement rotation: theta rate 1.6 rad/s, pitch/yaw/roll
- [ ] Implement applyGravity with mass/r² formula
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-1-5-implement-input-manager-keyboard-joystick-abstraction](phase-1-5-implement-input-manager-keyboard-joystick-abstraction.md)
- [depends-on phase-0-4-implement-vector3-z-up-internal-coordinate-system](phase-0-4-implement-vector3-z-up-internal-coordinate-system.md)
- [depends-on phase-0-6-implement-timer-fixed-timestep-accumulator](phase-0-6-implement-timer-fixed-timestep-accumulator.md)

## History

- type: created
  date: 2026-11-24T03:33:47.517Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
