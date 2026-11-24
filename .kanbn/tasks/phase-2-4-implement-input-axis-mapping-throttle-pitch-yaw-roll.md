---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:input'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:11:43.845Z
updated: 2026-11-24T04:57:25.251Z
---

# Phase 2.4: Implement input axis mapping (throttle, pitch, yaw, roll)

**Task ID:** 2.4
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 1.5 (InputManager), Phase 2.3 (Ship state machine)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §2.1 (Player Ship — controls: W/S throttle, A/D pitch, Q/E yaw, R/F roll, SPACE full stop)
- Consult `design/implementation_plan.md` §6.1 task 2.4 — Input axis mapping
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — input must be frame-independent)

---

**Objective:**
Map raw input from InputManager to flight control axes: throttle (W/S), pitch (A/D or joystick Y), yaw (Q/E or joystick X), roll (R/F or joystick roll). These axes feed directly into the FlightModel's update method.

**Deliverables:**
- `src/player/InputAxisMapper.ts` — maps InputManager output to flight axes
- `tests/player/input-axis-mapper.test.ts` — axis mapping, deadzone tests

**Subtasks:**
- [ ] Define `FlightAxes` interface: throttle, pitch, yaw, roll, fullStop, fire
- [ ] Implement `InputAxisMapper` class wrapping InputManager
- [ ] Map W/S keys to throttle axis (-1.0 to +1.0)
- [ ] Map A/D keys to pitch axis (negative = nose down)
- [ ] Map Q/E keys to yaw axis (negative = left)
- [ ] Map R/F keys to roll axis (negative = left roll)
- [ ] Map SPACE key to fullStop flag
- [ ] Map FIRE key to fire flag
- [ ] Apply joystick deadzone: ignore inputs below threshold (0.15)
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - W key pressed: throttle = +1.0
  - S key pressed: throttle = -1.0
  - A key pressed: pitch = -1.0 (nose down)
  - D key pressed: pitch = +1.0 (nose up)
  - Q key pressed: yaw = -1.0 (left)
  - E key pressed: yaw = +1.0 (right)
  - R key pressed: roll = -1.0 (left)
  - F key pressed: roll = +1.0 (right)
  - SPACE pressed: fullStop = true
  - FIRE pressed: fire = true
  - Joystick input: mapped to axes with deadzone applied
- **Sad paths:**
  - Joystick input below deadzone (0.15): treated as zero
  - Both W and S pressed: throttle = 0 (cancel out)
  - No input: all axes = 0, flags = false

**Completion Criteria:** All tests pass with ≥90% coverage. Input correctly mapped to flight control axes.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct axis mapping) and sad path (deadzone, conflicting inputs) tests must pass.**

## Sub-tasks

- [ ] Define FlightAxes interface: throttle, pitch, yaw, roll, fullStop, fire
- [ ] Implement InputAxisMapper class wrapping InputManager
- [ ] Map W/S to throttle axis (-1.0 to +1.0)
- [ ] Map A/D to pitch axis (negative = nose down)
- [ ] Map Q/E to yaw axis, R/F to roll axis
- [ ] Map SPACE to fullStop, FIRE to fire flag
- [ ] Apply joystick deadzone: ignore inputs below 0.15
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-1-5-implement-input-manager-keyboard-joystick-abstraction](phase-1-5-implement-input-manager-keyboard-joystick-abstraction.md)
- [depends-on phase-2-3-implement-ship-state-machine-with-shields-death-timer](phase-2-3-implement-ship-state-machine-with-shields-death-timer.md)

## History

- type: created
  date: 2026-11-24T04:11:43.845Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
