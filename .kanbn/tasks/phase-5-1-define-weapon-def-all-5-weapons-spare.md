---
tags:
  - 'type:feature'
  - 'phase:5'
  - 'area:combat'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:40:52.856Z
updated: 2026-11-24T04:58:39.560Z
---

# Phase 5.1: Define WeaponDef (all 5 weapons + spare)

**Task ID:** 5.1
**Phase:** 5 — Combat & Weapons
**Dependencies:** Phase 2 (Flight Physics), Phase 0.8 (RNG)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §3 (Weapons System — weapon definitions, mechanics, renderers)
- Consult `design/implementation_plan.md` §9 (Phase 5), §9.1 task 5.1, §9.2 (IWeaponDef interface)
- Consult `design/gap_analysis.md` §3.4 (Weapon renderer visuals — distinct per weapon type)

---

**Objective:**
Define all 5 weapons plus the spare/enemy weapon with their exact properties from the GDD: speed, yield (damage), idle time (cooldown), lifetime, renderer type, and color.

**Deliverables:**
- `src/weapons/WeaponDef.ts` — weapon definitions constants
- `tests/weapons/weapon-defs.test.ts` — all weapons have valid definitions tests

**Subtasks:**
- [ ] Define `IWeaponDef` interface: name, speed, yield, idleTime, lifetime, renderer, color
- [ ] Define `WeaponType` enum: Laser, PhotonRay, IonGun, Disruptor, Spare
- [ ] Implement Laser (index 0): speed=4000/KMU, yield=20, idle=0.2s, lifetime=1.0s, renderer=Cross, color=(1,1,0) yellow
- [ ] Implement PhotonRay (index 1): speed=5000/KMU, yield=30, idle=0.3s, lifetime=0.75s, renderer=Ray, color=(0,0,1) blue
- [ ] Implement IonGun (index 2): speed=4000/KMU, yield=30, idle=0.5s, lifetime=1.5s, renderer=Tetrahedron, color=(0,1,0) green
- [ ] Implement Disruptor (index 3): speed=3000/KMU, yield=60, idle=1.0s, lifetime=1.5s, renderer=Loop, color=(1,1,0) orange
- [ ] Implement Spare (enemy weapon): speed=6000/KMU, yield=15, idle=2.0s, lifetime=1.0s, renderer=None, color=(1,0,0) red
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - All 5 weapons defined with correct speed, yield, idleTime, lifetime
  - Renderer enum matches: Laser=Cross(1), PhotonRay=Ray(2), IonGun=Tetrahedron(3), Disruptor=Loop(4), Spare=None(0)
  - Colors match GDD: Laser=(1,1,0), PhotonRay=(0,0,1), IonGun=(0,1,0), Disruptor=(1,1,0), Spare=(1,0,0)
  - Range computed correctly: speed × lifetime for each weapon
- **Sad paths:**
  - Invalid weapon index (4+): throws error (only 0-3 valid for player)
  - Negative yield: clamped to zero or throws error
  - Zero idle time: weapon fires instantly (no cooldown)

**Completion Criteria:** All tests pass with ≥90% coverage. All weapons defined with correct properties from GDD.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all weapons valid) and sad path (invalid index, negative yield) tests must pass.**

## Sub-tasks

- [ ] Define IWeaponDef interface with name, speed, yield, idleTime, lifetime, renderer, color
- [ ] Define WeaponType enum: Laser, PhotonRay, IonGun, Disruptor, Spare
- [ ] Implement Laser weapon definition (yield=20, idle=0.2s)
- [ ] Implement PhotonRay weapon definition (yield=30, idle=0.3s)
- [ ] Implement IonGun weapon definition (yield=30, idle=0.5s)
- [ ] Implement Disruptor weapon definition (yield=60, idle=1.0s)
- [ ] Implement Spare/enemy weapon definition (yield=15, idle=2.0s)
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-2-3-implement-ship-state-machine-with-shields-death-timer](phase-2-3-implement-ship-state-machine-with-shields-death-timer.md)
- [depends-on phase-0-8-implement-rng-mulberry32-seeded-prng](phase-0-8-implement-rng-mulberry32-seeded-prng.md)

## History

- type: created
  date: 2026-11-24T03:40:52.856Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
