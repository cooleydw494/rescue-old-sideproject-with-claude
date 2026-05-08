---
description: Create a multi-session implementation plan from requirements
argument-hint: <requirements or feature description>
---

# Multi-Session Plan Creation

Create a structured plan for requirements too large for one session.

## Your Process

### 1. Understand Requirements
Analyze provided requirements. If none, ask.

### 2. Research the Codebase
Use @summarizer with **comprehensive depth**:
- What exists that relates to these requirements?
- What patterns should new work follow?
- What dependencies or constraints exist?

Request thorough output: *"For planning purposes, do a comprehensive analysis of..."* — planning needs the "why" behind existing patterns; don't let research get over-compressed. Delegate aggressively to preserve context for synthesis.

### 3. Break Into Chunks
Decompose into **session-sized chunks**. A good chunk:
- Achievable in one focused session (1-3 hours)
- Clear, testable completion criteria
- Minimizes cross-chunk dependencies
- Standalone value when possible
- Well-defined scope (in AND out)

Number in suggested order; note dependencies explicitly.

**MANDATORY: Every plan must end with a Capstone chunk** (see step 4 for template).

### 4. Create the Plan Structure

Create directory: `.claude-plans/<plan-name>/`

**OVERVIEW.md** — The big picture:
```markdown
# <Plan Name>

## Goal
<What we're trying to accomplish overall>

## Context
<Important background, constraints, technical considerations>

## Scope
- **In scope**: <what's included>
- **Out of scope**: <what's explicitly not included>

## Chunks
<Brief list of all chunks with one-line descriptions>
```

**CHECKLIST.md** — Progress tracking:
```markdown
# <Plan Name> - Progress

## Status
- [ ] 01 - <chunk name>
- [ ] 02 - <chunk name>
- [ ] ...

## Notes
<Cross-cutting concerns, discoveries, blockers>
```

**Individual chunk files** (`01-chunk-name.md`, `02-chunk-name.md`, etc.):
```markdown
# <Chunk Title>

## Goal
<Specific objective for this chunk>

## Scope
- **Includes**: <specific items>
- **Excludes**: <what to leave for other chunks>

## Context
<What the implementer needs to know>

## Acceptance Criteria
- [ ] <How we know it's done>
- [ ] <Testable criteria>

## Dependencies
<Other chunks that must be completed first, or "None">

## Implementation Notes
<Technical guidance, suggested approach, files likely involved>

## Execution Strategy
<Recommend the best approach>

**Parallel workers** (RECOMMEND when: 4+ files with similar mechanical changes):
- Split into parallel agents for independent file groups
- Preserves orchestrator context for coordination and review

**Direct** (RECOMMEND when: <4 files, nuanced judgment, or exploration needed):
- Orchestrator reads files, makes changes, reviews own work

Always include this section. Be explicit about which approach to use and why.

---
## Session Notes
<Leave empty - implementer fills this in during work>
```

**Capstone chunk** (`XX-capstone.md`):
```markdown
# Capstone

## Goal
Final verification, cleanup, and polish for the plan implementation.

## Scope
- **Includes**: Dead code removal, unused import cleanup, plan verification, loose ends
- **Excludes**: New features or scope expansion

## Context
This is the final chunk. All implementation is complete. Focus on quality and cleanup.

## Acceptance Criteria
- [ ] No dead code from replaced implementations
- [ ] No unused imports or stale references
- [ ] Plan objectives verified working
- [ ] Application runs without errors

## Dependencies
All other chunks completed.

## Implementation Notes
Run through the app manually, remove any scaffolding or temporary code, verify the plan's acceptance criteria are met.

## Pre-existing Issues
Issues discovered during plan execution that should be resolved before completion:
<Leave empty - workers append issues here during execution>

## Reminder
After this capstone completes, the work-plan process will run the Post-Plan Quality Sweep —
a codebase-wide audit that looks beyond plan scope for issues the capstone wouldn't catch.
```

### 5. Verify → Minesweep → Defend the Plan

**Mandatory.** Plans go through a triple-agent pipeline before being ready.

#### Pass 1: @verifier (correctness & feasibility)
- Chunks scoped with no gaps or overlaps?
- Dependencies correctly identified?
- Plan respects project architectural patterns?
- Implementation approaches will actually work?
- Factually wrong values, mappings, or assumptions?

#### Pass 2: @minesweeper (ripple effects)
Launch **in parallel** with the verifier. Sweep for files referencing changed things, stale docs, affected tests, config impacts.

#### Pass 3: Consolidate & @defender (adversarial filter)
Consolidate findings, send to @defender. Only findings that survive get acted on.

#### Pass 4: Fix confirmed issues
Apply fixes for confirmed CRITICAL and IMPORTANT findings.

#### Async: @deep-recall (institutional knowledge audit)
Launch **in parallel with Passes 1–3** (background OK). Search for prior decisions, rejected approaches, and lessons relating to this plan's domain:
- Git history for relevant commits, reverts, design discussions
- Archived plans for session notes and post-mortems in the same area
- Memory files for prior architectural decisions or user preferences
- Notes files, idea backlogs, deferred-work trackers
- Evidence contradicting the plan's assumptions

Report contradictions, tensions, or relevant prior context. Findings before finalization → incorporate; after → flag for user review before execution.

**Note:** Scales with project maturity — rich records (archived plans, descriptive git history, memory files) get more value; thin history may return little.

### 6. Report Back
Summarize: plan name/location, chunk count, verification results, concerns, suggested starting point.
