# Rescue an Old Side Project with Claude Code

A drop-in `.claude/` starter kit for reviving a dormant side project with [Claude Code](https://claude.ai/code).

You have a project that meant something to you, sitting on an old framework version, with broken pieces and no tests. This kit gets you from "I have this old thing" to "I have a modern foundation with the first chunk of real work shipped" in **one Claude Code session** — without you having to babysit a god-document.

It works because it ships:

- **9 specialized agents** — summarizer, verifier, code-reviewer, executor, minesweeper, deep-recall, defender, story-tracer, distill-verifier
- **10 slash commands** — `/init`, `/create-plan`, `/work-plan`, `/investigate`, `/walkthrough`, `/project-alignment`, `/distill`, `/distill-md`, `/handoff`, `/note`
- **Plan conventions** — `.claude-plans/` directory shape with capstone discipline and followup-plan culture
- **Operating principles** — accumulated lessons (sub-agents are not oracles, drop-with-prejudice, version your archives, etc.) loaded into every session
- **A custom `/init`** — replaces Claude Code's built-in init with a full kickoff: conversation, reconnaissance, CLAUDE.md scaffold, first plan, and first commit, atomically

---

## How to use it

### 1. Drop the kit into your project's root

```bash
# From your dormant project's root directory:
git clone --depth 1 https://github.com/cooleydw494/rescue-old-sideproject-with-claude.git /tmp/rescue-kit
rsync -a --exclude='.git' /tmp/rescue-kit/ ./
rm -rf /tmp/rescue-kit
```

This drops `CLAUDE.md`, `.claude/`, `.claude-plans/`, and `README.md` (this file) into your project's root, sitting alongside your existing code. None of your code is touched.

If your project already has a `CLAUDE.md` or `.claude/` directory, the kit will overlay them. The `/init` command detects prior bootstrapping and asks before overwriting.

### 2. Open Claude Code in the project root and run `/init`

Open a Claude Code session in the project directory. Run:

```
/init <your stream-of-consciousness about the project>
```

The argument to `/init` is the most important thing in the whole rescue. It is **not** a requirements doc — that lives in the codebase. It is your honest, current, unfiltered feelings about the project. Pull from depths you wouldn't normally write down.

Things to include:

- What the project is and why it mattered to you
- Why you stopped working on it
- What you wish were true about it
- What feels broken or wrong (even if you don't know why)
- What still feels right or worth keeping
- Where you think the design is weak
- What kind of user this is for
- What a v1 would look like that would feel indispensable to that user
- How you like to work with a collaborator — verbose vs. terse, fast vs. checkpoint-y, polish-now vs. ship-fast
- Anything else you'd say to a friend over coffee about this project

A thin one-line argument will produce a thin rescue. The system has to know who you are.

### 3. Have the kickoff conversation

`/init` will ask follow-up questions to fill in what your stream-of-consciousness didn't cover. Answer honestly — including the parts you're embarrassed about ("I never finished the auth", "I think the database schema is wrong"). The orchestrator needs the real picture, not the aspirational one.

The kickoff phase produces persistent memory files about you, the project, and how you want to work. These survive across all future sessions.

### 4. Let it run

After kickoff, `/init` runs reconnaissance (parallel summarizer agents), drafts a CLAUDE.md, creates the first plan, executes the first chunk of that plan, and commits everything together as one atomic "rescue era" commit. This typically takes 30-90 minutes depending on the project's size.

When it's done, this README.md is deleted and replaced with a project-specific README.md, the `STARTER SENTINEL` block is removed from CLAUDE.md, and the project is fully bootstrapped.

### 5. Continue with `/work-plan <plan-name>`

From here on, you can continue with structured slash commands or just talk conversationally to Claude. Both work. The orchestration infrastructure is there when you want it.

---

## What you should know going in

**You're a real collaborator, not a passive consumer.** Claude can't do this without you. Your judgment about the product, your taste, your sense of what's right — those are non-negotiable inputs. The system is built to make your input as valuable as possible per minute, but you have to bring it.

**The first session is the heaviest.** Phase 0 (kickoff conversation) takes 15-30 minutes of your engaged attention. After that, ongoing work is much lower friction — `/work-plan` resumes from where you left off, even weeks later.

**Slash commands are optional.** You don't need to memorize them. If you say "I want to investigate why X feels off," Claude will reach for `/investigate` itself. The commands are there for when you want them.

**The system instills itself.** Every session reads `.claude/operating-principles.md`. Every plan ends with a capstone. Every chunk has acceptance criteria and a "drop-with-prejudice" clause. You don't have to remember to enforce these — the rails are built in.

**Course-correct freely.** When Claude does something wrong, say so. The session captures the correction as a feedback memory and future sessions inherit it. The system is designed to absorb your taste over time.

---

## What's in the kit

```
rescue-old-sideproject-with-claude/
├── README.md                    ← This file. Deleted at end of /init.
├── CLAUDE.md                    ← Sentinel-stub at start; rewritten by /init.
├── .claude/
│   ├── operating-principles.md  ← Binding rules. Survives forever.
│   ├── agents/
│   │   ├── summarizer.md
│   │   ├── verifier.md
│   │   ├── code-reviewer.md
│   │   ├── executor.md
│   │   ├── minesweeper.md
│   │   ├── deep-recall.md
│   │   ├── defender.md
│   │   ├── story-tracer.md
│   │   └── distill-verifier.md
│   ├── commands/
│   │   ├── init.md
│   │   ├── create-plan.md
│   │   ├── work-plan.md
│   │   ├── investigate.md
│   │   ├── walkthrough.md
│   │   ├── project-alignment.md
│   │   ├── distill.md
│   │   ├── distill-md.md
│   │   ├── handoff.md
│   │   └── note.md
│   └── settings.local.json.template
└── .claude-plans/
    └── README.md                ← Plan directory conventions.
```

---

## Requirements

- A working [Claude Code](https://claude.ai/code) install
- A git repository for your project (or willingness to `git init` one)
- An hour of focused attention for the kickoff conversation

That's it. No external services, no API keys, nothing to configure.

---

## Why this works

It's the codification of a workflow that worked, repeatedly, on real dormant projects. The agents and commands are not theoretical — each was invented in response to a specific friction during a real rescue:

- `/walkthrough` and `story-tracer` came from "I need cognitive QA without paying for a full re-verification of every flow"
- `/distill` came from "the project is accumulating bloat faster than I can prune it manually"
- `/handoff` came from "compaction loses too much context; I need a real briefing"
- `/project-alignment` came from "the docs drifted and I didn't notice for two weeks"

The operating principles are the residue of mistakes. Every rule is "the lesson burned in" after a real failure mode. They are loaded into every session so you don't relive the mistake.

---

## License

MIT. Fork it, tune it, share your tuning. The kit is meant to evolve.
