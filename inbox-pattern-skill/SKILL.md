---
name: inbox-pattern-skill
description: Structurally prevent duplicate side effects (message sends, API calls, file commits) when multiple senders touch the same channel. Uses an append-only file as a shared inbox; every sender checks and appends before acting. Crash-safe, restart-safe, concurrent-safe enough for single-machine deployments.
---

# Inbox Pattern (Structural Deduplication)

When a system has multiple senders — manual, scheduled, retry, backup — duplicate side effects are inevitable without coordination. Locking in code is fragile across crashes and OS restarts. This skill installs a **file-based shared inbox** that makes duplicates structurally impossible.

## Activation

Invoked by the operator (or a generator agent) when a side effect is about to fire:

- "send through inbox"
- "dedupe send"
- "inbox pattern apply"
- "重複防止インボックス"

The skill wraps the side-effect call in a check-append-act sequence.

## Core sequence

For each side effect attempt:

```
1. Compute key = hash_function(payload, recipient, scope)
2. Read inbox file (today's + yesterday's)
3. If key exists in either → skip, log "already sent"
4. If key absent → append key + status="pending" + timestamp
5. Execute side effect
6. Update inbox entry: status="ok" or status="fail:<reason>"
```

The append happens **before** the side effect. If the side effect crashes mid-flight, the entry remains as `pending` and the operator sees it on the next inbox audit.

## Key composition

The dedup key MUST include enough context to distinguish unrelated calls but not so much that semantically-identical retries miss the dedup.

Recommended composition:

```
key = sha256(
  recipient_id + "\t" +
  content_fingerprint + "\t" +
  scope_unit
)
```

- **recipient_id**: the destination identity (email, user_id, channel_id). Different recipients should send separately.
- **content_fingerprint**: a stable hash of the payload body. Whitespace normalize first.
- **scope_unit**: the dedup window — usually `YYYY-MM-DD` for daily rotation. Cross-day retries cross the boundary explicitly.

## File layout

```
.state/inbox/
  inbox-2026-05-06.txt    # today's append-only
  inbox-2026-05-05.txt    # yesterday (kept for cross-day check)
  inbox-2026-05-04.txt    # day-before (auto-archived after N days)
  ...
```

Each line is a tab-separated record:

```
<key>\t<status>\t<recipient_id>\t<scope_unit>\t<iso_timestamp>\t<note>
```

- Append-only writes. Never edit prior lines.
- One file per scope_unit. Daily rotation by default.
- Yesterday's file is read on cross-day retries.

## Read-before-act semantics

When a sender calls into the skill:

1. Compute the key.
2. Read today's inbox file. Scan for the key.
3. If found and status is `ok` → skip, return `{skipped: true, reason: "already_processed"}`.
4. If found and status is `pending` and >5min old → log warning "stale pending — possible crashed sender", proceed at operator discretion.
5. If found and status is `fail:*` → return failed status; let caller decide whether to retry.
6. If not found → append `pending`, run side effect, append final status.

## What this prevents

- **Manual + scheduled double-fire**: Operator clicks send, then scheduler also fires for the same item. Both see the entry → only one proceeds.
- **Retry storms**: Failed-job retry runs before the original entry finalized. Pending entry blocks the storm.
- **Cross-machine duplicate** (single-machine version): On a single machine the file system handles ordering. For multi-machine, replace the file with a database row + `INSERT IF NOT EXISTS`.
- **Replay of stale logs**: Importing an old log re-fires sends. Inbox entries from prior days block the replay.

## What this does NOT prevent

- **Race within milliseconds**: Two writers can both pass the check-then-append sequence before either appends. For single-machine, the race window is small (~1ms file-lock); for multi-machine, use a database with row-level lock instead.
- **Different keys for the same intent**: If the key composition changes, prior entries don't match anymore. **Version your key composition function**.
- **Wrong scope_unit**: A scope of "minute" lets the same content fire 60x/hour. Pick the right unit.

## Forbidden-edit zone

The inbox files are append-only. The skill MUST refuse to:

- Edit prior lines (only append new lines).
- Delete entries within the current `scope_unit`.
- Roll over the day boundary mid-write.

Old files (>N days, configurable) can be archived but not deleted.

## Operator audit hooks

The skill exposes three commands for inspection:

- `inbox-status` → counts of ok / pending / fail / skipped today
- `inbox-pending` → list pending entries older than 5min (likely crashed senders)
- `inbox-replay <key>` → re-execute a specific failed entry (with explicit operator confirmation)

## Configuration

Required:
- `inbox_dir`: where to store the daily files. Recommend a state dir excluded from version control.
- `key_composition_version`: integer. Increment when the key function changes.
- `archive_after_days`: when to archive old files (default 30).

Optional:
- `cross_day_check`: whether to check yesterday's file (default true).
- `stale_pending_minutes`: when a `pending` entry warrants a warning (default 5).

## Why the key versioning matters

If you change what fields go into the key — say, you start including the channel name when before you only used recipient — old entries no longer match. The same content sends again. Version the function explicitly:

```
key = sha256(version + "\t" + recipient + "\t" + content + "\t" + scope)
```

When you bump the version, the new entries don't collide with old ones, but the old ones also don't dedupe new sends. Live with that briefly during migration; document the change.

## Pairs well with

- `polling-watcher-skill` — when a watcher polls for state and fires on detection, inbox prevents the same detection firing twice if the watcher restarts mid-poll.
- `generator-validator-pair-skill` — validator's "approved → side effect" step uses inbox.
- `eight-layer-validator-skill` — Layer 7 (provenance log) and inbox can share storage.

## Reference implementation

A working JavaScript implementation runs in production handling daily microblog and proposal-send pipelines. See:

- `https://github.com/pikuto1125-pixel/claude-skills-pikuto/tree/main/inbox-pattern-skill`

The reference includes:

- Single-file inbox class with daily rotation
- Cross-day check
- Operator audit commands
- Archive helper

## Origin

This pattern emerged from a kazu315 incident in 2026-04: a manual submit + a scheduled retry + a backup script all fired the same proposal four times to the same client. The damage was reputational, not financial, but the fix had to be structural — pre-condition checks in code repeatedly missed edge cases.

The append-only file approach was chosen over locks (which die with the process) and over a database (which adds operational surface for a one-person operation). It's been running unattended since then.
