---
description: "Cognitive QA walkthrough — traces user stories through code to find bugs, flow gaps, UX issues, and auth problems without a browser"
argument-hint: "[optional focus area or persona, e.g., 'character management', 'as a new user']"
---

# Walkthrough

Cognitive QA walkthrough — trace user stories through code, not through a browser. No rendering, no runtime. Systematic comprehension of what users experience at every step.

Process: map flows → generate stories → dispatch @story-tracer agents → synthesize prioritized report.

**User's input:** `$ARGUMENTS`

---

## Phase 1: Scope

Determine walkthrough scope from the user's input:

| Input | Scope | Recon Depth |
|-------|-------|-------------|
| No arguments | **Full walkthrough** — all flows, all roles | Full flow inventory |
| Feature area (e.g., "journal entries") | **Focused** — stories for that subsystem only | Targeted recon on relevant routes/pages |
| Persona (e.g., "as a new user") | **Persona-scoped** — full breadth from that perspective | Full inventory, stories filtered by persona |
| Specific concern (e.g., "permission gaps") | **Concern-scoped** — all flows, checking for that category | Full inventory, tracers briefed to prioritize the concern |

If genuinely ambiguous, ask one clarifying question — don't guess.

---

## Phase 2: Flow Mapping

Launch **1–2 @summarizer agents** to produce a compact **flow inventory** of the application (or the focused area).

### Summarizer Prompt — Full Walkthrough

> Produce a compact flow inventory of this application for a cognitive QA walkthrough. Read `routes/web.php` (or equivalent routing), navigation components, middleware, auth/policy files, and any project CLAUDE.md for stack context.
>
> I need:
>
> 1. **Entities & Roles** — Domain models and user role types with their permission differences
> 2. **Route Map** — Every user-facing route: method, URI, middleware, handler, page/view. Group by feature area.
> 3. **Navigation Structure** — How users move between sections (nav bars, tabs, links, breadcrumbs)
> 4. **Permission Boundaries** — What each role can vs. cannot do, and how that's enforced (middleware, policies, gates)
> 5. **Key State Transitions** — Entity lifecycles: create → view → edit → delete, for each major entity
>
> Format as a dense inventory, not prose. Maximize information per line. This output drives story generation — completeness matters more than explanation.

For **focused walkthroughs**, target only the relevant subsystem — skip the full route map.

Dispatch a second summarizer only if it adds value (e.g., split routing/auth vs. navigation/components).

**Wait for recon to complete before proceeding.**

### Extract Conventions

From the flow inventory and CLAUDE.md hierarchy, note conventions for tracer prompts. **Prevents false positives** — tracers unfamiliar with framework patterns misread normal behavior as bugs.

Extract:
- **Framework and view layer** — so tracers know what files to read and how data flows to views
- **Authorization pattern** — where auth checks live (middleware? controller method? both?) so tracers don't flag working auth as missing
- **Shared/global data** — props, variables, or context injected by middleware or framework (not by individual handlers). Tracers will flag "prop not provided by handler" if they don't know these exist
- **Implicit feedback patterns** — does the framework's redirect-after-POST serve as success feedback? Do form helpers provide built-in error display? Tracers need to know what the framework handles so they don't report missing UX that's actually present
- **Common patterns** — scoping, soft deletes, team/tenant context, anything that repeats across the codebase

Keep this to 4–6 bullet points. Dense, specific, factual.

---

## Phase 3: Story Generation

Generate user stories from the flow inventory. **This is your job — do not delegate it.**

### Story Taxonomy

Generate stories across these categories. Higher categories are higher priority — if you must cut stories for scope, cut from the bottom.

**1. Core Journeys** — The paths every user walks regularly
- First visit, signup, or onboarding flow
- The primary daily/weekly engagement loop
- Most common action in each feature area

**2. CRUD Cycles** — Full lifecycle of each major entity
- Create → View → Edit → Delete (where applicable)
- Focus on the *transitions* — what happens between states?

**3. Permission Boundaries** — Role-based access tested at the seams
- Elevated-role user performing privileged actions (do they have what they need?)
- Regular user encountering permission boundaries (is the boundary clear and graceful?)
- Unauthenticated user hitting auth gates (redirect? error? dead end?)

**4. Edge States** — The states developers forget
- Empty states: zero items of every listable type
- First-time states: user with no history, no content, no configuration
- Deletion aftermath: what happens to dependents when the parent is gone?
- Missing references: what if a linked entity was deleted?

**5. Cross-Feature Flows** — Stories that span feature boundaries
- Actions in feature A that should surface in feature B (feeds, notifications, dashboards)
- Shared components used in different contexts (do they behave consistently?)

**6. Destructive Flows** — Delete, leave, remove, cancel
- Is there confirmation?
- Is the outcome communicated?
- Can the user recover or undo?

### Story Format

```
STORY: [short descriptive name]
PERSONA: [who — role, experience level, data state]
STEPS:
1. [action] — [starting route or page]
2. [action] — [expected route or page]
3. [action] — [expected result]
...
FOCUS: [what the tracer should pay special attention to]
STARTING POINTS: [route paths, controller names, page component paths from flow inventory]
```

Each story: 3–7 steps. Specific enough to trace, bounded enough to finish.

### Coverage Calibration

| Scope | Target |
|-------|--------|
| Small app (< 15 routes) | 15–25 stories — aim for comprehensive coverage |
| Medium app (15–40 routes) | 20–35 stories — core flows + key edge cases |
| Large app (40+ routes) | 25–40 stories — prioritize ruthlessly, accept gaps |
| Focused walkthrough | 8–15 stories for the target area |

---

## Phase 4: Dispatch

Present the story list as a brief status update, then dispatch. Not a gate — the user can interrupt.

```markdown
## Walkthrough: [N] Stories Queued

**Core Journeys** ([n]): [story names]
**CRUD Cycles** ([n]): [story names]
**Permission Boundaries** ([n]): [story names]
**Edge States** ([n]): [story names]
**Cross-Feature** ([n]): [story names]
**Destructive** ([n]): [story names]

Dispatching now.
```

### Dispatch Strategy

Dispatch **all stories in parallel** — every story as a separate `Agent` call with `subagent_type="story-tracer"` in a single message. Synthesis handles pattern detection after all results return; no wave-by-wave processing.

If the runtime caps concurrent calls (dispatch errors), batch into groups of 10–12. No synthesis between batches.

### Tracer Prompt Template

For each story, send a prompt structured as:

> **Tech context:** [3–5 bullets: framework, routing system, view layer, auth pattern, common conventions]
>
> **Trace this story:**
>
> [paste the story block from Phase 3]
>
> Follow the trace method in your instructions. Return your findings in the output format specified — CLEAN verdict if no issues, structured findings if issues exist.

Include the Phase 2 conventions in every prompt — saves each tracer from re-reading CLAUDE.md.

---

## Phase 5: Synthesis

After all tracers complete, process the full set of reports.

### Steps

1. **Separate** — Clean stories vs. stories with findings. Count totals.
2. **Deduplicate** — Same file:line flagged by multiple stories → merge into one finding, list all story contexts.
3. **Detect patterns** — Are there systemic issues? ("8 of 14 pages have no empty state." "All delete flows lack confirmation." "Flash messages only set on error paths, never success.") Surface these as distinct systemic findings.
4. **Prioritize** — Critical findings first, then notable, then minor. Within each tier, systemic patterns outrank isolated findings.
5. **Collect blindspots** — Things tracers flagged as INFERRED, plus areas where code analysis fundamentally can't verify the experience (visual layout, animation, responsive behavior).
6. **Visual spot-check** (see below).

### Visual Verification

If browser tools are available (Chrome extension, Playwright MCP, etc.), use them to **selectively verify findings code analysis can't resolve.** Surgical confirmation, not exhaustive testing.

**Use for:** LIKELY/INFERRED findings a screenshot resolves (e.g., "missing empty state" — navigate and look); systemic patterns worth spot-checking 2–3 of; critical findings with high false-positive risk.

**How:** Navigate to the page in the state described, screenshot, upgrade confidence to VERIFIED or dismiss, attach screenshot to the report.

**Don't:** Screenshot every page (verification, not exploration); spend more than 3–6 targeted checks; treat it as a substitute for code analysis. Tracers do the heavy lifting; the browser resolves specific doubts.

**If browser tools are NOT available** and findings would clearly benefit from visual verification, note in the report:

> Some findings marked LIKELY could be confirmed or dismissed with visual verification. If browser tools (Chrome extension, Playwright) are available, I can spot-check these — let me know if you'd like to enable that.

### Report Format

```markdown
## Walkthrough Report

### Overview
- **Scope:** [full / focused on X / persona: Y]
- **Stories traced:** [N] total — [n] clean, [n] with findings
- **Findings:** [total] — [n] critical, [n] notable, [n] minor

---

### Critical Findings

FINDING: [description]
SEVERITY: critical
CONFIDENCE: [verified/likely]
STORIES: [which stories surfaced this]
EVIDENCE: [file:line] — [what's wrong]

---

[repeat for each]

### Notable Findings

[same format]

### Systemic Patterns

[If detected — these are often the most valuable findings]

PATTERN: [description — e.g., "Empty state handling absent across feature"]
AFFECTED: [list of pages/routes]
EVIDENCE: [representative file:line examples]

---

### Minor Findings

[brief — file:line and one-line description each]

### Clean Flows ✓

[List stories that traced without issues. This IS signal — it confirms what's working.]

### Blindspot Notes

[Areas where code analysis alone could not verify the experience — minus any resolved by visual spot-checks above.]
- [What couldn't be confirmed] — [why]
```

### Close

Present the report. Then offer the recommended next step:

> Walkthrough complete. The recommended next step is to create a hardening plan that addresses every finding systematically. Would you like me to proceed with `/create-plan`, or would you prefer to investigate specific findings first?

If the user says yes (or anything that isn't a redirect), proceed directly to Phase 6.

---

## Phase 6: Hardening Plan

When the user opts in, use `/create-plan` with the approach below — proven method for turning findings into hardening work.

### Plan Creation Instructions

Pass the report as context to `/create-plan` with these requirements:

> Create a plan that addresses every single finding from the walkthrough, with room to address each one well.
>
> **Chunking:** Group findings into chunks that make sense together and aren't too much work per chunk. Straightforward bug/fix findings should be grouped so verification can be done quickly and efficiently. Findings that need more consideration or involve product decisions should have room to breathe.
>
> **Per-finding process:**
> 1. **Re-examine** each finding to confirm it's valid and determine the proper course of action. If a finding requires a product decision that can't be resolved from context alone, flag it — ask the user rather than guess.
> 2. **Fix** the issue with a proper solution, not a band-aid.
> 3. **Test** — add test coverage that ensures this can never happen again. Not brittle or under-considered tests; *good* tests that genuinely protect against regression.
> 4. **Sweep** — search for any other similar issues in the codebase that the walkthrough didn't detect. This is critical: group findings in a way that avoids re-discovering the same class of issue in a later chunk. If chunk 3 fixes all missing auth checks, chunk 8 shouldn't stumble across more of them.
> 5. **Expand coverage** — fix any sweep findings and extend test coverage to account for them too.
>
> **Outcome:** By the end of this plan, absolutely everything should be addressed and the codebase should be significantly hardened in a way that genuinely improves the baseline. Start to finish, no mess left behind, nothing undone. Everything based in grounded facts.

The plan should include a **capstone chunk** at the end for final verification, cleanup, and dead code removal.

---

## Adaptation

**Speed up** when: Focused scope, few routes, stories tracing cleanly.
**Slow down** when: Cross-feature interactions are tangled, permission model is complex — generate more stories, not fewer.
**Abbreviate Phase 2** when: You already have deep context on this codebase from the current session. Still extract conventions for tracer prompts, but skip the full summarizer dispatch if you can produce the flow inventory yourself.

---

## Principles

- **You are the orchestrator.** Generate stories, dispatch tracers, synthesize findings. Don't trace stories yourself — you'll burn context and do a worse job than a focused sub-agent.
- **Context is your budget.** Clean verdicts ~30 tokens, issue reports ~60–100. 25 stories dispatched at once ≈ <2K tracer output. One dispatch, one synthesis.
- **Evidence discipline flows upward.** Tracers cite file:line; synthesis preserves citations. The user needs issues in actual code, not your paraphrase.
- **The value is systematic coverage.** A 25-story walkthrough beats manual review on consistency, not intelligence — every flow checked, every state considered, every role tested.
- **One session, one report.** Single conversation. No plan files, no checkpoints. Map → Generate → Trace → Report.
- **Diagnosis leads to action.** Natural conclusion is a hardening plan. Full workflow: walkthrough → report → plan → execute → capstone verification.
