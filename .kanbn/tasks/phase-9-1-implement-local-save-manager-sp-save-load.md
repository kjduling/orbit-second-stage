---
tags:
  - 'type:feature'
  - 'phase:9'
  - 'area:save'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:45:17.250Z
updated: 2026-11-24T04:59:34.373Z
---

# Phase 9.1: Implement LocalSaveManager (SP save/load)

**Task ID:** 9.1
**Phase:** 9 — Save System & Polish
**Dependencies:** Phase 8 (Events & Game Flow), Phase 0.12 (GameMode)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §12 (Save System — plain text save file format)
- Consult `design/implementation_plan.md` §13 (Phase 9), §13.1 task 9.1, §13.2 (Save System Architecture — strict separation)
- Consult `design/gap_analysis.md` §1.2 (Save System — local only for SP, server-only for multiplayer)

---

**Objective:**
Implement the LocalSaveManager for single-player saves. Native builds use file I/O; web builds use IndexedDB. Save format: one entry per line `mission_filename timestamp`, most recent at top, max entries configurable (NSAVES).

**Deliverables:**
- `src/save/LocalSaveManager.ts` — SP save/load with platform-specific storage
- `tests/save/local-save-manager.test.ts` — write, read, max entries tests

**Subtasks:**
- [ ] Define `ISaveManager` interface: save, load, list, delete
- [ ] Implement platform detection: native (file I/O) vs web (IndexedDB)
- [ ] Native: save to `~/.orbit-second-stage/saves/` directory
- [ ] Web: save to IndexedDB database `orbit-saves`
- [ ] Save format: `mission_filename unix_timestamp` per line
- [ ] Implement max entries limit (NSAVES, configurable)
- [ ] Most recent save at top of file
- [ ] Implement save menu UI data: numbered list with mission name + timestamp
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Save: writes `train01.msn 1699900800` to save file/IndexedDB
  - Load: reads most recent save and returns mission name + timestamp
  - List: returns all saves in order (most recent first)
  - Max entries: when limit reached, oldest entry removed automatically
  - Delete: removes specific save by index
- **Sad paths:**
  - Save file directory doesn't exist: creates it automatically
  - Corrupt save file (malformed line): skips bad line, logs warning
  - IndexedDB full: throws error with message about clearing saves
  - Save while mission not loaded: throws error requesting active mission first

**Completion Criteria:** All tests pass with ≥90% coverage. LocalSaveManager correctly saves and loads missions on both platforms.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (save/load roundtrip works) and sad path (corrupt data, full storage) tests must pass.**

## Sub-tasks

- [ ] Define ISaveManager interface with save, load, list, delete
- [ ] Implement platform detection: native (file I/O) vs web (IndexedDB)
- [ ] Native: save to ~/.orbit-second-stage/saves/ directory
- [ ] Web: save to IndexedDB database orbit-saves
- [ ] Implement save format and max entries limit
- [ ] Implement save menu data (numbered list with mission + timestamp)
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-8-1-implement-event-evaluator-all-trigger-types](phase-8-1-implement-event-evaluator-all-trigger-types.md)
- [depends-on phase-0-12-define-game-mode-enum-mode-selector-infrastructure](phase-0-12-define-game-mode-enum-mode-selector-infrastructure.md)

## History

- type: created
  date: 2026-11-24T03:45:17.250Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
