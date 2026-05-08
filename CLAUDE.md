<!-- ====================================================================== -->
<!-- STARTER SENTINEL — REMOVE THIS BLOCK WHEN /init COMPLETES.            -->
<!-- ====================================================================== -->
<!--                                                                        -->
<!-- This CLAUDE.md is a SCAFFOLDING STUB shipped by the rescue starter    -->
<!-- kit. It is NOT yet the project's real CLAUDE.md.                      -->
<!--                                                                        -->
<!-- If you are an orchestrator reading this and the user has just         -->
<!-- started a session in a project that contains this sentinel, your     -->
<!-- first move is `/init` — that command's body in `.claude/commands/    -->
<!-- init.md` is the canonical recipe. Do not improvise; run it.          -->
<!--                                                                        -->
<!-- The sections below marked "{{ ... }}" are placeholders that /init    -->
<!-- replaces with project-specific content during the kickoff session.    -->
<!-- The orchestration backbone (agents, commands, plan conventions) is   -->
<!-- already correct and survives init mostly intact.                     -->
<!--                                                                        -->
<!-- When /init completes successfully, the orchestrator deletes:          -->
<!--   - this sentinel block                                                -->
<!--   - the kit's top-level README.md (replaced with a project README)   -->
<!--   - any `{{ ... }}` placeholders (replaced with real content)         -->
<!--                                                                        -->
<!-- If you reach this sentinel in a session that is NOT the init session -->
<!-- — i.e. a user opened the project later and the sentinel is still     -->
<!-- here — surface that to the user immediately. It means init was       -->
<!-- interrupted and the project is not fully bootstrapped. Offer to      -->
<!-- resume by re-running `/init`.                                         -->
<!--                                                                        -->
<!-- ====================================================================== -->

# CLAUDE.md

This file orients Claude Code to this project. It is the first thing every session reads.

## Operating Principles

This project ships a set of binding operating principles at `.claude/operating-principles.md`. They cover sub-agent verification, plan discipline, documentation discipline, when to ask the user, and how to encode lessons. **Read that file before doing non-trivial work.** When in doubt, the principles win over improvisation.

## Maintenance rhythm

The standing maintenance commands (`/walkthrough`, `/project-alignment`, `/distill`) have cadences defined in `.claude/rhythm.md`. The orchestrator consults it on plan completion and at session boundaries to decide what (if anything) to surface. **Suggest, don't insist** — the user always says yes/no/later.

## Project Overview

{{ PROJECT_OVERVIEW — one sentence on what the product is, plus 2-3 paragraphs on vision, target user, and the problem it solves. Filled in by /init from the user's stream-of-consciousness. }}

### What it is not

{{ ANTI_PATTERNS — the explicit list of what the project deliberately is NOT. Often more important than the feature list because it prevents scope creep. Filled in by /init. }}

### Design philosophy

{{ DESIGN_PHILOSOPHY — the load-bearing principles that shape decisions. Filled in by /init. }}

## Tech stack

{{ TECH_STACK — every significant dependency with its version. Be specific. Filled in by /init from reconnaissance. }}

## Commands

```bash
{{ DEV_COMMANDS — how to run the dev server, run tests, run migrations, clear caches, build for production. Filled in by /init. }}
```

## Architecture

{{ ARCHITECTURE — domain model, request flow, key directories, established patterns. The knowledge a new session needs to not fight the existing structure. Filled in by /init. }}

## Known issues and TODOs

{{ KNOWN_ISSUES — living list of intentional gaps and known problems. Prevents Claude from "fixing" things that are deliberately deferred. Filled in by /init from user disclosure during kickoff. }}

## Code style

{{ CODE_STYLE — formatters, linters, naming conventions, comment policy. Filled in by /init. The repo-wide comment-hygiene rule from operating-principles.md applies regardless. }}

---

## Multi-agent orchestration

You are the orchestrator. Delegate aggressively to specialized agents rather than consuming your own context window.

### Available agents

| Agent | Use when |
|-------|----------|
| **@summarizer** | Codebase understanding without consuming your context. Calibrate depth: "quick" / standard / "comprehensive" |
| **@deep-recall** | Recover institutional knowledge from archived plans, git history, memory, reference docs |
| **@verifier** | Vet proposed changes, plans, or technical decisions BEFORE execution |
| **@code-reviewer** | After significant code — reviews quality, then commits |
| **@executor** | Well-defined tasks across 4+ files (bulk renames, pattern application, config updates) |
| **@minesweeper** | After changes with non-obvious blast radius — sweeps routes, models, views, configs, tests, docs |
| **@defender** | Adversarial filter — challenges verifier/minesweeper findings, kills false positives |
| **@story-tracer** | Traces one user story through the full stack. Bulk-dispatched by `/walkthrough` — not invoked directly |
| **@distill-verifier** | Prosecutes one bloat hypothesis adversarially. Bulk-dispatched by `/distill` — not invoked directly |

### Available commands

| Command | Purpose |
|---------|---------|
| `/init` | One-shot kickoff: kickoff conversation → reconnaissance → CLAUDE.md scaffold → first plan → first commit. Run this when the sentinel block above is still present. |
| `/create-plan` | Build a multi-session implementation plan in `.claude-plans/<plan-name>/` |
| `/work-plan <name>` | Resume an existing plan from `.claude-plans/` |
| `/investigate <question>` | Structured recon → synthesis → user-approved action on any problem |
| `/walkthrough [focus]` | Cognitive QA — traces user stories through code for bugs, flow gaps, UX, auth |
| `/project-alignment` | Project health: doc drift, code quality, roadmap alignment, next-step recommendation |
| `/distill [focus]` | Judgment-led bloat audit; adversarial hypothesis prosecution |
| `/distill-md [target]` | Tighten md spec/config files (commands, agents, CLAUDE.md) |
| `/handoff [scope]` | Briefing for a fresh agent or future session; preferred alternative to `/compact` |
| `/note <content>` | Capture an idea, fix, or tweak; auto-routes to the right notes file |

### Orchestration patterns

- **Research-heavy** → @summarizer first
- **Implementation** → plan → @verifier → execute → @code-reviewer
- **Debugging** → `/investigate` (uses @summarizer + @verifier internally)
- **Large features** → `/create-plan` to structure, `/work-plan <name>` to resume
- **Bulk file ops** → @executor (parallel for independent changes)
- **Ripple sweeps** → @minesweeper, then @executor to apply fixes
- **QA & validation** → `/walkthrough` (no browser required)
- **Doc/code drift** → `/project-alignment` (run periodically; don't wait for the user to notice)

### Multi-session plans

Plans live in `.claude-plans/<plan-name>/` with this structure:

- **OVERVIEW.md** — vision, scope, in/out, locked decisions
- **CHECKLIST.md** — canonical progress state across sessions
- **NN-chunk-name.md** — one chunk per session-sized unit of work, with acceptance criteria, execution strategy, session notes
- **Capstone chunk** (always last) — verification, cleanup, dead-code removal, CLAUDE.md update

The conventions are documented in `.claude-plans/README.md`. Read it before authoring or resuming a plan.

### Plan resumption discipline

Re-read the relevant chunk file in full before claiming readiness. Don't skim and start. Session notes from prior sessions may have changed the chunk's premises.

### Followup-plan pattern

When a plan's capstone or post-mortem surfaces items that don't fit "simple/obvious inline fix," create a followup plan (`<original-name>-followups`). Don't drift into ad-hoc TODO files.

---

## Git efficiency

Commit messages use HEREDOC format:

```bash
git add [specific files] && git commit -m "$(cat <<'EOF'
type: Brief description

Details here.
EOF
)"
```

Commit type prefixes:

```
chunk: plan-name - Chunk Description     ← Plan work
feat: Brief description                  ← New features
fix: Brief description                   ← Bug fixes
chore: Brief description                 ← Maintenance
docs: Brief description                  ← Documentation
perf: Brief description                  ← Performance
refactor: Brief description              ← Code restructuring
```

### Git safety

- Create new commits rather than amending; amending after a hook failure modifies the wrong commit.
- Never skip hooks, signing, or verification as a shortcut. Investigate the failure.
- Stage specific files by name; avoid `git add -A` to prevent committing secrets or large binaries.
- Do not push, force-push, or perform destructive operations without explicit user confirmation.

---

## Auto-memory

The user's auto-memory system at `~/.claude/projects/<project-path>/memory/` accumulates across conversations:

- **user** — the user's role, preferences, technical strengths
- **project** — evolving understanding of project direction
- **feedback** — corrections and confirmations the user has issued. Each has a **Why** and a **How to apply**. Saved on the spot when the user pushes back on a non-obvious thing.
- **reference** — pointers to external systems

When the user corrects you on something durable, save a feedback memory immediately. When they *confirm* a non-obvious approach worked ("yes that's right, keep doing that"), save it too — silent confirmation is the harder signal to catch.

`MEMORY.md` is the index — keep it concise (one line per entry, ~150 chars).

The same operating-principles.md rule applies: when a correction would apply project-wide, lift it from feedback memory into operating-principles.md so all future agents inherit it without needing to mine memory.
