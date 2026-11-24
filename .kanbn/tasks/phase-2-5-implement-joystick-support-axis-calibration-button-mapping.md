---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:input'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:11:43.861Z
updated: 2026-11-24T04:57:25.278Z
---

# Phase 2.5: Implement joystick support (axis calibration, button mapping)

**Task ID:** 2.5
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 1.5 (InputManager), Phase 2.4 (Input axis mapping)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §2.1 (Player Ship — joystick support, axis calibration)
- Consult `design/implementation_plan.md` §6.1 task 2.5 — Joystick support
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — joystick input must be frame-independent)

---

**Objective:**
Implement full joystick/gamepad support: axis detection, calibration, button mapping, and vibration feedback. Supports standard USB gamepads via Web Gamepad API and native HID.

**Deliverables:**
- `src/player/JoystickManager.ts` — joystick detection, calibration, button mapping
- `tests/player/joystick-manager.test.ts` — axis reading, button mapping tests

**Subtasks:**
- [ ] Implement `JoystickManager` class: scan, connect, disconnect
- [ ] Detect connected joysticks via Web Gamepad API / HID
- [ ] Read joystick axes: throttle (LT/RT), pitch (left stick Y), yaw (left stick X), roll (right stick X)
- [ ] Read joystick buttons: fire (A/X), full stop (B), warp (Y), menu (Start)
- [ ] Implement axis calibration: auto-deadzone detection, center point
- [ ] Implement button mapping: configurable key-to-button assignments
- [ ] Implement vibration feedback: rumble on weapon fire, shield hit
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Joystick connected: detected and listed in available devices
  - Axis reading: throttle axis returns correct value (-1.0 to +1.0)
  - Button pressed: fire button returns true while held
  - Axis calibration: deadzone auto-detected, center point adjusted
  - Vibration feedback: rumble triggered on weapon fire
- **Sad paths:**
  - Joystick disconnected: gracefully removed from available devices
  - Joystick with no axes: skipped, no crash
  - Invalid button index: ignored, no error thrown
  - Multiple joysticks connected: only first one used (or configurable)

**Completion Criteria:** All tests pass with ≥90% coverage. Joystick support works with axis calibration and button mapping.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (joystick detected, axes read correctly) and sad path (disconnection, invalid indices) tests must pass.**

## Sub-tasks

- [ ] Implement JoystickManager class: scan, connect, disconnect
- [ ] Detect connected joysticks via Web Gamepad API / HID
- [ ] Read joystick axes: throttle, pitch, yaw, roll
- [ ] Read joystick buttons: fire, full stop, warp, menu
- [ ] Implement axis calibration: auto-deadzone detection
- [ ] Implement button mapping: configurable assignments
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-1-5-implement-input-manager-keyboard-joystick-abstraction](phase-1-5-implement-input-manager-keyboard-joystick-abstraction.md)
- [depends-on phase-2-4-implement-input-axis-mapping-throttle-pitch-yaw-roll](phase-2-4-implement-input-axis-mapping-throttle-pitch-yaw-roll.md)

## History

- type: created
  date: 2026-11-24T04:11:43.861Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
