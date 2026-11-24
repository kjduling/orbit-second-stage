---
tags:
  - 'type:feature'
  - 'phase:4'
  - 'area:parsing'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:22:00.541Z
updated: 2026-11-24T04:58:25.472Z
---

# Phase 4.4-4.10: Mission loader, include resolver, briefing UI, event parser, action processor, mission packager, save format

**Task ID:** 4.4–4.10 (grouped mission system tasks)
**Phase:** 4 — Mission Parser
**Dependencies:** Phase 4.3 (Mission parser), Phase 9.1 (Save System)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §7 (Mission System — mission loading, include files, briefing display)
- Consult `design/implementation_plan.md` §8.1 tasks 4.4–4.10 — Mission system components
- Consult `design/gap_analysis.md` §2.2 (Mission File Access — platform differences in file paths)

---

**Objective:**
Implement remaining mission system components: mission file loading from disk, include file resolution, briefing text display, event parsing from missions, action processing, mission packager for distribution, and save format integration.

**Deliverables:**
- `src/missions/MissionLoader.ts` — load mission files from disk
- `src/missions/IncludeResolver.ts` — resolve include file paths
- `src/missions/BriefingUI.ts` — display briefing text to player
- `src/missions/EventParser.ts` — parse events from mission blocks
- `src/missions/ActionProcessor.ts` — process event actions
- `src/missions/MissionPackager.ts` — package missions for distribution
- `tests/missions/mission-system.test.ts` — all components tested

**Subtasks:**
- [ ] 4.4: Implement MissionLoader: read .msn/.inc files from disk
- [ ] 4.5: Implement IncludeResolver: resolve include paths relative to mission dir
- [ ] 4.6: Implement BriefingUI: display briefing text in HTML overlay
- [ ] 4.7: Implement EventParser: parse event triggers from mission blocks
- [ ] 4.8: Implement ActionProcessor: execute event actions (spawn target, change music)
- [ ] 4.9: Implement MissionPackager: zip mission + assets for distribution
- [ ] 4.10: Implement save format: serialize Mission to save file
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - MissionLoader reads .msn file and returns parsed Mission object
  - IncludeResolver resolves include path relative to mission directory
  - BriefingUI displays text correctly in HTML overlay
  - EventParser parses event triggers with correct conditions
  - ActionProcessor executes actions (spawn target, change music)
  - MissionPackager creates valid zip file
- **Sad paths:**
  - Missing mission file: throws descriptive error
  - Include file not found: logs warning, continues without include
  - Invalid briefing text: displays as-is, no crash
  - Action for unknown event type: ignored, logs warning

**Completion Criteria:** All tests pass with ≥90% coverage. Mission system fully functional: load, parse, display, execute.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (mission loading works) and sad path (missing files, invalid data) tests must pass.**

## Sub-tasks

- [ ] 4.4: Implement MissionLoader: read .msn/.inc files from disk
- [ ] 4.5: Implement IncludeResolver: resolve include paths relative to mission dir
- [ ] 4.6: Implement BriefingUI: display briefing text in HTML overlay
- [ ] 4.7: Implement EventParser: parse event triggers from mission blocks
- [ ] 4.8: Implement ActionProcessor: execute event actions
- [ ] 4.9: Implement MissionPackager: zip mission + assets for distribution
- [ ] 4.10: Implement save format: serialize Mission to save file
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-4-3-implement-full-mission-file-parser-all-block-types](phase-4-3-implement-full-mission-file-parser-all-block-types.md)
- [depends-on phase-9-1-implement-local-save-manager-sp-save-load](phase-9-1-implement-local-save-manager-sp-save-load.md)

## History

- type: created
  date: 2026-11-24T04:22:00.541Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
