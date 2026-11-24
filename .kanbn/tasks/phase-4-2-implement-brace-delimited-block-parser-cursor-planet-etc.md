---
tags:
  - 'type:feature'
  - 'phase:4'
  - 'area:parsing'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T04:20:51.113Z
updated: 2026-11-24T04:58:25.356Z
---

# Phase 4.2: Implement brace-delimited block parser (Cursor, Planet, etc.)

**Task ID:** 4.2
**Phase:** 4 — Mission Parser
**Dependencies:** Phase 4.1 (Tokenizer)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §7 (Mission System — brace-delimited blocks: Cursor { ... }, Planet { ... })
- Consult `design/implementation_plan.md` §8.1 task 4.2 — Brace parser
- Consult `design/gap_analysis.md` §2.2 (Mission File Access — brace structure parsing)

---

**Objective:**
Implement the brace-delimited block parser that processes mission file blocks like Cursor { ... }, Planet { ... }, Target { ... }. Handles nested braces, key-value pairs within blocks, and block-level comments.

**Deliverables:**
- `src/missions/BlockParser.ts` — brace-delimited block parsing
- `tests/missions/block-parser.test.ts` — nested blocks, key-value parsing tests

**Subtasks:**
- [ ] Implement `BlockParser` class consuming token stream from Tokenizer
- [ ] Parse block header: keyword + opening brace (e.g., "Planet {")
- [ ] Parse key-value pairs within block: "Name Earth" → {Name: "Earth"}
- [ ] Handle nested blocks: Cursor { Planet { Name Earth } }
- [ ] Match closing brace to opening block
- [ ] Return structured AST: array of blocks with name, properties, children
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Simple block: "Planet { Name Earth Radius 6371 }" → {name: "Planet", props: {Name: "Earth", Radius: "6371"}}
  - Nested block: "Cursor { Planet { Name Earth } }" → nested structure preserved
  - Multiple blocks: "Planet {...} Planet {...}" → array of two blocks
  - Empty block: "Planet {}" → block with no properties
- **Sad paths:**
  - Unclosed brace: throws error with line number
  - Mismatched braces: throws error with expected vs actual token
  - Missing block keyword: throws error, suggests valid keywords

**Completion Criteria:** All tests pass with ≥90% coverage. Block parser correctly parses mission file structure.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (blocks parsed correctly) and sad path (unclosed braces, mismatched) tests must pass.**

## Sub-tasks

- [ ] Implement BlockParser class consuming token stream from Tokenizer
- [ ] Parse block header: keyword + opening brace
- [ ] Parse key-value pairs within block
- [ ] Handle nested blocks with proper structure
- [ ] Match closing braces to opening blocks
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-4-1-implement-mission-file-tokenizer-whitespace-separated-comment-aware](phase-4-1-implement-mission-file-tokenizer-whitespace-separated-comment-aware.md)

## History

- type: created
  date: 2026-11-24T04:20:51.113Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
