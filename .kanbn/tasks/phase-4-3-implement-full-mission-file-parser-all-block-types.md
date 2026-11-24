---
tags:
  - 'type:feature'
  - 'phase:4'
  - 'area:parsing'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T04:20:51.132Z
updated: 2026-11-24T04:58:25.436Z
---

# Phase 4.3: Implement full mission file parser (all block types)

**Task ID:** 4.3
**Phase:** 4 — Mission Parser
**Dependencies:** Phase 4.2 (Block parser)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §7 (Mission System — all mission file block types: Mission, Cursor, Planet, Target, Event)
- Consult `design/implementation_plan.md` §8.1 task 4.3 — Full mission parser
- Consult `design/gap_analysis.md` §2.2 (Mission File Access — all block types supported)

---

**Objective:**
Implement the full mission file parser that processes all block types defined in mission files: Mission, Cursor, Planet, Target, Event, Include, etc. Returns a structured Mission object with all game state.

**Deliverables:**
- `src/missions/MissionParser.ts` — full mission file parser
- `tests/missions/mission-parser.test.ts` — all block types, error handling tests

**Subtasks:**
- [ ] Define `IMission` interface: name, cursor, planets, targets, events, briefing
- [ ] Implement `MissionParser` class consuming BlockParser output
- [ ] Parse Mission block: mission name, description, difficulty
- [ ] Parse Cursor block: starting planet, initial theta
- [ ] Parse Planet blocks: all planet properties from GDD
- [ ] Parse Target blocks: AI targets with strategies
- [ ] Parse Event blocks: trigger conditions and actions
- [ ] Parse Include directives: resolve include file paths
- [ ] Parse Briefing text: multi-line quoted string
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Full mission file parsed: all blocks converted to Mission object
  - Cursor block: starting planet and theta set correctly
  - Planet blocks: all properties (name, radius, mass, texture) set
  - Target blocks: AI strategy, weapon, shields set
  - Event blocks: trigger type, conditions, actions set
  - Include files resolved: included content merged into mission
  - Briefing text: multi-line string preserved correctly
- **Sad paths:**
  - Missing required block (Mission): throws error
  - Invalid property value (negative radius): clamped to positive, logs warning
  - Duplicate planet name in same mission: second one renamed with suffix
  - Unknown block type: skipped, logs warning

**Completion Criteria:** All tests pass with ≥90% coverage. Full mission file parsed into structured Mission object.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all blocks parsed) and sad path (missing required blocks, invalid values) tests must pass.**

## Sub-tasks

- [ ] Define IMission interface: name, cursor, planets, targets, events, briefing
- [ ] Implement MissionParser class consuming BlockParser output
- [ ] Parse Mission block: name, description, difficulty
- [ ] Parse Cursor block: starting planet, initial theta
- [ ] Parse Planet blocks: all properties from GDD
- [ ] Parse Target blocks: AI strategy, weapon, shields
- [ ] Parse Event blocks: trigger type, conditions, actions
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-4-2-implement-brace-delimited-block-parser-cursor-planet-etc](phase-4-2-implement-brace-delimited-block-parser-cursor-planet-etc.md)

## History

- type: created
  date: 2026-11-24T04:20:51.132Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
