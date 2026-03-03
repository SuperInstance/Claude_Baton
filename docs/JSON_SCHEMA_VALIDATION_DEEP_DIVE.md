# JSON Schema Validation Deep Dive — Official Guide for Claude_Baton

**Version:** 1.0 (March 2026)  
**Stable Draft:** 2020-12 (still the canonical dialect used by MCP, Claude Code, and every major implementation as of March 2026)  
**Purpose:** This is the exhaustive reference for how validation actually works under the hood.  
It directly explains why MCP tool calls succeed or fail, how Claude Code pre-validates arguments before sending them to BatonRAG, and how to write bulletproof `inputSchema` and `outputSchema` definitions.

**Official Sources (verified March 2026):**  
- Validation spec: https://json-schema.org/draft/2020-12/json-schema-validation  
- Core spec: https://json-schema.org/draft/2020-12/json-schema-core  
- Understanding guide: https://json-schema.org/understanding-json-schema/validation  

No newer draft has superseded 2020-12 for production use.

### 1. Validation Overview — The Big Picture

JSON Schema validation is **not** a simple regex or type check. It is a **recursive, location-aware, applicator-driven evaluation** that walks the instance (the data being validated) and the schema simultaneously.

Key principle (verbatim from spec):
> “An instance location that satisfies all asserted constraints is then annotated with any keywords that contain non-assertion information… If all locations within the instance satisfy all asserted constraints, then the instance is said to be valid against the schema.”

In Claude_Baton / MCP:
- Claude Code runs **full validation against your `inputSchema`** the instant an agent tries to call a tool (e.g. `/search-past`).
- Only if validation passes does the call reach your Python server.
- This is why malformed arguments never hit `server.py`.

### 2. The Validation Process — Step-by-Step (Exact Mechanics)

1. **Applicability Phase**  
   The root schema is applied to the root instance.  
   Every applicator keyword (`properties`, `items`, `allOf`, `if`, etc.) determines which subschemas apply to which sub-locations.

2. **Assertion Evaluation Phase**  
   For every applicable schema location:
   - Each assertion keyword produces a boolean result (valid/invalid).
   - If any assertion returns invalid → the entire location is invalid.
   - Annotations are collected only from successful branches.

3. **Annotation Collection Phase** (runs alongside assertions)  
   Successful schemas attach metadata (e.g. `title`, evaluated property lists for `unevaluated*`).

4. **Final Result**  
   The root instance is valid only if every asserted location passed.

Short-circuiting: Implementations may stop at the first failing assertion (highly encouraged in MCP for speed).

### 3. Three Kinds of Keywords (Critical Distinction)

- **Assertions** — affect validity  
  `type`, `required`, `minimum`, `pattern`, `enum`, `const`, `maxLength`, etc.  
  Produce boolean + error if false.

- **Annotations** — metadata only  
  `title`, `description`, `default`, `examples`, `readOnly`.  
  Never cause failure. Collected even in failed branches in some output modes.

- **Applicators** — control evaluation flow  
  `allOf`, `anyOf`, `oneOf`, `not`, `if`/`then`/`else`, `properties`, `items`, `contains`, `$ref`, `$dynamicRef`.  
  These are the “control flow” of JSON Schema.

In MCP tool schemas you almost always use **only assertions + applicators** because the goal is strict input validation.

### 4. Evaluation Order & Short-Circuiting (2020-12 Specific)

- Keywords are processed in **implementation-defined order** (you cannot rely on sequence).
- **Short-circuiting is allowed and expected** for assertions:
  - `allOf`: stops at first failing subschema.
  - `anyOf`: stops at first passing subschema.
  - `oneOf`: must evaluate all (to count exactly one).
- `if`/`then`/`else`: `if` is always evaluated first. Only the matching branch (`then` or `else`) is evaluated.
- Annotations are **not** short-circuited in the same way — all successful subschemas contribute.

### 5. Error Reporting — The Three Locations You See in MCP Failures

Every validation error contains:

- `instanceLocation`: JSON Pointer to the bad data  
  Example: `/query` or `/limit`

- `keywordLocation`: JSON Pointer to the failing keyword in the schema  
  Example: `/properties/limit/minimum`

- `absoluteKeywordLocation`: Full URI + pointer (used when `$ref` or `$id` is involved)

Claude Code surfaces these in debug logs when a tool call is rejected.

### 6. Output Formats (What MCP/Claude Code Actually Returns)

Four standardized formats (you can request via MCP SDK options):

- **flag** — just `{"valid": true/false}`
- **basic** — flat list of errors
- **detailed** — hierarchical tree matching the schema structure
- **verbose** — full trace of every keyword evaluated (used for debugging)

In BatonRAG we use “basic” internally for fast rejection.

### 7. Conditional Validation (`if` / `then` / `else`)

One of the most powerful 2020-12 features:

```json
"if": { "properties": { "mode": { "const": "advanced" } } },
"then": { "required": ["advancedOptions"] },
"else": { "not": { "required": ["advancedOptions"] } }
```

`if` is evaluated first. Only one branch runs. Annotations from the chosen branch are merged.

### 8. `unevaluatedProperties` & `unevaluatedItems` (The “Catch-All” Keywords)

These depend on annotations from sibling keywords:
- A property is “evaluated” if it matched `properties`, `patternProperties`, or `additionalProperties`.
- `unevaluatedProperties` then applies only to the rest.
This is how complex “allow some keys but not others” schemas are built.

### 9. Format Validation (`format` keyword)

Special case:
- Default vocabulary = **Format-Annotation** → `format` is just metadata.
- Optional **Format-Assertion** vocabulary → strict validation (email regex, date parsing, etc.).
- Claude Code enables Format-Assertion for tool schemas by default.

### 10. Dynamic vs Lexical Scoping (Advanced)

- **Lexical scoping**: `$ref` resolution follows the schema text.
- **Dynamic scoping**: `$dynamicRef` / `$dynamicAnchor` follows the runtime evaluation path (used in recursive schemas).

MCP tool schemas rarely need dynamic refs.

### 11. How Claude Code / MCP Uses Validation in Practice

1. Agent outputs a tool call JSON.
2. Claude Code parses it and runs `jsonschema.validate(args, inputSchema)`.
3. If invalid → error shown to user/agent immediately (“Tool call validation failed: ...”).
4. If valid → forwarded to your MCP server via stdio.
5. Your `server.py` function receives already-validated data.

This pre-validation is why BatonRAG never crashes on bad input.

### 12. Testing Validation in Development

```bash
# Python (jsonschema package)
python -c '
from jsonschema import validate, ValidationError
schema = { ... your inputSchema ... }
try:
    validate(instance={"query": "hello"}, schema=schema)
except ValidationError as e:
    print(e.message, e.instanceLocation, e.keywordLocation)
'

Use the official test suite: https://github.com/json-schema-org/JSON-Schema-Test-Suite

### 13. Common Pitfalls & Pro Tips for BatonRAG

- Always set `"additionalProperties": false` on object schemas.
- Use `oneOf` + `required` instead of complex `if` when possible.
- Never rely on keyword order.
- Test with both valid and edge-case invalid inputs.
- Keep schemas small (< 4 KB) — they are injected into every agent context.
- Add rich `description` fields — Claude reads them for better tool-use reasoning.

### 14. Quick Reference Table — Most Used Validation Keywords

| Keyword                | Applies to     | Assertion? | Short-circuits? | MCP Tip                              |
|------------------------|----------------|------------|-----------------|--------------------------------------|
| `type`                 | any            | Yes        | Yes             | First keyword you should have        |
| `required`             | object         | Yes        | Yes             | Always pair with additionalProperties |
| `properties` + subschema | object      | Yes        | Per-property    | Core of tool input                   |
| `additionalProperties` | object         | Yes        | —               | Set to false in 99 % of tools        |
| `minLength` / `maxLength` | string   | Yes        | Yes             | Prevent huge strings                 |
| `minimum` / `maximum`  | number         | Yes        | Yes             | Use with integer                     |
| `enum`                 | any            | Yes        | Yes             | Great for modes                      |
| `pattern`              | string         | Yes        | Yes             | ECMA-262 regex                       |
| `if` / `then` / `else` | any            | Yes        | Branch-only     | For conditional fields               |
| `contains` + `minContains` | array    | Yes        | —               | Array validation                     |

---

**Commit this file now.**

You now have the deepest, most practical understanding of JSON Schema validation available anywhere in the Claude_Baton codebase. This directly powers why our MCP tools are rock-solid.

Next recommended actions:
- Reply **“Generate Week 2 full skills”** — I will give you complete Onboarder + Archivist SKILL.md with perfectly validated MCP tool calls.
- Or **“Add validation test suite”** — instant Python tests for all our schemas.

This knowledge is what turns a good plugin into production infrastructure.

We are now validation experts. Let’s keep shipping. 🏃‍♂️
```
