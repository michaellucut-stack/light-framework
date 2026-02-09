# Start Here: Sample Prompt Sequence

A concrete example using the pipeline framework to build a reaction speed game
with a server scoreboard and web client.

**Key principles baked into every prompt**:
- AI NEVER invents requirements — it asks you via AskUserQuestion
- AI CHALLENGES your decisions when trade-offs are material to the project
- Implementation, tests, and docs run as parallel background agents
- Main thread = only questions, challenges, and orchestration
- Unit tests every slice, integration tests every refactor (from requirements, not code)
- MINIMIZE phase after every refactor to keep code lean
- Architecture Manifest = contracts + dependency graph + data flows (no source parsing)

---

## Step 0: Fill in CLAUDE.md (you do this manually, once)

Fill in your project-specific details. The AI Workflow Instructions (8 rules)
are already there. They tell AI to: ask questions, challenge decisions, use
background agents, keep code minimal, and maintain the Architecture Manifest.

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
```

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

## Prompt 3 — IMPLEMENT First Slice (background agents)

```
All questions for SLICE-001 are answered. Implement it now.
Run these as parallel background agents:
1. Implementation: build the slice bottom-up (DB → server → static page)
2. Unit tests: write tests for DB initialization and server startup
3. Documentation: update the Architecture Manifest (.pipeline/codebase-map.md)
   with initial module contracts, dependency graph, and data flows
Report back when all three are done. Ask me if anything comes up.
```

**What happens**:
- Three background agents start simultaneously
- Main thread is free — you can review slices.md, think ahead, etc.
- If an agent hits an ambiguity, AI asks you in main thread
- When all complete: AI reports "All done. 4 unit tests passing. Server starts on port 3000."

---

## Prompt 4 — CHECKPOINT (with post-implementation review)

```
SLICE-001 is done. Do these in parallel:
1. Update .pipeline/slices.md — mark SLICE-001 as DONE
2. Update CLAUDE.md — set last completed slice to SLICE-001, active to SLICE-002
3. Run the full unit test suite and report results
4. Update Size Budget in the Architecture Manifest with line counts
5. Show me git status so I can review before committing

Also: review what was just built. Is this the simplest implementation that
meets the requirements? Flag anything over-engineered or unnecessarily complex.
Use the CHALLENGE format. Write findings to the Post-Implementation Review
section for this slice.
```

**Example post-implementation challenge**:
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

## Prompt 6 — IMPLEMENT Second Slice (after refinement)

```
All questions for SLICE-002 are answered. Implement it now.
Run as parallel background agents:
1. Implementation: build the reaction test UI with timing logic
2. Unit tests: test the timing measurement and false-start detection
3. Documentation: update Architecture Manifest with frontend module contracts
Report back when done. Ask me if anything is unclear.
```

---

## Prompt 7 — REFINE + IMPLEMENT Third Slice (score submission + leaderboard)

Always REFINE first:

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

Then after answering:

```
Implement SLICE-003 as parallel background agents:
1. Implementation: POST /api/scores, GET /api/scores, UI form + leaderboard
2. Unit tests: test both endpoints (success, validation errors, edge cases)
3. Documentation: update Architecture Manifest with API contracts and data flows
Report back. Ask me if unclear.
```

---

## Prompt 8 — ANALYZE (after 3 slices)

```
Read CLAUDE.md. Phase is now ANALYZE.
Analyze the codebase for refactoring opportunities.
Investigate these areas in parallel:
1. Duplicated code across API endpoints
2. Frontend JS organization
3. Error handling consistency (server + client)
4. Input validation coverage
5. Database query patterns

Also: challenge whether the current architecture is still right.
After 3 slices, review Active Decisions in CLAUDE.md — should any be
revisited given what we've learned? Only re-open a decision if there's
NEW evidence. Use the CHALLENGE format.

Write findings to .pipeline/analysis/refactoring-001.md.
Ask me about anything you're unsure is a real issue vs intentional choice.
```

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

## Prompt 9 — REFACTOR (after reviewing analysis)

```
Read .pipeline/analysis/refactoring-001.md.
I approve items: [1, 3, 5]. Skip items: [2, 4].
Apply approved refactoring as parallel background agents.
After all changes, in parallel:
1. Run all unit tests — must still pass
2. Update the Architecture Manifest (contracts, dependencies, line counts)
3. Update CLAUDE.md Architecture Snapshot
4. Record any revisited Active Decisions with updated reasoning
```

---

## Prompt 10 — MINIMIZE (mandatory after refactoring)

```
Read CLAUDE.md and the Architecture Manifest.
Phase is now MINIMIZE. Investigate in parallel:

1. Dead code: functions/variables/imports never called or used
2. Over-abstraction: utilities, helpers, or base classes used only once — inline them
3. Premature generalization: config options nobody changes, parameters always
   passed the same value, generic code with only one concrete use
4. Redundant layers: modules that just pass through to another with no added logic
5. Size budget: update line counts per module, flag any that exceed budget

For each finding, state what you'd delete/simplify and how many lines it saves.
Ask me before deleting anything that might be intentional.
After applying: run unit tests, update Architecture Manifest.
```

---

## Prompt 11 — INTEGRATION TESTS (mandatory after minimize)

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

Run as parallel background agents:
1. Write integration tests
2. Execute them and report results
Ask me if any requirement is ambiguous for testing.
```

---

## Prompt 12 — SESSION HANDOFF

```
Session wrap-up. Do these as parallel background agents:
1. Write a handoff entry to .pipeline/session-log.md
2. Update .pipeline/slices.md with current statuses
3. Update CLAUDE.md current state section
4. Verify Architecture Manifest is up to date
Then show me git status so I can review before committing.
```

---

## Prompt 13 — RESUMING (new session, days/weeks later)

```
Read CLAUDE.md and the latest entry in .pipeline/session-log.md.
Pick up where we left off.
```

---

## What You Experience as the Engineer

```
┌──────────────────────────────────────────────────────────────────┐
│ YOUR EXPERIENCE (main thread)                                    │
│                                                                  │
│ 1. You say: "Plan the game"                                     │
│ 2. AI CHALLENGES: "SQLite for concurrent        ← you decide    │
│    scores — keep or switch?"                                     │
│ 3. You say: "Keep for v1"                                        │
│ 4. AI proposes slices, asks questions            ← you answer    │
│ 5. You say: "Refine SLICE-001"                                   │
│ 6. AI asks: 3 clarifications + 1 challenge       ← you answer    │
│ 7. You say: "Implement it"                                       │
│ 8. ... background agents work ...                ← you're free   │
│ 9. AI reports: "Done. 4 tests passing."                          │
│ 10. AI CHALLENGES: "Error middleware is          ← you decide    │
│     over-engineered for 1 route — remove?"                       │
│ 11. You say: "Yes, remove. Checkpoint."                          │
│ 12. You say: "Refine SLICE-002"                                  │
│ 13. AI asks: 4 questions + 1 challenge           ← you answer    │
│ 14. You say: "Implement it"                                      │
│ 15. ... background agents work ...               ← you're free   │
│ 16. AI reports: "Done. 11 tests passing."                        │
│ 17. You say: "Refine SLICE-003"                                  │
│ 18. AI CHALLENGES: "Service layer is             ← you decide    │
│     pass-through with 2 endpoints — inline?"                     │
│ 19. You say: "Inline for now. Implement."                        │
│ 20. ... background agents work ...               ← you're free   │
│ 21. You say: "Analyze"                                           │
│ 22. AI CHALLENGES: "Revisit no-auth?             ← you decide    │
│     Leaderboard is now a real attack surface"                    │
│ 23. You say: "Add rate limiting. Refactor items 1,3,5"           │
│ 24. ... background agents work ...               ← you're free   │
│ 25. You say: "Minimize"                                          │
│ 26. AI: "Found 23 dead lines, 1 unused helper.   ← you approve  │
│     Delete?"                                                     │
│ 27. You say: "Yes. Integration tests."                           │
│ 28. ... background agents work ...               ← you're free   │
│ 29. AI reports: "9/9 integration tests pass"                     │
│ 30. You say: "Session wrap-up"                                   │
│ 31. Done.                                                        │
└──────────────────────────────────────────────────────────────────┘

Total prompts from you: ~15
Total decisions made: ~8-12 (challenges answered)
Total questions answered: ~12-18 (clarifications)
Total code written by you: 0
Total time waiting: minimal (background agents)
Result: lean code, no surprises, documented reasoning for every decision
```

---

## The 9-Phase Cycle at a Glance

```
 PLAN ──→ REFINE ──→ IMPLEMENT ──→ CHECKPOINT
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
         │                                        │
      challenge                                   │
      old decisions                               │
         │                                        ↓
         └──────────────────────────────→ SESSION HANDOFF
```

---

## The Pattern

```
Every interaction follows:

1. CHALLENGE → AI surfaces trade-offs, you decide (main thread)
2. REFINE    → AI asks clarifications, you answer (main thread)
3. EXECUTE   → background agents build + test + document (parallel)
4. REVIEW    → AI checks for over-engineering, you approve (main thread)
5. RECORD    → tracking files + Architecture Manifest updated (parallel)
```

AI never guesses. AI never stays silent when it sees a problem.
You never wait. You never wonder why a decision was made.
Everything is recorded. Code stays minimal.
