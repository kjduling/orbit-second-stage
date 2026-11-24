---
tags:
  - 'type:feature'
  - 'phase:2'
  - 'area:physics'
  - 'priority:high'
assigned: Qwen
progress: 0
created: 2026-11-24T04:14:13.545Z
updated: 2026-11-24T04:57:25.390Z
---

# Phase 2.8: Implement warp speed mechanics

**Task ID:** 2.8
**Phase:** 2 — Flight Physics & Controls
**Dependencies:** Phase 2.1 (FlightModel base), Phase 2.7 (Full stop)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §2.2.1 (Newtonian Physics — warp speed mechanics, WARP_COEFF = 0.2)
- Consult `design/implementation_plan.md` §6.1 task 2.8 — Warp speed implementation
- Consult `design/gap_analysis.md` §3.4 (Deterministic physics — warp must be deterministic)

---

**Objective:**
Implement warp speed mechanics: when throttle is held at maximum for a sustained period, ship enters warp mode where acceleration increases exponentially. Warp coefficient (WARP_COEFF = 0.2) accumulates with each frame of full throttle.

**Deliverables:**
- `src/player/WarpSystem.ts` — warp speed accumulation and activation
- `tests/player/warp-system.test.ts` — warp accumulation, activation tests

**Subtasks:**
- [ ] Implement `WarpSystem` class tracking throttle duration
- [ ] Accumulate warp charge: WARP_COEFF × deltaT per frame at full throttle
- [ ] Activate warp when charge reaches threshold (configurable)
- [ ] In warp mode: acceleration = DELTAV + WARP_COEFF × |velocity|
- [ ] Visual feedback: engine glow intensifies as warp charges
- [ ] Deactivate warp when throttle released or full stop pressed
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Full throttle for N seconds: warp charge increases proportionally
  - Warp activated: acceleration increases with WARP_COEFF factor
  - Warp deactivated when throttle released: returns to normal acceleration
  - Warp survives full stop: charge maintained, can re-engage
- **Sad paths:**
  - Partial throttle (50%): no warp charge accumulation
  - Intermittent throttle: charge resets on any throttle release
  - Warp at zero velocity: acceleration still applies (ship starts moving)

**Completion Criteria:** All tests pass with ≥90% coverage. Warp speed mechanics work correctly with accumulation and activation.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (warp accumulates and activates) and sad path (partial throttle resets charge) tests must pass.**

## Sub-tasks

- [ ] Implement WarpSystem class tracking throttle duration
- [ ] Accumulate warp charge: WARP_COEFF × deltaT per frame at full throttle
- [ ] Activate warp when charge reaches threshold
- [ ] In warp mode: acceleration = DELTAV + WARP_COEFF × |velocity|
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-2-1-implement-flight-model-base-interface-newtonian-impl](phase-2-1-implement-flight-model-base-interface-newtonian-impl.md)
- [depends-on phase-2-7-implement-full-stop-space-key-behavior](phase-2-7-implement-full-stop-space-key-behavior.md)

## History

- type: created
  date: 2026-11-24T04:14:13.545Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
