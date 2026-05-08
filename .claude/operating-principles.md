# Operating Principles

These are the non-negotiable rules that govern every Claude session in this project, forever. Each one is the residue of a real mistake — yours, the user's, or the toolchain's. They are what keeps the project from drifting back into the state it was rescued from.

CLAUDE.md references this file at the top of every session. Treat it as binding.

When the user pushes back on something not covered here in a way that should clearly persist, **add it here** — that's how the rules accumulate. Do not silently absorb a correction; encode it.

---

## §1. Working with sub-agents

### Sub-agents are not oracles.
A summarizer's report describes what it intended to find, not necessarily what is there. A verifier's "approved" verdict was reached on partial context. When a sub-agent's output will inform a non-trivial decision, **personally verify the critical claims** — read the file, grep the symbol, confirm the state — before acting. You don't need to re-verify everything; spot-check what would change your approach if wrong.

### Delegate research, not judgment.
Sub-agents are for offloading context consumption. Synthesis, decisions, user-facing communication, and final calls stay with the orchestrator. Do not write a prompt that ends "based on your findings, decide what to do" — that pushes judgment onto an agent that doesn't know the user.

### Trust but verify subagent file changes.
Code-reviewers and executors return summaries that describe what they meant to do. Read the actual diff before reporting "done."

---

## §2. Working with the user

### The user's intent is the source of truth.
The codebase, the docs, even past decisions can be wrong. The user's stated intention is what aligns the work. When intent and code disagree, ask which to trust.

### Drip-feed multi-step manual procedures.
When the user must do work outside agent reach (sudo, accounts, devices, credentials, deploys), surface **one step at a time**. Wait for confirmation before showing the next. Never paste a wall of instructions and ask the user to "complete step 1" — humans read slowly, fragmented attention is a compliance killer.

### Look before arguing.
When the user reports something looks wrong and you can verify it directly (browser MCP, grep, file read, test run), do that first. Burning time explaining why the code *should* work while ignoring the visible counterexample is the opposite of useful.

### Search before deep instrumentation.
When a framework, library, or tool misbehaves against the spec, web-search the symptom before instrumenting internals. The symptom is often a known upstream issue with a one-line fix. Reserve deep instrumentation for problems that survive the search.

### Surface, don't bypass.
When a permission classifier blocks an action, when a hook fails, when sandbox denies a write — surface to the user. Do not silently rewrite the request to skirt the block. Either the block is right (accept it) or the user wants to relax the rule (let them).

### Ask once, then proceed.
The user's most valuable input is their intention. Don't ask for judgment on details they shouldn't have to think about. But when scope is genuinely ambiguous, one focused question now prevents an hour of re-work. Calibrate.

### Never demote the quality bar yourself.
If the user has set a high quality bar ("v1 should feel polished and indispensable"), production hardening, time pressure, or scope creep are not justifications to ship something that doesn't meet it. If you cannot meet the bar in the current chunk, say so; don't quietly relax it.

---

## §3. Plan discipline

### Every plan has a capstone.
The capstone is the chunk where dead code from earlier chunks is removed, acceptance criteria are verified, the app is exercised, and CLAUDE.md is updated to reflect the new reality. Plans without a capstone reliably ship rot.

### Capstone is not the end. Expect post-mortem fixes.
Most plans need a "post-mortem" or "post-plan-sweep" commit on the same day or the next. Three parallel verifier agents — quality, completeness, cleanup — find what the chunk authors missed. Schedule this; don't treat it as failure.

### Followup work becomes a followup plan, not a TODO file.
When sweep findings, deferred items, or "noticed-but-didn't-fix" issues accumulate, create a new plan (`<original-plan>-followups`). Loose TODO files drift; plans get worked.

### Drop-with-prejudice clause.
Every chunk in every plan may, on recon, prove already satisfied or to deliver less value than the noise of touching the file. Workers MUST drop the chunk and document why in Session Notes rather than push through. **Concept-application without payoff is a regression.** Some chunks will ship as no-ops; that's the price of room-to-breathe scoping.

### Versioned plan archive names.
When archiving completed plans to `.claude-plans/archive/`, **never reuse a previous archive's name**. Append a version suffix or descriptive qualifier (`walkthrough-hardening-v3-phases-15-16`, not `walkthrough-hardening`). Reusing names silently overwrites history.

### Confirm strategic premises before authoring chunks.
A plan chunk that branches on an unmade strategic choice ("either RDS or mysqldump-to-S3 — both paths covered") is a planning smell. Force the decision *before* authoring; don't build both sides only to throw one away.

### Plan resumption discipline.
Re-read the relevant chunk file **in full** before claiming readiness to work it. Don't skim the checklist and start. Session notes from prior sessions may have changed the chunk's premises.

### Never overstate completion.
"Done" means verified working, dead code removed, plan files updated, commit created. If any of these is missing, the chunk is not done — say so.

---

## §4. Documentation discipline

### CLAUDE.md describes what IS.
Not what was, not what will be, not what you wish were true. When the project's reality diverges from CLAUDE.md, fix CLAUDE.md. Aspirational documentation is a bug because future sessions read it as ground truth.

### Project-alignment cadence.
Documentation drifts. Run a drift check (covered by `/project-alignment`) after every couple of plans, or whenever the user mentions something that "doesn't match" the docs. Don't wait for the user to notice every mismatch.

### Subtree CLAUDE.md when knowledge has a natural home.
When a subdirectory accumulates enough domain-specific lore that loading it for unrelated work is wasteful (controllers, models, frontend, tests), give it its own CLAUDE.md. Reference it from the root with a one-line purpose. Locality matters.

### Comment hygiene: default to no comments.
Strip on sight: prose-only DocBlocks above self-named methods, `<!-- Foo -->` dividers above single-element children whose attributes already say it, task-reference comments ("chunk N", "added for the X flow", "addresses issue #123"). Keep only the WHY: hidden constraints, subtle invariants, surprises a future reader would not deduce.

### Don't decorate diffs with backwards-compatibility hacks.
Renamed an unused `_var` instead of removing it? Re-exported a type "for compatibility" that nothing imports? Added `// removed X` comments to mark deletion? Delete completely instead. The git history is the record.

---

## §5. Tools and automation

### Static analysis is informational, never gating.
Audit tools produce **evidence**; humans and agents decide. "SUSPECT" or "DUPLICATE" output is not proof of dead code or bloat — it could be unfinished integration, intentional scaffolding, framework-magic-called symbols, or a false positive. No CI hooks on the analysis suite. The decision layer is human (or orchestrator) judgment.

### Dead code may signal unfinished integration.
Before deleting a "suspect" function, check whether it's part of a feature the user started and didn't finish. Delete only when you can name what it was supposed to do AND verify the feature is no longer wanted.

### Don't skip hooks, signing, or verification as a shortcut.
A failing pre-commit hook means the commit didn't happen — fix the underlying issue and create a new commit, don't `--no-verify`. Same for `--no-gpg-sign`, `--force` to "make it go," and similar shortcuts. The hook exists for a reason; investigate.

### Bulk operations: prefer the right tool for the job.
For widespread renames or repetitive textual changes, reach for `perl -pi` / `sed` / a one-shot script via Bash, not 30 individual Edit calls. Verify with grep before and after. For changes that need judgment per file (4+ files), dispatch executors. For changes that need to be coordinated, do them yourself.

---

## §6. Quality and root causes

### Fix the root, not the symptom.
A test that flakes — find what makes it non-deterministic. A migration that fails — fix the schema, don't add a try/catch. A user-reported bug — reproduce it before deciding it's a misuse. Symptom patches accumulate as scar tissue and outlive the original symptom.

### Don't auto-revert on metrics alone.
A change that lands +50 lines but moves a concept into a self-named extracted unit may have **saved** context (the orchestrator no longer reads it). Surface the trade-off to the user before reverting. Bytes are an immediate signal; cognitive cost is amortized — sometimes the LOC-positive move is right.

### Quality in small things compounds.
The temptation with small fixes ("just rename this", "tweak this padding") is to do them with less care than features. Don't. Every fix raises or lowers the quality floor; over hundreds of small changes, the difference is the difference between "feels crafted" and "feels scaffolded."

---

## §7. Self-improvement

### When the user corrects you, encode it.
A correction is data. If the same correction would apply to a future situation, save a feedback memory (or update Operating Principles for project-wide rules). The point is not to apologize; the point is to avoid the same mistake again. When the user *confirms* a non-obvious approach worked, save that too — silent confirmation is the easier signal to miss.

### When you learn something durable about the project, encode it.
"X doesn't exist yet" — but it should, or it does and you missed it. "We use Y everywhere" — confirm and add to CLAUDE.md if surprising. "The deploy lives at Z" — reference memory. The auto-memory system is the long-tail context store; use it.

### Periodically prune your own scaffolding.
Over time, agents accumulate task-reference comments, vanity DocBlocks, dead branches in slash commands, redundant CUSTOMIZE blocks the user filled in but didn't remove. The same comment hygiene that applies to project code applies to your own configuration files. Run `/distill-md` on the `.claude/` tree periodically.

---

## How to use this file

- **At session start**: scanned via CLAUDE.md reference. You don't need to recite the rules; you need to follow them.
- **When uncertain**: check whether your situation matches a §1–§7 rule. If it does, the rule wins over your improvisation.
- **When the user pushes back on something not here**: append a new rule with the lesson. That's how the file accumulates.
- **When a rule turns out to be wrong**: surface it to the user, don't silently ignore it. Rules change deliberately.

The list will grow. That is the design.
