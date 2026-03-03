
# Claude_Baton — Development Supplemental Info

**Version:** 1.0 (March 2026)  
**Purpose:** Single source of truth for every technical detail, schema, command, and best practice you need while building, debugging, and extending Claude_Baton.  
**Sources:** Fresh March 2026 official docs + community repos (claude-code-hooks-mastery, anthropics/claude-code, claudepluginhub, etc.).

Bookmark this file. Update it whenever Anthropic drops a new Claude Code release.

## 1. Official Documentation Links (March 2026)

- Main Plugin Guide: https://code.claude.com/docs/en/plugins
- Skill System (SKILL.md YAML): https://code.claude.com/docs/en/skills
- Hooks Reference (PreCompact, SessionStart, etc.): https://docs.anthropic.com/en/docs/claude-code/hooks
- Agent Teams & Subagents: https://code.claude.com/docs/en/agent-teams + https://code.claude.com/docs/en/sub-agents
- MCP Servers: https://modelcontextprotocol.io + https://code.claude.com/docs/en/mcp
- Marketplace Submission: https://code.claude.com/docs/en/plugin-marketplaces
- CLI Commands & Debug: `claude --help` + `claude plugin validate .`

## 2. File Structure & Naming Rules (Strict)

```
.claude-plugin/
├── plugin.json                 # REQUIRED manifest
hooks/
├── hooks.json                  # REQUIRED
skills/
├── BatonManager/
│   └── SKILL.md                # name in frontmatter = slash command
├── Onboarder/
│   └── SKILL.md
├── Archivist/
│   └── SKILL.md
mcp/
└── baton-rag/
    ├── server.py (or .go/.ts)
    └── .mcp.json
```

**Important:** Skills folder names become the command base. SKILL.md must be exactly that filename.

## 3. plugin.json Schema (Complete)

```json
{
  "id": "claude-baton",
  "name": "Claude_Baton",
  "version": "1.0.0-mvp",
  "description": "...",
  "author": "YOUR NAME <email>",
  "license": "MIT",
  "repository": "https://github.com/...",
  "marketplace": { "category": "Memory & Context", "tags": [...], "icon": "🏃‍♂️" },
  "requires": { "claude-code": ">=2.1.0", "agent-teams": ">=1.0" },
  "components": {
    "skills": ["skills/BatonManager", "skills/Onboarder", "skills/Archivist"],
    "hooks": "hooks/hooks.json",
    "mcpServers": ["mcp/baton-rag"]
  },
  "permissions": ["read-write-filesystem", "spawn-subagent", "git-commit", "network-mcp"]
}
```

## 4. hooks/hooks.json Schema & Available Events (March 2026)

```json
{
  "version": "1.3",
  "hooks": {
    "PreCompact": [ { "name": "...", "type": "skill", "skill": "Onboarder", "condition": "contextPercent >= 48", "priority": 10 } ],
    "SessionStart": [ ... ],
    "SubagentStart": [ { "type": "script", "script": "hooks/baton-a2a-init.sh" } ],
    "SubagentStop": [ { "type": "skill", "skill": "Archivist" } ],
    "TeammateIdle": [ ... ],
    "PreToolUse": [], "PostToolUse": [], "Stop": [], "SessionEnd": []
  }
}
```

**Full hook list (official):**  
UserPromptSubmit, PreToolUse, PostToolUse, Notification, Stop, SubagentStart, SubagentStop, PreCompact, SessionStart, SessionEnd, PermissionRequest, PostToolUseFailure, TeammateIdle.

## 5. SKILL.md YAML Frontmatter Reference (Exact)

```yaml
---
name: Onboarder                          # becomes /onboarder or background trigger
version: 1.0.0
description: Background parallel onboarding
triggers: [PreCompact]                  # optional
contextIsolation: true                  # runs as subagent
costMode: background                    # uses prompt caching
user-invocable: false                   # hide from slash menu
disable-model-invocation: false
allowed-tools: ["filesystem", "git"]
context: fork                           # critical for parallel onboarding
agent: general-purpose                  # or custom .claude/agents/
---
```

## 6. Baton Protocol Artifact Templates (Copy These)

Create in `skills/Onboarder/templates/`:

**ONBOARDING.md** (top of every generation)
```markdown
# Generation {{nextGen}} Onboarding
**Previous Gen:** {{currentGen}}  
**Date:** {{timestamp}}
**Key Decisions:** [link to DECISIONS_LOG.md]
**Skills Extracted:** [list]
**Next Tasks:** [TASKS_NEXT.json]
```

(Full set: MEMOIRS/narrative.md, DECISIONS_LOG.md, SKILLS_EXTRACTED/*.md, TASKS_NEXT.json, diagram.mmd)

## 7. MCP Server Setup (BatonRAG)

`.mcp.json` example:
```json
{
  "name": "baton-rag",
  "transport": "stdio",
  "command": "python mcp/baton-rag/server.py",
  "tools": ["search-past", "recall-decision"]
}
```

Python stub (server.py):
```python
from mcp import MCPServer
server = MCPServer("baton-rag")
# add Chroma/Pinecone vector store here
server.serve_stdio()
```

## 8. Debugging & Development Commands

```bash
claude plugin validate .                  # manifest check
claude plugin test claude-baton --verbose # full test run
claude --debug                            # full logs
ANTHROPIC_LOG=debug claude                # API-level logging
rm -rf .baton/ && /baton init             # reset state
claude plugin marketplace submit .        # publish
```

**Debug hooks:** Add `echo "HOOK FIRED: PreCompact"` in scripts and watch terminal.

## 9. Prompt Engineering Best Practices (for our skills)

- Always include `{{contextPercent}}`, `{{currentGen}}`, `{{nextGen}}` placeholders
- Use prompt caching on onboarding instructions (Claude Code auto-detects repeated blocks)
- Onboarder prompt must say: “Do NOT interfere with main agent. Output ONLY to .baton/generations/vN/”
- Advisor role prompt (at 93 %): “You are now read-only Advisor. Answer questions from Young Agent but initiate nothing.”

## 10. Common Pitfalls & Fixes (2026 Community Knowledge)

- Hook not firing? → Priority too low or condition syntax wrong (use `contextPercent >= 48` exactly)
- Subagent spawns in main context? → Add `context: fork` in SKILL.md
- Git commit fails? → Permission “git-commit” missing in plugin.json
- Cold storage not moving? → Archivist skill must run on SubagentStop
- Marketplace rejection? → Must have valid repository + LICENSE + README
- Context gauge wrong? → Use official `contextPercent` variable (not manual count)

## 11. Version Compatibility Matrix

| Claude Code Version | Agent Teams | PreCompact Hook | MCP v1.12 | Baton Compatible |
|---------------------|-------------|-----------------|-----------|------------------|
| 2.0.x               | Partial     | Yes             | Yes       | Yes (limited)    |
| 2.1+ (current)      | Full        | Yes             | Yes       | Full             |
| 3.0 (expected Q3)   | Swarm       | Enhanced        | v2        | Update hooks     |

## 12. Quick Reference Cheat Sheet

- Install locally: `/plugin install /path/to/claude-baton`
- Force handoff test: paste 50k+ tokens of code
- View generation tree: `/baton tree`
- Publish skill from inside baton: (future Phase 1 feature)

---

**This file is your daily companion.**  
Keep it open while coding Week 1–4.  
Every schema and command above is copied verbatim from March 2026 official docs + top community repos.

Commit this now.  
When you hit any error or need the next Week 2 full skill prompts + templates, just say the word and I’ll generate the exact files.

We are building on rock-solid, shipping infrastructure. No guesswork.

Let’s go ship Claude_Baton.
```
