---
tags:
  - 'type:feature'
  - 'phase:4'
  - 'area:parsing'
  - 'priority:critical'
assigned: Qwen
progress: 0
created: 2026-11-24T03:38:24.923Z
updated: 2026-11-24T04:58:25.296Z
---

# Phase 4.1: Implement mission file tokenizer (whitespace-separated, comment-aware)

**Task ID:** 4.1
**Phase:** 4 — Mission Parser
**Dependencies:** Phase 0.7 (EventBus), Phase 0.8 (RNG)
**Type:** Feature | **Priority:** Critical

---

**References:**
- Consult `design/Orbit.md` §7 (Mission System — text-based mission files with brace-delimited blocks)
- Consult `design/implementation_plan.md` §8 (Phase 4), §8.1 task 4.1, §8.2 (Mission File Parser Architecture)
- Consult `design/gap_analysis.md` §2.2 (Mission File Access — Platform Differences)

---

**Objective:**
Implement the mission file tokenizer — the first step in parsing `.msn` and `.inc` mission files. Handles whitespace-separated tokens, comment stripping (`/* ... */`), quoted strings for briefing text, and include file references.

**Deliverables:**
- `src/missions/Tokenizer.ts` — mission file tokenizer
- `tests/missions/parser.test.ts` — comment stripping, whitespace handling, include paths tests

**Subtasks:**
- [ ] Implement `tokenize(input: string): string[]` function
- [ ] Strip `/* ... */` block comments
- [ ] Split on whitespace (spaces, tabs, newlines)
- [ ] Handle quoted strings for briefing text (preserve spaces within quotes)
- [ ] Detect `Include filename` directives for include file resolution
- [ ] Handle line comments (`//` style, if present)
- [ ] Return array of tokens preserving order
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Simple mission text: `Cursor { Earth }` → ['Cursor', '{', 'Earth', '}']
  - Block comment stripped: `/* this is a comment */ Token` → ['Token']
  - Quoted string preserved: `"Hello World"` → ['Hello World'] (single token)
  - Multiple whitespace collapsed: `Token1    Token2` → ['Token1', 'Token2']
  - Include directive detected: `Include extra.msn` → includes 'Include', 'extra.msn'
- **Sad paths:**
  - Unclosed comment `/* text`: returns tokens up to EOF without crash
  - Unclosed quote `"text`: returns remaining text as single token
  - Empty input: returns empty array
  - Only comments: returns empty array

**Completion Criteria:** All tests pass with ≥90% coverage. Tokenizer correctly parses mission file text into token stream.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (correct tokenization) and sad path (unclosed comments/quotes handled) tests must pass.**

## Sub-tasks

- [ ] Implement tokenize(input: string): string[] function
- [ ] Strip /* ... */ block comments
- [ ] Split on whitespace, handle quoted strings for briefing text
- [ ] Detect Include filename directives
- [ ] Handle line comments if present
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-7-implement-event-bus-typed-pub-sub](phase-0-7-implement-event-bus-typed-pub-sub.md)
- [depends-on phase-0-8-implement-rng-mulberry32-seeded-prng](phase-0-8-implement-rng-mulberry32-seeded-prng.md)

## History

- type: created
  date: 2026-11-24T03:38:24.923Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling
