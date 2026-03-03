# Claude_Baton Comprehensive Schema Reference

**Version:** 2.0.0  
**Protocol Version:** Baton Protocol v2.0  
**JSON Schema Compliance:** 2020-12 (SEP-1613 mandated)  
**Last Updated:** March 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Plugin Core Schemas](#plugin-core-schemas)
3. [Baton Protocol Artifact Schemas](#baton-protocol-artifact-schemas)
4. [MCP Tool Schemas](#mcp-tool-schemas)
5. [Agent & A2A Communication Schemas](#agent--a2a-communication-schemas)
6. [Configuration Schemas](#configuration-schemas)
7. [Error Handling Schemas](#error-handling-schemas)
8. [Schema Registry & Versioning](#schema-registry--versioning)

---

## Overview

This document provides the definitive schema reference for all Claude_Baton components. Every schema is JSON Schema 2020-12 compliant and validated against the official specification.

### Schema Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CLAUDE_BATON SCHEMA ECOSYSTEM                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐   │
│  │ Plugin Core     │   │ Baton Protocol  │   │ MCP Layer       │   │
│  │                 │   │ Artifacts       │   │                 │   │
│  │ • plugin.json   │   │ • ONBOARDING.md │   │ • Tool Schemas  │   │
│  │ • hooks.json    │   │ • DECISIONS_LOG │   │ • .mcp.json     │   │
│  │ • SKILL.md      │   │ • TASKS_NEXT    │   │ • Resources     │   │
│  └─────────────────┘   │ • KNOWLEDGE_... │   └─────────────────┘   │
│           │            └─────────────────┘            │             │
│           │                      │                     │             │
│           └──────────────────────┼─────────────────────┘             │
│                                  │                                   │
│                    ┌─────────────▼─────────────┐                    │
│                    │ Configuration Layer       │                    │
│                    │ • .baton/config.json      │                    │
│                    │ • .baton/state.json       │                    │
│                    │ • A2A Protocol            │                    │
│                    └───────────────────────────┘                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| Plugin ID | `^[a-z][a-z0-9-]*[a-z0-9]$` | `claude-baton` |
| Skill Name | `^[a-z][a-z0-9-]*[a-z0-9]$` | `jwt-refresh-rotation` |
| Generation ID | `^v\d+$` | `v5` |
| Decision ID | `^DEC-\d+$` | `DEC-007` |
| Task ID | `^TASK-\d+$` | `TASK-003` |
| Hook Name | `^(pre|post|on):[a-z]+(:[a-z]+)?$` | `pre:generation` |

---

## Plugin Core Schemas

### 1. plugin.json Schema

**Purpose:** Defines the plugin manifest for Claude Code marketplace integration.

**Location:** `.claude-plugin/plugin.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/plugin-manifest-v1.json",
  "title": "Claude_Baton Plugin Manifest",
  "description": "Defines a Claude_Baton plugin with skills, hooks, MCP servers, and artifacts",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^(@[a-z0-9-]+/)?[a-z0-9-]+$",
      "minLength": 3,
      "maxLength": 64,
      "description": "Unique plugin identifier (scoped or unscoped)"
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100,
      "description": "Human-readable plugin name"
    },
    "version": {
      "$ref": "#/$defs/semver",
      "description": "Plugin version (semantic versioning)"
    },
    "description": {
      "type": "string",
      "minLength": 10,
      "maxLength": 500,
      "description": "Brief description for marketplace and tool selection"
    },
    "author": {
      "type": "string",
      "description": "Author name or organization"
    },
    "license": {
      "type": "string",
      "pattern": "^[A-Z0-9.-]+$",
      "description": "SPDX license identifier",
      "examples": ["MIT", "Apache-2.0", "GPL-3.0"]
    },
    "repository": {
      "type": "string",
      "format": "uri",
      "description": "Source repository URL"
    },
    "homepage": {
      "type": "string",
      "format": "uri",
      "description": "Plugin homepage or documentation URL"
    },
    "keywords": {
      "type": "array",
      "items": {
        "type": "string",
        "minLength": 1,
        "maxLength": 50
      },
      "minItems": 1,
      "maxItems": 20,
      "description": "Searchable keywords for marketplace"
    },
    "engines": {
      "type": "object",
      "properties": {
        "claude-code": {
          "type": "string",
          "description": "Required Claude Code version (semver range)"
        },
        "claude-baton": {
          "type": "string",
          "description": "Required Baton version (semver range)"
        },
        "node": {
          "type": "string",
          "description": "Required Node.js version (semver range)"
        }
      },
      "required": ["claude-code"]
    },
    "marketplace": {
      "type": "object",
      "properties": {
        "category": {
          "type": "string",
          "enum": [
            "Memory & Context",
            "Developer Tools",
            "Code Quality",
            "Testing",
            "Security",
            "Integrations",
            "Workflows"
          ],
          "description": "Marketplace category"
        },
        "tags": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Marketplace tags"
        },
        "icon": {
          "type": "string",
          "description": "Emoji or icon URL"
        },
        "featured": {
          "type": "boolean",
          "default": false,
          "description": "Featured in marketplace"
        }
      },
      "required": ["category"]
    },
    "skills": {
      "type": "array",
      "items": { "$ref": "#/$defs/skillEntry" },
      "description": "Skills provided by this plugin"
    },
    "hooks": {
      "type": "object",
      "additionalProperties": { "$ref": "#/$defs/hookEntry" },
      "description": "Lifecycle hooks registered by this plugin"
    },
    "mcpServers": {
      "type": "object",
      "additionalProperties": { "$ref": "#/$defs/mcpServerEntry" },
      "description": "MCP servers provided by this plugin"
    },
    "artifacts": {
      "type": "array",
      "items": { "$ref": "#/$defs/artifactDef" },
      "description": "Artifact types this plugin handles"
    },
    "permissions": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": [
          "filesystem",
          "filesystem-read",
          "filesystem-write",
          "subagent-spawn",
          "git",
          "git-commit",
          "network",
          "network-mcp",
          "secrets"
        ]
      },
      "description": "Permissions required by this plugin"
    },
    "config": {
      "type": "object",
      "description": "Plugin-specific configuration schema"
    }
  },
  "required": [
    "id",
    "name",
    "version",
    "description",
    "license",
    "engines"
  ],
  "additionalProperties": false,
  "$defs": {
    "semver": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+(-[a-zA-Z0-9.-]+)?(\\+[a-zA-Z0-9.-]+)?$",
      "examples": ["1.0.0", "2.1.0-beta.1", "3.0.0+build.123"]
    },
    "skillEntry": {
      "type": "object",
      "properties": {
        "path": {
          "type": "string",
          "description": "Path to SKILL.md file"
        },
        "enabled": {
          "type": "boolean",
          "default": true
        },
        "priority": {
          "type": "integer",
          "minimum": 0,
          "maximum": 100,
          "default": 50
        }
      },
      "required": ["path"]
    },
    "hookEntry": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": ["skill", "command", "script", "http"],
          "description": "Handler type"
        },
        "skill": {
          "type": "string",
          "description": "Skill name if type is 'skill'"
        },
        "command": {
          "type": "string",
          "description": "Command to execute if type is 'command'"
        },
        "script": {
          "type": "string",
          "description": "Script path if type is 'script'"
        },
        "url": {
          "type": "string",
          "format": "uri",
          "description": "HTTP endpoint if type is 'http'"
        },
        "condition": {
          "type": "string",
          "description": "Conditional expression for hook activation"
        },
        "priority": {
          "type": "integer",
          "minimum": 0,
          "maximum": 1000,
          "default": 50
        },
        "enabled": {
          "type": "boolean",
          "default": true
        },
        "async": {
          "type": "boolean",
          "default": false,
          "description": "Whether hook runs asynchronously"
        },
        "timeout": {
          "type": "integer",
          "minimum": 100,
          "maximum": 30000,
          "default": 5000,
          "description": "Timeout in milliseconds"
        }
      },
      "required": ["type"]
    },
    "mcpServerEntry": {
      "oneOf": [
        {
          "type": "object",
          "properties": {
            "transport": { "const": "stdio" },
            "command": { "type": "string" },
            "args": {
              "type": "array",
              "items": { "type": "string" }
            },
            "env": {
              "type": "object",
              "additionalProperties": { "type": "string" }
            },
            "cwd": { "type": "string" },
            "restart": {
              "type": "object",
              "properties": {
                "enabled": { "type": "boolean", "default": true },
                "maxRetries": { "type": "integer", "default": 3 },
                "delayMs": { "type": "integer", "default": 1000 }
              }
            }
          },
          "required": ["transport", "command"]
        },
        {
          "type": "object",
          "properties": {
            "transport": { "const": "sse" },
            "url": { "type": "string", "format": "uri" },
            "headers": {
              "type": "object",
              "additionalProperties": { "type": "string" }
            },
            "timeout": { "type": "integer", "default": 30000 }
          },
          "required": ["transport", "url"]
        },
        {
          "type": "object",
          "properties": {
            "transport": { "const": "websocket" },
            "url": { "type": "string", "format": "uri" },
            "headers": {
              "type": "object",
              "additionalProperties": { "type": "string" }
            }
          },
          "required": ["transport", "url"]
        }
      ]
    },
    "artifactDef": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "description": "Artifact type identifier"
        },
        "schema": {
          "type": "object",
          "description": "JSON Schema for artifact validation"
        },
        "handler": {
          "type": "string",
          "description": "Handler skill for processing this artifact"
        },
        "extension": {
          "type": "string",
          "description": "Default file extension"
        }
      },
      "required": ["type"]
    }
  }
}
```

**Example:**

```json
{
  "id": "claude-baton",
  "name": "Claude_Baton",
  "version": "1.0.0",
  "description": "Infinite context via automatic generational baton passing",
  "author": "Casey DiGennaro",
  "license": "MIT",
  "repository": "https://github.com/SuperInstance/Claude_Baton",
  "keywords": ["infinite-context", "generational", "memory", "agent-teams"],
  "engines": {
    "claude-code": ">=2.1.0"
  },
  "marketplace": {
    "category": "Memory & Context",
    "icon": "🏃‍♂️"
  },
  "skills": [
    { "path": "skills/BatonManager/SKILL.md", "priority": 100 },
    { "path": "skills/Onboarder/SKILL.md", "priority": 90 },
    { "path": "skills/Archivist/SKILL.md", "priority": 80 }
  ],
  "hooks": {
    "PreCompact": {
      "type": "skill",
      "skill": "Onboarder",
      "condition": "contextPercent >= 48",
      "priority": 10
    },
    "SessionStart": {
      "type": "skill",
      "skill": "BatonManager",
      "condition": "reason in ['compact', 'resume']"
    },
    "SubagentStop": {
      "type": "skill",
      "skill": "Archivist"
    }
  },
  "mcpServers": {
    "baton-rag": {
      "transport": "stdio",
      "command": "python",
      "args": ["mcp/baton-rag/server.py"]
    }
  },
  "permissions": ["filesystem", "subagent-spawn", "git-commit", "network-mcp"]
}
```

---

### 2. hooks.json Schema

**Purpose:** Defines lifecycle hook configurations.

**Location:** `hooks/hooks.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/hooks-v1.json",
  "title": "Hook Configuration Schema",
  "description": "Configuration for Claude Code lifecycle hooks",
  "type": "object",
  "properties": {
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+$",
      "default": "1.3",
      "description": "Hooks configuration version"
    },
    "hooks": {
      "type": "object",
      "propertyNames": {
        "enum": [
          "UserPromptSubmit",
          "PreToolUse",
          "PostToolUse",
          "PostToolUseFailure",
          "Notification",
          "Stop",
          "SubagentStart",
          "SubagentStop",
          "PreCompact",
          "SessionStart",
          "SessionEnd",
          "PermissionRequest",
          "TeammateIdle",
          "TaskCompleted"
        ]
      },
      "additionalProperties": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "name": {
              "type": "string",
              "description": "Hook instance name"
            },
            "type": {
              "type": "string",
              "enum": ["skill", "command", "script", "http"],
              "description": "Handler type"
            },
            "skill": {
              "type": "string",
              "description": "Skill to invoke"
            },
            "command": {
              "type": "string",
              "description": "Shell command to execute"
            },
            "script": {
              "type": "string",
              "description": "Path to script file"
            },
            "url": {
              "type": "string",
              "format": "uri",
              "description": "HTTP endpoint"
            },
            "condition": {
              "type": "string",
              "description": "Conditional expression (e.g., 'contextPercent >= 48')"
            },
            "priority": {
              "type": "integer",
              "minimum": 0,
              "maximum": 1000,
              "default": 50
            },
            "enabled": {
              "type": "boolean",
              "default": true
            },
            "async": {
              "type": "boolean",
              "default": false
            },
            "timeout": {
              "type": "integer",
              "minimum": 100,
              "maximum": 60000,
              "default": 5000
            },
            "options": {
              "type": "object",
              "properties": {
                "contextIsolation": {
                  "type": "boolean",
                  "default": true
                },
                "costMode": {
                  "type": "string",
                  "enum": ["normal", "background"],
                  "default": "normal"
                },
                "promptCaching": {
                  "type": "boolean",
                  "default": false
                },
                "debounce": {
                  "type": "integer",
                  "description": "Debounce interval in milliseconds"
                }
              }
            }
          },
          "required": ["name", "type"]
        }
      }
    }
  },
  "required": ["version", "hooks"],
  "additionalProperties": false
}
```

**Example:**

```json
{
  "version": "1.3",
  "hooks": {
    "PreCompact": [
      {
        "name": "baton-onboard",
        "type": "skill",
        "skill": "Onboarder",
        "condition": "contextPercent >= 48",
        "priority": 10,
        "options": {
          "contextIsolation": true,
          "costMode": "background",
          "promptCaching": true
        }
      }
    ],
    "SessionStart": [
      {
        "name": "baton-resume",
        "type": "skill",
        "skill": "BatonManager",
        "condition": "reason in ['compact', 'resume']",
        "priority": 5
      }
    ],
    "SubagentStart": [
      {
        "name": "a2a-handshake",
        "type": "script",
        "script": "hooks/a2a-handshake.sh",
        "async": true
      }
    ],
    "SubagentStop": [
      {
        "name": "baton-archive",
        "type": "skill",
        "skill": "Archivist",
        "priority": 10
      }
    ],
    "PostToolUse": [
      {
        "name": "decision-capture",
        "type": "skill",
        "skill": "DecisionLogger",
        "condition": "tool in ['Write', 'Edit', 'MultiEdit']",
        "priority": 1,
        "options": {
          "debounce": 30000
        }
      }
    ],
    "TeammateIdle": [
      {
        "name": "a2a-quality-gate",
        "type": "skill",
        "skill": "BatonManager",
        "condition": "generationState == 'advisory'"
      }
    ]
  }
}
```

---

### 3. SKILL.md YAML Frontmatter Schema

**Purpose:** Defines skill metadata and configuration.

**Location:** `skills/{skill-name}/SKILL.md` (frontmatter section)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/skill-frontmatter-v1.json",
  "title": "SKILL.md Frontmatter Schema",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9-]*[a-z0-9]$",
      "minLength": 3,
      "maxLength": 64,
      "description": "Unique skill identifier (kebab-case)"
    },
    "version": {
      "$ref": "#/$defs/semver"
    },
    "description": {
      "type": "string",
      "minLength": 10,
      "maxLength": 500,
      "description": "Brief description for AI tool selection"
    },
    "author": {
      "type": "string"
    },
    "license": {
      "type": "string",
      "pattern": "^[A-Z0-9.-]+$"
    },
    "triggers": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": [
          "PreCompact",
          "SessionStart",
          "SubagentStart",
          "SubagentStop",
          "PostToolUse",
          "PreToolUse",
          "TeammateIdle",
          "UserPromptSubmit"
        ]
      },
      "default": []
    },
    "contextIsolation": {
      "type": "boolean",
      "default": true,
      "description": "Whether skill runs in isolated context"
    },
    "costMode": {
      "type": "string",
      "enum": ["normal", "background"],
      "default": "normal"
    },
    "userInvocable": {
      "type": "boolean",
      "default": true,
      "description": "Whether user can invoke via slash command"
    },
    "disableModelInvocation": {
      "type": "boolean",
      "default": false
    },
    "allowedTools": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": ["filesystem", "git", "network", "mcp", "subagent"]
      },
      "default": ["filesystem"]
    },
    "context": {
      "type": "string",
      "enum": ["fork", "shared", "isolated"],
      "default": "isolated",
      "description": "Context sharing mode"
    },
    "agent": {
      "type": "string",
      "default": "general-purpose",
      "description": "Agent type to use"
    },
    "priority": {
      "type": "integer",
      "minimum": 0,
      "maximum": 100,
      "default": 50
    },
    "dependencies": {
      "type": "object",
      "properties": {
        "packages": {
          "type": "array",
          "items": { "type": "string" }
        },
        "skills": {
          "type": "array",
          "items": { "type": "string" }
        },
        "environment": {
          "type": "object",
          "additionalProperties": { "type": "string" }
        }
      }
    },
    "config": {
      "type": "object",
      "properties": {
        "defaultModel": { "type": "string" },
        "maxTokens": { "type": "integer" },
        "timeout": { "type": "integer" }
      }
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": ["name", "version", "description"],
  "additionalProperties": false,
  "$defs": {
    "semver": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+(-[a-zA-Z0-9.-]+)?$"
    }
  }
}
```

**Example:**

```yaml
---
name: Onboarder
version: "1.0.0"
description: Background parallel onboarding at context threshold
author: Casey DiGennaro
license: MIT
triggers: [PreCompact]
contextIsolation: true
costMode: background
userInvocable: false
allowedTools: [filesystem, git]
context: fork
agent: general-purpose
dependencies:
  packages: [chromadb]
  skills: []
config:
  timeout: 60000
tags: [context, handoff, background]
---

# Onboarder Skill

You are the Onboarder subagent. Context is at {{contextPercent}}%.
Build the complete baton bundle in .baton/generations/v{{nextGen}}/
while the main agent continues working.

## Tasks
1. Generate ONBOARDING.md with executive summary
2. Create MEMOIRS/narrative.md with timeline
3. Extract decisions to DECISIONS_LOG.md
4. Identify reusable patterns for SKILLS_EXTRACTED/
5. Create TASKS_NEXT.json with continuation plan

Do NOT interfere with main agent. Output ONLY to .baton/generations/v{{nextGen}}/
```

---

## Baton Protocol Artifact Schemas

### 4. ONBOARDING.md Schema

**Purpose:** Primary entry document for generation handoffs.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/onboarding-frontmatter-v1.json",
  "title": "ONBOARDING.md Frontmatter",
  "type": "object",
  "properties": {
    "schema_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    },
    "generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "previous_generation": {
      "oneOf": [
        { "type": "string", "pattern": "^v\\d+$" },
        { "type": "null" }
      ]
    },
    "created_at": {
      "type": "string",
      "format": "date-time"
    },
    "total_runtime": {
      "type": "string",
      "pattern": "^\\d+h \\d+m$"
    },
    "handoff_reason": {
      "type": "string",
      "enum": ["context_threshold", "manual", "error_recovery"]
    },
    "context_percent": {
      "type": "number",
      "minimum": 0,
      "maximum": 100
    },
    "baton_protocol_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+$"
    }
  },
  "required": [
    "schema_version",
    "generation",
    "previous_generation",
    "created_at",
    "total_runtime",
    "handoff_reason",
    "context_percent"
  ],
  "additionalProperties": false
}
```

---

### 5. DECISIONS_LOG.md Entry Schema

**Purpose:** Preserve decision rationale with full context.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/decision-entry-v1.json",
  "title": "Decision Entry Schema",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^DEC-\\d+$"
    },
    "title": {
      "type": "string",
      "minLength": 1
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "status": {
      "type": "string",
      "enum": ["Active", "Superseded", "Reversed"],
      "default": "Active"
    },
    "confidence": {
      "type": "integer",
      "minimum": 0,
      "maximum": 100
    },
    "reversibility": {
      "type": "string",
      "enum": ["Low", "Medium", "High"]
    },
    "context": {
      "type": "string"
    },
    "options": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "pros": {
            "type": "array",
            "items": { "type": "string" }
          },
          "cons": {
            "type": "array",
            "items": { "type": "string" }
          }
        },
        "required": ["name", "pros", "cons"]
      },
      "minItems": 1
    },
    "decision": { "type": "string" },
    "rationale": { "type": "string" },
    "impact": {
      "type": "object",
      "properties": {
        "short_term": { "type": "string" },
        "long_term": { "type": "string" }
      },
      "required": ["short_term", "long_term"]
    },
    "dependencies": {
      "type": "object",
      "properties": {
        "depends_on": {
          "type": "array",
          "items": { "type": "string", "pattern": "^DEC-\\d+$" }
        },
        "influences": {
          "type": "array",
          "items": { "type": "string", "pattern": "^DEC-\\d+$" }
        }
      }
    },
    "reversal_trigger": { "type": "string" }
  },
  "required": [
    "id",
    "title",
    "timestamp",
    "status",
    "confidence",
    "reversibility",
    "context",
    "options",
    "decision",
    "rationale",
    "impact"
  ],
  "additionalProperties": false
}
```

---

### 6. TASKS_NEXT.json Schema

**Purpose:** Structured task queue with dependencies.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/tasks-next-v1.json",
  "title": "Task Queue Schema",
  "type": "object",
  "properties": {
    "schema_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    },
    "generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "created_at": {
      "type": "string",
      "format": "date-time"
    },
    "project_name": { "type": "string" },
    "project_phase": { "type": "string" },
    "summary": { "type": "string" },
    "tasks": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string",
            "pattern": "^TASK-\\d+$"
          },
          "title": {
            "type": "string",
            "minLength": 1
          },
          "description": { "type": "string" },
          "status": {
            "type": "string",
            "enum": ["pending", "in_progress", "blocked", "completed", "cancelled"],
            "default": "pending"
          },
          "priority": {
            "type": "string",
            "enum": ["critical", "high", "medium", "low"],
            "default": "medium"
          },
          "estimated_effort": { "type": "string" },
          "actual_effort": { "type": "string" },
          "dependencies": {
            "type": "array",
            "items": { "type": "string", "pattern": "^TASK-\\d+$" }
          },
          "related_decisions": {
            "type": "array",
            "items": { "type": "string", "pattern": "^DEC-\\d+$" }
          },
          "related_files": {
            "type": "array",
            "items": { "type": "string" }
          },
          "assignee": { "type": "string" },
          "due_date": {
            "type": "string",
            "format": "date"
          },
          "completed_at": {
            "type": "string",
            "format": "date-time"
          },
          "notes": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "timestamp": {
                  "type": "string",
                  "format": "date-time"
                },
                "note": { "type": "string" }
              },
              "required": ["timestamp", "note"]
            }
          }
        },
        "required": ["id", "title", "description", "status", "priority"]
      }
    },
    "blocking_issues": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "description": { "type": "string" },
          "blocking_tasks": {
            "type": "array",
            "items": { "type": "string" }
          },
          "resolution_needed": { "type": "string" }
        },
        "required": ["id", "description"]
      }
    },
    "diagram": {
      "type": "object",
      "properties": {
        "format": {
          "type": "string",
          "enum": ["mermaid", "dot", "json"],
          "default": "mermaid"
        },
        "content": { "type": "string" }
      }
    },
    "metrics": {
      "type": "object",
      "properties": {
        "total_tasks": { "type": "integer" },
        "completed_tasks": { "type": "integer" },
        "in_progress_tasks": { "type": "integer" },
        "blocked_tasks": { "type": "integer" },
        "completion_percentage": {
          "type": "number",
          "minimum": 0,
          "maximum": 100
        }
      }
    }
  },
  "required": ["schema_version", "generation", "created_at", "tasks"],
  "additionalProperties": false
}
```

---

### 7. KNOWLEDGE_GRAPH.json Schema

**Purpose:** Entity-relationship graph for semantic queries.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/knowledge-graph-v1.json",
  "title": "Knowledge Graph Schema",
  "type": "object",
  "properties": {
    "schema_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    },
    "generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "created_at": {
      "type": "string",
      "format": "date-time"
    },
    "entities": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string",
            "enum": [
              "module", "component", "service", "infrastructure",
              "api", "database", "decision", "concept", "file", "person"
            ]
          },
          "name": { "type": "string" },
          "status": {
            "type": "string",
            "enum": ["planned", "in_progress", "active", "deprecated", "removed"]
          },
          "technology": { "type": "string" },
          "description": { "type": "string" },
          "depends_on": {
            "type": "array",
            "items": { "type": "string" }
          },
          "decisions": {
            "type": "array",
            "items": { "type": "string", "pattern": "^DEC-\\d+$" }
          },
          "files": {
            "type": "array",
            "items": { "type": "string" }
          },
          "metadata": {
            "type": "object",
            "additionalProperties": true
          },
          "first_seen_generation": {
            "type": "string",
            "pattern": "^v\\d+$"
          },
          "last_modified_generation": {
            "type": "string",
            "pattern": "^v\\d+$"
          }
        },
        "required": ["type", "name"]
      }
    },
    "relationships": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "from": { "type": "string" },
          "to": { "type": "string" },
          "type": {
            "type": "string",
            "enum": [
              "uses", "implements", "extends", "depends_on",
              "produces", "consumes", "contains", "references"
            ]
          },
          "criticality": {
            "type": "string",
            "enum": ["low", "medium", "high", "critical"],
            "default": "medium"
          },
          "description": { "type": "string" }
        },
        "required": ["from", "to", "type"]
      }
    },
    "open_questions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string",
            "pattern": "^Q-\\d+$"
          },
          "question": { "type": "string" },
          "context": { "type": "string" },
          "related_entities": {
            "type": "array",
            "items": { "type": "string" }
          },
          "priority": {
            "type": "string",
            "enum": ["low", "medium", "high"],
            "default": "medium"
          }
        },
        "required": ["id", "question"]
      }
    }
  },
  "required": ["schema_version", "generation", "created_at", "entities"],
  "additionalProperties": false
}
```

---

## MCP Tool Schemas

### 8. MCP Server Configuration (.mcp.json)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/mcp-server-config-v1.json",
  "title": "MCP Server Configuration",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9-]*[a-z0-9]$"
    },
    "displayName": { "type": "string" },
    "description": { "type": "string" },
    "version": { "$ref": "#/$defs/semver" },
    "transport": {
      "type": "string",
      "enum": ["stdio", "sse", "websocket"],
      "default": "stdio"
    },
    "command": {
      "type": "string",
      "description": "Command for stdio transport"
    },
    "args": {
      "type": "array",
      "items": { "type": "string" }
    },
    "env": {
      "type": "object",
      "additionalProperties": { "type": "string" }
    },
    "url": {
      "type": "string",
      "format": "uri",
      "description": "URL for SSE/WebSocket transport"
    },
    "tools": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "description": { "type": "string" },
          "inputSchema": { "type": "object" },
          "outputSchema": { "type": "object" }
        },
        "required": ["name", "description", "inputSchema"]
      }
    },
    "resources": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "uri": { "type": "string", "format": "uri" },
          "name": { "type": "string" },
          "description": { "type": "string" },
          "mimeType": { "type": "string" }
        },
        "required": ["uri", "name"]
      }
    }
  },
  "required": ["name", "version", "transport"],
  "additionalProperties": false,
  "$defs": {
    "semver": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    }
  }
}
```

---

### 9. MCP Tool: search-past

```json
{
  "name": "search-past",
  "description": "Semantic search across all previous generations with temporal filtering, artifact type filtering, and pagination support",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "minLength": 1,
        "maxLength": 500,
        "description": "Search query in natural language"
      },
      "limit": {
        "type": "integer",
        "minimum": 1,
        "maximum": 50,
        "default": 5,
        "description": "Maximum number of results"
      },
      "generations": {
        "type": "array",
        "items": {
          "type": "string",
          "pattern": "^v\\d+$"
        },
        "description": "Filter by specific generations"
      },
      "time_range": {
        "type": "object",
        "properties": {
          "from": {
            "type": "string",
            "format": "date-time"
          },
          "to": {
            "type": "string",
            "format": "date-time"
          }
        },
        "description": "Temporal filter"
      },
      "artifact_type": {
        "type": "string",
        "enum": ["decision", "memoir", "skill", "task", "all"],
        "default": "all"
      },
      "include_content": {
        "type": "boolean",
        "default": true
      }
    },
    "required": ["query"],
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "success": { "type": "boolean" },
      "query": { "type": "string" },
      "total_found": { "type": "integer" },
      "returned": { "type": "integer" },
      "results": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "generation": { "type": "string" },
            "artifact_type": { "type": "string" },
            "source_file": { "type": "string" },
            "score": { "type": "number" },
            "content": { "type": "string" },
            "highlight": { "type": "string" },
            "timestamp": { "type": "string", "format": "date-time" }
          }
        }
      },
      "search_time_ms": { "type": "integer" }
    }
  }
}
```

---

### 10. MCP Tool: recall-decision

```json
{
  "name": "recall-decision",
  "description": "Fetch a specific decision with full context, impact analysis, and decision lineage",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "decision_id": {
        "type": "string",
        "pattern": "^DEC-\\d+$",
        "description": "Specific decision ID to retrieve"
      },
      "keywords": {
        "type": "array",
        "items": { "type": "string" },
        "minItems": 1,
        "description": "Keywords to search if decision_id not provided"
      },
      "include_lineage": {
        "type": "boolean",
        "default": true,
        "description": "Include related decisions (depends_on, influences)"
      },
      "include_history": {
        "type": "boolean",
        "default": false,
        "description": "Include status change history"
      }
    },
    "oneOf": [
      { "required": ["decision_id"] },
      { "required": ["keywords"] }
    ],
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "success": { "type": "boolean" },
      "decision": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "title": { "type": "string" },
          "status": { "type": "string" },
          "confidence": { "type": "integer" },
          "context": { "type": "string" },
          "decision": { "type": "string" },
          "rationale": { "type": "string" },
          "impact": { "type": "object" },
          "reversal_trigger": { "type": "string" },
          "generation": { "type": "string" },
          "timestamp": { "type": "string" }
        }
      },
      "lineage": {
        "type": "object",
        "properties": {
          "depends_on": { "type": "array" },
          "influenced": { "type": "array" }
        }
      },
      "alternatives": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "pros": { "type": "array" },
            "cons": { "type": "array" }
          }
        }
      }
    }
  }
}
```

---

### 11. MCP Tool: trace-context

```json
{
  "name": "trace-context",
  "description": "Follow the evolution of a concept or entity across all generations",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "concept": {
        "type": "string",
        "minLength": 1,
        "description": "Concept or entity to trace"
      },
      "show_timeline": {
        "type": "boolean",
        "default": true
      },
      "include_changes": {
        "type": "boolean",
        "default": true,
        "description": "Detect significant changes in understanding"
      },
      "max_generations": {
        "type": "integer",
        "minimum": 1,
        "maximum": 50,
        "default": 10
      }
    },
    "required": ["concept"],
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "concept": { "type": "string" },
      "first_seen": {
        "type": "object",
        "properties": {
          "generation": { "type": "string" },
          "timestamp": { "type": "string" }
        }
      },
      "timeline": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "generation": { "type": "string" },
            "understanding": { "type": "string" },
            "timestamp": { "type": "string" },
            "source": { "type": "string" }
          }
        }
      },
      "changes": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "generation": { "type": "string" },
            "type": {
              "type": "string",
              "enum": ["introduced", "refined", "deprecated", "reversed"]
            },
            "description": { "type": "string" }
          }
        }
      },
      "evolution_summary": { "type": "string" }
    }
  }
}
```

---

### 12. MCP Tool: suggest-context

```json
{
  "name": "suggest-context",
  "description": "Proactively suggest relevant context based on current work",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "current_files": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Files currently being worked on"
      },
      "current_task": {
        "type": "string",
        "description": "Description of current task"
      },
      "max_suggestions": {
        "type": "integer",
        "minimum": 1,
        "maximum": 20,
        "default": 5
      }
    },
    "required": ["current_task"],
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "relevant_decisions": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "id": { "type": "string" },
            "title": { "type": "string" },
            "relevance": { "type": "string" },
            "summary": { "type": "string" }
          }
        }
      },
      "similar_past_work": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "generation": { "type": "string" },
            "description": { "type": "string" },
            "outcome": { "type": "string" }
          }
        }
      },
      "suggested_files": {
        "type": "array",
        "items": { "type": "string" }
      },
      "warnings": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "type": {
              "type": "string",
              "enum": ["repeated_mistake", "conflict", "deprecated_pattern"]
            },
            "description": { "type": "string" },
            "source_generation": { "type": "string" }
          }
        }
      }
    }
  }
}
```

---

### 13. MCP Tool: get-generation-tree

```json
{
  "name": "get-generation-tree",
  "description": "Retrieve the generation hierarchy for visualization",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "format": {
        "type": "string",
        "enum": ["tree", "list", "timeline", "summary"],
        "default": "tree"
      },
      "include_stats": {
        "type": "boolean",
        "default": true
      },
      "from_generation": {
        "type": "string",
        "pattern": "^v\\d+$",
        "description": "Start from specific generation"
      },
      "depth": {
        "type": "integer",
        "minimum": 1,
        "maximum": 50,
        "default": 10
      }
    },
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "format": { "type": "string" },
      "total_generations": { "type": "integer" },
      "current_generation": { "type": "string" },
      "tree": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "generation": { "type": "string" },
            "parent": { "type": "string" },
            "duration": { "type": "string" },
            "decisions_count": { "type": "integer" },
            "skills_extracted": { "type": "integer" },
            "status": {
              "type": "string",
              "enum": ["active", "advisor", "archived"]
            },
            "children": {
              "type": "array",
              "items": { "type": "object" }
            }
          }
        }
      },
      "mermaid_diagram": {
        "type": "string",
        "description": "Mermaid diagram source"
      }
    }
  }
}
```

---

### 14. MCP Tool: validate-handoff

```json
{
  "name": "validate-handoff",
  "description": "Check handoff readiness and quality",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "generation": {
        "type": "string",
        "pattern": "^v\\d+$",
        "description": "Generation to validate"
      },
      "checks": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "artifacts_complete",
            "decisions_logged",
            "tasks_documented",
            "skills_extracted",
            "rag_indexed",
            "git_committed"
          ]
        },
        "default": ["artifacts_complete", "decisions_logged", "tasks_documented"]
      },
      "simulate_handoff": {
        "type": "boolean",
        "default": false,
        "description": "Run simulation without actual handoff"
      }
    },
    "required": ["generation"],
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "generation": { "type": "string" },
      "ready": { "type": "boolean" },
      "score": {
        "type": "integer",
        "minimum": 0,
        "maximum": 100
      },
      "checks": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "passed": { "type": "boolean" },
            "score": { "type": "integer" },
            "details": { "type": "string" }
          }
        }
      },
      "blocking_issues": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "issue": { "type": "string" },
            "severity": {
              "type": "string",
              "enum": ["critical", "warning", "info"]
            },
            "suggestion": { "type": "string" }
          }
        }
      },
      "recommendations": {
        "type": "array",
        "items": { "type": "string" }
      }
    }
  }
}
```

---

### 15. MCP Tool: index-generation

```json
{
  "name": "index-generation",
  "description": "Internal tool for indexing new generation artifacts into RAG",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "generation": {
        "type": "string",
        "pattern": "^v\\d+$"
      },
      "path": {
        "type": "string",
        "description": "Path to generation directory"
      },
      "force_reindex": {
        "type": "boolean",
        "default": false
      },
      "chunk_size": {
        "type": "integer",
        "minimum": 100,
        "maximum": 2000,
        "default": 500
      },
      "async": {
        "type": "boolean",
        "default": true
      }
    },
    "required": ["generation", "path"],
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "success": { "type": "boolean" },
      "generation": { "type": "string" },
      "indexed": {
        "type": "object",
        "properties": {
          "chunks": { "type": "integer" },
          "decisions": { "type": "integer" },
          "memoirs": { "type": "integer" },
          "skills": { "type": "integer" }
        }
      },
      "collection_size": { "type": "integer" },
      "index_time_ms": { "type": "integer" },
      "errors": {
        "type": "array",
        "items": { "type": "string" }
      }
    }
  }
}
```

---

## Agent & A2A Communication Schemas

### 16. A2A Message Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/a2a-message-v1.json",
  "title": "A2A Communication Message",
  "type": "object",
  "properties": {
    "message_id": {
      "type": "string",
      "format": "uuid"
    },
    "correlation_id": {
      "type": "string",
      "format": "uuid",
      "description": "For request/response correlation"
    },
    "type": {
      "type": "string",
      "enum": [
        "question",
        "answer",
        "validation_request",
        "validation_response",
        "handoff_initiation",
        "handoff_confirmation",
        "error",
        "heartbeat"
      ]
    },
    "from": {
      "type": "object",
      "properties": {
        "generation": { "type": "string" },
        "role": {
          "type": "string",
          "enum": ["primary", "young", "advisor", "onboarder", "archivist"]
        }
      },
      "required": ["generation", "role"]
    },
    "to": {
      "type": "object",
      "properties": {
        "generation": { "type": "string" },
        "role": {
          "type": "string",
          "enum": ["primary", "young", "advisor", "onboarder", "archivist", "broadcast"]
        }
      },
      "required": ["generation", "role"]
    },
    "payload": {
      "type": "object",
      "description": "Message-specific payload"
    },
    "channel": {
      "type": "string",
      "enum": ["artifacts", "mcp_messages", "git"],
      "default": "mcp_messages"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "ttl": {
      "type": "integer",
      "description": "Time-to-live in seconds",
      "default": 300
    },
    "response_expected": {
      "type": "boolean",
      "default": false
    },
    "timeout": {
      "type": "integer",
      "default": 30000
    }
  },
  "required": [
    "message_id",
    "type",
    "from",
    "to",
    "payload",
    "timestamp"
  ],
  "additionalProperties": false
}
```

---

### 17. Agent Role Configuration Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/agent-role-v1.json",
  "title": "Agent Role Configuration",
  "type": "object",
  "properties": {
    "generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "role": {
      "type": "string",
      "enum": ["primary", "young", "advisor", "onboarder", "archivist"]
    },
    "permissions": {
      "type": "object",
      "properties": {
        "can_initiate": { "type": "boolean", "default": true },
        "can_modify_files": { "type": "boolean", "default": true },
        "can_spawn_subagents": { "type": "boolean", "default": false },
        "can_access_rag": { "type": "boolean", "default": true },
        "can_answer_questions": { "type": "boolean", "default": true },
        "can_log_decisions": { "type": "boolean", "default": true }
      }
    },
    "role_injection_prompt": {
      "type": "string",
      "description": "System prompt addition for role transition"
    },
    "transition": {
      "type": "object",
      "properties": {
        "trigger_context_percent": {
          "type": "integer",
          "minimum": 0,
          "maximum": 100
        },
        "grace_period_seconds": {
          "type": "integer",
          "default": 60
        },
        "max_questions_answered": {
          "type": "integer",
          "default": 10
        }
      }
    },
    "status": {
      "type": "string",
      "enum": ["spawning", "active", "advisory", "retiring", "archived"]
    }
  },
  "required": ["generation", "role", "permissions"],
  "additionalProperties": false
}
```

---

### 18. Generation Lifecycle Event Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/lifecycle-event-v1.json",
  "title": "Generation Lifecycle Event",
  "type": "object",
  "properties": {
    "event_id": {
      "type": "string",
      "format": "uuid"
    },
    "event_type": {
      "type": "string",
      "enum": [
        "predictive_analysis_started",
        "onboarding_spawned",
        "onboarding_complete",
        "skill_extraction_started",
        "skill_extraction_complete",
        "advisory_preparation_started",
        "young_agent_spawned",
        "a2a_handshake_initiated",
        "a2a_handshake_complete",
        "advisor_transition",
        "quality_gate_passed",
        "quality_gate_failed",
        "archive_started",
        "archive_complete",
        "generation_retired"
      ]
    },
    "generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "context_percent": {
      "type": "number",
      "minimum": 0,
      "maximum": 100
    },
    "threshold": {
      "type": "integer",
      "enum": [40, 48, 60, 75, 82, 93, 98]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object"
    },
    "previous_state": {
      "type": "string"
    },
    "new_state": {
      "type": "string"
    },
    "duration_ms": {
      "type": "integer"
    },
    "success": {
      "type": "boolean"
    },
    "error": {
      "type": "object",
      "properties": {
        "code": { "type": "string" },
        "message": { "type": "string" },
        "recoverable": { "type": "boolean" }
      }
    }
  },
  "required": [
    "event_id",
    "event_type",
    "generation",
    "timestamp"
  ],
  "additionalProperties": false
}
```

---

### 19. Quality Gate Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/quality-gate-v1.json",
  "title": "Quality Gate Check",
  "type": "object",
  "properties": {
    "gate_id": {
      "type": "string"
    },
    "generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "phase": {
      "type": "string",
      "enum": ["pre_handoff", "a2a_overlap", "post_handoff"]
    },
    "checks": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "check_id": { "type": "string" },
          "name": { "type": "string" },
          "description": { "type": "string" },
          "category": {
            "type": "string",
            "enum": [
              "artifact_completeness",
              "decision_coverage",
              "task_continuity",
              "rag_indexing",
              "git_state"
            ]
          },
          "passed": { "type": "boolean" },
          "score": {
            "type": "number",
            "minimum": 0,
            "maximum": 100
          },
          "threshold": {
            "type": "number"
          },
          "details": { "type": "string" },
          "remediation": { "type": "string" }
        },
        "required": ["check_id", "name", "passed"]
      }
    },
    "overall_passed": {
      "type": "boolean"
    },
    "overall_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 100
    },
    "blocking_issues": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "issue": { "type": "string" },
          "severity": {
            "type": "string",
            "enum": ["critical", "high", "medium", "low"]
          },
          "blocking": { "type": "boolean" }
        }
      }
    },
    "escalation_required": {
      "type": "boolean"
    },
    "escalation_reason": {
      "type": "string"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    }
  },
  "required": [
    "gate_id",
    "generation",
    "phase",
    "checks",
    "overall_passed"
  ],
  "additionalProperties": false
}
```

---

## Configuration Schemas

### 20. .baton/config.json Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/baton-config-v1.json",
  "title": "Claude_Baton User Configuration",
  "type": "object",
  "properties": {
    "schema_version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$",
      "default": "1.0.0"
    },
    "thresholds": {
      "type": "object",
      "properties": {
        "predictive_analysis": {
          "type": "integer",
          "minimum": 30,
          "maximum": 50,
          "default": 40
        },
        "onboard": {
          "type": "integer",
          "minimum": 40,
          "maximum": 60,
          "default": 48
        },
        "skill_extraction": {
          "type": "integer",
          "minimum": 50,
          "maximum": 70,
          "default": 60
        },
        "advisory_prep": {
          "type": "integer",
          "minimum": 65,
          "maximum": 80,
          "default": 75
        },
        "spawn": {
          "type": "integer",
          "minimum": 75,
          "maximum": 90,
          "default": 82
        },
        "retire": {
          "type": "integer",
          "minimum": 88,
          "maximum": 98,
          "default": 93
        },
        "archive": {
          "type": "integer",
          "minimum": 95,
          "maximum": 99,
          "default": 98
        }
      },
      "default": {}
    },
    "mode": {
      "type": "string",
      "enum": ["aggressive", "conservative", "human_gated"],
      "default": "aggressive",
      "description": "Operational mode"
    },
    "rag_enabled": {
      "type": "boolean",
      "default": true
    },
    "auto_commit": {
      "type": "boolean",
      "default": true
    },
    "max_generations_kept": {
      "type": "integer",
      "minimum": 5,
      "maximum": 100,
      "default": 20
    },
    "artifact_settings": {
      "type": "object",
      "properties": {
        "compression_enabled": {
          "type": "boolean",
          "default": true
        },
        "delta_encoding": {
          "type": "boolean",
          "default": true
        },
        "max_memoir_size_kb": {
          "type": "integer",
          "default": 100
        },
        "max_decision_size_kb": {
          "type": "integer",
          "default": 50
        }
      }
    },
    "rag_settings": {
      "type": "object",
      "properties": {
        "embedding_model": {
          "type": "string",
          "default": "default"
        },
        "chunk_size": {
          "type": "integer",
          "default": 500
        },
        "chunk_overlap": {
          "type": "integer",
          "default": 50
        },
        "indexing_mode": {
          "type": "string",
          "enum": ["sync", "async"],
          "default": "async"
        },
        "vector_db": {
          "type": "string",
          "enum": ["chroma", "pinecone", "lancedb"],
          "default": "chroma"
        }
      }
    },
    "team_settings": {
      "type": "object",
      "properties": {
        "enabled": {
          "type": "boolean",
          "default": false
        },
        "shared_repo": {
          "type": "string"
        },
        "conflict_resolution": {
          "type": "string",
          "enum": ["local_wins", "remote_wins", "manual"],
          "default": "manual"
        },
        "sync_interval_seconds": {
          "type": "integer",
          "default": 60
        }
      }
    },
    "notifications": {
      "type": "object",
      "properties": {
        "on_handoff": {
          "type": "boolean",
          "default": true
        },
        "on_quality_gate_failure": {
          "type": "boolean",
          "default": true
        },
        "on_skill_extracted": {
          "type": "boolean",
          "default": false
        }
      }
    },
    "telemetry": {
      "type": "object",
      "properties": {
        "enabled": {
          "type": "boolean",
          "default": false
        },
        "anonymous": {
          "type": "boolean",
          "default": true
        }
      }
    }
  },
  "additionalProperties": false
}
```

---

### 21. .baton/state.json Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/baton-state-v1.json",
  "title": "Claude_Baton Runtime State",
  "type": "object",
  "properties": {
    "schema_version": {
      "type": "string",
      "default": "1.0.0"
    },
    "project_name": {
      "type": "string"
    },
    "initialized_at": {
      "type": "string",
      "format": "date-time"
    },
    "last_updated": {
      "type": "string",
      "format": "date-time"
    },
    "current_generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "generation_count": {
      "type": "integer",
      "minimum": 0
    },
    "context_percent": {
      "type": "number",
      "minimum": 0,
      "maximum": 100
    },
    "context_tokens": {
      "type": "integer"
    },
    "state": {
      "type": "string",
      "enum": [
        "idle",
        "predictive_analysis",
        "onboarding",
        "skill_extraction",
        "advisory_prep",
        "handoff_pending",
        "a2a_overlap",
        "retiring",
        "archiving",
        "error"
      ]
    },
    "active_subagents": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "skill": { "type": "string" },
          "spawned_at": {
            "type": "string",
            "format": "date-time"
          },
          "status": { "type": "string" }
        }
      }
    },
    "pending_events": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "event_type": { "type": "string" },
          "scheduled_for": {
            "type": "string",
            "format": "date-time"
          },
          "payload": { "type": "object" }
        }
      }
    },
    "metrics": {
      "type": "object",
      "properties": {
        "total_runtime_hours": { "type": "number" },
        "total_tokens_processed": { "type": "integer" },
        "total_decisions_logged": { "type": "integer" },
        "total_skills_extracted": { "type": "integer" },
        "total_handoffs": { "type": "integer" },
        "average_generation_duration_hours": { "type": "number" }
      }
    },
    "last_error": {
      "type": "object",
      "properties": {
        "timestamp": {
          "type": "string",
          "format": "date-time"
        },
        "code": { "type": "string" },
        "message": { "type": "string" },
        "generation": { "type": "string" }
      }
    }
  },
  "required": [
    "schema_version",
    "current_generation",
    "state",
    "context_percent"
  ],
  "additionalProperties": false
}
```

---

## Error Handling Schemas

### 22. Error Response Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.dev/schemas/error-response-v1.json",
  "title": "Standard Error Response",
  "type": "object",
  "properties": {
    "success": {
      "type": "boolean",
      "const": false
    },
    "error": {
      "type": "object",
      "properties": {
        "code": {
          "type": "string",
          "enum": [
            "VALIDATION_ERROR",
            "NOT_FOUND",
            "ALREADY_EXISTS",
            "STATE_ERROR",
            "PERMISSION_DENIED",
            "CONTEXT_OVERFLOW",
            "HANDOFF_FAILED",
            "RAG_ERROR",
            "GIT_ERROR",
            "INTERNAL_ERROR"
          ]
        },
        "message": {
          "type": "string"
        },
        "details": {
          "type": "object",
          "additionalProperties": true
        },
        "suggestion": {
          "type": "string",
          "description": "Suggested remediation"
        },
        "retryable": {
          "type": "boolean",
          "default": false
        },
        "retry_after_ms": {
          "type": "integer",
          "description": "Milliseconds to wait before retry"
        }
      },
      "required": ["code", "message"]
    },
    "generation": {
      "type": "string",
      "pattern": "^v\\d+$"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "request_id": {
      "type": "string",
      "format": "uuid"
    }
  },
  "required": ["success", "error", "timestamp"],
  "additionalProperties": false
}
```

---

## Schema Registry & Versioning

### Schema URI Pattern

```
https://claude-baton.dev/schemas/{type}-v{major}.json
```

### Available Schemas

| Schema ID | URI | Purpose |
|-----------|-----|---------|
| plugin-manifest-v1 | `/schemas/plugin-manifest-v1.json` | Plugin manifest validation |
| hooks-v1 | `/schemas/hooks-v1.json` | Hook configuration |
| skill-frontmatter-v1 | `/schemas/skill-frontmatter-v1.json` | SKILL.md YAML validation |
| onboarding-frontmatter-v1 | `/schemas/onboarding-frontmatter-v1.json` | ONBOARDING.md validation |
| decision-entry-v1 | `/schemas/decision-entry-v1.json` | Decision entry validation |
| tasks-next-v1 | `/schemas/tasks-next-v1.json` | Task queue validation |
| knowledge-graph-v1 | `/schemas/knowledge-graph-v1.json` | Knowledge graph validation |
| mcp-server-config-v1 | `/schemas/mcp-server-config-v1.json` | MCP server configuration |
| a2a-message-v1 | `/schemas/a2a-message-v1.json` | A2A message validation |
| agent-role-v1 | `/schemas/agent-role-v1.json` | Agent role configuration |
| lifecycle-event-v1 | `/schemas/lifecycle-event-v1.json` | Event validation |
| quality-gate-v1 | `/schemas/quality-gate-v1.json` | Quality gate validation |
| baton-config-v1 | `/schemas/baton-config-v1.json` | User configuration |
| baton-state-v1 | `/schemas/baton-state-v1.json` | Runtime state |
| error-response-v1 | `/schemas/error-response-v1.json` | Error responses |

---

## Validation Commands

```bash
# Validate plugin.json
python -m jsonschema --instance .claude-plugin/plugin.json \
  https://claude-baton.dev/schemas/plugin-manifest-v1.json

# Validate ONBOARDING.md frontmatter
python -m jsonschema --instance <(python -c "import yaml; print(json.dumps(yaml.safe_load(open('ONBOARDING.md').read().split('---')[1])))" \
  https://claude-baton.dev/schemas/onboarding-frontmatter-v1.json

# Validate TASKS_NEXT.json
python -m jsonschema --instance .baton/generations/v5/TASKS_NEXT.json \
  https://claude-baton.dev/schemas/tasks-next-v1.json

# Validate MCP tool input
python -m jsonschema --instance '{"query": "test"}' \
  https://claude-baton.dev/schemas/tools/search-past-input-v1.json
```

---

## TypeScript Type Generation

```bash
# Generate TypeScript types from schemas
npx json-schema-to-typescript \
  https://claude-baton.dev/schemas/plugin-manifest-v1.json \
  --output types/plugin-manifest.ts

npx json-schema-to-typescript \
  https://claude-baton.dev/schemas/decision-entry-v1.json \
  --output types/decision-entry.ts
```

---

**End of Schema Reference**

*This document is auto-generated from the Claude_Baton schema registry.*
