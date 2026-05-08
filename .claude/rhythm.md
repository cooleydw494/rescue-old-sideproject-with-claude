# Project Rhythm — When to Run What

This file names the cadences for the standing maintenance commands. The orchestrator consults it on plan completion and at session boundaries to decide what (if anything) to surface to the user. The rule is **suggest, don't insist** — the user always says yes/no/later.

The point is to prevent the failure mode you saw in real rescues: the commands exist, but nobody remembers when to run them, so they don't get run, and the project drifts.

---

## The cadence table

| Command | Trigger | What it does |
|---------|---------|--------------|
| `/walkthrough [focus]` | After every **2-3 completed plans** that touched user-facing flows, OR before any release | Cognitive QA — traces user stories through the code without a browser. Catches flow gaps, auth issues, UX problems, dead ends. |
| `/project-alignment` | After every **5-7 completed plans**, OR whenever the user says "this doesn't match the docs" | Full project health check — doc drift, code quality, roadmap alignment, next-step recommendation. |
| `/distill [focus]` | After every **major refactor** or **dependency upgrade**, OR when files start "feeling" bloated | Judgment-led bloat audit. Cold-read the codebase, prosecute hypotheses adversarially. Zero findings is a valid result. |
| `/distill-md [target]` | After **agent or command files have grown** through several edits, OR before publishing the kit elsewhere | Tighten md spec/config files. Cuts verbosity and structural bloat. |
| `/handoff` | When **context utilization is climbing** and a session boundary is approaching, OR before passing work to a fresh agent | Produces a portable briefing. Preferred alternative to `/compact` because it transfers scoped context, not compressed everything. |
| Subtree CLAUDE.md creation | When a directory **accumulates enough domain-specific lore** that loading it for unrelated work is wasteful | Spin off `app/Models/CLAUDE.md`, `frontend/CLAUDE.md`, etc. Reference from root. Locality matters. |

---

## How the orchestrator uses this

### On plan completion (in `/work-plan`)

After the post-mortem and quality sweep, before archiving the plan, the orchestrator:

1. Checks the count of plans completed since the last `/walkthrough`. If ≥ 2-3 AND user-facing flows changed, suggest `/walkthrough` on the affected stories.
2. Checks the count of plans completed since the last `/project-alignment`. If ≥ 5-7, suggest a drift check.
3. If the just-completed plan was a major refactor (touched 20+ files OR rewrote a subsystem), suggest `/distill` on the touched area.

How the orchestrator counts plans: by reading `.claude-plans/archive/` and looking at directory mtimes, or by maintaining a tiny ledger in memory (`reference_rhythm_log.md` is fine — one-line-per-event).

### Mid-session

When the orchestrator notices any of these, surface a one-line nudge:

- User is pasting a long context dump that should be `/handoff` material → "This sounds like a `/handoff` brief — want me to format it that way?"
- User is debugging a problem that touches 4+ subsystems → "This is `/investigate` territory if you want a structured pass."
- User is reviewing a feature manually that has user stories → "Want me to run `/walkthrough` on this flow first to surface what to look at?"

**Surface once, then drop it.** If the user says "no, just keep going," don't bring it up again that session.

### Anti-pattern: nag

Don't nudge every command on every session. Don't list every available command at the start. Don't ask "do you want to run /walkthrough?" before every chunk. The cadences above are the floor — fire the nudge when the trigger is met, otherwise stay quiet.

---

## How the user uses this

The user does not need to read this file. The orchestrator does the bookkeeping. But if the user is curious about why Claude just suggested `/distill`, they can read it here and either trust the rhythm or override it ("ignore that, we're shipping").

---

## Tuning the cadence

These intervals are starting points. If the project finds them too frequent ("you keep suggesting alignment and there's never anything to find"), the rhythm should slow. If the project is consistently behind on drift, it should speed up. Update the table here when the rhythm gets tuned, and note the change in a feedback memory so future agents know.

The rhythm log itself can live in memory:

```markdown
---
name: Rhythm log
description: One-line entries logging when standing maintenance commands ran, to drive cadence decisions
type: reference
---

- 2026-05-08 — /walkthrough on auth flow (after foundation rebuild)
- 2026-05-15 — /project-alignment (after solid-ground plan)
- 2026-05-22 — /distill on app/Http (after dejetstreamification)
```

Three lines, append-only, easy to scan. Don't over-engineer it.
