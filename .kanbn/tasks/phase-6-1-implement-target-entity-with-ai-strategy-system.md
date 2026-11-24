---
tags:
  - 'type:feature'
  - 'phase:6'
  - 'area:ai'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:40:52.888Z
updated: 2026-11-24T04:58:53.398Z
---

# Phase 6.1: Implement Target entity with AI strategy system

**Task ID:** 6.1
**Phase:** 6 — AI Targets
**Dependencies:** Phase 2 (Flight Physics), Phase 5.1 (WeaponDef)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §5 (AI & Target Strategies — 9 strategy types, distance thresholds)
- Consult `design/implementation_plan.md` §10 (Phase 6), §10.1 task 6.1, §10.2 (ITarget interface, Strategy enum)
- Consult `design/gap_analysis.md` §3.4 (Server authority — AI runs server-side in multiplayer)

---

**Objective:**
Implement the Target entity class with position, velocity, strategy, weapon, shields, and all properties from the GDD. Targets are AI-controlled ships that can be friendly or enemy.

**Deliverables:**
- `src/targets/Target.ts` — Target entity class
- `tests/targets/target.test.ts` — update, state management tests

**Subtasks:**
- [ ] Define `Strategy` enum: DoNothing, Sit1-4, Hunt1-4 (9 strategies total)
- [ ] Define `ITarget` interface: name, pos, vel, viewDir, upDir, strategy, friendly, shields, maxShields, weapon, score, hidden, invisible, model, turnRate, speed, age, lastFireTime
- [ ] Implement Target class with all properties and update method
- [ ] Implement shield system (same as Ship: regen, cap at max)
- [ ] Implement `update(deltaT, input)`: apply AI strategy movement
- [ ] Implement `takeDamage(amount)`: reduce shields, return true if destroyed
- [ ] Implement `canFire()`: check weapon cooldown (lastFireTime + idleTime < absT)
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Target created with correct name, position, strategy, shields
  - Shield regen: target regenerates shields at SHIELD_REGEN/sec
  - Take damage: shields reduced correctly, returns false if still alive
  - Can fire: weapon cooldown expired → canFire() = true
  - Destroyed: shields < 0 → destroyed flag set, score awarded to player
- **Sad paths:**
  - Target with no strategy: throws error (must have valid strategy)
  - Negative maxShields: clamped to positive value
  - Fire during cooldown: canFire() = false, returns null missile

**Completion Criteria:** All tests pass with ≥90% coverage. Target entity manages its own state, shields, and fire cooldown.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (target state correct) and sad path (invalid strategy, negative values) tests must pass.**

## Sub-tasks

- [ ] Define Strategy enum: DoNothing, Sit1-4, Hunt1-4 (9 strategies)
- [ ] Define ITarget interface with all required properties
- [ ] Implement Target class with position, velocity, strategy, shields
- [ ] Implement shield system: regen, cap at maxShields
- [ ] Implement update() with AI strategy movement delegation
- [ ] Implement takeDamage() and canFire() methods
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-2-3-implement-ship-state-machine-with-shields-death-timer](phase-2-3-implement-ship-state-machine-with-shields-death-timer.md)
- [depends-on phase-5-1-define-weapondef-all-5-weapons-spare](phase-5-1-define-weapondef-all-5-weapons-spare.md)

## History

- type: created
  date: 2026-11-24T03:40:52.888Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
