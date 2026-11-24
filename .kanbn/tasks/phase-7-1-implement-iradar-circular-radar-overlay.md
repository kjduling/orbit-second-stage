---
tags:
  - 'type:feature'
  - 'phase:7'
  - 'area:ui'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:43:01.030Z
updated: 2026-11-24T04:59:07.196Z
---

# Phase 7.1: Implement IRadar (circular radar overlay)

**Task ID:** 7.1
**Phase:** 7 — HUD & UI
**Dependencies:** Phase 3 (Planets), Phase 6 (AI Targets)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §8 (HUD — radar details: circular, center screen, faction-colored targets)
- Consult `design/implementation_plan.md` §11 (Phase 7), §11.1 task 7.1, §11.2 (IRadar interface)
- Consult `design/gap_analysis.md` §3.4 (Server authority — radar shows server-authoritative target positions)

---

**Objective:**
Implement the circular radar overlay — the iconic center-screen radar showing all targets (red=enemy, green=friendly), planets (blue), and moons (cyan). Player position always at center. Range ring at configurable distance.

**Deliverables:**
- `src/hud/Radar.ts` — circular radar implementation using BabylonJS GUI or HTML overlay
- `tests/hud/radar.test.ts` — position mapping, range ring tests

**Subtasks:**
- [ ] Define `IRadar` interface: center, range, render(targets, planets)
- [ ] Implement circular radar background (dark circle with subtle border)
- [ ] Map target positions to radar coordinates (relative to player)
- [ ] Color targets by faction: red=enemy, green=friendly
- [ ] Color planets blue, moons cyan
- [ ] Draw range ring at configurable distance
- [ ] Player icon always at center of radar
- [ ] Scale radar display to fit screen
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Target within range: appears on radar at correct relative position
  - Target outside range: not shown on radar
  - Enemy target colored red, friendly target colored green
  - Planet colored blue, moon colored cyan
  - Range ring drawn at correct radius
- **Sad paths:**
  - Empty target list: radar renders empty circle without error
  - All targets outside range: radar shows only center dot
  - Zero range value: radar renders at minimum size

**Completion Criteria:** All tests pass with ≥90% coverage. Radar correctly displays targets, planets, and moons.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct position mapping) and sad path (empty list, zero range) tests must pass.**

## Sub-tasks

- [ ] Define IRadar interface with center, range, render methods
- [ ] Implement circular radar background with subtle border
- [ ] Map target positions to radar coordinates relative to player
- [ ] Color targets by faction (red=enemy, green=friendly)
- [ ] Color planets blue, moons cyan
- [ ] Draw range ring at configurable distance
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-3-1-define-planet-data-structure-initial-solar-system-30-planets-moons](phase-3-1-define-planet-data-structure-initial-solar-system-30-planets-moons.md)
- [depends-on phase-6-1-implement-target-entity-with-ai-strategy-system](phase-6-1-implement-target-entity-with-ai-strategy-system.md)

## History

- type: created
  date: 2026-11-24T03:43:01.030Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
