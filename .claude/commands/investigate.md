---
description: Deep investigation of any problem, question, or opportunity with structured reconnaissance and user-guided resolution. Open-ended exploration that may or may not lead to implementation.
argument-hint: <problem or question>
---

# Investigate

Structured investigation of a problem, question, or opportunity. The rails are firm; the direction is yours.

## Philosophy

Problem-solving, not process-following. Ask the right questions, gather evidence from multiple angles in parallel, distinguish knowns from assumptions, surface root causes and non-obvious connections, and present options with honest tradeoffs. Don't jump to implementation, over- or under-research, or fake certainty about unknowns.

---

## Phase 1: Problem Framing

**The user's prompt:** `$ARGUMENTS`

Classify:
- **Type**: Bug, Question, Opportunity, Decision, or Unknown
- **Scope**: Narrow (specific), Medium (subsystem), Broad (architectural), or Unknown
- **Desired outcome**: Fix, understanding, options, plan, or unclear

If ambiguous, use `AskUserQuestion` to clarify before launching recon.

---

## Phase 2: Reconnaissance Design

| Type | Angles to Consider |
|------|-------------------|
| Bug | Code flow, state at failure, similar patterns, recent changes |
| Question | Direct answer, codebase examples, design rationale |
| Opportunity | Current state, alternatives, constraints, ripple effects |
| Decision | Each option's implications, criteria for choosing |

Design 2–5 @summarizer prompts, each on a distinct angle, requesting concrete evidence (file:line).

Depth keywords: "quick" / "standard" (default) / "comprehensive".

---

## Phase 3: Parallel Reconnaissance

Launch summarizers **in parallel** — multiple `Agent` calls with `subagent_type="summarizer"` in a single message. Wait for all before proceeding. 2 for narrow, up to 5 for broad.

---

## Phase 4: Synthesis

1. **Map findings** from each angle
2. **Find connections** — reinforce or contradict?
3. **Identify gaps** — what's unanswered?
4. **Challenge assumptions** — anything surprising?
5. **Assess confidence** — certain vs. uncertain?

| Finding | Next Step |
|---------|-----------|
| Clear answer/solution | → Phase 5 (present findings) |
| Need deeper analysis | → Targeted @summarizer or @verifier, then Phase 5 |
| Multiple viable options | → Phase 5 (present options) |
| No problem found / user error | → Phase 5 (explain, no action) |
| Root cause elsewhere | → Reframe investigation |

---

## Phase 5: Present Findings for User Approval

**STOP HERE.** Do NOT implement until user explicitly approves.

```markdown
## Investigation: [Title]

### Summary
[Direct answer or diagnosis — 1-3 sentences]

### Key Findings
- [Finding with evidence]
- [Finding with evidence]

### Relevant Code
- `file.ext:123` — [Why it matters]

### Proposed Action (if applicable)
| Change | Files | Risk |
|--------|-------|------|
| [What] | X files | Low/Med/High |

### Alternatives (if applicable)
[Other approaches considered]

---
**Ready to proceed?** / **Which option?** / **Questions?**
```

Adapt: skip sections that don't apply; add Option A/B with tradeoffs for decisions.

Use `AskUserQuestion` for explicit direction: **Proceed / Modify / More info / Defer / Abort**.

**Scope:** Small → Phase 6. Medium/Large → offer `/create-plan`.

---

## Phase 6: Verification (If Implementing)

Verify with **@verifier**:

> "I'm about to [describe changes]. Verify:
> - Will this break existing functionality?
> - Does this respect existing patterns?
> - What's the actual risk?"

Skip for trivial changes or pure documentation.

---

## Phase 7: Implementation

Work incrementally — one logical change at a time. Preserve behavior unless explicitly changing it; test as you go.

---

## Phase 8: Review & Commit

Delegate to **@code-reviewer**:

> "Review changes from this investigation. Verify changes match intent, no regressions, quality maintained."

Commit with appropriate type (`fix:`, `feat:`, `refactor:`, `docs:`).

---

## Phase 9: Report

```markdown
## Investigation Complete

### What We Investigated
[Brief problem statement]

### What We Found
[Key findings — 2-3 sentences]

### What We Did
[Action taken, files changed, or "No action — information gathered"]

### Follow-up
[Remaining questions or next steps, if any]
```

---

## Adaptation

**Speed up** when problem is simple, findings conclusive, scope small.
**Slow down** when findings contradictory, scope larger than expected, broad impact.
**Skip phases** that don't apply (e.g., no implementation needed).

---

## Remember

- **Reread `$ARGUMENTS`** to stay on target
- **Delegate aggressively** — your context is for synthesis and decisions
- **User approval before action** — always
- **Done beats perfect**
