---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:core'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:19:56.706Z
updated: 2026-11-24T04:55:07.034Z
---

# Phase 0.7: Implement EventBus (typed pub/sub)

**Task ID:** 0.7
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (project scaffolding)
**Type:** Feature | **Priority:** High

---

**References:**
- Consult `design/Orbit.md` §15 (Game Flow) for event-driven game loop
- Consult `design/implementation_plan.md` §4.1 task 0.7, §4.2 (EventBus interface)
- Consult `design/gap_analysis.md` §4 (EventBus pub/sub architectural decision)

---

**Objective:**
Implement a typed event bus for decoupled communication between game systems. Used by GameState, EventEvaluator, and network layer for cross-system messaging.

**Deliverables:**
- `src/core/EventBus.ts` — generic typed EventBus implementation
- `tests/core/event-bus.test.ts` — ≥90% coverage
- Methods: `subscribe`, `emit`, `unsubscribe`, one-time listeners

**Subtasks:**
- [ ] Define generic `EventBus<TEvents extends Record<string, unknown>>` class
- [ ] Implement `subscribe<K extends keyof TEvents>(event: K, handler): void`
- [ ] Implement `emit<K extends keyof TEvents>(event: K, data: TEvents[K]): void`
- [ ] Implement `unsubscribe<K extends keyof TEvents>(event: K, handler): void`
- [ ] Support one-time listeners (auto-unsubscribe after first emission)
- [ ] Handle errors in handlers gracefully (don't crash other handlers)
- [ ] Write unit tests with ≥90% coverage

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Subscribe + emit: handler receives correct event data
  - Multiple subscribers: all handlers called with same data
  - Unsubscribe: handler no longer receives events after removal
  - One-time listener: fires once then unsubscribes automatically
  - Typed events: TypeScript enforces correct event/data types
- **Sad paths:**
  - Emit to non-existent event type: TypeScript compile error (type safety)
  - Handler throws error: other handlers still execute
  - Unsubscribe non-existent handler: no error thrown
  - Subscribe same handler twice: both subscriptions tracked independently

**Completion Criteria:** All tests pass with ≥90% coverage. EventBus provides type-safe decoupled communication.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (subscribe/emit/unsubscribe work) and sad path (error isolation, type safety) tests must pass.**

## Sub-tasks

- [ ] Define generic EventBus<TEvents extends Record<string, unknown>> class
- [ ] Implement subscribe, emit, unsubscribe methods
- [ ] Support one-time listeners (auto-unsubscribe after first emission)
- [ ] Handle errors in handlers gracefully without crashing others
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T03:19:56.706Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
