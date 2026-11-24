---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:assets'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T03:26:33.581Z
updated: 2026-11-24T04:55:57.856Z
---

# Phase 0.14-0.17: Source/create all original game assets (models, textures, sounds, music)

**Task ID:** 0.14–0.17 (grouped asset creation tasks)
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §9 (Models & Visuals), §10 (Audio System), §6 (Planets & Solar System) for asset specifications
- Consult `design/implementation_plan.md` §4.1 tasks 0.14–0.17, §5.3 (Original Asset Pipeline)
- Consult `design/gap_analysis.md` §1.3 (Asset Creation — all original, no licensing concerns)

---

**Objective:**
Source or create all original game assets from scratch. No legacy Orbit game assets, no licensing concerns. All assets in BabylonJS-native formats.

**Deliverables:**
- `data/models/` — GLTF/GLB ship/station/starfield models
- `data/textures/` — PNG/WebP planet/space/background textures
- `data/sounds/` — WAV (native) / Ogg (web) SFX: explosions, engines, weapons
- `data/music/` — Ogg/MP3 music tracks

**Subtasks:**
- [ ] **0.14 Models:** Create GLTF/GLB models for all ships (light1, light2, medium1, heavy1, fight1), freighters, stations (station1-4), bases, platforms, solar arrays, comm arrays, artifacts, portals, ejection modules, destroyed ships
- [ ] **0.15 Textures:** Create PNG/WebP textures for all planets (Mercury through Neptune), moons (Phobos, Deimos, Io, Europa, Titan, Triton, etc.), space backgrounds, HUD elements
- [ ] **0.16 Sounds:** Source/create WAV/Ogg sound effects: phaser/weapon fire, explosion, communications
- [ ] **0.17 Music:** Source/create Ogg/MP3 music tracks for menu, gameplay, victory, defeat themes

**Asset Sources (all CC0 or original):**
- Models: Blender (free), Kenney asset packs (CC0), OpenGameArt
- Textures: Krita/Photoshop (original), Kenney (CC0), procedural generation
- SFX: Freesound.org (CC0), BFXR, original compositions
- Music: Incompetia (free), Kenney (CC0), original compositions

**Unit Tests Required:** N/A for asset creation (assets verified by manual testing in Phase 1)

**Completion Criteria:** All asset directories populated with correct formats. Assets load correctly in BabylonJS scene.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all assets load and render) and sad path (missing/corrupt asset handling) tests must pass.**

## Sub-tasks

- [ ] 0.14: Create GLTF/GLB models for all ships, stations, artifacts
- [ ] 0.15: Create PNG/WebP textures for all planets, moons, backgrounds
- [ ] 0.16: Source/create WAV/Ogg sound effects (phaser, explosion, comm)
- [ ] 0.17: Source/create Ogg/MP3 music tracks (menu, gameplay, victory, defeat)

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:26:33.581Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
