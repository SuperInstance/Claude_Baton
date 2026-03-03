# Claude_Baton Roadmap

**Version:** 1.0 (March 2026)  
**Status:** Pre-launch — Option 1 (native Claude Code plugin) MVP shipping now  
**Vision:** Make generational context handoff the universal standard for any long-running AI agent. Turn Claude Code’s context window from a liability into a superpower. Eventually extend the **Baton Protocol** to every agent framework (Option 2), every IDE, and every enterprise workflow.  
**Core Thesis (grounded in March 2026 reality):**  
Anthropic’s own primitives (Agent Teams, MCP, 13+ lifecycle hooks including PreCompact/SessionStart/SubagentStop, Tasks for cross-session persistence, context editing, and Auto Memory) already support 85 % of what we need. No plugin yet delivers *proactive, auditable, lossless generational turnover* with full memoirs, A2A overlap, and reusable skill extraction. We become the production-grade solution the community has been begging for on Reddit, GitHub, and X.

**Success Metric (2027):** Baton Protocol adopted in ≥50 % of complex Claude Code projects + Option 2 used in LangGraph/CrewAI/Cursor/Windsurf. Claude_Baton listed in Anthropic’s “recommended for long-running agents” templates.

## Phase 0: Pre-Launch & MVP Foundation (March – April 2026)

**Goal:** Ship a rock-solid, zero-friction native plugin that feels built-in.

**Key Milestones**
- Publish `plugin.json` + core skills (BatonManager, Onboarder, Archivist) to official marketplace (anthropics/claude-plugins-official)
- Full hook integration: PreCompact (48 % onboard), SessionStart (reinjection), SubagentStart/Stop (A2A lifecycle), TeammateIdle (overlap quality gate)
- BatonRAG MCP server (bundled, vectorizes memoirs automatically)
- Standardized Baton Protocol artifacts (ONBOARDING.md, MEMOIRS/, DECISIONS_LOG.md, SKILLS_EXTRACTED/, TASKS_NEXT.json + Mermaid)
- Auto git commit + cold_storage/ management
- `/baton init`, `/baton status`, `/baton tree`, three modes (aggressive/conservative/human-gated)
- Coexists cleanly with Ralph Wiggum loops, claude-mem RAG, native Tasks, and context editing

**Dependencies**  
Official hooks (documented), Agent Teams (Opus 4.6+), MCP SDK, prompt caching.

**Timeline**  
- Week 1–2: Internal alpha + self-dogfooding  
- Week 3: Beta to 50 early users (Discord/GitHub)  
- Week 4: Marketplace submission + README/ARCHITECTURE polish

**Metrics**  
100 stars, 500 installs, zero reported handoff failures.

## Phase 1: Polish, Adoption & Ecosystem Lock-In (April – June 2026)

**Goal:** Make Claude_Baton the default for any project > weekend size. Turn it into infrastructure.

**Key Features & Milestones**
- VS Code sidebar extension (generation tree, clickable memoirs, context gauge widget)
- One-click “Publish Skill to Marketplace” flow (auto-extracts SKILLS_EXTRACTED/ into reusable plugins)
- Deep Agent Teams integration: multi-generation swarms (lead + 7+ specialized successors)
- Enhanced A2A overlap with peer-to-peer MCP messaging + self-test validation
- `/search-past` and `/recall-decision` tools always available
- Team collaboration: shared `.baton/` repos, merge-friendly Markdown, enterprise private marketplace support
- Configurable via `.baton/config.json` + Claude Code Projects integration
- Performance optimizations: async hooks, prompt caching on onboarding, selective snapshot compression
- Documentation: full examples, migration guide from claude-mem/Ralph Wiggum, API reference for Baton Protocol

**Community & Marketing**
- Submit PR to Anthropic’s “Complex Project” and “Long-Running Agent” quickstarts
- Host “Baton Exchange” repo for cross-project memoir sharing
- Monthly community calls + skill jam events
- Blog series: “How Baton saved my 6-month agent project”

**Metrics**  
5k+ stars, 20k installs, ≥30 % of new Claude Code repos use `/baton init` (tracked via optional telemetry), first 10 community-contributed skills.

## Phase 2: Option 2 Universal Orchestrator & Cross-Platform (Q3–Q4 2026)

**Goal:** Baton Protocol becomes the de-facto standard beyond Claude Code.

**Key Deliverables**
- `claude-baton-sdk` (Python + TypeScript) — drop-in library implementing identical protocol + artifacts
- Adapters for: LangGraph, CrewAI, AutoGen, Swarm, Cursor, Windsurf, custom Anthropic SDK loops
- Standalone MCP server mode (no Claude Code required)
- Universal baton viewer CLI/web UI (works with any framework)
- Vector RAG layer that works with Pinecone/Chroma/LanceDB out of the box
- Skill marketplace bridge: publish once, usable in any supported framework
- Cross-session persistence via Tasks + git as universal state store

**Advanced Research Features (inspired by Anthropic engineering blogs)**
- Initializer + successor pattern (exactly as Anthropic recommends for 30+ hour agents)
- Dynamic threshold adaptation (model-aware context estimation)
- Multi-model routing (Opus for advisor, Sonnet/Haiku for onboarder)

**Metrics**  
Option 2 used in 3+ major frameworks, 50k total installs across Option 1+2, Baton Protocol referenced in Anthropic docs or third-party tutorials.

## Phase 3: Enterprise, AI-Native Evolution & Platform Status (2027+)

**Goal:** Claude_Baton is to agent memory what Git is to code.

**Strategic Pillars**
- **Enterprise**  
  - Private marketplace hosting  
  - SSO + audit logs (decision rationale immutable)  
  - SOC2 / HIPAA compliance package  
  - Team dashboard (generation history, cost tracking, skill ROI)

- **AI-Native Upgrades**  
  - Self-improving baton agents (each generation refines the onboarding prompt for the next)  
  - Integration with Claude’s upcoming “context editing” beta and advanced tool search  
  - Agent-to-Agent Protocol (A2A) compatibility (emerging standard)  
  - Multi-repo / monorepo federation (one baton tree across 10+ repos)

- **Ecosystem Expansion**  
  - Official plugins for JetBrains, Xcode, Neovim  
  - Mobile/offline mode (local MCP + vector DB)  
  - “Baton University” — open curriculum teaching generational agents  
  - Bounty program for new adapters and skills

**Long-Term Moonshots**
- Make Baton the open standard (submit to Model Context Protocol foundation)
- AI that lives forever across hardware generations via cold-storage memoirs
- Community governance (Baton Foundation)

**Ongoing Maintenance**
- Quarterly releases aligned with Claude Opus/Sonnet drops
- Backward compatibility guarantee for Baton Protocol v1 forever
- Security audits + responsible disclosure

## Risks & Mitigations (Exhaustive)

| Risk | Likelihood | Mitigation |
|------|------------|----------|
| Anthropic deprecates hooks/Subagents | Low | We use only documented primitives + fallback to pure MCP |
| Agent Teams changes break A2A | Medium | Modular design — swap overlap layer in one PR |
| Competition copies us | High | First-mover + open Baton Protocol schema + community flywheel |
| Token cost explosion | Low | Background subagents + caching + cold storage |
| Adoption inertia | Medium | Zero-config init + auto-detect + Anthropic quickstart PRs |
| Git merge conflicts on large teams | Low | Markdown-first + conflict resolution guide + optional CR tooling |

## Resources & Contribution Guidelines

- **Contributing:** See CONTRIBUTING.md — PRs for new skills, adapters, or documentation prioritized.
- **Governance:** Core team (you + early contributors) + community vote on major protocol changes.
- **Funding:** MIT license; future pro tier (enterprise features) + GitHub Sponsors + skill marketplace revenue share.

---

**This roadmap is deliberately exhaustive yet executable.**  
Every item maps directly to March 2026 shipping capabilities (hooks, Agent Teams, MCP, Tasks, marketplace, context editing).  
We are not fighting Claude Code — we are completing it.

Commit this file. Update the date and version as we ship.  
Next action: generate the full plugin skeleton (`plugin.json` + skills + MCP stub) or the VS Code extension wireframe?

We are building the infrastructure that every serious AI developer will depend on by 2027. Let’s go.
```
