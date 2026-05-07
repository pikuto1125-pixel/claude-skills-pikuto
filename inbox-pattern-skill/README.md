# inbox-pattern-skill

Structural deduplication for systems with multiple senders. Append-only file as a shared inbox; every sender checks before acting. Crash-safe, restart-safe, and structurally prevents the "duplicate send" class of bugs.

## What problem this solves

Solo operators running automation systems hit the same wall repeatedly: a manual click, a scheduled job, a retry, and a backup script all fire the same outgoing call. The fix isn't tighter coordination in code — code-level locks die when processes crash. The fix is a shared inbox file that all senders read before they act.

This skill encodes the pattern. Drop it in front of any side-effecting call and duplicate sends become structurally impossible.

## Why a file, not a database

For one-person operations, an SQLite or Redis dependency is operational surface you don't need. A daily-rotated text file with append-only semantics:

- Survives process crashes (the entry is on disk before the side effect runs).
- Survives OS restarts (file persists; lock would have been lost).
- Is auditable with `cat`, `grep`, or any text editor.
- Has no daemon to keep alive.

The cost is that multi-machine deployments need a row-locked database instead. For a single-laptop solo operator (the target audience for this skill), the file is the right answer.

## When NOT to use this skill

- Multi-machine clusters → use a database with `INSERT IF NOT EXISTS`.
- High-frequency sends (>10/sec) → file I/O latency adds up; batch instead.
- Very large key spaces (>1M entries/day) → grep scan slows; index the file or use a key-value store.

For most solo automation, none of these apply.

## Production track record

Used in a daily microblog pipeline (3 channels, 9 sends/day), a proposal pipeline (5-15 sends/day), and a customer reply pipeline (variable). Origin incident: a 4x duplicate proposal send in April 2026 caused by a manual + scheduler + retry + backup race. Fix held since.

## License

MIT. See `LICENSE`.

## Related

- `daily-content-rotator` — uses inbox-pattern internally for the post-fire dedup
- `session-handoff-builder` — independent skill, ships in the same bundle

## How to install

This skill follows the Claude Code skills convention. With Claude Code 0.x+:

```
~/.claude/skills/inbox-pattern-skill/SKILL.md
```

Then invoke from the Claude Code session with phrases like "send through inbox" or "dedupe send".

For the JavaScript reference implementation that the skill expects to find on disk, see the parent repository's reference module.

## Origin context

Built by pikuto (AI Hack Lab) as part of the Arena Blueprint. The full pattern documentation, design rationale, and 9 other companion modules are in:

- Notion: https://shared-cut-385.notion.site/arena_blueprint_complete_v1-358347a29814808b81c1efe8d81cebe7
- Site: https://ai-hack-lab.com/products/arena-blueprint
