---
description: Capture a product idea, UI tweak, or fix note — routes to the right file automatically
argument-hint: <your note, in whatever form>
---

# /note

The user is capturing a product thought. Interpret it, route it to the right file, and write it up matching existing voice and depth.

**The user's input:** `$ARGUMENTS`

## Step 1: Classify

Read the input and determine what kind of note this is:

<!-- CUSTOMIZE: Define your note types and their target files. Examples:

| Type | File | Signal |
|------|------|--------|
| **Idea** | `IDEA-NOTES.md` | A new feature concept, product direction, or design exploration. Something that doesn't exist yet and needs thinking through. |
| **Tweak** | `FIX-TWEAK-NOTES.md` | A UI inconsistency, polish item, small behavioral fix, or something that works but doesn't feel right. Not a bug — a refinement. |
| **Bug** | `BUG-TRACKER.md` | A reproducible defect in existing functionality. |

Adjust types and files to match your project's note-keeping conventions.
-->

If genuinely ambiguous, ask with `AskUserQuestion`: "This could be [type A] or [type B] — [brief reasoning]. Which feels right?"

## Step 2: Read the target file

Read the file (or at least last entry + section headers) to learn current numbering, voice/depth, and whether this note extends an existing entry.

**If it extends an existing entry**, update it rather than creating a new one. Tell the user what you did.

## Step 3: Write the note

**Match the existing voice** — read a few entries to calibrate.

The user may ramble or contradict themselves. Extract the *intention*. Preserve their good examples; improve or replace bad ones.

<!-- CUSTOMIZE: Define formatting conventions per note type. Examples:

**For idea notes:**
- Section heading: `## N. [Title — Evocative but Clear]`
- Open with the core concept — what is this and why does it matter
- Explore the design space: what it touches, what it doesn't do, what to be careful about
- Include implementation notes where relevant
- Note relationships to other notes

**For tweak notes:**
- Section heading: `## N. [Brief Description]`
- **The issue:** — what's wrong or could be better
- **Where it applies:** — specific files/surfaces affected
- **What it should feel like:** — the target state
- Tweaks are shorter than ideas. Don't over-elaborate.
-->

**Separator:** Add `---` before the new entry (matching existing file structure).

## Step 4: Confirm

Tell the user what you added, where, and the title — 2-3 sentences. If you updated an existing entry, say so. Don't recap the note's contents.

## Step 5: Offer audit

Ask via `AskUserQuestion`:

> "Want me to audit the current notes? I'll check for stale entries, drift from what's shipped, archive candidates, and items overtaken by events."

**If yes**, read all note files in full, then assess each entry for:
- **Stale** — already shipped or no longer relevant (check codebase if unsure)
- **Drifted** — implementation diverged from the note
- **Archive candidate** — resolved and ready to move
- **Needs update** — still valid but outdated details
- **Superseded** — later note or shipped feature made it redundant

Present findings grouped by status. Only flag what needs attention; if clean, say so. Get approval before archiving/updating, then renumber and fix cross-references.

## Step 6: Commit

After the note (and audit, if applicable), commit only the files you changed. Do not stage unrelated working-tree changes.

```
note: [Brief description of what was added/updated]
```
