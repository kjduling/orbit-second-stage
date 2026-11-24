---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:core'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:17:09.011Z
updated: 2026-11-24T04:55:06.930Z
---

# Phase 0.4: Implement Vector3 (Z-up internal coordinate system)

**Task ID:** 0.4
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §14 (Data Structures) for coordinate system details
- Consult `design/implementation_plan.md` §4.1 task 0.4, §4.2 (Key Interfaces — Vector3 interface)
- Consult `design/gap_analysis.md` §3.5 (Coordinate System alignment — Z-up internal, Y-up for BabylonJS)

---

**Objective:**
Implement a robust Z-up internal Vector3 class with full vector math operations. This is the foundational data structure — every other system uses it. Must include conversion helpers to/from BabylonJS Y-up coordinates.

**Deliverables:**
- `src/core/Vector3.ts` — full Vector3 implementation
- `tests/core/vector3.test.ts` — ≥90% coverage
- Static methods: `zero()`, `fromBabylon(v)`, `toBabylon()`
- Instance methods: `add`, `subtract`, `scale`, `dot`, `cross`, `magnitude`, `magnitudeSquared`, `normalize`, `clone`

**Subtasks:**
- [ ] Define `Vector3` interface with all required properties and methods
- [ ] Implement static factory: `Vector3.zero()` returning (0, 0, 0)
- [ ] Implement `Vector3.fromBabylon(v: BABYLON.Vector3): Vector3` — Y-up to Z-up conversion
- [ ] Implement `toBabylon(): BABYLON.Vector3` — Z-up to Y-up conversion using `(x, -z, y)` formula
- [ ] Implement `add(other: Vector3): Vector3` — component-wise addition
- [ ] Implement `subtract(other: Vector3): Vector3` — component-wise subtraction
- [ ] Implement `scale(scalar: number): Vector3` — scalar multiplication
- [ ] Implement `dot(other: Vector3): number` — dot product
- [ ] Implement `cross(other: Vector3): Vector3` — cross product
- [ ] Implement `magnitude(): number` and `magnitudeSquared(): number`
- [ ] Implement `normalize(): Vector3` — return normalized vector (handle zero vector)
- [ ] Implement `clone(): Vector3` — deep copy
- [ ] Write unit tests with ≥90% coverage

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - `dot`: Known vectors (1,0,0)·(0,1,0) = 0, (1,2,3)·(4,5,6) = 32
  - `cross`: (1,0,0)×(0,1,0) = (0,0,1)
  - `normalize`: (3,4,0) → (0.6, 0.8, 0)
  - `add`: (1,2,3)+(4,5,6) = (5,7,9)
  - `subtract`: (5,5,5)-(2,2,2) = (3,3,3)
  - `scale`: (1,2,3)*2 = (2,4,6)
  - `magnitude`: (3,4,0) → 5.0
  - `fromBabylon/toBabylon`: Round-trip conversion preserves values
- **Sad paths:**
  - `normalize` of zero vector returns (0, 0, 0) without crashing
  - `clone` produces independent copy (mutating clone doesn't affect original)
  - `magnitudeSquared` is always non-negative
  - Cross product of parallel vectors = zero vector

**Completion Criteria:** All tests pass with ≥90% coverage. Vector3 is the data structure used throughout the entire codebase.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all operations correct) and sad path (edge cases handled) tests must pass.**

## Sub-tasks

- [ ] Define Vector3 interface with all required properties and methods
- [ ] Implement static factory: Vector3.zero()
- [ ] Implement Vector3.fromBabylon() Y-up to Z-up conversion
- [ ] Implement toBabylon() Z-up to Y-up conversion using (x, -z, y)
- [ ] Implement add, subtract, scale operations
- [ ] Implement dot, cross product operations
- [ ] Implement magnitude, magnitudeSquared, normalize, clone
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:17:09.011Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
