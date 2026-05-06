---
name: session-handoff-builder
description: End-of-session handoff builder for Claude Code. Triggers on "session end" / "handoff" / "本当にお休み" and produces a structured HANDOFF file so the next session (or tomorrow's you) can pick up cold without losing context.
---

# Session Handoff Builder

A Claude Code Skill that closes long-running sessions cleanly. It updates task tracking files, snapshots the work-in-progress, and writes a structured handoff document so the next session can resume with zero context loss.

Built for solo developers and indie hackers who run multi-hour sessions and switch context daily (other projects, school, sleep).

## Trigger phrases

The skill activates when the user input contains any of:

- `session end` / `end session`
- `handoff` / `引継ぎ`
- `本当にお休み` (literal "really good night" — explicit Japanese close phrase)

It deliberately does **not** trigger on softer phrases like `お休み` / `bye` / `bed` to avoid burning credits on premature wrap-ups.

## Execution steps

### 1. Finish pending writes

- Complete any in-flight Edit/Write tool calls.
- Verify there is no half-saved state (TaskUpdate, partial file rewrites).
- Drain background tasks: read their output and either finalize or stop them.

### 2. Refresh the active task file

Look for one of these (in order, first match wins):

- `tasks/current.md`
- `Tasks/current_tasks.md`
- `TODO.md`
- `docs/tasks.md`

If found:

- Mark today's completed items with `[x]`.
- Update status of in-progress items.
- Bump the file header's "last updated" timestamp to now.
- Refresh any "next session top priority" section.

### 3. Roll up to the completed log

Look for one of these:

- `tasks/completed.md`
- `Tasks/completed.md`
- `docs/changelog.md`

Append a new section dated today:

```markdown
## YYYY-MM-DD (Weekday)

### <Category 1, e.g. Feature work>
- <task name> — <one-line outcome with file path>

### <Category 2, e.g. Ops>
- ...
```

Always include numeric results (counts, amounts, % deltas) when relevant, and always include file paths so future readers can navigate.

### 4. Memory hygiene (if applicable)

If the project uses Claude Code's memory directory (`~/.claude/projects/.../memory/` or similar):

- Check that today's new memory files appear in `MEMORY.md`'s index.
- Update or remove memories whose facts have changed.
- Merge near-duplicates.

Skip this step if no memory directory is in use — do not invent one.

### 5. Write the HANDOFF document

Path:

```
reports/HANDOFF_YYYYMMDD_<morning|afternoon|evening|night>.md
```

(Adjust to project conventions — `docs/handoffs/`, `notes/`, etc. — if those already exist.)

Required sections:

1. **Core outcomes** (chronological, importance-ordered, with file paths and line numbers when applicable)
2. **Top 3-5 priorities for the next session** (concrete, ready to act on)
3. **Known blockers** (what would stall the next session if not resolved)
4. **Session stats** (duration, commits made, files touched)
5. **Memory additions** (if memory hygiene step applied)

The HANDOFF is the cold-start document. Write it for someone who has not seen this conversation.

### 6. Closing line

End with:

> Good work today. Next session: open the latest HANDOFF first.

(Translate to the user's working language if appropriate.)

## Behavior notes

- If the user says "actually one more thing" / "もうひと仕事" after triggering, abort the protocol and return to normal mode.
- If the trigger fires twice in one session, run the full protocol the first time only. The second time, write a "supplemental HANDOFF" that only contains delta items.
- Do **not** auto-commit code as part of this protocol unless the user explicitly says to. Some users have pre-commit checks they want to run manually.

## Customization

Edit this file to adjust:

- Trigger phrases (e.g. add team-specific keywords)
- Task file paths (default search order shown above)
- HANDOFF location and naming format
- Whether to attempt memory hygiene (skip in projects without a memory dir)

## Why this skill exists

Long Claude Code sessions accumulate tribal knowledge that lives only in your scrollback. Re-deriving that context tomorrow morning is the most expensive part of multi-day work. This skill standardizes the wrap-up so the cost stays low.
