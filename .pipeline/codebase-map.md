# Architecture Manifest

## Purpose
This file is the **complete mental model** of the codebase.
An AI agent reads THIS FILE ONLY to understand the system — no source parsing needed.

It has three layers, each answering a different question:
1. **Module Registry** → What exists and what does each piece do?
2. **Contracts** → What does each module expose? (public interface)
3. **Dependency Graph** → Who calls whom?

**Last updated**: _never_
**Last updated after**: _n/a_
**Total lines of code**: _0_

---

## Directory Structure
```
src/
  (not yet populated)
```

---

## Module Registry

| Module | Purpose | Key Files | Lines | Depends On |
|--------|---------|-----------|-------|------------|
| _example_ | _Handles user auth_ | _src/auth/index.ts_ | _45_ | _Database, Config_ |

---

## Module Contracts

The contract is what each module EXPOSES to other modules.
AI reads these to understand how to USE a module without reading its source.
Update after every slice and refactoring.

<!--
### [Module Name]
**Exposes**:
```
functionName(param: Type): ReturnType — what it does
functionName(param: Type): ReturnType — what it does
```
**Events/Signals** (if any):
```
eventName → when it fires, what payload
```
**Data shapes**:
```
TypeName { field: Type, field: Type }
```
**Error behavior**: how this module signals errors
**Side effects**: what external state it changes (DB writes, file I/O, network calls)
-->

---

## Dependency Graph

Which modules depend on which. Read top-to-bottom as "calls into."
An agent uses this to know the blast radius of a change.

```
[Not yet mapped]

Example when populated:
  UI Layer
    └── calls → API Client
         └── calls → Express Routes
              └── calls → Service Layer
                   └── calls → Repository Layer
                        └── calls → SQLite Database
```

---

## Data Flow

How data moves through the system for key operations.
Each flow maps to a user-visible behavior.

<!--
### [Flow Name] — e.g., "Submit Score"
```
User clicks submit
  → frontend collects { name, time }
  → POST /api/scores { name: string, time: number }
  → validate(name: 1-20 chars, time: > 0)
  → scoresRepository.insert({ name, time, created_at })
  → return { id, rank }
  → frontend shows "You ranked #N!"
```
-->

---

## Shared Patterns

Patterns that repeat across the codebase. AI MUST follow these when
adding new code. If a new slice needs a different pattern, ASK first.

<!-- Example:
- **Error handling**: all API endpoints use `try/catch` → `res.status(500).json({ error: message })`
- **Validation**: Zod schemas in `src/validation/`, one file per resource
- **DB access**: repository pattern, one file per table in `src/repositories/`
-->

---

## Size Budget

Track lines of code per module. If a module grows beyond its budget,
it's a signal to split or refactor. Updated after each slice.

| Module | Current Lines | Budget | Status |
|--------|--------------|--------|--------|
| _total_ | _0_ | _—_ | _—_ |

---

## Known Technical Debt
- _none yet_

## Cross-Cutting Concerns
<!-- Logging, error handling, auth, validation — how they work globally -->
- _none yet_
