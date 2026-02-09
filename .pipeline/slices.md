# Slice Backlog

## How to Use
Each slice is a vertical cut through the application: UI -> Business Logic -> Data Layer.
Mark status as you go. After completing a batch of slices, trigger an ANALYZE phase.

**IMPORTANT**: Every slice MUST go through REFINE before IMPLEMENT.
AI must ask all questions, raise all challenges, and record outcomes before writing code.

## Status Legend
- [ ] **TODO** — not started
- [?] **REFINING** — questions and challenges being discussed
- [~] **IN PROGRESS** — actively being built (background agents running)
- [x] **DONE** — implemented, unit tests pass, committed
- [R] **REFACTORED** — touched during a refactoring pass
- [T] **INTEGRATION TESTED** — integration tests written and passing

---

## Slice Batch 1

### SLICE-001: [Name]
- **Goal**: <!-- What does this slice deliver end-to-end? -->
- **Layers touched**: <!-- UI | API | Service | Repository | DB Migration -->
- **Acceptance criteria**:
  - <!-- Criterion 1: specific, testable behavior -->
  - <!-- Criterion 2 -->
  - <!-- Criterion 3 -->
- **Unit test coverage**:
  - [ ] <!-- DB: what queries to test -->
  - [ ] <!-- Service: what logic to test -->
  - [ ] <!-- API: what endpoints/responses to test -->
  - [ ] <!-- UI: what interactions to test -->
- **Status**: TODO
- **Git tag**: <!-- e.g., slice-001-done -->

#### Clarifications Log
<!-- AI records answers from the REFINE phase here -->
<!-- Format: Q: [question] → A: [your answer] -->

#### Challenges Log
<!-- AI records challenges raised and their outcomes here -->
<!-- Format:
CHALLENGE: [what was questioned]
FOR THIS PROJECT: [why it matters here]
OUTCOME: [kept / changed / deferred] — [reasoning]
-->

#### Implementation Notes
<!-- AI records decisions made during implementation here -->

#### Post-Implementation Review
<!-- AI flags anything over-engineered or unnecessarily complex after building -->

---

### SLICE-002: [Name]
- **Goal**:
- **Layers touched**:
- **Acceptance criteria**:
  - [ ]
  - [ ]
- **Unit test coverage**:
  - [ ]
  - [ ]
- **Status**: TODO
- **Git tag**:

#### Clarifications Log

#### Challenges Log

#### Implementation Notes

#### Post-Implementation Review

---

### SLICE-003: [Name]
- **Goal**:
- **Layers touched**:
- **Acceptance criteria**:
  - [ ]
  - [ ]
- **Unit test coverage**:
  - [ ]
  - [ ]
- **Status**: TODO
- **Git tag**:

#### Clarifications Log

#### Challenges Log

#### Implementation Notes

#### Post-Implementation Review

---

## Refactoring Checkpoints

### After Batch 1
- **Analysis file**: `.pipeline/analysis/refactoring-001.md`
- **Date**:
- **Summary**:
- **Slices affected**:
- **Architecture challenges raised**: <!-- any decisions revisited -->
- **Integration tests**: <!-- path to integration test file(s) -->
- **Integration test result**: <!-- PASS / FAIL + details -->
