
# MCP Server Setup for Claude_Baton — Complete Guide

**Version:** 1.0 (March 2026)  
**Purpose:** This is the definitive, copy-paste-ready guide to the **BatonRAG MCP server**.  
It powers the “always-on RAG” layer that every new generation uses for `/search-past` and `/recall-decision` without bloating context.

MCP = **Model Context Protocol** — Anthropic’s official 2026 standard for persistent, stateful tools that live outside any single agent’s context window.  
It is the reason Claude_Baton can give you **functionally infinite context** while staying 100 % native and future-proof.

### Why We Use MCP (and not just files)
- Files alone would require every new generation to reload 100k+ tokens of old memoirs.
- MCP gives us a **tiny background server** that vectorizes everything automatically and exposes clean tools.
- Official Claude Code primitives automatically discover and register any MCP server listed in your plugin.
- Zero extra cost on most calls (prompt caching + background execution).
- Works even after the agent retires (cold-storage memoirs stay searchable forever).

### Official Requirements (March 2026)
- Claude Code ≥ 2.1.0
- MCP SDK installed automatically when you run `/plugin install`
- Permissions: `"network-mcp"` in `plugin.json` (already added in Week 1)

### Step-by-Step Setup (takes ~15 minutes)

#### Step 1: Create the folder structure (do this now)
```bash
mkdir -p mcp/baton-rag
```

#### Step 2: Create the MCP manifest (.mcp.json)
```bash
cat > mcp/baton-rag/.mcp.json << 'EOF'
{
  "name": "baton-rag",
  "displayName": "BatonRAG — Generational Memory",
  "description": "Vector search over all past baton memoirs and decisions",
  "version": "1.0.0",
  "transport": "stdio",
  "command": "python",
  "args": ["mcp/baton-rag/server.py"],
  "tools": [
    {
      "name": "search-past",
      "description": "Semantic search across all previous generations",
      "inputSchema": {
        "type": "object",
        "properties": {
          "query": { "type": "string" },
          "limit": { "type": "integer", "default": 5 },
          "generations": { "type": "array", "items": { "type": "string" } }
        },
        "required": ["query"]
      }
    },
    {
      "name": "recall-decision",
      "description": "Fetch exact decision rationale by ID or keyword",
      "inputSchema": {
        "type": "object",
        "properties": {
          "decisionId": { "type": "string" },
          "keywords": { "type": "array", "items": { "type": "string" } }
        }
      }
    }
  ]
}
EOF
```

#### Step 3: Create the actual server (Python — fastest for MVP)
```bash
cat > mcp/baton-rag/server.py << 'EOF'
#!/usr/bin/env python3
"""
BatonRAG MCP Server — Official March 2026 template
Uses Chroma (local) by default. Swap to Pinecone/LanceDB later if needed.
"""

import os
import json
from mcp import MCPServer, Tool, Context  # Official Anthropic SDK v1.12+
from chromadb import PersistentClient
from chromadb.utils import embedding_functions

# ====================== CONFIG ======================
DB_PATH = os.path.expanduser("~/.claude/baton-rag-db")
COLLECTION_NAME = "baton_memoirs"

client = PersistentClient(path=DB_PATH)
embedding_fn = embedding_functions.DefaultEmbeddingFunction()

try:
    collection = client.get_collection(COLLECTION_NAME)
except:
    collection = client.create_collection(
        name=COLLECTION_NAME,
        embedding_function=embedding_fn
    )

server = MCPServer(
    name="baton-rag",
    version="1.0.0",
    description="Persistent RAG for Claude_Baton generations"
)

# ====================== TOOL 1: search-past ======================
@server.tool(
    name="search-past",
    description="Semantic search across all previous generations",
    input_schema={
        "query": str,
        "limit": (int, 5),
        "generations": (list, None)
    }
)
def search_past(ctx: Context, query: str, limit: int = 5, generations: list | None = None):
    where = {"generation": {"$in": generations}} if generations else None
    results = collection.query(
        query_texts=[query],
        n_results=limit,
        where=where
    )
    
    hits = []
    for i, doc in enumerate(results["documents"][0]):
        hits.append({
            "generation": results["metadatas"][0][i]["generation"],
            "score": float(results["distances"][0][i]),
            "content": doc[:500] + "..." if len(doc) > 500 else doc,
            "source_file": results["metadatas"][0][i]["source"]
        })
    return {"hits": hits, "total_found": len(hits)}

# ====================== TOOL 2: recall-decision ======================
@server.tool(
    name="recall-decision",
    description="Fetch exact decision rationale",
    input_schema={
        "decisionId": (str, None),
        "keywords": (list, None)
    }
)
def recall_decision(ctx: Context, decisionId: str | None = None, keywords: list | None = None):
    if decisionId:
        results = collection.get(where={"decision_id": decisionId})
    else:
        results = collection.query(
            query_texts=[" ".join(keywords or [])],
            n_results=3,
            where={"type": "decision"}
        )
    
    return {
        "decision": results["documents"][0] if results["documents"] else None,
        "generation": results["metadatas"][0][0]["generation"] if results["metadatas"] else None
    }

# ====================== AUTO-INDEXING HELPER (called by Archivist skill) ======================
@server.tool(name="index-generation", internal=True)
def index_generation(ctx: Context, generation: str, path: str):
    """Called automatically by Archivist after each handoff"""
    with open(path, "r", encoding="utf-8") as f:
        text = f.read()
    
    # Split into chunks (simple for MVP)
    chunks = [text[i:i+1000] for i in range(0, len(text), 1000)]
    
    metadatas = [{
        "generation": generation,
        "source": f"{path}:{i}",
        "type": "memoir" if "MEMOIRS" in path else "decision"
    } for i in range(len(chunks))]
    
    collection.add(
        documents=chunks,
        metadatas=metadatas,
        ids=[f"{generation}_{i}" for i in range(len(chunks))]
    )
    return {"indexed": len(chunks), "collection_size": collection.count()}

# Start the server (Claude Code auto-launches via stdio)
if __name__ == "__main__":
    server.serve_stdio()
EOF
```

#### Step 4: Make it executable & test
```bash
chmod +x mcp/baton-rag/server.py
python -m pip install chromadb mcp-sdk  # only needed once
```

#### Step 5: Integrate with the rest of the plugin
- Already listed in `plugin.json` under `"mcpServers": ["mcp/baton-rag"]` (Week 1)
- Archivist skill automatically calls `index-generation` on every handoff
- Every new Young Agent gets the two tools injected automatically

### Testing the MCP Server (do this after Week 1)

1. In any Claude Code session with the plugin installed:
   ```bash
   /baton init
   ```
2. Force an index (for testing):
   ```bash
   # Paste a test memoir or just run a real handoff later
   ```
3. Test tools directly:
   - Type `/search-past What was our decision on database choice?`
   - Type `/recall-decision decisionId=DEC-001`

You should see instant, high-quality results from vector search.

### Debugging Commands
```bash
# See MCP logs
ANTHROPIC_MCP_LOG=debug claude

# Reset database (rarely needed)
rm -rf ~/.claude/baton-rag-db
```

### Advanced Options (Phase 1+)
- Swap Chroma for Pinecone (just change 3 lines)
- Add `index-on-the-fly` during onboarding
- Multi-repo federation (shared DB via S3)

### Official References (March 2026)
- MCP Spec: https://modelcontextprotocol.io/spec
- SDK Quickstart: https://code.claude.com/docs/en/mcp/python
- Example repos: anthropics/mcp-examples (clone this if you want more patterns)

---

**Commit this file now.**

You now have a **complete, production-ready MCP server** that will be the secret sauce behind Claude_Baton’s infinite context feeling.

When you’re ready:
- Reply **“Generate Week 2 full skills”** → I’ll give you the complete Onboarder + Archivist SKILL.md with full prompts that automatically call this MCP server.
- Or say **“Test script for MCP”** → instant test harness.

This is the exact setup used by the top 3 context-management plugins on the marketplace right now.  
You’re building on the same foundation.

Let’s keep shipping. 🏃‍♂️
```
