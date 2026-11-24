---
tags:
  - 'type:feature'
  - 'phase:1'
  - 'area:input'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:31:22.654Z
updated: 2026-11-24T04:56:44.603Z
---

# Phase 1.5: Implement InputManager (keyboard + joystick abstraction)

**Task ID:** 1.5
**Phase:** 1 — Eye Candy (Starfield + Ship)
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §2.3 (Movement Controls — full keyboard and joystick mapping)
- Consult `design/implementation_plan.md` §5.1 task 1.5, §5.2 (IInputManager interface)
- Consult `design/gap_analysis.md` §3.1 (Joystick & Mouse Configuration — deadzone, throttle mapping)

---

**Objective:**
Implement an input abstraction layer that normalizes keyboard and joystick input into a unified interface. This is the bridge between raw browser/WebGL input events and game logic.

**Deliverables:**
- `src/player/InputManager.ts` — IInputManager interface + KeyboardInputImpl
- `tests/player/input-manager.test.ts` — key state tracking, axis mapping tests

**Subtasks:**
- [ ] Define `IInputManager` interface: getKeyState, getAxis, isKeyPressed, isKeyDown
- [ ] Key state bit format: bit7=down, bit0=pressed-this-frame (GLUT-compatible)
- [ ] Implement keyboard input handler using BabylonJS `onKeyboardObservable`
- [ ] Map keys to game actions: W/S (forward/back), A/D (yaw), Q/E (pitch), Z/X (roll), Tab (fire), 1-4 (weapons), F (flight model toggle), etc.
- [ ] Implement joystick input via BabylonJS `OnControllerConnectedObservable`
- [ ] Map joystick axes: X→yaw, Y→pitch, R→roll, Z→throttle
- [ ] Implement deadzone (default 0.1) and throttle mapping (50-100%)
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Key pressed: isKeyPressed('Tab') returns true for one frame, then bit7 stays set
  - Key held: isKeyDown('W') returns true every frame while held
  - Joystick X axis maps to yaw with deadzone applied
  - Throttle axis maps to forward thrust with configurable range
- **Sad paths:**
  - Unknown key: getKeyState returns 0 without error
  - Joystick disconnected: no crash, axes return 0
  - Deadzone filters small joystick values (values < 0.1 → 0)

**Completion Criteria:** All tests pass with ≥90% coverage. InputManager correctly tracks keyboard and joystick state.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (key/joystick state correct) and sad path (unknown keys, disconnected joystick) tests must pass.**

## Sub-tasks

- [ ] Define IInputManager interface with getKeyState, getAxis, isKeyPressed, isKeyDown
- [ ] Implement key state bit format: bit7=down, bit0=pressed-this-frame
- [ ] Implement keyboard handler via BabylonJS onKeyboardObservable
- [ ] Map all game keys (WASDQEZX, Tab, 1-4, F, etc.)
- [ ] Implement joystick input with deadzone and throttle mapping
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:31:22.654Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
