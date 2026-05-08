---
description: "Judgment-led bloat audit. Cold-read the codebase, name what feels off, prosecute each finding adversarially against the strongest case for the status quo. Zero findings is a valid result."
argument-hint: "[optional focus, e.g., 'controllers', 'the feed system', 'the genre services feel fragmented']"
---

# Distill

A senior dev opens the codebase cold. Some shape is wrong — drifted intent, unearned abstractions, missing abstractions, vestigial wiring, framework power reimplemented, subsystems carrying weight they don't need. Find what's real, prove each finding against the strongest case for the status quo, merge what survives, harvest the obvious follow-ups.

**This is a judgment tool, not a checklist.** The codebase warrants what it warrants — could be zero findings, could be eight. Quotas pad runs with noise; absence of findings is itself a valuable result.

**User input:** `$ARGUMENTS`

---

## Phase 0: Scope

<!-- PROJECT CUSTOMIZATION:
Replace the in/out lists with your project's source dirs and exclusions.
Out-of-scope should at minimum cover vendored deps, build artifacts, generated migrations,
docs (that's /project-alignment's territory), and the .claude/ tree.
-->
**In scope:** `<src>/`, `<frontend>/`, `<config>/`, `<routes>/`, `<tests>/`.
**Out of scope:** `<migrations>/`, `vendor/` or equivalent, `node_modules/`, `<build dir>/`, `docs/`, `.claude/`, `.claude-plans/`.

`$ARGUMENTS` interpretation:
- Empty → full codebase.
- Layer/feature/lens word ("controllers", "the feed system", "verbose CRUD") → narrow Phase 1's reading and lens emphasis.
- Hypothesis-shaped ("I think the nudge system got over-engineered") → that becomes a mandatory finding; orchestrator may surface others.

---

## Phase 1: Cold-read, find what's real

Hypothesis quality needs architectural context only the orchestrator holds. Don't delegate this phase.

### How to read

1. **Read CLAUDE.md** (root + nested). Note conventions, named exemplars, areas explicitly under construction (don't flag in-progress work).
2. **`git log --oneline -30`** over product code. Recent churn, follow-up sequences, fix-after-refactor patterns mark awkward shapes.
3. **Cold-read 8–12 load-bearing files** across layers — controllers, services, components, models, stores, composables, route files, key configs. Pick by judgment: high inbound reference count, oldest-touched, spec-critical, frustration-touched, subsystems the user has named.
4. <!-- PROJECT CUSTOMIZATION:
   Replace with your project's static-analysis orchestrator path, if any.
   If your project has none, skip this step entirely — see `guides/distill-audit-tooling.md`.
   -->
   **Run `<your static-analysis orchestrator>`** as sanity check (background it: `<orchestrator> > .claude-distill-scratch/sa.txt 2>&1 &`). Read it *after* the cold pass so structural heat doesn't bias the eye toward mechanical findings.

For each cold read, jot a 2–3 line note in `.claude-distill-scratch/cold-read.md`: what the file does, what feels off (or "clean"), one-line hypothesis seed if real.

### Lens table — thinking aid, not a quota

Walk these as questions to ask. Don't force coverage; an empty lens is fine.

| Lens | Question to ask |
|------|----------------|
| **Wrong/drifted abstraction** | Does this helper/component/service earn its weight? Has its name drifted from what it does? Is a missing abstraction causing spread across N sites? |
| **Subsystem over-engineering** | Could this entire feature be 30–50% smaller without losing capability? Extra layers, premature flexibility, defensive scaffolding, vestigial knobs? |
| **Convention drift / convergence** | A load-bearing pattern violated across 3+ sites where converging clarifies (not just compresses). |
| **Framework power missed** | The framework provides this. We reimplemented it (or wrap it three times). |
| **Vestigial wiring** | Config, middleware, traits, defensive guards, env flags, composables, services that no longer do real work. |
| **Data shape churn** | Data reshaped 3+ times before reaching the consumer. A typed shape, accessor, computed, or store-derived state would compress the path. |

### What "real" means

A finding worth prosecuting has all of:
- **Material upside** — confirming it would meaningfully improve readability, correctness, or maintainability. Not bytes for bytes' sake.
- **Concrete claim** — names files, names the wrong-thing's shape, sketches a specific alternative.
- **Falsifiable** — a verifier can prosecute it and either confirm or kill it cleanly.
- **Big enough to matter** — affects a real subsystem, a recurring pattern across ≥3 sites, or load-bearing architecture.

Not real: "this 8-line method could be 6 lines," "this one fallback prop is unused," "this comment is stale."

### Hypothesis format

Write each finding to `.claude-distill-scratch/hypotheses.md`, one block per finding (≤150 words):

```
H<n>
LENS: wrong-abstraction | subsystem-over-engineering | convergence | framework-power | vestigial | data-shape
SCOPE: mechanical | judgment
CLAIM: <one-line>
CITES: <files + line ranges>
ALTERNATIVE: <prose sketch — must show it handles every case the original handles>
```

`SCOPE: mechanical` = pure extraction/consolidation/deletion where tests-green proves correctness. Tier A eligible.
`SCOPE: judgment` = wrong-shape refactors where tests-green ≠ readability-improved. Tier B sketch only.

**If zero findings:** that is the report. Skip Phases 2–3, jump to the Zero-findings close line. Reading the codebase and finding nothing wrong is a successful run, not a failed one.

---

## Phase 2: Asymmetric verification

Dispatch one `@distill-verifier` per finding, all in parallel.

### Dispatch prompt (per hypothesis)

> **Project conventions** (4–6 bullets pulled from CLAUDE.md / nested CLAUDE.md):
> - …
>
> **Hypothesis:**
>
> ```
> [the H<n> block, unannotated]
> ```
>
> Prosecute per your agent instructions. Default verdict is REJECTED. Flip to CONFIRMED only when the status-quo steelman clearly fails. Return the compact structured verdict — no prose essays.

Do not pass other hypotheses. Do not pass full CLAUDE.md.

### Verdict block (what verifiers return)

```
HYPOTHESIS: H<n>
VERDICT: REJECTED | PARTIAL | CONFIRMED
TIER: A (worktree, tests green) | B (sketch) | C (sketch, multi-session) | n/a (rejected)
WORKTREE: <path if Tier A, else n/a>
DELTA: <±LoC / ±bytes — measured for Tier A, estimated for Tier B, n/a for rejected>
READABILITY: +1 | 0 | -1
AMORTIZATION: none | weak | strong
STATUS_QUO_ARGUMENT: <one line — strongest reason the current code is correct>
WHY_VERDICT: <one line>
RISK: <one line — test coverage, blast radius>
SKETCH: <path to .claude-distill-scratch/verdicts/H<n>.md, or n/a>
```

`READABILITY: +1` + `AMORTIZATION: strong` is CONFIRMED even at byte-flat or byte-positive deltas — see Principle #3.

---

## Phase 3: Apply, sweep, report

Linear. Capture `PRE_DISTILL_HEAD=$(git rev-parse HEAD)` before any apply. Do not emit the user-facing report until **step 8**.

If Phase 2 returns zero Tier A drafts — or if all Tier A drafts get dropped during steps 1–3 — skip to step 7 cleanup, then step 8 with the no-commits-landed close line.

1. **Pre-finalization review** — dispatch `@code-reviewer` once per Tier A draft, in parallel. Brief each with hypothesis claim, draft path, verifier's `DELTA` + `READABILITY` + `AMORTIZATION` + `RISK`. Pass → keep as Tier A. Flag → demote to Tier B with the flag as risk note. Do not ask the reviewer to commit. Skip if zero Tier A drafts.

2. **Apply Tier A diffs** — for each Tier A draft, export the worktree's uncommitted edits and apply in main:
   ```bash
   git -C <worktree> diff HEAD -- . > <patch>
   git apply <patch>
   ```
   If two drafts touch the same file, apply sequentially and reconcile.

3. <!-- PROJECT CUSTOMIZATION:
   Replace with your project's test commands. Both backend and frontend, where applicable.
   -->
   **Run tests** — `<your test commands>` — green is the gate. Failure → fix or drop the offending draft. Never commit on red.

4. **Pre-commit review** — stage all intended changes. Dispatch `@code-reviewer` over the staged diff with one paragraph summarizing applied hypotheses. Focus on what tests don't catch: subtle behavior shifts, lost error paths, naming regressions, hand-merge conflicts.

5. **Commit per coherent change** — default to one commit per hypothesis (each has its own "why"). Bundle only when tightly coupled (same file, same rationale).

6. **Minesweeper sweep (harvest follow-ups)** — dispatch `@minesweeper`:

   > Sweep for refactor candidates structurally identical to those just landed.
   >
   > **Confirmed patterns:** [pattern + cited files + reference commit SHA per Tier A/B finding].
   > **Kill-shots:** [each rejection's `STATUS_QUO_ARGUMENT` verbatim]. Anything that would fail these is excluded.
   > **Scope:** layer just touched plus one level of adjacency.
   > **Output:** APPLY-NOW candidates only — file + line range, one-line patch shape (referencing the just-landed commit as template). If you can't describe the patch without hedging, drop it.
   >
   > Do NOT propose new hypotheses. Do NOT modify files. An empty list is fine.

   Skip if zero confirmed findings. ≤3 candidates → orchestrator applies directly. ≥4 → dispatch `@executor` with patch shape + cited sites. For each: re-read end-to-end before patching (divergent contract = drop). Run tests, dispatch `@code-reviewer`, commit:
   ```
   refactor: apply <pattern> to N more sites (post-H<n> minesweeper)
   ```

7. **Cleanup** — for each Tier A worktree, `git worktree remove <path>`. If `rm` fails with "Operation not permitted" on `.claude/agents/*.md` or `.claude/commands/*.md`, retry with `dangerouslyDisableSandbox: true` — sandbox escape, not a user-facing question. Then `rm -rf .claude-distill-scratch`. Verify: `git status` clean, `git worktree list` shows only main, scratch dir gone.

8. **Close — tiered report**

   Compute `TOTAL_LINES_SAVED` and `TOTAL_BYTES_SAVED` as the sum of `DELTA` across CONFIRMED + PARTIAL findings (Tier A merged + Tier B sketches). Bytes correlates with agent token cost on every future visit; LoC is the human-readable companion.

   ```markdown
   ## Distill Report

   ### Overview
   - **Scope:** [full / focused on X]
   - **Hypotheses:** [N] across [L] lenses
   - **Outcome:** [a] Tier A merged, [b] Tier B confirmed, [c] Tier C structural, [d] rejected

   ### Tier A — Merged
   Per entry: claim, commit SHA, verified ±LoC / ±bytes, readability, risk note. If `AMORTIZATION: strong` (especially when bytes are wrong-direction), add one line naming the call-site contract that's now self-documenting. Tag follow-on commits as `(follow-on from H<n>)`.

   ### Tier B — Confirmed, needs execution
   Per entry: claim, cited files, sketch path, estimated ±LoC / ±bytes, amortization signal.

   ### Tier C — Structural
   Too large for one session — recommend `/create-plan`. Include drafted direction.

   ### Tier D — Rejected
   Per entry: claim + one-line kill-shot.

   ### Areas read clean
   One line per area cold-read where nothing surfaced. Reported with the same confidence as findings — knowing the codebase is clean is worth as much as cleaning it.
   ```

   Then the close line:
   - **Commits landed:** `Distill complete. **[TOTAL_LINES_SAVED] lines / [TOTAL_BYTES_SAVED] bytes saved** across [N] hypothesis commits + [F] follow-on commits ([M] confirmed, [K] rejected). HEAD <sha>, actual net delta **[±X] LoC / [±Y] bytes**.`
   - **No commits landed:** `Distill complete. **[TOTAL_LINES_SAVED] lines / [TOTAL_BYTES_SAVED] bytes saved** across 0 commits, [M] confirmed, [K] rejected. HEAD remains <sha>.`
   - **Zero findings:** `Distill complete. Codebase clean against the lenses scanned. Read [N] files cold across [L] layers; nothing earned a hypothesis. HEAD remains <sha>.`

   Nothing follows this line.

---

## Phase 4: Optional plan

Only if user asks. Use `/create-plan`:

> Create a plan addressing Tier B and Tier C distill findings. For Tier B, each chunk is a single-session execution. For Tier C, decompose into chunks that preserve correctness incrementally.

---

## Principles

1. **Asymmetric prosecution.** Default REJECTED. The refactor must survive the strongest case *for the status quo*, not a symmetric steelman. Burden is on the change.

2. **Tier A is mechanical only.** Worktree + tests-green proves extraction/consolidation/deletion. It overpromises on wrong-abstraction refactors where tests-green ≠ readability-improved. Don't promise what the scope can't deliver.

3. **Bytes are immediate; context cost is amortized.** A byte-positive refactor can still net negative on expected agent token cost when the call site becomes one declarative element naming the concept and the wrapper's bytes are paid once. Conditions for `strong` amortization (all three): (a) the abstraction has a name conveying what it does without descending, (b) the hidden machinery is rarely-debugged-at-call-site (ARIA, modal state, framework boilerplate, defensive parsing), (c) call sites are visited more often than the wrapper. When all three hold and `READABILITY: +1`, that's CONFIRMED — surface the trade-off and let the user revert if they disagree.

4. **Cold reading is the primary engine.** Static analysis sees only what it knows how to see — dead symbols, duplication, oversized files. It does not see wrong abstraction, drifted intent, unearned helper weight, or vestigial wiring. Read first; audit second.

5. **Zero is a valid result.** A run that surfaces no real findings is a successful run. Report what's clean with the same confidence as what's broken. Don't pad to fill a quota; don't stretch a borderline observation to feel productive.

6. **Harvest within the run.** Structurally-identical sites to a just-confirmed pattern are this run's work, not next run's lead. The reviewer, the tests, and the context are warm — use them.

7. **One session, one report.** Phases 1 → 2 → 3 in a single conversation. Phase 4 plan can span sessions.
