---
description: Produce a handoff briefing a fresh agent or future session can use to continue current work. User's preferred alternative to /compact.
argument-hint: <optional scope, focus, style, or destination directive>
---

# /handoff

Produce a **handoff briefing** — a self-contained document the user can paste to a fresh Claude Code agent (or themselves later) to continue work without re-explaining anything.

**Not `/compact`.** Compaction shrinks the current context; a handoff *transfers* scoped context to a recipient who has none.

**User's directive (if any):** `$ARGUMENTS`

---

## Step 1 — Calibrate scope before you write a word

The directive caps both *what* to include and *how much*. Treat narrowing phrases as hard caps, not weighting — a narrow directive may yield a 15-line handoff. If you're writing sections the directive excluded, delete them.

### (a) Who is the recipient?

| Cue | Assume |
|---|---|
| "for a fresh agent" / no cue / in a long multi-topic session | **Cold agent.** Zero shared context. Must stand fully alone. |
| "for myself" / "for next week" / short focused session | **Future self.** Shares user memory + CLAUDE.md + muscle memory. Don't re-explain the stack, their preferences, or daily commands. |
| "for the PR reviewer" / "for the security reviewer" / named audience | **Specific audience.** Tune to what they need to judge the work — usually the diff and reasoning, not the full project. |

### (b) What is the scope?

| Directive shape | Scope |
|---|---|
| Empty / "full" / "everything" | Whole active-work surface. Longest output. |
| "focus on X" | X primary; other threads mentioned only if load-bearing. |
| "just the Y work" / "the deferred Z items" / any narrowing phrase | **Hard cap.** Produce *only* that slice. Mention other context only if omission would mislead. |
| "terse" / "verbose" | Density override — can shrink a full handoff or expand a narrow one. |

---

## Step 2 — Gather what the recipient will actually need

Before drafting:

1. **Inventory the conversation** within scope. Decided and why? In flight? Tried and rejected?
2. **Verify artifacts on disk.** `git status`, re-read files you edited. Trust-but-verify subagent outputs.
3. **Pull direct user quotes** where phrasing shaped direction — paraphrase loses tone.
4. **Consult memory.** Check `MEMORY.md` for preferences/hazards affecting the recipient. Flag only what's load-bearing.
5. **Identify conversation-only knowledge** — decisions with reasoning, dead ends and why, non-obvious trade-offs, constraints mentioned in passing. Highest-value content at any size.

---

## Step 3 — Structure the handoff

### Minimum required — every handoff has these three

**1. What's going on** — one short paragraph. Initiative / scope / north star. If scope is a punchlist, one sentence is fine.

**2. What's been done + what's left** — the substance. Decisions with their *why*, dead ends with their *why*, on-disk state, work remaining. Lists over prose. Paths + line numbers over descriptions.

**3. Next action** — what the recipient should concretely do. Even "read these three files and tell me your understanding before acting" is valid.

### Optional appendix sections — include ONLY when scope genuinely demands

Each is worth its weight only when "would the recipient fail without this?" is *yes*. Default to omitting. If a cold agent could infer it from CLAUDE.md + a quick `ls`, skip it.

- **Required reading** — paths + one-line descriptions, ordered. Include when the recipient must absorb external artifacts (plans, specs, guides, CLAUDE.md files) before acting. Skip for future-self or narrow scopes where the next-action list stands alone.
- **Environment / project context** — stack, branch, running services, env state. Include only for cold agents OR unusual state (uncommitted changes, running process, branch ahead of remote). Skip when routine.
- **User disposition / working preferences** — autonomy level, tone, hard rules, things they hate. Include for cold agents who will pick up tools and act. Skip for future-self.
- **Hazards and boundaries** — what NOT to do. Include when work touches something destructive, shared, or easy-to-misuse. Skip for tidy local refactoring.
- **Useful commands / tools** — CLIs, subagents, slash commands. Include when tooling is non-obvious. Skip when CLAUDE.md covers it.
- **Success criteria** — how the recipient knows they're done. Include when "done" isn't self-evident. Often one sentence.

### Reference, don't repeat

If a file contains the authoritative version of something, link to it. The handoff is a *map* plus the *conversation-only* knowledge that has no other home.

---

## Style rules for the block

- **Concrete over general.** Paths, line numbers, exact commands.
- **Quote the user** where their phrasing mattered.
- **Separate fact from judgment.** "I chose X because Y" ≠ "X is the case." Mark both.
- **Include dead ends with their why.** Otherwise the recipient re-walks them.
- **Skimmability.** Headers, lists, short paragraphs.
- **No editorializing the recipient.** Don't say "you should think carefully about X." Give them what they need to think with.
- **Preserve structure.** Wrap the block in a clear delimiter (horizontal rule, `### Handoff begins` marker, or triple-backtick fence) so it copies cleanly.

---

## Output format

1. **Brief preamble** — 1–2 sentences. If the directive scoped the work, acknowledge it ("Scoping to the deferred items per your direction"). Point to the block.
2. **Clear delimiter.**
3. **The handoff block** — structured per Step 3.
4. **Optional postamble** — things the *current* user should know but that shouldn't appear in the handoff itself (e.g., "you have uncommitted `.env` changes — worth stashing before the fresh agent picks up"). Terse; omit if nothing to add.

---

## Anti-patterns to avoid

- **Padding narrow scopes into full-scope handoffs.** "Just the Y work" doesn't mean "include 10 unrelated sections for completeness."
- **Producing every section because the template lists it.** Minimum required is 3; everything else is conditional.
- **Rehashing what the user already knows.** Handoffs are for the recipient. No "as you know…"
- **Summarizing when specifics are what's needed.** "Made some settings changes" ≠ "Rewrote `.claude/settings.local.json` from 3 deny + 17 allow to 26 ask + 1 allow + 12 sandbox hosts."
- **Confident claims without verification.** If unsure what's on disk, read it.
- **Ending without a next action.** Even "read these three files and report back" counts.
- **Over-condensing concerns from the prior session.** 7 nuanced open items shouldn't flatten to "some concerns remain."
- **Losing the *why* behind rejected approaches.** Without the why, the recipient re-tries the rejection.
- **Re-stating content that lives in a file.** Link to `CLAUDE.md` or the plan; don't quote it.
