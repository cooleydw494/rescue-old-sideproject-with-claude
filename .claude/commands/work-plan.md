---
description: Resume work on an existing multi-session plan
argument-hint: <plan-name>
---

# Resuming Plan: $ARGUMENTS

Continuing a multi-session implementation plan.

## Orientation

### 1. Load Core Context
From `.claude-plans/$ARGUMENTS/`:
1. **CHECKLIST.md** — progress state
2. **OVERVIEW.md** — vision and goals

### 2. Session Context via @summarizer
> "I'm resuming work on the plan at `.claude-plans/$ARGUMENTS/`. I've read CHECKLIST.md and OVERVIEW.md. Read through all the chunk files and brief me on anything that's evolved — session notes, discoveries, deviations, or context from earlier work that might matter for what's next."

### 3. Select Next Chunk
First unchecked `[ ]` in CHECKLIST.md whose dependencies are satisfied (or the chunk the user named). Read ONLY that chunk's file.

### 4. Announce Your Focus
Chunk, goal, acceptance criteria, relevant prior context.

## Execution

If the chunk has an "Execution Strategy", **follow it**. Otherwise use judgment.

- **@summarizer** for research
- **@verifier** before non-trivial changes
- **@code-reviewer** after significant implementation

Follow the chunk's "Implementation Notes".

## Completion

### 1. Code Review (REQUIRED)
**Run @code-reviewer first.** Tell it to **skip committing** — commit everything together after updating plan files. Fix any issues.

### 2. Update Plan Files

**Chunk file** — append to "Session Notes": what was implemented, deviations, discoveries relevant to other chunks.

**CHECKLIST.md** — mark the chunk `[x]`. Add cross-cutting notes.

**Capstone chunk** — if you deferred pre-existing issues unrelated to this plan, append to "Pre-existing Issues":
```markdown
## Pre-existing Issues
Issues discovered during plan execution that should be resolved before completion:
- [ ] <description of issue and where it was observed>
```

### 3. Commit
Implementation + plan updates together: `chunk: [plan-name] - [chunk description]`. Uncommitted work is lost between sessions.

### 4. Report
Accomplishments, chunk status, commit hash, suggested next chunk, concerns.

## Plan Completion (Final Chunk Only)

When the chunk you just completed is the **last chunk** (or the capstone chunk), run the full completion sequence.

### Step 1: Post-Mortem (3 Verifiers)

Launch 3 parallel @verifier agents to answer: **"Did we do it well?"**

1. **Quality**: Code quality, test coverage, error handling across ALL plan changes
2. **Completeness**: Every acceptance criterion met, no partial implementations
3. **Cleanup**: No debug code, no TODO comments from plan work, no stale imports

Fix any issues found, then proceed.

### Step 2: Quality Sweep (4 Verifiers)

Launch 4 parallel @verifier agents to answer: **"What did we miss that no plan would catch?"**

<!-- CUSTOMIZE: Adapt these prompts to your project's architecture and technology -->

1. **Cross-page consistency**: Patterns that individually look fine but don't match across pages (headers, empty states, modals, form layouts, navigation, card structures, loading states). Look across the WHOLE app, not just files the plan touched.

2. **Stale code & drift**: Unused imports across ALL files (not just modified ones), unused CSS classes, orphaned routes/views/model methods, dead variables, scaffold files, unnecessary dependencies.

3. **UX polish & edge cases**: Accessibility gaps (aria-labels, focus management, screen reader support), text overflow/truncation with long user content, empty/null state handling, form validation display, transition consistency.

4. **Backend hygiene**: Missing authorization checks, N+1 query risks, security surface (mass assignment, file upload validation), test coverage gaps, configuration drift (do configs match what code actually produces?), documentation accuracy.

**After collecting findings:**
- **Fix immediately** anything simple and obvious (dead code, missing auth, stale docs)
- **Present to the user** anything requiring judgment (accessibility sweeps, architectural decisions, consistency choices)
- **Commit fixes** with message: `fix: Post-plan quality sweep — <brief description>`

**Note:** If the sweep surfaces significant work (e.g., accessibility across 30+ files), flag it as a follow-up rather than blocking archive.

### Step 3: Archive

1. Congratulate the user on completing the plan
2. Move the plan to archive:
   ```bash
   mv .claude-plans/<plan-name> .claude-plans/archive/<plan-name>
   git add .claude-plans/ && git commit -m "chore: Archive completed plan <plan-name>"
   ```

## If Blocked
- Document the blocker in the chunk file
- Mark `[~]` in CHECKLIST.md with a note
- Suggest a different chunk or resolving the blocker
