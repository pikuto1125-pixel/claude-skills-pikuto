---
name: daily-content-rotator
description: Rotate daily content drafts for multi-channel posting (microblog / newsletter / blog / video script). Reads a content idea bank, picks next-day themes by day-of-week + recency dedup, writes drafts to a configurable target. Triggers automatically the night before, after the last post of the current day has fired.
---

# Daily Content Rotator

Operators running multiple posting channels (microblog 3x/day, daily blog, weekly newsletter) lose hours on the night-before rotation work: picking themes, avoiding repeats, drafting first cuts. This skill automates the rotation while leaving final wording to a human review pass in the morning.

## Activation

Invoked by the operator (or a scheduled task) at one of:

- "rotate tomorrow"
- "次の日のローテーション"
- "tomorrow content"
- "明日のコンテンツ"

The skill checks one timing precondition before doing work (see Pre-flight).

## Pre-flight: post-fire timing check

Most multi-channel rotations have a "last post of the day" that fires near end-of-day. If the rotator runs **before** that last post fires and rewrites the schedule, the last post may render with wrong content.

Read the current daily schedule file (path configurable below) to find the latest scheduled post time for today. If the current time is before that time **plus a 30-minute safety buffer**, abort with the message:

> "Today's last scheduled post hasn't fired yet (scheduled HH:MM, current HH:MM). Skipping rotation to avoid clobbering — rerun after the last post fires."

This avoids the classic "wrote tomorrow's content too early and overwrote today's" failure mode.

## Pre-flight: idempotency check

Read the daily schedule file. If the header date is already tomorrow (i.e., a previous run already rotated), abort:

> "Daily schedule already shows tomorrow's date. Already rotated. Skipping."

## Configuration

The skill reads a config file, default path: `./content-rotator.config.json`.

```json
{
  "schedule_file": "./content/daily-schedule.md",
  "schedule_backup": "./backup/daily-schedule.md",
  "idea_bank": "./content/idea-bank.md",
  "guardrails": "./content/guardrails.md",
  "channels": [
    { "name": "morning_micro", "time": "08:00", "char_limit": 140, "themes": ["tasks", "industry-take"] },
    { "name": "midday_micro",  "time": "12:30", "char_limit": 140, "themes": ["everyday", "relatable"] },
    { "name": "evening_micro", "time": "21:00", "char_limit": 140, "themes": ["cta", "reflection"], "cta_per_week_max": 2 }
  ],
  "recency_window_days": 5,
  "post_fire_safety_buffer_minutes": 30,
  "timezone": "Asia/Tokyo"
}
```

## Rotation steps

1. Read the schedule file's current state (channels and themes)
2. For each channel, pick a theme:
   - Avoid any theme used in the last `recency_window_days` for the same channel
   - Respect day-of-week conventions (e.g., Friday's morning channel = "weekend prep" if your guardrails say so)
   - For channels with `cta_per_week_max`, count CTAs in the last 7 days from the schedule history; if at limit, pick a non-CTA theme
3. Update the schedule file:
   - Header date → tomorrow (with day-of-week)
   - Each channel section → new theme
   - Append "today's fires" to the running log section
   - Append a one-line entry to the changelog footer
4. Mirror the file to `schedule_backup` (atomic-ish: write backup first, then overwrite primary)
5. Verify the primary write by re-reading and checking the header date

## Failure recovery

If primary write succeeds but verification fails (e.g., file system sync delay on cloud storage):

- Retry verification up to 3 times with 10-second intervals
- If all 3 fail, leave the local primary file as-is, write a report to `./logs/rotator-YYYY-MM-DD.log`, and surface a notification to the operator (configurable channel: stdout / file / external webhook)

The local file is updated either way; the failure mode is "verification couldn't confirm cloud sync," which the operator can resolve manually if the cloud sync recovers slowly.

## Theme dedup logic

The dedup uses the last `recency_window_days` of theme history per channel. Pseudocode:

```
for channel in channels:
  recent_themes = []
  for day in past N days:
    recent_themes.append(schedule_history[day][channel.name].theme)
  available_themes = channel.themes - set(recent_themes)
  pick from available_themes (random or guardrail-driven)
```

If `available_themes` is empty (your channel themes are too narrow for the window), the skill falls back to picking the **least recently used** theme.

## Customization points

- **Channel definitions**: add / remove channels by editing the config
- **Theme lists**: per-channel theme dictionaries can be expanded
- **Guardrails file**: an optional Markdown file with brand rules (banned phrases, tone notes); the skill includes its content as a context hint when generating drafts
- **Cloud sync target**: if the schedule lives on cloud-synced storage, the backup path can point at a separate local-only mirror for forensic recovery

## Output

A status line for the operator:

```
✅ Tomorrow's schedule rotated.
  - Date: YYYY-MM-DD (Day-of-week)
  - morning_micro: <theme>
  - midday_micro: <theme>
  - evening_micro: <theme> (CTA: yes/no, this week's CTA count: N/2)
  - Backup written: <path>
  - Cloud sync verified: <yes/no, retries used: M>
```

## Why this exists

This skill was extracted from a production multi-channel posting workflow. The two failure modes it specifically prevents:

1. **Premature rotation**: writing tomorrow's content before today's last scheduled post has fired, causing the live post to render with wrong context
2. **Repetition fatigue**: theme repeat within a 5-day window across multiple channels, which signals "auto-generated" to followers and reduces engagement

Both failure modes were observed in production before this skill existed, with downstream cost (deleted posts, audience confusion, hand-rolled rotation work eating evening hours).

## Related

- Companion skill `session-handoff-builder` for end-of-session context preservation
- Pattern reference: "Inbox Pattern" for post deduplication across multiple senders
