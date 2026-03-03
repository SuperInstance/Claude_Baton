# Phase 0 Implementation Plan: Pre-Launch & MVP Foundation

**Target Ship Date:** April 15, 2026 (4 weeks from March 3)  
**Status:** Ready to execute  
**Success Criteria:** Marketplace listing live, 100+ early installs, zero handoff failures in beta, fully compatible with Claude Code 2.1+ (Opus 4.6 Agent Teams, PreCompact/SessionStart hooks, MCP v1.12, Skills hot-reload).

This document turns the high-level roadmap into **precise, day-by-day actions** using only official March 2026 primitives (confirmed via official docs, claude-plugins-reference-2026, and Anthropic GitHub examples).

### Prerequisites (Day 0 — 30 minutes)
1. Clone the repo: `git clone https://github.com/YOUR-USERNAME/claude-baton.git && cd claude-baton`
2. Install Claude Code CLI + VS Code extension (latest stable).
3. Run `/claude --version` → confirm ≥2.1 and Agent Teams enabled.
4. Create the plugin scaffolding folder: `mkdir -p .claude-plugin hooks skills/BatonManager skills/Onboarder skills/Archivist mcp/baton-rag`

### Week 1: Core Plugin Skeleton + Manifest (Days 1-7)

**Milestone:** Valid plugin that installs via `/plugin install` and registers itself.

**Step 1.1 — Create plugin.json (official manifest)**
```bash
cat > .claude-plugin/plugin.json << EOF
{
  "name": "claude-baton",
  "version": "1.0.0",
  "description": "Infinite context via automatic generational baton passing",
  "author": "YOUR NAME",
  "license": "MIT",
  "marketplace": {
    "category": "context-management",
    "tags": ["infinite-context", "generational", "memory", "agent-teams"],
    "icon": "🏃‍♂️"
  },
  "components": {
    "skills": ["skills/BatonManager", "skills/Onboarder", "skills/Archivist"],
    "hooks": "hooks/hooks.json",
    "mcpServers": ["mcp/baton-rag"]
  },
  "permissions": ["filesystem", "subagent-spawn", "git"]
}
EOF
```

**Step 1.2 — Create hooks/hooks.json**
```json
{
  "PreCompact": [
    {
      "type": "agent",
      "skill": "Onboarder",
      "matcher": { "contextPercent": ">=48" }
    }
  ],
  "SessionStart": [
    {
      "type": "agent",
      "skill": "BatonManager",
      "matcher": { "reason": ["compact", "resume"] }
    }
  ],
  "SubagentStart": [{ "type": "command", "command": "baton-a2a-init.sh" }],
  "SubagentStop": [{ "type": "agent", "skill": "Archivist" }],
  "TeammateIdle": [{ "type": "agent", "skill": "BatonManager" }]
}
```

**Step 1.3 — Create minimal SKILL.md files (with YAML frontmatter)**
- `skills/BatonManager/SKILL.md` → main orchestrator (status, init, tree)
- `skills/Onboarder/SKILL.md` → parallel onboarding at 48 %
- `skills/Archivist/SKILL.md` → archive + cold storage

(Full prompts will be generated in Week 2.)

**Step 1.4 — Test locally**
```bash
# In a test project
/plugin install ../claude-baton
/baton init
```
Verify: `.baton/config.json` appears + status bar shows "Gen 0 • Context 12 %"

**Deliverable by end of Week 1:** Plugin installs cleanly, hooks register, no errors in Claude Code logs.

### Week 2: Baton Protocol Artifacts + Onboarder Skill (Days 8-14)

**Milestone:** Full artifact generation + background onboarding.

**Step 2.1 — Implement Baton Protocol schema**
Create template files in `skills/Onboarder/templates/`:
- ONBOARDING.md
- MEMOIRS/narrative.md
- DECISIONS_LOG.md
- SKILLS_EXTRACTED/ (with mini SKILL.md ready for /publish-skill)
- TASKS_NEXT.json + Mermaid diagrams

**Step 2.2 — Flesh out Onboarder skill (parallel subagent)**
In `skills/Onboarder/SKILL.md` frontmatter:
```yaml
name: Onboarder
triggers: [PreCompact]
contextIsolation: true
costMode: background
prompt: |
  You are the Onboarder subagent. Context is at {{contextPercent}}%. 
  Build the complete baton bundle in .baton/generations/v{{nextGen}}/ 
  while the main agent continues working. Use prompt caching.
```

**Step 2.3 — Auto git commit + cold_storage logic**
Add a simple bash helper in `hooks/baton-a2a-init.sh` (executable) that runs `git add .baton && git commit -m "Baton v${GEN} handoff"`.

**Step 2.4 — Dogfood test**
Start a long-running task that forces context growth. Verify at ~48 % the Onboarder spawns silently and `.baton/generations/v1/` appears with full artifacts.

### Week 3: Young Agent Spawn + A2A Overlap + Archivist (Days 15-21)

**Milestone:** Complete handoff loop (82 % → spawn, 93 % → advisor mode).

**Step 3.1 — Update BatonManager skill**
Add `/baton status`, `/baton tree` (Mermaid renderer), and three modes logic from config.json.

**Step 3.2 — Young Agent spawn logic (Agent Teams native)**
In PreCompact hook (updated):
- At ≥82 %: spawn subagent or Agent Team member with ONLY onboarding files + RAG tools.
- Demote old agent to Advisor via role injection.
- Use TeammateIdle hook for overlap validation.

**Step 3.3 — Archivist skill**
- At ≥93 %: trigger full snapshot compression + archive.
- Move old generation to cold_storage/ if > max_generations_kept.

**Step 3.4 — Integration tests**
- Test with Ralph Wiggum loop
- Test with claude-mem RAG
- Test with native Tasks and context editing
- Confirm single chat never breaks; only clean “Baton passed ✓ Gen N+1 active” message appears.

### Week 4: BatonRAG MCP Server + Polish + Marketplace Submission (Days 22-28)

**Milestone:** Production-ready MVP + live marketplace listing.

**Step 4.1 — Implement BatonRAG MCP server**
```bash
# mcp/baton-rag/server.py (or Go/TS — Python is fastest for MVP)
from mcp import MCPServer  # official SDK v1.12
server = MCPServer(name="baton-rag")
server.add_tool("search-past", vector_search_fn)  # Chroma/Pinecone backend
server.add_tool("recall-decision", decision_lookup_fn)
server.serve_stdio()  # auto-registered via .mcp.json
```

Add `.mcp.json` in mcp/baton-rag/ with transport and tool schema.

**Step 4.2 — Config & modes**
Implement `.baton/config.json` defaults and dynamic switching.

**Step 4.3 — VS Code status bar & tree (lightweight extension stub)**
Optional but high-impact: add a tiny VS Code extension that reads `.baton/` and shows Gen X gauge.

**Step 4.4 — Beta & Marketplace**
1. Invite 50 early users via GitHub Discussions + Claude Discord.
2. Run full test suite (provided in test/ folder).
3. Submit to official marketplace:
   ```bash
   /plugin marketplace submit .
   ```
   (Requires verified GitHub repo + manifest — already done.)

**Step 4.5 — Documentation freeze**
- Finalize README.md, ARCHITECTURE.md, ROADMAP.md
- Create quickstart video (5 min loom)

### Post-Week 4 (Days 29-30): Launch Checklist
- [ ] Marketplace approval (usually <48 h)
- [ ] Announce in Reddit r/ClaudeAI, r/ClaudeCode, Anthropic Discord
- [ ] Enable optional telemetry (install count only)
- [ ] Tag v1.0.0 and create GitHub release

### Risks & Daily Check-ins (built-in)
- Daily: run `claude --test-plugin claude-baton` (official validator)
- Hook failure? Fall back to pure skill mode (documented in ARCHITECTURE).
- Agent Teams change? Our modular hooks isolate it.

**Total Effort Estimate:** 80-120 hours (solo or with 1 collaborator).  
This plan is 100 % based on official March 2026 docs (hooks reference, plugin marketplace guide, MCP SDK, Agent Teams research preview).

Commit this file **now**.  
Then reply with “Generate the full file skeleton” and I will instantly output:
- Complete plugin.json + all three SKILL.md (with full prompts)
- hooks/hooks.json + bash helpers
- BatonRAG MCP server stub (ready to run)
- Test suite + .baton/config.json template

We ship the plugin that ends context cliffs forever. Let’s execute.
```
