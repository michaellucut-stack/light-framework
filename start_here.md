# Start Here: Sample Prompt Sequence

A concrete example using the pipeline framework to build a reaction speed game
with a server scoreboard and web client.

**Key principles baked into every prompt**:
- AI NEVER invents requirements — it asks you via AskUserQuestion
- AI CHALLENGES your decisions when trade-offs are material to the project
- Implementation, tests, and docs run as parallel background agents (max 4)
- Main thread = only questions, challenges, and orchestration
- Unit tests every slice, integration tests every refactor (from requirements, not code)
- MINIMIZE phase after every refactor to keep code lean
- Architecture Manifest = contracts + dependency graph + data flows (no source parsing)

**Model allocation** (mandatory for all agent dispatches):

| Model | Work Type |
|-------|-----------|
| **Opus** | Architecture, planning, refinement, challenges, analysis |
| **Sonnet** | Code implementation, unit tests, integration tests, refactoring |
| **Haiku** | Documentation, manifest updates, session log, tracking files |

---

## Step 0: Fill in CLAUDE.md (you do this manually, once)

Fill in your project-specific details. The AI Workflow Instructions (8 rules)
are already there. They tell AI to: ask questions, challenge decisions, use
background agents with correct models, keep code minimal, and maintain the
Architecture Manifest.

```markdown
# Project: Reaction Speed Game

## Overview
A browser-based reaction speed game where players click a target as fast
as possible. Scores submitted to a server, displayed on a global leaderboard.

## Tech Stack
- Frontend: Vanilla HTML/CSS/JS (single page, no framework)
- Backend: Node.js + Express
- Database: SQLite (via better-sqlite3, zero setup)
- Testing: Vitest (backend unit + integration)

## Active Decisions
(empty — will be populated as AI challenges and you confirm decisions)
```

---

## Prompt 1 — PLAN Phase

```
Read CLAUDE.md. I'm building a reaction speed game with a server scoreboard.

Help me define vertical slices. The game should have:
- A reaction test (click when the screen turns green, measure time)
- Score submission with player name
- A leaderboard showing top 10 scores
- A "play again" flow

Break this into 3-4 vertical slices (each going from UI to database).
For each slice define: goal, layers touched, acceptance criteria,
and what unit tests should cover. Write to .pipeline/slices.md.

Before defining slices, challenge my tech stack and architecture choices.
For each concern, use the CHALLENGE format from Rule 7.
Only raise challenges that are MATERIAL to this specific project.
Record confirmed decisions (with reasoning) in CLAUDE.md Active Decisions.
Ask me questions for anything that isn't obvious.

If codebase exploration is needed, spawn up to 4 Opus agents in parallel.
```

**Model**: Main thread (Opus-level for challenges). Exploration: Opus agents.

**What happens**:
- AI reads CLAUDE.md (auto + explicit)
- AI CHALLENGES before planning:

```
CHALLENGE: SQLite for a multiplayer game scoreboard
FOR THIS PROJECT: concurrent score submissions from multiple players
PROS of current choice: zero config, single file, no separate DB server
CONS for THIS project: single-writer lock — if 10 players submit
  scores simultaneously, they queue. Acceptable at small scale.
ALTERNATIVE: PostgreSQL — handles concurrency natively, but requires
  setup and a running server
RECOMMENDATION: keep for v1 (< 100 concurrent users), revisit if scaling
```

- AI presents via AskUserQuestion: "Keep SQLite?" / "Switch to PostgreSQL" / "Discuss further"
- You answer: "Keep SQLite for v1"
- AI records in Active Decisions: "SQLite — chosen over PostgreSQL because v1 targets < 100 users, zero-config is worth the single-writer tradeoff"
- AI then proposes slices
- AI asks clarification questions: "Should the leaderboard be on the same page or a separate route?"
- You answer, AI records to slices.md

---

## Prompt 2 — REFINE First Slice (mandatory before implementing)

```
Read CLAUDE.md. Refine SLICE-001 from .pipeline/slices.md.
Identify every ambiguity, gap, or assumption.
Challenge the approach for this slice — patterns, data model, libraries.
Use the CHALLENGE format for any concern material to this project.
Ask me all questions before starting implementation.
Do NOT write any code until all questions are answered.
Record my answers AND challenge outcomes in the Clarifications Log
and Challenges Log for this slice.
```

**Model**: Main thread only (Opus-level reasoning). No background agents —
this is an interactive dialogue.

**What happens** (main thread only — no code yet):

AI asks **clarifications**:
- "The slice mentions 'set up database with scores table' — what columns?
  Just (id, name, time, created_at) or do you also want (ip, user_agent, round_count)?"
- "Should the server auto-create the DB file on startup, or should there
  be a separate migration command?"
- "For the minimal index.html — do you want just a blank page confirming
  connection, or should it already have the game layout skeleton?"

AI raises **challenges**:

```
CHALLENGE: Separate migration command vs auto-create on startup
FOR THIS PROJECT: a simple game with one table, no team of DBAs
PROS of migration command: standard practice, explicit control
CONS for THIS project: adds a step to dev workflow for a single table,
  over-engineering for v1
ALTERNATIVE: auto-create on startup with a simple CREATE IF NOT EXISTS
RECOMMENDATION: auto-create — simpler, and you can add migrations later
  if schema grows
```

- You answer each question and each challenge
- AI records in Clarifications Log and Challenges Log
- AI signals: "SLICE-001 is fully refined. Ready to implement."

---

## Prompt 3 — IMPLEMENT First Slice (3 background agents)

```
All questions for SLICE-001 are answered. Implement it now.
Run these as parallel background agents (3 agents):
1. (Sonnet) Implementation: build the slice bottom-up (DB → server → static page)
2. (Sonnet) Unit tests: write tests for DB initialization and server startup
3. (Haiku)  Documentation: update the Architecture Manifest (.pipeline/codebase-map.md)
            with initial module contracts, dependency graph, and data flows
Report back when all three are done. Ask me if anything comes up.
```

**Models**: Sonnet x2 (code + tests), Haiku x1 (docs). 3 agents total.

**What happens**:
- Three background agents start simultaneously
- Main thread is free — you can review slices.md, think ahead, etc.
- If an agent hits an ambiguity, AI asks you in main thread
- When all complete: AI reports "All done. 4 unit tests passing. Server starts on port 3000."

---

## Prompt 4 — CHECKPOINT (4 agents with post-implementation review)

```
SLICE-001 is done. Do these in parallel (4 agents):
1. (Haiku)  Update .pipeline/slices.md — mark SLICE-001 as DONE
2. (Haiku)  Update CLAUDE.md — set last completed slice, active to SLICE-002
3. (Sonnet) Run the full unit test suite and report results
4. (Opus)   Post-implementation review: is this the simplest implementation?
            Flag anything over-engineered. Use CHALLENGE format.
            Write to Post-Implementation Review section for this slice.

Show me git status after all agents complete.
```

**Models**: Haiku x2 (tracking), Sonnet x1 (tests), Opus x1 (review). 4 agents.

**Example post-implementation challenge** (from the Opus agent):
```
CHALLENGE: Express error middleware was added with 3 error types
FOR THIS PROJECT: SLICE-001 only has one route (serve static files)
PROS of current choice: ready for future error handling
CONS for THIS project: 15 lines of code handling errors that can't occur yet
ALTERNATIVE: remove, add when SLICE-003 introduces API endpoints
RECOMMENDATION: remove — YAGNI, add back in SLICE-003 REFINE
```

---

## Prompt 5 — REFINE Second Slice

```
Read CLAUDE.md. Refine SLICE-002 from .pipeline/slices.md.
This is the game UI — the green-screen reaction test.
Challenge the approach for this slice.
Ask me all questions before starting implementation.
Record answers in the Clarifications Log, challenges in the Challenges Log.
```

**Model**: Main thread (Opus-level for challenges). No agents.

**Example clarifications AI asks**:
- "How long should the random delay be before the screen turns green? Fixed range or configurable?"
- "What happens if the player clicks before the screen turns green? (false start — penalty? restart? ignore?)"
- "Should there be a countdown ('3... 2... 1...') or just a 'Wait...' message?"

**Example challenges AI raises**:
```
CHALLENGE: Storing timing logic in a separate timer.js module
FOR THIS PROJECT: one timing operation (measure reaction) in one place
PROS of separate module: testable in isolation, reusable
CONS for THIS project: only used once, adds a file and an import for 10 lines
ALTERNATIVE: inline the timing in the game flow, extract later if reused
RECOMMENDATION: inline — only one use case exists right now
```

---

## Prompt 6 — IMPLEMENT Second Slice (3 background agents)

```
All questions for SLICE-002 are answered. Implement it now.
Run as parallel background agents (3 agents):
1. (Sonnet) Implementation: build the reaction test UI with timing logic
2. (Sonnet) Unit tests: test the timing measurement and false-start detection
3. (Haiku)  Documentation: update Architecture Manifest with frontend module contracts
Report back when done. Ask me if anything is unclear.
```

**Models**: Sonnet x2, Haiku x1. 3 agents.

---

## Prompt 7 — REFINE + IMPLEMENT Third Slice (score submission + leaderboard)

Always REFINE first (main thread, Opus-level):

```
Read CLAUDE.md. Refine SLICE-003 from .pipeline/slices.md.
This connects the game to the scoreboard (submit scores, view leaderboard).
Challenge the approach — especially the API design and data model decisions.
Ask me all questions before coding.
Record answers and challenge outcomes.
```

**Example challenge AI raises**:
```
CHALLENGE: Adding a service layer between route handlers and database
FOR THIS PROJECT: 2 endpoints (POST score, GET leaderboard), each with
  one DB query
PROS of service layer: separates concerns, testable business logic
CONS for THIS project: at 2 endpoints with trivial logic, it's pure
  pass-through — adds files and indirection for no behavior
ALTERNATIVE: call DB directly from route handlers, extract service
  when logic appears (validation rules, ranking calculations, etc.)
RECOMMENDATION: discuss — depends on whether SLICE-004+ will add
  complex scoring logic
```

Then after answering (3 background agents):

```
Implement SLICE-003 as parallel background agents (3 agents):
1. (Sonnet) Implementation: POST /api/scores, GET /api/scores, UI form + leaderboard
2. (Sonnet) Unit tests: test both endpoints (success, validation errors, edge cases)
3. (Haiku)  Documentation: update Architecture Manifest with API contracts and data flows
Report back. Ask me if unclear.
```

---

## Prompt 8 — ANALYZE (4 Opus agents in parallel)

```
Read CLAUDE.md. Phase is now ANALYZE.
Analyze the codebase for refactoring opportunities.
Spawn 4 Opus agents in parallel:
1. (Opus) Duplicated code across API endpoints + frontend JS organization
2. (Opus) Error handling consistency (server + client)
3. (Opus) Input validation coverage + database query patterns
4. (Opus) Security issues + review Active Decisions — should any be revisited
          given what we've learned? Only re-open with NEW evidence. CHALLENGE format.

Write findings to .pipeline/analysis/refactoring-001.md.
Ask me about anything you're unsure is a real issue vs intentional choice.
```

**Models**: Opus x4. Maximum parallelism for deep analysis.

**Example architecture challenge during ANALYZE**:
```
CHALLENGE: Revisit "no auth" decision
FOR THIS PROJECT: leaderboard is now live. Without auth, anyone can submit
  fake scores via curl. After 3 slices, this is now a real attack surface.
PROS of current choice: simpler, faster to build
CONS for THIS project: leaderboard integrity is the core feature —
  fake scores destroy it
ALTERNATIVE: add simple rate limiting (IP-based) as a pragmatic middle ground
  instead of full auth. Blocks casual abuse without login complexity.
RECOMMENDATION: discuss — rate limiting is low-effort and high-value here
```

---

## Prompt 9 — REFACTOR (Sonnet agents, then Haiku cleanup)

```
Read .pipeline/analysis/refactoring-001.md.
I approve items: [1, 3, 5]. Skip items: [2, 4].

Apply refactoring — one Sonnet agent per approved item (max 4 simultaneous):
- (Sonnet) Refactoring item 1
- (Sonnet) Refactoring item 3
- (Sonnet) Refactoring item 5

After all refactoring agents complete, run in parallel (3 agents):
1. (Sonnet) Run all unit tests — must still pass
2. (Haiku)  Update Architecture Manifest (contracts, dependencies, line counts)
3. (Haiku)  Update CLAUDE.md Architecture Snapshot + Active Decisions
```

**Models**: Sonnet for code changes + tests, Haiku for documentation.

---

## Prompt 10 — MINIMIZE (4 agents)

```
Read CLAUDE.md and the Architecture Manifest.
Phase is now MINIMIZE. Run 4 agents in parallel:

1. (Sonnet) Dead code: functions/variables/imports never called or used
2. (Sonnet) Over-abstraction: utilities/helpers used only once — inline them
3. (Sonnet) Premature generalization + redundant pass-through layers
4. (Haiku)  Size budget: update line counts per module, flag any exceeding budget

For each finding, state what you'd delete/simplify and how many lines it saves.
Ask me before deleting anything that might be intentional.
After applying: (Sonnet) run unit tests, (Haiku) update Architecture Manifest.
```

**Models**: Sonnet x3 (code analysis + deletion), Haiku x1 (tracking).

---

## Prompt 11 — INTEGRATION TESTS (2 Sonnet agents)

```
Read CLAUDE.md and .pipeline/slices.md (all completed slices + clarification logs).
Design integration tests based on the REQUIREMENTS, not the code.
Do NOT read src/ to design these tests — only read requirements and acceptance criteria.

Test these end-to-end flows:
- Player completes a reaction round and sees their time
- Player submits a score with their name
- Leaderboard shows top 10 scores in correct order
- Invalid inputs are rejected with appropriate errors
- [Any new requirements from challenge outcomes, e.g., rate limiting]

Run as 2 agents:
1. (Sonnet) Write integration tests based on slice requirements
2. (Sonnet) Execute integration tests and report results (after agent 1 completes)
Ask me if any requirement is ambiguous for testing.
```

**Models**: Sonnet x2 (sequential — write then execute).

---

## Prompt 12 — SESSION HANDOFF (3 Haiku agents)

```
Session wrap-up. Run 3 Haiku agents in parallel:
1. (Haiku) Write a handoff entry to .pipeline/session-log.md
2. (Haiku) Update .pipeline/slices.md with current statuses
3. (Haiku) Update CLAUDE.md current state section + verify Architecture Manifest
Then show me git status so I can review before committing.
```

**Models**: Haiku x3. Fast, cheap, parallel bookkeeping.

---

## Prompt 13 — RESUMING (new session, days/weeks later)

```
Read CLAUDE.md and the latest entry in .pipeline/session-log.md.
Pick up where we left off.
```

---

## Agent Dispatch Summary

| Phase | Opus | Sonnet | Haiku | Total | Notes |
|-------|------|--------|-------|-------|-------|
| PLAN | main thread | — | — | 0-4 | Opus agents if exploration needed |
| REFINE | main thread | — | — | 0 | Interactive dialogue, no agents |
| IMPLEMENT | — | 2 | 1 | 3 | Code + tests + docs |
| CHECKPOINT | 1 | 1 | 2 | 4 | Review + tests + tracking |
| ANALYZE | 4 | — | — | 4 | Maximum parallel deep analysis |
| REFACTOR | — | 1-4 + 1 | 2 | 3-7 | Batched if >4 items |
| MINIMIZE | — | 3 | 1 | 4 | Code analysis + size tracking |
| INTEGRATION | — | 2 | — | 2 | Write then execute |
| HANDOFF | — | — | 3 | 3 | Fast parallel bookkeeping |

---

## What You Experience as the Engineer

```
┌──────────────────────────────────────────────────────────────────┐
│ YOUR EXPERIENCE (main thread)                                    │
│                                                                  │
│ 1. You say: "Plan the game"                                     │
│ 2. AI CHALLENGES (Opus): "SQLite for concurrent  ← you decide   │
│    scores — keep or switch?"                                     │
│ 3. You say: "Keep for v1"                                        │
│ 4. AI proposes slices, asks questions            ← you answer    │
│ 5. You say: "Refine SLICE-001"                                   │
│ 6. AI asks (Opus): 3 clarifications + 1 challenge← you answer   │
│ 7. You say: "Implement it"                                       │
│ 8. ... 2x Sonnet + 1x Haiku agents work ...     ← you're free   │
│ 9. AI reports: "Done. 4 tests passing."                          │
│ 10. You say: "Checkpoint"                                        │
│ 11. ... Opus reviews, Sonnet tests, 2x Haiku    ← you're free   │
│     update tracking ...                                          │
│ 12. Opus CHALLENGES: "Error middleware is         ← you decide   │
│     over-engineered for 1 route — remove?"                       │
│ 13. You say: "Yes remove. Refine SLICE-002"                      │
│ 14. AI asks (Opus): 4 questions + 1 challenge    ← you answer    │
│ 15. You say: "Implement it"                                      │
│ 16. ... 2x Sonnet + 1x Haiku ...                ← you're free   │
│ 17. AI reports: "Done. 11 tests passing."                        │
│ 18. You say: "Refine SLICE-003"                                  │
│ 19. Opus CHALLENGES: "Service layer is            ← you decide   │
│     pass-through — inline?"                                      │
│ 20. You say: "Inline. Implement."                                │
│ 21. ... 2x Sonnet + 1x Haiku ...                ← you're free   │
│ 22. You say: "Analyze"                                           │
│ 23. ... 4x Opus agents analyze in parallel ...   ← you're free   │
│ 24. Opus CHALLENGES: "Revisit no-auth?"          ← you decide    │
│ 25. You say: "Add rate limiting. Refactor 1,3,5"                 │
│ 26. ... 3x Sonnet refactor, then 2x Haiku doc ..← you're free   │
│ 27. You say: "Minimize"                                          │
│ 28. ... 3x Sonnet + 1x Haiku ...                ← you're free   │
│ 29. AI: "23 dead lines, 1 unused helper. Delete?"← you approve   │
│ 30. You say: "Yes. Integration tests."                           │
│ 31. ... 2x Sonnet (write then run) ...           ← you're free   │
│ 32. AI: "9/9 integration tests pass"                             │
│ 33. You say: "Session wrap-up"                                   │
│ 34. ... 3x Haiku update all tracking ...         ← you're free   │
│ 35. Done.                                                        │
└──────────────────────────────────────────────────────────────────┘

Total prompts from you: ~15
Total decisions made: ~8-12 (challenges answered)
Total questions answered: ~12-18 (clarifications)
Total code written by you: 0
Total time waiting: minimal (background agents, max 4 parallel)
Models used: Opus for thinking, Sonnet for building, Haiku for bookkeeping
Result: lean code, no surprises, documented reasoning for every decision
```

---

## The 9-Phase Cycle at a Glance

```
 PLAN ──→ REFINE ──→ IMPLEMENT ──→ CHECKPOINT
(Opus)    (Opus)    (2xSonnet     (Opus review
                     1xHaiku)      Sonnet test
                                   2xHaiku track)
  │         │           │              │
  │      questions    background    post-impl
  │    + challenges    agents       review
  │                                    │
  │         ┌──────────────────────────┘
  │         ↓
  │    (repeat for each slice in batch)
  │         │
  │         ↓
  └──→ ANALYZE ──→ REFACTOR ──→ MINIMIZE ──→ INTEGRATION TEST
      (4xOpus)   (Sonnet code   (3xSonnet    (2xSonnet)
                  Haiku docs)    1xHaiku)         │
                                                  ↓
                                          SESSION HANDOFF
                                            (3xHaiku)
```

---

## The Pattern

```
Every interaction follows:

1. CHALLENGE → Opus surfaces trade-offs, you decide (main thread)
2. REFINE    → Opus asks clarifications, you answer (main thread)
3. EXECUTE   → Sonnet builds + tests, Haiku documents (parallel background, max 4)
4. REVIEW    → Opus checks for over-engineering, you approve (main thread)
5. RECORD    → Haiku updates tracking + Architecture Manifest (parallel background)
```

Opus thinks. Sonnet builds. Haiku records.
AI never guesses. AI never stays silent when it sees a problem.
You never wait. You never wonder why a decision was made.
Everything is recorded. Code stays minimal.
