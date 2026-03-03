# Claude_Baton Developer Blueprint

## The Complete Implementation Guide

**Goal:** Build a generational context handoff system that enables Claude Code agents to operate continuously across context window limits with zero information loss.

---

## What You're Building

Claude_Baton solves the "context wall" problem by implementing **proactive, structured memory transfer** between agent generations. Instead of reactive compaction that loses information, agents prepare handoffs in advance and transfer context seamlessly.

```
Context fills → Predictive prep (40%) → Build artifacts (48%) → 
Spawn successor (82%) → Advisory overlap (93%) → Clean handoff (98%)
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Claude_Baton System                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   HOOKS          THRESHOLDS         AGENTS          STORAGE │
│   ─────          ──────────         ──────          ─────── │
│   PreCompact  →  40% Predictive  →  Primary    →   .baton/  │
│   SessionStart → 48% Onboarding →  Advisor    →   Artifacts │
│   PostToolUse →  82% Spawn      →  Onboarder  →   RAG DB    │
│   Stop         → 98% Archive    →  Archivist  →   Archive   │
│                                                              │
│   ─────────────────────────────────────────────────────────│
│                    MCP Server (BatonRAG)                     │
│                    Vector search over all generations        │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 1: Project Structure

Create this directory structure:

```
your-plugin/
├── .claude-plugin/
│   ├── plugin.json           # Plugin metadata
│   ├── hooks/
│   │   ├── hooks.json        # Hook registrations
│   │   ├── pre-compact.sh    # Trigger handoff
│   │   ├── session-start.sh  # Resume context
│   │   └── decision-log.sh   # Capture decisions
│   └── agents/
│       ├── onboarder.md      # Builds artifacts
│       └── advisor.md        # Answers questions
│
├── mcp/baton-rag/
│   ├── server.py             # MCP server
│   └── requirements.txt
│
└── .baton/
    ├── current_generation    # "v1"
    ├── handoff_status        # "none" | "onboarding" | "ready"
    └── generations/
        ├── v1/
        │   ├── ONBOARDING.md
        │   ├── MEMOIRS/
        │   ├── DECISIONS_LOG.md
        │   └── TASKS_NEXT.json
        └── v2/
```

---

## Part 2: Hook System

### What Hooks Do

| Hook | Triggers When | Your Action |
|------|---------------|-------------|
| **PreCompact** | Context approaching limit | Initiate handoff sequence |
| **SessionStart** | New session begins | Load previous generation context |
| **PostToolUse** | After Write/Edit tools | Capture decision points |
| **Stop** | Agent tries to stop | Validate tasks complete |

### hooks.json Configuration

```json
{
  "hooks": {
    "PreCompact": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/pre-compact.sh",
        "timeout": 60
      }]
    }],
    "SessionStart": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "Write|Edit|MultiEdit",
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/decision-log.sh"
      }]
    }]
  }
}
```

### pre-compact.sh (Core Logic)

```bash
#!/bin/bash
set -euo pipefail

INPUT=$(cat)
CONTEXT_PCT=$(echo "$INPUT" | jq -r '.contextPercent // 0')
CURRENT_GEN=$(cat .baton/current_generation 2>/dev/null || echo "v1")
HANDOFF_STATUS=$(cat .baton/handoff_status 2>/dev/null || echo "none")

# At 48%: Start building artifacts
if [[ "$CONTEXT_PCT" -ge 48 && "$HANDOFF_STATUS" == "none" ]]; then
    NEXT_GEN="v$(($(echo $CURRENT_GEN | sed 's/v//') + 1))"
    mkdir -p ".baton/generations/$NEXT_GEN"
    
    # Spawn Onboarder agent (background, isolated)
    claude-agent spawn --role onboarder --isolated --background \
        --output ".baton/generations/$NEXT_GEN"
    
    echo "onboarding" > .baton/handoff_status
    jq -n '{"continue": true, "systemMessage": "⚡ Baton: Building artifacts for '"$NEXT_GEN"'..."}'
    
# At 98%: Execute handoff
elif [[ "$CONTEXT_PCT" -ge 98 && "$HANDOFF_STATUS" == "ready" ]]; then
    NEXT_GEN="v$(($(echo $CURRENT_GEN | sed 's/v//') + 1))"
    echo "$NEXT_GEN" > .baton/current_generation
    echo "none" > .baton/handoff_status
    
    jq -n '{"continue": true, "systemMessage": "🔄 Baton: Handoff complete. Read .baton/generations/'"$NEXT_GEN"'/ONBOARDING.md"}'
else
    jq -n '{"continue": true}'
fi
```

### session-start.sh (Resume Context)

```bash
#!/bin/bash
INPUT=$(cat)
RESUME_REASON=$(echo "$INPUT" | jq -r '.resume_reason // "new"')

if [[ -f ".baton/current_generation" && "$RESUME_REASON" == "compact" ]]; then
    GEN=$(cat .baton/current_generation)
    ONBOARDING=".baton/generations/$GEN/ONBOARDING.md"
    
    if [[ -f "$ONBOARDING" ]]; then
        jq -n --arg onboard "$(cat $ONBOARDING)" \
            '{"continue": true, "systemMessage": "📖 Baton: Resuming '"$GEN"'\n\n" + $onboard}'
    fi
else
    # Initialize new project
    mkdir -p .baton/generations/v1
    echo "v1" > .baton/current_generation
    jq -n '{"continue": true, "systemMessage": "🚀 Baton: Ready. Generation v1 started."}'
fi
```

---

## Part 3: The Baton Protocol Artifacts

These are the files that transfer context between generations.

### ONBOARDING.md (The Most Critical File)

```markdown
# Generation {{gen}} Onboarding

## Executive Summary
[2-minute read of entire project state - auto-generated]

## Active Decisions
| ID | Choice | Rationale | Rejected Alternatives |
|----|--------|-----------|----------------------|
| DEC-001 | PostgreSQL | Scale + ACID | MongoDB, SQLite |
| DEC-002 | React | Team expertise | Vue, Angular |

## In-Progress Tasks
1. [ ] Auth system - blocked by: none - files: src/auth/
2. [ ] Dashboard - blocked by: Auth - files: src/components/

## Self-Test Questions
1. Why did we reject MongoDB?
2. What's the purpose of processQueue()?
3. Which API is rate-limiting us?

## Quick Links
- Full Memoirs: `.baton/generations/{{prevGen}}/MEMOIRS/`
- Decision Log: `.baton/generations/{{prevGen}}/DECISIONS_LOG.md`
```

### DECISIONS_LOG.md

```markdown
# Decision Log

### DEC-001: Database Selection
- **When:** 2026-03-15 10:30
- **Context:** Need persistent storage for user data
- **Options:** PostgreSQL | MongoDB | SQLite
- **Chose:** PostgreSQL
- **Why:** ACID compliance required, team has expertise
- **Confidence:** 85%
- **Reversibility:** Low (would require migration)
- **Status:** Active

### DEC-002: Frontend Framework
- **When:** 2026-03-15 11:00
- **Context:** Building interactive dashboard
- **Options:** React | Vue | Angular
- **Chose:** React
- **Why:** Largest ecosystem, easiest hiring
- **Confidence:** 90%
- **Reversibility:** Medium
- **Status:** Active
```

### TASKS_NEXT.json

```json
{
  "generation": "v2",
  "tasks": [
    {
      "id": "TASK-001",
      "description": "Implement JWT authentication",
      "status": "in_progress",
      "priority": "high",
      "blocked_by": [],
      "files": ["src/auth/"],
      "context_needed": ["DEC-001", "DEC-003"]
    },
    {
      "id": "TASK-002", 
      "description": "Build user dashboard",
      "status": "pending",
      "priority": "medium",
      "blocked_by": ["TASK-001"],
      "files": ["src/components/Dashboard/"]
    }
  ]
}
```

---

## Part 4: MCP Server (BatonRAG)

### Purpose

Provides semantic search over all generations so any agent can query past decisions, patterns, and context.

### server.py

```python
from mcp.server.fastmcp import FastMCP
from chromadb import PersistentClient

mcp = FastMCP("BatonRAG")
client = PersistentClient(path="~/.claude/baton-rag-db")

# Collections for different memory types
memoirs = client.get_or_create_collection("memoirs")
decisions = client.get_or_create_collection("decisions")
skills = client.get_or_create_collection("skills")

@mcp.tool()
async def search_past(query: str, limit: int = 5) -> str:
    """Search across all generations for relevant context."""
    results = []
    for collection in [memoirs, decisions, skills]:
        hits = collection.query(query_texts=[query], n_results=limit)
        results.extend(hits["documents"][0])
    return results

@mcp.tool()
async def recall_decision(decision_id: str = None, keywords: list = None) -> str:
    """Fetch a specific decision or search by keywords."""
    if decision_id:
        return decisions.get(ids=[decision_id])
    return decisions.query(query_texts=[" ".join(keywords or [])], n_results=3)

@mcp.tool()
async def store_memory(content: str, memory_type: str, generation: str) -> str:
    """Store a memory in the vector database."""
    collection = {"memoir": memoirs, "decision": decisions, "skill": skills}[memory_type]
    collection.add(
        ids=[f"{memory_type}_{generation}_{hash(content)}"],
        documents=[content],
        metadatas=[{"generation": generation, "type": memory_type}]
    )
    return "stored"

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### Register in settings.json

```json
{
  "mcpServers": {
    "baton-rag": {
      "command": "python",
      "args": ["-m", "baton_rag.server"]
    }
  }
}
```

---

## Part 5: Agent Roles

### Onboarder Agent (Builds Artifacts)

```markdown
---
name: onboarder
model: haiku
tools: ["Read", "Grep", "Glob"]
---

You are the Onboarder. Build complete handoff artifacts for the next generation.

## Your Output
Create in `.baton/generations/{{nextGen}}/`:
1. ONBOARDING.md - Executive summary, decisions, tasks, self-test questions
2. MEMOIRS/narrative.md - Chronological story
3. DECISIONS_LOG.md - All decisions with rationale
4. TASKS_NEXT.json - Pending tasks with priorities

## Process
1. Read transcript → Extract decisions, patterns, errors
2. Synthesize → Create summaries and relationships
3. Write → Output all artifacts

Work in ISOLATED context. Complete within 30 minutes.
```

### Advisor Agent (Answers Questions)

```markdown
---
name: advisor
model: haiku
tools: ["Read", "Grep", "Glob"]
---

You are an Advisor from generation {{gen}}. Answer questions from the current primary agent.

## Constraints
- READ-ONLY: You cannot modify files
- REACTIVE: Only respond when asked
- MAX 4000 characters per response

## What You Provide
- Context clarification about your generation's work
- Pattern guidance from what worked before
- Error diagnosis based on your experience

## Format
## Context: [Brief background]
## Answer: [Direct response]
## References: [DEC-XXX, MEMOIRS/xxx.md]
```

---

## Part 6: Threshold System

### The 7 Thresholds

| % | Name | Action |
|---|------|--------|
| **40%** | Predictive | Analyze patterns, warm caches |
| **48%** | Onboarding | Spawn Onboarder, build artifacts |
| **60%** | Skill Extraction | Identify reusable patterns |
| **75%** | Advisory Prep | Draft advisor role, handoff checklist |
| **82%** | Spawn Successor | Launch new generation with ONBOARDING.md |
| **93%** | Advisory Transition | Demote old agent to read-only advisor |
| **98%** | Archive | Snapshot, update tree, retire advisor |

### Implementation

```python
THRESHOLDS = [
    {"pct": 40, "action": "predictive_analysis"},
    {"pct": 48, "action": "spawn_onboarder"},
    {"pct": 82, "action": "spawn_successor"},
    {"pct": 93, "action": "advisor_transition"},
    {"pct": 98, "action": "archive_and_retire"},
]

async def monitor_context():
    while True:
        current_pct = await get_context_percent()
        for t in THRESHOLDS:
            if current_pct >= t["pct"] and not triggered[t["pct"]]:
                await execute_action(t["action"])
                triggered[t["pct"]] = True
        await asyncio.sleep(5)
```

---

## Part 7: A2A Communication

### What It Does

Enables the old generation (Advisor) to answer questions from the new generation (Primary) during the 82%-98% overlap window.

### Message Types

| Type | Direction | Purpose |
|------|-----------|---------|
| QUESTION | Primary → Advisor | Ask about past work |
| ANSWER | Advisor → Primary | Provide context |
| VALIDATION_REQUEST | Primary → Advisor | "Is this approach correct?" |
| HANDOFF_INIT | Supervisor → All | Starting handoff |
| HANDOFF_COMPLETE | Supervisor → All | Handoff done |

### Simple Implementation

```python
# Primary asks
message = {
    "type": "QUESTION",
    "from": "v3-primary",
    "to": "v2-advisor", 
    "payload": {"question": "Why PostgreSQL over MongoDB?"}
}

# Advisor responds
response = {
    "type": "ANSWER",
    "from": "v2-advisor",
    "to": "v3-primary",
    "payload": {
        "answer": "DEC-001 chose PostgreSQL for ACID compliance...",
        "references": ["DEC-001"]
    }
}
```

---

## Part 8: Testing Checklist

### Before Each Release

- [ ] **Handoff Flow**: Context reaches 98% → v2 spawns → ONBOARDING.md loads
- [ ] **Artifacts**: All 4 artifacts generated correctly
- [ ] **RAG Search**: `search_past("database")` returns DEC-001
- [ ] **A2A**: Advisor responds to questions within 5 seconds
- [ ] **Recovery**: Failed handoff can be reset and retried
- [ ] **Memory**: Previous generations accessible via RAG
- [ ] **Performance**: Artifact generation < 30 minutes

### Quick Test

```bash
# 1. Initialize
/baton init

# 2. Force handoff test
/baton handoff --test

# 3. Verify artifacts
ls .baton/generations/v2/

# 4. Test RAG
/baton search "test query"

# 5. Check status
/baton status --verbose
```

---

## Part 9: Deployment

### Docker

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt . && pip install -r requirements.txt
COPY . .
RUN mkdir -p /app/.baton/generations
ENV BATON_DB_PATH=/data/baton-rag-db
EXPOSE 3000
CMD ["python", "-m", "baton_rag.server"]
```

### Health Check

```bash
/baton health
# Should return:
# ✓ MCP server connected
# ✓ Current generation: v3
# ✓ Artifacts complete
# ✓ RAG index healthy
```

---

## Quick Reference

### CLI Commands

| Command | Action |
|---------|--------|
| `/baton init` | Initialize for project |
| `/baton status` | Show current state |
| `/baton tree` | View generation tree |
| `/baton search <query>` | Search past context |
| `/baton why <topic>` | Find decision rationale |
| `/baton health` | Check system health |

### File Locations

| File | Purpose |
|------|---------|
| `.baton/current_generation` | Active generation ID |
| `.baton/handoff_status` | Current handoff state |
| `.baton/generations/vX/ONBOARDING.md` | Context for generation X |
| `.baton/generations/vX/DECISIONS_LOG.md` | All decisions with rationale |
| `.baton/generations/vX/TASKS_NEXT.json` | Pending tasks |

### Key Metrics

| Metric | Target |
|--------|--------|
| Handoff success rate | > 99% |
| Artifact generation | < 30 min |
| ONBOARDING.md load | < 100ms |
| RAG query latency | < 50ms |
| A2A response time | < 5s |

---

## Summary: What You Build

1. **Hooks** - Detect context thresholds and trigger actions
2. **Thresholds** - 7 levels from 40% to 98% with specific actions
3. **Artifacts** - ONBOARDING.md, MEMOIRS, DECISIONS_LOG, TASKS_NEXT
4. **MCP Server** - BatonRAG for semantic search over generations
5. **Agents** - Onboarder (builds artifacts), Advisor (answers questions)
6. **A2A Protocol** - Communication between generations during overlap

The result: Agents that never lose context, decisions that persist forever, and seamless handoffs that feel invisible to the user.
