# Light Framework

A reusable pipeline for AI-assisted software development using Claude Code.

Build software in vertical slices across multiple sessions without losing
context, without AI inventing requirements, and without code bloat.

---

## The Problem

When building software with AI across multiple sessions:

1. **Context loss** — every new session starts from zero. AI re-reads every file, misses prior decisions, and asks questions you already answered.
2. **AI hallucination** — AI invents requirements when things are ambiguous instead of asking.
3. **Code bloat** — AI over-engineers by default. Abstractions appear before they're needed. Dead code accumulates.
4. **No pushback** — AI is a yes-man. If you pick the wrong database for your use case, it builds on it silently.
5. **Sequential bottleneck** — you wait while AI reads, writes, and tests, one step at a time.

## The Solution

A set of markdown files and a 9-phase development cycle that:

- Gives AI a **persistent brain** (`CLAUDE.md`, auto-read every session)
- Caches architecture as an **Architecture Manifest** (module contracts, dependency graphs, data flows) so AI reasons about the codebase without parsing source files
- Forces a **REFINE gate** before every implementation — AI must ask all questions first
- Runs a **Devil's Advocate** at every phase — AI challenges your decisions when trade-offs are material
- Keeps code lean with a **MINIMIZE phase** that actively deletes dead code and over-abstractions
- Tests against **requirements, not code** — integration tests survive refactoring
- Runs implementation, testing, and documentation as **parallel background agents** — you only handle questions and decisions

---

## Quick Start

```bash
# 1. Copy the framework into your project
cp -r light_framework/.pipeline  your-project/.pipeline
cp light_framework/CLAUDE.md     your-project/CLAUDE.md
cp light_framework/PIPELINE.md   your-project/PIPELINE.md

# 2. Fill in CLAUDE.md with your project details
#    (name, overview, tech stack, conventions)

# 3. Start a Claude Code session and say:
```

```
Read CLAUDE.md. Help me break [my feature] into vertical slices.
Challenge my tech stack choices. Ask me questions for anything unclear.
Write slices to .pipeline/slices.md.
```

For a detailed walkthrough with copy-paste prompts, see `start_here.md`.

---

## File Structure

```
your-project/
├── CLAUDE.md                          # Auto-read by AI every session
│                                      # Contains: 8 mandatory rules, current state,
│                                      # architecture snapshot, active decisions log
├── PIPELINE.md                        # Full process guide with 9 phases,
│                                      # prompt templates, and anti-patterns
├── start_here.md                      # Step-by-step example with sample prompts
│
├── .pipeline/
│   ├── slices.md                      # Slice backlog with per-slice:
│   │                                  # goals, acceptance criteria, test checklists,
│   │                                  # clarifications log, challenges log,
│   │                                  # post-implementation review
│   │
│   ├── codebase-map.md                # Architecture Manifest:
│   │                                  # module registry, contracts (public interfaces),
│   │                                  # dependency graph, data flows, size budget
│   │
│   ├── session-log.md                 # Session handoff notes (most recent first)
│   │
│   └── analysis/
│       └── refactoring-template.md    # Copy per analysis cycle
│
└── src/                               # Your actual code
```

---

## The 9-Phase Cycle

```
 PLAN ──→ REFINE ──→ IMPLEMENT ──→ CHECKPOINT ──┐
  │         │           │              │          │
  │      questions    background    post-impl    │
  │    + challenges    agents       review       │
  │                                              │
  │              (repeat per slice)              │
  │                                              │
  └──→ ANALYZE ──→ REFACTOR ──→ MINIMIZE ──→ INTEGRATION TEST ──→ SESSION HANDOFF
```

| Phase | What Happens | Who Does the Work |
|-------|-------------|-------------------|
| **PLAN** | Define vertical slices. AI challenges tech stack. | Main thread |
| **REFINE** | AI asks all clarifications + challenges approach. No code. | Main thread |
| **IMPLEMENT** | Code + unit tests + docs built simultaneously. | Background agents |
| **CHECKPOINT** | Update tracking, run tests, post-implementation review. | Parallel |
| **ANALYZE** | Find refactoring opportunities. Revisit past decisions. | Parallel agents |
| **REFACTOR** | Apply approved changes. | Background agents |
| **MINIMIZE** | Delete dead code, inline over-abstractions, enforce size budgets. | Background agents |
| **INTEGRATION TEST** | Test against requirements (not code). | Background agents |
| **SESSION HANDOFF** | Save state for next session. | Parallel |

---

## The 8 Rules (in CLAUDE.md)

| Rule | Purpose |
|------|---------|
| 1. NEVER INVENT — ALWAYS ASK | AI stops and asks when anything is ambiguous |
| 2. SLICE REFINEMENT IS MANDATORY | All questions answered before any code is written |
| 3. TESTING PROTOCOL | Unit tests per slice, integration tests from requirements after refactoring |
| 4. BACKGROUND EXECUTION MODEL | Main thread = questions only. Agents do all building. |
| 5. PARALLELISM | Independent work runs simultaneously |
| 6. MINIMUM CODE — MAXIMUM COVERAGE | Fewest lines that satisfy requirements. Size budgets enforced. |
| 7. DEVIL'S ADVOCATE | AI challenges decisions using structured format. Records reasoning. |
| 8. ARCHITECTURE MANIFEST IS SOURCE OF TRUTH | AI reads contracts, not source files. Manifest always current. |

---

## Pros

**Solves the context problem**
- `CLAUDE.md` is auto-read every session — zero warm-up
- Architecture Manifest lets AI reason about the codebase without parsing files
- Session log bridges sessions even weeks apart

**Prevents AI mistakes**
- REFINE gate stops AI from inventing requirements
- Devil's Advocate catches bad tech/architecture decisions early
- Structured CHALLENGE format forces concrete trade-off analysis, not vague concerns
- All decisions recorded with reasoning — no "why did we do this?" moments

**Keeps code lean**
- MINIMIZE phase actively deletes code after every refactor
- Size budgets per module signal when to split
- Post-implementation review catches over-engineering immediately
- Rule 6 prevents premature abstraction ("three similar lines > one abstraction")

**Maximizes throughput**
- Implementation, tests, and docs run as parallel background agents
- Main thread only handles decisions and questions — you're never blocked
- Research and implementation are split — research parallelizes aggressively

**Testing that survives refactoring**
- Integration tests designed from requirements, not from reading code
- They verify the spec, not the implementation
- Refactoring can't silently break what the user expects

**Reusable across projects**
- Copy `.pipeline/`, `CLAUDE.md`, `PIPELINE.md` into any project
- Fill in project-specific sections and start
- Works for games, web apps, APIs, CLI tools — any software with layers

---

## Cons

**Overhead for small projects**
- A script that's 50 lines of Python doesn't need 9 phases, 7 tracking files, and an Architecture Manifest. This framework is designed for projects with multiple features, multiple sessions, and enough complexity that context loss is a real problem. For a quick script, it's overhead.

**Depends on AI discipline**
- The rules in `CLAUDE.md` are instructions to the AI, not enforced constraints. If the AI model is weak at following instructions, it may skip the REFINE phase, forget to challenge, or invent anyway. This framework works best with capable models (Claude Opus/Sonnet) that respect structured instructions.

**Architecture Manifest maintenance burden**
- The manifest must be updated after every slice and refactoring. If it falls out of date, AI starts making decisions based on stale information — which is worse than no manifest at all. The framework tries to mitigate this by making manifest updates part of every phase, but it relies on the engineer (or AI) actually doing it.

**Challenge fatigue**
- The Devil's Advocate mechanism can become noisy if the AI raises low-value challenges. The "MATERIAL to this project" filter is supposed to prevent this, but in practice the AI may over-challenge or under-challenge depending on how it interprets materiality. You may need to calibrate by telling AI to be more or less aggressive.

**Not validated at scale**
- This framework was designed through reasoning about best practices in AI-assisted development — vertical slices, session continuity, requirements-based testing, YAGNI. It has not been stress-tested on a 100-slice project with 50 sessions. Edge cases will appear: manifest growing too large, challenge logs becoming unwieldy, session log flooding.

**Sequential gating adds latency**
- REFINE must complete before IMPLEMENT can start. You can't start building while questions are still open. For an engineer who wants to "just start coding and figure it out," this feels slow. The trade-off is intentional — rework from wrong assumptions is slower — but it's still friction.

**Single-engineer focus**
- The session log and slice tracking assume one person. For a team, you'd need: per-developer session logs, conflict resolution for simultaneous manifest updates, and a way to merge challenge outcomes across branches. The "Scaling up" section in PIPELINE.md gestures at this but doesn't solve it.

---

## How It Could Be Improved

**Automation**
- A CLI tool or Claude Code hook that auto-runs the CHECKPOINT bookkeeping (update slices.md, CLAUDE.md, manifest, git tag) instead of prompting AI to do it. Reduces human prompts from ~15 to ~8 per batch.

**Manifest auto-generation**
- A script that parses the actual codebase (AST, imports, exports) and generates/validates the Architecture Manifest automatically. This would catch manifest drift — the biggest risk in the framework. Could run as a git pre-commit hook.

**Adaptive challenge threshold**
- Track which challenges were accepted vs dismissed. If 80% of challenges of type X are dismissed, reduce frequency. If a challenge led to a real change, increase sensitivity for that category. This would reduce challenge fatigue over time.

**Slice dependency visualization**
- A generated diagram showing slice dependencies, which slices are blocked, and the critical path through the batch. Currently this is implicit in the text. A visual would help for batches of 5+ slices.

**Metrics dashboard**
- Track across sessions: total lines of code, lines added/deleted per slice, test count, test coverage, challenge acceptance rate, time per phase. This data would reveal whether the framework is actually keeping code lean or just adding process.

**Multi-agent orchestration improvements**
- Currently the framework describes background agents conceptually, but the actual parallelism depends on Claude Code's capabilities. A custom MCP server or orchestration layer could enforce: "don't start IMPLEMENT until REFINE is marked complete in slices.md" as a hard gate rather than an instruction.

**Team support**
- Per-developer branches for session logs with automated merging. A shared Architecture Manifest with conflict detection. Challenge outcomes that propagate across branches. Slice assignment and ownership tracking.

**Integration test isolation**
- The framework says "design tests from requirements, don't read code." In practice, AI may still be influenced by knowing the codebase. A stronger isolation would be: export requirements to a separate file, spawn a fresh agent with no codebase access, and have it write tests purely from the spec.

**Graduated complexity**
- A "light mode" for small projects (just CLAUDE.md + slices.md, skip manifest and analysis) and "full mode" for larger ones. Currently it's all-or-nothing. An engineer starting a 3-slice weekend project doesn't need the MINIMIZE phase.

---

## Files Reference

| File | Purpose | When to Read | When to Update |
|------|---------|-------------|----------------|
| `CLAUDE.md` | AI brain — auto-read every session | Every session start | When state changes |
| `PIPELINE.md` | Process guide — how the 9 phases work | When learning the framework | Rarely |
| `start_here.md` | Copy-paste prompt examples | When starting first project | Never |
| `.pipeline/slices.md` | Slice backlog + clarifications + challenges | Before each slice | After each slice |
| `.pipeline/codebase-map.md` | Architecture Manifest | Before coding against other modules | After every slice + refactor |
| `.pipeline/session-log.md` | Session handoff notes | Start of every session | End of every session |
| `.pipeline/analysis/` | Refactoring analysis files | During ANALYZE/REFACTOR | During ANALYZE |

---

## Designed For

- Software engineers using Claude Code (or similar AI coding assistants) for multi-session development
- Projects with 3+ vertical slices and enough complexity to warrant structured tracking
- Engineers who want AI to challenge their decisions, not just execute them
- Codebases where you care about keeping code minimal and maintainable

## Not Designed For

- Quick scripts or one-file utilities
- Teams larger than 2-3 without adaptation
- Projects where you want AI to make all decisions autonomously
- Non-Claude AI tools (the CLAUDE.md auto-read feature is Claude Code specific; other tools would need equivalent configuration)
