---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:ship'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:36:09.472Z
updated: 2026-11-24T04:57:25.084Z
---

# Phase 2.3: Implement Ship state machine with shields, death timer

**Task ID:** 2.3
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 2.1-2.2 (FlightModels), Phase 1.5 (InputManager)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §2.1 (Player Ship — attributes, state machine: NORMAL, DEAD1, DEAD2, INIT, LOADGAME)
- Consult `design/implementation_plan.md` §6.1 task 2.3, §6.2 (IShip interface)
- Consult `design/gap_analysis.md` §1.2 (Save System — local saves only for single-player)

---

**Objective:**
Implement the Ship class with full state machine, shield system, weapon selection, and death/respawn sequence. The ship is the player's avatar — its state drives the entire game loop.

**Deliverables:**
- `src/player/Ship.ts` — Ship class with state machine
- `tests/player/ship.test.ts` — state transitions, shield regen, death timer tests

**Subtasks:**
- [ ] Implement `IShip` interface: pos, vel, viewDir, upDir, rightDir, shields, maxShields, score, weapon, state, flightModel
- [ ] Implement ShipState enum: INIT, NORMAL, DEAD1, DEAD2, LOADGAME
- [ ] Implement shield system: regen at SHIELD_REGEN/sec, auto-capped at maxShields (100)
- [ ] Implement death sequence: shields < 0 → STATE_DEAD1 (5s countdown) → STATE_DEAD2 (press fire to respawn)
- [ ] Implement weapon selection (0-3): currently selected weapon index
- [ ] Implement `update(deltaT, input)`: delegate to flightModel, handle state transitions
- [ ] Implement `takeDamage(amount)`: reduce shields, return true if died
- [ ] Implement `regenerateShields(deltaT)`: add SHIELD_REGEN × deltaT, cap at max
- [ ] Implement `fireMissile(weaponIndex)`: delegate to weapon system
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Shield regen: 100 shields, no damage for 5s → stays at 100 (capped)
  - Take damage: 80 shields, take 30 damage → 50 shields
  - Death: 10 shields, take 20 damage → shields = -10, state = DEAD1
  - Dead1 countdown: 5 seconds elapses → state = DEAD2
  - Respawn from DEAD2: press fire → state = NORMAL, shields reset to max
- **Sad paths:**
  - Take damage when already DEAD2: no additional death processing
  - Shield regen when shields at max: no overflow (stays at 100)
  - Fire missile while DEAD1/DEAD2: no missiles spawned
  - State transitions only follow valid paths (no INIT→DEAD2 skip)

**Completion Criteria:** All tests pass with ≥90% coverage. Ship state machine, shields, and death sequence work correctly.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (state transitions correct) and sad path (invalid states prevented) tests must pass.**

## Sub-tasks

- [ ] Define IShip interface with all required properties and methods
- [ ] Implement ShipState enum: INIT, NORMAL, DEAD1, DEAD2, LOADGAME
- [ ] Implement shield system: regen, cap at maxShields (100)
- [ ] Implement death sequence: DEAD1 (5s countdown) → DEAD2 → respawn
- [ ] Implement weapon selection and fireMissile delegation
- [ ] Implement update() with flightModel delegation and state transitions
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-2-1-implement-flight-model-base-interface-newtonian-impl](phase-2-1-implement-flight-model-base-interface-newtonian-impl.md)
- [depends-on phase-2-2-implement-arcade-flight-model](phase-2-2-implement-arcade-flight-model.md)
- [depends-on phase-1-5-implement-input-manager-keyboard-joystick-abstraction](phase-1-5-implement-input-manager-keyboard-joystick-abstraction.md)

## History

- type: created
  date: 2026-11-24T03:36:09.472Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
