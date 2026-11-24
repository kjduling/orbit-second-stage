---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:core'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:22:40.357Z
updated: 2026-11-24T04:55:57.740Z
---

# Phase 0.12: Define GameMode enum + mode selector infrastructure

**Task ID:** 0.12
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §16.5 (Dual Game Mode Architecture) for Original vs Second Stage design
- Consult `design/implementation_plan.md` §4.1 task 0.12, §1.5 (Dual Game Mode Architecture)
- Consult `design/gap_analysis.md` §1.4 (Dual Game Mode decision — Original + Second Stage)

---

**Objective:**
Implement the GameMode enum and mode selector infrastructure. This determines which mission sets, ships, weapons, and HUD elements are available. Both modes share the same core engine.

**Deliverables:**
- `src/core/game-mode.ts` — GameMode enum + mode selector
- `tests/core/game-mode.test.ts` — mode selection and save tracking tests

**Subtasks:**
- [ ] Define `GameMode` enum: `ORIGINAL = 'original'`, `SECOND_STAGE = 'second-stage'`
- [ ] Implement `ActiveGameMode` class with singleton pattern
- [ ] Implement `getMode(): GameMode` — returns current active mode
- [ ] Implement `setMode(mode: GameMode): void` — switches active mode
- [ ] Implement `isUnlocked(mode: GameMode): boolean` — Second Stage unlocked after Original completion
- [ ] Implement save tracking: track which missions completed per mode
- [ ] Implement default mode selection (Original if no save data)
- [ ] Write unit tests

**Unit Tests Required:**
- **Happy paths:**
  - Default mode is ORIGINAL
  - Switching to SECOND_STAGE changes active mode
  - Mode persistence across getMode/setMode calls
  - Second Stage unlocks after completing all Original missions
- **Sad paths:**
  - Invalid mode string: throws error
  - Setting mode before initialization: throws descriptive error
  - Toggle mode while mission loading: prevents switch, logs warning

**Completion Criteria:** All tests pass. GameMode infrastructure correctly manages mode selection and unlock tracking.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (mode switching works) and sad path (invalid input handled) tests must pass.**

## Sub-tasks

- [ ] Define GameMode enum: ORIGINAL and SECOND_STAGE
- [ ] Implement ActiveGameMode singleton with get/set mode
- [ ] Implement isUnlocked() — Second Stage unlocks after Original completion
- [ ] Implement save tracking per mode
- [ ] Write unit tests including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:22:40.357Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
