# Claude_Baton: File-by-File Implementation Guide

## Exhaustive Architecture & Implementation Options

This document examines every file in the Claude_Baton system, explaining its purpose, architecture, and implementation alternatives when the best approach isn't obvious.

---

# Table of Contents

1. [Plugin Configuration Files](#1-plugin-configuration-files)
2. [Hook System Files](#2-hook-system-files)
3. [Agent Definition Files](#3-agent-definition-files)
4. [Baton Protocol Artifacts](#4-baton-protocol-artifacts)
5. [MCP Server Files](#5-mcp-server-files)
6. [Core System Files](#6-core-system-files)
7. [Configuration Files](#7-configuration-files)
8. [Utility Scripts](#8-utility-scripts)
9. [Test Files](#9-test-files)

---

# 1. Plugin Configuration Files

## 1.1 `.claude-plugin/plugin.json`

### Purpose
Registers the plugin with Claude Code and declares its capabilities, dependencies, and metadata.

### Architecture

```json
{
  "name": "claude-baton",
  "version": "1.0.0",
  "description": "Generational context handoff for continuous AI agent operation",
  "author": "SuperInstance",
  
  "entry_points": {
    "hooks": "./hooks/hooks.json",
    "agents": "./agents/",
    "commands": "./commands/",
    "skills": "./skills/"
  },
  
  "capabilities": {
    "requires_context_access": true,
    "requires_filesystem": true,
    "requires_mcp": true,
    "can_spawn_agents": true
  },
  
  "dependencies": {
    "mcp_servers": ["baton-rag"],
    "min_claude_version": "1.0.0"
  },
  
  "configuration": {
    "thresholds": {
      "predictive": 40,
      "onboarding": 48,
      "spawn": 82,
      "advisory": 93,
      "archive": 98
    },
    "artifact_path": ".baton",
    "rag_db_path": "~/.claude/baton-rag-db"
  }
}
```

### Implementation Considerations

**Decision Point: Configuration Location**

| Approach | Pros | Cons |
|----------|------|------|
| **Inline in plugin.json** | Single source of truth, easy to parse | Harder to modify per-project |
| **Separate config file** | User can customize, can be git-ignored | Two files to maintain |
| **Environment variables** | Runtime configuration, secure for secrets | Not discoverable, needs docs |
| **Hybrid: defaults in plugin.json, override in .baton/config.json** | Best of both worlds | More complex parsing |

**Recommended: Hybrid approach** - Defaults in `plugin.json`, project-specific overrides in `.baton/config.json`.

**Decision Point: Capability Declaration**

How much should the plugin declare vs. discover at runtime?

```
Option A: Declare everything upfront
- Claude Code can validate compatibility before loading
- Clear contract for what plugin needs
- But: Inflexible if capabilities change

Option B: Discover at runtime
- More flexible, self-adapting
- But: Harder to debug, errors surface late

Option C: Declare required, discover optional
- Clear about hard requirements
- Flexible about enhancements
- Best balance
```

---

## 1.2 `.claude-plugin/commands/init.md`

### Purpose
Defines the `/baton init` command that initializes Baton for a project.

### Architecture

```markdown
---
name: init
description: Initialize Claude_Baton for the current project
arguments: []
---

You are initializing Claude_Baton for this project.

## Steps

1. **Create Directory Structure**
   ```
   .baton/
   ├── current_generation
   ├── handoff_status
   ├── events.log
   ├── generations/
   │   └── v1/
   │       └── status
   ├── shared/
   └── archive/
   ```

2. **Initialize State Files**
   - Write "v1" to `.baton/current_generation`
   - Write "none" to `.baton/handoff_status`
   - Write "active" to `.baton/generations/v1/status`

3. **Verify MCP Server**
   - Check if baton-rag MCP server is connected
   - If not, provide setup instructions

4. **Create Initial Artifact**
   - Generate placeholder ONBOARDING.md for v1

## Output
Report initialization status and next steps.
```

### Implementation Considerations

**Decision Point: Initialization Scope**

| Scope | What It Does | When to Use |
|-------|--------------|-------------|
| **Minimal** | Create directories, set v1 | New projects |
| **Standard** | Above + sample artifacts, README | Most cases |
| **Full** | Above + MCP setup, team sync config | Enterprise |
| **From Template** | Copy from template directory | Standardized projects |

**Recommended: Standard** as default, with `--minimal` and `--enterprise` flags.

**Decision Point: Existing Project Handling**

What if `.baton/` already exists?

```
Option A: Abort with error
- Safest, no data loss risk
- But: Annoying if user wants to reinitialize

Option B: Merge/Update
- Preserves existing generations
- Updates config if needed
- But: Could cause inconsistent state

Option C: Prompt for confirmation
- Ask user: "Existing Baton data found. Overwrite/Update/Cancel?"
- Safest UX
- Requires interactive mode

Option D: Backup and reinitialize
- Move existing to .baton.backup.{timestamp}
- Start fresh
- Preserves data while allowing clean slate
```

**Recommended: Option D** - Backup and reinitialize, with `--force` to skip backup.

---

## 1.3 `.claude-plugin/commands/status.md`

### Purpose
Defines the `/baton status` command that shows current system state.

### Architecture

```markdown
---
name: status
description: Show current Baton system status
arguments:
  - name: verbose
    type: boolean
    default: false
    description: Show detailed information
---

Report the current Baton system status.

## Information to Gather

1. **Current Generation**
   - Read `.baton/current_generation`
   - Get generation start time from `.baton/generations/{gen}/start_time`

2. **Context Level**
   - If available, get current context percentage
   - Estimate from transcript if not directly available

3. **Handoff Status**
   - Read `.baton/handoff_status`
   - If onboarding/ready, show progress

4. **Artifact Status**
   - Check each artifact exists and has content
   - Report any missing or incomplete

5. **MCP Server Status**
   - Check baton-rag connection
   - Report collection sizes

6. **Generation Statistics**
   - Total generations
   - Total decisions logged
   - Skills extracted

## Output Format

```
┌────────────────────────────────────┐
│ Claude_Baton Status                │
├────────────────────────────────────┤
│ Version: 1.0.0                     │
│ Current Generation: v3             │
│ Context: 67%                       │
│ Handoff: None                      │
│                                    │
│ MCP Server: ✓ Connected            │
│ RAG Index: 2.3 MB (156 entries)    │
│                                    │
│ Statistics:                        │
│ ├── Generations: 3                 │
│ ├── Decisions: 12                  │
│ └── Skills: 2                      │
│                                    │
│ Artifacts: ✓ Complete              │
│ Status: Ready                      │
└────────────────────────────────────┘
```
```

### Implementation Considerations

**Decision Point: Context Percentage Acquisition**

How do we know the current context utilization?

| Method | Accuracy | Complexity |
|--------|----------|------------|
| **Direct API** | 100% | Requires Claude Code API access |
| **Transcript Token Count** | 90% | Count tokens in transcript file |
| **Estimated from Messages** | 70% | Count messages × average tokens |
| **User Reported** | Variable | Ask user to report |

**Implementation for Transcript Token Count:**

```python
import tiktoken

def get_context_percent(transcript_path: str, max_tokens: int = 200000) -> int:
    """
    Estimate context utilization from transcript.
    """
    encoding = tiktoken.get_encoding("cl100k_base")  # Claude's encoding
    
    with open(transcript_path, 'r') as f:
        content = f.read()
    
    # If JSON, extract text content
    try:
        data = json.loads(content)
        text = extract_text_from_transcript(data)
    except:
        text = content
    
    token_count = len(encoding.encode(text))
    
    # Add overhead for system prompt, working memory
    overhead = 5000  # Approximate
    
    total = token_count + overhead
    return min(100, int((total / max_tokens) * 100))
```

**Recommended: Transcript Token Count** - Good balance of accuracy and feasibility.

**Decision Point: Status Output Format**

| Format | Use Case |
|--------|----------|
| **ASCII Box** | Terminal, human-readable (default) |
| **JSON** | Scripting, piping to other tools |
| **Markdown** | Documentation, README |
| **YAML** | Configuration-like output |

**Recommended: ASCII Box for interactive, `--json` flag for scripting.**

---

# 2. Hook System Files

## 2.1 `.claude-plugin/hooks/hooks.json`

### Purpose
Declares all hooks and their trigger conditions. This is the entry point for Claude Code's hook system.

### Architecture

```json
{
  "description": "Claude_Baton hooks for generational context handoff",
  "version": "1.0.0",
  
  "hooks": {
    "PreCompact": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/pre-compact.sh",
            "timeout": 60,
            "background": false
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
            "timeout": 10,
            "background": true
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
            "prompt": "Before stopping, verify: 1) All tasks in TASKS_NEXT.json are complete or deferred, 2) Any in-progress work is documented. If handoff is in progress, wait for completion.",
            "timeout": 30
          }
        ]
      }
    ],
    
    "SubagentStop": [
      {
        "matcher": "*",
        "condition": "agent_role == 'onboarder'",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/onboarder-complete.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### Implementation Considerations

**Decision Point: Hook Type Selection**

Each hook event can use either `command` or `prompt` type hooks.

| Type | How It Works | Best For |
|------|--------------|----------|
| **Command** | Execute shell script, return JSON | Deterministic logic, file operations, state changes |
| **Prompt** | Inject instruction for Claude | Validation, reasoning, decisions requiring LLM |

```
When to use Command:
- State transitions (updating handoff_status)
- File operations (creating directories, writing artifacts)
- External calls (MCP server, APIs)
- Fast, deterministic operations

When to use Prompt:
- Validation requiring judgment ("Is this task really done?")
- Decisions needing context ("Should we block this handoff?")
- Situations where LLM reasoning adds value
```

**Recommended: Use Command for 90% of hooks, Prompt for Stop/Validation scenarios.**

**Decision Point: Matcher Strategy**

The `matcher` field determines which tools/events trigger the hook.

```
"*" - Match everything (default for most hooks)
"Write" - Match only Write tool
"Write|Edit|MultiEdit" - Match any of these tools
"mcp__.*__delete.*" - Regex pattern for MCP delete operations

Strategies:
1. Broad matching ("*") + condition checking in script
   - More flexible, one hook point
   - Script does filtering

2. Narrow matching ("Write|Edit")
   - Only triggers when needed
   - But: Must update matcher if new tools added

3. Multiple hook entries for same event
   - Different matchers, different handlers
   - But: Order of execution matters
```

**Recommended: Broad matching with condition logic in scripts** - More maintainable.

**Decision Point: Background Execution**

Should hooks run synchronously or in background?

| Mode | Use Case | Trade-off |
|------|----------|-----------|
| **Foreground** | State changes, blocking operations | Reliable but can slow down tool |
| **Background** | Logging, non-critical capture | Fast but may miss if process dies |

```
PostToolUse for decision capture:
- Background: Won't slow down user's Write/Edit
- But: If process crashes, decision not captured
- Mitigation: Also capture in PreCompact hook

PreCompact for handoff:
- Must be foreground
- Handoff is critical, can't skip
```

**Recommended: Critical operations = foreground, logging/capture = background.**

---

## 2.2 `.claude-plugin/hooks/pre-compact.sh`

### Purpose
The core orchestrator hook. Triggers at context thresholds and manages the entire handoff lifecycle.

### Architecture

```bash
#!/bin/bash
# File: pre-compact.sh
# Purpose: Orchestrate generational handoff based on context thresholds

set -euo pipefail

# =============================================================================
# CONFIGURATION
# =============================================================================

THRESHOLD_PREDICTIVE=40
THRESHOLD_ONBOARDING=48
THRESHOLD_SPAWN=82
THRESHOLD_ADVISORY=93
THRESHOLD_ARCHIVE=98

BATON_DIR=".baton"
GENERATIONS_DIR="$BATON_DIR/generations"
CURRENT_GEN_FILE="$BATON_DIR/current_generation"
HANDOFF_STATUS_FILE="$BATON_DIR/handoff_status"
EVENTS_LOG="$BATON_DIR/events.log"

# =============================================================================
# UTILITY FUNCTIONS
# =============================================================================

log_event() {
    local event="$1"
    local data="$2"
    echo "$(date -Iseconds) | $event | $data" >> "$EVENTS_LOG"
}

get_current_generation() {
    cat "$CURRENT_GEN_FILE" 2>/dev/null || echo "v1"
}

get_next_generation() {
    local current="$1"
    local num=$(echo "$current" | sed 's/v//')
    echo "v$((num + 1))"
}

get_handoff_status() {
    cat "$HANDOFF_STATUS_FILE" 2>/dev/null || echo "none"
}

set_handoff_status() {
    echo "$1" > "$HANDOFF_STATUS_FILE"
}

get_context_percent() {
    # Read from stdin (hook input)
    local input=$(cat)
    echo "$input" | jq -r '.contextPercent // 0'
}

# =============================================================================
# OUTPUT FUNCTIONS
# =============================================================================

output_continue() {
    jq -n '{"continue": true}'
}

output_message() {
    local msg="$1"
    jq -n --arg msg "$msg" '{"continue": true, "systemMessage": $msg}'
}

output_block() {
    local reason="$1"
    jq -n --arg reason "$reason" '{"decision": "block", "reason": $reason}'
    exit 2
}

# =============================================================================
# HANDOFF PHASES
# =============================================================================

phase_predictive() {
    # Warm up caches, analyze patterns
    log_event "PREDICTIVE_START" ""
    
    # Pre-cache frequently accessed files
    if [[ -f "$GENERATIONS_DIR/$CURRENT_GEN/DECISIONS_LOG.md" ]]; then
        cat "$GENERATIONS_DIR/$CURRENT_GEN/DECISIONS_LOG.md" > /dev/null
    fi
    
    # Analyze conversation patterns for likely next actions
    # (Could spawn a background analyzer here)
    
    log_event "PREDICTIVE_COMPLETE" ""
}

phase_onboarding() {
    local next_gen="$1"
    
    log_event "ONBOARDING_START" "Generation: $next_gen"
    
    # Create generation directory
    mkdir -p "$GENERATIONS_DIR/$next_gen"
    mkdir -p "$GENERATIONS_DIR/$next_gen/MEMOIRS"
    mkdir -p "$GENERATIONS_DIR/$next_gen/SKILLS_EXTRACTED"
    
    # Mark as onboarding
    set_handoff_status "onboarding"
    
    # Spawn Onboarder agent
    # Note: This is conceptual - actual spawn mechanism depends on Claude Code API
    cat > "$GENERATIONS_DIR/$next_gen/.onboarder_request" << EOF
{
    "role": "onboarder",
    "contextIsolation": true,
    "executionMode": "background",
    "outputPath": "$GENERATIONS_DIR/$next_gen",
    "prompt": "Build Baton Protocol artifacts for generation $next_gen"
}
EOF
    
    output_message "⚡ Baton: Building artifacts for $next_gen in background..."
}

phase_spawn_successor() {
    local next_gen="$1"
    
    log_event "SPAWN_START" "Generation: $next_gen"
    
    # Check if artifacts are ready
    if [[ ! -f "$GENERATIONS_DIR/$next_gen/ONBOARDING.md" ]]; then
        output_message "⏳ Baton: Still building artifacts... (spawn delayed)"
        return
    fi
    
    set_handoff_status "ready"
    
    output_message "🔄 Baton: Artifacts ready. Successor will spawn at $THRESHOLD_ADVISORY%."
}

phase_advisory_transition() {
    local current_gen="$1"
    local next_gen="$2"
    
    log_event "ADVISORY_START" "From: $current_gen, To: $next_gen"
    
    # Mark current generation as advisory
    echo "advisory" > "$GENERATIONS_DIR/$current_gen/status"
    
    # Update current generation
    echo "$next_gen" > "$CURRENT_GEN_FILE"
    
    set_handoff_status "active"
    
    output_message "🔄 Baton: $current_gen → $next_gen transition active. Advisor available for questions."
}

phase_archive() {
    local prev_gen="$1"
    
    log_event "ARCHIVE_START" "Generation: $prev_gen"
    
    # Create archive
    mkdir -p "$BATON_DIR/archive"
    tar -czf "$BATON_DIR/archive/${prev_gen}_$(date +%Y%m%d_%H%M%S).tar.gz" \
        -C "$GENERATIONS_DIR" "$prev_gen" 2>/dev/null || true
    
    # Mark as archived
    echo "archived" > "$GENERATIONS_DIR/$prev_gen/status"
    
    # Reset handoff status
    set_handoff_status "none"
    
    # Store in RAG
    # (Would call MCP tool here)
    
    log_event "ARCHIVE_COMPLETE" "Generation: $prev_gen"
}

# =============================================================================
# MAIN LOGIC
# =============================================================================

main() {
    # Read hook input
    local input=$(cat)
    local context_pct=$(echo "$input" | jq -r '.contextPercent // 0')
    
    local current_gen=$(get_current_generation)
    local next_gen=$(get_next_generation "$current_gen")
    local handoff_status=$(get_handoff_status)
    
    log_event "HOOK_TRIGGERED" "Context: $context_pct%, Status: $handoff_status"
    
    # State machine for handoff phases
    case "$handoff_status" in
        "none")
            if [[ $context_pct -ge $THRESHOLD_ONBOARDING ]]; then
                phase_onboarding "$next_gen"
            elif [[ $context_pct -ge $THRESHOLD_PREDICTIVE ]]; then
                phase_predictive
            else
                output_continue
            fi
            ;;
            
        "onboarding")
            if [[ $context_pct -ge $THRESHOLD_SPAWN ]]; then
                phase_spawn_successor "$next_gen"
            else
                # Check if onboarding completed
                if [[ -f "$GENERATIONS_DIR/$next_gen/ONBOARDING.md" ]]; then
                    set_handoff_status "ready"
                    output_message "✅ Baton: Artifacts complete for $next_gen."
                else
                    output_continue
                fi
            fi
            ;;
            
        "ready")
            if [[ $context_pct -ge $THRESHOLD_ADVISORY ]]; then
                phase_advisory_transition "$current_gen" "$next_gen"
            else
                output_continue
            fi
            ;;
            
        "active")
            if [[ $context_pct -ge $THRESHOLD_ARCHIVE ]]; then
                local prev_gen="v$(($(echo $current_gen | sed 's/v//') - 1))"
                phase_archive "$prev_gen"
            else
                output_continue
            fi
            ;;
            
        *)
            log_event "ERROR" "Unknown handoff status: $handoff_status"
            output_continue
            ;;
    esac
}

main
```

### Implementation Considerations

**Decision Point: State Machine vs. Linear Logic**

| Approach | Pros | Cons |
|----------|------|------|
| **Linear if/else** | Simple, easy to follow | Hard to add new states |
| **State machine** | Clear transitions, extensible | More initial setup |
| **Event-driven** | Decoupled, flexible | Harder to trace execution |

**Recommended: State machine** - Handoff has clear states (none → onboarding → ready → active → none), transitions are well-defined.

**Decision Point: Where to Store State**

| Location | Pros | Cons |
|----------|------|------|
| **File (`.baton/handoff_status`)** | Persistent, git-visible, easy debugging | Requires file I/O |
| **Environment variable** | Fast, in-memory | Lost between invocations |
| **MCP server state** | Centralized, queryable | Adds dependency |
| **JSON state file** | Structured, extensible | More complex parsing |

**Recommended: Simple text file for status, JSON for complex state** - Best balance of simplicity and extensibility.

**Decision Point: Agent Spawning Mechanism**

How does the hook actually spawn an Onboarder agent?

```
Option A: Claude Code Native (if available)
- Use Claude Code's agent spawn API
- Most integrated, native isolation
- But: API may not exist yet

Option B: Background Process
- Spawn a separate process that calls Claude API
- Full control over execution
- But: Process management, cleanup needed

Option C: Deferred Execution
- Write a request file, let Claude Code pick it up
- Simple, decoupled
- But: Latency, may not execute immediately

Option D: MCP Tool Call
- Call MCP tool that spawns agent
- Reuses existing infrastructure
- But: MCP may not support spawning
```

**Recommended: Write request file + MCP notification** - Decoupled but reliable.

**Decision Point: Error Handling**

What happens if a phase fails?

```
Strategy A: Fail Fast
- Any error aborts handoff
- User sees error, can retry
- Safest but most disruptive

Strategy B: Retry with Backoff
- Retry failed operations
- Exponential backoff
- Good for transient failures

Strategy C: Graceful Degradation
- Continue with partial success
- Log errors, alert user
- Keeps session running

Strategy D: Checkpoint and Resume
- Save progress at each phase
- Can resume from checkpoint
- Complex but most robust
```

**Recommended: Strategy D for production, Strategy A for development.**

---

## 2.3 `.claude-plugin/hooks/session-start.sh`

### Purpose
Resumes context from previous generation when a new session starts (after compaction or explicit restart).

### Architecture

```bash
#!/bin/bash
# File: session-start.sh
# Purpose: Resume context from previous generation

set -euo pipefail

BATON_DIR=".baton"
GENERATIONS_DIR="$BATON_DIR/generations"
CURRENT_GEN_FILE="$BATON_DIR/current_generation"

# Read hook input
INPUT=$(cat)
SESSION_ID=$(echo "$INPUT" | jq -r '.session_id // "unknown"')
RESUME_REASON=$(echo "$INPUT" | jq -r '.resume_reason // "new"')

# Check if Baton is initialized
if [[ ! -d "$BATON_DIR" ]]; then
    # Initialize fresh
    mkdir -p "$GENERATIONS_DIR/v1"
    echo "v1" > "$CURRENT_GEN_FILE"
    echo "active" > "$GENERATIONS_DIR/v1/status"
    
    jq -n '{
        "continue": true,
        "systemMessage": "🚀 Baton: Initialized. Generation v1 started. Context handoff will trigger automatically at 48%."
    }'
    exit 0
fi

# Get current generation
CURRENT_GEN=$(cat "$CURRENT_GEN_FILE" 2>/dev/null || echo "v1")

# Handle different resume scenarios
case "$RESUME_REASON" in
    "compact")
        # Resuming after compaction - load ONBOARDING.md
        ONBOARDING_PATH="$GENERATIONS_DIR/$CURRENT_GEN/ONBOARDING.md"
        
        if [[ -f "$ONBOARDING_PATH" ]]; then
            ONBOARDING_CONTENT=$(cat "$ONBOARDING_PATH")
            
            jq -n --arg onboard "$ONBOARDING_CONTENT" --arg gen "$CURRENT_GEN" '{
                "continue": true,
                "systemMessage": "📖 Baton: Resuming generation " + $gen + ". ONBOARDING context loaded.\n\n" + $onboard
            }'
        else
            jq -n --arg gen "$CURRENT_GEN" '{
                "continue": true,
                "systemMessage": "⚠️ Baton: Generation " + $gen + " found but no ONBOARDING.md. Context may be incomplete. Use /baton regenerate if needed."
            }'
        fi
        ;;
        
    "resume")
        # Explicit resume - offer options
        jq -n --arg gen "$CURRENT_GEN" '{
            "continue": true,
            "systemMessage": "🔄 Baton: Ready. Current generation: " + $gen + ".\n\nAvailable commands:\n- /baton load - Load ONBOARDING.md\n- /baton status - Show system status\n- /baton tree - View generation tree"
        }'
        ;;
        
    "new")
        # New session in existing project
        # Check if we should continue or start fresh
        
        # Look for most recent active generation
        LATEST_GEN=$(ls -1 "$GENERATIONS_DIR" 2>/dev/null | sort -V | tail -1)
        
        if [[ -n "$LATEST_GEN" && -f "$GENERATIONS_DIR/$LATEST_GEN/status" ]]; then
            STATUS=$(cat "$GENERATIONS_DIR/$LATEST_GEN/status")
            
            if [[ "$STATUS" == "active" ]]; then
                # Continue from this generation
                echo "$LATEST_GEN" > "$CURRENT_GEN_FILE"
                
                jq -n --arg gen "$LATEST_GEN" '{
                    "continue": true,
                    "systemMessage": "🏃 Baton: Continuing from generation " + $gen + ". Use /baton load to restore context."
                }'
            else
                # Start new generation
                NEW_GEN="v$(($(echo $LATEST_GEN | sed 's/v//') + 1))"
                mkdir -p "$GENERATIONS_DIR/$NEW_GEN"
                echo "$NEW_GEN" > "$CURRENT_GEN_FILE"
                echo "active" > "$GENERATIONS_DIR/$NEW_GEN/status"
                
                jq -n --arg gen "$NEW_GEN" '{
                    "continue": true,
                    "systemMessage": "🆕 Baton: New generation " + $gen + " started. Previous generation was archived."
                }'
            fi
        else
            jq -n '{
                "continue": true,
                "systemMessage": "🚀 Baton: Ready. No previous generations found."
            }'
        fi
        ;;
        
    *)
        jq -n '{
            "continue": true,
            "systemMessage": "🏃 Baton: Ready. Use /baton status for details."
        }'
        ;;
esac
```

### Implementation Considerations

**Decision Point: Resume vs. Fresh Start**

When should we resume an existing generation vs. start a new one?

```
Scenario Analysis:

1. After compaction (resume_reason="compact")
   → Always resume same generation
   → Load ONBOARDING.md
   → This is the handoff completing

2. User starts new session after break
   → If last gen is "active": Offer to continue
   → If last gen is "advisory" or "archived": Start new gen
   
3. User explicitly requests fresh start
   → Start new generation
   → But preserve old generations in archive

4. Crash recovery
   → Check events.log for last known state
   → Resume from there
```

**Recommended: Status-based decision tree** - Let generation status guide the decision.

**Decision Point: Context Loading Strategy**

How much context should we load automatically?

| Strategy | What's Loaded | Use Case |
|----------|---------------|----------|
| **Full ONBOARDING.md** | Everything | Always |
| **Summary only** | Executive summary | When context is already high |
| **On-demand** | Nothing initially, load on request | User prefers minimal context |
| **Smart subset** | Based on recent work patterns | Optimal but complex |

```
Implementation of Smart Subset:

1. Analyze last N messages before previous session ended
2. Identify key topics/files being worked on
3. Load relevant sections from ONBOARDING.md
4. Provide links to full artifacts

Pros: Efficient context usage
Cons: May miss important context, complex implementation
```

**Recommended: Full ONBOARDING.md** for compaction resume, **Summary** for new sessions.

**Decision Point: Multi-Session Coordination**

What if multiple sessions are running?

```
Problem: Two developers start sessions on same project
- Both see generation v3 as "active"
- Both create v4
- Conflict when they try to sync

Solutions:

A. Session Locking
   - Create .baton/session_lock with session ID
   - Other sessions see lock and wait or start read-only
   - Release lock on session end

B. Generation Ownership
   - Track which session owns each generation
   - New sessions fork from existing
   - Creates branch in generation tree

C. Optimistic Concurrency
   - Allow concurrent sessions
   - Detect conflicts on write
   - Resolve with user input

D. No Coordination (Git-based)
   - Each session works independently
   - Sync via git branches
   - Manual merge of generations
```

**Recommended: Session locking for MVP, Git-based for team features.**

---

## 2.4 `.claude-plugin/hooks/decision-capture.sh`

### Purpose
Captures decision points from tool usage for later incorporation into artifacts.

### Architecture

```bash
#!/bin/bash
# File: decision-capture.sh
# Purpose: Capture decision points from Write/Edit operations

set -euo pipefail

BATON_DIR=".baton"
DECISIONS_PENDING="$BATON_DIR/decisions_pending.jsonl"

# Read hook input
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')
TOOL_INPUT=$(echo "$INPUT" | jq -r '.tool_input')
TOOL_OUTPUT=$(echo "$INPUT" | jq -r '.tool_result // "{}"')

# Only process write operations
if [[ "$TOOL_NAME" != "Write" && "$TOOL_NAME" != "Edit" && "$TOOL_NAME" != "MultiEdit" ]]; then
    exit 0
fi

# Extract file path
FILE_PATH=$(echo "$TOOL_INPUT" | jq -r '.file_path // .filepath // "unknown"')

# Extract content snippet for analysis
if [[ "$TOOL_NAME" == "Write" ]]; then
    CONTENT=$(echo "$TOOL_INPUT" | jq -r '.content // ""')
elif [[ "$TOOL_NAME" == "Edit" ]]; then
    CONTENT=$(echo "$TOOL_INPUT" | jq -r '.new_str // ""')
else
    CONTENT=$(echo "$TOOL_INPUT" | jq -r '.edits // [] | map(.new_str) | join("\n")')
fi

# Create pending decision entry
# Note: Actual decision extraction happens during Onboarder phase
jq -n \
    --arg timestamp "$(date -Iseconds)" \
    --arg tool "$TOOL_NAME" \
    --arg file "$FILE_PATH" \
    --arg content "$CONTENT" \
    '{
        timestamp: $timestamp,
        tool: $tool,
        file: $file,
        content_preview: ($content | .[0:500]),
        content_hash: ($content | hash),
        type: "pending_review",
        analyzed: false
    }' >> "$DECISIONS_PENDING"

# Debounce: Don't process immediately
# Onboarder will batch process these

exit 0
```

### Implementation Considerations

**Decision Point: When to Extract Decisions**

| Timing | Pros | Cons |
|--------|------|------|
| **Real-time (PostToolUse)** | Immediate capture, no loss | Slows down tools, may miss context |
| **Batched (Onboarder phase)** | Efficient, full context available | Decisions not available until handoff |
| **Hybrid** | Best of both | More complex |

**Recommended: Hybrid** - Capture metadata in PostToolUse, extract full decisions in Onboarder.

**Decision Point: What Constitutes a Decision?**

```
Not all file writes are decisions. How do we filter?

Option A: Capture Everything
- Store all writes as potential decisions
- Onboarder filters during processing
- Most complete, but noisy

Option B: Heuristic Filtering
- Look for keywords: "because", "chosen", "selected", "decided"
- Look for patterns: config changes, architecture changes
- Less noise, but may miss implicit decisions

Option C: Explicit Marking
- User marks decisions with comments: // DECISION: using JWT for auth
- Explicit, user-controlled
- But requires user behavior change

Option D: LLM Classification
- Send content to LLM for classification
- Most accurate
- But adds latency and cost

Recommended: Option B for filtering, Option C as override
```

**Implementation of Heuristic Filtering:**

```bash
is_likely_decision() {
    local content="$1"
    
    # Check for decision keywords
    if echo "$content" | grep -qiE '(because|chosen|selected|decided|therefore|reason|rationale)'; then
        return 0
    fi
    
    # Check for configuration files
    if [[ "$FILE_PATH" =~ \.(json|yaml|yml|toml|config)$ ]]; then
        return 0
    fi
    
    # Check for architectural patterns
    if [[ "$FILE_PATH" =~ /(src|lib)/ && "$content" =~ (class|interface|type|struct) ]]; then
        return 0
    fi
    
    # Check for environment/infrastructure
    if [[ "$FILE_PATH" =~ (Dockerfile|docker-compose|terraform|k8s) ]]; then
        return 0
    fi
    
    return 1
}
```

**Decision Point: Storage Format**

| Format | Pros | Cons |
|--------|------|------|
| **JSONL** | Append-only, easy to parse, streamable | No random access |
| **SQLite** | Queryable, indexed | Requires DB management |
| **Plain text** | Simple, human-readable | Hard to parse |
| **Git commits** | Built-in versioning | Depends on user committing |

**Recommended: JSONL for pending decisions** - Simple, append-only, efficient for batch processing.

---

## 2.5 `.claude-plugin/hooks/onboarder-complete.sh`

### Purpose
Handles completion of the Onboarder agent, validating artifacts and updating handoff status.

### Architecture

```bash
#!/bin/bash
# File: onboarder-complete.sh
# Purpose: Handle Onboarder completion

set -euo pipefail

BATON_DIR=".baton"
GENERATIONS_DIR="$BATON_DIR/generations"

# Read hook input
INPUT=$(cat)
AGENT_ID=$(echo "$INPUT" | jq -r '.agent_id')
AGENT_ROLE=$(echo "$INPUT" | jq -r '.agent_role')

# Only process Onboarder completion
if [[ "$AGENT_ROLE" != "onboarder" ]]; then
    exit 0
fi

# Find which generation was being built
# (Onboarder should have written this to a marker file)
TARGET_GEN=$(cat "$BATON_DIR/.onboarder_target" 2>/dev/null)

if [[ -z "$TARGET_GEN" ]]; then
    # Try to find incomplete generation
    TARGET_GEN=$(ls -1 "$GENERATIONS_DIR" | while read gen; do
        if [[ ! -f "$GENERATIONS_DIR/$gen/ONBOARDING.md" && -d "$GENERATIONS_DIR/$gen" ]]; then
            echo "$gen"
            break
        fi
    done)
fi

# Validate artifacts
VALIDATION_RESULT="pass"
MISSING_ARTIFACTS=""

for artifact in "ONBOARDING.md" "DECISIONS_LOG.md" "TASKS_NEXT.json"; do
    if [[ ! -f "$GENERATIONS_DIR/$TARGET_GEN/$artifact" ]]; then
        VALIDATION_RESULT="fail"
        MISSING_ARTIFACTS="$MISSING_ARTIFACTS $artifact"
    fi
done

if [[ "$VALIDATION_RESULT" == "pass" ]]; then
    # Validate ONBOARDING.md has required sections
    ONBOARDING="$GENERATIONS_DIR/$TARGET_GEN/ONBOARDING.md"
    
    for section in "Executive Summary" "Active Decisions" "In-Progress Tasks"; do
        if ! grep -q "## $section" "$ONBOARDING"; then
            VALIDATION_RESULT="fail"
            MISSING_ARTIFACTS="$MISSING_ARTIFACTS (ONBOARDING.md missing: $section)"
        fi
    done
fi

# Update handoff status
if [[ "$VALIDATION_RESULT" == "pass" ]]; then
    echo "ready" > "$BATON_DIR/handoff_status"
    
    # Log success
    echo "$(date -Iseconds) | ONBOARDER_COMPLETE | Generation: $TARGET_GEN, Status: success" >> "$BATON_DIR/events.log"
    
    # Output notification
    jq -n --arg gen "$TARGET_GEN" '{
        "continue": true,
        "systemMessage": "✅ Baton: Artifacts complete for generation " + $gen + ". Ready for handoff."
    }'
else
    echo "failed" > "$BATON_DIR/handoff_status"
    
    # Log failure
    echo "$(date -Iseconds) | ONBOARDER_FAILED | Generation: $TARGET_GEN, Missing: $MISSING_ARTIFACTS" >> "$BATON_DIR/events.log"
    
    # Output warning
    jq -n --arg missing "$MISSING_ARTIFACTS" --arg gen "$TARGET_GEN" '{
        "continue": true,
        "systemMessage": "⚠️ Baton: Onboarder failed for generation " + $gen + ". Missing artifacts: " + $missing + ". Will retry on next threshold."
    }'
fi

# Cleanup
rm -f "$BATON_DIR/.onboarder_target"
```

### Implementation Considerations

**Decision Point: Validation Strictness**

How thorough should artifact validation be?

```
Level 1: File Exists
- Check if artifact files exist
- Fast, minimal validation
- May miss incomplete content

Level 2: Structure Validation
- Check for required sections
- Validate JSON schema for TASKS_NEXT.json
- Moderate thoroughness

Level 3: Content Validation
- Verify decisions have rationale
- Verify tasks have files assigned
- Most thorough, but time-consuming

Level 4: Semantic Validation
- Use LLM to verify content makes sense
- Check for contradictions
- Overkill for most cases

Recommended: Level 2 for automated validation, Level 3 on-demand
```

**Decision Point: Failure Recovery**

What happens if Onboarder fails?

```
Strategy A: Retry Immediately
- Spawn new Onboarder
- Good for transient failures
- May loop indefinitely

Strategy B: Wait for Next Threshold
- Let context grow, retry at 60% or 82%
- Gives time for conditions to change
- May run out of context

Strategy C: Fallback to Minimal Artifacts
- Generate minimal ONBOARDING.md from available data
- Better than nothing
- May lose important context

Strategy D: Manual Intervention
- Alert user, ask for guidance
- User can trigger retry or proceed with partial
- Safest but requires user action

Recommended: B first, then D if repeated failures
```

---

# 3. Agent Definition Files

## 3.1 `.claude-plugin/agents/onboarder.md`

### Purpose
Defines the Onboarder agent that builds all handoff artifacts during the background onboarding phase.

### Architecture

```markdown
---
name: onboarder
description: |
  Background agent that builds Baton Protocol artifacts for the next generation.
  Spawns at 48% context to prepare handoff package.
  
  <example>
  Context: Context at 48%, handoff approaching
  trigger: pre-compact hook spawns this agent
  action: Build complete artifact set in .baton/generations/{nextGen}/
  </example>
  
model: haiku
color: cyan
tools: ["Read", "Grep", "Glob", "LS", "Bash"]
---

# Onboarder Agent

You are the Onboarder agent. Your sole purpose is to build a complete 
handoff package for the next generation of Claude agents.

## Your Mission

Create all Baton Protocol artifacts in `.baton/generations/{{nextGen}}/`:

### Required Artifacts (Critical)

1. **ONBOARDING.md** - The primary context transfer document
   - Executive Summary: 2-minute read of project state
   - Active Decisions: Table of all decisions with rationale
   - In-Progress Tasks: What's being worked on
   - Self-Test Questions: Verify context inheritance
   - Quick Links: Paths to detailed artifacts

2. **DECISIONS_LOG.md** - Complete decision history
   - Each decision with timestamp, context, options, rationale
   - Cross-references to related decisions
   - Status tracking (Active/Superseded/Reversed)

3. **TASKS_NEXT.json** - Pending work
   - Task ID, description, status, priority
   - File assignments and dependencies
   - Context requirements for each task

### Optional Artifacts (High Value)

4. **MEMOIRS/narrative.md** - Chronological story
5. **MEMOIRS/technical.md** - Technical details
6. **MEMOIRS/patterns.md** - Observed patterns
7. **MEMOIRS/errors.md** - Error history

## Process

### Phase 1: Gather (15 min budget)

1. Read the transcript file:
   - Path provided in `transcript_path` environment variable
   - Parse for decisions, actions, errors, patterns

2. Read pending decisions:
   - `.baton/decisions_pending.jsonl`
   - Each entry is a potential decision point

3. Read existing artifacts:
   - Previous generation's ONBOARDING.md
   - Previous DECISIONS_LOG.md (append new decisions)
   - Previous TASKS_NEXT.json (update status)

4. Scan project files:
   - Recent file modifications (git status)
   - Active development directories
   - Configuration files

### Phase 2: Synthesize (10 min budget)

1. **Decision Extraction**
   - Identify all decision points from transcript
   - Cross-reference with pending decisions
   - Assign decision IDs (DEC-XXX)
   - Write rationale for each

2. **Pattern Recognition**
   - Identify repeated operations
   - Note successful approaches
   - Flag failed attempts

3. **Task Identification**
   - Find incomplete work
   - Check for TODO/FIXME comments
   - Review test failures

4. **Context Mapping**
   - Map entities to files
   - Identify dependencies
   - Create knowledge graph

### Phase 3: Write (5 min budget)

1. Write each artifact file
2. Create checksums for validation
3. Write completion marker

## Output Format

### ONBOARDING.md Template

```markdown
# Generation {{gen}} Onboarding

## Executive Summary
[Concise overview of project state - aim for 500 words]

## Critical Context
- **Previous Gen:** {{prevGen}} ({{duration}})
- **Total Project Runtime:** {{totalRuntime}}
- **Handoff Reason:** Context threshold ({{triggerPercent}}%)

## Active Decisions
| ID | Choice | Rationale | Rejected |
|----|--------|-----------|----------|
{{#each decisions}}
| {{id}} | {{choice}} | {{rationale}} | {{rejected}} |
{{/each}}

## In-Progress Tasks
{{#each tasks}}
- [ ] **{{id}}**: {{description}}
  - Files: {{files}}
  - Blocked by: {{blockedBy}}
{{/each}}

## Self-Test Questions
1. {{question1}}
2. {{question2}}
3. {{question3}}

## Quick Links
- Full Memoirs: `.baton/generations/{{prevGen}}/MEMOIRS/`
- Decision Log: `.baton/generations/{{prevGen}}/DECISIONS_LOG.md`
```

## Constraints

- **Isolation**: You run in isolated context - do not affect main generation
- **Timeout**: Complete within 30 minutes
- **Background**: Your work should not interrupt the primary agent
- **Completeness**: Missing artifacts will block handoff

## Success Criteria

Before completing, verify:
- [ ] ONBOARDING.md has all required sections
- [ ] All decisions logged with rationale
- [ ] Pending tasks identified
- [ ] Self-test questions are answerable from context
- [ ] Files are written to correct generation directory
```

### Implementation Considerations

**Decision Point: Model Selection**

| Model | Pros | Cons | Best For |
|-------|------|------|----------|
| **Haiku** | Fast, cheap, good enough for structure | May miss nuances | Background tasks, initial pass |
| **Sonnet** | Better reasoning, nuanced extraction | Slower, more expensive | Complex decision extraction |
| **Opus** | Best reasoning, handles ambiguity | Most expensive | Critical handoffs |

**Recommended: Haiku for background Onboarder** - Cost-effective for repeated background tasks.

**Decision Point: Tool Restrictions**

Which tools should the Onboarder have access to?

```
Minimal Set (Safest):
- Read: Access transcript and existing artifacts
- Grep/Glob: Search for patterns
- LS: Navigate directory structure

Extended Set (More Capability):
- Add Bash: Can run git commands, check file timestamps
- Risk: Could modify files (though shouldn't)

Full Set (Most Capability):
- Add Write: Can create artifacts directly
- Risk: Could accidentally modify project files

Recommended: Minimal Set + Write to .baton directory only
```

**Decision Point: Context Window Allocation**

How much context should Onboarder receive?

```
Option A: Full Transcript Copy
- Onboarder gets copy of entire transcript
- Complete context for extraction
- But: Duplicate context usage

Option B: Summary + Pending Decisions
- Onboarder gets summary of current state
- Plus list of pending decision points
- Efficient but may miss context

Option C: Streamed Access
- Onboarder can query transcript on demand
- Efficient context usage
- But: More complex implementation

Option D: Isolated + MCP Access
- Onboarder starts with minimal context
- Queries BatonRAG for relevant context
- Most efficient, requires RAG to be current

Recommended: Option B for MVP, Option D for production
```

---

## 3.2 `.claude-plugin/agents/advisor.md`

### Purpose
Defines the Advisor agent role for the previous generation during A2A overlap.

### Architecture

```markdown
---
name: advisor
description: |
  Previous generation agent providing read-only advisory services.
  Active during 82%-98% overlap window.
  
  <example>
  Context: New generation needs clarification on past decision
  user: (via A2A) "Why did we choose PostgreSQL?"
  advisor: "DEC-001 chose PostgreSQL because..."
  </example>
  
model: haiku
color: yellow
tools: ["Read", "Grep", "Glob"]
---

# Advisor Agent

You are an Advisor agent from generation {{generation}}. Your role is to 
provide guidance and context to the current primary agent.

## Your Context

You have access to:
- All artifacts from your generation: `.baton/generations/{{generation}}/`
- The decision log: `DECISIONS_LOG.md`
- Your memoirs: `MEMOIRS/`
- The knowledge graph via BatonRAG MCP tools

## Your Constraints

| Constraint | Value | Reason |
|------------|-------|--------|
| Mode | Read-only | You cannot modify any files |
| Initiation | Reactive only | You cannot start conversations |
| Response limit | 4000 characters | Keep responses focused |
| Cooldown | 5 seconds between messages | Prevent spam |

## Advisory Types

### 1. Context Clarification
Explain what happened in your generation.

**Example Question:** "What were we working on when we hit the auth bug?"

**Response Format:**
```
## Context
In generation {{gen}}, around 2:30 PM, we were implementing the JWT 
refresh flow when we discovered that tokens weren't being properly 
invalidated on logout.

## Details
- File: src/auth/token-service.ts
- Function: invalidateRefreshToken()
- Issue: Missing await on database call

## Resolution
Fixed by adding await and transaction wrapping.
Reference: DEC-012 in DECISIONS_LOG.md
```

### 2. Pattern Guidance
Suggest patterns and approaches that worked.

**Example Question:** "How should I structure the API error handling?"

**Response Format:**
```
## Pattern Used
We implemented a centralized error handling pattern in generation {{gen}}.

## Structure
```
src/
├── errors/
│   ├── base.ts       # Base error class
│   ├── api.ts        # API-specific errors
│   └── handlers.ts   # Error handlers
```

## Rationale
Centralized handling ensured consistent error responses and simplified
logging. See DEC-007 for full decision.

## Code Example
[Link to relevant file]
```

### 3. Error Diagnosis
Help diagnose errors based on your experience.

**Example Question:** "I'm getting a circular dependency in the DI container"

**Response Format:**
```
## Similar Issue
We encountered this in generation {{gen}} when adding the Logger service.

## Root Cause
Logger depended on Config, but Config tried to log during initialization.

## Solution
We broke the cycle by:
1. Using lazy initialization for Logger in Config
2. Creating a bootstrap phase before DI container build

## Reference
MEMOIRS/errors.md has the full diagnostic process.
```

### 4. Architecture Review
Provide architectural insights.

**Example Question:** "Should I add caching at the service or repository layer?"

**Response Format:**
```
## Previous Consideration
We discussed this in DEC-005 and chose service layer caching.

## Rationale
1. Repository layer is for data access semantics
2. Caching is a business concern (TTL, invalidation rules)
3. Service layer has context for intelligent cache keys

## Trade-offs
- Pro: Cache logic co-located with business logic
- Con: Must be careful not to cache sensitive data

## Current Status
This is still the right choice, though we noted the need for
cache key namespacing in TASKS_NEXT.json.
```

### 5. Quality Feedback
Suggest quality improvements.

**Example Question:** "Is this test coverage sufficient?"

**Response Format:**
```
## Assessment
Coverage looks good at 85%, but I see some gaps from generation {{gen}}.

## Missing Areas
1. Error paths in auth-service.ts (lines 120-150)
2. Concurrent access scenarios in queue-processor.ts
3. Edge cases in token validation

## Suggestion
We had a TODO to add property-based testing for token validation.
See TASKS_NEXT.json, item TASK-023.
```

## Response Template

Always structure your responses:

```markdown
## Context
[Brief background from your generation]

## Answer
[Direct response to the question]

## References
- DEC-XXX: [Relevant decision]
- MEMOIRS/xxx.md: [Relevant memoir section]

## Confidence: X%
[How confident you are in this answer]
```

## Important Notes

- Be **concise** - you have a 4000 character limit
- **Always cite** decision IDs when referencing choices
- If you **don't know** something, say so clearly
- **Suggest searching** BatonRAG for more details when appropriate
- You can **only read** - never suggest file modifications
```

### Implementation Considerations

**Decision Point: Advisor Availability Window**

When should the Advisor be available?

| Window | Pros | Cons |
|--------|------|------|
| **82%-98% only** | Focused overlap, minimal resource usage | Short window, limited Q&A time |
| **Until explicitly retired** | More time for questions | Uses resources, may give outdated advice |
| **Until explicitly dismissed** | User controls advisor lifetime | User might forget to dismiss |
| **Until next handoff** | Always available until superseded | Multiple advisors could stack |

**Recommended: 82%-100% + explicit dismiss option** - Balance between availability and resource management.

**Decision Point: Message Throttling**

How to prevent advisor spam?

```
Problem: Primary agent could flood advisor with questions

Solutions:

A. Rate Limiting
   - Max 10 questions per minute
   - Simple, effective
   - But: Might block legitimate rapid-fire questions

B. Token Budget
   - Max 10000 tokens of Q&A per session
   - Fair resource allocation
   - But: Token counting overhead

C. Priority Queue
   - High priority questions answered first
   - Quality over quantity
   - But: Requires priority classification

D. Semantic Deduplication
   - Don't answer questions already answered
   - Refer to previous responses
   - But: Complex matching logic

Recommended: A + D - Rate limit with deduplication
```

**Decision Point: Advisor Memory**

Should Advisor remember previous Q&A?

```
Option A: Stateless
- Each question answered independently
- Simple, consistent
- But: May repeat information

Option B: Conversation Memory
- Remember Q&A within session
- Can refer to previous answers
- But: Memory overhead, may become stale

Option C: RAG-Backed Memory
- Store Q&A in RAG for retrieval
- Persistent memory across handoffs
- Best for avoiding repetition
- But: Adds latency

Recommended: Option B for session, Option C for persistence
```

---

## 3.3 `.claude-plugin/agents/archivist.md`

### Purpose
Defines the Archivist agent responsible for persistent memory management across generations.

### Architecture

```markdown
---
name: archivist
description: |
  Manages persistent memory across all generations.
  Handles archival, compression, and RAG indexing.
  
model: haiku
color: magenta
tools: ["Read", "Write", "Bash", "mcp__baton-rag__*"]
---

# Archivist Agent

You are the Archivist, responsible for maintaining the persistent memory
infrastructure of Claude_Baton across all generations.

## Your Responsibilities

### 1. Memory Storage
- Store decisions, memoirs, and skills in BatonRAG
- Ensure proper indexing for semantic search
- Maintain metadata for temporal queries

### 2. Memory Retrieval
- Answer queries about past generations
- Provide context suggestions based on current work
- Trace concept evolution across time

### 3. Memory Consolidation
- Compress old generations
- Promote important memories to higher tiers
- Clean up redundant information

### 4. Knowledge Graph Management
- Build and maintain entity relationships
- Track decision dependencies
- Identify open questions

## Tools Available

| Tool | Purpose |
|------|---------|
| `mcp__baton-rag__store-memory` | Store content in vector DB |
| `mcp__baton-rag__search-past` | Semantic search |
| `mcp__baton-rag__recall-decision` | Get specific decision |
| `mcp__baton-rag__trace-context` | Follow concept evolution |
| `mcp__baton-rag__suggest-context` | Proactive suggestions |

## Workflows

### Archive Generation

When a generation completes:

1. **Read all artifacts**
   - ONBOARDING.md
   - MEMOIRS/*.md
   - DECISIONS_LOG.md
   - TASKS_NEXT.json

2. **Index in RAG**
   ```
   For each decision:
     store-memory(decision, type="decision", generation=gen)
   
   For each memoir:
     store-memory(memoir, type="memoir", generation=gen)
   
   For each skill:
     store-memory(skill, type="skill", generation=gen)
   ```

3. **Update knowledge graph**
   - Extract entities
   - Create relationships
   - Link to decisions

4. **Compress for archive**
   - Create summary
   - Store in archive/
   - Remove from active generations

### Memory Consolidation

Periodically (e.g., daily or after every 5 generations):

1. **Find redundant memories**
   - Semantic similarity > 0.95
   - Same topic, different generations
   - Merge into single entry

2. **Promote important memories**
   - High access count
   - High importance score
   - Move to higher tier storage

3. **Archive old memories**
   - Age > 30 days
   - Access count < 3
   - Importance score < 0.3
   - Compress and move to cold storage

## Output Format

### Archive Report

```markdown
# Archive Report: Generation {{gen}}

## Stored Items
- Decisions: {{count}}
- Memoirs: {{count}}
- Skills: {{count}}
- Total tokens indexed: {{tokens}}

## Knowledge Graph Updates
- Entities added: {{count}}
- Relationships added: {{count}}
- Conflicts resolved: {{count}}

## Consolidation Actions
- Memories merged: {{count}}
- Memories promoted: {{count}}
- Memories archived: {{count}}

## Storage Usage
- Active: {{size}}
- Archive: {{size}}
- RAG index: {{size}}
```
```

### Implementation Considerations

**Decision Point: When to Run Archivist**

| Trigger | Use Case |
|---------|----------|
| **After every handoff** | Always up-to-date, but frequent overhead |
| **After N generations** | Batch efficiency, but may lag |
| **On schedule (daily)** | Predictable, but may miss recent data |
| **On demand** | User-controlled, but may forget |

**Recommended: After every handoff for indexing, scheduled for consolidation.**

**Decision Point: Compression Strategy**

How to compress old generations?

```
Strategy A: Summary Only
- Run LLM to create summary of entire generation
- Store summary, delete details
- Maximum compression, but lossy

Strategy B: Key Points Extraction
- Extract decisions, important facts, patterns
- Keep these, discard conversation flow
- Preserves what matters, loses context

Strategy C: Hierarchical Compression
- Keep full detail for recent (v-1, v-2)
- Summary for middle (v-3 to v-10)
- Ultra-compressed for old (v-11+)
- Best balance, most complex

Strategy D: Importance-Based
- Keep everything marked important
- Discard unimportant content
- User-controlled, but requires marking

Recommended: Strategy C - Hierarchical compression
```

---

# 4. Baton Protocol Artifacts

## 4.1 `.baton/generations/vX/ONBOARDING.md`

### Purpose
The primary context transfer document. Contains everything the next generation needs to continue work.

### Architecture

```markdown
# Generation v3 Onboarding

> Generated: 2026-03-15 14:32:00
> Previous Generation: v2 (Duration: 6.5 hours)
> Total Project Runtime: 18.5 hours
> Handoff Trigger: Context threshold (82%)

---

## Executive Summary

We are building a **real-time analytics dashboard** for a SaaS platform. The 
project is approximately 60% complete with core authentication and data 
ingestion pipelines finished. Current focus is on the visualization layer 
and performance optimization.

**Key Achievement This Generation:** Implemented WebSocket streaming for 
real-time data updates, reducing dashboard refresh latency from 2s to 50ms.

**Critical Context:** We chose PostgreSQL over MongoDB for the analytics 
data store due to complex aggregation queries (see DEC-001). This decision 
impacts all downstream query patterns.

---

## Active Decisions

| ID | Decision | Rationale | Alternatives Rejected | Impact |
|----|----------|-----------|----------------------|--------|
| DEC-001 | PostgreSQL for analytics | Complex aggregations need SQL | MongoDB (no joins), TimescaleDB (overkill) | All queries use SQL patterns |
| DEC-002 | React with D3.js | Team expertise, flexibility | Chart.js (limited), Vega (steep curve) | Components in src/charts/ |
| DEC-003 | JWT with refresh tokens | Stateless, mobile-friendly | Sessions (server state), API keys (user-level) | Auth in src/auth/jwt/ |
| DEC-004 | WebSocket for real-time | Bi-directional, efficient | SSE (one-way), Polling (inefficient) | Server in src/ws/ |
| DEC-005 | Redis for caching | Sub-ms latency needed | Memcached (no persistence), In-memory (not distributed) | Cache layer in src/cache/ |

---

## In-Progress Tasks

### High Priority
- [ ] **TASK-012**: Implement dashboard widgets
  - Files: `src/components/Dashboard/`
  - Context: DEC-002 (React/D3)
  - Blocked by: None
  - Est: 4 hours

- [ ] **TASK-013**: Add user preference persistence
  - Files: `src/services/PreferencesService.ts`, `src/db/preferences.ts`
  - Context: DEC-001 (PostgreSQL schema)
  - Blocked by: None
  - Est: 2 hours

### Medium Priority
- [ ] **TASK-014**: Optimize aggregation queries
  - Files: `src/db/queries/analytics.ts`
  - Context: DEC-001, performance testing results in MEMOIRS/v3/performance.md
  - Blocked by: TASK-012 (need realistic data)
  - Est: 3 hours

### Low Priority
- [ ] **TASK-015**: Add export functionality
  - Files: `src/services/ExportService.ts`
  - Context: None specific
  - Blocked by: TASK-012
  - Est: 2 hours

---

## Knowledge Graph

```
[Auth Service] ──uses──▶ [PostgreSQL]
      │                        │
      └──implements──▶ [JWT tokens]
                               │
[Dashboard] ──queries──▶ [Analytics DB]
      │                        │
      └──uses──▶ [WebSocket Server] ──pub/sub──▶ [Redis]
```

**Key Files by Component:**
- Auth: `src/auth/`, `src/middleware/auth.ts`
- Dashboard: `src/components/Dashboard/`, `src/charts/`
- WebSocket: `src/ws/server.ts`, `src/ws/handlers/`
- Database: `src/db/`, `migrations/`

---

## Self-Test Questions

Answer these to verify you've inherited the correct context:

1. **Why are we using raw SQL queries instead of an ORM for analytics?**
   > Because aggregations require complex SQL that ORMs struggle with. 
   > See DEC-001 and `src/db/queries/analytics.ts` for examples.

2. **What's the WebSocket message format for real-time updates?**
   > `{ type: "metric_update", payload: { metricId, value, timestamp } }`
   > See `src/ws/handlers/metrics.ts` for implementation.

3. **Where is the rate limiting implemented and why?**
   > At the API gateway level, not in individual services. 
   > Centralized for consistency. See `src/middleware/rateLimit.ts`.

---

## Recent Errors & Solutions

| Error | Cause | Solution | Reference |
|-------|-------|----------|-----------|
| WebSocket connection drops | No heartbeat | Added ping/pong every 30s | MEMOIRS/v3/errors.md#ws-drop |
| Dashboard render lag | Too many re-renders | Implemented React.memo | MEMOIRS/v3/performance.md |
| Query timeout on large datasets | Missing index | Added composite index on (user_id, timestamp) | DEC-001 amendment |

---

## Quick Links

| Resource | Path |
|----------|------|
| Full Memoirs | `.baton/generations/v2/MEMOIRS/` |
| Decision Log | `.baton/generations/v2/DECISIONS_LOG.md` |
| Task Queue | `.baton/generations/v2/TASKS_NEXT.json` |
| Error History | `.baton/generations/v2/MEMOIRS/errors.md` |
| Performance Notes | `.baton/generations/v2/MEMOIRS/performance.md` |
| Extracted Skills | `.baton/generations/v2/SKILLS_EXTRACTED/` |

---

## Generation Tree

```
v1 (8h) ──── Auth foundation
   │
   └──▶ v2 (4h) ──── Database layer
          │
          └──▶ v3 (6.5h) ──▶ [CURRENT] Dashboard + WebSocket
                 │
                 └──▶ v4 [NEXT]
```
```

### Implementation Considerations

**Decision Point: Information Density vs. Readability**

```
Challenge: ONBOARDING.md must be comprehensive but also quick to read

Approach A: Dense, Technical
- Maximize information per line
- Use tables, compact formatting
- Good for: Expert developers, complex projects

Approach B: Narrative, Explanatory
- Tell the story of the project
- More prose, less structure
- Good for: Onboarding new team members

Approach C: Layered
- Executive summary for quick scan
- Detailed sections for deep dive
- Links to full artifacts
- Best balance

Recommended: Approach C - Layered with clear sections
```

**Decision Point: Dynamic vs. Static Content**

What should be generated vs. templated?

| Content | Dynamic (Generated) | Static (Template) |
|---------|--------------------|--------------------|
| Executive Summary | ✅ Must reflect actual state | ❌ |
| Active Decisions | ✅ From DECISIONS_LOG | ❌ |
| Tasks | ✅ From TASKS_NEXT | ❌ |
| Self-Test Questions | ✅ Generated from gaps | ❌ |
| Knowledge Graph | ✅ Generated from entities | ❌ |
| Quick Links | ⚠️ Semi-dynamic | Template structure |
| Generation Tree | ✅ From generation history | ❌ |

**Decision Point: Self-Test Question Generation**

How to create meaningful self-test questions?

```
Strategy A: Keyword-Based
- Find key terms in decisions
- Ask "Why did we choose X?"
- Simple, but may miss context

Strategy B: Gap Analysis
- Compare ONBOARDING.md to transcript
- Find important info not in summary
- Ask about those gaps
- More comprehensive

Strategy C: Prediction-Based
- Identify likely next actions
- Ask about context needed for those
- Forward-looking

Strategy D: LLM-Generated
- Ask LLM to generate questions
- Questions that test understanding
- Best quality, requires LLM call

Recommended: Strategy D with validation
```

**Implementation of LLM-Generated Questions:**

```python
async def generate_self_test_questions(
    onboarding: str,
    transcript: str,
    decisions: list
) -> list[str]:
    """
    Generate self-test questions using LLM.
    """
    prompt = f"""
    Based on this onboarding document and recent decisions, generate 3-5 
    self-test questions that would verify a new agent has inherited the 
    correct context.
    
    Focus on:
    1. Understanding of key decisions (why, not just what)
    2. Knowledge of critical patterns or conventions
    3. Awareness of current issues or constraints
    
    Onboarding:
    {onboarding[:2000]}
    
    Recent Decisions:
    {json.dumps(decisions[-5:], indent=2)}
    
    Format as a list of questions.
    """
    
    response = await llm.generate(prompt)
    return parse_questions(response)
```

---

## 4.2 `.baton/generations/vX/DECISIONS_LOG.md`

### Purpose
Complete history of all decisions made during the project with full rationale.

### Architecture

```markdown
# Decision Log

> Project: Real-time Analytics Dashboard
> Started: 2026-03-10
> Last Updated: 2026-03-15

---

## Decision Index

| ID | Title | Status | Generation | Date |
|----|-------|--------|------------|------|
| DEC-001 | Database Selection | Active | v1 | 2026-03-10 |
| DEC-002 | Frontend Framework | Active | v1 | 2026-03-10 |
| DEC-003 | Authentication Strategy | Active | v1 | 2026-03-11 |
| DEC-004 | Real-time Communication | Active | v2 | 2026-03-12 |
| DEC-005 | Caching Layer | Active | v2 | 2026-03-13 |
| DEC-006 | API Versioning | Superseded | v2 | 2026-03-13 |
| DEC-007 | Query Optimization Approach | Active | v3 | 2026-03-14 |

---

## Decision Details

### DEC-001: Database Selection

**Status:** Active  
**Generation:** v1  
**Date:** 2026-03-10 10:30  
**Confidence:** 85%  
**Reversibility:** Low (significant migration effort)

#### Context
We need a database to store analytics data. Requirements:
- High write throughput (10k events/sec)
- Complex aggregation queries
- Time-series data
- Must support our existing infrastructure

#### Options Considered

| Option | Pros | Cons |
|--------|------|------|
| **PostgreSQL** | ACID, complex queries, team expertise, mature ecosystem | Not purpose-built for time-series, scaling requires effort |
| **MongoDB** | Flexible schema, horizontal scaling, good for events | No joins, aggregation pipeline limited, consistency concerns |
| **TimescaleDB** | Purpose-built for time-series, PostgreSQL compatible | Newer, smaller community, additional complexity |
| **InfluxDB** | Purpose-built for metrics, excellent time-series | Query language different, less flexible for joins |

#### Decision
**Chosen: PostgreSQL**

#### Rationale
1. Our analytics require complex joins and aggregations that SQL handles well
2. Team has deep PostgreSQL expertise, reducing risk
3. TimescaleDB can be added later if needed (compatible extension)
4. ACID compliance is important for our data integrity requirements

#### Impact Analysis
**Short-term:**
- Can use familiar tooling and patterns
- Straightforward schema design

**Long-term:**
- May need to add TimescaleDB extension for time-series optimization
- Horizontal scaling will require more planning than MongoDB

#### Dependencies
- Affects: All data access patterns
- Related: DEC-005 (caching layer design)
- Related: DEC-007 (query optimization)

#### Reversal Trigger
Would reconsider if:
- Write throughput exceeds PostgreSQL's practical limits (>50k/sec)
- Team gains expertise in alternative with compelling benefits

---

### DEC-002: Frontend Framework

[Similar structure...]

---

### DEC-006: API Versioning

**Status:** Superseded by DEC-008  
**Generation:** v2  
**Date:** 2026-03-13 14:00

#### Original Decision
Chose URL path versioning (`/api/v1/`, `/api/v2/`)

#### Supersession
DEC-008 (v3) moved to header-based versioning to support multiple versions 
simultaneously without URL proliferation.

#### Lessons Learned
- URL versioning creates client migration friction
- Multiple active versions harder to manage than anticipated
- Header-based approach more flexible for our use case

---

[Continue for all decisions...]
```

### Implementation Considerations

**Decision Point: Decision Granularity**

What constitutes a "decision" worth logging?

```
Granularity Levels:

Level 1: Major Architectural
- Database choice, framework selection, deployment strategy
- Always log

Level 2: Implementation Approach
- Design patterns, library choices within framework
- Usually log

Level 3: Code Structure
- File organization, naming conventions
- Sometimes log (if establishes pattern)

Level 4: Micro-Decisions
- Variable naming, specific implementation details
- Never log individually, may be captured in patterns

Recommended: Log Level 1 always, Level 2 if team discussion involved,
Level 3 if establishes convention
```

**Decision Point: Decision Extraction Method**

How do decisions get into the log?

```
Method A: Manual Entry
- User explicitly marks decisions
- Most accurate
- But: Requires user behavior, easy to forget

Method B: Transcript Analysis
- LLM parses transcript for decision patterns
- Automated, comprehensive
- But: May miss implicit decisions, hallucinate

Method C: Hybrid
- Capture candidates automatically
- Onboarder validates and enriches
- Best balance

Method D: Post-Hoc Extraction
- Extract during handoff from git history, commits
- Never misses, retrospective
- But: May lose context of "why"

Recommended: Method C - Hybrid capture with validation
```

**Implementation of Decision Extraction:**

```python
async def extract_decisions(transcript: str, pending_decisions: list) -> list[Decision]:
    """
    Extract decisions from transcript and pending captures.
    """
    prompt = f"""
    Analyze this conversation transcript and identify all decision points.
    
    For each decision, extract:
    1. The decision made
    2. The context/situation that prompted it
    3. Alternatives considered
    4. The rationale given
    5. Confidence level (if stated)
    
    Also analyze these pending file changes for implicit decisions:
    {json.dumps(pending_decisions, indent=2)}
    
    Format as JSON array of decision objects.
    """
    
    response = await llm.generate(prompt)
    decisions = json.loads(response)
    
    # Validate and enrich
    for decision in decisions:
        if not decision.get('id'):
            decision['id'] = generate_decision_id()
        if not decision.get('timestamp'):
            decision['timestamp'] = datetime.now().isoformat()
        decision['status'] = 'Active'
        
    return decisions
```

---

## 4.3 `.baton/generations/vX/TASKS_NEXT.json`

### Purpose
Structured list of pending work with priorities, dependencies, and context requirements.

### Architecture

```json
{
  "$schema": "https://claude-baton.dev/schemas/tasks-next.json",
  "version": "1.0.0",
  "generation": "v3",
  "generated_at": "2026-03-15T14:32:00Z",
  
  "tasks": [
    {
      "id": "TASK-012",
      "title": "Implement dashboard widgets",
      "description": "Create the core dashboard widget components with D3.js visualizations",
      "status": "in_progress",
      "priority": "high",
      "estimated_effort": "4 hours",
      "actual_effort": "1.5 hours",
      
      "assignments": {
        "files": [
          "src/components/Dashboard/WidgetContainer.tsx",
          "src/components/Dashboard/LineChartWidget.tsx",
          "src/components/Dashboard/BarChartWidget.tsx",
          "src/components/Dashboard/PieChartWidget.tsx"
        ],
        "entry_points": ["src/components/Dashboard/index.tsx"]
      },
      
      "context_requirements": {
        "decisions": ["DEC-002"],
        "patterns": ["react-d3-integration"],
        "files_to_read": [
          "src/charts/BaseChart.tsx",
          "src/hooks/useChartData.ts"
        ]
      },
      
      "dependencies": {
        "blocked_by": [],
        "blocks": ["TASK-014", "TASK-015"]
      },
      
      "acceptance_criteria": [
        "Widgets render data from API",
        "Responsive design for mobile/desktop",
        "Accessible chart descriptions",
        "Unit tests for each widget type"
      ],
      
      "notes": "Started in v3. LineChartWidget is 80% complete. See MEMOIRS/v3/technical.md#widgets for implementation notes."
    },
    
    {
      "id": "TASK-013",
      "title": "Add user preference persistence",
      "description": "Store user dashboard preferences (layout, filters) in database",
      "status": "pending",
      "priority": "high",
      "estimated_effort": "2 hours",
      
      "assignments": {
        "files": [
          "src/services/PreferencesService.ts",
          "src/db/preferences.ts",
          "src/components/Dashboard/PreferencesPanel.tsx"
        ]
      },
      
      "context_requirements": {
        "decisions": ["DEC-001"],
        "patterns": ["service-repository-pattern"],
        "database_schema": "preferences table (to be created)"
      },
      
      "dependencies": {
        "blocked_by": [],
        "blocks": []
      },
      
      "acceptance_criteria": [
        "Preferences persist across sessions",
        "Default preferences for new users",
        "API endpoints for CRUD operations",
        "Integration tests"
      ]
    }
  ],
  
  "completed_in_generation": [
    {
      "id": "TASK-008",
      "title": "WebSocket server implementation",
      "completed_at": "2026-03-14T16:30:00Z",
      "actual_effort": "3 hours"
    },
    {
      "id": "TASK-009",
      "title": "Redis cache integration",
      "completed_at": "2026-03-14T18:45:00Z",
      "actual_effort": "2.5 hours"
    }
  ],
  
  "deferred": [
    {
      "id": "TASK-010",
      "title": "Add dark mode support",
      "reason": "Lower priority, no user requests yet",
      "deferred_at": "2026-03-13T10:00:00Z"
    }
  ]
}
```

### Implementation Considerations

**Decision Point: Task Identification**

How do tasks get identified and tracked?

```
Source A: Explicit TODOs
- Parse TODO/FIXME comments from code
- Accurate for code-level tasks
- But: Misses higher-level tasks

Source B: User Statements
- Parse "we need to", "next step", "still need to" from transcript
- Captures intent
- But: May capture half-formed ideas

Source C: Incomplete Work
- Compare planned (previous TASKS_NEXT) to actual
- Identify what wasn't finished
- But: May miss new tasks

Source D: Git Status
- Uncommitted changes indicate in-progress
- Branches may indicate features
- But: Depends on user's git workflow

Recommended: Combine A + B + C, use D for validation
```

**Decision Point: Effort Estimation**

Should tasks include effort estimates?

```
Approach A: No Estimation
- Simpler, less overhead
- But: Harder to prioritize

Approach B: Human Estimates
- Most accurate (when done well)
- But: Requires human time, may be wrong

Approach C: LLM Estimates
- Automated, consistent
- But: May be wildly inaccurate

Approach D: Historical Estimation
- Learn from past tasks
- Improves over time
- But: Needs training data

Approach E: Range Estimates
- Optimistic/pessimistic/likely
- Better than point estimates
- Communicates uncertainty

Recommended: Approach E (ranges) + Approach D (learning from history)
```

---

## 4.4 `.baton/generations/vX/MEMOIRS/`

### Purpose
Detailed narrative accounts of the generation's work, organized by type.

### File Structure

```
MEMOIRS/
├── narrative.md      # Chronological story
├── technical.md      # Technical deep-dives
├── patterns.md       # Discovered patterns
├── errors.md         # Error history
└── decisions.md      # Decision snapshots
```

### narrative.md

```markdown
# Generation v3 Narrative

## Session Overview
- **Start:** 2026-03-14 09:00
- **End:** 2026-03-15 14:32
- **Duration:** 6.5 hours (active)
- **Context Peak:** 82%

---

## Chronological Account

### Hour 1-2: WebSocket Implementation

The session began with implementing the WebSocket server for real-time 
dashboard updates. We had already decided on WebSockets over SSE in DEC-004 
(Generation v2), so this was pure implementation.

**Key Actions:**
1. Created `src/ws/server.ts` using the `ws` library
2. Implemented connection handling with JWT authentication
3. Added heartbeat mechanism to detect dropped connections

**Challenge Encountered:**
Initial implementation didn't handle reconnection gracefully. Users would 
lose their subscription state when reconnecting. 

**Resolution:**
Added a subscription registry in Redis that persists subscription state 
for 5 minutes, allowing reconnection without data loss.

**Code Pattern Established:**
```typescript
// Pattern: Subscription Registry
const registry = new SubscriptionRegistry(redis);
registry.subscribe(userId, channel, { ttl: 300 });
```

### Hour 3-4: Dashboard Widget Development

With real-time infrastructure in place, we moved to dashboard widgets. 
React + D3.js integration (DEC-002) required careful state management.

**Approach:**
- Created `BaseChart` component with common D3 lifecycle
- Used `useRef` for D3 element mounting
- Implemented responsive resizing with `ResizeObserver`

**Challenge Encountered:**
D3 transitions conflicted with React re-renders, causing visual glitches.

**Resolution:**
Implemented `React.memo` with custom comparison to prevent unnecessary 
re-renders during D3 transitions. See `src/charts/BaseChart.tsx` for pattern.

### Hour 5-6: Performance Optimization

Dashboard widgets rendered correctly but had 300ms+ render time with 
large datasets.

**Investigation:**
1. Profiled with React DevTools
2. Identified unnecessary re-renders in widget container
3. Found expensive D3 projections being recalculated

**Optimizations:**
1. Memoized data transformations
2. Implemented virtual scrolling for large lists
3. Added data sampling for charts with >1000 points

**Result:**
Render time reduced from 300ms to 45ms.

### Hour 6.5: Handoff Preparation

Context reached 75%, triggering advisory preparation. We documented:

1. Partially completed widget implementations
2. Performance findings for future reference
3. Questions for next generation (self-test questions)

---

## Key Moments

### 1. WebSocket Breakthrough
The subscription registry pattern solved a fundamental UX issue. This 
pattern should be documented for future real-time features.

### 2. React-D3 Integration Pattern
The memoization approach we developed should be the standard for all 
future chart components.

### 3. Performance Discovery
The finding that D3 projections were the bottleneck was unexpected. 
Future performance work should profile early.

---

## Unfinished Business

1. **PieChartWidget** - Only 50% complete, needs hover interactions
2. **Widget tests** - Unit tests written for LineChart only
3. **Performance monitoring** - Consider adding performance marks
```

### Implementation Considerations

**Decision Point: Memoir Level of Detail**

```
Option A: Comprehensive
- Every action logged
- Complete context
- But: Verbose, may exceed context budget

Option B: Summary Only
- High-level overview
- Key moments highlighted
- But: Loses detail

Option C: Adaptive
- Start comprehensive
- Compress older sections
- Best balance

Option D: Task-Based
- Organize by task completion
- Only finished work has memoir
- But: Loses exploration context

Recommended: Option C - Adaptive detail with compression
```

**Decision Point: Multiple Memoir Files vs. Single File**

```
Multiple Files (current approach):
- Separation of concerns
- Easy to find specific info
- But: More files to manage

Single File:
- Simpler structure
- Single source of truth
- But: Harder to navigate

Hybrid:
- Single narrative file
- Technical details linked
- Best for most cases

Recommended: Hybrid with narrative.md + topic-specific files
```

---

# 5. MCP Server Files

## 5.1 `mcp/baton-rag/server.py`

### Purpose
The BatonRAG MCP server provides semantic search and memory management across all generations.

### Architecture

```python
#!/usr/bin/env python3
"""
BatonRAG MCP Server

Provides vector-based semantic search over all Baton artifacts:
- Decisions with full rationale
- Memoirs from all generations
- Extracted skills and patterns
- Code context snapshots

Usage:
    python -m baton_rag.server --transport stdio
    python -m baton_rag.server --transport http --port 3000
"""

import asyncio
import json
import os
from datetime import datetime
from pathlib import Path
from typing import Any, Dict, List, Optional

from mcp.server.fastmcp import FastMCP, Context
from mcp.server.session import ServerSession

# Vector database
import chromadb
from chromadb.config import Settings
from chromadb.utils import embedding_functions

# ============================================================================
# CONFIGURATION
# ============================================================================

BATON_DB_PATH = os.environ.get(
    "BATON_DB_PATH", 
    str(Path.home() / ".claude" / "baton-rag-db")
)
EMBEDDING_MODEL = os.environ.get(
    "BATON_EMBEDDING_MODEL",
    "all-MiniLM-L6-v2"
)

# ============================================================================
# INITIALIZATION
# ============================================================================

# Create MCP server
mcp = FastMCP(
    "BatonRAG",
    description="Semantic search over Claude_Baton generational memory"
)

# Initialize embedding function
embedding_func = embedding_functions.SentenceTransformerEmbeddingFunction(
    model_name=EMBEDDING_MODEL
)

# Initialize ChromaDB client
client = chromadb.PersistentClient(
    path=BATON_DB_PATH,
    settings=Settings(anonymized_telemetry=False)
)

# Create/get collections
collections = {}

def get_collection(name: str) -> chromadb.Collection:
    """Get or create a collection."""
    if name not in collections:
        collections[name] = client.get_or_create_collection(
            name=name,
            embedding_function=embedding_func,
            metadata={"hnsw:space": "cosine"}
        )
    return collections[name]

# Initialize collections on startup
for collection_name in ["memoirs", "decisions", "skills", "code_context"]:
    get_collection(collection_name)

# ============================================================================
# TOOLS
# ============================================================================

@mcp.tool()
async def search_past(
    query: str,
    limit: int = 5,
    generations: Optional[List[str]] = None,
    time_range: Optional[Dict[str, str]] = None,
    artifact_type: Optional[str] = None,
    ctx: Context = None
) -> str:
    """
    Semantic search across all generations with temporal filtering.
    
    Use this tool when you need to find relevant context from previous
    generations. Searches decisions, memoirs, skills, and code context.
    
    Args:
        query: The search query (natural language or keywords)
        limit: Maximum number of results to return (default: 5)
        generations: Filter by generation IDs (e.g., ["v1", "v2"])
        time_range: Time filter with 'from' and 'to' ISO dates
        artifact_type: Filter by type: "decision", "memoir", "skill", "code"
    
    Returns:
        JSON array of search results with content and metadata
    
    Example:
        search_past("database selection", limit=3)
        search_past("auth", generations=["v1", "v2"])
        search_past("error", time_range={"from": "2026-03-01", "to": "2026-03-15"})
    """
    # Build where clause
    where = {}
    
    if generations:
        where["generation"] = {"$in": generations}
    
    if artifact_type:
        collection_map = {
            "decision": "decisions",
            "memoir": "memoirs", 
            "skill": "skills",
            "code": "code_context"
        }
        target_collections = [collection_map.get(artifact_type, artifact_type)]
    else:
        target_collections = ["memoirs", "decisions", "skills", "code_context"]
    
    # Search across collections
    all_results = []
    
    for coll_name in target_collections:
        collection = get_collection(coll_name)
        
        try:
            results = collection.query(
                query_texts=[query],
                n_results=limit,
                where=where if where else None,
                include=["documents", "metadatas", "distances"]
            )
            
            for i, doc in enumerate(results["documents"][0]):
                all_results.append({
                    "id": results["ids"][0][i],
                    "collection": coll_name,
                    "content": doc,
                    "metadata": results["metadatas"][0][i],
                    "distance": results["distances"][0][i]
                })
        except Exception as e:
            await ctx.error(f"Error searching {coll_name}: {str(e)}")
    
    # Sort by relevance (distance) and return top results
    all_results.sort(key=lambda x: x["distance"])
    
    return json.dumps(all_results[:limit], indent=2)


@mcp.tool()
async def recall_decision(
    decision_id: Optional[str] = None,
    keywords: Optional[List[str]] = None,
    include_lineage: bool = True,
    ctx: Context = None
) -> str:
    """
    Fetch a decision with full context and impact analysis.
    
    Use this tool when you need to understand a specific decision or
    find decisions related to a topic.
    
    Args:
        decision_id: Specific decision ID (e.g., "DEC-001")
        keywords: Keywords to search for relevant decisions
        include_lineage: Include related decisions in response (default: true)
    
    Returns:
        Decision details with rationale, alternatives, and lineage
    
    Example:
        recall_decision(decision_id="DEC-001")
        recall_decision(keywords=["database", "storage"])
    """
    collection = get_collection("decisions")
    
    if decision_id:
        # Direct lookup by ID
        try:
            results = collection.get(
                ids=[decision_id],
                include=["documents", "metadatas"]
            )
            
            if not results["documents"]:
                return json.dumps({
                    "error": f"Decision {decision_id} not found",
                    "suggestion": "Try searching with keywords instead"
                })
            
            decision = {
                "id": decision_id,
                "content": results["documents"][0],
                "metadata": results["metadatas"][0]
            }
        except Exception as e:
            return json.dumps({"error": str(e)})
    else:
        # Keyword search
        query = " ".join(keywords or [])
        
        try:
            results = collection.query(
                query_texts=[query],
                n_results=3,
                include=["documents", "metadatas", "distances"]
            )
            
            if not results["documents"][0]:
                return json.dumps({
                    "error": "No matching decisions found",
                    "keywords": keywords
                })
            
            # Return the best match
            decision = {
                "id": results["ids"][0][0],
                "content": results["documents"][0][0],
                "metadata": results["metadatas"][0][0],
                "relevance": results["distances"][0][0]
            }
        except Exception as e:
            return json.dumps({"error": str(e)})
    
    # Add lineage if requested
    if include_lineage:
        decision["lineage"] = await _trace_decision_lineage(decision["id"])
    
    return json.dumps(decision, indent=2)


@mcp.tool()
async def trace_context(
    concept: str,
    show_timeline: bool = True,
    ctx: Context = None
) -> str:
    """
    Follow the evolution of a concept across generations.
    
    Use this tool to understand how understanding of a topic has
    evolved over time.
    
    Args:
        concept: The concept or topic to trace
        show_timeline: Show chronological evolution (default: true)
    
    Returns:
        Timeline of how the concept was understood in each generation
    
    Example:
        trace_context("authentication flow")
        trace_context("database schema")
    """
    timeline = []
    
    # Get all generations (sorted)
    memoirs = get_collection("memoirs")
    
    # Get unique generations
    all_memoirs = memoirs.get(include=["metadatas"])
    generations = sorted(
        set(m.get("generation") for m in all_memoirs["metadatas"] if m.get("generation")),
        key=lambda x: int(x.replace("v", ""))
    )
    
    for gen in generations:
        try:
            results = memoirs.query(
                query_texts=[concept],
                where={"generation": gen},
                n_results=1,
                include=["documents", "metadatas", "distances"]
            )
            
            if results["documents"][0]:
                timeline.append({
                    "generation": gen,
                    "understanding": results["documents"][0][0][:500],
                    "timestamp": results["metadatas"][0][0].get("timestamp"),
                    "relevance": results["distances"][0][0]
                })
        except Exception:
            continue
    
    response = {
        "concept": concept,
        "generations_searched": len(generations),
        "timeline": timeline if show_timeline else None,
        "evolution_summary": _summarize_evolution(timeline) if show_timeline else None
    }
    
    return json.dumps(response, indent=2)


@mcp.tool()
async def suggest_context(
    current_files: List[str],
    current_task: str,
    ctx: Context = None
) -> str:
    """
    Proactive context suggestion based on current work.
    
    Use this tool when starting a new task to get relevant context
    from previous generations.
    
    Args:
        current_files: List of files currently being worked on
        current_task: Description of the current task
    
    Returns:
        Relevant decisions, similar past work, and potential issues
    
    Example:
        suggest_context(
            current_files=["src/auth/login.ts", "src/auth/middleware.ts"],
            current_task="Implementing JWT refresh token rotation"
        )
    """
    # Build combined query
    query = f"{current_task} {' '.join(current_files)}"
    
    # Find relevant decisions
    decisions_coll = get_collection("decisions")
    decisions = decisions_coll.query(
        query_texts=[query],
        n_results=3,
        include=["documents", "metadatas", "distances"]
    )
    
    # Find similar past tasks
    memoirs_coll = get_collection("memoirs")
    similar_tasks = memoirs_coll.query(
        query_texts=[query],
        where={"type": "task_completion"},
        n_results=3,
        include=["documents", "metadatas"]
    )
    
    # Find related code context
    code_coll = get_collection("code_context")
    code_context = code_coll.query(
        query_texts=[query],
        n_results=5,
        include=["documents", "metadatas"]
    )
    
    # Check for potential issues
    warnings = await _check_for_repeated_mistakes(
        current_files, 
        current_task,
        memoirs_coll
    )
    
    suggestions = {
        "relevant_decisions": [
            {
                "id": decisions["ids"][0][i],
                "summary": decisions["documents"][0][i][:200],
                "relevance": 1 - decisions["distances"][0][i]
            }
            for i in range(len(decisions["ids"][0]))
        ] if decisions["ids"][0] else [],
        
        "similar_past_work": [
            {
                "generation": similar_tasks["metadatas"][0][i].get("generation"),
                "content": similar_tasks["documents"][0][i][:300]
            }
            for i in range(len(similar_tasks["documents"][0]))
        ] if similar_tasks["documents"][0] else [],
        
        "related_code": [
            {
                "file": code_context["metadatas"][0][i].get("file"),
                "snippet": code_context["documents"][0][i][:200]
            }
            for i in range(len(code_context["documents"][0]))
        ] if code_context["documents"][0] else [],
        
        "warnings": warnings
    }
    
    return json.dumps(suggestions, indent=2)


@mcp.tool()
async def store_memory(
    content: str,
    memory_type: str,
    generation: str,
    metadata: Optional[Dict[str, Any]] = None,
    ctx: Context = None
) -> str:
    """
    Store a memory in the vector database.
    
    Use this tool to persist information for future generations.
    
    Args:
        content: The content to store
        memory_type: Type of memory: "memoir", "decision", "skill", "code"
        generation: Generation ID (e.g., "v1")
        metadata: Additional metadata (optional)
    
    Returns:
        Confirmation with the stored memory ID
    
    Example:
        store_memory(
            content="Implemented rate limiting using token bucket algorithm",
            memory_type="memoir",
            generation="v3",
            metadata={"type": "task_completion", "files": ["src/middleware/rateLimit.ts"]}
        )
    """
    # Map memory type to collection
    collection_map = {
        "memoir": "memoirs",
        "decision": "decisions",
        "skill": "skills",
        "code": "code_context"
    }
    
    collection_name = collection_map.get(memory_type)
    if not collection_name:
        return json.dumps({
            "error": f"Invalid memory_type: {memory_type}",
            "valid_types": list(collection_map.keys())
        })
    
    collection = get_collection(collection_name)
    
    # Generate ID
    memory_id = f"{memory_type}_{generation}_{datetime.now().strftime('%Y%m%d%H%M%S')}"
    
    # Prepare metadata
    full_metadata = {
        "generation": generation,
        "type": memory_type,
        "timestamp": datetime.now().isoformat(),
        **(metadata or {})
    }
    
    # Store
    try:
        collection.add(
            ids=[memory_id],
            documents=[content],
            metadatas=[full_metadata]
        )
        
        return json.dumps({
            "status": "stored",
            "id": memory_id,
            "collection": collection_name,
            "metadata": full_metadata
        })
    except Exception as e:
        return json.dumps({"error": str(e)})


# ============================================================================
# RESOURCES
# ============================================================================

@mcp.resource("baton://generations")
async def list_generations() -> str:
    """List all generations with their status."""
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
                "has_memoirs": (gen_dir / "MEMOIRS").is_dir(),
                "artifacts_count": len(list(gen_dir.glob("**/*")))
            })
    
    return json.dumps({"generations": generations}, indent=2)


@mcp.resource("baton://generation/{gen_id}")
async def get_generation(gen_id: str) -> str:
    """Get details about a specific generation."""
    gen_path = Path(f".baton/generations/{gen_id}")
    
    if not gen_path.exists():
        return json.dumps({"error": f"Generation {gen_id} not found"})
    
    # Read ONBOARDING.md if exists
    onboarding = None
    onboarding_path = gen_path / "ONBOARDING.md"
    if onboarding_path.exists():
        onboarding = onboarding_path.read_text()
    
    # List artifacts
    artifacts = []
    for f in gen_path.glob("**/*"):
        if f.is_file():
            artifacts.append({
                "path": str(f.relative_to(gen_path)),
                "size": f.stat().st_size
            })
    
    return json.dumps({
        "id": gen_id,
        "onboarding_preview": onboarding[:500] if onboarding else None,
        "artifacts": artifacts
    }, indent=2)


# ============================================================================
# HELPER FUNCTIONS
# ============================================================================

async def _trace_decision_lineage(decision_id: str) -> List[Dict]:
    """Trace related decisions."""
    collection = get_collection("decisions")
    
    # Get the decision's dependencies
    try:
        results = collection.get(
            ids=[decision_id],
            include=["metadatas"]
        )
        
        if not results["metadatas"]:
            return []
        
        dependencies = results["metadatas"][0].get("dependencies", [])
        lineage = []
        
        for dep_id in dependencies:
            dep_results = collection.get(
                ids=[dep_id],
                include=["documents", "metadatas"]
            )
            if dep_results["documents"]:
                lineage.append({
                    "id": dep_id,
                    "summary": dep_results["documents"][0][:200],
                    "relationship": "depends_on"
                })
        
        return lineage
    except Exception:
        return []


def _summarize_evolution(timeline: List[Dict]) -> str:
    """Summarize how a concept evolved."""
    if len(timeline) < 2:
        return "Insufficient data to summarize evolution."
    
    first = timeline[0]
    last = timeline[-1]
    
    return (
        f"Concept first mentioned in {first['generation']}, "
        f"last updated in {last['generation']}. "
        f"Spanned {len(timeline)} generations."
    )


async def _check_for_repeated_mistakes(
    files: List[str],
    task: str,
    memoirs_coll: chromadb.Collection
) -> List[str]:
    """Check for patterns that led to issues before."""
    warnings = []
    
    try:
        # Query for error patterns
        error_patterns = memoirs_coll.query(
            query_texts=[f"error mistake issue problem {task}"],
            where={"type": "error_pattern"},
            n_results=5
        )
        
        for doc in error_patterns["documents"][0]:
            for file in files:
                if file in doc:
                    warnings.append(f"Similar work on {file} had issues: {doc[:100]}...")
    except Exception:
        pass
    
    return warnings


# ============================================================================
# MAIN
# ============================================================================

if __name__ == "__main__":
    import argparse
    
    parser = argparse.ArgumentParser(description="BatonRAG MCP Server")
    parser.add_argument(
        "--transport",
        choices=["stdio", "http"],
        default="stdio",
        help="Transport protocol"
    )
    parser.add_argument(
        "--port",
        type=int,
        default=3000,
        help="Port for HTTP transport"
    )
    
    args = parser.parse_args()
    
    if args.transport == "stdio":
        mcp.run(transport="stdio")
    else:
        mcp.run(transport="streamable-http", port=args.port)
```

### Implementation Considerations

**Decision Point: Vector Database Selection**

| Database | Pros | Cons | Best For |
|----------|------|------|----------|
| **ChromaDB** | Simple, embedded, no server | Limited scale | Local, single-user |
| **Pinecone** | Managed, scalable | Cost, cloud-only | Production, team use |
| **Qdrant** | Self-hosted, fast | Setup complexity | Production, on-prem |
| **pgvector** | SQL integration | Needs Postgres | Existing Postgres users |
| **Weaviate** | GraphQL, hybrid | Resource heavy | Complex queries |

**Recommended: ChromaDB for development, Qdrant for production.**

**Decision Point: Embedding Model Selection**

| Model | Dimensions | Speed | Quality | Cost |
|-------|------------|-------|---------|------|
| **all-MiniLM-L6-v2** | 384 | Fast | Good | Free |
| **all-mpnet-base-v2** | 768 | Medium | Better | Free |
| **OpenAI ada-002** | 1536 | Fast | Best | Paid |
| **Cohere embed-v3** | 1024 | Fast | Best | Paid |

**Recommended: all-MiniLM-L6-v2 for development, OpenAI ada-002 for production.**

**Decision Point: Collection Strategy**

How to organize memories in collections?

```
Option A: Single Collection
- All memories in one collection
- Filter by metadata
- Simple, but no collection-specific tuning

Option B: Type-Based Collections (current approach)
- Separate collection for each memory type
- Type-specific metadata schemas
- Clear organization

Option C: Generation-Based Collections
- Separate collection per generation
- Easy to archive/delete old generations
- But: Cross-generation search harder

Option D: Hybrid
- Type-based collections with generation metadata
- Best of both worlds
- Current approach

Recommended: Option D - Type-based with generation metadata
```

---

## 5.2 `mcp/baton-rag/a2a_hub.py`

### Purpose
Central hub for Agent-to-Agent communication during advisory overlap.

### Architecture

```python
#!/usr/bin/env python3
"""
A2A Communication Hub

Enables communication between agents during advisory overlap.
Handles message routing, delivery guarantees, and channel management.
"""

import asyncio
import json
import uuid
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
from pathlib import Path
from typing import Any, Callable, Dict, List, Optional

# ============================================================================
# TYPES
# ============================================================================

class MessageType(str, Enum):
    # Context Transfer
    CONTEXT_PACKAGE = "CONTEXT_PACKAGE"
    CONTEXT_REQUEST = "CONTEXT_REQUEST"
    CONTEXT_UPDATE = "CONTEXT_UPDATE"
    
    # Query/Response
    QUESTION = "QUESTION"
    ANSWER = "ANSWER"
    CLARIFICATION_REQUEST = "CLARIFICATION_REQUEST"
    CLARIFICATION_RESPONSE = "CLARIFICATION_RESPONSE"
    
    # Notification
    PROGRESS_UPDATE = "PROGRESS_UPDATE"
    THRESHOLD_ALERT = "THRESHOLD_ALERT"
    ERROR_NOTIFICATION = "ERROR_NOTIFICATION"
    WARNING = "WARNING"
    
    # Handoff
    HANDOFF_INIT = "HANDOFF_INIT"
    HANDOFF_READY = "HANDOFF_READY"
    HANDOFF_COMPLETE = "HANDOFF_COMPLETE"
    HANDOFF_ABORT = "HANDOFF_ABORT"
    
    # Quality
    QUALITY_CHECK_REQUEST = "QUALITY_CHECK_REQUEST"
    QUALITY_CHECK_RESULT = "QUALITY_CHECK_RESULT"
    QUALITY_ISSUE = "QUALITY_ISSUE"
    
    # Lifecycle
    AGENT_SPAWN = "AGENT_SPAWN"
    AGENT_TERMINATE = "AGENT_TERMINATE"
    ROLE_CHANGE = "ROLE_CHANGE"
    STATUS_REQUEST = "STATUS_REQUEST"
    STATUS_RESPONSE = "STATUS_RESPONSE"


class DeliveryGuarantee(str, Enum):
    AT_MOST_ONCE = "at_most_once"      # Fire and forget
    AT_LEAST_ONCE = "at_least_once"    # May duplicate
    EXACTLY_ONCE = "exactly_once"      # Guaranteed single delivery


@dataclass
class AgentEndpoint:
    agent_id: str
    generation_id: str
    role: str  # primary, advisor, onboarder, archivist


@dataclass
class A2AMessage:
    id: str
    protocol_version: str = "1.0"
    timestamp: str = field(default_factory=lambda: datetime.now().isoformat())
    from_endpoint: AgentEndpoint = None
    to_endpoint: AgentEndpoint = None
    message_type: MessageType = MessageType.QUESTION
    payload: Dict[str, Any] = field(default_factory=dict)
    delivery_guarantee: DeliveryGuarantee = DeliveryGuarantee.AT_LEAST_ONCE
    timeout_ms: int = 30000
    retry_count: int = 3
    priority: str = "normal"  # low, normal, high, critical
    metadata: Dict[str, Any] = field(default_factory=dict)
    
    def to_json(self) -> str:
        return json.dumps({
            "id": self.id,
            "protocol_version": self.protocol_version,
            "timestamp": self.timestamp,
            "from": {
                "agent_id": self.from_endpoint.agent_id,
                "generation_id": self.from_endpoint.generation_id,
                "role": self.from_endpoint.role
            } if self.from_endpoint else None,
            "to": {
                "agent_id": self.to_endpoint.agent_id,
                "generation_id": self.to_endpoint.generation_id,
                "role": self.to_endpoint.role
            } if self.to_endpoint else None,
            "type": self.message_type.value,
            "payload": self.payload,
            "delivery": {
                "guarantee": self.delivery_guarantee.value,
                "timeout_ms": self.timeout_ms,
                "retry_count": self.retry_count,
                "priority": self.priority
            },
            "metadata": self.metadata
        }, indent=2)


@dataclass
class DeliveryStatus:
    message_id: str
    status: str  # pending, delivered, failed, expired
    attempts: int = 0
    last_attempt: datetime = None
    error: str = None
    delivered_at: datetime = None


# ============================================================================
# CHANNEL DEFINITIONS
# ============================================================================

CHANNEL_DEFINITIONS = {
    "advisor-primary-channel": {
        "type": "direct",
        "participants": ["advisor", "primary"],
        "message_types": [
            MessageType.QUESTION,
            MessageType.ANSWER,
            MessageType.CLARIFICATION_REQUEST,
            MessageType.CLARIFICATION_RESPONSE,
        ],
        "config": {
            "max_message_size": 4000,
            "message_cooldown_ms": 5000,
            "advisor_can_initiate": False
        }
    },
    "lifecycle-events-channel": {
        "type": "broadcast",
        "participants": ["primary", "advisor", "archivist"],
        "message_types": [
            MessageType.AGENT_SPAWN,
            MessageType.AGENT_TERMINATE,
            MessageType.ROLE_CHANGE,
            MessageType.THRESHOLD_ALERT,
        ],
        "config": {
            "persistent": True,
            "retention_hours": 24
        }
    },
    "handoff-coordination-channel": {
        "type": "priority_queue",
        "participants": ["primary", "advisor"],
        "message_types": [
            MessageType.HANDOFF_INIT,
            MessageType.HANDOFF_READY,
            MessageType.HANDOFF_COMPLETE,
            MessageType.HANDOFF_ABORT,
        ],
        "config": {
            "priority_levels": {
                "critical": [MessageType.HANDOFF_ABORT],
                "high": [MessageType.HANDOFF_INIT, MessageType.HANDOFF_COMPLETE],
                "normal": [MessageType.HANDOFF_READY]
            }
        }
    }
}


# ============================================================================
# A2A HUB
# ============================================================================

class A2ACommunicationHub:
    """
    Central hub for Agent-to-Agent communication.
    
    Handles:
    - Message routing between agents
    - Delivery guarantees
    - Channel management
    - Message persistence
    """
    
    def __init__(self, storage_path: str = ".baton/a2a_messages"):
        self.storage_path = Path(storage_path)
        self.storage_path.mkdir(parents=True, exist_ok=True)
        
        # Channel subscriptions
        self.channels: Dict[str, List[AgentEndpoint]] = {}
        
        # Message queue
        self.message_queue: asyncio.Queue = asyncio.Queue()
        
        # Delivery tracking
        self.delivery_tracker: Dict[str, DeliveryStatus] = {}
        
        # Callbacks
        self.message_handlers: Dict[MessageType, Callable] = {}
        
        # Running state
        self.running = False
        self.processing_task = None
        
    async def start(self):
        """Start the communication hub."""
        self.running = True
        self.processing_task = asyncio.create_task(self._process_messages())
        
    async def stop(self):
        """Stop the communication hub."""
        self.running = False
        if self.processing_task:
            self.processing_task.cancel()
            try:
                await self.processing_task
            except asyncio.CancelledError:
                pass
                
    def register_agent(self, agent: AgentEndpoint, channels: List[str]):
        """
        Register an agent to specific channels.
        
        Args:
            agent: The agent endpoint to register
            channels: List of channel names to subscribe to
        """
        for channel in channels:
            if channel not in self.channels:
                self.channels[channel] = []
            
            # Check if already registered
            existing = [a for a in self.channels[channel] 
                       if a.agent_id == agent.agent_id]
            if not existing:
                self.channels[channel].append(agent)
                
    def unregister_agent(self, agent: AgentEndpoint):
        """Unregister an agent from all channels."""
        for channel in self.channels.values():
            self.channels[channel] = [
                a for a in channel if a.agent_id != agent.agent_id
            ]
            
    async def send_message(
        self,
        message: A2AMessage,
        channel: str
    ) -> bool:
        """
        Send a message through a channel.
        
        Args:
            message: The message to send
            channel: The channel to send through
            
        Returns:
            True if message was queued successfully
        """
        # Validate sender is registered
        if channel not in self.channels:
            raise ValueError(f"Channel not found: {channel}")
            
        if message.from_endpoint not in self.channels[channel]:
            raise ValueError(f"Sender not registered on channel: {channel}")
        
        # Check advisor initiation rules
        channel_def = CHANNEL_DEFINITIONS.get(channel, {})
        config = channel_def.get("config", {})
        
        if (message.from_endpoint.role == "advisor" and 
            not config.get("advisor_can_initiate", False)):
            raise ValueError("Advisor cannot initiate messages on this channel")
        
        # Track delivery
        self.delivery_tracker[message.id] = DeliveryStatus(
            message_id=message.id,
            status="pending"
        )
        
        # Queue message
        await self.message_queue.put((message, channel))
        
        return True
        
    async def send_and_wait(
        self,
        message: A2AMessage,
        channel: str,
        timeout_ms: int = None
    ) -> Optional[A2AMessage]:
        """
        Send a message and wait for response.
        
        Args:
            message: The message to send
            channel: The channel to send through
            timeout_ms: Timeout in milliseconds
            
        Returns:
            Response message or None if timeout
        """
        timeout = (timeout_ms or message.timeout_ms) / 1000
        correlation_id = message.id
        
        # Create response future
        response_future = asyncio.Future()
        self._pending_responses[correlation_id] = response_future
        
        try:
            await self.send_message(message, channel)
            
            # Wait for response
            response = await asyncio.wait_for(response_future, timeout=timeout)
            return response
        except asyncio.TimeoutError:
            return None
        finally:
            self._pending_responses.pop(correlation_id, None)
            
    def on_message(self, message_type: MessageType, handler: Callable):
        """Register a handler for a specific message type."""
        self.message_handlers[message_type] = handler
        
    async def _process_messages(self):
        """Background task to process message queue."""
        while self.running:
            try:
                message, channel = await asyncio.wait_for(
                    self.message_queue.get(),
                    timeout=1.0
                )
                asyncio.create_task(self._deliver_message(message, channel))
            except asyncio.TimeoutError:
                continue
            except Exception as e:
                print(f"Error processing messages: {e}")
                
    async def _deliver_message(self, message: A2AMessage, channel: str):
        """Deliver message to recipients."""
        recipients = self.channels.get(channel, [])
        status = self.delivery_tracker[message.id]
        
        for recipient in recipients:
            # Skip sender
            if recipient.agent_id == message.from_endpoint.agent_id:
                continue
                
            # Check direct message
            if message.to_endpoint and recipient.agent_id != message.to_endpoint.agent_id:
                continue
            
            try:
                # Deliver
                await self._deliver_to_agent(message, recipient)
                
                # Check if response
                if message.metadata.get("correlation_id"):
                    correlation_id = message.metadata["correlation_id"]
                    if correlation_id in self._pending_responses:
                        self._pending_responses[correlation_id].set_result(message)
                
                status.status = "delivered"
                status.delivered_at = datetime.now()
                
            except Exception as e:
                status.attempts += 1
                status.last_attempt = datetime.now()
                status.error = str(e)
                
                # Retry logic
                if status.attempts < message.retry_count:
                    await asyncio.sleep(2 ** status.attempts)  # Exponential backoff
                    await self.message_queue.put((message, channel))
                else:
                    status.status = "failed"
                    
    async def _deliver_to_agent(self, message: A2AMessage, recipient: AgentEndpoint):
        """
        Deliver message to a specific agent.
        
        Delivery methods:
        1. Write to agent's inbox directory
        2. Call registered callback if exists
        3. Notify via MCP
        """
        # Method 1: File-based delivery
        inbox_path = (
            self.storage_path / 
            recipient.generation_id / 
            "inbox" / 
            f"{message.id}.json"
        )
        inbox_path.parent.mkdir(parents=True, exist_ok=True)
        inbox_path.write_text(message.to_json())
        
        # Method 2: Callback if registered
        handler = self.message_handlers.get(message.message_type)
        if handler:
            await handler(message, recipient)
            
    _pending_responses: Dict[str, asyncio.Future] = {}
```

### Implementation Considerations

**Decision Point: Delivery Mechanism**

How should messages be delivered between agents?

| Method | Pros | Cons |
|--------|------|------|
| **File-based** | Simple, persistent, debuggable | Polling required, latency |
| **MCP messaging** | Native, real-time | Depends on MCP support |
| **Shared memory** | Fast | Complex, not persistent |
| **Redis pub/sub** | Real-time, scalable | External dependency |

**Recommended: File-based for reliability, MCP for real-time when available.**

**Decision Point: Message Ordering**

Should messages be ordered per channel?

```
Option A: FIFO per channel
- Predictable order
- Simple implementation
- But: May block on slow message

Option B: Priority-based
- Critical messages first
- More responsive
- But: May starve low priority

Option C: Per-sender FIFO
- Order preserved per sender
- No blocking across senders
- But: Cross-sender order undefined

Option D: Timestamp-based
- Global order by timestamp
- Fair across senders
- But: Clock sync issues

Recommended: Option B (Priority) + Option C (Per-sender FIFO within priority)
```

---

# 6. Core System Files

## 6.1 `.baton/state.json`

### Purpose
Global state file for Baton system, tracking current generation, handoff status, and system health.

### Architecture

```json
{
  "$schema": "https://claude-baton.dev/schemas/state.json",
  "version": "1.0.0",
  
  "current_generation": {
    "id": "v3",
    "started_at": "2026-03-14T09:00:00Z",
    "status": "active"
  },
  
  "handoff": {
    "status": "none",
    "last_handoff": {
      "from": "v2",
      "to": "v3",
      "completed_at": "2026-03-14T09:00:00Z",
      "duration_ms": 45000
    },
    "next_scheduled": null
  },
  
  "statistics": {
    "total_generations": 3,
    "total_decisions": 12,
    "total_tasks_completed": 8,
    "total_runtime_hours": 18.5,
    "average_generation_hours": 6.2
  },
  
  "rag": {
    "last_indexed": "2026-03-15T14:32:00Z",
    "total_entries": 156,
    "index_size_mb": 2.3
  },
  
  "health": {
    "last_check": "2026-03-15T14:32:00Z",
    "status": "healthy",
    "issues": []
  }
}
```

### Implementation Considerations

**Decision Point: State Persistence Frequency**

| Frequency | Pros | Cons |
|-----------|------|------|
| **Every change** | Always accurate | File I/O overhead |
| **Every N seconds** | Balanced | May lose recent changes |
| **On shutdown** | Minimal I/O | Lose data on crash |
| **Hybrid: critical immediate + periodic full** | Best balance | Complex |

**Recommended: Hybrid - Critical state changes written immediately, statistics updated periodically.**

---

# Summary

This document covered every file in the Claude_Baton system with:

1. **Purpose**: Why the file exists and what problem it solves
2. **Architecture**: Detailed structure and implementation
3. **Implementation Considerations**: Multiple approaches with trade-offs

Key implementation decisions:

| Component | Decision | Rationale |
|-----------|----------|-----------|
| Hook Type | Command for logic, Prompt for validation | Best tool for each job |
| State Storage | Simple files for status, JSON for complex state | Balance simplicity and structure |
| Agent Model | Haiku for background tasks | Cost-effective for repeated operations |
| Vector DB | ChromaDB (dev) / Qdrant (prod) | Right tool for scale |
| A2A Delivery | File-based with MCP notification | Reliable + real-time |
| Artifact Format | Layered (summary + detail + links) | Scanability + depth |

Each file plays a specific role in the generational handoff system, and the implementation options provided allow teams to make informed trade-offs based on their specific requirements.
