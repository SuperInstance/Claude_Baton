# Claude_Baton 🏃‍♂️

**Infinite context for Claude Code through automatic generational baton passing.**

Claude Code's 1M-token context (Opus 4.6) and native compaction are powerful — but long-running projects still hit the wall: slow compaction loops, lost nuance, and agents that "forget" across sessions.  

Claude_Baton fixes this at the root. When context hits 48 %, it quietly spawns an Onboarder subagent that builds a complete handover package *while you keep shipping*. At 82 % it launches a fresh Young Agent (native subagent/Agent Team), overlaps in A2A advisory mode, then retires the Old Agent gracefully.  

Result: functionally infinite context, perfect audit trail, reusable skills, and a living project memory that grows stronger with every generation.

100 % native Claude Code plugin — uses only official hooks, subagents, MCP, and Agent Teams. Zero external wrappers for the MVP.

[![Claude Code Marketplace](https://img.shields.io/badge/Install%20from%20Marketplace-blue)](https://code.claude.com/docs/en/discover-plugins)  
[![GitHub Repo](https://img.shields.io/badge/GitHub-claude-baton-black)](https://github.com/YOUR-USERNAME/claude-baton)  
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)  
![Stars](https://img.shields.io/github/stars/SuperInstance/claude-baton?style=social)

## Why this is becoming essential in 2026

- Native Claude Code already gives you Auto Memory (CLAUDE.md), context compaction, parallel subagents (up to 7+ in Agent Teams), and MCP for persistent storage — yet developers are still manually fighting context cliffs.
- Existing solutions like **claude-mem** (32k+ stars) do great session-to-session compression and RAG, but they summarize and lose detail. Ralph Wiggum gives long-running loops but resets context. No plugin yet delivers *proactive generational handoff* with full human-readable memoirs, decision logs, and A2A overlap.
- Claude_Baton is the production version the community has been asking for on Reddit, GitHub issues, and X: infinite context without compaction pain, auditable history, and automatic skill extraction.

## 🚀 60-Second Install

```bash
# In any Claude Code session
/plugin install claude-baton
/baton init
```

(Claude Code auto-detects complex projects and prompts you on first use. For brand-new repos it sets up `.baton/` and registers hooks instantly.)

## How It Works (built on official 2026 primitives)

| Trigger (via PreCompact + SessionStart hooks) | Action |
|-----------------------------------------------|--------|
| **48 %** | Background Onboarder subagent (parallel, zero main-context impact) builds baton bundle |
| **82 %** | Spawns Young Agent (native subagent or Agent Team) loaded only with fresh onboarding files |
| **93 %** | Old Agent demoted to pure Advisor. Full A2A overlap via shared Artifacts/MCP |
| **Retirement** | Full context snapshot archived; Young Agent takes over seamlessly |

All files are git-tracked and auto-committed. Unused generations move to cold storage.

## The Baton Protocol — Standardized & Open

Every generation produces the same clean artifact set in `.baton/generations/vN/`:

- `ONBOARDING.md` — instant ramp-up for the next agent
- `MEMOIRS/narrative.md` + compressed snapshot
- `DECISIONS_LOG.md` — rationale tree (why we chose what we chose)
- `SKILLS_EXTRACTED/` — generalized, ready-to-publish mini-skills
- `TASKS_NEXT.json` + Mermaid diagrams + successor self-test
- Vectorized copy via built-in MCP server (Chroma-style RAG)

New agents get free `/search-past` and `/recall-decision` tools.

## Features

- Live status bar (CLI + VS Code): `Gen 7 • 67 % • Onboarding active`
- `/baton tree` — beautiful interactive generation tree
- `/baton status` — full dashboard
- Three modes: Aggressive (long projects), Conservative, Human-Gated
- Automatic skill extraction → one-command publish to marketplace
- Works with teams (pure git — merge conflicts are just Markdown)
- Coexists perfectly with Claude’s Auto Memory, compaction, and Ralph Wiggum
- Negligible cost (prompt caching + background subagents)

## Configuration (`.baton/config.json`)

```json
{
  "thresholds": { "onboard": 48, "spawn": 82, "retire": 93 },
  "mode": "aggressive",
  "rag_enabled": true,
  "auto_commit": true
}
```

## Roadmap — Community-Driven (first 60 days)

- [ ] Official marketplace submission + VS Code sidebar visualizer
- [ ] Deep Agent Teams integration for multi-generation swarms
- [ ] One-click skill publishing to anthropics/claude-plugins-official
- [ ] Universal orchestrator (Option 2) — drop-in library for LangGraph, CrewAI, Cursor, Windsurf, etc.
- [ ] Team dashboard & cross-repo baton sharing

## Comparison to Existing Tools

| Tool              | Approach                     | Strengths                     | Claude_Baton Advantage                     |
|-------------------|------------------------------|-------------------------------|--------------------------------------------|
| Claude-Mem        | Session compression + RAG   | Great across-session recall  | Generational handoff + full memoirs + A2A |
| Ralph Wiggum      | Long-running loops          | Autonomous iteration         | Prevents context death entirely            |
| Native Compaction | Summarization               | Built-in                     | We avoid it — proactive relay instead     |
| Custom Subagents  | Manual delegation           | Flexible                     | Fully automatic + standardized protocol   |

## Contributing

This is meant to become infrastructure. PRs for new skills, hook improvements, or Option 2 orchestrator are gold. Join [Discussions](https://github.com/SuperInstance/claude-baton/discussions).

## License

MIT © 2026 Casey DiGennaro
---

**Built for the Claude Code community that wants agents to ship forever.**

Star if you’re tired of context cliffs. Let’s make generational memory the default for every serious project.
```

