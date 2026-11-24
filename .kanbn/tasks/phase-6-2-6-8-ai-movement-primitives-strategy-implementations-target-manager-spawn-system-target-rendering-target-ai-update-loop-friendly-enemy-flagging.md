---
tags:
  - 'type:feature'
  - 'phase:6'
  - 'area:ai'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:24:25.585Z
updated: 2026-11-24T04:58:53.453Z
---

# Phase 6.2-6.8: AI movement primitives, strategy implementations, target manager, spawn system, target rendering, target AI update loop, friendly/enemy flagging

**Task ID:** 6.2–6.8 (grouped AI target system tasks)
**Phase:** 6 — AI Targets
**Dependencies:** Phase 6.1 (Target entity), Phase 5.1 (WeaponDef)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §5 (AI & Target Strategies — 9 strategies, movement primitives, target management)
- Consult `design/implementation_plan.md` §10.1 tasks 6.2–6.8 — AI system components
- Consult `design/gap_analysis.md` §3.4 (Server authority — AI runs server-side in multiplayer)

---

**Objective:**
Implement the complete AI target system: movement primitives (sit, hunt, flee), strategy implementations for all 9 strategies, target manager for spawning/destroying targets, target rendering, AI update loop, and friendly/enemy flagging.

**Deliverables:**
- `src/targets/AIMovement.ts` — AI movement primitives
- `src/targets/StrategyImplementations.ts` — all 9 strategy implementations
- `src/targets/TargetManager.ts` — spawn, destroy, manage targets
- `src/targets/TargetRenderer.ts` — render AI targets
- `tests/targets/ai-system.test.ts` — all AI components tested

**Subtasks:**
- [ ] 6.2: Implement AIMovement: sit (stay still), hunt (chase player), flee (run from player)
- [ ] 6.3: Implement DoNothing strategy: target does nothing
- [ ] 6.4: Implement Sit1-4 strategies: different behaviors based on distance
- [ ] 6.5: Implement Hunt1-4 strategies: chase player with varying aggression
- [ ] 6.6: Implement TargetManager: spawn/destroy targets, maintain target list
- [ ] 6.7: Implement TargetRenderer: render AI targets with faction colors
- [ ] 6.8: Implement friendly/enemy flagging: green=friendly, red=enemy
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Sit strategy: target stays at fixed position
  - Hunt strategy: target moves toward player at max speed
  - Flee strategy: target moves away from player at max speed
  - Target spawned: added to manager's list, rendered correctly
  - Target destroyed: removed from list, score awarded to player
  - Friendly target: colored green on radar and in 3D
  - Enemy target: colored red on radar and in 3D
- **Sad paths:**
  - Destroy non-existent target: ignored, logs warning
  - Spawn target with invalid strategy: throws error
  - All targets destroyed: manager returns empty list, no crash
  - Target out of render distance: not rendered, no crash

**Completion Criteria:** All tests pass with ≥90% coverage. AI target system fully functional: strategies work, targets spawn/destroy correctly.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (AI strategies work) and sad path (invalid strategies, non-existent targets) tests must pass.**

## Sub-tasks

- [ ] 6.2: Implement AIMovement: sit, hunt, flee primitives
- [ ] 6.3: Implement DoNothing strategy
- [ ] 6.4: Implement Sit1-4 strategies
- [ ] 6.5: Implement Hunt1-4 strategies
- [ ] 6.6: Implement TargetManager: spawn/destroy targets
- [ ] 6.7: Implement TargetRenderer: render AI targets with faction colors
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-6-1-implement-target-entity-with-ai-strategy-system](phase-6-1-implement-target-entity-with-ai-strategy-system.md)
- [depends-on phase-5-1-define-weapondef-all-5-weapons-spare](phase-5-1-define-weapondef-all-5-weapons-spare.md)

## History

- type: created
  date: 2026-11-24T04:24:25.585Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
