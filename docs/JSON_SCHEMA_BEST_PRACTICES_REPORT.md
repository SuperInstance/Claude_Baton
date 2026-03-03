# JSON Schema Best Practices for AI Agent Plugin Systems

**Research Report for Claude_Baton Schema Design**

**Date**: March 2025  
**Version**: 1.0  
**Focus**: Plugin manifests, skill definitions, hooks, MCP servers, and artifact schemas

---

## Executive Summary

This report synthesizes JSON Schema 2020-12 best practices specifically for AI agent plugin systems. Based on analysis of existing standards (VS Code extensions, npm packages, MCP tool schemas), we provide actionable recommendations for Claude_Baton's schema architecture.

**Key Recommendations:**
1. Use JSON Schema 2020-12 with `$schema` declaration in all schemas
2. Implement `$defs` for reusable type definitions
3. Use `additionalProperties: false` strictly for MCP tool inputs
4. Design for versioning with `$id` URIs and semantic versioning
5. Leverage `oneOf`/`anyOf` for flexible plugin configurations

---

## 1. JSON Schema 2020-12 Best Practices for Plugin Manifests

### 1.1 Core Schema Structure

Every plugin manifest should follow this canonical structure:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/claude-baton/schemas/plugin-manifest-v1.json",
  "title": "Claude_Baton Plugin Manifest",
  "description": "Defines a plugin for the Claude_Baton AI agent system",
  "type": "object",
  "properties": {
    "name": { "$ref": "#/$defs/pluginName" },
    "version": { "$ref": "#/$defs/semver" },
    "description": { "type": "string", "maxLength": 500 },
    "main": { "type": "string", "description": "Entry point file" },
    "skills": {
      "type": "array",
      "items": { "$ref": "#/$defs/skillReference" }
    },
    "hooks": {
      "type": "object",
      "additionalProperties": { "$ref": "#/$defs/hookConfig" }
    },
    "mcpServers": {
      "type": "array",
      "items": { "$ref": "#/$defs/mcpServerConfig" }
    }
  },
  "required": ["name", "version"],
  "additionalProperties": false,
  "$defs": {
    "pluginName": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9-]*[a-z0-9]$",
      "minLength": 3,
      "maxLength": 64,
      "description": "Unique plugin identifier (kebab-case)"
    },
    "semver": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+(-[a-zA-Z0-9.-]+)?$",
      "description": "Semantic version (e.g., 1.0.0, 2.1.0-beta.1)"
    }
  }
}
```

### 1.2 Critical Keywords for Plugin Schemas

| Keyword | Usage | Recommendation |
|---------|-------|----------------|
| `$schema` | Declares dialect | **Required**: Always use 2020-12 |
| `$id` | Schema identifier URI | **Required**: Use versioned URIs |
| `$defs` | Reusable definitions | **Recommended**: All shared types |
| `additionalProperties` | Extra property handling | **Strict**: `false` for MCP tools |
| `required` | Mandatory fields | Use with `oneOf` for alternatives |
| `description` | Documentation | **Essential**: Claude reads every word |
| `default` | Default values | Use for optional parameters |
| `examples` | Example values | Add 2-3 realistic examples |

### 1.3 Composition Patterns

**Use `oneOf` for mutually exclusive configurations:**

```json
{
  "oneOf": [
    {
      "properties": {
        "type": { "const": "local" },
        "path": { "type": "string" }
      },
      "required": ["type", "path"]
    },
    {
      "properties": {
        "type": { "const": "remote" },
        "url": { "format": "uri" }
      },
      "required": ["type", "url"]
    }
  ]
}
```

**Use `anyOf` for optional enhancements:**

```json
{
  "anyOf": [
    { "required": ["skillRef"] },
    { "required": ["hookRef"] },
    { "required": ["toolRef"] }
  ]
}
```

---

## 2. Well-Designed Plugin Schema Examples

### 2.1 VS Code Extension Manifest Pattern

VS Code's `package.json` extension manifest is a gold standard:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "vscode-extension-manifest-v3.json",
  "type": "object",
  "properties": {
    "name": { "type": "string", "pattern": "^[a-z0-9-]+$" },
    "displayName": { "type": "string" },
    "version": { "$ref": "#/$defs/semver" },
    "engines": {
      "type": "object",
      "properties": {
        "vscode": { "type": "string" }
      },
      "required": ["vscode"]
    },
    "categories": {
      "type": "array",
      "items": {
        "enum": ["Programming Languages", "Debuggers", "Linters", "Snippets", "Other"]
      }
    },
    "activationEvents": {
      "type": "array",
      "items": {
        "type": "string",
        "pattern": "^on(Language|Command|View|FileSystem):.+$"
      }
    },
    "contributes": {
      "type": "object",
      "properties": {
        "commands": { "type": "array", "items": { "$ref": "#/$defs/command" } },
        "menus": { "type": "object" },
        "configuration": { "$ref": "#/$defs/configuration" }
      }
    }
  },
  "required": ["name", "version", "engines"]
}
```

**Key learnings:**
- `activationEvents` pattern is similar to hook triggers
- `contributes` section maps well to plugin capabilities
- `engines` field for version constraints is essential

### 2.2 npm package.json Pattern

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "properties": {
    "name": { "type": "string", "pattern": "^(@[a-z0-9-~][a-z0-9-._~]*/)?[a-z0-9-~][a-z0-9-._~]*$" },
    "version": { "$ref": "#/$defs/semver" },
    "type": { "enum": ["module", "commonjs"] },
    "exports": {
      "oneOf": [
        { "type": "string" },
        {
          "type": "object",
          "properties": {
            ".": { "type": "string" },
            "./types": { "type": "string" }
          },
          "additionalProperties": { "type": "string" }
        }
      ]
    },
    "peerDependencies": {
      "type": "object",
      "additionalProperties": { "type": "string" }
    }
  }
}
```

**Key learnings:**
- Scoped package names (`@org/package`) pattern
- `exports` field for multiple entry points
- `peerDependencies` for plugin host requirements

### 2.3 MCP Tool Schema Pattern (From Claude_Baton)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/claude-baton/mcp-tool-schema.json",
  "title": "MCP Tool Input Schema",
  "type": "object",
  "properties": {
    "toolName": {
      "type": "string",
      "minLength": 1,
      "maxLength": 64,
      "description": "Unique tool identifier"
    },
    "inputSchema": {
      "type": "object",
      "description": "JSON Schema for tool parameters"
    },
    "outputSchema": {
      "type": "object",
      "description": "JSON Schema for tool output"
    }
  },
  "required": ["toolName", "inputSchema"],
  "additionalProperties": false
}
```

---

## 3. Patterns for Skills, Hooks, and MCP Servers

### 3.1 Skill Definition Schema (SKILL.md YAML Frontmatter)

```yaml
# Recommended YAML schema for SKILL.md frontmatter
---
$schema: "https://claude-baton.ai/schemas/skill-v1.json"
name: string (required, kebab-case, 3-64 chars)
version: semver (required)
description: string (required, max 500 chars, used in tool selection)
author: string (optional)
license: string (required, SPDX identifier)
tags: string[] (optional, for categorization)
priority: number (optional, 0-100, default 50)
triggers:
  patterns: string[] (regex patterns)
  keywords: string[] (exact match keywords)
  tools: string[] (tool names that activate this skill)
dependencies:
  packages: string[] (npm packages required)
  skills: string[] (other skills required)
  environment: object (env var requirements)
config:
  defaultModel: string (optional)
  maxTokens: number (optional)
  timeout: number (optional, milliseconds)
---
```

**JSON Schema equivalent:**

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.ai/schemas/skill-v1.json",
  "title": "Skill Definition Schema",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9-]*[a-z0-9]$",
      "minLength": 3,
      "maxLength": 64,
      "description": "Unique skill identifier"
    },
    "version": { "$ref": "#/$defs/semver" },
    "description": {
      "type": "string",
      "maxLength": 500,
      "description": "Used for AI agent tool selection"
    },
    "license": {
      "type": "string",
      "pattern": "^[A-Z0-9.-]+$",
      "description": "SPDX license identifier"
    },
    "priority": {
      "type": "integer",
      "minimum": 0,
      "maximum": 100,
      "default": 50
    },
    "triggers": {
      "type": "object",
      "properties": {
        "patterns": {
          "type": "array",
          "items": { "type": "string", "format": "regex" }
        },
        "keywords": {
          "type": "array",
          "items": { "type": "string" }
        },
        "tools": {
          "type": "array",
          "items": { "type": "string" }
        }
      }
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
    }
  },
  "required": ["name", "version", "description", "license"],
  "additionalProperties": true
}
```

### 3.2 Hook Configuration Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.ai/schemas/hooks-v1.json",
  "title": "Hook Configuration Schema",
  "type": "object",
  "properties": {
    "hooks": {
      "type": "object",
      "propertyNames": {
        "enum": [
          "pre:tool:call",
          "post:tool:call",
          "pre:generation",
          "post:generation",
          "on:error",
          "on:context:overflow",
          "pre:skill:activate",
          "post:skill:activate"
        ]
      },
      "additionalProperties": {
        "type": "object",
        "properties": {
          "handler": {
            "type": "string",
            "description": "Function path or inline code"
          },
          "priority": {
            "type": "integer",
            "minimum": 0,
            "maximum": 1000
          },
          "enabled": {
            "type": "boolean",
            "default": true
          },
          "condition": {
            "type": "object",
            "description": "Conditional execution rules"
          },
          "timeout": {
            "type": "integer",
            "minimum": 100,
            "maximum": 30000,
            "default": 5000
          }
        },
        "required": ["handler"]
      }
    }
  },
  "required": ["hooks"],
  "$defs": {
    "hookName": {
      "type": "string",
      "pattern": "^(pre|post|on):[a-z]+(:[a-z]+)?$"
    }
  }
}
```

**Hook naming convention:**
- `pre:*` - Before an action
- `post:*` - After an action
- `on:*` - Event-triggered

### 3.3 MCP Server Configuration Schema (.mcp.json)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.ai/schemas/mcp-server-v1.json",
  "title": "MCP Server Configuration",
  "type": "object",
  "properties": {
    "mcpServers": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "oneOf": [
          {
            "properties": {
              "command": { "type": "string" },
              "args": {
                "type": "array",
                "items": { "type": "string" }
              },
              "env": {
                "type": "object",
                "additionalProperties": { "type": "string" }
              }
            },
            "required": ["command"]
          },
          {
            "properties": {
              "url": { "type": "string", "format": "uri" },
              "transport": { "const": "sse" }
            },
            "required": ["url"]
          }
        ]
      }
    }
  },
  "required": ["mcpServers"],
  "examples": [
    {
      "mcpServers": {
        "filesystem": {
          "command": "npx",
          "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
        },
        "baton-rag": {
          "url": "http://localhost:8000/mcp",
          "transport": "sse"
        }
      }
    }
  ]
}
```

---

## 4. Versioning and Backward Compatibility

### 4.1 Schema Versioning Strategy

**Three-tier versioning approach:**

```
URI Pattern: https://claude-baton.ai/schemas/{type}-v{major}.json

Examples:
- https://claude-baton.ai/schemas/skill-v1.json
- https://claude-baton.ai/schemas/skill-v2.json
- https://claude-baton.ai/schemas/plugin-manifest-v1.json
```

**Versioning rules:**

| Change Type | Action | URI Change |
|-------------|--------|------------|
| Added optional field | Minor version | Same URI |
| Added required field | Major version | New URI |
| Removed field | Major version | New URI |
| Changed type | Major version | New URI |
| Added enum value | Minor version | Same URI |
| Removed enum value | Major version | New URI |

### 4.2 Backward Compatibility Patterns

**Pattern 1: Deprecated field handling**

```json
{
  "properties": {
    "toolName": {
      "type": "string",
      "description": "Preferred field for tool name"
    },
    "name": {
      "type": "string",
      "description": "DEPRECATED: Use toolName instead",
      "deprecated": true
    }
  },
  "oneOf": [
    { "required": ["toolName"] },
    { "required": ["name"] }
  ]
}
```

**Pattern 2: Version negotiation**

```json
{
  "properties": {
    "schemaVersion": {
      "type": "string",
      "enum": ["1.0", "1.1", "2.0"],
      "default": "1.0"
    },
    "data": {
      "oneOf": [
        { "$ref": "#/$defs/dataV1" },
        { "$ref": "#/$defs/dataV1_1" },
        { "$ref": "#/$defs/dataV2" }
      ]
    }
  },
  "allOf": [
    {
      "if": { "properties": { "schemaVersion": { "const": "1.0" } } },
      "then": { "properties": { "data": { "$ref": "#/$defs/dataV1" } } }
    },
    {
      "if": { "properties": { "schemaVersion": { "const": "1.1" } } },
      "then": { "properties": { "data": { "$ref": "#/$defs/dataV1_1" } } }
    },
    {
      "if": { "properties": { "schemaVersion": { "const": "2.0" } } },
      "then": { "properties": { "data": { "$ref": "#/$defs/dataV2" } } }
    }
  ]
}
```

**Pattern 3: Extension points**

```json
{
  "properties": {
    "extensions": {
      "type": "object",
      "description": "Extension point for future fields",
      "additionalProperties": true
    }
  }
}
```

### 4.3 Migration Schema

```json
{
  "$id": "migration-v1-to-v2.json",
  "type": "object",
  "properties": {
    "fromVersion": { "const": "1.0" },
    "toVersion": { "const": "2.0" },
    "migrations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "path": { "type": "string" },
          "transform": { "type": "string" },
          "description": { "type": "string" }
        }
      }
    }
  }
}
```

---

## 5. Validation Patterns for Complex Nested Objects

### 5.1 Recursive Schemas

**Tree structure validation:**

```json
{
  "$defs": {
    "node": {
      "type": "object",
      "properties": {
        "id": { "type": "string" },
        "children": {
          "type": "array",
          "items": { "$ref": "#/$defs/node" }
        }
      },
      "required": ["id"]
    }
  },
  "$ref": "#/$defs/node"
}
```

### 5.2 Conditional Validation

**if-then-else pattern:**

```json
{
  "properties": {
    "type": { "enum": ["local", "remote", "hybrid"] }
  },
  "allOf": [
    {
      "if": { "properties": { "type": { "const": "local" } } },
      "then": {
        "properties": {
          "path": { "type": "string" }
        },
        "required": ["path"]
      }
    },
    {
      "if": { "properties": { "type": { "const": "remote" } } },
      "then": {
        "properties": {
          "url": { "type": "string", "format": "uri" },
          "auth": { "$ref": "#/$defs/authConfig" }
        },
        "required": ["url"]
      }
    },
    {
      "if": { "properties": { "type": { "const": "hybrid" } } },
      "then": {
        "properties": {
          "path": { "type": "string" },
          "fallbackUrl": { "type": "string", "format": "uri" }
        },
        "required": ["path", "fallbackUrl"]
      }
    }
  ]
}
```

### 5.3 Array Validation Patterns

**Tuple validation (prefixItems in 2020-12):**

```json
{
  "type": "array",
  "prefixItems": [
    { "type": "string", "description": "Command name" },
    { "type": "integer", "description": "Exit code" },
    { "type": "string", "description": "Output" }
  ],
  "items": false,
  "minItems": 3,
  "maxItems": 3
}
```

**Unique items with constraints:**

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "id": { "type": "string" },
      "name": { "type": "string" }
    },
    "required": ["id"]
  },
  "uniqueItems": true
}
```

### 5.4 Cross-field Validation

```json
{
  "type": "object",
  "properties": {
    "startDate": { "type": "string", "format": "date-time" },
    "endDate": { "type": "string", "format": "date-time" }
  },
  "allOf": [
    {
      "if": {
        "properties": {
          "startDate": { "type": "string" },
          "endDate": { "type": "string" }
        },
        "required": ["startDate", "endDate"]
      },
      "then": {
        "comment": "endDate must be after startDate (use custom validator)"
      }
    }
  ]
}
```

**Note**: JSON Schema cannot express "endDate > startDate". Use application-level validation.

### 5.5 Polymorphic Object Validation

```json
{
  "type": "object",
  "properties": {
    "type": { "type": "string" }
  },
  "required": ["type"],
  "oneOf": [
    {
      "properties": {
        "type": { "const": "text" },
        "content": { "type": "string" }
      },
      "required": ["content"]
    },
    {
      "properties": {
        "type": { "const": "image" },
        "url": { "type": "string", "format": "uri" },
        "alt": { "type": "string" }
      },
      "required": ["url"]
    },
    {
      "properties": {
        "type": { "const": "artifact" },
        "artifactId": { "$ref": "#/$defs/uuid" },
        "mimeType": { "type": "string" }
      },
      "required": ["artifactId"]
    }
  ]
}
```

---

## 6. Specific Recommendations for Claude_Baton

### 6.1 Plugin Manifest Schema (plugin.json)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.ai/schemas/plugin-manifest-v1.json",
  "title": "Claude_Baton Plugin Manifest",
  "type": "object",
  "properties": {
    "id": { "$ref": "#/$defs/pluginId" },
    "name": { "type": "string", "maxLength": 100 },
    "version": { "$ref": "#/$defs/semver" },
    "description": { "type": "string", "maxLength": 500 },
    "author": { "type": "string" },
    "license": { "type": "string" },
    "repository": { "type": "string", "format": "uri" },
    "keywords": {
      "type": "array",
      "items": { "type": "string", "maxLength": 50 },
      "maxItems": 20
    },
    "engines": {
      "type": "object",
      "properties": {
        "claude-baton": { "type": "string", "description": "Semver range" }
      },
      "required": ["claude-baton"]
    },
    "skills": {
      "type": "array",
      "items": { "$ref": "#/$defs/skillEntry" }
    },
    "hooks": {
      "type": "object",
      "additionalProperties": { "$ref": "#/$defs/hookEntry" }
    },
    "mcpServers": {
      "type": "object",
      "additionalProperties": { "$ref": "#/$defs/mcpServerEntry" }
    },
    "artifacts": {
      "type": "array",
      "items": { "$ref": "#/$defs/artifactDef" }
    },
    "config": {
      "type": "object",
      "description": "Plugin-specific configuration schema"
    }
  },
  "required": ["id", "version", "description", "engines"],
  "additionalProperties": false,
  "$defs": {
    "pluginId": {
      "type": "string",
      "pattern": "^(@[a-z0-9-]+/)?[a-z0-9-]+$",
      "description": "Scoped or unscoped plugin ID"
    },
    "semver": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+(-[a-zA-Z0-9.-]+)?(\\+[a-zA-Z0-9.-]+)?$"
    },
    "skillEntry": {
      "type": "object",
      "properties": {
        "path": { "type": "string" },
        "enabled": { "type": "boolean", "default": true },
        "priority": { "type": "integer", "minimum": 0, "maximum": 100 }
      },
      "required": ["path"]
    },
    "hookEntry": {
      "type": "object",
      "properties": {
        "handler": { "type": "string" },
        "priority": { "type": "integer", "default": 50 },
        "enabled": { "type": "boolean", "default": true },
        "async": { "type": "boolean", "default": false },
        "timeout": { "type": "integer", "default": 5000 }
      },
      "required": ["handler"]
    },
    "mcpServerEntry": {
      "oneOf": [
        {
          "type": "object",
          "properties": {
            "command": { "type": "string" },
            "args": { "type": "array", "items": { "type": "string" } },
            "env": { "type": "object", "additionalProperties": { "type": "string" } }
          },
          "required": ["command"]
        },
        {
          "type": "object",
          "properties": {
            "url": { "type": "string", "format": "uri" },
            "transport": { "enum": ["sse", "websocket", "stdio"] }
          },
          "required": ["url"]
        }
      ]
    },
    "artifactDef": {
      "type": "object",
      "properties": {
        "type": { "type": "string" },
        "schema": { "type": "object" },
        "handler": { "type": "string" }
      },
      "required": ["type"]
    }
  }
}
```

### 6.2 Baton Protocol Artifact Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.ai/schemas/artifact-v1.json",
  "title": "Baton Artifact Schema",
  "type": "object",
  "properties": {
    "artifactId": { "$ref": "#/$defs/uuid" },
    "type": {
      "type": "string",
      "enum": ["text", "code", "markdown", "json", "image", "file", "structured"]
    },
    "mimeType": { "type": "string" },
    "content": { "type": "string" },
    "metadata": {
      "type": "object",
      "properties": {
        "created": { "type": "string", "format": "date-time" },
        "modified": { "type": "string", "format": "date-time" },
        "source": { "type": "string" },
        "checksum": { "type": "string" },
        "size": { "type": "integer" }
      }
    },
    "relations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": { "enum": ["parent", "child", "derived", "reference"] },
          "targetId": { "$ref": "#/$defs/uuid" }
        }
      }
    }
  },
  "required": ["artifactId", "type"],
  "additionalProperties": true,
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    }
  }
}
```

### 6.3 Validation Best Practices for Claude_Baton

1. **Always validate before execution**
   - Use `jsonschema` library in Python
   - Validate at MCP tool call boundary

2. **Provide helpful error messages**
   ```python
   from jsonschema import validate, ValidationError
   
   try:
       validate(instance=data, schema=schema)
   except ValidationError as e:
       # Provide path to error
       path = "->".join(str(p) for p in e.absolute_path)
       raise ValueError(f"Validation error at {path}: {e.message}")
   ```

3. **Use schema for documentation**
   - Generate TypeScript types with `json-schema-to-typescript`
   - Generate docs with `json-schema-to-md`

4. **Test schemas extensively**
   ```bash
   # CLI validation
   python -m jsonschema --instance plugin.json schema.json
   ```

---

## 7. Summary: Quick Reference

### Schema Checklist

| Element | Required | Recommendation |
|---------|----------|----------------|
| `$schema` | ✅ | Always use 2020-12 |
| `$id` | ✅ | Versioned URI |
| `title` | ✅ | Human-readable name |
| `description` | ✅ | For AI consumption |
| `type` | ✅ | Usually "object" |
| `properties` | ✅ | Define all fields |
| `required` | ⚠️ | Minimal required set |
| `additionalProperties` | ⚠️ | `false` for MCP tools |
| `$defs` | 📝 | Reusable types |
| `examples` | 📝 | 2-3 examples |
| `default` | 📝 | For optional fields |

### Common Patterns

| Need | Pattern |
|------|---------|
| Optional field | Omit from `required` |
| Nullable field | `"type": ["string", "null"]` |
| Either/or | `oneOf` with `required` |
| At least one | `anyOf` with `required` |
| Conditional | `if/then/else` |
| Reusable type | `$defs` + `$ref` |
| Extensible | `additionalProperties: true` |
| Strict | `additionalProperties: false` |

---

## 8. Next Actions for Claude_Baton

1. **Implement core schemas**
   - [ ] `plugin-manifest-v1.json`
   - [ ] `skill-v1.json`
   - [ ] `hooks-v1.json`
   - [ ] `mcp-server-v1.json`
   - [ ] `artifact-v1.json`

2. **Create validation tooling**
   - [ ] JSON Schema validator CLI
   - [ ] VS Code schema association
   - [ ] Pre-commit hooks for validation

3. **Generate type definitions**
   - [ ] TypeScript types from schemas
   - [ ] Zod schemas for runtime validation

4. **Document schemas**
   - [ ] Auto-generate markdown docs
   - [ ] Create migration guides

---

**Report compiled from:**
- JSON Schema 2020-12 specification
- VS Code extension manifest documentation
- npm package.json specification
- MCP (Model Context Protocol) specification
- Claude_Baton existing documentation (JSON_SCHEMA_DEEP_DIVE.md)
- Analysis of existing SKILL.md patterns in the codebase
