---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:core'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:19:56.725Z
updated: 2026-11-24T04:55:07.087Z
---

# Phase 0.8: Implement RNG (Mulberry32 seeded PRNG)

**Task ID:** 0.8
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §4.4 (Explosion System — randomized color and size)
- Consult `design/implementation_plan.md` §4.1 task 0.8, §4.2 (RNG interface)
- Consult `design/gap_analysis.md` §2.3 (Deterministic PRNG for reproducible physics/AI)

---

**Objective:**
Implement a deterministic seeded PRNG using Mulberry32 algorithm. Essential for reproducible explosions, random enemy spawns, and network-synchronized randomness.

**Deliverables:**
- `src/core/RNG.ts` — Mulberry32 seeded PRNG implementation
- `tests/core/rng.test.ts` — ≥90% coverage
- Methods: `seed(value)`, `next()`, `nextInt(min, max)`, `nextFloat(min, max)`

**Subtasks:**
- [ ] Implement Mulberry32 algorithm (32-bit PRNG with seed)
- [ ] Implement `seed(value: number)`: initialize PRNG state from seed
- [ ] Implement `next(): number` — returns float in [0, 1)
- [ ] Implement `nextInt(min: number, max: number): number` — integer in [min, max]
- [ ] Implement `nextFloat(min: number, max: number): number` — float in [min, max)
- [ ] Verify determinism: same seed → identical sequence
- [ ] Write unit tests with ≥90% coverage

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Determinism: seed(42).next() == seed(42).next() (same value)
  - Sequence: first 10 values from seed(1) match known sequence
  - `nextInt(0, 100)`: returns integer in [0, 100] inclusive
  - `nextFloat(0.0, 1.0)`: returns float in [0.0, 1.0)
  - Different seeds produce different sequences
- **Sad paths:**
  - `nextInt` with min > max: throws error or swaps bounds
  - `nextInt` with same min and max: always returns that value
  - `nextFloat` with min == max: always returns that value
  - Zero seed: handled correctly (no division by zero)
  - Negative seed: handled correctly

**Completion Criteria:** All tests pass with ≥90% coverage. RNG produces deterministic, reproducible sequences from any seed.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (deterministic sequence correct) and sad path (edge cases in bounds) tests must pass.**

## Sub-tasks

- [ ] Implement Mulberry32 algorithm with seed method
- [ ] Implement next() returning float in [0, 1)
- [ ] Implement nextInt(min, max) and nextFloat(min, max)
- [ ] Verify determinism: same seed produces identical sequence
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:19:56.725Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
