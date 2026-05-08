---
description: "Tighten one or two md spec/config files (slash commands, sub-agents, skills, CLAUDE.md) — cuts verbosity and structural bloat while preserving every rule, template, enum, and anchor. Bytes-first metric."
argument-hint: "<path/to/file.md> [second/file.md]"
---

# /distill-md

You are tightening one — or at most two — markdown configuration files. Targets are slash commands, sub-agent definitions, skill files, CLAUDE.md, or similar Claude-Code config. `/distill` is for code; this is for the docs that govern how agents behave.

**The bloat is cross-sectional.** Meta-commentary, rationale paragraphs after self-evident rules, table↔prose duplication, multi-paragraph intros that say one thing thrice, sub-phase numbering inflation. None of it is visible chunk-by-chunk — only holistically. The chunked-sub-agent reflex caps savings at ~5%. Read end-to-end yourself.

**Bytes is the primary metric.** Long lines collapse content without removing lines; ten blank lines are ten LoC but few bytes. Track both, headline bytes.

**The user's input:** `$ARGUMENTS`

---

## Phase 0 — Target & baseline

Resolve the path(s). If none provided, ask which file.

For each target: `wc -lc <file>`, record baseline. Read it end-to-end personally. Do not delegate the read; the redundancy you're hunting is only visible to one reader holding the whole file at once.

If two files: full workflow on each, sequentially. Don't interleave.

## Phase 1 — Preservation map

Before cutting anything, list what must survive verbatim (or near-verbatim). Hold it in context — no scratch file needed.

| Category | What | Why |
|---|---|---|
| **Frontmatter** | `name:`, `description:`, `tools:`, `model:`, `argument-hint:` | For sub-agent files the `description:` is the **Task-tool routing API** — do not shorten. For commands it controls `/` autocomplete. |
| **Literal templates** | Schemas, output contracts, dispatch `>` blocks meant to be pasted, ledger formats, report templates | Agents read these verbatim; whitespace and field order matter. |
| **Enums** | Status values, fixed-member lists referenced from prose | Trimming a value silently changes behavior. |
| **Conditionals** | Skip-rules, override clauses, "if X then Y" logic | Easy to lose in a paragraph compression. |
| **Section anchors** | `### 1a.3`, `## Phase 2`, etc. | External refs may point at them. Keep the IDs and ordering. |
| **Project-customization HTML comments** | `<!-- PROJECT CUSTOMIZATION ... -->` | Survive across copy/paste between projects; never drop. |

If something looks borderline (load-bearing or not?), confirm with the user before deciding.

## Phase 2 — Bloat hypotheses

What you're allowed to cut, in priority order:

- **Meta-commentary** — sentences whose only job is justifying the previous sentence ("this closes the learning loop", "the rule accepts that…")
- **Rationale paragraphs after self-evident rules** — when the rule alone is enough, the why-paragraph is bloat
- **Table↔prose duplication** — same content in a table cell *and* the surrounding paragraph; pick one location
- **Multi-paragraph phase/section intros** restating the same thing two or three ways
- **Verbose connective tissue** — "Before drafting the final hypothesis set…", "After all sub-agents return…"
- **Phase-number inflation** — Phases 3 / 3.5 / 3.6 / 3.65 that should be one phase with numbered steps
- **Restated CLAUDE.md context** — agents re-explaining the stack when CLAUDE.md auto-loads

What you must **not** cut:

- Anything in the Preservation Map
- Content you disagree with — you're tightening verbosity, not editing for content
- Section anchor IDs (rename = silent rot in callers)
- Project-customization HTML comments

## Phase 3 — Pilot pass

Pick the densest, most rationale-heavy section in the file. Rewrite it. Show the user the diff for that section only and get explicit sign-off on the aggressiveness level.

This is the calibration step. Over-timid and over-aggressive both get caught here, before propagating to the rest.

## Phase 4 — Apply stance to remainder

Single pass over the rest of the file with the calibrated stance. Use `Edit` for surgical cuts, `Write` only if the rewrite is structural across many sections.

**Do not** chunk-and-dispatch. The whole reason this command exists.

## Phase 5 — Self-verification

Re-read the file end-to-end. Classify every cut against the original:

- **Meta-commentary removed** — sentence justified the prior rule; rule remains intact
- **Rationale compressed** — multi-sentence why → one sentence; mechanism preserved
- **Redundancy dedup'd** — same content existed twice; one location chosen
- **Connective tightened** — verbose transition → terser; flow preserved

If a cut doesn't fit any of these four, re-examine — you may have removed a rule.

`wc -lc <file>` again. Report:
- **Original:** X lines / Y bytes
- **New:** X' lines / Y' bytes
- **Reduction:** __% bytes / __% lines

If byte reduction is <10% on a file you knew was bloated, the pass was timid. Do a second targeted pass before finalizing. (The user is more annoyed by a still-bloated rewrite than by an aggressive stance that removes one thing they'll add back.)

## Phase 6 — Adversarial review

Dispatch `@code-reviewer` with the diff and the Preservation Map. Brief them like this:

> Review uncommitted changes to `<file>`. Goal: this was a verbosity-tightening pass — meaning must be fully preserved.
>
> **Must be intact (verbatim or near-verbatim):**
> - [list every concrete item from the Preservation Map for this file]
>
> **What I cut:** meta-commentary, redundant rationale, table↔prose duplication, multi-paragraph intros saying the same thing twice. Every cut classifies as one of: meta-commentary removed / rationale compressed / redundancy dedup'd / connective tightened.
>
> Verify: any rule, template, enum, anchor, or conditional that disappeared or changed meaning. Cite file:line.

Real findings: address before commit. False positives: explain and proceed.

## Phase 7 — Commit

```bash
git add <file> && git commit -m "$(cat <<'EOF'
refactor: tighten <file> verbosity

[1–2 sentences on what was cut and why this preserves meaning.]
~__% byte / __% line reduction.
EOF
)"
```

---

## Failure modes (avoid)

- **Chunked sub-agents** — they cap at ~5% because they're forbidden from restructuring. The bloat is cross-sectional and only visible holistically. This is the whole reason `/distill-md` exists distinct from `/distill`.
- **Excess timidity on the first pass** — verbose files reliably need a second cut. <10% byte reduction is a signal, not a result.
- **Disagree-and-delete** — preserve rules even ones you'd write differently. The user iterates these files actively.
- **Touching a sub-agent's `description:`** — it's the Task-tool routing API. Shortening it can silently change which agent gets dispatched.

## Out of scope

- Long-form project documentation (`docs/*.md`, design specs, architecture write-ups) — owned by `/project-alignment`; intentionally long-form
- Vendored or generated trees (`vendor/`, `node_modules/`, framework migrations, build output) — irrelevant
- Multi-file structural reorgs — one or two files at a time, focused

## Calibration

Realistic byte reductions:
- **10–15%** on a file already through one tightening pass
- **25–40%** on files that have never been audited
- **<5%** is the chunked-sub-agent ceiling — if you're seeing that, you delegated when you should have read the whole file yourself
