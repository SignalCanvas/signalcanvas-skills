---
name: context-handoff
description: >
  Writes a HANDOFF.md file capturing the active task, exact stopping point, key decisions,
  and next steps so the user can clear context and resume cleanly in a fresh session.
  Use this skill whenever the user says the context is full, wants to "write down where
  we left off", "save progress before clearing", "make a handoff note", or anything like
  "I need to clear context" or "write this down so we can continue later". Also trigger
  when the conversation is visibly long and the user asks to save or capture the current
  state for resumption. This is NOT a general summary — it's a forward-focused brief
  written for a fresh Claude instance that has never seen this conversation.
---

# Context Handoff

Write a `HANDOFF.md` in the active project's git root so a fresh Claude instance can read
it and immediately continue the work — no conversation history, no clarifying questions needed.

## Choosing the filename

1. Check whether `HANDOFF.md` already exists in the target directory.
2. If it does NOT exist: write `HANDOFF.md`.
3. If it DOES exist: write `HANDOFF-YYYY-MM-DD-HH-MM.md` using the current date and time
   (e.g., `HANDOFF-2026-06-12-14-35.md`). Do not overwrite the existing file.

## Where to write

Write to the git root of the repo being actively worked on. In a monorepo, use the root
of the specific sub-repo (e.g., `SignalCanvasFrontend/`) unless the work spans multiple
repos — in that case use the monorepo root.

## What goes in the file

Write exactly these sections, in this order. Keep each one dense and specific — a fresh
Claude is smart but has zero session context.

### Active Task
One paragraph. What were we literally working on right now? Name the feature, bug, refactor,
or decision. Be concrete enough that the fresh Claude doesn't have to guess the scope.

### Where We Left Off
The exact stopping point: which files, which functions or line numbers, what was partially
done, what error message or decision was pending. Include paths and line numbers verbatim.
If something was mid-edit, say so.

### Key Decisions
A short bullet list of decisions made during this session that constrain what comes next.
Only include things a fresh Claude needs to avoid undoing work or going the wrong direction.
Skip anything that predates this session or that's obvious from the code.

### Next Steps
A numbered list of the immediate next actions, in order. Each step should be concrete enough
to execute without asking a follow-up question. Start with what to do first.

### Critical Context
Any constraints, environment quirks, or gotchas discovered during this session that are NOT
derivable by reading the code. Keep this ruthlessly short — if it's in the code or docs, omit it.

## What NOT to include

- How we got here, historical background, or work completed long before the handoff
- Anything a fresh Claude can derive by reading the current files
- Long prose — bullets and numbered lists over paragraphs
- Repetition of things already in CLAUDE.md or project docs

## After writing

Tell the user the file path. Suggest they clear context and start fresh with something like:
> "Read [filename] and pick up from there."

## Length target

Under 400 words total. Dense and specific beats thorough and vague.
