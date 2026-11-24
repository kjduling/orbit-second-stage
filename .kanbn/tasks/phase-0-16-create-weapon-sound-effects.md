---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:audio'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T04:06:19.215Z
updated: 2026-11-24T04:55:58.137Z
---

# Phase 0.16: Create weapon sound effects

**Task ID:** 0.16
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (Project scaffolding)
**Type:** Feature | **Priority:** Medium

---

**References:**
- Consult `design/Orbit.md` §3 (Weapons System — each weapon has distinct sound)
- Consult `design/implementation_plan.md` §4.2 task 0.16 — Create sound effects deliverable
- Consult `design/gap_analysis.md` §9.2 (Asset Pipeline — audio formats for BabylonJS)

---

**Objective:**
Create sound effects for all weapons and ship systems: laser fire, photon ray fire, ion gun fire, disruptor fire, missile launch, explosion, shield hit, death explosion, engine hum, warp sound.

**Deliverables:**
- `assets/audio/weapons/laser.wav` — laser weapon sound
- `assets/audio/weapons/photon_ray.wav` — photon ray weapon sound
- `assets/audio/weapons/ion_gun.wav` — ion gun weapon sound
- `assets/audio/weapons/disruptor.wav` — disruptor weapon sound
- `assets/audio/weapons/explosion.wav` — ship explosion sound
- `assets/audio/effects/shield_hit.wav` — shield damage sound
- `assets/audio/effects/death_explosion.wav` — death explosion sound
- `assets/audio/effects/engine_hum.wav` — engine background hum
- `assets/audio/effects/warp_sound.wav` — warp speed transition

**Subtasks:**
- [ ] Create laser fire sound: short, high-pitched "pew" (0.1s)
- [ ] Create photon ray fire sound: medium "crack" (0.2s)
- [ ] Create ion gun fire sound: deep "thump" (0.3s)
- [ ] Create disruptor fire sound: distorted "boom" (0.4s)
- [ ] Create missile launch sound: rocket "whoosh" (0.5s)
- [ ] Create explosion sound: low rumble (1.0s)
- [ ] Create shield hit sound: metallic ping (0.2s)
- [ ] Create death explosion sound: deep crash (2.0s)
- [ ] Create engine hum: continuous low drone
- [ ] Create warp sound: rising tone (1.0s)

**Unit Tests Required:** N/A (audio verified manually in Phase 9.8)

**Completion Criteria:** All sound effects created as WAV files, <2s each, mono or stereo.

---

⚠️ **This task is NOT complete without verification. All audio files must be valid WAV format and loadable by BabylonJS.**

## Sub-tasks

- [ ] Create laser, photon ray, ion gun, disruptor fire sounds
- [ ] Create missile launch sound
- [ ] Create explosion and shield hit sounds
- [ ] Create engine hum and warp sounds

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T04:06:19.215Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
