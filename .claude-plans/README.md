# `.claude-plans/` — Multi-Session Plan Conventions

Multi-session plans live here. They are the continuity mechanism that makes development survive across sessions, days, or weeks.

This README is the canonical reference for plan structure. `/create-plan` and `/work-plan` both rely on these conventions. Read it before authoring a new plan or resuming an existing one.

---

## Directory shape

```
.claude-plans/
├── README.md                      ← This file. Conventions reference.
├── <active-plan-name>/            ← Currently in progress
│   ├── OVERVIEW.md
│   ├── CHECKLIST.md
│   ├── 01-<chunk-name>.md
│   ├── 02-<chunk-name>.md
│   ├── ...
│   └── NN-capstone.md
└── archive/                       ← Completed plans
    ├── <past-plan-name>/
    └── <past-plan-name-v2>/       ← Versioned to avoid collisions
```

There is **never more than one active plan at a time** unless the user explicitly says otherwise. If you find two non-archived plan directories, ask before proceeding — it's almost certainly a leftover that should have been archived.

---

## File-by-file conventions

### `OVERVIEW.md`

The plan's vision, scope, and locked decisions. Read it on every plan resumption. Once authored, it changes rarely — append to it (Locked Decisions, Discovered Constraints), don't rewrite history.

```markdown
# <Plan Name>

## Goal
<What we're trying to accomplish overall — 1-3 sentences>

## Context
<Why this plan exists. The problem it solves. The state of the project at plan-creation time.>

## Scope
- **In scope**: <bulleted list of what's included>
- **Out of scope**: <bulleted list of what's explicitly excluded — often more important than the in-scope list>

## Locked Decisions
<Decisions made during plan authoring that should not be revisited per chunk. Examples:
- "Fresh install over incremental upgrade"
- "Prisma over TypeORM"
- "API-first; UI comes in a later plan"
Each decision should have a one-line rationale.>

## Risks
<Known risks at plan-creation time. Updated only when a risk materializes or is retired.>

## Chunks
<Brief list of all chunks with one-line descriptions. The CHECKLIST.md is the canonical progress tracker; this is just the at-a-glance index.>
```

### `CHECKLIST.md`

The canonical progress tracker. The orchestrator reads this on every `/work-plan` resumption.

```markdown
# <Plan Name> — Progress

## Status
- [x] 01 — <chunk name>
- [ ] 02 — <chunk name>
- [~] 03 — <chunk name> (in progress; blocked on <X>)
- [ ] 04 — <chunk name>
- [ ] NN — Capstone

## Notes
<Cross-cutting concerns, discoveries, blockers. Append, don't rewrite.>
```

Status markers: `[ ]` not started, `[~]` in progress (with block reason), `[x]` complete, `[-]` dropped (with rationale).

### `NN-<chunk-name>.md` — One per chunk

Each chunk file is the contract for one session of work. Authored at plan-creation time; gets a "Session Notes" section appended during execution.

```markdown
# <Chunk Title>

## Goal
<Specific objective for this chunk>

## Scope
- **Includes**: <specific items>
- **Excludes**: <what to leave for other chunks or different plans>

## Context
<What the implementer needs to know that isn't in OVERVIEW.md>

## Acceptance Criteria
- [ ] <Concrete, testable criterion>
- [ ] <Another>

## Dependencies
<Other chunks that must be completed first, or "None">

## Implementation Notes
<Technical guidance, suggested approach, files likely involved>

## Execution Strategy
<One of:
- **Executor-based** (4+ files with clear, mechanical changes — split into parallel executors)
- **Direct** (fewer files, nuanced judgment, or exploration needed first)
Be explicit. Vague execution strategies produce vague execution.>

## Drop-with-Prejudice Clause
<Always include this section. Workers MUST drop the chunk and document why in Session Notes if recon shows it's already satisfied or not worth the noise of touching the file. Concept-application without payoff is a regression.>

---
## Session Notes
<Empty at authoring. Implementer fills this in during work.>
```

### `NN-capstone.md` — Always last, always present

```markdown
# Capstone

## Goal
Final verification, cleanup, and polish for the plan implementation.

## Scope
- **Includes**: Dead code removal, unused import cleanup, plan verification, CLAUDE.md update, loose ends
- **Excludes**: New features or scope expansion

## Context
This is the final chunk. All implementation is complete. Focus on quality and cleanup.

## Acceptance Criteria
- [ ] No dead code from replaced implementations
- [ ] No unused imports or stale references
- [ ] Plan objectives verified working
- [ ] Application runs without errors
- [ ] CLAUDE.md updated to reflect the new reality
- [ ] All `Pre-existing Issues` items below resolved or migrated to a followup plan

## Dependencies
All other chunks completed.

## Pre-existing Issues
<Issues discovered during plan execution that are out of scope for the current chunk. Append during execution; resolve in capstone.>

---
## Session Notes
<Empty at authoring.>
```

---

## Plan lifecycle

```
1. /create-plan
   ↓ (commit: "plan: <plan-name> — create")
2. /work-plan <plan-name>          ← repeat per chunk
   ↓ (commit: "chunk: <plan-name> — <chunk description>")
3. Capstone chunk
   ↓ (commit: "chunk: <plan-name> — Capstone")
4. Post-mortem (3 parallel @verifier — quality, completeness, cleanup)
   ↓ (commit: "fix: Post-mortem fixes for <plan-name>")
5. Archive: mv .claude-plans/<plan-name> .claude-plans/archive/
   ↓ (commit: "chore: Archive completed plan <plan-name>")
```

### Versioned archive names

**Never reuse an archive name.** If the plan name `walkthrough-hardening` would collide with a previous archive, append a version suffix or descriptive qualifier:

- `walkthrough-hardening-v2-reactions-uploads-gm`
- `walkthrough-hardening-v3-phases-15-16`

Archive collisions silently overwrite history. Don't trust your memory; check `.claude-plans/archive/` before naming.

---

## Followup-plan pattern

Plans rarely produce a clean Done state. The capstone or post-mortem will surface items that don't fit "simple/obvious inline fix":

- Pre-existing issues collected during chunks
- Sweep findings from quality verifier
- Cleanup items the cleanup verifier flagged

Don't drift these into ad-hoc TODO files. Create a followup plan:

- `<original-plan>-followups`
- `<original-plan>-wave-2` (for systematic second sweeps)
- `post-plan-sweep-followups` (for cross-plan accumulated items)

The followup plan inherits the drop-with-prejudice clause for every chunk — most followup chunks ship as no-ops because the situation changed during execution. That's the point.

---

## Drop-with-prejudice culture

Every chunk in every plan may, on recon, prove already satisfied or to deliver less value than the noise of touching the file. **Workers MUST drop the chunk** and document why in Session Notes rather than push through.

Concept-application without payoff is a regression. Some chunks will ship as no-ops. That's the price of room-to-breathe scoping.

Drop the chunk by:

1. Adding a Session Notes entry explaining what was checked and why dropping is correct.
2. Marking it `[-]` in CHECKLIST.md.
3. Committing nothing for that chunk (or just the Session Notes update).

Do not try to manufacture work to justify a chunk. Trust the recon.

---

## Plan resumption discipline

When `/work-plan <name>` runs, the orchestrator must **re-read the relevant chunk file in full** before claiming readiness. Don't skim CHECKLIST.md and start. Session notes from prior sessions may have:

- Changed the chunk's premises (e.g., "while doing chunk 02, we realized chunk 04's foundation is wrong")
- Added Pre-existing Issues that affect this chunk
- Documented dropped sub-items that should not be re-attempted

The chunk file is contracted state. Read it.

---

## Anti-patterns

1. **Plan creation without a capstone.** Capstone is non-optional. Most plans need post-mortem fixes immediately after capstone — schedule that, don't treat it as failure.
2. **Two active plans at once.** Almost always means a previous plan didn't get archived. Archive it before starting another.
3. **Aspirational chunks.** "Eventually we should also tighten X" — that's a followup plan, not an out-of-scope chunk.
4. **Skipping Session Notes.** Session Notes are how the next session knows what actually happened. Skipping them is the single biggest cause of context loss across sessions.
5. **Renaming chunks mid-flight.** Once a chunk is committed, its name is in git history. Renaming the file orphans the commit reference.
6. **Capstone that defers cleanup.** Capstone IS the cleanup. If you're deferring it to "next plan," you don't have a capstone — you have a partial chunk.
