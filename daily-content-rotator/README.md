# daily-content-rotator

A [Claude Code](https://docs.claude.com/en/docs/claude-code/) Skill that rotates tomorrow's content drafts across your posting channels — microblog, daily blog, weekly newsletter — without you having to do the rotation work each evening.

Built for operators running multi-channel content with limited evening time.

## Why

Solo operators with multi-channel posting routines lose 20-40 minutes every night picking themes, scanning recent posts to avoid repetition, and writing first-cut drafts. Skipping a night breaks the rhythm; doing it half-asleep ships sloppy work.

This skill does the rotation deterministically and surfaces drafts for a quick morning review pass.

## What it does

When you say `rotate tomorrow`, `次の日のローテーション`, `tomorrow content`, or `明日のコンテンツ`:

1. Confirms the current day's last scheduled post has already fired (refuses to rotate prematurely)
2. Confirms tomorrow's slot isn't already filled (idempotent — re-running is safe)
3. Picks themes per channel based on day-of-week conventions and recent-N-day deduplication
4. Updates the schedule file with tomorrow's date and per-channel themes
5. Mirrors to a backup path
6. Verifies the write (with cloud-sync retry tolerance)
7. Logs a one-line entry to the changelog footer

## Installation

### Method 1: Direct copy

```bash
mkdir -p ~/.claude/skills/daily-content-rotator
curl -o ~/.claude/skills/daily-content-rotator/SKILL.md \
  https://raw.githubusercontent.com/<your-username>/claude-skills-pikuto/main/daily-content-rotator/SKILL.md
```

### Method 2: Clone the whole pack

```bash
git clone https://github.com/<your-username>/claude-skills-pikuto.git
cp -r claude-skills-pikuto/daily-content-rotator ~/.claude/skills/
```

### Method 3: Plugin marketplace

Listing on [claudemarketplaces.com](https://claudemarketplaces.com/) — TBD.

## Trigger phrases

| Trigger | Language |
|---------|----------|
| `rotate tomorrow`, `tomorrow content` | English |
| `次の日のローテーション`, `明日のコンテンツ` | Japanese |

The skill deliberately ignores generic phrases (`tomorrow`, `next day`) to avoid premature firing.

## Configuration

The skill reads a config file (`./content-rotator.config.json` by default). See `SKILL.md` for the full schema.

Key knobs:

- `channels` — array of channel definitions with name, time, character limit, theme list
- `recency_window_days` — how far back to look for theme deduplication (default 5)
- `post_fire_safety_buffer_minutes` — how long after the last scheduled post to wait before allowing rotation (default 30)
- `cta_per_week_max` per channel — caps CTA-style themes (e.g., 2 CTAs per week)

## Compatibility

- Claude Code (CLI, IDE extensions, web app at `claude.ai/code`)
- Tested with `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5`
- Skill format follows [Claude Code Skills spec](https://docs.claude.com/en/docs/claude-code/skills)

## What it does NOT do

- It does not auto-publish posts. Drafts go to your schedule file; publishing is your separate workflow's job.
- It does not call external APIs. The skill reads/writes local files only.
- It does not invent themes. Themes come from your config — if your config lists 3 themes per channel and you want a 5-day window, expand the config first.

## Author

Built by **pikuto** ([@ue6654](https://x.com/ue6654)) — running [AI Hack Lab](https://ai-hack-lab.com/).

A 20-year-old college student building AI automation tools with no prior coding experience as of summer 2025. This skill is one pattern extracted from that production workflow.

## License

MIT — see `LICENSE`.

## Other skills in the pack

- `session-handoff-builder` — end-of-session context preservation
- `memory-audit-helper` — periodic memory directory hygiene (planned)
- `inbox-pattern-skill` — structural deduplication for multi-sender systems (planned)
