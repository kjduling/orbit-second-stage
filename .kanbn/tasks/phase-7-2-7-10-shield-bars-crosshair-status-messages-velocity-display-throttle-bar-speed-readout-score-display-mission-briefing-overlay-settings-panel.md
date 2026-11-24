---
tags:
  - 'type:feature'
  - 'phase:7'
  - 'area:ui'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:25:45.105Z
updated: 2026-11-24T04:59:07.256Z
---

# Phase 7.2-7.10: Shield bars, crosshair, status messages, velocity display, throttle bar, speed readout, score display, mission briefing overlay, settings panel

**Task ID:** 7.2–7.10 (grouped HUD/UI tasks)
**Phase:** 7 — HUD & UI
**Dependencies:** Phase 7.1 (Radar), Phase 2.3 (Ship state machine), Phase 8 (Events & Game Flow)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §8 (HUD — shield bars, crosshair, status messages, velocity, throttle, speed, score)
- Consult `design/implementation_plan.md` §11.1 tasks 7.2–7.10 — HUD components
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — HUD updates must be deterministic)

---

**Objective:**
Implement the complete HUD system: shield bars (left/right), crosshair (center), status messages (text overlay), velocity display, throttle bar, speed readout, score display, mission briefing overlay, and settings panel.

**Deliverables:**
- `src/hud/ShieldBar.ts` — shield level indicator
- `src/hud/Crosshair.ts` — center crosshair
- `src/hud/StatusMessage.ts` — text message overlay
- `src/hud/VelocityDisplay.ts` — velocity readout
- `src/hud/ThrottleBar.ts` — throttle level indicator
- `src/hud/SpeedReadout.ts` — speed display
- `src/hud/ScoreDisplay.ts` — score counter
- `src/hud/BriefingOverlay.ts` — mission briefing display
- `src/hud/SettingsPanel.ts` — settings UI
- `tests/hud/hud-system.test.ts` — all HUD components tested

**Subtasks:**
- [ ] 7.2: Implement ShieldBar: left/right shield indicators (green bars)
- [ ] 7.3: Implement Crosshair: center screen, changes with target lock
- [ ] 7.4: Implement StatusMessage: scrollable text messages (top-left)
- [ ] 7.5: Implement VelocityDisplay: velocity magnitude readout
- [ ] 7.6: Implement ThrottleBar: throttle level indicator (bottom-center)
- [ ] 7.7: Implement SpeedReadout: current speed display
- [ ] 7.8: Implement ScoreDisplay: player score counter (top-right)
- [ ] 7.9: Implement BriefingOverlay: mission briefing text display
- [ ] 7.10: Implement SettingsPanel: graphics, audio, controls settings
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Shield bar updates: reflects current shield level correctly
  - Crosshair changes: different appearance when locked on target
  - Status message displayed: text appears and scrolls off screen
  - Velocity display: shows correct velocity magnitude
  - Throttle bar: reflects current throttle level
  - Speed readout: shows correct speed in km/s
  - Score display: updates when targets destroyed
  - Briefing overlay: displays mission briefing text correctly
  - Settings panel: settings saved and restored on reload
- **Sad paths:**
  - Shield at zero: bar empty, no crash
  - No targets locked: crosshair shows default appearance
  - Status message queue full: oldest message removed
  - Negative velocity: displays as absolute value

**Completion Criteria:** All tests pass with ≥90% coverage. HUD fully functional: all elements display correctly.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (HUD elements display correctly) and sad path (zero shields, no targets) tests must pass.**

## Sub-tasks

- [ ] 7.2: Implement ShieldBar: left/right shield indicators
- [ ] 7.3: Implement Crosshair: center screen, changes with target lock
- [ ] 7.4: Implement StatusMessage: scrollable text messages
- [ ] 7.5: Implement VelocityDisplay: velocity magnitude readout
- [ ] 7.6: Implement ThrottleBar: throttle level indicator
- [ ] 7.7: Implement SpeedReadout: current speed display
- [ ] 7.8: Implement ScoreDisplay: player score counter
- [ ] 7.9: Implement BriefingOverlay: mission briefing text display
- [ ] 7.10: Implement SettingsPanel: graphics, audio, controls settings
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-7-1-implement-iradar-circular-radar-overlay](phase-7-1-implement-iradar-circular-radar-overlay.md)
- [depends-on phase-2-3-implement-ship-state-machine-with-shields-death-timer](phase-2-3-implement-ship-state-machine-with-shields-death-timer.md)
- [depends-on phase-8-1-implement-event-evaluator-all-trigger-types](phase-8-1-implement-event-evaluator-all-trigger-types.md)

## History

- type: created
  date: 2026-11-24T04:25:45.105Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
