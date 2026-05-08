 # claude-skills-pikuto

  > Production-tested Claude Code skills, extracted from a 9-month solo AI automation system.
  >
  > Part of [Arena Blueprint](https://ai-hack-lab.com/products/arena-blueprint/) — a 10-module documentation of patterns that survived contact with reality.

  [![Website](https://img.shields.io/badge/Website-ai--hack--lab.com-10b981)](https://ai-hack-lab.com/)
  [![Blueprint](https://img.shields.io/badge/Arena_Blueprint-Lite_$39-emerald)](https://ai-hack-lab.com/en/products/arena-blueprint/)
  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
  [![Twitter](https://img.shields.io/badge/Twitter-@ue6654-1da1f2)](https://x.com/ue6654)

  ## What this is

  A small, opinionated collection of Claude Code skills that I (pikuto, a 20-year-old solo founder) use daily in production. They're extracted from a larger system documented in **Arena Blueprint**.

  These are battle-tested, not theoretical: the failure logs that shaped each skill are documented in the corresponding Blueprint module.

  ## Skills

  ### Stable

  | Skill | Description | Module |
  |---|---|---|
  | `session-handoff-builder` | Generate a structured handoff document at session-end. Captures state, files touched, decisions, next priorities. | Module 5: Self-Evolution |
  | `daily-content-rotator` | Multi-channel daily content rotation engine. Per-day idea selection from a shared bank. | Module 3: Content Distribution |

  ### Planned (releases roll out post-launch)

  - `inbox-pattern-skill` — file-based dedup that survives crashes (Module 7)
  - `polling-watcher-skill` — lock-based single-instance scheduler (Modules 1, 9)
  - `eight-layer-validator-skill` — pipeline orchestrator (Module 4)
  - `generator-validator-pair-skill` — template pair structure (Module 8)
  - `polling-daemon-skill` — daemon with heartbeat + sandbox (Module 10)
  - `anti-drift-guard-skill` — rule-set template + sprint binding (Module 6)

  ## Installation

  Each skill is a standalone Claude Code skill. Drop the folder into `~/.claude/skills/<skill-name>/` and Claude Code picks it up automatically.

  ```bash
  git clone https://github.com/pikuto1125-pixel/claude-skills-pikuto.git
  cp -r claude-skills-pikuto/<skill-name> ~/.claude/skills/
  ```

  ## Read the full Blueprint

  The patterns above are documented in detail at:

  - 🇯🇵 **[Arena Blueprint (JA)](https://ai-hack-lab.com/products/arena-blueprint/)** — 10 モジュール完全版・失敗ログ + 教訓 + Validator チェックリスト
  - 🇺🇸 **[Arena Blueprint (EN)](https://ai-hack-lab.com/en/products/arena-blueprint/)** — Full architecture, design decisions, and adaptation guides
  - 📖 **[Read it free on Notion](https://shared-cut-385.notion.site/arena_blueprint_complete_v1-358347a29814808b81c1efe8d81cebe7)** — All 10 modules readable for free
  - 🆓 **[5-min workflow diagnosis (free)](https://ai-hack-lab.com/diagnosis/)** — Find the right pattern for your project

  Tiers:
  - **Lite ($39)** — Notion Blueprint + this Skills Bundle
  - **Pro ($99)** — Lite + lifetime updates + private Q&A
  - **Enterprise ($199)** — Pro + 60-min 1:1 adaptation review (Zoom)

  Launching **Mon May 12, 2026 · 8 AM JST**.

  ## Articles by pikuto

  - [Qiita: 8 層 Validator（全体像）](https://qiita.com/pikuto1125-pixel/items/1655c87495424d4ba447)
  - [Zenn: Validator 設計の 3 つの落とし穴](https://zenn.dev/pikuto/articles/ai-validator-3-pitfalls)
  - [AI Hack Lab Daily Digest](https://ai-hack-lab.com/insights/) — daily 06:00 JST

  ## Contact

  - 🐦 [@ue6654](https://x.com/ue6654) on X
  - 💬 [Contact form](https://ai-hack-lab.com/en/contact/) (24h reply)

  ## License

  MIT. See [LICENSE](LICENSE).

  — pikuto, AI Hack Lab
