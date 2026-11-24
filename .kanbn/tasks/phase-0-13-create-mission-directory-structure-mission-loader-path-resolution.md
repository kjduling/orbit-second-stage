---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:parsing'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T03:22:40.446Z
updated: 2026-11-24T04:55:57.826Z
---

# Phase 0.13: Create mission directory structure + mission loader path resolution

**Task ID:** 0.13
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding), Phase 0.12 (GameMode enum)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §7 (Mission System) and Appendix A (Campaign Mission Chain Map)
- Consult `design/implementation_plan.md` §4.1 task 0.13, §1.5 (Dual Game Mode — mission file organization)
- Consult `design/gap_analysis.md` §2.2 (Mission File Access — Platform Differences: web vs native)

---

**Objective:**
Create the mission directory structure and implement path resolution that selects the correct mission set based on active game mode. Web platform needs URL fetch support; native needs filesystem access.

**Deliverables:**
- `data/missions/original/` — classic Orbit missions directory
- `data/missions/second-stage/` — expansion missions directory
- `src/core/mission-loader.ts` — path resolution per mode + platform
- `tests/core/mission-loader.test.ts` — path resolution per mode

**Subtasks:**
- [ ] Create `data/missions/original/` directory with placeholder files
- [ ] Create `data/missions/second-stage/` directory (empty, ready for future content)
- [ ] Implement `resolveMissionPath(missionName: string, mode: GameMode): string` — returns correct path prefix
- [ ] Implement platform detection: web (URL fetch) vs native (filesystem)
- [ ] Web: mission files served as static assets from `data/missions/`
- [ ] Native: direct filesystem access to `data/missions/`
- [ ] MissionParser: support both filesystem and URL-based include resolution
- [ ] Write unit tests for path resolution

**Unit Tests Required:**
- **Happy paths:**
  - ORIGINAL mode resolves to `data/missions/original/train01.msn`
  - SECOND_STAGE mode resolves to `data/missions/second-stage/ss01-intro.msn`
  - Web platform returns URL path: `/data/missions/original/train01.msn`
  - Native platform returns filesystem path: `data/missions/original/train01.msn`
- **Sad paths:**
  - Non-existent mission file: throws descriptive error with available missions listed
  - Mode not set before loading: throws error requesting mode selection first
  - Include file resolution fails gracefully (circular include detection)

**Completion Criteria:** All tests pass. Mission paths resolve correctly per game mode and platform.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct path resolution) and sad path (missing files, invalid mode) tests must pass.**

## Sub-tasks

- [ ] Create data/missions/original/ and data/missions/second-stage/ directories
- [ ] Implement resolveMissionPath() with mode-based path prefix selection
- [ ] Implement platform detection: web (URL fetch) vs native (filesystem)
- [ ] Support both filesystem and URL-based include resolution in MissionParser
- [ ] Write unit tests including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)
- [depends-on phase-0-12-define-gamemode-enum-mode-selector-infrastructure](phase-0-12-define-gamemode-enum-mode-selector-infrastructure.md)

## History

- type: created
  date: 2026-11-24T03:22:40.446Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
