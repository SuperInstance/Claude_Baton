{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://claude-baton.ai/schemas/artifact-v1.json",
  "title": "Claude_Baton Artifact Schema (Baton Protocol)",
  "description": "Defines the artifact format for the Baton Protocol. Artifacts are structured outputs that can be stored, referenced, and composed across agent sessions.",
  "type": "object",
  "properties": {
    "$schema": {
      "type": "string",
      "const": "https://claude-baton.ai/schemas/artifact-v1.json"
    },
    "artifactId": {
      "$ref": "#/$defs/uuid",
      "description": "Unique artifact identifier (UUID v4)"
    },
    "version": {
      "type": "integer",
      "minimum": 1,
      "default": 1,
      "description": "Artifact version for immutable versioning"
    },
    "parentVersion": {
      "type": "integer",
      "description": "Parent version if this is a modification"
    },
    "type": {
      "type": "string",
      "enum": [
        "text",
        "markdown",
        "code",
        "json",
        "yaml",
        "html",
        "css",
        "image",
        "audio",
        "video",
        "data",
        "document",
        "spreadsheet",
        "presentation",
        "archive",
        "binary",
        "structured"
      ],
      "description": "Artifact content type"
    },
    "customType": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9-]*:[a-z][a-z0-9-]*$",
      "description": "Custom type in vendor:type format (e.g., 'baton:memory')"
    },
    "mimeType": {
      "type": "string",
      "pattern": "^[a-z]+/[a-z0-9+-]+$",
      "description": "MIME type of the content"
    },
    "encoding": {
      "type": "string",
      "enum": ["utf-8", "base64", "binary", "gzip"],
      "default": "utf-8",
      "description": "Content encoding"
    },
    "content": {
      "type": "string",
      "description": "Artifact content (text or base64-encoded binary)"
    },
    "contentRef": {
      "$ref": "#/$defs/contentReference",
      "description": "Reference to external content instead of inline content"
    },
    "metadata": {
      "$ref": "#/$defs/artifactMetadata",
      "description": "Artifact metadata"
    },
    "schema": {
      "type": "object",
      "description": "JSON Schema for structured artifacts"
    },
    "relations": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/artifactRelation"
      },
      "description": "Relationships to other artifacts"
    },
    "annotations": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/annotation"
      },
      "description": "User or agent annotations"
    },
    "provenance": {
      "$ref": "#/$defs/provenance",
      "description": "Origin and history information"
    },
    "access": {
      "$ref": "#/$defs/accessControl",
      "description": "Access control settings"
    },
    "ttl": {
      "type": "integer",
      "minimum": 0,
      "description": "Time-to-live in seconds (0 = permanent)"
    },
    "checksum": {
      "$ref": "#/$defs/checksum",
      "description": "Content checksum for integrity verification"
    }
  },
  "required": ["artifactId", "type"],
  "anyOf": [
    { "required": ["content"] },
    { "required": ["contentRef"] }
  ],
  "additionalProperties": true,
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$",
      "description": "UUID v4"
    },
    "contentReference": {
      "type": "object",
      "properties": {
        "uri": {
          "type": "string",
          "format": "uri",
          "description": "URI to external content"
        },
        "storageType": {
          "type": "string",
          "enum": ["file", "s3", "http", "ipfs", "dataurl"],
          "description": "Storage backend type"
        },
        "size": {
          "type": "integer",
          "minimum": 0,
          "description": "Content size in bytes"
        },
        "range": {
          "type": "object",
          "properties": {
            "start": { "type": "integer", "minimum": 0 },
            "end": { "type": "integer", "minimum": 0 }
          },
          "description": "Byte range for partial content"
        }
      },
      "required": ["uri", "storageType"],
      "additionalProperties": false
    },
    "artifactMetadata": {
      "type": "object",
      "properties": {
        "title": {
          "type": "string",
          "maxLength": 500,
          "description": "Human-readable title"
        },
        "description": {
          "type": "string",
          "maxLength": 2000,
          "description": "Description of the artifact"
        },
        "tags": {
          "type": "array",
          "items": {
            "type": "string",
            "pattern": "^[a-z][a-z0-9-]*$",
            "maxLength": 50
          },
          "maxItems": 20,
          "description": "Tags for categorization"
        },
        "created": {
          "type": "string",
          "format": "date-time",
          "description": "Creation timestamp"
        },
        "modified": {
          "type": "string",
          "format": "date-time",
          "description": "Last modification timestamp"
        },
        "author": {
          "type": "string",
          "description": "Author identifier (user or agent)"
        },
        "source": {
          "type": "object",
          "properties": {
            "type": {
              "type": "string",
              "enum": ["user", "agent", "tool", "import", "generated"]
            },
            "id": { "type": "string" },
            "version": { "type": "string" }
          },
          "description": "Source information"
        },
        "language": {
          "type": "string",
          "pattern": "^[a-z]{2}(-[A-Z]{2})?$",
          "description": "Language code (ISO 639-1)"
        },
        "size": {
          "type": "integer",
          "minimum": 0,
          "description": "Content size in bytes"
        },
        "lines": {
          "type": "integer",
          "minimum": 0,
          "description": "Number of lines (for text/code)"
        },
        "extension": {
          "type": "string",
          "pattern": "^\\.[a-z0-9]+$",
          "description": "File extension"
        },
        "filename": {
          "type": "string",
          "description": "Original filename"
        },
        "custom": {
          "type": "object",
          "description": "Custom metadata fields"
        }
      },
      "additionalProperties": true
    },
    "artifactRelation": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": [
            "parent",
            "child",
            "derived",
            "reference",
            "supersedes",
            "superseded-by",
            "fork",
            "merge",
            "embed",
            "attachment",
            "related"
          ],
          "description": "Relationship type"
        },
        "targetId": {
          "$ref": "#/$defs/uuid",
          "description": "Target artifact ID"
        },
        "targetVersion": {
          "type": "integer",
          "description": "Target artifact version (optional)"
        },
        "context": {
          "type": "string",
          "description": "Context or reason for the relationship"
        }
      },
      "required": ["type", "targetId"],
      "additionalProperties": false
    },
    "annotation": {
      "type": "object",
      "properties": {
        "id": {
          "$ref": "#/$defs/uuid",
          "description": "Annotation ID"
        },
        "type": {
          "type": "string",
          "enum": ["comment", "highlight", "bookmark", "tag", "rating", "correction"],
          "description": "Annotation type"
        },
        "range": {
          "type": "object",
          "properties": {
            "startLine": { "type": "integer" },
            "endLine": { "type": "integer" },
            "startColumn": { "type": "integer" },
            "endColumn": { "type": "integer" },
            "startOffset": { "type": "integer" },
            "endOffset": { "type": "integer" }
          },
          "description": "Range within the content"
        },
        "content": {
          "type": "string",
          "description": "Annotation content"
        },
        "author": {
          "type": "string",
          "description": "Annotation author"
        },
        "created": {
          "type": "string",
          "format": "date-time"
        },
        "metadata": {
          "type": "object"
        }
      },
      "required": ["id", "type"],
      "additionalProperties": false
    },
    "provenance": {
      "type": "object",
      "properties": {
        "sessionId": {
          "type": "string",
          "description": "Session where artifact was created"
        },
        "toolCall": {
          "type": "object",
          "properties": {
            "tool": { "type": "string" },
            "arguments": { "type": "object" },
            "timestamp": { "type": "string", "format": "date-time" }
          },
          "description": "Tool call that produced this artifact"
        },
        "modelInfo": {
          "type": "object",
          "properties": {
            "model": { "type": "string" },
            "provider": { "type": "string" },
            "version": { "type": "string" }
          },
          "description": "Model used to generate content"
        },
        "prompt": {
          "type": "string",
          "description": "Prompt used to generate content (truncated if long)"
        },
        "chain": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "step": { "type": "integer" },
              "artifactId": { "$ref": "#/$defs/uuid" },
              "action": { "type": "string" }
            }
          },
          "description": "Chain of artifacts leading to this one"
        }
      },
      "additionalProperties": true
    },
    "accessControl": {
      "type": "object",
      "properties": {
        "visibility": {
          "type": "string",
          "enum": ["private", "session", "project", "public"],
          "default": "session",
          "description": "Visibility level"
        },
        "permissions": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "principal": { "type": "string" },
              "actions": {
                "type": "array",
                "items": {
                  "type": "string",
                  "enum": ["read", "write", "delete", "share", "admin"]
                }
              }
            }
          },
          "description": "Access control list"
        },
        "encrypted": {
          "type": "boolean",
          "default": false,
          "description": "Whether content is encrypted"
        }
      },
      "additionalProperties": false
    },
    "checksum": {
      "type": "object",
      "properties": {
        "algorithm": {
          "type": "string",
          "enum": ["md5", "sha1", "sha256", "sha512", "blake3"],
          "description": "Hash algorithm"
        },
        "value": {
          "type": "string",
          "pattern": "^[a-f0-9]+$",
          "description": "Hex-encoded checksum value"
        }
      },
      "required": ["algorithm", "value"],
      "additionalProperties": false
    }
  },
  "examples": [
    {
      "artifactId": "550e8400-e29b-41d4-a716-446655440000",
      "version": 1,
      "type": "markdown",
      "mimeType": "text/markdown",
      "content": "# Project Overview\n\nThis document provides an overview of the project...",
      "metadata": {
        "title": "Project Overview",
        "tags": ["documentation", "project"],
        "created": "2025-01-15T10:30:00Z",
        "author": "agent-claude"
      },
      "provenance": {
        "sessionId": "session-123",
        "modelInfo": {
          "model": "claude-3-opus",
          "provider": "anthropic"
        }
      }
    },
    {
      "artifactId": "660e8400-e29b-41d4-a716-446655440001",
      "type": "code",
      "mimeType": "application/typescript",
      "content": "export class UserService {\n  private users: Map<string, User>;\n  \n  constructor() {\n    this.users = new Map();\n  }\n  \n  async findById(id: string): Promise<User | undefined> {\n    return this.users.get(id);\n  }\n}",
      "metadata": {
        "title": "User Service",
        "extension": ".ts",
        "language": "en"
      },
      "relations": [
        {
          "type": "derived",
          "targetId": "550e8400-e29b-41d4-a716-446655440000",
          "context": "Generated from project overview"
        }
      ],
      "annotations": [
        {
          "id": "770e8400-e29b-41d4-a716-446655440002",
          "type": "comment",
          "range": {
            "startLine": 10,
            "endLine": 15
          },
          "content": "Consider adding error handling here",
          "author": "user",
          "created": "2025-01-15T11:00:00Z"
        }
      ]
    }
  ]
}
