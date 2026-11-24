---
tags:
  - 'type:feature'
  - 'phase:8'
  - 'area:events'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T04:27:02.015Z
updated: 2026-11-24T04:59:20.613Z
---

# Phase 8.2-8.7: Action processor, game state machine, death/respawn sequence, mission progression, victory condition, game over screen

**Task ID:** 8.2–8.7 (grouped events & game flow tasks)
**Phase:** 8 — Events & Game Flow
**Dependencies:** Phase 8.1 (EventEvaluator), Phase 5 (Combat), Phase 6 (AI Targets), Phase 7 (HUD)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §7 (Mission System — event actions, game flow, victory/defeat conditions)
- Consult `design/implementation_plan.md` §12.1 tasks 8.2–8.7 — Game flow components
- Consult `design/gap_analysis.md` §3.4 (Server authority — game state synchronized server-side in multiplayer)

---

**Objective:**
Implement the complete game flow: action processor (execute event actions), game state machine (INIT, PLAYING, PAUSED, DEAD, VICTORY, GAMEOVER), death/respawn sequence, mission progression (complete objectives → next mission), victory condition (all targets destroyed), and game over screen.

**Deliverables:**
- `src/game/ActionProcessor.ts` — execute event actions
- `src/game/GameStateMachine.ts` — game state transitions
- `src/game/DeathRespawn.ts` — death/respawn sequence
- `src/game/MissionProgression.ts` — mission objectives tracking
- `src/game/VictoryCondition.ts` — victory detection
- `src/game/GameOverScreen.ts` — game over / victory screen
- `tests/game/game-flow.test.ts` — all game flow components tested

**Subtasks:**
- [ ] 8.2: Implement ActionProcessor: execute actions (spawn target, play sound, change music)
- [ ] 8.3: Implement GameStateMachine: INIT → PLAYING → PAUSED → DEAD → VICTORY/GAMEOVER
- [ ] 8.4: Implement DeathRespawn: death sequence, respawn with full shields
- [ ] 8.5: Implement MissionProgression: track objectives, advance to next mission
- [ ] 8.6: Implement VictoryCondition: all targets destroyed → victory screen
- [ ] 8.7: Implement GameOverScreen: game over / victory screen with score
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Action executed: spawn target, play sound, or change music triggered
  - State transition: INIT → PLAYING when mission starts
  - Death sequence: shields < 0 → DEAD1 (5s countdown) → DEAD2 → respawn
  - Mission completed: all objectives met → next mission loaded
  - Victory achieved: all targets destroyed → victory screen shown
  - Game over: too many deaths → game over screen shown
- **Sad paths:**
  - Action for unknown event type: ignored, logs warning
  - Respawn during DEAD1: countdown continues, no reset
  - Mission with no objectives: completes immediately
  - Victory screen shown twice: no crash, idempotent

**Completion Criteria:** All tests pass with ≥90% coverage. Game flow fully functional: states transition correctly, victory/defeat detected.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (game flow works end-to-end) and sad path (unknown actions, invalid states) tests must pass.**

## Sub-tasks

- [ ] 8.2: Implement ActionProcessor: execute actions (spawn target, play sound)
- [ ] 8.3: Implement GameStateMachine: INIT → PLAYING → PAUSED → DEAD → VICTORY/GAMEOVER
- [ ] 8.4: Implement DeathRespawn: death sequence, respawn with full shields
- [ ] 8.5: Implement MissionProgression: track objectives, advance to next mission
- [ ] 8.6: Implement VictoryCondition: all targets destroyed → victory screen
- [ ] 8.7: Implement GameOverScreen: game over / victory screen with score
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-8-1-implement-event-evaluator-all-trigger-types](phase-8-1-implement-event-evaluator-all-trigger-types.md)
- [depends-on phase-5-2-5-9-missile-system-weapon-pool-collision-detection-explosions-weapon-renderers-fire-control-weapon-selection-ui-ammo-management](phase-5-2-5-9-missile-system-weapon-pool-collision-detection-explosions-weapon-renderers-fire-control-weapon-selection-ui-ammo-management.md)
- [depends-on phase-6-2-6-8-ai-movement-primitives-strategy-implementations-target-manager-spawn-system-target-rendering-target-ai-update-loop-friendly-enemy-flagging](phase-6-2-6-8-ai-movement-primitives-strategy-implementations-target-manager-spawn-system-target-rendering-target-ai-update-loop-friendly-enemy-flagging.md)
- [depends-on phase-7-2-7-10-shield-bars-crosshair-status-messages-velocity-display-throttle-bar-speed-readout-score-display-mission-briefing-overlay-settings-panel](phase-7-2-7-10-shield-bars-crosshair-status-messages-velocity-display-throttle-bar-speed-readout-score-display-mission-briefing-overlay-settings-panel.md)

## History

- type: created
  date: 2026-11-24T04:27:02.015Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
