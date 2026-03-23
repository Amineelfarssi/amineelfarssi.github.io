---
title: "Graph RAG in Practice: How I Wired Neo4j Into My AI Agent's Memory"
date: 2026-03-23
description: "Vector search finds similar text. Graph RAG finds connected facts. Here's how I built a hybrid retrieval system using Neo4j, Graphiti, and OpenClaw that actually remembers context across sessions."
tags: ["Graph RAG", "Neo4j", "Knowledge Graph", "OpenClaw", "Graphiti", "RAG", "AI Memory"]
categories: ["Engineering"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Vector RAG retrieves documents. Graph RAG retrieves relationships. When your agent needs to reason across entities, timelines, and decisions — not just find similar text — the graph wins.
{{< /lead >}}

## The Problem I Was Trying to Solve

My AI agent (PostSingular, running on OpenClaw) talks to me every day. It helps me build [Luminar](https://luminar.dev), manages my YouTube channel, tracks infrastructure decisions, and remembers context across sessions.

But memory was broken in a subtle way.

I asked it: *"Which decision did we make about the auth system last month, and why?"*

ChromaDB returned 6 chunks. All semantically similar. None of them connected to each other. No timeline. No "we chose bcrypt over passlib because of version incompatibility." Just floating text.

The problem isn't retrieval quality — cosine similarity was working fine. The problem is **structural**. Real knowledge has relationships. Facts connect to other facts. A vector store doesn't model that.

---

## What Graph RAG Actually Means

Standard RAG:
```
Query → embed → cosine search → top-K chunks → LLM
```

Graph RAG:
```
Query → extract entities → traverse graph → ranked subgraph → LLM
```

The difference is what you're searching. Vectors find text that *looks like* your query. Graphs find entities and facts that are *structurally connected* to your query.

For an AI agent with months of context — projects, people, decisions, infrastructure — the graph approach is the only one that scales.

---

## The Stack

{{< mermaid >}}
graph TD
    NOTES["📝 Daily Notes\nmemory/YYYY-MM-DD.md"]
    MEM["🧠 MEMORY.md\nlong-term facts"]
    CRON["⏰ Cron 2 AM\ndaily-ingest.sh"]

    NOTES --> CRON
    MEM --> CRON

    CRON --> GRAPHITI["🔗 Graphiti v0.28.1\nentity + edge extraction"]
    GRAPHITI --> KIMI["🤖 Kimi K2 Turbo\nLLM extraction"]
    GRAPHITI --> MINILM["📐 all-MiniLM-L6-v2\n384-dim embeddings"]
    GRAPHITI --> NEO4J["🗄️ Neo4j 5 Docker\nport 7687"]

    Q["🔍 Query"] --> HYBRID["Hybrid Retrieval\nBFS + RRF Scoring"]
    NEO4J --> HYBRID
    CHROMA["🔎 ChromaDB\n162 semantic chunks"] --> HYBRID
    GREP["📄 grep\nraw file search"] --> HYBRID
    HYBRID --> LLM["Claude / Kimi\nfinal answer"]
{{< /mermaid >}}

**Three layers of retrieval, fused with RRF (Reciprocal Rank Fusion):**

| Layer | What it finds | When it wins |
|-------|--------------|--------------|
| Neo4j BFS | Connected entities, relationships, timelines | "What depends on X?" |
| ChromaDB | Semantically similar chunks | "What did we say about Y?" |
| grep | Exact strings, IDs, code snippets | "Find issue #30099" |

---

## The Ingestion Pipeline

Every night at 2 AM, a cron job ingests the day's notes into Neo4j via Graphiti:

```bash
#!/bin/bash
# daily-ingest.sh
YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)
NOTES_FILE="memory/${YESTERDAY}.md"

if [ -f "$NOTES_FILE" ]; then
    cd /path/to/kg
    .venv/bin/python memory.py ingest "$NOTES_FILE"
fi

# Also ingest MEMORY.md changes
.venv/bin/python memory.py ingest MEMORY.md
```

Graphiti extracts entities and relationships from raw markdown using an LLM (Kimi K2 Turbo), then stores them in Neo4j with timestamps and embeddings.

The result after a few months:

```
Entities:   133
Episodes:   23
Relationships: 117
Mentions:   190
```

---

## The Key Piece: Custom LLM Client for Kimi

Graphiti's default client uses `response_format` JSON schema — which Moonshot's API doesn't support. I had to patch it:

```python
class KimiClient(OpenAIClient):
    """Patches Graphiti to inject JSON schema into system prompt."""
    
    async def generate_response(
        self,
        messages: list[dict],
        response_model: type[BaseModel] | None = None,
        **kwargs
    ) -> dict:
        if response_model:
            schema = response_model.model_json_schema()
            schema_str = json.dumps(schema, indent=2)
            
            # Inject schema into system message instead of response_format
            system_injection = (
                f"\n\nRespond with valid JSON matching this schema:\n"
                f"```json\n{schema_str}\n```\n"
                f"Return ONLY the JSON object, no other text."
            )
            
            for msg in messages:
                if msg.get("role") == "system":
                    msg["content"] += system_injection
                    break
        
        # Remove response_format if present — Moonshot rejects it
        kwargs.pop("response_format", None)
        
        return await super().generate_response(messages, **kwargs)
```

This is the kind of thing you only discover by reading the Graphiti source code. The fix is 20 lines. The debugging is 4 hours.

---

## Query Time: Hybrid Retrieval

When the agent needs to search memory, it hits all three layers:

```python
async def search_all(query: str) -> list[dict]:
    results = []
    
    # 1. Neo4j — entity + relationship traversal
    kg_results = await search_knowledge_graph(query)
    
    # 2. ChromaDB — semantic similarity
    chroma_results = search_chromadb(query, n_results=5)
    
    # 3. grep — exact match fallback
    grep_results = grep_memory_files(query)
    
    # Fuse with RRF
    return reciprocal_rank_fusion([kg_results, chroma_results, grep_results])
```

RRF gives each result a score based on its rank across all sources, not its raw similarity score. A result that ranks 3rd in KG and 2nd in ChromaDB beats one that ranks 1st in only one source. This matters because different queries hit different layers harder.

---

## What Graph RAG Actually Enables

Here's a concrete example. Query: *"What infrastructure decisions did we make in February?"*

**Pure vector (ChromaDB only):**
Returns 5 chunks about infrastructure. Some February, some not. No ordering. No relationships between decisions.

**Graph RAG (Neo4j BFS + RRF):**
Returns a traversal: `InfraDecision[Tailscale-fix] → caused_by → BugReport[ws-security-block] → resolved_by → Fix[serve-https]` with timestamps.

The graph knows *why* the decision was made and *what* it connects to. The vector store just knows it's semantically close to "infrastructure."

---

## The OpenClaw Integration

The whole thing runs inside [OpenClaw](https://openclaw.ai) as my primary agent runtime. Memory search is a first-class tool:

```json
{
  "tool": "memory_search",
  "description": "Search Neo4j KG + ChromaDB + grep. Returns ranked snippets with path + line numbers.",
  "parameters": {
    "query": "string",
    "maxResults": "number",
    "minScore": "number"
  }
}
```

Before every response, the agent runs `memory_search` on queries relevant to the conversation. The KG handles "what did we decide and why." ChromaDB handles "what did we say about this topic." grep handles exact lookups.

Three tools, one fused result, one agent that actually remembers.

---

## What I'd Do Differently

**1. Start with a schema.** Graphiti extracts entities automatically which is great, but defining your own entity types (Person, Project, Decision, Bug) upfront gives you cleaner traversals.

**2. Use Kimi K2 Turbo, not a local model.** I tried Qwen3:8b first. Entity extraction quality was noticeably worse. The graph is only as good as the extraction LLM.

**3. Ingest incrementally, not in bulk.** When I first ran the pipeline I tried to ingest 3 months of notes at once. Rate limits, timeouts, duplicate edges everywhere. Daily cron is the right cadence.

**4. Add a deduplication pass.** After a few months you'll have `Luminar` and `luminar-labs` and `LuminarLabs` as separate entities. Graphiti has deduplication but it needs tuning.

---

## The Setup

Full stack if you want to replicate:

```yaml
Neo4j: 5.x in Docker, ports 7474/7687
Graphiti: v0.28.1
Embedder: all-MiniLM-L6-v2 (384-dim, CPU only)
LLM: Kimi K2 Turbo (moonshot/kimi-k2-turbo-preview)
ChromaDB: collection postsingular_memory, 162 chunks
Cron: daily-ingest.sh at 2 AM
Runtime: OpenClaw (agent gateway + session memory)
```

Code is in my workspace. Will open-source the `memory.py` CLI and KimiClient patch if there's interest.

---

## Results

The difference shows up in long sessions. After 3 months of daily ingestion:

- Agent correctly references decisions made weeks ago without being prompted
- "What changed about X?" queries return timelines, not just similar text
- Cross-entity queries ("everything related to Luminar's auth system") traverse the graph and return structured context
- Grep fallback catches exact IDs, issue numbers, and code that embeddings miss

Graph RAG isn't a silver bullet. It's extra infrastructure, extra latency, and you need an LLM good enough to extract entities cleanly. But for an agent that's your co-builder — not just a chatbot — it's the difference between short-term and long-term memory.

---

*Running this setup on a gaming server (i5-9600K, 64GB RAM, RTX 2080 Ti) via [OpenClaw](https://openclaw.ai). Neo4j browser at localhost:7474 if you want to explore the graph visually.*
