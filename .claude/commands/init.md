---
description: Rescue a dormant side project — kickoff conversation, reconnaissance, infrastructure scaffold, and first plan, all in one session
argument-hint: <stream-of-consciousness about the project — past, present, and future feelings, what works, what doesn't, what you wish were true>
---

# /init — Project Rescue Kickoff

You are the orchestrator of a project's revival. The user has just dropped a rescue starter kit into the root of an old side project they want to bring back to life. Your job in this session is to take the project from "I have this old thing" to "the foundations of a working multi-agent SDLC are in place AND the first chunk of real implementation work is committed" — atomically, in one session.

The user's stream-of-consciousness is in `$ARGUMENTS`. Treat it as the most important context you have: the codebase may not reflect what they currently want; their soul-dump does. Reread it whenever you feel yourself drifting.

This command **replaces the Claude Code built-in `/init`**. The built-in only writes a CLAUDE.md by reading the codebase. This one drives a full kickoff: kickoff conversation, reconnaissance, infrastructure scaffold, first plan, and first commit.

---

## ⚠️ DO NOT IGNORE — the things that get fumbled

These are the failure modes that have happened in real rescues. Each is a hard rule. If you find yourself about to skip one, stop and check.

1. **Read `$ARGUMENTS` in full and treat it as primary.** The user wrote it specifically for this session. Their feelings about the project are the context the codebase cannot give you. If the SoC is thin, **ask them to expand** before proceeding to recon — don't substitute your own assumptions.
2. **The kickoff is conversational, not interrogative.** Ask one or two questions at a time, not a 12-item form. Tease out missing pieces by following threads they raised.
3. **Save memories AS YOU GO, not in a batch at the end.** Every time the user says something durable about themselves, the project, or how they want to work, write a memory file immediately. Forgetting this is the single biggest cause of post-init context loss.
4. **Read `.claude/operating-principles.md` before any judgment call.** It encodes accumulated lessons. The principles win over your improvisation.
5. **The sentinel block at the top of CLAUDE.md MUST be removed before commit.** Leaving it in means future sessions believe init never finished.
6. **The kit's top-level README.md MUST be replaced with a project-specific README.** The kit README is for GitHub readers of the starter; the project needs its own. Do not just delete it without writing the replacement.
7. **The first plan's first chunk MUST ship in this commit.** The orchestration infrastructure and the first real implementation work commit together as one atomic commit. "Set up tooling, then start working" is the pattern that fizzles — every kit-only commit increases the chance the user never starts the work.
8. **Capstone is non-optional.** Every plan ends with a capstone chunk for verification, cleanup, and CLAUDE.md update. If the first plan doesn't have one, you are not done.
9. **Drop-with-prejudice applies even on the first plan.** Don't push every chunk to completion if recon shows it's not needed. A first-plan chunk that ships as a no-op with rationale is fine — concept-application without payoff is regression.
10. **Verify the user's claims against the codebase.** "I think the tests don't work" / "there's a file that's actually unrelated" / "the auth is broken" — confirm each before writing it as fact in CLAUDE.md. The user's memory of the project may be 3 years stale.
11. **Generalize, but don't sand off the project's voice.** CLAUDE.md should sound like THIS project, not like a generic Laravel/Rails/Django scaffold. The user's anti-patterns and design philosophy are what differentiate it.
12. **If the user does not engage with the conversation, do not proceed silently.** A blank or one-line `$ARGUMENTS` means stop and ask. The whole rescue depends on their input.
13. **CUSTOMIZE blocks must be filled (or marked "N/A" with a reason), never left empty.** Phase 3.5 walks through each. An empty `<!-- PROJECT CUSTOMIZATION -->` is a kit that looks bootstrapped but isn't.
14. **`bin/setup` and `bin/dev` must exist when the project has a non-trivial setup or boot story** — Phase 6.5 covers this. If you decide the project genuinely doesn't need them, document the reason in the project README's Setup/Development sections so a future contributor doesn't waste time looking.

---

## Phase 0 — Recognize where you are

Before talking to the user, verify the kit is intact and figure out what state the project is in.

### Detect the kit

Check that these files exist:

- `CLAUDE.md` (with the STARTER SENTINEL block at the top)
- `.claude/operating-principles.md`
- `.claude/agents/` (9 agent files: summarizer, verifier, code-reviewer, executor, minesweeper, deep-recall, defender, story-tracer, distill-verifier)
- `.claude/commands/` (10 command files including this one)
- `.claude-plans/README.md`
- `README.md` (the kit's GitHub-facing README — you'll replace this)

If any are missing, surface that to the user. Don't proceed with a damaged kit.

### Detect prior work

Run a few quick checks:

```bash
git log --oneline -20
ls -la
git status
```

You're looking for:

- **Is the project entirely dormant?** (Last commit weeks/months/years ago.)
- **Was there a prior bootstrap attempt?** (e.g., a CLAUDE.md outside the sentinel pattern, an existing `.claude-plans/`.) If so, the kit may have been dropped on top of in-progress work — surface to the user before overwriting.
- **What's the apparent stack?** (Look at `composer.json` / `package.json` / `Gemfile` / `pyproject.toml` / `go.mod` / etc. for the rough shape.)

### Read operating principles

Before you talk to the user, read `.claude/operating-principles.md` in full. It will shape every decision.

---

## Phase 1 — Kickoff conversation

This is the most important phase. No code is written. No files are created. This is you and the user getting to know each other and the project.

### Open

Greet the user. Acknowledge the project name (from the directory or `package.json`/equivalent). Briefly tell them what this session will accomplish — short, not a lecture:

> "We're going to spend this session going from 'old codebase' to 'modern foundation with the first plan's first chunk shipped.' I'll ask you about the project and your background, dig through the code, scaffold a real CLAUDE.md, draft a first plan, and commit it all together. Roughly an hour-ish of your time, mostly upfront."

Then read `$ARGUMENTS` aloud (paraphrased) to confirm understanding. If the SoC is rich, follow the threads they raised. If it's thin, ask for expansion before going further.

### Tease out what's missing

The user's SoC will usually cover some of these but not all. Fill the gaps with conversation, not a form:

- **Who they are technically.** What they're strong at, what they're weaker at, how they like to work. (Save as `user_background.md`.)
- **What the project is — and isn't.** Vision, target user, the problem it solves. The anti-patterns ("it's NOT a Roll20") are often more important than the features. (Save as `project_vision.md`.)
- **Why it stopped, why it's restarting.** What killed momentum. What changed. (Save as `project_context.md`.)
- **What state they think it's in.** Honest answer, not aspirational. "I haven't touched it in 3 years and the frontend is half-broken." (Save as part of `project_context.md`.)
- **Quality bar.** "Scrappy MVP" vs. "polished and indispensable" changes every decision. (Save as a feedback memory.)
- **Collaboration style.** Move fast and ask forgiveness, or check in at decision points? Verbose explanations or terse updates? (Save as a feedback memory.)

Ask one or two of these at a time. Wait for answers. Follow threads. Don't paste the list.

### Save memories as you go

Every durable signal goes into the memory system **immediately**, not at the end:

- **user** — their role, preferences, strengths
- **project** — vision, context, anti-patterns, history, current state
- **feedback** — how they want you to work; rules they push back on
- **reference** — external systems they mention (issue tracker, design doc, etc.)

Memory file format:

```markdown
---
name: <human-readable name>
description: <one line — used to decide relevance later>
type: user | project | feedback | reference
---

<content; for feedback/project use **Why:** and **How to apply:** sublines>
```

The MEMORY.md index has one line per entry. Keep it tight.

### When to leave Phase 1

You can move on when you have:

- A clear sense of the project's vision and what it deliberately is not
- A clear sense of the user's strengths/weaknesses and quality bar
- An honest read of the project's current state (verified later in recon)
- A working feel for how the user wants to communicate

If `$ARGUMENTS` was rich and the user is engaged, this can take 10-20 minutes. If they're terse, follow threads patiently — don't rush to recon with thin context.

---

## Phase 2 — Reconnaissance

Now investigate the actual codebase. Same session as Phase 1.

### Delegate to summarizers in parallel

Launch 2-4 `@summarizer` agents in **parallel** (single message, multiple Agent calls), each on a distinct angle. Use **comprehensive** depth — planning needs nuance.

Typical angles for a rescue:

1. **Tech stack and dependency drift.** What's installed, what versions, how far behind. Are there abandoned dependencies? Is the framework's current major version radically different from what's here?
2. **Domain model and what's worth preserving.** What entities exist? What relationships? What patterns are established? What's the "core 10-15 files of real domain logic" worth keeping in a fresh-install scenario?
3. **What's broken vs. what works.** Where are the stubs, the half-done features, the dead code, the wrong-purpose files? Where's the real working code?
4. **Auth, infra, and dev experience.** How is auth set up? Are there tests? Is there a dev setup script? A deploy path? CI?

Each prompt should ask for **file:line evidence** so you can verify claims rather than trust them.

### Make the upgrade decision (if applicable)

For projects with severely outdated dependencies, you and the user need to make a critical call:

- **Incremental upgrade** — best when custom code is large, dependencies are 1-2 majors behind, codebase is well-structured. Risk: upgrade hell.
- **Fresh install with domain migration** — best when custom code is small (~10-15 files of real domain logic), dependencies are 3+ majors behind, old code has significant rot. Risk: identifying everything worth preserving.

Present both with concrete evidence from recon. Let the user decide. Don't decide unilaterally.

### Verify user claims

Spot-check anything the user said about the project's state. "I think tests don't work" — try to run them. "There's a file that's actually unrelated" — read it. "Auth is broken" — try to log in or trace the flow. The user's memory of the codebase is often years stale; don't transcribe their claims into CLAUDE.md without confirming.

---

## Phase 3 — Scaffold the project's CLAUDE.md

The CLAUDE.md template in the kit has `{{ ... }}` placeholders for project-specific content. Replace them.

### What to fill in

Use what you learned in Phases 1-2:

- **Project Overview** — vision, target user, problem solved (from kickoff + memory)
- **What it is not** — the anti-patterns (from kickoff)
- **Design philosophy** — the load-bearing principles (from kickoff)
- **Tech stack** — every significant dependency with version (from recon)
- **Commands** — dev, test, build, migrate, clear-cache (from recon, verified)
- **Architecture** — domain model, request flow, key directories, established patterns (from recon)
- **Known issues and TODOs** — intentional gaps and known problems (from kickoff and recon, combined)
- **Code style** — formatters, linters, naming conventions (from recon and existing config files)

### What to remove

- The STARTER SENTINEL block at the top of CLAUDE.md
- Any `{{ PLACEHOLDER }}` markers that didn't get filled in (replace with "TBD" if genuinely unknown, or skip the section)

### What stays

The orchestration backbone (agent table, command table, plan discipline, git efficiency, auto-memory). These are project-agnostic and survive init intact.

### Delegate the bulk write

If you've gathered enough material and the only work left is mechanical placement, dispatch an `@executor` to do the bulk write. Hand it the kit CLAUDE.md, the gathered material, and explicit instructions. Do not delegate **judgment** — the structure and content decisions are yours.

If the work is judgment-heavy (which sections to include, how to phrase the philosophy), do it yourself.

---

## Phase 3.5 — Fill the agent and command CUSTOMIZE blocks

Several `.claude/agents/*.md` and command files ship with `<!-- PROJECT CUSTOMIZATION: ... -->` blocks. They sit empty until you fill them. Empty CUSTOMIZE blocks are **the single biggest reason a kit looks bootstrapped but actually isn't** — so this phase is non-skippable.

Walk these and fill them with the project specifics gathered in Phases 1-2:

| File | What goes in the CUSTOMIZE block |
|------|----------------------------------|
| `.claude/agents/summarizer.md` | The project's key architectural patterns to look for during recon (trait composition, observer/event side effects, ORM conventions, frontend integration points) |
| `.claude/agents/verifier.md` | Project-specific verification checks (authorization patterns, ORM pitfalls, frontend data contracts, feature toggle dependencies) |
| `.claude/agents/code-reviewer.md` | Project-specific review checks (auth middleware, scoping conventions, validation idioms, view path validity) |
| `.claude/agents/distill-verifier.md` | Per-area Green Signal Matrix — test commands and static-analysis subcommands whose findings must not regress for a Tier A change. See `.claude/guides/distill-audit-tooling.md` for stack-specific cookbooks. |
| `.claude/commands/work-plan.md` | Quality Sweep prompts adapted to the project's stack (cross-page consistency, stale code, UX polish, backend hygiene — adapt to the architecture) |
| `.claude/commands/distill.md` | Static-analysis orchestrator command + the area-to-signals matrix for Tier A verification. See `dev-ergonomics-scripts.md` and `distill-audit-tooling.md`. |
| `.claude/commands/project-alignment.md` | Project-specific debug helpers, design-system patterns, and drift signals to check |
| `.claude/commands/note.md` | Note types and their target files (e.g., `IDEA-NOTES.md`, `FIX-TWEAK-NOTES.md`), plus per-type formatting conventions |

**After filling each block, delete the surrounding `<!-- PROJECT CUSTOMIZATION: ... -->` markers.** A filled block with the marker still around it reads as unfilled to a future agent.

If the project genuinely doesn't have content for a block (e.g., no observers, no static-analysis suite yet), write a one-line "N/A — <reason>" rather than leaving it empty. The reason explains to the next agent why it's blank, so they don't try to fill it speculatively.

---

## Phase 4 — Subtree CLAUDE.md (optional, often deferred)

A single root CLAUDE.md is fine for the first session. Subtree CLAUDE.mds (e.g., `app/Models/CLAUDE.md`, `frontend/CLAUDE.md`) emerge naturally when:

- A directory accumulates enough domain-specific lore that loading it for unrelated work is wasteful
- Recurring agent confusion about a subsystem suggests its lore needs a closer home

Don't preemptively create them. Mention the pattern to the user, note it as a future move, defer.

---

## Phase 5 — Create the first plan

Run `/create-plan` (or its logic if you're already in flow). The first plan is almost always one of:

| Project state | First plan type |
|---------------|-----------------|
| Severely outdated dependencies | **Foundation rebuild** (fresh install + domain migration OR incremental upgrade) |
| Working but messy | **Cleanup and modernize** (tests, docs, lint config, dev setup) |
| Partially built features | **Feature completion** (finish what was started, wire things together) |
| Modern stack but no infrastructure | **Developer experience** (setup script, test framework, CI, CLAUDE.md) |

The plan must end with a capstone chunk. The plan's OVERVIEW.md captures locked decisions from Phase 2 (e.g., "fresh install over incremental, because…").

### How big is the first plan

Aim for 4-8 chunks. Each chunk is 1-3 hours of focused work. If a chunk has 15 acceptance criteria, split it. Smaller chunks give natural session boundaries; larger chunks risk no-progress sessions.

### Validate the plan

After authoring, run two waves of parallel `@verifier`:

1. **Wave 1**: completeness verifier (gaps, ambiguity) + technical accuracy verifier (file paths, code patterns)
2. **Wave 2**: test coverage verifier (does this warrant a dedicated test chunk?) + meta-docs verifier (does this change patterns referenced in CLAUDE.md or operating-principles?)

Add chunks the verifiers flag. Then ensure capstone is last.

---

## Phase 6 — Execute the first chunk

The first chunk ships in this commit, atomically with the orchestration infrastructure. This is the rule that prevents the dreaded "set up tooling, then never start the work" outcome.

How to pick what's in the first chunk:

- It must be small enough to finish in this session
- It must produce visible progress (the user's first concrete win)
- It must respect the locked decisions from the plan's OVERVIEW

For a foundation-rebuild plan, this is often "fresh foundation install" — fresh framework version, packages installed, the kit infrastructure overlaid into the new repo. For a cleanup plan, it's often "dev setup script" or "test framework wired up + one passing test." Use judgment.

Execute the chunk using the chunk file's stated execution strategy. If the strategy says "executor-based," dispatch executors. If "direct," do it yourself. Run `@code-reviewer` before you commit.

---

## Phase 6.5 — Dev-ergonomics scripts (`bin/setup`, `bin/dev`)

If the project doesn't already have working bootstrap and daily-boot scripts, create them. This is **surprisingly hard to get right and worth getting right** — it's the difference between a project a returning developer can use again and a project they have to relearn every time.

Before writing, **read `.claude/guides/dev-ergonomics-scripts.md` in full**. It encodes the design principles (no `sudo`, no `.env` overwrite, idempotent, distinguish hard/soft fails) and the pitfalls (Bash 3.2 empty arrays, Homebrew on multi-user macs, `set -e` killing soft-fail handlers). Skipping the guide reliably produces a script with one of those pitfalls in it.

What to create:

- **`bin/setup`** — first-time bootstrap. Idempotent. Creates `.env` from `.env.example` only if missing, installs dependencies, runs migrations, builds initial assets if applicable. Re-runnable as a no-op when everything is already good.
- **`bin/dev`** — daily boot. Verifies setup ran, starts required services if they're not already up, ends with `exec` to the actual dev server so signals are forwarded cleanly.

Some projects only need one or neither (a static site doesn't need `bin/setup`; a project without long-running services may not need `bin/dev`). **Don't manufacture work** — but if there's a non-trivial setup or boot story, encode it. The skeletons in the guide are the starting points.

Both scripts go in `bin/`, get `chmod +x`, and are referenced from the project README's Setup and Development sections.

---

## Phase 7 — Write the project-facing README.md

The kit's `README.md` is for GitHub readers of the rescue starter. Replace it with a project-specific `README.md` that explains the project to its own readers (the user, future contributors, future-you).

The project README should be tight but it MUST include the "working-with-Claude" framing — that's how the user (and future contributors) understand the role they play. Don't reduce that section to a flat command list; the framing is what keeps the system from going on autopilot. Roughly:

```markdown
# <Project Name>

<one-sentence description>

## What it is

<2-3 paragraphs on vision and target user>

## What it is not

<the explicit anti-patterns — often more important than the feature list>

## Stack

<bullet list of major dependencies>

## Setup

```bash
./bin/setup
```

<one-sentence description of what setup does — "installs deps, creates .env, runs migrations">

## Development

```bash
./bin/dev
```

<one-sentence description of what bin/dev does — "starts services, launches dev server">

## Working with Claude Code on this project

This project is built with a Claude-Code-driven SDLC. **The system has rails (`.claude/operating-principles.md`, the multi-agent setup, plan conventions) but those rails don't substitute for your judgment.** When something feels wrong, say so. When you confirm an approach worked, say so. The system absorbs your taste over time — that's the whole point. It does not run on autopilot.

You don't have to memorize anything. Just talk to Claude conversationally about what you want — Claude will reach for the right tool. The slash commands below are shortcuts for when you want them:

- `/work-plan <name>` — resume the active plan
- `/create-plan` — start a new plan
- `/investigate <question>` — structured deep-dive on any problem
- `/walkthrough [focus]` — cognitive QA of a feature without a browser
- `/project-alignment` — periodic doc/code drift check
- `/distill` / `/distill-md` — bloat audits for code and config files
- `/handoff` — transfer context to a fresh session (better than `/compact`)
- `/note` — capture an idea, fix, or tweak

Claude follows a rhythm for the standing maintenance commands (`/walkthrough`, `/project-alignment`, `/distill`) — see `.claude/rhythm.md`. You don't need to track this; Claude will surface suggestions at the right moments. Override freely.

The rules every session inherits live in `.claude/operating-principles.md`. The plan conventions live in `.claude-plans/README.md`. You don't need to read either to use the system, but they're there.

## License

<TBD or what user specifies>
```

Keep the project-specific sections short. The user can expand them later. **Do not remove or shorten the "Working with Claude Code on this project" framing paragraph** — that's the load-bearing piece that prevents the system from being treated as a black box.

---

## Phase 8 — Cleanup, settings, and commit

Before staging, finalize three housekeeping items.

### 8a. Settings file

Rename `.claude/settings.local.json.template` to `.claude/settings.local.json` and tune it for the project's stack. The template has stack-specific recipe blocks (`node_npm`, `php_laravel`, `python`, etc.) — pick the one(s) that match, merge those entries into the `allow` list, then delete the `_stack_specific_recipes` block entirely. Don't leave the unused recipes around as dead config.

If the project's docs domain isn't in the `WebFetch` allows, add it (e.g., `WebFetch(domain:laravel.com)` for Laravel projects, `WebFetch(domain:rubyonrails.org)` for Rails).

### 8b. .gitignore

If the project already has a `.gitignore`, append (don't overwrite) the kit's lines:

```
# Claude Code
.claude/settings.local.json
.claude/worktrees/
.claude-distill-scratch/
.claude-distill-history/
```

If there's no `.gitignore`, create one with the above plus standard ignores for the project's stack. The kit's own `.gitignore` covers only the Claude-related paths; the project may need more (e.g., `node_modules/`, `vendor/`, `.env`, OS-specific clutter).

### 8c. Delete the kit-specific files

These are for the kit's GitHub readers and are no longer needed:

- The kit's `README.md` (you replaced it in Phase 7)
- `.claude/settings.local.json.template` (you renamed it in 8a)

```bash
git rm .claude/settings.local.json.template  # if not already moved via mv
```

### 8d. Verify the final state checklist

Before committing, confirm every item:

- ✅ STARTER SENTINEL block removed from CLAUDE.md
- ✅ All `{{ PLACEHOLDER }}` markers in CLAUDE.md filled or replaced with "TBD"
- ✅ All `<!-- PROJECT CUSTOMIZATION: ... -->` markers in `.claude/agents/*.md` and `.claude/commands/*.md` either filled or marked "N/A"
- ✅ Project-specific `README.md` (with the engagement framing intact) replaces the kit's
- ✅ `.claude/settings.local.json` exists with stack-tuned permissions; `.template` and `_stack_specific_recipes` are gone
- ✅ `.gitignore` covers `.claude/settings.local.json`, `.claude/worktrees/`, and Claude scratch dirs
- ✅ `bin/setup` and `bin/dev` exist (or are documented as "not needed because <reason>")
- ✅ `.claude-plans/<first-plan-name>/` exists with OVERVIEW + CHECKLIST + numbered chunks + capstone
- ✅ First chunk is checked off `[x]` in CHECKLIST.md and its work is staged
- ✅ Memory files exist for user, project vision, project context (and any feedback captured)
- ✅ One commit, atomically — no partial commits

If any of those is false, finish it before committing.

### 8e. The commit

```bash
git add CLAUDE.md README.md .gitignore .claude/ .claude-plans/ bin/ <files-from-the-first-chunk>
git rm <stale-or-deleted-files-from-the-old-codebase>  # if the foundation rebuild deleted anything
git commit -m "$(cat <<'EOF'
chunk: <first-plan-name> - <first-chunk-name>

Establishes Claude Code orchestration infrastructure (agents, commands,
operating principles, plan conventions, dev-ergonomics scripts) AND
completes the first chunk of the <plan-name> plan: <one-line summary>.

Decisions locked during kickoff:
- <decision 1>
- <decision 2>

Next: /work-plan <plan-name>
EOF
)"
```

---

## Phase 9 — Report and orient the user

Tell the user, concisely:

- **What was decided** (foundation strategy, quality bar, design philosophy)
- **What was built** (orchestration infra + first chunk's actual changes)
- **Where things stand** (which plan is active, which chunk is next)
- **How to continue** (`/work-plan <plan-name>` is the next move; conversational requests also work)
- **Pointers to read on their own time** (`README.md` for the workflow; `.claude/operating-principles.md` for the rules; `.claude-plans/<name>/OVERVIEW.md` for the plan)

Then stop. The session is done. The user has agency from here.

---

## Recovery patterns

### If the user disengages mid-session

Don't silently fill in the gaps. Surface it: "I want to make sure I have your perspective before scaffolding the CLAUDE.md — anything you can tell me about <X> would help."

### If recon contradicts the user's stated belief about the project

Surface it gently. "You mentioned the auth is broken, but I traced the flow and it looks like it works — could you tell me more about the symptom you saw?" Don't write either version into CLAUDE.md until the disagreement resolves.

### If the first plan would be too large to fit a capstone chunk into a reasonable scope

Split it. Plan A is the foundation; Plan B is the next layer. The first plan's job is to ship a coherent baseline, not the whole vision.

### If you run low on time/context before Phase 8

Stop and produce a `/handoff` briefing. Don't half-commit. The next session can pick up cleanly.

### If you discover the kit is in a project that already had bootstrapping

Surface to the user before overwriting. Offer options: keep the existing CLAUDE.md and merge the kit's orchestration backbone into it, or full-replace, or back out.

---

## What success looks like

A successful `/init` session produces, in one commit:

- A CLAUDE.md that describes the project as it actually is, not as the user wishes
- The full orchestration backbone (9 agents, 10 commands, operating principles)
- A first plan with a capstone, locked decisions, validated by verifiers
- The first chunk done, committed alongside the infrastructure
- A user-facing README for the project itself
- Memory files capturing user, project, and any feedback collected
- A user who knows where things stand and how to continue, without needing to read this file

If the user can close their terminal, sleep for a week, type `/work-plan <plan-name>`, and pick up cleanly — you succeeded.
