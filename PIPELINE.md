# AI-Assisted Development Pipeline

A repeatable workflow for building software in vertical slices with AI,
preserving context across sessions and avoiding redundant codebase parsing.

---

## The Core Idea

```
CLAUDE.md is the brain. AI reads it every session.
The main thread is YOU — you answer questions and approve work.
Background agents do ALL the building, testing, and documentation.
You never start from zero. AI never invents requirements.
```

## Execution Model

```
┌─────────────────────────────────────────────────────┐
│  MAIN THREAD (what you see and interact with)       │
│                                                     │
│  Questions ←→ Your Answers ←→ Progress Updates      │
│                                                     │
├─────────────────────────────────────────────────────┤
│  BACKGROUND AGENTS (parallel, you don't wait)       │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ Implementation│  │Documentation │                 │
│  │ Agent         │  │Agent         │                 │
│  └──────────────┘  └──────────────┘                 │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ Unit Test    │  │ Integration  │                 │
│  │ Agent        │  │ Test Agent   │                 │
│  └──────────────┘  └──────────────┘                 │
└─────────────────────────────────────────────────────┘
```

## File Structure

```
project/
├── CLAUDE.md                      # AUTO-READ by AI every session
├── PIPELINE.md                    # This file (the process guide)
├── .pipeline/
│   ├── slices.md                  # Slice backlog + status + clarifications
│   ├── codebase-map.md            # Architecture snapshot (the cache)
│   ├── session-log.md             # Session handoff notes
│   └── analysis/
│       ├── refactoring-template.md
│       └── refactoring-001.md     # (created during first analysis)
└── src/
    └── ...
```

---

## The 9-Phase Cycle

### Phase 1: PLAN (start of project or new batch)

**You do**:
1. Define slices in `.pipeline/slices.md`
2. Each slice = one vertical feature (UI → logic → data)
3. Group slices into batches of 2-4

**Tell AI**:
```
Read CLAUDE.md. Help me break [feature] into vertical slices.
For each slice, define: goal, layers touched, acceptance criteria,
and what unit tests should cover. Write to .pipeline/slices.md.
Ask me questions for anything that isn't obvious.

Before defining slices, challenge my tech stack and architecture choices.
For each concern, use the CHALLENGE format from Rule 8.
Only raise challenges that are MATERIAL to this specific project.
Record confirmed decisions (with reasoning) in CLAUDE.md Active Decisions.
```

**What AI challenges during PLAN** (examples):
- "You chose Python/Flask for a game with real-time leaderboard. Flask has no
  native WebSocket support. Node.js + Express + ws would be simpler for this.
  Pros of Python: [X]. Cons for THIS project: [Y]. Keep or reconsider?"
- "SQLite is single-writer. If you expect concurrent score submissions,
  this could bottleneck. Acceptable for v1 with <100 users?"
- "You have 4 slices but slice 3 depends on all of slices 1 and 2.
  Consider reordering to reduce blocking."

### Phase 2: REFINE (mandatory, before each slice)

**This is the gate that prevents AI from inventing things.**

AI reads the slice definition, identifies every ambiguity, and asks
ALL questions upfront via AskUserQuestion — before writing a single line.

**Tell AI**:
```
Read CLAUDE.md. Refine SLICE-XXX from .pipeline/slices.md.
Identify every ambiguity, gap, or assumption in this slice.
Challenge the approach for this slice — patterns, libraries, data model.
Use the CHALLENGE format for any concern material to this project.
Ask me all questions before starting implementation.
Do NOT write any code until all questions are answered.
Record my answers AND challenge outcomes in the Clarifications Log.
```

**What AI does**:
1. Reads the slice definition and the Architecture Manifest
2. Identifies gaps (error handling? edge cases? validation rules? UI behavior?)
3. Challenges the approach — is this the simplest way? is the chosen pattern right for this?
4. Asks you via AskUserQuestion — batched: clarifications + challenges together
5. Records your answers AND challenge outcomes in `.pipeline/slices.md`
6. Only then signals readiness to implement

**Example clarification questions**:
- "The slice says 'validate player name' — what are the rules? Min/max length? Allowed characters?"
- "Should the leaderboard update in real-time or only on page refresh?"
- "What happens if score submission fails? Retry? Show error? Silently log?"

**Example challenges**:
- "This slice adds a service layer between the route and the DB query. With only
  one query (insert score), the service layer adds indirection for no benefit yet.
  RECOMMENDATION: inline the DB call in the route handler for now, extract a service
  when a second operation appears. Keep or add the layer?"
- "You're storing reaction time as INTEGER milliseconds. If you later want sub-ms
  precision (e.g., for tie-breaking), you'd need a migration. Store as REAL now?
  Tradeoff: REAL uses more storage but avoids future migration."

### Phase 3: IMPLEMENT (per slice — runs in background)

After REFINE is complete, implementation runs as background agents.
You stay in the main thread, free to answer follow-up questions.

**Tell AI**:
```
All questions for SLICE-XXX are answered. Implement it now.
Run these as parallel background agents:
1. Implementation: build the slice bottom-up (DB → service → API → UI)
2. Unit tests: write tests for each layer based on the slice requirements
3. Documentation: update inline docs and .pipeline/codebase-map.md
Report back when all three are done. Ask me if anything is unclear.
```

**What happens**:
- Background Agent 1: writes the code (DB migration → repository → service → API → UI)
- Background Agent 2: writes unit tests for each layer touched
- Background Agent 3: updates documentation and codebase map
- Main thread: you're free. AI asks you questions if any come up during implementation
- When all agents complete: AI reports results and runs tests

**Unit tests are mandatory**. Every slice must have tests covering:
- Each database query / migration
- Each service function
- Each API endpoint (request/response, error cases)
- UI behavior (if testable with the chosen framework)

### Phase 4: CHECKPOINT (after each slice)

**Tell AI**:
```
SLICE-XXX is done. Do these in parallel:
1. Update .pipeline/slices.md — mark SLICE-XXX as DONE, record what was built
2. Update CLAUDE.md — set last completed slice, advance active slice
3. Run the full unit test suite and report results
4. Show me git status so I can review before committing

Also: review what was just built. Is this the simplest implementation that
meets the requirements? Flag anything that looks over-engineered, unnecessarily
complex, or that could be done in fewer lines. Use the CHALLENGE format.
```

```bash
git add -A
git commit -m "checkpoint: SLICE-XXX complete"
git tag slice-xxx-done
```

### Phase 5: ANALYZE (after every 2-4 slices)

This is where you prevent architectural rot and **cache your analysis**.

**Tell AI**:
```
Read CLAUDE.md. Phase is now ANALYZE.
Analyze the codebase for refactoring opportunities.
Investigate these areas in parallel:
1. Duplicated code across modules
2. Consistency of patterns (error handling, validation, responses)
3. Naming and file organization
4. Performance concerns
5. Security issues

Also: challenge whether the current architecture is still right.
After N slices, patterns that made sense at slice 1 may no longer fit.
Review Active Decisions in CLAUDE.md — should any be revisited
given what we've learned? Use the CHALLENGE format for any concerns.
Only re-open a decision if there's NEW evidence it's wrong.

Write findings to .pipeline/analysis/refactoring-NNN.md.
For each finding, ask me if you're unsure whether it's a real issue
or an intentional design choice. Do not assume.
```

### Phase 6: REFACTOR (apply the analysis)

**You do**:
1. Review the analysis file, approve/reject items
2. Update CLAUDE.md: set `Phase: REFACTOR`

**Tell AI**:
```
Read .pipeline/analysis/refactoring-NNN.md.
I approve items: [1, 3, 5]. Skip items: [2, 4].
Apply approved refactoring as parallel background agents (one per item).
After all changes, in parallel:
1. Run unit tests — everything must still pass
2. Update .pipeline/codebase-map.md with the new architecture
3. Update CLAUDE.md Architecture Snapshot
```

### Phase 7: MINIMIZE (mandatory after each refactoring)

**The phase that keeps your codebase small.**

After refactoring, code may still have: dead paths, unused helpers created
"just in case", over-abstractions, or functions that are only called once
and could be inlined. This phase actively looks for code to DELETE.

**Tell AI**:
```
Read CLAUDE.md and the Architecture Manifest (.pipeline/codebase-map.md).
Phase is now MINIMIZE. Investigate in parallel:

1. Dead code: functions/variables/imports never called or used
2. Over-abstraction: utilities, helpers, or base classes used only once — inline them
3. Premature generalization: config options nobody changes, parameters always
   passed the same value, generic code with only one concrete use
4. Redundant layers: modules that just pass through to another module with
   no added logic — collapse them
5. Size budget: update line counts per module, flag any that exceed budget

For each finding, state what you'd delete/simplify and how many lines it saves.
Ask me before deleting anything that might be intentional.
Write findings alongside the refactoring analysis file.
```

**What AI does NOT do in this phase**:
- Does not add code
- Does not refactor for style
- Only simplifies and removes

**After MINIMIZE**:
1. Run unit tests — everything must still pass
2. Update the Architecture Manifest (contracts may have simplified, line counts changed)
3. Update Size Budget table

### Phase 8: INTEGRATION TESTS (mandatory after each refactoring + minimize)

**This is requirements-based testing, not code-based.**

AI reads the REQUIREMENTS (from slices.md, CLAUDE.md, and clarification logs)
and designs integration tests that verify the system still meets them.
AI does NOT read the implementation code to design these tests.

**Tell AI**:
```
Read CLAUDE.md and .pipeline/slices.md (all completed slices).
Design and implement integration tests based on the REQUIREMENTS, not the code.
Each test should verify an end-to-end user flow described in the slice definitions.
Do NOT read src/ to design these tests — only read requirements.

Run as parallel background agents:
1. Write integration tests based on slice requirements
2. Execute the integration tests and report results

Ask me if any requirement is ambiguous for testing purposes.
```

**What makes these tests valuable**:
- They verify the SPEC, not the implementation
- If refactoring broke a requirement, these catch it
- They survive code rewrites because they're based on behavior, not structure

### Phase 9: SESSION HANDOFF (end of any session)

**Tell AI**:
```
Session wrap-up. Do these as parallel background agents:
1. Write a handoff entry to .pipeline/session-log.md
2. Update .pipeline/slices.md with current statuses
3. Update CLAUDE.md current state section
Then show me git status so I can review before committing.
```

---

## The Key Insight: What Gets Cached and Where

| What                     | Where                           | Updated When             |
|--------------------------|---------------------------------|--------------------------|
| Project overview         | CLAUDE.md                       | Rarely                   |
| Module contracts         | Architecture Manifest           | Every slice + refactoring|
| Dependency graph         | Architecture Manifest           | Every slice + refactoring|
| Data flows               | Architecture Manifest           | Every slice + refactoring|
| Size budget (LOC)        | Architecture Manifest           | Every slice + minimize   |
| Slice status + answers   | slices.md                       | After each slice         |
| Refactoring findings     | analysis/refactoring-NNN.md     | During ANALYZE phase     |
| Session continuity       | session-log.md                  | End of every session     |
| Design decisions         | CLAUDE.md Active Decisions      | When decisions change    |
| Unit tests               | test/ directory                 | Every slice              |
| Integration tests        | test/integration/ directory     | After every refactoring  |

---

## Commands Cheat Sheet

### Starting a new session
```
"Read CLAUDE.md and the latest entry in .pipeline/session-log.md.
Pick up where we left off."
```

### Refining a slice (ALWAYS before implementing)
```
"Read CLAUDE.md. Refine SLICE-XXX. Ask me all questions before coding.
Record answers in the Clarifications Log."
```

### Implementing a slice (after refinement)
```
"Implement SLICE-XXX as parallel background agents:
implementation, unit tests, and documentation.
Ask me if anything is unclear."
```

### Triggering analysis
```
"Analyze the codebase in parallel:
duplications, patterns, naming, performance, security.
Write to .pipeline/analysis/refactoring-NNN.md.
Ask me about anything you're unsure of."
```

### Minimizing code after refactoring
```
"MINIMIZE phase. Investigate in parallel: dead code, over-abstraction,
premature generalization, redundant layers, size budget.
For each finding, state what to delete and lines saved.
Ask me before removing anything that might be intentional."
```

### Running integration tests after minimize
```
"Design integration tests from requirements in slices.md (not from code).
Run them in background and report results."
```

### Ending a session
```
"Session wrap-up in parallel background agents:
session-log, slices status, CLAUDE.md update.
Then show git status."
```

---

## Prompt Engineering: Parallelism and Expert Agents

### Rule 1: Lists Enable Parallelism, "Then" Kills It

```
SEQUENTIAL (slow):
"Look at the auth module. Then look at the API module.
Then look at the database module. Then compare them."

PARALLEL (fast):
"Analyze these modules for shared patterns and inconsistencies:
- src/auth/
- src/api/
- src/database/
Compare findings across all three."
```

### Rule 2: Scope Triggers Agent Type

| Request Scope | What Happens | Example |
|---------------|-------------|---------|
| **Narrow** (1-2 files) | Direct tool call | "Fix the null check in src/auth/login.ts" |
| **Medium** (one module) | Explore agent | "How does error handling work in auth?" |
| **Broad** (cross-cutting) | Multiple parallel agents | "Investigate validation, errors, and logging across the codebase" |
| **Architectural** | Plan agent | "Design the approach for adding notifications" |

### Rule 3: Say "in parallel" and "in background" Explicitly

```
"Investigate these areas in parallel as background agents:
1. How does auth work? (trace from UI to DB)
2. What validation patterns exist?
3. How are DB queries structured?
Report findings when all complete."
```

### Rule 4: Separate Research from Implementation

```
RESEARCH (parallel, fast):
"Before SLICE-005, research in parallel:
- How do existing slices handle validation?
- What's the current API error format?
- How are migrations structured?"

IMPLEMENTATION (sequential, after research, background):
"Now implement SLICE-005 in background following the patterns found.
Ask me if anything is unclear."
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Fix |
|-------------|-------------|-----|
| "First X, then Y, then Z" for independent tasks | Forces sequential | Bulleted list of independent items |
| "Read every file in src/" | Overwhelms context | Use codebase-map.md |
| Mixed research + implementation in one prompt | Can't parallelize | Split into two prompts |
| AI assumes when requirements are unclear | Wrong code, rework | AI must AskUserQuestion |
| Tests based on reading the code | Tests follow bugs, don't catch them | Tests from requirements only |
| Waiting for implementation in main thread | Blocked on slow work | Run in background agents |
| Skipping REFINE phase | AI invents requirements | REFINE is mandatory |
| Integration tests only at the end | Late feedback | Integration tests after each refactor |
| No MINIMIZE phase | Code grows forever, bloat | Delete dead code and over-abstractions after each refactor |
| Creating abstractions "for later" | YAGNI — unused code is negative value | Only abstract when used 2+ times RIGHT NOW |
| Not tracking line counts | No signal for when to split/simplify | Size Budget in Architecture Manifest |

---

## Adapting This Framework

**For a new project**: Copy `.pipeline/`, CLAUDE.md, PIPELINE.md.
Fill in project-specific sections. Start with Phase 1.

**For an existing project**: Start with ANALYZE to populate codebase map,
then define slices.

**Scaling up**: Each developer has their own session-log entries.
Codebase map and CLAUDE.md remain shared.
