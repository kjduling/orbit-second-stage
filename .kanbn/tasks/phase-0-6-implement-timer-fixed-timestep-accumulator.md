---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:core'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:19:56.605Z
updated: 2026-11-24T04:55:07.004Z
---

# Phase 0.6: Implement Timer (fixed-timestep accumulator)

**Task ID:** 0.6
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §15 (Game Flow) for per-frame loop and delta time
- Consult `design/implementation_plan.md` §4.1 task 0.6, §4.2 (Timer interface)
- Consult `design/gap_analysis.md` §2.1 (Deterministic physics requirement for netcode)

---

**Objective:**
Implement a fixed-timestep accumulator pattern (1/60s tick). This is essential for deterministic physics and reproducible network simulation. The accumulator collects real time and releases fixed-size physics ticks.

**Deliverables:**
- `src/core/Timer.ts` — Timer implementation with fixed-timestep accumulator
- `tests/core/timer.test.ts` — ≥90% coverage
- Methods: `tick(now)`, `getDeltaT()`, `getAccumulator()`, `reset()`

**Subtasks:**
- [ ] Implement Timer class with fixed timestep of 1/60 seconds
- [ ] Implement `tick(now: number)`: accumulate elapsed time, release fixed ticks
- [ ] Implement `getDeltaT(): number` — returns fixed timestep (1/60)
- [ ] Implement `getAccumulator(): number` — raw accumulator value for interpolation
- [ ] Implement `reset(): void` — clear accumulator and state
- [ ] Support interpolation factor: accumulator / fixedTimestep (for smooth rendering)
- [ ] Write unit tests with ≥90% coverage

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Single tick at 60 FPS: accumulator stays near zero, deltaT = 1/60
  - Variable frame rate (30 FPS): accumulator collects 2 ticks per second
  - Variable frame rate (120 FPS): accumulator releases partial ticks correctly
  - Reset clears all state: accumulator returns to 0
  - Interpolation factor in [0, 1) range
- **Sad paths:**
  - Large deltaT (> 1 second, e.g., tab reactivation): processes multiple ticks without infinite loop
  - Negative deltaT (clock goes backward): handled gracefully (clamp to zero)
  - Zero deltaT: no ticks released, accumulator unchanged

**Completion Criteria:** All tests pass with ≥90% coverage. Timer correctly handles variable frame rates and large time gaps.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct accumulation) and sad path (large gaps, negative delta, zero delta) tests must pass.**

## Sub-tasks

- [ ] Implement Timer class with fixed timestep of 1/60 seconds
- [ ] Implement tick(now) with accumulator and tick release logic
- [ ] Implement getDeltaT(), getAccumulator(), reset()
- [ ] Support interpolation factor for smooth rendering
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:19:56.605Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
