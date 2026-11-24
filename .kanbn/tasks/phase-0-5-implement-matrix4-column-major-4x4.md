---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:core'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:17:09.066Z
updated: 2026-11-24T04:55:06.963Z
---

# Phase 0.5: Implement Matrix4 (column-major 4x4)

**Task ID:** 0.5
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §14 (Data Structures) for position/orientation storage
- Consult `design/implementation_plan.md` §4.1 task 0.5 — Matrix4 deliverables and tests
- Consult `design/gap_analysis.md` §3.5 (Coordinate System alignment)

---

**Objective:**
Implement a column-major 4x4 matrix class for transform operations. Used by Ship, Target, Planet, and Camera systems for orientation and position transforms.

**Deliverables:**
- `src/core/Matrix4.ts` — full Matrix4 implementation
- `tests/core/matrix4.test.ts` — ≥90% coverage
- Methods: identity, rotation (axis-angle + Euler), translation, scale, multiply, transformVec3, transformNormal

**Subtasks:**
- [ ] Define `Matrix4` class with column-major storage (Float32Array of 16 elements)
- [ ] Implement static `identity()`: returns identity matrix
- [ ] Implement `multiply(other: Matrix4): Matrix4` — standard 4x4 matrix multiplication
- [ ] Implement `transformVec3(v: Vector3): Vector3` — apply transform to position vector
- [ ] Implement `transformNormal(v: Vector3): Vector3` — apply upper-left 3x3 to normal vector
- [ ] Implement static `rotationX(angle)`, `rotationY(angle)`, `rotationZ(angle)` — axis rotations
- [ ] Implement static `rotationAbout(axis: Vector3, angle: number): Matrix4` — axis-angle rotation
- [ ] Implement static `translation(x, y, z): Matrix4` — translation matrix
- [ ] Implement static `scale(x, y, z): Matrix4` — scale matrix
- [ ] Implement `clone(): Matrix4` — deep copy
- [ ] Write unit tests with ≥90% coverage

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Identity matrix leaves vectors unchanged: `identity × v = v`
  - Rotation about Z-axis by 90°: (1,0,0) → (0,1,0)
  - Translation by (3,4,5): (0,0,0) → (3,4,5)
  - Scale by (2,2,2): (1,1,1) → (2,2,2)
  - Multiply: `A × B` produces correct result for known matrices
  - Transform normal ignores translation component
- **Sad paths:**
  - Multiplication is not commutative: `A × B ≠ B × A` (verify this!)
  - Zero rotation angle produces identity
  - Scale by zero produces valid matrix (no NaN)
  - Transform with identity matrix preserves all values

**Completion Criteria:** All tests pass with ≥90% coverage. Matrix4 supports all transform operations needed by Ship, Target, and Camera systems.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (all operations correct) and sad path (edge cases handled) tests must pass.**

## Sub-tasks

- [ ] Define Matrix4 class with column-major Float32Array storage
- [ ] Implement identity(), multiply(), transformVec3(), transformNormal()
- [ ] Implement rotationX/Y/Z and rotationAbout(axis, angle)
- [ ] Implement translation() and scale() static factories
- [ ] Implement clone() deep copy
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:17:09.066Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
