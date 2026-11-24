---
tags:
  - 'type:feature'
  - 'phase:9'
  - 'area:polish'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T04:28:30.445Z
updated: 2026-11-24T04:59:34.436Z
---

# Phase 9.2-9.10: Server save, save/game separation, mission select UI, screen flash effects, explosion effects, sound effects, music system, settings persistence, difficulty selection

**Task ID:** 9.2–9.10 (grouped save system & polish tasks)
**Phase:** 9 — Save System & Polish
**Dependencies:** Phase 9.1 (LocalSaveManager), Phase 8 (Events & Game Flow)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §12 (Save System — save file format, game separation)
- Consult `design/implementation_plan.md` §13.1 tasks 9.2–9.10 — Save and polish components
- Consult `design/gap_analysis.md` §1.2 (Save System — strict separation of save data from game state)

---

**Objective:**
Implement remaining save system components and game polish: server-side save persistence, strict separation of save data from game state, mission select UI, screen flash effects, explosion visual effects, sound effects system, music system, settings persistence, and difficulty selection.

**Deliverables:**
- `src/save/ServerSaveManager.ts` — server-side save persistence
- `src/save/SaveData.ts` — save data structure (strict separation)
- `src/ui/MissionSelectUI.ts` — mission selection screen
- `src/effects/ScreenFlash.ts` — screen flash effects
- `src/effects/ExplosionEffects.ts` — explosion visual effects
- `src/audio/SoundManager.ts` — sound effects manager
- `src/audio/MusicManager.ts` — background music manager
- `src/settings/SettingsPersistence.ts` — settings save/load
- `src/game/DifficultySelector.ts` — difficulty selection UI
- `tests/save/polish.test.ts` — all polish components tested

**Subtasks:**
- [ ] 9.2: Implement ServerSaveManager: server-side save persistence (multiplayer)
- [ ] 9.3: Implement SaveData: strict separation of save data from game state
- [ ] 9.4: Implement MissionSelectUI: numbered list of missions with briefings
- [ ] 9.5: Implement ScreenFlash: flash screen on damage/death
- [ ] 9.6: Implement ExplosionEffects: particle effects for explosions
- [ ] 9.7: Implement SoundManager: play weapon sounds, shield hits, explosions
- [ ] 9.8: Implement MusicManager: play background music, crossfade tracks
- [ ] 9.9: Implement SettingsPersistence: save/load player preferences
- [ ] 9.10: Implement DifficultySelector: easy/medium/hard difficulty options
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Server save: multiplayer save persisted to server database
  - Save data separation: game state changes don't affect save file until saved
  - Mission select UI: lists all available missions with briefings
  - Screen flash: flashes red on damage, white on death
  - Explosion effects: particle effect plays for correct duration
  - Sound manager: weapon sounds play correctly
  - Music manager: background music loops seamlessly
  - Settings persistence: settings saved and restored on reload
  - Difficulty selector: affects game difficulty (enemy strength, spawn rate)
- **Sad paths:**
  - Server save unavailable: falls back to local save, logs warning
  - Corrupt save file: loads default state, logs error
  - Sound file missing: ignored, no crash
  - Settings file corrupt: loads defaults, logs warning

**Completion Criteria:** All tests pass with ≥90% coverage. Save system and polish fully functional.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (save/polish works) and sad path (corrupt files, missing assets) tests must pass.**

## Sub-tasks

- [ ] 9.2: Implement ServerSaveManager: server-side save persistence
- [ ] 9.3: Implement SaveData: strict separation of save data from game state
- [ ] 9.4: Implement MissionSelectUI: numbered list of missions with briefings
- [ ] 9.5: Implement ScreenFlash: flash screen on damage/death
- [ ] 9.6: Implement ExplosionEffects: particle effects for explosions
- [ ] 9.7: Implement SoundManager: play weapon sounds, shield hits, explosions
- [ ] 9.8: Implement MusicManager: play background music, crossfade tracks
- [ ] 9.9: Implement SettingsPersistence: save/load player preferences
- [ ] 9.10: Implement DifficultySelector: easy/medium/hard difficulty options
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-9-1-implement-local-save-manager-sp-save-load](phase-9-1-implement-local-save-manager-sp-save-load.md)
- [depends-on phase-8-2-8-7-action-processor-game-state-machine-death-respawn-sequence-mission-progression-victory-condition-game-over-screen](phase-8-2-8-7-action-processor-game-state-machine-death-respawn-sequence-mission-progression-victory-condition-game-over-screen.md)

## History

- type: created
  date: 2026-11-24T04:28:30.445Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
