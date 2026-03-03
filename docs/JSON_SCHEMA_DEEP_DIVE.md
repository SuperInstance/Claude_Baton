
# JSON Schema Deep Dive — Official Guide for Claude_Baton Developers

**Version:** 1.0 (March 2026)  
**Current Stable Draft:** 2020-12 (still the recommended & default dialect as of March 2026)  
**Why this file exists:** In Claude_Baton (and MCP in general) every tool’s `inputSchema` and `outputSchema` **must** be valid JSON Schema 2020-12. This is the single deepest reference you will ever need for writing perfect MCP tool schemas, BatonRAG extensions, or any future Claude Code plugins.

**Official Sources (March 2026):**  
- Meta-schema: https://json-schema.org/draft/2020-12/schema  
- Core spec: https://json-schema.org/draft/2020-12/json-schema-core.html  
- Validation spec: https://json-schema.org/draft/2020-12/json-schema-validation.html  
- MCP requirement: SEP-1613 explicitly mandates 2020-12 as the default dialect for all `inputSchema` / `outputSchema`.

### 1. What Is JSON Schema?

JSON Schema is a **vocabulary** that lets you describe the structure, types, constraints, and semantics of JSON data using… JSON itself.

It does four things extremely well:
- **Validation** — “Does this data match the rules?”
- **Documentation** — Human-readable + machine-generatable
- **Hypermedia** — `$ref`, `format`, etc. for links and navigation
- **Code generation** — TypeScript interfaces, Go structs, etc.

In MCP/Claude Code it is **the contract** between the model and your server. If the schema is wrong, the tool call fails before it even reaches your Python/Go code.

### 2. The Anatomy of a JSON Schema Document

Every schema is a JSON object with these top-level keys:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",   // REQUIRED for clarity
  "$id": "https://github.com/yourname/claude-baton/mcp/search-past", // Unique identifier
  "title": "Search Past Generations",
  "description": "Semantic search across baton memoirs...",
  "type": "object",
  "properties": { ... },
  "required": ["query"],
  "additionalProperties": false
}
```

- `$schema` → tells validators which dialect to use (always 2020-12 for us).
- `$id` → absolute URI for `$ref` resolution (critical in large schemas).
- `type` → root type (usually `object` for tool parameters).

### 3. Core Keywords (2020-12) — The Foundation

#### Type System
- `"type": "string" | "number" | "integer" | "boolean" | "array" | "object" | "null"`
- Array form: `"type": ["string", "null"]` (nullable)

#### Object Keywords
- `properties`: { "field": { schema } }
- `required`: ["field1", "field2"]
- `additionalProperties`: boolean or schema (set to `false` in MCP tools!)
- `propertyNames`: schema for key names
- `minProperties` / `maxProperties`

#### Array Keywords
- `items`: schema for every item (or boolean)
- `prefixItems`: tuple validation (replaces old `items` array in 2020-12)
- `minItems` / `maxItems`
- `uniqueItems`

#### String Keywords
- `minLength` / `maxLength`
- `pattern` (regex)
- `format`: "date", "email", "uri", "uuid", "ipv4", etc. (Claude respects these)

#### Number Keywords
- `minimum` / `maximum` / `exclusiveMinimum` / `exclusiveMaximum`
- `multipleOf`

#### Generic Keywords
- `enum`: ["value1", "value2"]
- `const`: exact value
- `default`: suggested value (Claude uses this heavily)

### 4. Advanced Features That Make MCP Tools Powerful

#### Composition Keywords (the real power)
- `allOf`: must match ALL subschemas
- `anyOf`: must match AT LEAST ONE
- `oneOf`: must match EXACTLY ONE
- `not`: must NOT match

Example in BatonRAG (recall-decision):
```json
"oneOf": [
  { "required": ["decisionId"] },
  { "required": ["keywords"] }
]
```

#### Conditional Logic (if / then / else)
```json
"if": { "properties": { "type": { "const": "advanced" } } },
"then": { "required": ["advancedOptions"] },
"else": { "not": { "required": ["advancedOptions"] } }
```

#### References (`$ref`)
- Internal: `"$ref": "#/$defs/myType"`
- External: `"$ref": "https://json-schema.org/draft/2020-12/schema"`
- Recursive references are fully supported in 2020-12

#### Definitions (`$defs`)
Store reusable types:
```json
"$defs": {
  "generationId": { "type": "string", "pattern": "^v[0-9]+$" }
}
```

### 5. How JSON Schema Works in MCP Tool Schemas (BatonRAG Specific)

Every `@server.tool` or `.mcp.json` entry contains an `inputSchema` that is a full JSON Schema document.

Claude Code does this automatically:
1. Loads your MCP server
2. Calls the internal `list_tools()` (or reads .mcp.json)
3. Injects the tools into the agent’s context with your exact schema
4. When the agent calls the tool, Claude Code validates the arguments against `inputSchema` **before** sending to your server

**Best practices we follow in BatonRAG:**
- Always set `"additionalProperties": false` — prevents junk parameters
- Use `description` on every field and on the tool itself
- Add `default` values wherever sensible
- Use `enum` for mode strings
- Keep schemas < 4 KB (Claude token limit)

### 6. Output Schemas (outputSchema)

You can also declare what your tool returns. Example for `search-past`:

```json
"outputSchema": {
  "type": "object",
  "properties": {
    "hits": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "generation": { "type": "string" },
          "score": { "type": "number" },
          "content": { "type": "string" }
        },
        "required": ["generation", "score", "content"]
      }
    }
  },
  "required": ["hits"]
}
```

Claude uses this for better reasoning on the returned data.

### 7. Validation Behavior (2020-12 Changes from Older Drafts)

Key improvements over 2019-09 / draft-07:
- `prefixItems` + `items` for proper tuples
- `unevaluatedProperties` / `unevaluatedItems` (we rarely need them)
- Better `$ref` resolution and recursive anchoring
- `format` keywords are now annotation-only unless you enable format validation

### 8. Practical Examples for Claude_Baton

**Full `search-past` schema (exact copy from our server):**
(See MCP_SERVER_SETUP.md — it is 100 % 2020-12 compliant.)

**Complex example — future “publish-skill” tool:**
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "skillName": { "type": "string", "minLength": 3 },
    "mode": { "enum": ["aggressive", "conservative", "human-gated"] },
    "generationsToInclude": { "type": "array", "items": { "$ref": "#/$defs/generationId" } }
  },
  "required": ["skillName", "mode"],
  "$defs": {
    "generationId": { "type": "string", "pattern": "^v\\d+$" }
  }
}
```

### 9. Tools & Libraries You Will Use

- Python: `jsonschema` + `mcp-sdk` (already in our server.py)
- Testing: `python -m jsonschema --instance test.json schema.json`
- Generation: `datamodel-codegen` or `quicktype`
- VS Code: “JSON Schema” extension + our repo’s `.vscode/settings.json`

### 10. Common Pitfalls & Pro Tips (2026 Edition)

- Never omit `$schema` in published schemas
- `additionalProperties: false` is mandatory for MCP tools
- `oneOf` + `required` is the cleanest way to do “either A or B”
- Descriptions are gold — Claude reads every word
- Test with real agent calls, not just validators
- If you change a schema, restart the MCP server (Claude Code auto-detects but sometimes needs a nudge)

---

**Commit this file now.**

You now have the deepest, most practical JSON Schema reference tailored exactly for Claude_Baton and MCP tool development.

Next recommended actions:
- Reply **“Generate Week 2 full skills”** → complete Onboarder/Archivist SKILL.md with perfect MCP tool calls
- Or **“Add outputSchema to BatonRAG server”** → I’ll update server.py instantly

This knowledge is what separates toy MCP servers from production-grade ones like BatonRAG.

We are now fully armed. Let’s ship the plugin that makes context windows obsolete.
```
