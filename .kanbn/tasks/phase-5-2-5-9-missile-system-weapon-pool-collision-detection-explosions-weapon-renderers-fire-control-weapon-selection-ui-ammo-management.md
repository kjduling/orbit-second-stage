---
tags:
  - 'type:feature'
  - 'phase:5'
  - 'area:combat'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T04:23:15.920Z
updated: 2026-11-24T04:58:39.617Z
---

# Phase 5.2-5.9: Missile system, weapon pool, collision detection, explosions, weapon renderers, fire control, weapon selection UI, ammo management

**Task ID:** 5.2–5.9 (grouped combat system tasks)
**Phase:** 5 — Combat & Weapons
**Dependencies:** Phase 5.1 (WeaponDef), Phase 2.3 (Ship state machine)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §3 (Weapons System — missiles, collision, explosions, renderers)
- Consult `design/implementation_plan.md` §9.1 tasks 5.2–5.9 — Combat system components
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — missile physics must be deterministic)

---

**Objective:**
Implement the complete combat system: missile spawning, pooling, collision detection with targets/planets, explosion effects, weapon renderers (cross, ray, tetrahedron, loop), fire control with cooldowns, weapon selection UI, and ammo management.

**Deliverables:**
- `src/weapons/Missile.ts` — missile entity class
- `src/weapons/WeaponPool.ts` — object pool for missiles
- `src/weapons/CollisionDetector.ts` — missile-target/planet collision
- `src/weapons/ExplosionRenderer.ts` — explosion visual effects
- `src/weapons/WeaponRendererFactory.ts` — factory for weapon renderers
- `src/weapons/FireControl.ts` — fire cooldown and targeting
- `src/weapons/WeaponSelectionUI.ts` — weapon selection display
- `tests/weapons/combat-system.test.ts` — all combat components tested

**Subtasks:**
- [ ] 5.2: Implement Missile class: pos, vel, type, lifetime, yield, active
- [ ] 5.3: Implement WeaponPool: object pool for missile reuse
- [ ] 5.4: Implement CollisionDetector: missile vs target/planet checks
- [ ] 5.5: Implement ExplosionRenderer: particle system for explosions
- [ ] 5.6: Implement WeaponRendererFactory: cross, ray, tetrahedron, loop renderers
- [ ] 5.7: Implement FireControl: cooldown management, targeting logic
- [ ] 5.8: Implement WeaponSelectionUI: show selected weapon index (0-3)
- [ ] 5.9: Implement Ammo management: unlimited for now, reserve for future
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Missile spawned: created from pool with correct type and velocity
  - Collision detected: missile hits target, yield applied as damage
  - Explosion rendered: particle effect plays for correct duration
  - Weapon renderer displayed: correct geometry for weapon type
  - Fire cooldown: weapon can't fire again until idleTime elapsed
  - Weapon selection: UI shows correct selected weapon index
- **Sad paths:**
  - Missile lifetime expired: removed from pool, no crash
  - Collision with planet: missile destroyed, explosion plays
  - No targets in range: fire command ignored, no error
  - Pool exhausted: creates new missile (graceful degradation)

**Completion Criteria:** All tests pass with ≥90% coverage. Combat system fully functional: missiles fire, collide, explode.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (combat works end-to-end) and sad path (pool exhausted, lifetime expired) tests must pass.**

## Sub-tasks

- [ ] 5.2: Implement Missile class: pos, vel, type, lifetime, yield, active
- [ ] 5.3: Implement WeaponPool: object pool for missile reuse
- [ ] 5.4: Implement CollisionDetector: missile vs target/planet checks
- [ ] 5.5: Implement ExplosionRenderer: particle system for explosions
- [ ] 5.6: Implement WeaponRendererFactory: cross, ray, tetrahedron, loop renderers
- [ ] 5.7: Implement FireControl: cooldown management, targeting logic
- [ ] 5.8: Implement WeaponSelectionUI: show selected weapon index (0-3)
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-5-1-define-weapondef-all-5-weapons-spare](phase-5-1-define-weapondef-all-5-weapons-spare.md)
- [depends-on phase-2-3-implement-ship-state-machine-with-shields-death-timer](phase-2-3-implement-ship-state-machine-with-shields-death-timer.md)

## History

- type: created
  date: 2026-11-24T04:23:15.920Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
