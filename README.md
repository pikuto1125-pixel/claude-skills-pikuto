# claude-skills-pikuto

[Claude Code](https://docs.claude.com/en/docs/claude-code/) Skills extracted from a solo AI automation operation. Patterns that work for a one-person business, not the kind that need a platform team to operate.

Built and maintained by **pikuto** ([@ue6654](https://x.com/ue6654)) — running [AI Hack Lab](https://ai-hack-lab.com/).

## Skills in this pack

| Skill | What it does | Status |
|-------|--------------|--------|
| [`session-handoff-builder`](./session-handoff-builder/) | Ends long Claude Code sessions cleanly. Updates task files, writes a structured handoff, leaves the next session ready to resume cold. | Stable |
| [`daily-content-rotator`](./daily-content-rotator/) | Rotates tomorrow's content drafts across multi-channel posting (microblog / blog / newsletter). Theme-by-weekday + recency dedup + post-fire safety. | Stable |
| `memory-audit-helper` | Periodic memory directory hygiene. Detects duplicates, stale entries, orphaned files. | Planned |
| [`inbox-pattern-skill`](./inbox-pattern-skill/) | Structural deduplication for systems with multiple senders. File-based, durable, no DB. | Stable |
| `polling-watcher-skill` | Lock-based single-instance scheduler pattern with dual triggers (logon + interval). | Planned |
| `eight-layer-validator-skill` | Multi-layer validator pipeline with single-responsibility per layer. Cheap regex first, expensive LLM judge later. | Planned |

## Why these patterns

These skills were extracted from a system that runs daily in production for a solo operator: sales pipeline, customer reply, content distribution, and self-evolution feedback loops, all running across school hours and part-time shifts.

The bias of the pack is toward **operational durability**. Patterns survive crashes, OS restarts, cloud-sync conflicts, schedule mismatches, and the occasional 3-day silent failure that no one notices until Monday. Most of these patterns came from incidents — they're the lessons paid for in lost evenings.

## Common installation

Each skill ships as a single `SKILL.md` plus a README and license. Two installation paths:

### Option A: copy one skill

```bash
mkdir -p ~/.claude/skills/<skill-name>
curl -o ~/.claude/skills/<skill-name>/SKILL.md \
  https://raw.githubusercontent.com/<your-username>/claude-skills-pikuto/main/<skill-name>/SKILL.md
```

### Option B: clone the whole pack

```bash
git clone https://github.com/<your-username>/claude-skills-pikuto.git
cp -r claude-skills-pikuto/* ~/.claude/skills/
```

Each skill's own README has its specific trigger phrases, configuration knobs, and customization points.

## Compatibility

- Claude Code (CLI, IDE extensions, web app at `claude.ai/code`)
- Tested with `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5`
- Skills format follows [Claude Code Skills spec](https://docs.claude.com/en/docs/claude-code/skills)

## The full architecture

These skills are extracted patterns from a larger documented system, **Arena Blueprint** — the complete documentation of a 14-phase AI automation pipeline distilled into 10 transferable modules.

If the skills are useful and you want the full architecture (sales pipeline, customer reply, content distribution, self-evolution mechanism, anti-drift guard, inbox pattern, generator+validator principle, scheduling+health-check, mobile remote operation), Arena Blueprint launches in May 2026 via [Polar](https://polar.sh/).

Pre-launch info: [ai-hack-lab.com/en/products/arena-blueprint](https://ai-hack-lab.com/en/products/arena-blueprint)

## Contributing

These skills are extracted from real workflows. If you adapt one to your use case and find a generalization that fits more cases without losing precision, a PR is welcome. Bugs, edge cases, missing failure modes — all valid.

The deliberate non-goal is feature creep. A skill should do one thing reliably, not 12 things with config flags.

## License

MIT — see each skill's `LICENSE` file.

## About

A 20-year-old college student building AI automation tools with no prior coding experience as of summer 2025. The skills here are real production patterns, not tutorial snippets.

- Twitter / X: [@ue6654](https://x.com/ue6654)
- Site: [ai-hack-lab.com](https://ai-hack-lab.com/)
- English Edition: [ai-hack-lab.com/en/](https://ai-hack-lab.com/en/)

If any of this is useful to your project, the patterns are yours to take and adapt.
