---
description: Full project health check — documentation drift, code quality, roadmap alignment, and next-step recommendation
---

# Project Alignment Check

Comprehensive project health check. NOT scoped to any single plan or feature — verifying the entire project is in a good, consistent, actionable state.

## Phase 1: Reconnaissance

Launch **3 parallel @summarizer agents**:

### Agent 1: Documentation & Config Audit
> "Comprehensive audit of project documentation and configuration health. Read the project's primary documentation files (README, CLAUDE.md, roadmap, changelog, etc.) and config files. Check for:
> - Stale references to completed/removed features
> - Documentation sections that list models, controllers, components, or routes that may be missing recent additions
> - Config files referencing missing env vars or stale defaults
> - .env.example missing vars that .env has (if applicable)
> - Test count or coverage claims that may be outdated (run tests and compare)
> - Tech stack section accuracy (new packages, removed dependencies)
> Report each discrepancy with file path and line number."

### Agent 2: Code Quality & Consistency Sweep
<!-- CUSTOMIZE: Replace debug functions and style patterns with your project's equivalents -->
> "Quick code quality sweep across the entire codebase. Check for:
> - Debug statements left in production code (console.log, debugger, print statements, dd(), dump(), ray(), etc.)
> - `TODO`, `FIXME`, `HACK` comments (report with context — some may be intentional)
> - Unused imports across recently modified files
> - Style/convention violations specific to the project's stated standards (check CLAUDE.md or style guides)
> - External service dependencies that may have been replaced
> - Routes or endpoints that reference missing handlers
> Report only real issues."

### Agent 3: Roadmap & Plan State
> "Read any roadmap or planning documents fully. Then check for active plans (e.g., `.claude-plans/` or similar). For each:
> - Is the plan fully complete? Should it be archived?
> - Are there partially complete plans that need attention?
> Verify the 'current status' of the project matches reality.
> Also read any project memories for context on recent decisions.
> Finally, identify: What work is NEXT according to the roadmap? Summarize what it entails."

## Phase 2: Synthesis

After all 3 agents return, synthesize their findings:

### Issue Categories
1. **Must Fix** — Broken, incorrect, or misleading (will cause problems)
2. **Should Fix** — Stale, inconsistent, or drifted (creates confusion)
3. **Noted** — Minor observations, not blocking

### For each issue:
- File path and line number
- What's wrong
- What it should be

## Phase 3: Fix

Fix all **Must Fix** and **Should Fix** issues directly — they're documentation/config corrections, safe to apply immediately.

After fixes, run the project's test suite and build process to confirm nothing broke.

## Phase 4: Commit

If anything changed, commit with:
```
chore: Project alignment — [brief summary of what was fixed]
```

Archive any completed plans not yet moved:
```bash
mv .claude-plans/<plan-name> .claude-plans/archive/<plan-name>
git add .claude-plans/ && git commit -m "chore: Archive completed plan <plan-name>"
```

## Phase 5: Punchlist Scan

After commit, launch a **@summarizer agent** to mine for quick-win items directly actionable right now:

> "Thorough scan for immediately actionable quick wins scattered across the project. Read ALL of these sources:
>
> - Any notes files, issue trackers, or backlog documents in the repo — items that could be knocked out in a focused session
> - Ideas/feature notes that are nearly complete or partially implemented (look for strikethrough/fixed markers showing partial completion with remaining work)
> - Roadmap or planning docs — any 'done except...' items or small gaps in completed phases
> - All TODO/FIXME/HACK comments in source code (grep for them) — which ones are actually small and actionable vs. large feature work?
> - Project memories mentioning deferred items, known gaps, or 'not yet' items
> - Code comments mentioning things like 'temporary', 'placeholder', 'workaround', 'should be', 'eventually'
> - Archived notes or completed plans — skim for items marked done that might have loose ends
>
> For each item found, evaluate against these criteria (all three required):
> 1. **Directly actionable now** — no design decisions needed, no external dependencies, no ambiguity about what 'done' looks like
> 2. **Non-disruptive** — won't conflict with or derail whatever the roadmap says is next
> 3. **Cleans something up** — fixes documentation drift, removes dead code, resolves a known inconsistency, or polishes a rough edge
>
> Bonus (not required): produces a noticeable improvement for users.
>
> Return a ranked punchlist of up to 8 items. For each item:
> - **What**: one-line description
> - **Where**: file path(s) and line numbers
> - **Source**: where you found it (which notes file, comment, memory, etc.)
> - **Effort**: trivial / small / medium
> - **User-visible?**: yes/no — would a user notice the improvement?
> - **Why now**: why this is a good candidate for immediate action
>
> If fewer than 3 items qualify, say so — don't pad the list. Quality over quantity."

## Phase 6: Next Step Recommendation

Based on roadmap, memories, current state, AND the Phase 5 punchlist, present **up to three options**. Only include options that genuinely apply.

### Option A: New Plan (from roadmap)

Draft a **specific, actionable prompt** for `/create-plan` describing the next major piece of work:

- **1–2 paragraphs max** — enough for a fresh agent to understand scope
- **Grounded in the roadmap** — reference the specific phase/section
- **Context-aware** — relevant constraints, dependencies, or prior decisions from memory
- **Ready to hand off** — copy-pasteable into `/create-plan`

### Option B: Tight Punchlist (from Phase 5)

If the punchlist scan returned 3+ qualifying items, present as a focused session:
- Item count + estimated total effort
- Which (if any) are user-visible
- Frame as cleanup, not feature work — momentum and hygiene between larger efforts

### Option C: Something Else

Only if you genuinely believe it's the best path. Examples:

- **QA walkthrough (`/walkthrough`)** — recon found no major drift but significant code has shipped since the last QA pass, or the next phase touches critical flows.
- **Strategic pivot** — the roadmap's next step no longer makes sense (dependency shifted, priorities changed, systemic issue surfaced).
- **Tech debt reckoning** — recon surfaced a pattern of related issues that warrants a focused session.
- **Documentation overhaul** — multiple docs are significantly stale or contradictory in ways that will compound.
- **Investigation (`/investigate`)** — recon raised questions that need answers before committing to direction.

If nothing qualifies, omit Option C. If no roadmap exists, flag this and suggest creating one first.

Format:

```
## What's Next?

### Option A: [Phase/Area name]
[The prompt text, ready to paste into /create-plan]

### Option B: Quick Wins Punchlist
[Summary of punchlist items and why they're worth a session]

### Option C: [Alternative direction, if applicable]
[Reasoning and suggested approach]
```

## Remember

- Works at ANY point in the project lifecycle, not just after plan completion.
- Goal is alignment and clarity, not perfection — fix real drift, skip style preferences.
- Punchlist scan runs late intentionally — by then you have full context to judge what's actually actionable.
- Present options honestly — if the punchlist is thin, say so; if the roadmap's next step is clearly right, make that obvious.
