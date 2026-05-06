# session-handoff-builder

A [Claude Code](https://docs.claude.com/en/docs/claude-code/) Skill that ends long-running sessions cleanly. Updates your task files, writes a structured handoff document, and leaves the next session ready to resume cold.

Built for solo developers, indie hackers, and anyone running multi-hour Claude Code sessions across days.

## Why

Long sessions accumulate tribal knowledge that lives only in your scrollback. Re-deriving it tomorrow morning is the most expensive part of multi-day work — you pay it as either lost time or worse decisions.

This skill standardizes the wrap-up so the cost stays low.

## What it does

When you say `session end`, `handoff`, or `本当にお休み`:

1. Finishes any pending Edit/Write operations
2. Marks today's completed items in `tasks/current.md` (or equivalent)
3. Appends today's outcomes to `tasks/completed.md` with categories and file paths
4. Optionally tidies up your memory directory
5. Writes `reports/HANDOFF_YYYYMMDD_<period>.md` containing:
   - Core outcomes (chronological, with file paths)
   - Top 3-5 priorities for the next session
   - Known blockers
   - Session stats
6. Closes with a one-line "open the HANDOFF first" reminder

## Installation

### Method 1: Direct copy

```bash
mkdir -p ~/.claude/skills/session-handoff-builder
curl -o ~/.claude/skills/session-handoff-builder/SKILL.md \
  https://raw.githubusercontent.com/pikuto1125-pixel/claude-skills-pikuto/main/session-handoff-builder/SKILL.md
```

### Method 2: Clone the whole pack

```bash
git clone https://github.com/pikuto1125-pixel/claude-skills-pikuto.git
cp -r claude-skills-pikuto/session-handoff-builder ~/.claude/skills/
```

### Method 3: Plugin marketplace

Listing on [claudemarketplaces.com](https://claudemarketplaces.com/) — TBD.

## Trigger phrases

| Trigger | Language |
|---------|----------|
| `session end`, `end session` | English |
| `handoff` | English |
| `引継ぎ` | Japanese |
| `本当にお休み` | Japanese (explicit close phrase) |

The skill deliberately ignores softer phrases (`bye`, `お休み`, `bed`) to avoid premature wrap-ups eating credits.

## Customization

Open `SKILL.md` and edit:

- **Trigger phrases** — add team-specific keywords
- **Task file paths** — the skill searches `tasks/current.md`, `Tasks/current_tasks.md`, `TODO.md`, `docs/tasks.md` in order. Add yours.
- **HANDOFF path** — default is `reports/HANDOFF_*.md`. Change to match your repo (`docs/handoffs/`, etc.)
- **Memory hygiene** — disable the step entirely if your project has no memory dir.

## Compatibility

- Claude Code (CLI, IDE extensions, web app at `claude.ai/code`).
- Tested with `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5`.
- Skill format follows [Claude Code Skills spec](https://docs.claude.com/en/docs/claude-code/skills).

## What it does NOT do

- It does not auto-commit your code. Some workflows have pre-commit checks the user wants to run manually.
- It does not push to remotes.
- It does not delete branches or files. Reversibility first.

## Author

Built by **pikuto** ([@ue6654](https://x.com/ue6654)) — running [AI Hack Lab](https://ai-hack-lab.com/).

A 20-year-old college student building AI automation tools with no prior coding experience as of summer 2025. This skill is one pattern extracted from that production workflow.

## License

MIT — see `LICENSE`.

## Other skills in the pack

- `daily-content-rotator` — auto-rotates tomorrow's content draft from today's, with theme-by-weekday selection (coming soon)
- `memory-audit-helper` — periodic memory directory hygiene (planned)
