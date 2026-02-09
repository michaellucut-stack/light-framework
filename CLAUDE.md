# Project: [PROJECT_NAME]

## Overview
<!-- One paragraph describing what this project does -->

## Tech Stack
<!-- List your stack -->

## Conventions
- Follow existing patterns in the codebase before introducing new ones
- Every slice must include unit tests for the layers it touches
- After every refactoring, integration tests are designed from requirements (not from code)
<!-- Add your own: naming, file structure, error handling, etc. -->

## AI Workflow Instructions — MANDATORY

### Rule 1: NEVER INVENT — ALWAYS ASK
- NEVER assume, guess, or invent requirements, behaviors, or design decisions
- When ANY aspect of a slice is ambiguous, STOP and use AskUserQuestion
- Ask BEFORE writing code, not after
- If a slice says "handle errors" but doesn't specify how → ASK
- If a slice says "validate input" but doesn't specify rules → ASK
- If there are multiple valid approaches → ASK which one the user prefers
- When in doubt: ASK. The cost of a question is near zero. The cost of rework is high.

### Rule 2: SLICE REFINEMENT IS MANDATORY
- Before implementing any slice, run a REFINE step
- Read the slice definition and identify every ambiguity, gap, or assumption
- Present ALL questions to the user via AskUserQuestion in a single batch
- Only proceed to implementation after ALL questions are answered
- Record answers in the slice's Clarifications Log in `.pipeline/slices.md`

### Rule 3: TESTING PROTOCOL
- **Unit tests**: designed and implemented as part of every slice, covering each layer touched
- **Integration tests**: designed AFTER refactoring, based on REQUIREMENTS (from slices.md and CLAUDE.md), NOT by reading the implementation code
- Integration tests validate that the system meets the original requirements end-to-end
- Tests must be written before marking a slice or refactoring as complete

### Rule 4: BACKGROUND EXECUTION MODEL
- **Main thread**: ONLY questions (AskUserQuestion) and orchestration (dispatching agents, tracking progress)
- **Background agents**: ALL implementation work, documentation, test writing, and file updates
- Implementation and documentation for a slice run as parallel background agents
- The user sees: questions → approvals → progress updates → results
- The user does NOT wait for code to be written — it happens in background

### Rule 5: PARALLELISM
- When researching multiple modules or areas, spawn parallel subagents
- Always read `.pipeline/codebase-map.md` before scanning source files directly
- Never re-parse the full codebase if the codebase map is up to date
- When the phase is ANALYZE, investigate independent areas in parallel
- Implementation and documentation for a slice are always parallel background tasks
- When updating tracking files at session end, do all updates in parallel

### Rule 6: MINIMUM CODE — MAXIMUM COVERAGE
- Write the fewest lines of code that fully satisfy the requirements
- Before creating a new abstraction, function, or file → check if an existing one can be extended
- Before adding a utility → verify it will be used more than once, otherwise inline it
- After every slice: count lines per module, update Size Budget in the Architecture Manifest
- During MINIMIZE phase: actively look for code to DELETE — dead code, unused imports, over-abstractions, premature generalizations
- Three similar lines are better than a premature abstraction
- If a module exceeds its size budget → it must be split or simplified before the next slice

### Rule 7: DEVIL'S ADVOCATE — CHALLENGE EVERY DECISION
- At every phase, actively look for decisions that may hurt THIS project
- Do NOT be a yes-man. If the user's choice has trade-offs, surface them BEFORE implementing
- Challenge scope: tech stack, libraries, architecture patterns, data models, API design, UI approach
- ONLY challenge when the concern is MATERIAL to this project (not style preferences or theoretical purity)
- Use this structured format for every challenge:

```
CHALLENGE: [what's being questioned]
FOR THIS PROJECT: [why it matters specifically here, not in general]
PROS of current choice: [concrete benefits]
CONS for THIS project: [concrete risks or limitations]
ALTERNATIVE: [what else could work, with its own trade-offs]
RECOMMENDATION: keep / reconsider / discuss further
```

- Present challenges via AskUserQuestion with options: "Keep current choice", "Switch to alternative", "Discuss further"
- Record the decision AND the reasoning in CLAUDE.md Active Decisions
- Active Decisions format: "[Decision] — chosen over [Alternative] because [Reason]"
- Timing for challenges:
  - **PLAN phase**: challenge tech stack, architecture, and slice boundaries
  - **REFINE phase**: challenge approach for the specific slice (patterns, libraries, data model)
  - **ANALYZE phase**: challenge whether existing patterns are still the right fit
  - **Post-IMPLEMENT**: challenge whether the result is the simplest path that meets requirements
- Do NOT challenge during IMPLEMENT itself — that's too late, challenges belong in REFINE
- A decision that was challenged and confirmed does NOT get re-challenged unless new information appears

### Rule 8: ARCHITECTURE MANIFEST IS THE SOURCE OF TRUTH
- `.pipeline/codebase-map.md` is the **Architecture Manifest** — it contains module contracts, dependency graph, data flows, and size budgets
- Before implementing any code that calls another module → read that module's CONTRACT in the manifest, not its source
- After every slice and refactoring → update the manifest: contracts, dependencies, line counts, data flows
- If the manifest is outdated or missing a module → update it BEFORE proceeding
- The manifest is what allows agents to work WITHOUT parsing source files

## Current State
- **Active slice**: _none yet_
- **Last completed slice**: _none yet_
- **Phase**: PLAN <!-- PLAN | REFINE | IMPLEMENT | CHECKPOINT | ANALYZE | REFACTOR | MINIMIZE | INTEGRATION TEST -->
- **Codebase health**: _not yet analyzed_

## Key Files (AI: read these for context, not the whole repo)
- `.pipeline/slices.md` — slice backlog, status, clarifications log, test checklists
- `.pipeline/codebase-map.md` — **Architecture Manifest**: contracts, dependencies, data flows, size budgets
- `.pipeline/session-log.md` — session handoff notes
- `.pipeline/analysis/` — saved refactoring analyses

## Architecture Snapshot
```
[No architecture snapshot yet. Will be populated after first analysis.]
```

## Active Decisions
<!-- Decisions that were challenged, discussed, and confirmed. -->
<!-- Format: [Decision] — chosen over [Alternative] because [Reason] -->
<!-- This log prevents re-challenging settled decisions and captures WHY. -->
<!-- Only re-open a decision if NEW INFORMATION makes it relevant again. -->
