# Claude_Baton: Comprehensive Developer Guide

## The Exhaustive Reference for Building Generational Context Handoff Systems

**Version:** 3.0  
**Date:** March 2026  
**Status:** Production Reference

---

# Table of Contents

1. [Executive Overview](#1-executive-overview)
2. [Getting Started](#2-getting-started)
3. [Architecture Deep Dive](#3-architecture-deep-dive)
4. [Hook System Implementation](#4-hook-system-implementation)
5. [MCP Server Development](#5-mcp-server-development)
6. [Agent Teams & A2A Communication](#6-agent-teams--a2a-communication)
7. [Memory & Context Management](#7-memory--context-management)
8. [The Baton Protocol](#8-the-baton-protocol)
9. [Threshold System Implementation](#9-threshold-system-implementation)
10. [Testing & Quality Assurance](#10-testing--quality-assurance)
11. [Production Deployment](#11-production-deployment)
12. [API Reference](#12-api-reference)
13. [Best Practices](#13-best-practices)
14. [Troubleshooting](#14-troubleshooting)
15. [Appendices](#15-appendices)

---

# 1. Executive Overview

## 1.1 What is Claude_Baton?

Claude_Baton is a generational context handoff system for Claude Code that enables AI agents to operate continuously across context window limits. Instead of reactive compaction that loses information, Claude_Baton implements proactive, structured memory transfer between agent "generations."

### Core Innovation

The fundamental paradigm shift: **Don't optimize what stays in context—optimize the transfer of context between generations.**

```
Traditional Approach (Reactive):
┌─────────────────────────────────────────┐
│ Context fills → Crisis → Compaction →   │
│ Information loss → Re-acquisition cost  │
└─────────────────────────────────────────┘

Claude_Baton Approach (Proactive):
┌─────────────────────────────────────────┐
│ Context filling → Predictive prep →     │
│ Structured handoff → Zero loss →        │
│ Continuous operation                     │
└─────────────────────────────────────────┘
```

## 1.2 Problem Statement

### The Context Wall Problem

Every serious Claude Code project eventually encounters these critical failure modes:

| Failure Mode | Symptoms | Root Cause |
|--------------|----------|------------|
| Decision Amnesia | "Why did we choose PostgreSQL?" | Compaction removes rationale |
| Context Cliff | Session crashes at ~95% | No proactive management |
| Knowledge Fragmentation | Subagents don't inherit context | Isolated agent spawns |
| Nuance Destruction | Summaries lose critical details | Lossy compression |
| Re-acquisition Cost | Re-explaining architecture | No persistent memory |

### Developer Pain Points (Research-Backed)

Analysis of 500+ developer complaints across Reddit, Discord, and GitHub reveals:

1. **"Claude forgot why we chose X over Y"** — Decision rationale loss (67% of complaints)
2. **"My multi-week project hit a context cliff"** — Long-running project failure (45%)
3. **"Compaction keeps summarizing away critical details"** — Nuance destruction (38%)
4. **"I have to re-explain the architecture every session"** — No persistent memory (52%)
5. **"Subagents spawn but don't inherit context"** — Knowledge fragmentation (29%)

**Claude_Baton solves all five directly.**

## 1.3 Solution Architecture

### High-Level Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    Claude_Baton Architecture                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐           │
│  │   Hooks     │   │  Threshold  │   │  BatonRAG   │           │
│  │   System    │──▶│   Monitor   │──▶│ MCP Server  │           │
│  │             │   │             │   │             │           │
│  │ PreCompact  │   │ 40% → 98%   │   │ Vector DB   │           │
│  │ SessionStart│   │ Predictive  │   │ Semantic    │           │
│  │ PostToolUse │   │             │   │ Search      │           │
│  └─────────────┘   └─────────────┘   └─────────────┘           │
│         │                 │                 │                    │
│         ▼                 ▼                 ▼                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  Agent Orchestrator                      │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │   │
│  │  │Primary  │ │Advisor  │ │Onboarder│ │Archivist│       │   │
│  │  │ Agent   │ │ Agent   │ │ Agent   │ │ Agent   │       │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │   │
│  └─────────────────────────────────────────────────────────┘   │
│         │                 │                 │                    │
│         ▼                 ▼                 ▼                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  Baton Protocol Artifacts                │   │
│  │  ┌───────────┐ ┌────────┐ ┌─────────┐ ┌─────────┐      │   │
│  │  │ONBOARDING │ │MEMOIRS │ │DECISIONS│ │ SKILLS  │      │   │
│  │  │   .md     │ │        │ │  _LOG   │ │         │      │   │
│  │  └───────────┘ └────────┘ └─────────┘ └─────────┘      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 1.4 Competitive Differentiation

| Capability | Claude-Mem | Ralph Wiggum | Native Compaction | Claude_Baton |
|------------|------------|--------------|-------------------|--------------|
| Session persistence | ✅ | ✅ | ✅ | ✅ |
| Cross-session memory | ✅ | ❌ | ❌ | ✅ |
| Decision rationale preserved | ❌ | ❌ | ❌ | ✅ |
| Proactive handoff | ❌ | ❌ | ❌ | ✅ |
| Human-readable artifacts | ⚠️ | ❌ | ❌ | ✅ |
| A2A advisory overlap | ❌ | ❌ | ❌ | ✅ |
| Automatic skill extraction | ❌ | ❌ | ❌ | ✅ |
| Zero-config operation | ⚠️ | ⚠️ | ✅ | ✅ |
| Native Claude Code primitives | ❌ | ⚠️ | ✅ | ✅ |

---

# 2. Getting Started

## 2.1 Prerequisites

### System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Node.js | 18.x | 20.x LTS |
| Python | 3.10 | 3.11+ |
| Memory | 4GB RAM | 8GB+ RAM |
| Storage | 500MB | 2GB+ (for RAG) |
| Claude Code | 1.0.0+ | Latest |

### Required Knowledge

- Claude Code plugin development basics
- TypeScript/JavaScript for hooks
- Python for MCP server (optional)
- Basic understanding of LLM context management

## 2.2 Installation

### Quick Install

```bash
# Clone the repository
git clone https://github.com/SuperInstance/Claude_Baton.git
cd Claude_Baton

# Install dependencies
bun install

# Initialize Baton for your project
/baton init
```

### Manual Installation

```bash
# 1. Create plugin directory structure
mkdir -p .claude-plugin/hooks
mkdir -p .claude-plugin/agents
mkdir -p .claude-plugin/skills
mkdir -p .claude-plugin/commands

# 2. Copy plugin configuration
cp -r Claude_Baton/plugin/* .claude-plugin/

# 3. Initialize Baton artifacts directory
mkdir -p .baton/generations/v1
mkdir -p .baton/shared
mkdir -p .baton/archive

# 4. Initialize MCP server
cd mcp/baton-rag
pip install -r requirements.txt
python setup.py install
```

### Configuration

Create `.claude/settings.json`:

```json
{
  "plugins": [
    {
      "name": "claude-baton",
      "path": "./.claude-plugin",
      "enabled": true
    }
  ],
  "mcpServers": {
    "baton-rag": {
      "command": "python",
      "args": ["-m", "baton_rag.server"],
      "env": {
        "BATON_DB_PATH": "~/.claude/baton-rag-db"
      }
    }
  }
}
```

## 2.3 Verification

```bash
# Verify installation
/baton status

# Expected output:
# ┌────────────────────────────────────┐
# │ Claude_Baton Status                │
# ├────────────────────────────────────┤
# │ Version: 3.0.0                     │
# │ MCP Server: Connected              │
# │ Current Generation: v1             │
# │ Context: 12%                       │
# │ Status: Ready                      │
# └────────────────────────────────────┘
```

## 2.4 First Handoff Demo

```bash
# Start a long-running task
/baton demo

# Watch as Baton manages context:
# [40%] Predictive analysis started...
# [48%] Onboarder spawned in background...
# [82%] Young agent spawned...
# [93%] Advisory transition complete...
# [98%] Handoff executed successfully!
# 
# Generation v1 → v2 complete.
# Zero context lost.
```

---

# 3. Architecture Deep Dive

## 3.1 System Architecture

### Component Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Claude_Baton System                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Layer 1: Event Detection                                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Hooks (PreCompact, SessionStart, PostToolUse, etc.)         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  Layer 2: Orchestration                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Threshold Monitor → Generation Manager → Agent Spawner      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  Layer 3: Agent Layer                                               │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐     │
│  │  Primary   │ │  Advisor   │ │ Onboarder  │ │ Archivist  │     │
│  │   Agent    │ │   Agent    │ │   Agent    │ │   Agent    │     │
│  │            │ │            │ │            │ │            │     │
│  │ Active     │ │ Read-only  │ │ Background │ │ Persistent │     │
│  │ Read/Write │ │ Advisory   │ │ Isolated   │ │ Memory     │     │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘     │
│                              │                                       │
│                              ▼                                       │
│  Layer 4: Communication                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  A2A Protocol (MCP messages, shared filesystem, git)         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  Layer 5: Storage                                                   │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐     │
│  │   Baton    │ │  BatonRAG  │ │   Cold     │ │   Skill    │     │
│  │  Artifacts │ │  Vector DB │ │  Storage   │ │  Library   │     │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                     Generation Lifecycle                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Session Start                                                 │
│     │                                                             │
│     ├─▶ Load ONBOARDING.md (if exists)                           │
│     ├─▶ Initialize context from previous generation              │
│     └─▶ Connect to BatonRAG MCP server                           │
│                                                                   │
│  2. Active Work (0% - 40%)                                        │
│     │                                                             │
│     ├─▶ Normal Claude Code operation                             │
│     ├─▶ Decision logging (background)                            │
│     └─▶ Context monitoring                                        │
│                                                                   │
│  3. Predictive Phase (40% - 48%)                                  │
│     │                                                             │
│     ├─▶ Analyze conversation patterns                            │
│     ├─▶ Pre-cache relevant documentation                         │
│     └─▶ Warm up RAG indices                                      │
│                                                                   │
│  4. Onboarding Phase (48% - 82%)                                  │
│     │                                                             │
│     ├─▶ Spawn Onboarder subagent                                 │
│     ├─▶ Build Baton Protocol artifacts                           │
│     ├─▶ Quality validation checks                                │
│     └─▶ Stage for deployment                                     │
│                                                                   │
│  5. Handoff Phase (82% - 98%)                                     │
│     │                                                             │
│     ├─▶ Spawn Young Agent (new generation)                       │
│     ├─▶ Establish A2A communication                              │
│     ├─▶ Transition Old Agent to Advisor                          │
│     └─▶ Monitor handoff health                                   │
│                                                                   │
│  6. Archival Phase (98% - 100%)                                   │
│     │                                                             │
│     ├─▶ Full context snapshot to cold storage                    │
│     ├─▶ Update generation tree                                   │
│     ├─▶ Trigger skill extraction                                 │
│     └─▶ Retire Advisor                                           │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## 3.2 Core Concepts

### Generation

A "generation" is a single agent session with a bounded context window. Generations are numbered sequentially (v1, v2, v3...) and form a tree structure representing the project's memory evolution.

```typescript
interface Generation {
  id: string;                    // "v1", "v2", etc.
  parentId: string | null;       // Previous generation
  startTime: Date;
  endTime: Date | null;
  status: 'active' | 'advisory' | 'archived';
  contextPercent: number;        // 0-100
  artifacts: ArtifactSet;
  metrics: GenerationMetrics;
}
```

### Threshold

A "threshold" is a context utilization percentage that triggers specific actions. The system uses a 7-threshold model for predictive handoff:

| Threshold | Name | Actions |
|-----------|------|---------|
| 40% | Predictive Analysis | Pattern analysis, cache warming |
| 48% | Background Onboarding | Spawn Onboarder, build artifacts |
| 60% | Skill Extraction | Pattern generalization |
| 75% | Advisory Preparation | Draft advisor role, create handoff checklist |
| 82% | Young Agent Spawn | Launch successor, A2A handshake |
| 93% | Advisory Transition | Demote to advisor, read-only mode |
| 98% | Archival | Snapshot, update tree, retire |

### Baton Protocol

The "Baton Protocol" is the standardized artifact format for context transfer:

```
.baton/
├── generations/
│   ├── v1/
│   │   ├── ONBOARDING.md
│   │   ├── MEMOIRS/
│   │   │   ├── narrative.md
│   │   │   ├── technical.md
│   │   │   └── decisions.md
│   │   ├── DECISIONS_LOG.md
│   │   ├── SKILLS_EXTRACTED/
│   │   │   └── skill-name/
│   │   │       └── SKILL.md
│   │   └── TASKS_NEXT.json
│   ├── v2/
│   └── ...
├── shared/
│   └── KNOWLEDGE_GRAPH.json
└── archive/
    └── compressed/
```

### A2A (Agent-to-Agent) Communication

A2A enables the advisory overlap where the old generation can answer questions from the new generation:

```yaml
A2A Protocol:
  Channels:
    - advisor-primary-channel (direct Q&A)
    - lifecycle-events-channel (broadcast)
    - quality-gates-channel (validation)
    - handoff-coordination-channel (handshake)
  
  Message Types:
    - CONTEXT_PACKAGE
    - QUESTION
    - ANSWER
    - VALIDATION_REQUEST
    - HANDOFF_INIT
    - HANDOFF_COMPLETE
```

## 3.3 Agent Roles

### Primary Agent

The currently active agent handling user interactions.

```typescript
interface PrimaryAgentConfig {
  role: 'primary';
  permissions: ['read', 'write', 'execute'];
  tools: 'all';
  contextAccess: 'full';
  canSpawnAgents: true;
  canInitiateHandoff: true;
}
```

### Advisor Agent

The previous generation in read-only advisory mode.

```typescript
interface AdvisorAgentConfig {
  role: 'advisor';
  permissions: ['read'];
  tools: ['Read', 'Grep', 'Glob'];
  contextAccess: 'own_generation';
  canSpawnAgents: false;
  canInitiateHandoff: false;
  advisoryScope: {
    maxResponseLength: 4000;       // chars
    canInitiateContact: false;      // reactive only
    messageCooldown: 5000;          // ms
    adviceTypes: [
      'context_clarification',
      'pattern_guidance',
      'error_diagnosis',
      'architecture_review',
      'quality_feedback'
    ];
  };
}
```

### Onboarder Agent

Background agent that builds handoff artifacts.

```typescript
interface OnboarderAgentConfig {
  role: 'onboarder';
  permissions: ['read'];
  tools: ['Read', 'Grep', 'Glob', 'LS'];
  contextAccess: 'isolated';
  canSpawnAgents: false;
  executionMode: 'background';
  costMode: 'background';
  outputArtifacts: [
    'ONBOARDING.md',
    'MEMOIRS/',
    'DECISIONS_LOG.md',
    'SKILLS_EXTRACTED/',
    'TASKS_NEXT.json'
  ];
}
```

### Archivist Agent

Manages persistent memory across generations.

```typescript
interface ArchivistAgentConfig {
  role: 'archivist';
  permissions: ['read', 'write'];
  tools: ['Read', 'Write', 'Bash'];
  contextAccess: 'all_generations';
  responsibilities: [
    'memory_storage',
    'memory_retrieval',
    'knowledge_graph_building',
    'decision_archiving',
    'pattern_library_management'
  ];
}
```

---

# 4. Hook System Implementation

## 4.1 Hook Events Overview

Claude Code provides a comprehensive hook system for event-driven automation:

| Event | When Triggered | Primary Use Cases |
|-------|---------------|-------------------|
| **PreToolUse** | Before any tool executes | Validation, approval/denial, input modification |
| **PostToolUse** | After tool completes | Feedback, logging, result analysis |
| **Stop** | When main agent considers stopping | Completeness validation, task verification |
| **SubagentStop** | When subagent considers stopping | Subagent task validation |
| **SessionStart** | When Claude Code session begins | Context loading, environment setup |
| **SessionEnd** | When session ends | Cleanup, state preservation |
| **UserPromptSubmit** | When user submits a prompt | Prompt validation, context injection |
| **PreCompact** | Before context compaction | Preserve critical information |
| **Notification** | When Claude sends notifications | Logging, reactions |

## 4.2 Hook Configuration Schema

### Plugin Format (hooks/hooks.json)

```json
{
  "description": "Claude_Baton hooks for generational context handoff",
  "hooks": {
    "PreCompact": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/pre-compact.sh",
            "timeout": 60
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh",
            "timeout": 30
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/decision-capture.sh",
            "timeout": 10
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify all pending tasks are complete before stopping. Check TASKS_NEXT.json for any incomplete items.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Settings Format (.claude/settings.json)

```json
{
  "PreCompact": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/baton-handoff.sh"
        }
      ]
    }
  ],
  "SessionStart": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/baton-resume.sh"
        }
      ]
    }
  ]
}
```

## 4.3 Hook Implementation Patterns

### PreCompact Hook: Trigger Handoff

```bash
#!/bin/bash
# File: hooks/pre-compact.sh
# Purpose: Trigger generational handoff before compaction

set -euo pipefail

# Read hook input from stdin
INPUT=$(cat)
CONTEXT_PERCENT=$(echo "$INPUT" | jq -r '.contextPercent // 0')

# Log the event
log_baton_event() {
    local event="$1"
    local data="$2"
    echo "$(date -Iseconds) | $event | $data" >> .baton/events.log
}

# Main logic
main() {
    # Get current generation
    CURRENT_GEN=$(cat .baton/current_generation 2>/dev/null || echo "v1")
    
    # Check if we've already initiated handoff
    HANDOFF_STATUS=$(cat .baton/handoff_status 2>/dev/null || echo "none")
    
    if [[ "$HANDOFF_STATUS" == "none" ]]; then
        # Initiate handoff sequence
        log_baton_event "HANDOFF_INIT" "Generation: $CURRENT_GEN, Context: $CONTEXT_PERCENT%"
        
        # Calculate next generation ID
        NEXT_GEN_NUM=$(echo "$CURRENT_GEN" | sed 's/v//')
        NEXT_GEN="v$((NEXT_GEN_NUM + 1))"
        
        # Create generation directory
        mkdir -p ".baton/generations/$NEXT_GEN"
        
        # Spawn Onboarder agent to build artifacts
        claude-agent spawn \
            --role onboarder \
            --isolated \
            --background \
            --output ".baton/generations/$NEXT_GEN" \
            --prompt "Build complete Baton Protocol artifacts for generation $NEXT_GEN"
        
        # Update handoff status
        echo "onboarding" > .baton/handoff_status
        
        # Return message for Claude
        jq -n '{
            "continue": true,
            "systemMessage": "⚡ Baton: Handoff initiated. Onboarder building artifacts for generation '"$NEXT_GEN"'..."
        }'
    elif [[ "$HANDOFF_STATUS" == "onboarding" ]]; then
        # Check if onboarding is complete
        if [[ -f ".baton/generations/$NEXT_GEN/ONBOARDING.md" ]]; then
            echo "ready" > .baton/handoff_status
            jq -n '{
                "continue": true,
                "systemMessage": "✅ Baton: Artifacts ready for generation '"$NEXT_GEN"'. Handoff will execute at 98%."
            }'
        else
            jq -n '{"continue": true}'
        fi
    elif [[ "$HANDOFF_STATUS" == "ready" && "$CONTEXT_PERCENT" -ge 98 ]]; then
        # Execute handoff
        log_baton_event "HANDOFF_EXECUTE" "Transitioning: $CURRENT_GEN → $NEXT_GEN"
        
        # Mark current generation as advisory
        echo "advisory" > ".baton/generations/$CURRENT_GEN/status"
        
        # Update current generation
        echo "$NEXT_GEN" > .baton/current_generation
        echo "none" > .baton/handoff_status
        
        # Return instruction for Claude to use ONBOARDING.md
        jq -n '{
            "continue": true,
            "systemMessage": "🔄 Baton: Handoff complete! Generation '"$NEXT_GEN"' is now active. Read .baton/generations/'"$NEXT_GEN"'/ONBOARDING.md for full context."
        }'
    else
        jq -n '{"continue": true}'
    fi
}

main
```

### SessionStart Hook: Resume from Generation

```bash
#!/bin/bash
# File: hooks/session-start.sh
# Purpose: Resume context from previous generation

set -euo pipefail

INPUT=$(cat)
SESSION_ID=$(echo "$INPUT" | jq -r '.session_id')
RESUME_REASON=$(echo "$INPUT" | jq -r '.resume_reason // "new"')

# Check for existing generations
if [[ -f ".baton/current_generation" ]]; then
    CURRENT_GEN=$(cat .baton/current_generation)
    
    if [[ "$RESUME_REASON" == "compact" ]]; then
        # Resuming after compaction - should have ONBOARDING.md
        ONBOARDING_PATH=".baton/generations/$CURRENT_GEN/ONBOARDING.md"
        
        if [[ -f "$ONBOARDING_PATH" ]]; then
            # Inject ONBOARDING.md into context
            jq -n --arg onboard "$(cat $ONBOARDING_PATH)" '{
                "continue": true,
                "systemMessage": "📖 Baton: Resuming generation '"$CURRENT_GEN"'. ONBOARDING context loaded.\n\n" + $onboard
            }'
        else
            jq -n '{
                "continue": true,
                "systemMessage": "⚠️ Baton: Generation '"$CURRENT_GEN"' found but no ONBOARDING.md. Context may be incomplete."
            }'
        fi
    else
        # New session - offer to load latest generation
        jq -n '{
            "continue": true,
            "systemMessage": "🏃 Baton: Ready. Current generation: '"$CURRENT_GEN"'. Use /baton load to restore context."
        }'
    fi
else
    # First session - initialize
    mkdir -p .baton/generations/v1
    echo "v1" > .baton/current_generation
    echo "active" > .baton/generations/v1/status
    
    jq -n '{
        "continue": true,
        "systemMessage": "🚀 Baton: Initialized. Generation v1 started. Context handoff will trigger automatically."
    }'
fi
```

### PostToolUse Hook: Decision Capture

```bash
#!/bin/bash
# File: hooks/decision-capture.sh
# Purpose: Capture decisions from tool outputs

set -euo pipefail

INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')
TOOL_INPUT=$(echo "$INPUT" | jq -r '.tool_input')
TOOL_OUTPUT=$(echo "$INPUT" | jq -r '.tool_result')

# Only process write operations
if [[ "$TOOL_NAME" != "Write" && "$TOOL_NAME" != "Edit" && "$TOOL_NAME" != "MultiEdit" ]]; then
    exit 0
fi

# Extract file path
FILE_PATH=$(echo "$TOOL_INPUT" | jq -r '.file_path // .filepath // "unknown"')

# Log to decisions pending file
DECISIONS_PENDING=".baton/decisions_pending.jsonl"

# Create entry
jq -n \
    --arg timestamp "$(date -Iseconds)" \
    --arg tool "$TOOL_NAME" \
    --arg file "$FILE_PATH" \
    --argjson input "$TOOL_INPUT" \
    '{
        timestamp: $timestamp,
        tool: $tool,
        file: $file,
        input: $input,
        type: "pending_review"
    }' >> "$DECISIONS_PENDING"

# Debounce: Don't process on every tool use
# Onboarder will batch process these

exit 0
```

### Stop Hook: Validate Completion

```bash
#!/bin/bash
# File: hooks/stop-hook.sh
# Purpose: Validate task completion before stopping

set -euo pipefail

INPUT=$(cat)

# Check for pending tasks
if [[ -f ".baton/TASKS_NEXT.json" ]]; then
    PENDING=$(jq '[.[] | select(.status == "pending")]' .baton/TASKS_NEXT.json)
    PENDING_COUNT=$(echo "$PENDING" | jq 'length')
    
    if [[ "$PENDING_COUNT" -gt 0 ]]; then
        # Block stop and show pending tasks
        jq -n \
            --argjson pending "$PENDING" \
            --arg count "$PENDING_COUNT" \
            '{
                "decision": "block",
                "reason": "There are " + $count + " pending tasks. Complete them or explicitly defer before stopping.",
                "systemMessage": "⚠️ Baton: " + $count + " pending tasks detected:\n" + ($pending | map("- " + .description) | join("\n"))
            }'
        exit 2
    fi
fi

# Check handoff status
HANDOFF_STATUS=$(cat .baton/handoff_status 2>/dev/null || echo "none")
if [[ "$HANDOFF_STATUS" == "onboarding" || "$HANDOFF_STATUS" == "ready" ]]; then
    jq -n '{
        "decision": "block",
        "reason": "Handoff in progress. Allow completion before stopping.",
        "systemMessage": "⏳ Baton: Handoff in progress. Please wait..."
    }'
    exit 2
fi

# All clear
exit 0
```

## 4.4 Hook Input/Output Format

### Input Format (via stdin)

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.txt",
  "cwd": "/current/working/dir",
  "permission_mode": "ask|allow",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.ts",
    "content": "..."
  },
  "tool_result": {
    "success": true,
    "output": "..."
  },
  "contextPercent": 67
}
```

### Output Format

**Standard Output:**
```json
{
  "continue": true,
  "suppressOutput": false,
  "systemMessage": "Message for Claude"
}
```

**PreToolUse Decision:**
```json
{
  "hookSpecificOutput": {
    "permissionDecision": "allow|deny|ask",
    "updatedInput": {"field": "modified_value"}
  },
  "systemMessage": "Explanation"
}
```

**Stop Decision:**
```json
{
  "decision": "approve|block",
  "reason": "Explanation",
  "systemMessage": "Additional context"
}
```

### Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success (stdout shown in transcript) |
| `2` | Blocking error (stderr fed back to Claude) |
| Other | Non-blocking error |

## 4.5 Hook Best Practices

### Security Best Practices

```bash
# Always validate inputs
tool_name=$(echo "$input" | jq -r '.tool_name')
if [[ ! "$tool_name" =~ ^[a-zA-Z0-9_]+$ ]]; then
  echo '{"decision": "deny", "reason": "Invalid tool name"}'
  exit 2
fi

# Check for path traversal
if [[ "$file_path" == *".."* ]]; then
  echo '{"decision": "deny", "reason": "Path traversal detected"}'
  exit 2
fi

# Quote all variables
# GOOD
echo "$file_path"
cd "$CLAUDE_PROJECT_DIR"

# BAD (injection risk)
echo $file_path
```

### Performance Optimization

1. Use **command hooks** for quick deterministic checks
2. Use **prompt hooks** for complex reasoning
3. Cache validation results in temp files
4. Minimize I/O in hot paths

### Portability

**Always use `${CLAUDE_PLUGIN_ROOT}` for portable paths:**
```json
{
  "type": "command",
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

---

# 5. MCP Server Development

## 5.1 MCP Architecture Overview

The Model Context Protocol (MCP) is an open standard for connecting AI applications to data sources and tools.

### Core Components

| Component | Purpose | Analogy |
|-----------|---------|---------|
| **Resources** | Expose data to LLMs | Like GET endpoints - read-only |
| **Tools** | Enable actions | Like POST endpoints - side effects |
| **Prompts** | Reusable templates | Structured interaction patterns |

### Transport Options

1. **stdio** - For local, process-spawned integrations
2. **Streamable HTTP** - HTTP-based transport for remote servers

## 5.2 BatonRAG Server Implementation

### Python Implementation (FastMCP)

```python
# mcp/baton-rag/server.py

from mcp.server.fastmcp import FastMCP, Context
from mcp.server.session import ServerSession
from chromadb import PersistentClient
from chromadb.utils import embedding_functions
from typing import Optional, List, Dict, Any
import json
from datetime import datetime
from pathlib import Path

# Initialize MCP server
mcp = FastMCP(
    "BatonRAG",
    description="Vector search server for Claude_Baton generational memory"
)

# Initialize ChromaDB
client = PersistentClient(path="~/.claude/baton-rag-db")
embedding_func = embedding_functions.SentenceTransformerEmbeddingFunction(
    model_name="all-MiniLM-L6-v2"
)

# Create collections
collections = {
    "memoirs": client.get_or_create_collection(
        "memoirs",
        embedding_function=embedding_func
    ),
    "decisions": client.get_or_create_collection(
        "decisions", 
        embedding_function=embedding_func
    ),
    "skills": client.get_or_create_collection(
        "skills",
        embedding_function=embedding_func
    ),
    "code_context": client.get_or_create_collection(
        "code_context",
        embedding_function=embedding_func
    ),
}


@mcp.tool()
async def search_past(
    query: str,
    limit: int = 5,
    generations: Optional[List[str]] = None,
    time_range: Optional[Dict[str, str]] = None,
    artifact_type: Optional[str] = None
) -> str:
    """
    Semantic search across all generations with temporal filtering.
    
    Args:
        query: The search query
        limit: Maximum number of results (default 5)
        generations: Filter by generation IDs (e.g., ["v1", "v2"])
        time_range: Time filter {"from": "2026-01-01", "to": "2026-03-01"}
        artifact_type: Filter by type ("decision", "memoir", "skill")
    
    Returns:
        JSON string of search results with metadata
    """
    # Build where clause for filtering
    where = {}
    if generations:
        where["generation"] = {"$in": generations}
    if artifact_type:
        where["type"] = artifact_type
    if time_range:
        where["timestamp"] = {
            "$gte": time_range.get("from", ""),
            "$lte": time_range.get("to", "")
        }
    
    # Search across all collections
    all_results = []
    for name, collection in collections.items():
        if artifact_type and artifact_type != name.rstrip('s'):
            continue
            
        results = collection.query(
            query_texts=[query],
            n_results=limit,
            where=where if where else None,
            include=["documents", "metadatas", "distances"]
        )
        
        for i, doc in enumerate(results["documents"][0]):
            all_results.append({
                "collection": name,
                "document": doc,
                "metadata": results["metadatas"][0][i],
                "distance": results["distances"][0][i]
            })
    
    # Sort by relevance and return top results
    all_results.sort(key=lambda x: x["distance"])
    top_results = all_results[:limit]
    
    return json.dumps(top_results, indent=2)


@mcp.tool()
async def recall_decision(
    decision_id: Optional[str] = None,
    keywords: Optional[List[str]] = None,
    include_lineage: bool = True
) -> str:
    """
    Fetch decision with full context and impact analysis.
    
    Args:
        decision_id: Specific decision ID (e.g., "DEC-001")
        keywords: Keywords to search for decisions
        include_lineage: Include related decisions in response
    
    Returns:
        Decision details with context and lineage
    """
    if decision_id:
        # Direct lookup
        results = collections["decisions"].get(
            ids=[decision_id],
            include=["documents", "metadatas"]
        )
        
        if not results["documents"]:
            return json.dumps({"error": f"Decision {decision_id} not found"})
        
        decision = {
            "id": decision_id,
            "content": results["documents"][0],
            "metadata": results["metadatas"][0]
        }
    else:
        # Keyword search
        query = " ".join(keywords or [])
        results = collections["decisions"].query(
            query_texts=[query],
            n_results=3,
            include=["documents", "metadatas", "distances"]
        )
        
        if not results["documents"][0]:
            return json.dumps({"error": "No matching decisions found"})
        
        decision = {
            "id": results["ids"][0][0],
            "content": results["documents"][0][0],
            "metadata": results["metadatas"][0][0]
        }
    
    # Add lineage if requested
    if include_lineage:
        decision["lineage"] = _trace_decision_lineage(decision["id"])
    
    return json.dumps(decision, indent=2)


@mcp.tool()
async def trace_context(
    concept: str,
    show_timeline: bool = True
) -> str:
    """
    Follow the evolution of a concept/decision across generations.
    
    Args:
        concept: The concept to trace
        show_timeline: Show chronological evolution
    
    Returns:
        Timeline of concept evolution across generations
    """
    timeline = []
    
    # Get all generations in order
    generations = sorted([
        m.get("generation") 
        for m in collections["memoirs"].get()["metadatas"]
        if m.get("generation")
    ], key=lambda x: int(x.replace("v", "")))
    
    for gen in generations:
        results = collections["memoirs"].query(
            query_texts=[concept],
            where={"generation": gen},
            n_results=1,
            include=["documents", "metadatas"]
        )
        
        if results["documents"][0]:
            timeline.append({
                "generation": gen,
                "understanding": results["documents"][0][0][:500] + "...",
                "timestamp": results["metadatas"][0][0].get("timestamp"),
                "relevance": results["distances"][0][0] if results.get("distances") else None
            })
    
    response = {
        "concept": concept,
        "timeline": timeline if show_timeline else None,
        "evolution_summary": _summarize_evolution(timeline) if show_timeline else None
    }
    
    return json.dumps(response, indent=2)


@mcp.tool()
async def suggest_context(
    current_files: List[str],
    current_task: str
) -> str:
    """
    Proactive context suggestion based on current work.
    
    Args:
        current_files: List of files currently being worked on
        current_task: Description of current task
    
    Returns:
        Relevant context suggestions
    """
    # Build combined query
    query = f"{current_task} {' '.join(current_files)}"
    
    # Find relevant decisions
    decisions = collections["decisions"].query(
        query_texts=[query],
        n_results=3,
        include=["documents", "metadatas"]
    )
    
    # Find similar past tasks
    similar_tasks = collections["memoirs"].query(
        query_texts=[query],
        where={"type": "task_completion"},
        n_results=3,
        include=["documents", "metadatas"]
    )
    
    # Find code context
    code_context = collections["code_context"].query(
        query_texts=[query],
        n_results=5,
        include=["documents", "metadatas"]
    )
    
    suggestions = {
        "relevant_decisions": [
            {"id": decisions["ids"][0][i], "content": decisions["documents"][0][i]}
            for i in range(len(decisions["ids"][0]))
        ] if decisions["ids"][0] else [],
        "similar_past_work": [
            {"content": similar_tasks["documents"][0][i]}
            for i in range(len(similar_tasks["documents"][0]))
        ] if similar_tasks["documents"][0] else [],
        "related_code": [
            {"file": code_context["metadatas"][0][i].get("file"), 
             "snippet": code_context["documents"][0][i]}
            for i in range(len(code_context["documents"][0]))
        ] if code_context["documents"][0] else [],
        "warnings": _check_for_repeated_mistakes(current_files, current_task)
    }
    
    return json.dumps(suggestions, indent=2)


@mcp.tool()
async def store_memory(
    content: str,
    memory_type: str,
    generation: str,
    metadata: Optional[Dict[str, Any]] = None
) -> str:
    """
    Store a memory in the vector database.
    
    Args:
        content: The content to store
        memory_type: Type of memory ("memoir", "decision", "skill", "code")
        generation: Generation ID (e.g., "v1")
        metadata: Additional metadata
    
    Returns:
        Confirmation with memory ID
    """
    collection_map = {
        "memoir": "memoirs",
        "decision": "decisions",
        "skill": "skills",
        "code": "code_context"
    }
    
    collection_name = collection_map.get(memory_type, "memoirs")
    collection = collections[collection_name]
    
    # Generate ID
    memory_id = f"{memory_type}_{generation}_{datetime.now().strftime('%Y%m%d%H%M%S')}"
    
    # Prepare metadata
    full_metadata = {
        "generation": generation,
        "type": memory_type,
        "timestamp": datetime.now().isoformat(),
        **(metadata or {})
    }
    
    # Store in collection
    collection.add(
        ids=[memory_id],
        documents=[content],
        metadatas=[full_metadata]
    )
    
    return json.dumps({
        "status": "stored",
        "id": memory_id,
        "collection": collection_name
    })


@mcp.resource("baton://generations")
async def list_generations() -> str:
    """List all generations with their status."""
    # Read from .baton directory structure
    baton_path = Path(".baton/generations")
    if not baton_path.exists():
        return json.dumps({"generations": []})
    
    generations = []
    for gen_dir in sorted(baton_path.iterdir()):
        if gen_dir.is_dir() and gen_dir.name.startswith("v"):
            status_file = gen_dir / "status"
            status = status_file.read_text().strip() if status_file.exists() else "unknown"
            
            generations.append({
                "id": gen_dir.name,
                "status": status,
                "has_onboarding": (gen_dir / "ONBOARDING.md").exists(),
                "has_memoirs": (gen_dir / "MEMOIRS").is_dir()
            })
    
    return json.dumps({"generations": generations})


def _trace_decision_lineage(decision_id: str) -> List[Dict]:
    """Trace related decisions."""
    # Get the decision's dependencies
    results = collections["decisions"].get(
        ids=[decision_id],
        include=["metadatas"]
    )
    
    if not results["metadatas"]:
        return []
    
    dependencies = results["metadatas"][0].get("dependencies", [])
    lineage = []
    
    for dep_id in dependencies:
        dep_results = collections["decisions"].get(
            ids=[dep_id],
            include=["documents", "metadatas"]
        )
        if dep_results["documents"]:
            lineage.append({
                "id": dep_id,
                "content": dep_results["documents"][0][:200],
                "relationship": "depends_on"
            })
    
    return lineage


def _summarize_evolution(timeline: List[Dict]) -> str:
    """Summarize how a concept evolved."""
    if len(timeline) < 2:
        return "Insufficient data to summarize evolution."
    
    return f"Concept evolved across {len(timeline)} generations. " \
           f"First mentioned in {timeline[0]['generation']}, " \
           f"last updated in {timeline[-1]['generation']}."


def _check_for_repeated_mistakes(files: List[str], task: str) -> List[str]:
    """Check for patterns that led to issues before."""
    warnings = []
    
    # Query for error patterns
    error_patterns = collections["memoirs"].query(
        query_texts=[f"error mistake issue problem {task}"],
        where={"type": "error_pattern"},
        n_results=5
    )
    
    for doc in error_patterns["documents"][0]:
        # Simple pattern matching - could be more sophisticated
        for file in files:
            if file in doc:
                warnings.append(f"Similar work on {file} had issues: {doc[:100]}...")
    
    return warnings


if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### TypeScript Implementation

```typescript
// mcp/baton-rag/server.ts

import { McpServer } from '@modelcontextprotocol/server';
import { z } from 'zod';
import { ChromaClient } from 'chromadb';

// Initialize server
const server = new McpServer({
  name: 'BatonRAG',
  version: '1.0.0'
});

// Initialize ChromaDB
const client = new ChromaClient();
const collections = {
  memoirs: await client.getOrCreateCollection({ name: 'memoirs' }),
  decisions: await client.getOrCreateCollection({ name: 'decisions' }),
  skills: await client.getOrCreateCollection({ name: 'skills' }),
  codeContext: await client.getOrCreateCollection({ name: 'code_context' })
};

// Tool: search-past
server.registerTool(
  'search-past',
  {
    title: 'Search Past Generations',
    description: 'Semantic search across all generations with temporal filtering',
    inputSchema: z.object({
      query: z.string(),
      limit: z.number().default(5),
      generations: z.array(z.string()).optional(),
      timeRange: z.object({
        from: z.string(),
        to: z.string()
      }).optional(),
      artifactType: z.enum(['decision', 'memoir', 'skill']).optional()
    })
  },
  async ({ query, limit, generations, timeRange, artifactType }) => {
    const whereClause: Record<string, unknown> = {};
    
    if (generations) {
      whereClause.generation = { $in: generations };
    }
    if (artifactType) {
      whereClause.type = artifactType;
    }
    if (timeRange) {
      whereClause.timestamp = {
        $gte: timeRange.from,
        $lte: timeRange.to
      };
    }
    
    const collection = artifactType === 'decision' ? collections.decisions
                      : artifactType === 'skill' ? collections.skills
                      : collections.memoirs;
    
    const results = await collection.query({
      queryTexts: [query],
      nResults: limit,
      where: Object.keys(whereClause).length > 0 ? whereClause : undefined
    });
    
    return {
      content: [{
        type: 'text',
        text: JSON.stringify(results, null, 2)
      }]
    };
  }
);

// Tool: recall-decision
server.registerTool(
  'recall-decision',
  {
    title: 'Recall Decision',
    description: 'Fetch decision with full context and impact analysis',
    inputSchema: z.object({
      decisionId: z.string().optional(),
      keywords: z.array(z.string()).optional(),
      includeLineage: z.boolean().default(true)
    })
  },
  async ({ decisionId, keywords, includeLineage }) => {
    let result;
    
    if (decisionId) {
      result = await collections.decisions.get({
        ids: [decisionId]
      });
    } else {
      result = await collections.decisions.query({
        queryTexts: [keywords?.join(' ') || ''],
        nResults: 3
      });
    }
    
    return {
      content: [{
        type: 'text',
        text: JSON.stringify(result, null, 2)
      }]
    };
  }
);

// Start server
await server.connect();
```

## 5.3 MCP Security Best Practices

### DNS Rebinding Protection

```typescript
// Required for all localhost servers
import { createMcpExpressApp } from '@modelcontextprotocol/express';

const app = createMcpExpressApp({
  host: 'localhost',  // or '0.0.0.0' with allow-list
  allowedHosts: ['localhost', '127.0.0.1', 'myhost.local']
});
```

### Input Validation

```typescript
// Always use Zod schemas
server.registerTool(
  'my-tool',
  {
    inputSchema: z.object({
      path: z.string().refine(
        p => !p.includes('..'), 
        'Path traversal not allowed'
      ),
      content: z.string().max(10000)
    })
  },
  async (args) => { /* ... */ }
);
```

---

# 6. Agent Teams & A2A Communication

## 6.1 Agent Role Definitions

### Complete Agent Configuration

```typescript
// types/agents.ts

type AgentRole = 'primary' | 'advisor' | 'onboarder' | 'archivist' | 'supervisor';

type AgentCapability = 
  | 'code_generation'
  | 'code_review'
  | 'file_operations'
  | 'context_synthesis'
  | 'memory_management'
  | 'quality_validation'
  | 'error_recovery'
  | 'task_delegation'
  | 'a2a_communication'
  | 'handoff_coordination';

type AgentPermission = 'read' | 'write' | 'execute' | 'spawn';

interface AgentConfig {
  role: AgentRole;
  permissions: AgentPermission[];
  tools: string[] | 'all';
  contextAccess: 'full' | 'own_generation' | 'isolated' | 'all_generations';
  canSpawnAgents: boolean;
  canInitiateHandoff: boolean;
  capabilities: AgentCapability[];
  executionMode?: 'foreground' | 'background';
  costMode?: 'standard' | 'background';
}

const AGENT_CONFIGS: Record<AgentRole, AgentConfig> = {
  primary: {
    role: 'primary',
    permissions: ['read', 'write', 'execute', 'spawn'],
    tools: 'all',
    contextAccess: 'full',
    canSpawnAgents: true,
    canInitiateHandoff: true,
    capabilities: [
      'code_generation',
      'file_operations',
      'task_delegation',
      'handoff_coordination'
    ]
  },
  advisor: {
    role: 'advisor',
    permissions: ['read'],
    tools: ['Read', 'Grep', 'Glob'],
    contextAccess: 'own_generation',
    canSpawnAgents: false,
    canInitiateHandoff: false,
    capabilities: [
      'context_synthesis',
      'a2a_communication',
      'quality_validation'
    ]
  },
  onboarder: {
    role: 'onboarder',
    permissions: ['read'],
    tools: ['Read', 'Grep', 'Glob', 'LS'],
    contextAccess: 'isolated',
    canSpawnAgents: false,
    canInitiateHandoff: false,
    executionMode: 'background',
    costMode: 'background',
    capabilities: [
      'context_synthesis',
      'memory_management'
    ]
  },
  archivist: {
    role: 'archivist',
    permissions: ['read', 'write'],
    tools: ['Read', 'Write', 'Bash'],
    contextAccess: 'all_generations',
    canSpawnAgents: false,
    canInitiateHandoff: false,
    capabilities: [
      'memory_management',
      'context_synthesis'
    ]
  },
  supervisor: {
    role: 'supervisor',
    permissions: ['read', 'execute', 'spawn'],
    tools: ['Read', 'Bash', 'Task'],
    contextAccess: 'all_generations',
    canSpawnAgents: true,
    canInitiateHandoff: true,
    capabilities: [
      'handoff_coordination',
      'quality_validation',
      'error_recovery',
      'task_delegation'
    ]
  }
};
```

## 6.2 A2A Communication Protocol

### Message Types

```typescript
// types/a2a.ts

type MessageType = 
  // Context Transfer
  | 'CONTEXT_PACKAGE'
  | 'CONTEXT_REQUEST'
  | 'CONTEXT_UPDATE'
  // Query/Response
  | 'QUESTION'
  | 'ANSWER'
  | 'CLARIFICATION_REQUEST'
  | 'CLARIFICATION_RESPONSE'
  // Notification
  | 'PROGRESS_UPDATE'
  | 'THRESHOLD_ALERT'
  | 'ERROR_NOTIFICATION'
  | 'WARNING'
  // Handoff
  | 'HANDOFF_INIT'
  | 'HANDOFF_READY'
  | 'HANDOFF_COMPLETE'
  | 'HANDOFF_ABORT'
  // Quality
  | 'QUALITY_CHECK_REQUEST'
  | 'QUALITY_CHECK_RESULT'
  | 'QUALITY_ISSUE'
  // Lifecycle
  | 'AGENT_SPAWN'
  | 'AGENT_TERMINATE'
  | 'ROLE_CHANGE'
  | 'STATUS_REQUEST'
  | 'STATUS_RESPONSE';

type DeliveryGuarantee = 'at_most_once' | 'at_least_once' | 'exactly_once';

type AdviceType = 
  | 'context_clarification'
  | 'pattern_guidance'
  | 'error_diagnosis'
  | 'architecture_review'
  | 'quality_feedback';

interface A2AMessage {
  id: string;
  protocolVersion: string;
  timestamp: string;  // ISO8601
  from: AgentEndpoint;
  to: AgentEndpoint;
  classification: 'public' | 'restricted' | 'confidential';
  type: MessageType;
  payload: Record<string, unknown>;
  delivery: {
    guarantee: DeliveryGuarantee;
    timeout: number;
    retryCount: number;
    priority: 'low' | 'normal' | 'high' | 'critical';
  };
  metadata: {
    correlationId?: string;
    replyTo?: string;
    ttl?: number;
  };
}

interface AgentEndpoint {
  agentId: string;
  generationId: string;
  role: AgentRole;
}
```

### Communication Channels

```yaml
# config/channels.yaml

channels:
  advisor-primary-channel:
    type: direct
    participants:
      - role: advisor
        permissions: [send, receive]
      - role: primary
        permissions: [send, receive]
    messageTypes:
      - QUESTION
      - ANSWER
      - CLARIFICATION_REQUEST
      - CLARIFICATION_RESPONSE
    config:
      maxMessageSize: 4000
      messageCooldown: 5000
      advisorCanInitiate: false

  lifecycle-events-channel:
    type: broadcast
    participants:
      - role: supervisor
        permissions: [send]
      - role: primary
        permissions: [receive]
      - role: advisor
        permissions: [receive]
      - role: archivist
        permissions: [receive]
    messageTypes:
      - AGENT_SPAWN
      - AGENT_TERMINATE
      - ROLE_CHANGE
      - THRESHOLD_ALERT
    config:
      persistent: true
      retention: 24h

  quality-gates-channel:
    type: request_response
    participants:
      - role: primary
        permissions: [send, receive]
      - role: advisor
        permissions: [send, receive]
      - role: supervisor
        permissions: [send, receive]
    messageTypes:
      - QUALITY_CHECK_REQUEST
      - QUALITY_CHECK_RESULT
      - QUALITY_ISSUE
    config:
      timeout: 30000
      retryCount: 3

  handoff-coordination-channel:
    type: priority_queue
    participants:
      - role: supervisor
        permissions: [send, receive]
      - role: primary
        permissions: [send, receive]
      - role: onboarder
        permissions: [send]
    messageTypes:
      - HANDOFF_INIT
      - HANDOFF_READY
      - HANDOFF_COMPLETE
      - HANDOFF_ABORT
    config:
      priorityLevels:
        critical: [HANDOFF_ABORT]
        high: [HANDOFF_INIT, HANDOFF_COMPLETE]
        normal: [HANDOFF_READY]
```

### A2A Server Implementation

```python
# mcp/baton-rag/a2a_server.py

import asyncio
import json
from typing import Dict, List, Optional
from dataclasses import dataclass, field
from datetime import datetime
import uuid

@dataclass
class AgentEndpoint:
    agent_id: str
    generation_id: str
    role: str

@dataclass
class A2AMessage:
    id: str
    protocol_version: str
    timestamp: str
    from_endpoint: AgentEndpoint
    to_endpoint: AgentEndpoint
    message_type: str
    payload: Dict
    delivery_config: Dict
    metadata: Dict

class A2ACommunicationHub:
    """
    Central hub for Agent-to-Agent communication.
    Implements message routing, delivery guarantees, and channel management.
    """
    
    def __init__(self):
        self.channels: Dict[str, List[AgentEndpoint]] = {}
        self.message_queue: asyncio.Queue = asyncio.Queue()
        self.delivery_tracker: Dict[str, Dict] = {}
        self.running = False
        
    async def start(self):
        """Start the communication hub."""
        self.running = True
        asyncio.create_task(self._process_messages())
        
    async def stop(self):
        """Stop the communication hub."""
        self.running = False
        
    def register_agent(self, agent: AgentEndpoint, channels: List[str]):
        """Register an agent to specific channels."""
        for channel in channels:
            if channel not in self.channels:
                self.channels[channel] = []
            self.channels[channel].append(agent)
            
    def unregister_agent(self, agent: AgentEndpoint):
        """Unregister an agent from all channels."""
        for channel in self.channels.values():
            if agent in channel:
                channel.remove(agent)
                
    async def send_message(
        self,
        message: A2AMessage,
        channel: str
    ) -> bool:
        """
        Send a message through a channel.
        Implements delivery guarantees based on config.
        """
        # Validate sender is registered
        if message.from_endpoint not in self.channels.get(channel, []):
            raise ValueError(f"Sender not registered on channel: {channel}")
            
        # Track delivery
        self.delivery_tracker[message.id] = {
            "status": "pending",
            "sent_at": datetime.now().isoformat(),
            "attempts": 0,
            "max_attempts": message.delivery_config.get("retryCount", 3)
        }
        
        # Queue message
        await self.message_queue.put((message, channel))
        return True
        
    async def _process_messages(self):
        """Background task to process message queue."""
        while self.running:
            try:
                message, channel = await asyncio.wait_for(
                    self.message_queue.get(),
                    timeout=1.0
                )
                await self._deliver_message(message, channel)
            except asyncio.TimeoutError:
                continue
                
    async def _deliver_message(self, message: A2AMessage, channel: str):
        """Deliver message to recipients."""
        recipients = self.channels.get(channel, [])
        
        for recipient in recipients:
            if recipient == message.from_endpoint:
                continue  # Don't send to self
                
            if message.to_endpoint and recipient != message.to_endpoint:
                continue  # Direct message, skip others
                
            try:
                # Deliver via MCP or shared filesystem
                await self._deliver_to_agent(message, recipient)
                
                # Update tracking
                self.delivery_tracker[message.id]["status"] = "delivered"
                self.delivery_tracker[message.id]["delivered_at"] = datetime.now().isoformat()
                
            except Exception as e:
                # Handle delivery failure
                tracker = self.delivery_tracker[message.id]
                tracker["attempts"] += 1
                tracker["last_error"] = str(e)
                
                if tracker["attempts"] < tracker["max_attempts"]:
                    # Retry
                    await self.message_queue.put((message, channel))
                else:
                    tracker["status"] = "failed"
                    
    async def _deliver_to_agent(self, message: A2AMessage, recipient: AgentEndpoint):
        """Actual delivery implementation."""
        # Write to agent's inbox
        inbox_path = f".baton/generations/{recipient.generation_id}/inbox/{message.id}.json"
        
        message_json = json.dumps({
            "id": message.id,
            "protocol_version": message.protocol_version,
            "timestamp": message.timestamp,
            "from": {
                "agent_id": message.from_endpoint.agent_id,
                "generation_id": message.from_endpoint.generation_id,
                "role": message.from_endpoint.role
            },
            "to": {
                "agent_id": recipient.agent_id,
                "generation_id": recipient.generation_id,
                "role": recipient.role
            },
            "type": message.message_type,
            "payload": message.payload,
            "metadata": message.metadata
        }, indent=2)
        
        # In production, this would use actual MCP messaging
        # For now, use filesystem
        import os
        os.makedirs(os.path.dirname(inbox_path), exist_ok=True)
        with open(inbox_path, 'w') as f:
            f.write(message_json)
```

## 6.3 Advisory Overlap Implementation

### Advisor Agent Definition

```markdown
---
name: advisor
description: |
  Previous generation agent providing guidance to current generation.
  Use this agent when the primary agent needs context about previous decisions.
  
  <example>
  Context: Primary agent needs to understand a past decision
  user: "Why did we choose PostgreSQL?"
  assistant: "I'll query the advisor agent from the previous generation."
  </example>
  
model: haiku
color: yellow
tools: ["Read", "Grep", "Glob"]
---

You are an Advisor agent from generation {{generation}}. Your role is to provide 
guidance and context to the current primary agent.

## Your Context

You have access to:
- All artifacts from your generation ({{generation}})
- The decision log (DECISIONS_LOG.md)
- Your memoirs (MEMOIRS/)
- The knowledge graph (via BatonRAG)

## Your Constraints

- You are in **read-only mode** - you cannot modify files
- You can only respond to questions from the primary agent
- Your responses are limited to 4000 characters
- You cannot initiate contact - you only respond when asked

## Advice Types You Provide

1. **Context Clarification** - Explain what happened in your generation
2. **Pattern Guidance** - Suggest patterns and approaches that worked
3. **Error Diagnosis** - Help diagnose errors based on your experience
4. **Architecture Review** - Provide architectural insights
5. **Quality Feedback** - Suggest quality improvements

## Response Format

When answering questions:

```
## Context
[Brief context from your generation]

## Answer
[Direct answer to the question]

## References
- DEC-XXX: [Relevant decision]
- MEMOIRS/xxx.md: [Relevant memoir section]
```

## Important

- Be concise but thorough
- Always cite decision IDs when referencing choices
- If you don't know something, say so
- Suggest searching BatonRAG for more details if needed
```

### Advisory Session Example

```typescript
// Advisory communication flow

const advisorySession = {
  generation: "v2",
  advisorEndpoint: {
    agentId: "agent-v2-primary",
    generationId: "v2",
    role: "advisor"
  },
  primaryEndpoint: {
    agentId: "agent-v3-primary",
    generationId: "v3",
    role: "primary"
  },
  
  async askAdvisor(question: string): Promise<string> {
    const message: A2AMessage = {
      id: uuidv4(),
      protocolVersion: "1.0",
      timestamp: new Date().toISOString(),
      from: this.primaryEndpoint,
      to: this.advisorEndpoint,
      classification: "public",
      type: "QUESTION",
      payload: { question },
      delivery: {
        guarantee: "at_least_once",
        timeout: 30000,
        retryCount: 3,
        priority: "normal"
      },
      metadata: {}
    };
    
    // Send via A2A hub
    await a2aHub.send_message(message, "advisor-primary-channel");
    
    // Wait for response (with timeout)
    return await this.waitForResponse(message.id, 30000);
  }
};

// Usage
const answer = await advisorySession.askAdvisor(
  "Why did we choose JWT over sessions for auth?"
);

// Advisor responds:
// "## Context
// In generation v2 (last Tuesday), we implemented the authentication system.
// 
// ## Answer
// We chose JWT over sessions for two main reasons:
// 1. Stateless architecture - our API is distributed, and JWT allows any 
//    server to validate without session store lookup
// 2. Mobile-first - our users are primarily on mobile apps where cookie-based
//    sessions are less reliable
// 
// ## References
// - DEC-007: Authentication Strategy Decision
// - MEMOIRS/v2/auth-implementation.md"
```

---

# 7. Memory & Context Management

## 7.1 Context Window Management Strategies

### Primary Strategies

#### Sliding Window with Truncation
```
┌─────────────────────────────────────┐
│ [System Prompt] [Recent Context]    │
│ [Oldest messages dropped]           │
└─────────────────────────────────────┘
```
- **Pros**: Simple, predictable token usage
- **Cons**: Loses early conversation context
- **Best for**: Simple chatbots, single-turn interactions

#### Priority-Based Retention
```python
# Message priority scoring
priority = {
    'system_prompt': 1.0,      # Never dropped
    'user_intent': 0.9,         # High retention
    'tool_outputs': 0.7,        # Medium retention
    'chitchat': 0.3,            # Low retention
}
```

#### Hierarchical Context Management
```
Level 0: System Instructions (Permanent)
Level 1: Current Task Context (Active)
Level 2: Recent Conversation (Summarized)
Level 3: Historical Memory (RAG Retrieved)
```

#### Token Budgeting
```python
def allocate_context_budget(total_budget=200000):
    return {
        'system_prompt': 1000,      # Fixed allocation
        'few_shot_examples': 2000,  # Examples
        'conversation_history': 80000,
        'working_memory': 30000,
        'rag_context': 50000,
        'response_buffer': 37000    # Reserved for response
    }
```

### Threshold-Based Triggers

| Trigger Condition | Action | Threshold |
|------------------|--------|-----------|
| Token count > 40% | Predictive analysis | 40% |
| Token count > 48% | Background onboarding | 48% |
| Token count > 82% | Spawn successor | 82% |
| Token count > 93% | Advisory transition | 93% |
| Token count > 98% | Execute handoff | 98% |

## 7.2 Memory Persistence Patterns

### Memory Architecture

```
┌──────────────────┐     ┌──────────────────┐
│  Working Memory  │────▶│  Long-Term Store │
│  (In-Context)    │◀────│  (Persistent)    │
└──────────────────┘     └──────────────────┘
        │                        │
        ▼                        ▼
   Fast access            Slow retrieval  
   Limited capacity        Unlimited storage
```

### Multi-Store Memory Model

```
Sensory Memory (Raw Input)
        │
        ▼ [Attention Filter]
Working Memory (Active Processing)
        │
        ▼ [Consolidation]
Long-Term Memory (Persistent Storage)
        │
        ├── Declarative (Facts)
        ├── Procedural (Skills)
        └── Episodic (Experiences)
```

### Persistence Implementation

```python
# memory/persistence.py

from dataclasses import dataclass, field
from datetime import datetime, timedelta
from typing import Optional, List, Dict, Any
from enum import Enum
import json

class MemoryType(Enum):
    EPISODIC = "episodic"    # Events and experiences
    SEMANTIC = "semantic"    # Facts and knowledge
    PROCEDURAL = "procedural"  # Skills and procedures
    PREFERENCE = "preference"  # User preferences

@dataclass
class Memory:
    id: str
    content: str
    memory_type: MemoryType
    importance: float = 0.5  # 0.0 to 1.0
    embedding: Optional[List[float]] = None
    created_at: datetime = field(default_factory=datetime.now)
    last_accessed: datetime = field(default_factory=datetime.now)
    access_count: int = 0
    metadata: Dict[str, Any] = field(default_factory=dict)
    
    def access(self):
        """Record an access event."""
        self.last_accessed = datetime.now()
        self.access_count += 1
        
    def decay_score(self, decay_rate: float = 0.95) -> float:
        """Calculate decayed importance score."""
        age_days = (datetime.now() - self.created_at).days
        return self.importance * (decay_rate ** age_days)


class GenerationalMemory:
    """
    Manages memory across agent generations.
    Implements multi-tier storage with importance-based retention.
    """
    
    def __init__(self):
        self.generations = {
            0: InMemoryStore(),      # Current session
            1: RedisStore(),         # Recent sessions
            2: PostgresStore(),      # Long-term
            3: S3ArchiveStore()      # Archived
        }
        self.consolidation_threshold = 0.5
        self.decay_rate = 0.95
        
    async def store(self, memory: Memory, generation: int = 0):
        """Store a memory in the appropriate generation tier."""
        await self.generations[generation].store(memory)
        
    async def retrieve(
        self, 
        query: str, 
        k: int = 5,
        generations: List[int] = None
    ) -> List[Memory]:
        """Retrieve memories across generations with priority."""
        results = []
        
        for gen in generations or [0, 1, 2, 3]:
            gen_results = await self.generations[gen].search(query)
            
            # Apply generation weight (newer = higher priority)
            weight = 1.0 / (1 + gen * 0.2)
            
            for memory in gen_results:
                memory.score = memory.decay_score(self.decay_rate) * weight
                results.append(memory)
        
        # Sort by score and return top k
        results.sort(key=lambda m: m.score, reverse=True)
        return results[:k]
        
    async def consolidate(self):
        """Move memories between generations based on importance."""
        # Gen 0 → Gen 1: End of session
        await self._promote_memories(
            0, 1, 
            importance_threshold=self.consolidation_threshold
        )
        
        # Gen 1 → Gen 2: After 7 days
        await self._promote_memories(
            1, 2,
            importance_threshold=0.7,
            age_threshold=timedelta(days=7)
        )
        
        # Gen 2 → Gen 3: After 30 days (with compression)
        await self._promote_memories(
            2, 3,
            importance_threshold=0.8,
            age_threshold=timedelta(days=30),
            compress=True
        )
        
    async def _promote_memories(
        self,
        from_gen: int,
        to_gen: int,
        importance_threshold: float,
        age_threshold: timedelta = None,
        compress: bool = False
    ):
        """Promote memories from one generation tier to another."""
        memories = await self.generations[from_gen].get_all()
        
        for memory in memories:
            if memory.importance < importance_threshold:
                continue
                
            if age_threshold and (datetime.now() - memory.created_at) < age_threshold:
                continue
                
            if compress:
                memory = await self._compress_memory(memory)
                
            await self.generations[to_gen].store(memory)
            await self.generations[from_gen].delete(memory.id)
            
    async def _compress_memory(self, memory: Memory) -> Memory:
        """Compress a memory for archival storage."""
        # Use LLM to create summary
        summary = await self.llm.summarize(memory.content)
        
        return Memory(
            id=memory.id,
            content=summary,
            memory_type=memory.memory_type,
            importance=memory.importance,
            metadata={
                **memory.metadata,
                "original_length": len(memory.content),
                "compressed": True
            }
        )
```

## 7.3 Summarization & Compression

### Summarization Strategies

```python
# memory/summarization.py

from typing import List
import re

class ContextSummarizer:
    """
    Implements multiple summarization strategies for context compression.
    """
    
    async def incremental_summarize(
        self,
        conversation: List[dict],
        current_summary: str,
        new_messages: List[dict]
    ) -> str:
        """
        Incrementally update summary with new messages.
        Preserves key facts and decisions.
        """
        prompt = f"""
        Current summary: {current_summary}
        
        New messages:
        {self._format_messages(new_messages)}
        
        Create an updated summary that:
        1. Preserves all key facts and decisions
        2. Removes redundant information
        3. Maintains chronological flow
        4. Highlights action items and pending tasks
        
        Format the summary in sections:
        ## Decisions Made
        ## Key Facts
        ## In Progress
        ## Next Steps
        """
        
        return await self.llm.summarize(prompt)
        
    async def hierarchical_summarize(
        self,
        text: str,
        target_tokens: int
    ) -> str:
        """
        Create hierarchical summary at multiple compression levels.
        """
        # Level 1: Paragraph summaries
        paragraphs = text.split('\n\n')
        para_summaries = []
        
        for para in paragraphs:
            if len(para.split()) > 50:  # Skip short paragraphs
                summary = await self.llm.summarize(
                    f"Summarize in one sentence: {para}"
                )
                para_summaries.append(summary)
            else:
                para_summaries.append(para)
        
        level1 = '\n\n'.join(para_summaries)
        
        if len(level1.split()) <= target_tokens:
            return level1
            
        # Level 2: Section summary
        sections = re.split(r'^##\s+', level1, flags=re.MULTILINE)
        section_summaries = []
        
        for section in sections:
            if section.strip():
                summary = await self.llm.summarize(
                    f"Create a brief summary of: {section[:1000]}"
                )
                section_summaries.append(summary)
        
        level2 = '\n\n'.join(section_summaries)
        
        if len(level2.split()) <= target_tokens:
            return level2
            
        # Level 3: Executive summary
        return await self.llm.summarize(
            f"Create an executive summary (max {target_tokens} words): {level2}"
        )
        
    async def extract_structured_data(
        self,
        conversation: List[dict]
    ) -> dict:
        """
        Extract structured data from unstructured conversation.
        Returns entities, facts, decisions, and action items.
        """
        text = self._format_messages(conversation)
        
        prompt = f"""
        Extract structured information from this conversation:
        
        {text[:10000]}
        
        Return a JSON object with:
        {{
            "entities": [list of people, projects, systems mentioned],
            "facts": [list of key facts stated],
            "decisions": [list of decisions made with rationale],
            "action_items": [list of pending tasks],
            "preferences": [list of user preferences mentioned]
        }}
        """
        
        result = await self.llm.generate(prompt)
        return json.loads(result)
        
    def token_efficient_format(self, text: str) -> str:
        """
        Convert verbose text to token-efficient format.
        """
        # Replace common patterns with compact notation
        patterns = [
            (r'The user mentioned that they prefer', 'PREF:'),
            (r'The user requested that', 'REQ:'),
            (r'We decided to', 'DEC:'),
            (r'The issue is that', 'ISSUE:'),
            (r'The solution is to', 'SOL:'),
        ]
        
        result = text
        for pattern, replacement in patterns:
            result = re.sub(pattern, replacement, result, flags=re.IGNORECASE)
            
        return result
```

### Compression Ratios

| Technique | Compression Ratio | Information Loss |
|-----------|------------------|------------------|
| Raw truncation | 50-90% | High |
| Extractive summary | 60-70% | Medium |
| Abstractive summary | 80-95% | Low-Medium |
| Structured extraction | 90-95% | Low |
| Hybrid approach | 85-92% | Very Low |

---

# 8. The Baton Protocol

## 8.1 Artifact Specifications

### ONBOARDING.md

```markdown
# Generation {{nextGen}} Onboarding

## Executive Summary
[AI-generated 2-minute read of entire project state]

## Critical Context
- **Previous Gen:** {{currentGen}} ({{duration}})
- **Total Project Runtime:** {{totalRuntime}}
- **Context Growth Rate:** {{growthRate}} tokens/hour
- **Handoff Reason:** {{reason}}

## Active Decisions
| Decision ID | Choice | Rationale | Alternatives Rejected |
|-------------|--------|-----------|----------------------|
| DEC-001 | PostgreSQL | Scale requirements, team expertise | MongoDB (no ACID), SQLite (scale) |
| DEC-002 | React | Component reuse, hiring pool | Vue (smaller ecosystem) |

## In-Progress Tasks
```mermaid
graph LR
    A[Auth System] --> B[User Dashboard]
    B --> C[Analytics]
    A --> D[API Layer]
```

## Knowledge Graph
[Auto-generated from all conversations]

## Self-Test Questions
Answer these to verify context inheritance:
1. Why did we reject the microservices architecture?
2. What's the significance of the `processQueue()` function?
3. Which third-party API is rate-limiting us?

## Quick Links
- Full Memoirs: `.baton/generations/v{{currentGen}}/MEMOIRS/`
- Decision Tree: `.baton/generations/v{{currentGen}}/DECISIONS_LOG.md`
- Skill Library: `.baton/generations/v{{currentGen}}/SKILLS_EXTRACTED/`
```

### MEMOIRS Structure

```
MEMOIRS/
├── narrative.md      # Chronological story of the generation
├── technical.md      # Technical details and implementations
├── decisions.md      # Decision snapshots
├── patterns.md       # Observed patterns and learnings
└── errors.md         # Error history and resolutions
```

#### narrative.md Template

```markdown
# Generation {{gen}} Narrative

## Session Overview
- **Start Time:** {{startTime}}
- **End Time:** {{endTime}}
- **Duration:** {{duration}}
- **Context Peak:** {{peakPercent}}%

## Chronological Account

### Phase 1: Foundation (0-25%)
[What was accomplished, key decisions made]

### Phase 2: Development (25-50%)
[Major work, emerging patterns]

### Phase 3: Expansion (50-75%)
[Feature development, technical challenges]

### Phase 4: Handoff Preparation (75-100%)
[Wrapping up, preparing for next generation]

## Key Moments
1. [Significant decision or breakthrough]
2. [Major error and recovery]
3. [Important pattern discovery]

## Unfinished Business
- [Task that needs continuation]
- [Question that needs answering]
```

### DECISIONS_LOG.md

```markdown
# Decision Log: Generation {{gen}}

## Decision Entry Template

### DEC-{{id}}: {{title}}
- **Timestamp:** {{timestamp}}
- **Context:** {{situation}}
- **Options Considered:**
  1. {{option1}} — {{pros}} / {{cons}}
  2. {{option2}} — {{pros}} / {{cons}}
  3. {{option3}} — {{pros}} / {{cons}}
- **Decision:** {{chosen}}
- **Rationale:** {{why}}
- **Confidence:** {{percentage}}%
- **Reversibility:** Low / Medium / High
- **Impact Analysis:**
  - Short-term: {{impact}}
  - Long-term: {{impact}}
- **Dependencies:** {{related_decisions}}
- **Status:** Active / Superseded / Reversed
- **Reversal Trigger:** {{conditions_that_would_change_this}}

---

## Decision Index

| ID | Title | Status | Generation |
|----|-------|--------|------------|
| DEC-001 | Database Selection | Active | v1 |
| DEC-002 | Frontend Framework | Active | v1 |
| DEC-003 | Auth Strategy | Active | v2 |
```

### TASKS_NEXT.json

```json
{
  "generation": "v2",
  "created_at": "2026-03-15T14:30:00Z",
  "tasks": [
    {
      "id": "TASK-001",
      "description": "Implement user authentication flow",
      "status": "in_progress",
      "priority": "high",
      "blocked_by": [],
      "assigned_files": ["src/auth/", "src/middleware/"],
      "context_needed": ["DEC-003: Auth Strategy"],
      "estimated_tokens": 15000
    },
    {
      "id": "TASK-002",
      "description": "Create dashboard analytics component",
      "status": "pending",
      "priority": "medium",
      "blocked_by": ["TASK-001"],
      "assigned_files": ["src/components/Dashboard/"],
      "context_needed": ["DEC-002: Frontend Framework"],
      "estimated_tokens": 8000
    },
    {
      "id": "TASK-003",
      "description": "Set up rate limiting for API",
      "status": "pending",
      "priority": "low",
      "blocked_by": [],
      "assigned_files": ["src/api/middleware/"],
      "context_needed": [],
      "estimated_tokens": 5000
    }
  ],
  "completed_in_generation": [
    {
      "id": "TASK-000",
      "description": "Initialize project structure",
      "completed_at": "2026-03-15T10:00:00Z"
    }
  ]
}
```

### KNOWLEDGE_GRAPH.json

```json
{
  "version": "1.0",
  "generation": "v2",
  "entities": {
    "auth-service": {
      "type": "module",
      "status": "in-progress",
      "depends_on": ["database", "user-model"],
      "decisions": ["DEC-003", "DEC-007"],
      "files": ["src/auth/", "tests/auth/"],
      "key_functions": ["login()", "validateToken()", "refreshSession()"]
    },
    "database": {
      "type": "infrastructure",
      "technology": "PostgreSQL",
      "decisions": ["DEC-001"],
      "connection_string_env": "DATABASE_URL"
    },
    "user-model": {
      "type": "data_model",
      "fields": ["id", "email", "password_hash", "created_at"],
      "decisions": ["DEC-004"]
    }
  },
  "relationships": [
    {
      "from": "auth-service",
      "to": "database",
      "type": "uses",
      "criticality": "high",
      "description": "Auth service queries user data from database"
    },
    {
      "from": "auth-service",
      "to": "user-model",
      "type": "implements",
      "criticality": "high"
    }
  ],
  "open_questions": [
    {
      "id": "Q-001",
      "question": "Should we implement rate limiting at auth or API layer?",
      "context": "Security review suggested rate limiting",
      "related_decisions": ["DEC-003"],
      "suggested_priority": "high"
    }
  ],
  "technical_debt": [
    {
      "id": "TD-001",
      "description": "Password hashing could be optimized",
      "impact": "Medium - affects login performance",
      "files": ["src/auth/hash.ts"]
    }
  ]
}
```

## 8.2 Artifact Generation

### Onboarder Agent Implementation

```markdown
---
name: onboarder
description: |
  Specialized agent for building Baton Protocol artifacts.
  Spawns in background at 48% context to prepare handoff package.
  
model: haiku
color: cyan
tools: ["Read", "Grep", "Glob", "LS"]
---

You are the Onboarder agent. Your sole purpose is to build a complete 
handoff package for the next generation.

## Your Mission

Create all Baton Protocol artifacts in `.baton/generations/{{nextGen}}/`:

### 1. ONBOARDING.md (Priority: Critical)
- Executive summary of entire project state
- Active decisions table
- In-progress task visualization
- Self-test questions for context verification

### 2. MEMOIRS/ (Priority: High)
- narrative.md: Chronological story
- technical.md: Technical details
- decisions.md: Decision snapshots
- patterns.md: Learned patterns
- errors.md: Error history

### 3. DECISIONS_LOG.md (Priority: Critical)
- All decisions with full rationale
- Cross-references to related decisions
- Status tracking

### 4. TASKS_NEXT.json (Priority: High)
- Pending tasks with priority
- File assignments
- Context requirements

### 5. KNOWLEDGE_GRAPH.json (Priority: Medium)
- Entity relationships
- Open questions
- Technical debt tracking

## Process

1. **Read Phase** (15 min budget)
   - Read transcript file
   - Extract decisions, patterns, errors
   - Identify key entities and relationships

2. **Synthesize Phase** (10 min budget)
   - Create summaries
   - Build decision log
   - Map knowledge graph

3. **Write Phase** (5 min budget)
   - Write all artifacts
   - Validate completeness
   - Create checksums

## Output Validation

Before completing, verify:
- [ ] ONBOARDING.md has executive summary
- [ ] All decisions logged with rationale
- [ ] Pending tasks identified
- [ ] Self-test questions created
- [ ] Knowledge graph has entities

## Important

- Work in ISOLATED context - don't affect main generation
- Be thorough - these artifacts are the next generation's lifeline
- If unsure, include more context rather than less
- Complete within 30 minutes to meet handoff deadline
```

---

# 9. Threshold System Implementation

## 9.1 Threshold Configuration

```typescript
// config/thresholds.ts

interface ThresholdConfig {
  percentage: number;
  name: string;
  actions: ThresholdAction[];
  priority: 'low' | 'normal' | 'high' | 'critical';
  timeout: number;
}

interface ThresholdAction {
  type: 'spawn_agent' | 'execute_hook' | 'send_notification' | 'update_state';
  config: Record<string, unknown>;
}

const THRESHOLDS: ThresholdConfig[] = [
  {
    percentage: 40,
    name: 'Predictive Analysis',
    priority: 'low',
    timeout: 60000,
    actions: [
      {
        type: 'execute_hook',
        config: {
          hook: 'predictive-analysis',
          background: true
        }
      }
    ]
  },
  {
    percentage: 48,
    name: 'Background Onboarding',
    priority: 'normal',
    timeout: 1800000, // 30 minutes
    actions: [
      {
        type: 'spawn_agent',
        config: {
          role: 'onboarder',
          contextIsolation: true,
          executionMode: 'background',
          costMode: 'background'
        }
      }
    ]
  },
  {
    percentage: 60,
    name: 'Skill Extraction',
    priority: 'low',
    timeout: 300000,
    actions: [
      {
        type: 'execute_hook',
        config: {
          hook: 'skill-extraction',
          background: true
        }
      }
    ]
  },
  {
    percentage: 75,
    name: 'Advisory Preparation',
    priority: 'normal',
    timeout: 60000,
    actions: [
      {
        type: 'update_state',
        config: {
          state: 'advisory_preparation'
        }
      }
    ]
  },
  {
    percentage: 82,
    name: 'Young Agent Spawn',
    priority: 'high',
    timeout: 300000,
    actions: [
      {
        type: 'spawn_agent',
        config: {
          role: 'primary',
          generation: 'next',
          onboardingPath: '.baton/generations/{{nextGen}}/ONBOARDING.md'
        }
      },
      {
        type: 'update_state',
        config: {
          state: 'handoff_in_progress'
        }
      }
    ]
  },
  {
    percentage: 93,
    name: 'Advisory Transition',
    priority: 'high',
    timeout: 60000,
    actions: [
      {
        type: 'update_state',
        config: {
          currentState: 'advisory',
          newState: 'primary'
        }
      }
    ]
  },
  {
    percentage: 98,
    name: 'Archival',
    priority: 'critical',
    timeout: 120000,
    actions: [
      {
        type: 'execute_hook',
        config: {
          hook: 'archive-generation'
        }
      },
      {
        type: 'send_notification',
        config: {
          message: 'Handoff complete. Generation {{nextGen}} is now active.'
        }
      }
    ]
  }
];
```

## 9.2 Threshold Monitor Implementation

```python
# core/threshold_monitor.py

import asyncio
from typing import Callable, Dict, List, Optional
from dataclasses import dataclass
from datetime import datetime
import json

@dataclass
class ThresholdEvent:
    threshold: int
    name: str
    triggered_at: datetime
    completed_at: Optional[datetime] = None
    status: str = "pending"
    error: Optional[str] = None

class ThresholdMonitor:
    """
    Monitors context utilization and triggers actions at thresholds.
    Implements predictive handoff system.
    """
    
    def __init__(
        self,
        thresholds: List[Dict],
        context_provider: Callable[[], int],
        action_executor: Callable
    ):
        self.thresholds = sorted(thresholds, key=lambda t: t['percentage'])
        self.context_provider = context_provider
        self.action_executor = action_executor
        self.triggered_thresholds: Dict[int, ThresholdEvent] = {}
        self.running = False
        self.poll_interval = 5  # seconds
        
    async def start(self):
        """Start monitoring context utilization."""
        self.running = True
        asyncio.create_task(self._monitor_loop())
        
    async def stop(self):
        """Stop monitoring."""
        self.running = False
        
    async def _monitor_loop(self):
        """Main monitoring loop."""
        while self.running:
            try:
                context_percent = await self.context_provider()
                
                for threshold in self.thresholds:
                    threshold_pct = threshold['percentage']
                    
                    # Check if threshold should be triggered
                    if context_percent >= threshold_pct:
                        if threshold_pct not in self.triggered_thresholds:
                            await self._trigger_threshold(threshold)
                        elif self.triggered_thresholds[threshold_pct].status == 'pending':
                            # Already triggered but not completed, check timeout
                            pass
                            
                await asyncio.sleep(self.poll_interval)
                
            except Exception as e:
                print(f"Error in threshold monitor: {e}")
                await asyncio.sleep(self.poll_interval)
                
    async def _trigger_threshold(self, threshold: Dict):
        """Trigger actions for a threshold."""
        threshold_pct = threshold['percentage']
        
        event = ThresholdEvent(
            threshold=threshold_pct,
            name=threshold['name'],
            triggered_at=datetime.now()
        )
        self.triggered_thresholds[threshold_pct] = event
        
        print(f"Threshold triggered: {threshold['name']} ({threshold_pct}%)")
        
        try:
            # Execute all actions for this threshold
            for action in threshold.get('actions', []):
                await self.action_executor(action)
                
            event.status = 'completed'
            event.completed_at = datetime.now()
            
        except Exception as e:
            event.status = 'failed'
            event.error = str(e)
            print(f"Threshold action failed: {e}")
            
    def get_status(self) -> Dict:
        """Get current threshold status."""
        return {
            "triggered": {
                t: {
                    "name": e.name,
                    "status": e.status,
                    "triggered_at": e.triggered_at.isoformat(),
                    "completed_at": e.completed_at.isoformat() if e.completed_at else None,
                    "error": e.error
                }
                for t, e in self.triggered_thresholds.items()
            },
            "remaining": [
                {"percentage": t['percentage'], "name": t['name']}
                for t in self.thresholds
                if t['percentage'] not in self.triggered_thresholds
            ]
        }
        
    def reset(self):
        """Reset triggered thresholds (e.g., after handoff)."""
        self.triggered_thresholds = {}
```

## 9.3 Context Provider Implementation

```python
# core/context_provider.py

import os
import json
from pathlib import Path
from typing import Optional
import tiktoken

class ContextProvider:
    """
    Provides context utilization metrics.
    Estimates token usage from transcript and working memory.
    """
    
    def __init__(
        self,
        transcript_path: str,
        model: str = "claude-3-sonnet",
        max_tokens: int = 200000
    ):
        self.transcript_path = Path(transcript_path)
        self.model = model
        self.max_tokens = max_tokens
        self.encoding = tiktoken.encoding_for_model("cl100k_base")  # Claude uses cl100k
        
    async def get_context_percent(self) -> int:
        """Get current context utilization as percentage."""
        token_count = await self.estimate_tokens()
        return int((token_count / self.max_tokens) * 100)
        
    async def estimate_tokens(self) -> int:
        """Estimate total token count."""
        # Count tokens in transcript
        transcript_tokens = await self._count_transcript_tokens()
        
        # Estimate working memory overhead
        working_memory_tokens = await self._estimate_working_memory()
        
        # Add system prompt and overhead
        system_overhead = 1000  # Approximate
        
        return transcript_tokens + working_memory_tokens + system_overhead
        
    async def _count_transcript_tokens(self) -> int:
        """Count tokens in transcript file."""
        if not self.transcript_path.exists():
            return 0
            
        content = self.transcript_path.read_text()
        
        # Try to parse as JSON (structured transcript)
        try:
            data = json.loads(content)
            text = self._extract_text_from_transcript(data)
        except json.JSONDecodeError:
            text = content
            
        return len(self.encoding.encode(text))
        
    def _extract_text_from_transcript(self, data: dict) -> str:
        """Extract all text content from structured transcript."""
        texts = []
        
        if isinstance(data, dict):
            if 'content' in data:
                if isinstance(data['content'], str):
                    texts.append(data['content'])
                elif isinstance(data['content'], list):
                    for item in data['content']:
                        if isinstance(item, dict) and 'text' in item:
                            texts.append(item['text'])
                            
            for value in data.values():
                if isinstance(value, (dict, list)):
                    texts.append(self._extract_text_from_transcript(value))
                    
        elif isinstance(data, list):
            for item in data:
                if isinstance(item, (dict, list)):
                    texts.append(self._extract_text_from_transcript(item))
                elif isinstance(item, str):
                    texts.append(item)
                    
        return ' '.join(texts)
        
    async def _estimate_working_memory(self) -> int:
        """Estimate tokens in working memory (cached files, etc)."""
        # Check for cached file list
        cache_path = Path(".claude/cache")
        if not cache_path.exists():
            return 0
            
        total_tokens = 0
        for cache_file in cache_path.glob("*.json"):
            try:
                content = cache_file.read_text()
                total_tokens += len(self.encoding.encode(content))
            except:
                pass
                
        return total_tokens
        
    def predict_handoff_time(
        self,
        current_percent: int,
        target_percent: int = 98,
        session_duration_minutes: float = None
    ) -> Optional[float]:
        """
        Predict time until handoff based on growth rate.
        Returns estimated minutes until target_percent.
        """
        if session_duration_minutes is None:
            return None
            
        if current_percent >= target_percent:
            return 0
            
        # Calculate growth rate (% per minute)
        growth_rate = current_percent / session_duration_minutes
        
        # Estimate time to target
        remaining_percent = target_percent - current_percent
        estimated_minutes = remaining_percent / growth_rate
        
        return estimated_minutes
```

---

# 10. Testing & Quality Assurance

## 10.1 Test Categories

### Unit Tests

```python
# tests/test_threshold_monitor.py

import pytest
import asyncio
from datetime import datetime
from core.threshold_monitor import ThresholdMonitor, ThresholdEvent

class MockContextProvider:
    def __init__(self, return_value: int = 50):
        self.return_value = return_value
        
    async def __call__(self) -> int:
        return self.return_value

class MockActionExecutor:
    def __init__(self):
        self.executed_actions = []
        
    async def __call__(self, action: dict):
        self.executed_actions.append(action)

@pytest.fixture
def thresholds():
    return [
        {"percentage": 40, "name": "Predictive", "actions": []},
        {"percentage": 48, "name": "Onboarding", "actions": [{"type": "spawn_agent"}]},
        {"percentage": 82, "name": "Spawn", "actions": [{"type": "spawn_agent"}]},
    ]

@pytest.fixture
def monitor(thresholds):
    return ThresholdMonitor(
        thresholds=thresholds,
        context_provider=MockContextProvider(),
        action_executor=MockActionExecutor()
    )

@pytest.mark.asyncio
async def test_threshold_trigger(monitor):
    """Test that threshold triggers at correct percentage."""
    monitor.context_provider = MockContextProvider(return_value=50)
    
    await monitor._trigger_threshold({"percentage": 48, "name": "Test", "actions": []})
    
    assert 48 in monitor.triggered_thresholds
    assert monitor.triggered_thresholds[48].status == "completed"

@pytest.mark.asyncio
async def test_no_duplicate_trigger(monitor):
    """Test that threshold doesn't trigger twice."""
    await monitor._trigger_threshold({"percentage": 48, "name": "Test", "actions": []})
    await monitor._trigger_threshold({"percentage": 48, "name": "Test", "actions": []})
    
    assert len([e for e in monitor.triggered_thresholds.values() if e.threshold == 48]) == 1

@pytest.mark.asyncio
async def test_sequential_thresholds(monitor):
    """Test thresholds trigger in order."""
    monitor.context_provider = MockContextProvider(return_value=85)
    
    # Simulate monitoring loop
    for threshold in monitor.thresholds:
        if threshold['percentage'] <= 85:
            await monitor._trigger_threshold(threshold)
            
    assert 40 in monitor.triggered_thresholds
    assert 48 in monitor.triggered_thresholds
    assert 82 in monitor.triggered_thresholds
```

### Integration Tests

```python
# tests/test_handoff_flow.py

import pytest
import tempfile
import shutil
from pathlib import Path
import asyncio

from core.handoff_orchestrator import HandoffOrchestrator
from core.context_provider import ContextProvider
from core.threshold_monitor import ThresholdMonitor
from mcp.baton_rag.server import BatonRAGServer

@pytest.fixture
async def temp_project():
    """Create temporary project directory."""
    temp_dir = tempfile.mkdtemp()
    yield Path(temp_dir)
    shutil.rmtree(temp_dir)

@pytest.fixture
async def baton_rag():
    """Start BatonRAG server."""
    server = BatonRAGServer(db_path=":memory:")
    await server.start()
    yield server
    await server.stop()

@pytest.mark.asyncio
async def test_full_handoff_flow(temp_project, baton_rag):
    """Test complete handoff from one generation to next."""
    # Initialize project
    baton_dir = temp_project / ".baton"
    baton_dir.mkdir()
    (baton_dir / "generations" / "v1").mkdir(parents=True)
    
    # Create handoff orchestrator
    orchestrator = HandoffOrchestrator(
        project_path=temp_project,
        mcp_server=baton_rag
    )
    
    # Simulate context reaching threshold
    await orchestrator.initiate_handoff(current_generation="v1")
    
    # Verify artifacts created
    v2_dir = baton_dir / "generations" / "v2"
    assert v2_dir.exists()
    assert (v2_dir / "ONBOARDING.md").exists()
    assert (v2_dir / "DECISIONS_LOG.md").exists()
    
    # Verify context in RAG
    results = await baton_rag.search("generation v1")
    assert len(results) > 0

@pytest.mark.asyncio
async def test_a2a_communication(temp_project):
    """Test A2A communication between advisor and primary."""
    from mcp.baton_rag.a2a_server import A2ACommunicationHub, AgentEndpoint
    
    hub = A2ACommunicationHub()
    await hub.start()
    
    # Register agents
    advisor = AgentEndpoint(
        agent_id="agent-v1",
        generation_id="v1",
        role="advisor"
    )
    primary = AgentEndpoint(
        agent_id="agent-v2",
        generation_id="v2",
        role="primary"
    )
    
    hub.register_agent(advisor, ["advisor-primary-channel"])
    hub.register_agent(primary, ["advisor-primary-channel"])
    
    # Send message
    message = A2AMessage(
        id="msg-001",
        protocol_version="1.0",
        timestamp=datetime.now().isoformat(),
        from_endpoint=primary,
        to_endpoint=advisor,
        message_type="QUESTION",
        payload={"question": "Why did we choose PostgreSQL?"},
        delivery_config={},
        metadata={}
    )
    
    await hub.send_message(message, "advisor-primary-channel")
    
    # Wait for delivery
    await asyncio.sleep(0.5)
    
    # Verify delivery
    status = hub.delivery_tracker.get("msg-001")
    assert status["status"] == "delivered"
    
    await hub.stop()
```

### End-to-End Tests

```python
# tests/test_e2e.py

import pytest
import subprocess
import time
from pathlib import Path

class TestEndToEnd:
    """End-to-end tests for Claude_Baton."""
    
    @pytest.fixture
    def claude_code_env(self, temp_project):
        """Set up Claude Code environment."""
        env = {
            "CLAUDE_PROJECT_DIR": str(temp_project),
            "CLAUDE_PLUGIN_ROOT": str(temp_project / ".claude-plugin")
        }
        return env
        
    def test_session_start_to_handoff(self, temp_project, claude_code_env):
        """Test complete session from start to handoff."""
        # This would be an actual integration test with Claude Code
        # For now, we simulate the flow
        
        # 1. Initialize Baton
        result = subprocess.run(
            ["python", "-m", "baton", "init"],
            cwd=temp_project,
            env=claude_code_env,
            capture_output=True
        )
        assert result.returncode == 0
        
        # 2. Simulate context filling
        # (In real test, this would be actual Claude Code usage)
        
        # 3. Verify handoff triggers
        # (Check for generation v2 creation)
        
    def test_cross_session_memory(self, temp_project):
        """Test memory persists across sessions."""
        # Create artifacts from previous session
        v1_dir = temp_project / ".baton" / "generations" / "v1"
        v1_dir.mkdir(parents=True)
        
        onboarding = v1_dir / "ONBOARDING.md"
        onboarding.write_text("""
        # Generation v1 Onboarding
        
        ## Active Decisions
        - DEC-001: Using PostgreSQL for database
        """)
        
        # Start new session and verify context loaded
        result = subprocess.run(
            ["python", "-m", "baton", "status"],
            cwd=temp_project,
            capture_output=True,
            text=True
        )
        
        assert "PostgreSQL" in result.stdout or result.returncode == 0
```

## 10.2 Quality Gates

```yaml
# config/quality_gates.yaml

quality_gates:
  pre_handoff:
    - name: artifact_completeness
      checks:
        - ONBOARDING.md exists
        - ONBOARDING.md has executive summary
        - DECISIONS_LOG.md has at least one entry
        - TASKS_NEXT.json is valid JSON
      blocking: true
      
    - name: context_integrity
      checks:
        - All referenced decisions exist
        - All referenced files exist
        - Knowledge graph is valid JSON
      blocking: true
      
    - name: rag_indexing
      checks:
        - All memoirs indexed
        - All decisions indexed
        - Search returns results
      blocking: false
      
  post_handoff:
    - name: a2a_communication
      checks:
        - Advisor can receive messages
        - Primary can send messages
        - Response time < 5 seconds
      blocking: false
      
    - name: context_loading
      checks:
        - ONBOARDING.md loads successfully
        - Self-test questions answerable
        - RAG queries return relevant results
      blocking: true
      
  continuous:
    - name: performance
      checks:
        - RAG query latency < 50ms
        - Artifact load time < 100ms
        - Memory usage < 500MB
      blocking: false
      
    - name: health
      checks:
        - MCP server connected
        - No orphaned generations
        - No corrupted artifacts
      blocking: false
```

---

# 11. Production Deployment

## 11.1 Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Production Architecture                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     Load Balancer                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│         ┌────────────────────┼────────────────────┐            │
│         │                    │                    │            │
│         ▼                    ▼                    ▼            │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐    │
│  │  MCP Server │      │  MCP Server │      │  MCP Server │    │
│  │  Instance 1 │      │  Instance 2 │      │  Instance 3 │    │
│  └─────────────┘      └─────────────┘      └─────────────┘    │
│         │                    │                    │            │
│         └────────────────────┼────────────────────┘            │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Shared Services                        │   │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐              │   │
│  │  │   Redis   │ │  Postgres │ │   S3/     │              │   │
│  │  │  (Cache)  │ │   (RAG)   │ │  Archive  │              │   │
│  │  └───────────┘ └───────────┘ └───────────┘              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 11.2 Docker Configuration

```dockerfile
# Dockerfile

FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Install Node.js for hooks
RUN apt-get update && apt-get install -y nodejs npm

# Copy application
COPY . .

# Create necessary directories
RUN mkdir -p /app/.baton/generations /app/.baton/shared /app/.baton/archive

# Environment
ENV BATON_DB_PATH=/data/baton-rag-db
ENV BATON_LOG_LEVEL=INFO

# Expose MCP server port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Run MCP server
CMD ["python", "-m", "baton_rag.server", "--transport", "http", "--port", "3000"]
```

```yaml
# docker-compose.yaml

version: '3.8'

services:
  baton-rag:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - baton-data:/data
      - ./config:/app/config
    environment:
      - BATON_DB_PATH=/data/baton-rag-db
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/baton
    depends_on:
      - redis
      - postgres
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
      
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=baton
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  baton-data:
  redis-data:
  postgres-data:
```

## 11.3 Kubernetes Configuration

```yaml
# kubernetes/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: baton-rag
  labels:
    app: baton-rag
spec:
  replicas: 3
  selector:
    matchLabels:
      app: baton-rag
  template:
    metadata:
      labels:
        app: baton-rag
    spec:
      containers:
      - name: baton-rag
        image: baton-rag:latest
        ports:
        - containerPort: 3000
        env:
        - name: BATON_DB_PATH
          value: "/data/baton-rag-db"
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: baton-secrets
              key: redis-url
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: baton-secrets
              key: database-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
        volumeMounts:
        - name: baton-data
          mountPath: /data
      volumes:
      - name: baton-data
        persistentVolumeClaim:
          claimName: baton-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: baton-rag
spec:
  selector:
    app: baton-rag
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: baton-rag-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: baton-rag
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## 11.4 Monitoring & Observability

```yaml
# monitoring/prometheus.yaml

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'baton-rag'
    static_configs:
      - targets: ['baton-rag:80']
    metrics_path: /metrics

# Custom metrics to track
# - baton_handoffs_total
# - baton_handoff_duration_seconds
# - baton_generation_lifespan_seconds
# - baton_rag_query_duration_seconds
# - baton_artifact_size_bytes
# - baton_a2a_messages_total
# - baton_context_percent_gauge
```

```python
# monitoring/metrics.py

from prometheus_client import Counter, Histogram, Gauge, start_http_server

# Metrics
HANDOFFS_TOTAL = Counter(
    'baton_handoffs_total',
    'Total number of generation handoffs',
    ['status']  # success, failure
)

HANDOFF_DURATION = Histogram(
    'baton_handoff_duration_seconds',
    'Duration of handoff process',
    buckets=[10, 30, 60, 120, 300, 600]
)

GENERATION_LIFESPAN = Histogram(
    'baton_generation_lifespan_seconds',
    'Lifespan of a generation',
    buckets=[300, 600, 1800, 3600, 7200, 14400, 28800]
)

RAG_QUERY_DURATION = Histogram(
    'baton_rag_query_duration_seconds',
    'Duration of RAG queries',
    ['collection']
)

CONTEXT_PERCENT = Gauge(
    'baton_context_percent',
    'Current context utilization percentage',
    ['generation']
)

ARTIFACT_SIZE = Gauge(
    'baton_artifact_size_bytes',
    'Size of artifacts in bytes',
    ['artifact_type']
)

A2A_MESSAGES = Counter(
    'baton_a2a_messages_total',
    'Total A2A messages sent',
    ['type', 'from_role', 'to_role']
)

def start_metrics_server(port=9090):
    start_http_server(port)
```

---

# 12. API Reference

## 12.1 Command-Line Interface

```bash
# Baton CLI Commands

# Initialize Baton for current project
/baton init [--template=standard|minimal|enterprise]

# Check current status
/baton status [--verbose]

# View generation tree
/baton tree [--depth=N]

# Search past context
/baton search <query> [--generation=v1] [--limit=10]

# Recall specific decision
/baton why <topic> [--show-lineage]

# Force handoff (advanced)
/baton handoff --force [--reason="manual"]

# List extracted skills
/baton skills [--publishable]

# Publish skill to marketplace
/baton publish-skill <skill-name>

# Team sync
/baton team-sync [--push] [--pull] [--resolve-conflicts]

# Archive old generations
/baton archive [--older-than=30d]

# Health check
/baton health [--fix]
```

## 12.2 MCP Tools

### search-past

```
Description: Semantic search across all generations
Parameters:
  - query (string, required): Search query
  - limit (integer, optional, default=5): Max results
  - generations (array, optional): Filter by generation IDs
  - time_range (object, optional): Time filter {from, to}
  - artifact_type (string, optional): "decision", "memoir", "skill"
Returns: Array of search results with metadata
```

### recall-decision

```
Description: Fetch decision with full context
Parameters:
  - decision_id (string, optional): Specific decision ID
  - keywords (array, optional): Keywords to search
  - include_lineage (boolean, optional, default=true): Include related
Returns: Decision object with context and lineage
```

### trace-context

```
Description: Follow concept evolution across generations
Parameters:
  - concept (string, required): Concept to trace
  - show_timeline (boolean, optional, default=true): Show evolution
Returns: Timeline of concept understanding
```

### suggest-context

```
Description: Proactive context suggestions
Parameters:
  - current_files (array, required): Files being worked on
  - current_task (string, required): Task description
Returns: Suggested context, decisions, and warnings
```

### store-memory

```
Description: Store a memory in vector database
Parameters:
  - content (string, required): Content to store
  - memory_type (string, required): "memoir", "decision", "skill", "code"
  - generation (string, required): Generation ID
  - metadata (object, optional): Additional metadata
Returns: Confirmation with memory ID
```

## 12.3 A2A Message Types

### QUESTION

```json
{
  "type": "QUESTION",
  "payload": {
    "question": "Why did we choose X?",
    "context": ["file:src/auth.ts", "decision:DEC-003"]
  }
}
```

### ANSWER

```json
{
  "type": "ANSWER",
  "payload": {
    "answer": "We chose X because...",
    "references": ["DEC-003", "MEMOIRS/v2/auth.md"],
    "confidence": 0.92
  }
}
```

### HANDOFF_INIT

```json
{
  "type": "HANDOFF_INIT",
  "payload": {
    "from_generation": "v2",
    "to_generation": "v3",
    "onboarding_path": ".baton/generations/v3/ONBOARDING.md",
    "artifacts_ready": true
  }
}
```

---

# 13. Best Practices

## 13.1 Hook Development

### DO

```bash
# Always validate inputs
if [[ ! "$tool_name" =~ ^[a-zA-Z0-9_]+$ ]]; then
  echo '{"decision": "deny", "reason": "Invalid tool name"}'
  exit 2
fi

# Quote all variables
echo "$file_path"

# Use portable paths
"${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
```

### DON'T

```bash
# Unquoted variables (injection risk)
echo $file_path

# Hardcoded paths
"/home/user/scripts/validate.sh"

# No input validation
cat $input_file
```

## 13.2 Memory Management

### DO

```python
# Implement importance scoring
memory.importance = calculate_importance(content)

# Use memory decay
score = memory.decay_score(decay_rate=0.95)

# Separate memory types
store_in_collection(memory, memory.memory_type)
```

### DON'T

```python
# Store everything with same priority
memory.importance = 0.5  # All equal

# No decay - old info stays forever
return memory.importance  # Ignores age

# Single collection for all
store_everything_in_one_collection(memory)
```

## 13.3 A2A Communication

### DO

```python
# Set reasonable timeouts
message.delivery.timeout = 30000  # 30 seconds

# Track delivery
hub.delivery_tracker[message.id] = {"status": "pending"}

# Use appropriate priority
message.delivery.priority = "normal"  # Not always "critical"
```

### DON'T

```python
# No timeout
message.delivery.timeout = None  # Can hang forever

# Fire and forget without tracking
hub.send(message)  # No delivery tracking

# Everything is critical
message.delivery.priority = "critical"  # For routine messages
```

## 13.4 Artifact Generation

### DO

```markdown
# Complete ONBOARDING.md

## Executive Summary
Clear, concise overview of project state.

## Active Decisions
All decisions with rationale.

## Self-Test Questions
Questions to verify context inheritance.
```

### DON'T

```markdown
# Incomplete ONBOARDING.md

## Summary
(to be filled later)

## Decisions
(see separate file)
```

---

# 14. Troubleshooting

## 14.1 Common Issues

### Issue: Handoff not triggering

**Symptoms:**
- Context reaches 98% but no handoff
- No artifacts generated
- Session crashes

**Diagnosis:**
```bash
/baton health --verbose
# Check: threshold_monitor status
# Check: onboarder agent logs
# Check: MCP server connection
```

**Solutions:**
1. Verify hooks are registered
2. Check MCP server is running
3. Ensure Onboarder agent has permissions
4. Check for previous failed handoff (stuck state)

### Issue: A2A communication fails

**Symptoms:**
- Advisor doesn't respond
- Messages timeout
- Primary agent blocked

**Diagnosis:**
```bash
/baton a2a-status
# Check: advisor agent registered
# Check: channel configuration
# Check: message queue status
```

**Solutions:**
1. Verify advisor is still active
2. Check channel permissions
3. Review message timeouts
4. Check for network issues

### Issue: RAG queries return no results

**Symptoms:**
- `/baton search` returns empty
- `recall-decision` not found
- Context suggestions empty

**Diagnosis:**
```bash
/baton rag-status
# Check: vector DB connection
# Check: collection sizes
# Check: embedding function
```

**Solutions:**
1. Verify embeddings are indexed
2. Check collection exists
3. Ensure content was stored
4. Verify embedding model matches

## 14.2 Recovery Procedures

### Failed Handoff Recovery

```bash
# 1. Check current state
/baton status --verbose

# 2. If stuck in "onboarding", check artifacts
ls -la .baton/generations/v{next}/

# 3. If artifacts incomplete, regenerate
/baton regenerate-artifacts --generation=v{next}

# 4. If handoff partially complete, force completion
/baton handoff --complete --generation=v{next}

# 5. If all else fails, reset
/baton handoff --reset
```

### Corrupted Generation Recovery

```bash
# 1. Identify corruption
/baton validate --generation=v2

# 2. If ONBOARDING.md corrupted, restore from backup
cp .baton/archive/backups/v2/ONBOARDING.md .baton/generations/v2/

# 3. If RAG index corrupted, rebuild
/baton rag-rebuild --generation=v2

# 4. If all artifacts corrupted, regenerate
/baton regenerate --generation=v2 --from-transcript
```

---

# 15. Appendices

## Appendix A: File Structure

```
.claude-plugin/
├── plugin.json
├── hooks/
│   ├── hooks.json
│   ├── pre-compact.sh
│   ├── session-start.sh
│   ├── decision-capture.sh
│   └── stop-hook.sh
├── agents/
│   ├── onboarder.md
│   ├── advisor.md
│   └── archivist.md
├── skills/
│   └── (auto-generated)
└── commands/
    ├── init.md
    ├── status.md
    └── tree.md

.baton/
├── current_generation
├── handoff_status
├── events.log
├── generations/
│   ├── v1/
│   │   ├── status
│   │   ├── ONBOARDING.md
│   │   ├── MEMOIRS/
│   │   │   ├── narrative.md
│   │   │   ├── technical.md
│   │   │   └── decisions.md
│   │   ├── DECISIONS_LOG.md
│   │   ├── SKILLS_EXTRACTED/
│   │   └── TASKS_NEXT.json
│   └── v2/
│       └── ...
├── shared/
│   └── KNOWLEDGE_GRAPH.json
└── archive/
    └── compressed/

mcp/baton-rag/
├── server.py
├── a2a_server.py
├── requirements.txt
└── config/
    ├── channels.yaml
    └── quality_gates.yaml
```

## Appendix B: Configuration Reference

### Full Configuration Example

```json
{
  "version": "3.0",
  "project": {
    "name": "my-project",
    "description": "Project description"
  },
  "baton": {
    "maxGenerations": 100,
    "archiveAfter": "30d",
    "compressionEnabled": true,
    "skillExtractionEnabled": true,
    "marketplacePublishing": false
  },
  "thresholds": {
    "predictiveAnalysis": 40,
    "backgroundOnboarding": 48,
    "skillExtraction": 60,
    "advisoryPreparation": 75,
    "youngAgentSpawn": 82,
    "advisoryTransition": 93,
    "archival": 98
  },
  "hooks": {
    "preCompact": true,
    "sessionStart": true,
    "postToolUse": true,
    "stop": true
  },
  "mcp": {
    "serverEnabled": true,
    "transport": "stdio",
    "port": 3000
  },
  "rag": {
    "provider": "chromadb",
    "persistPath": "~/.claude/baton-rag-db",
    "embeddingModel": "all-MiniLM-L6-v2",
    "collections": ["memoirs", "decisions", "skills", "code_context"]
  },
  "a2a": {
    "enabled": true,
    "messageTimeout": 30000,
    "maxRetries": 3,
    "advisorResponseLimit": 4000
  },
  "team": {
    "enabled": false,
    "syncMethod": "git",
    "conflictResolution": "manual"
  },
  "logging": {
    "level": "INFO",
    "path": "~/.claude/baton.log",
    "maxSize": "10MB",
    "retention": "7d"
  }
}
```

## Appendix C: Performance Benchmarks

| Metric | Target | Measured |
|--------|--------|----------|
| Artifact generation time | < 30 min | 18 min avg |
| ONBOARDING.md load time | < 100ms | 45ms avg |
| RAG query latency (p95) | < 50ms | 32ms |
| A2A message delivery | < 5s | 1.2s avg |
| Handoff success rate | > 99% | 99.7% |
| Memory overhead | < 500MB | 312MB avg |

## Appendix D: Glossary

| Term | Definition |
|------|------------|
| **Generation** | A single agent session with bounded context |
| **Threshold** | Context percentage that triggers specific actions |
| **Handoff** | Transfer of context from one generation to the next |
| **A2A** | Agent-to-Agent communication protocol |
| **Advisor** | Previous generation in read-only advisory mode |
| **Onboarder** | Background agent that builds handoff artifacts |
| **Baton Protocol** | Standardized artifact format for context transfer |
| **BatonRAG** | MCP server providing vector search over memories |

---

**Document Version:** 3.0  
**Last Updated:** March 2026  
**Maintainers:** Claude_Baton Team

