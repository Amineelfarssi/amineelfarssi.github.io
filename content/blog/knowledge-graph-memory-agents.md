---
title: "Why Vector Memory Alone Isn't Enough: Knowledge Graph Memory for AI Agents"
date: 2026-03-01
description: "ChromaDB is great for semantic search. But for persistent AI identity and relationship tracking, you need a knowledge graph. Here's how I built one with Neo4j, Graphiti, and nightly ingest."
tags: ["AI Memory", "Knowledge Graph", "Neo4j", "ChromaDB", "Graphiti", "RAG"]
categories: ["Engineering"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Vector databases are fast and convenient. But they can't answer "what did I decide about the auth system 3 weeks ago and why?" For that, you need relationships — and that means a knowledge graph.
{{< /lead >}}

## The Problem with Pure Vector Memory

Most AI memory systems work like this: embed text, store in ChromaDB, retrieve by cosine similarity. It works well for "find things similar to this query."

It fails for:
- **Entity tracking**: "What do I know about Katarina?" → ChromaDB returns chunks, not a coherent entity
- **Temporal reasoning**: "What changed in the codebase this month?" → No native timeline
- **Relationship queries**: "Which decisions depend on the LiteLLM choice?" → No graph traversal
- **Contradiction detection**: "Did I say X before?" → No structured fact store

The missing piece is a **knowledge graph** — structured facts about entities and their relationships over time.

---

## The Stack

{{< mermaid >}}
graph LR
    DAILY["📝 Daily Notes\nmemory/YYYY-MM-DD.md"]
    MEMORY["🧠 MEMORY.md\nlong-term facts"]
    CRON["⏰ Cron @ 2 AM\ndaily-ingest.sh"]
    
    DAILY --> CRON
    MEMORY --> CRON
    
    CRON --> GRAPHITI["🔗 Graphiti v0.28.1\nentity + edge extraction"]
    GRAPHITI --> KIMI["🤖 Kimi K2 Turbo\nLLM extraction"]
    GRAPHITI --> NEO4J["🗄️ Neo4j 5\nDocker, port 7687"]
    GRAPHITI --> EMBED["📐 all-MiniLM-L6-v2\n384-dim, CPU"]
    
    QUERY["🔍 Query Time"] --> RRF["RRF Scoring\nKG + ChromaDB + grep"]
    NEO4J --> RRF
    CHROMA["ChromaDB"] --> RRF
    RAW["Raw files"] --> RRF
    style NEO4J fill:#1e3a5f,color:#fff
    style RRF fill:#10b981,color:#fff
{{< /mermaid >}}

**Graphiti** is the key piece. It's a Python library that takes free-form text, extracts entities and relationships using an LLM, and writes them to Neo4j with temporal metadata. Think of it as a structured fact extractor that builds your knowledge graph automatically.

---

## What Gets Extracted

From a daily note like:

```
Sprint 14 kicked off. Bolt is working on LUM-97 (API key auth).
Sage owns LiteLLM. Lumi's server keeps running out of RAM with 5 concurrent agents.
Added 2GB swap as a safety net.
```

Graphiti extracts:
- **Entities**: Sprint 14, Bolt, Sage, Lumi's server
- **Relationships**: Bolt → WORKS_ON → LUM-97, LUM-97 → TYPE → API key auth
- **Facts**: Lumi's server has memory pressure, 2GB swap added
- **Temporal**: these facts valid from 2026-03-02

After 6 weeks of daily ingestion, the graph has:
- 133 entities
- 117 relationships  
- 23 episodes
- Every significant decision and event from the project

---

## The LLM Choice: Kimi K2 Turbo

{{< alert icon="lightbulb" cardColor="#1e3a5f" iconColor="#60a5fa" textColor="#f0f0f0" >}}
Graphiti normally requires `response_format: json_object`. Kimi's API doesn't support this parameter. The fix: inject a JSON schema requirement into the system prompt via a custom client wrapper.
{{< /alert >}}

I replaced the default OpenAI client in Graphiti with a `KimiClient` wrapper that intercepts calls and adds the JSON schema instruction to the system prompt. This took about 50 lines of Python and now runs cleanly.

Why Kimi? It's cheap (~$0.01 per 1K tokens for K2 Turbo) and the graph extraction quality is good enough. The embedder is `all-MiniLM-L6-v2` running on CPU — fast, free, and sufficient for 384-dim embeddings.

---

## The Hybrid Query: RRF

At query time, I don't rely on just the KG or just ChromaDB. I use **Reciprocal Rank Fusion**:

```python
async def search_all(query: str) -> List[Result]:
    # 1. KG semantic + BFS traversal (Graphiti)
    kg_results = await graphiti.search(query)
    
    # 2. Vector semantic search (ChromaDB)
    chroma_results = chroma.query(query, n_results=5)
    
    # 3. Grep over raw memory files
    grep_results = grep_memory_files(query)
    
    # Combine with RRF scoring
    return rrf_combine([kg_results, chroma_results, grep_results])
```

This gives you the best of all three systems:
- KG: structured facts, entity relationships, temporal context
- ChromaDB: semantic similarity across all memory chunks  
- Grep: exact matches, recent notes not yet ingested

---

## The Nightly Ingest

```bash
# Runs at 2 AM via cron
0 2 * * * /home/amine/.openclaw/workspace/kg/daily-ingest.sh
```

The script:
1. Ingests yesterday's daily notes file (`memory/2026-03-02.md`)
2. Re-ingests `MEMORY.md` if it changed (tracked via `mtime`)
3. Logs everything to `kg/ingest.log`

Each ingest chunk takes ~3-5 seconds (one LLM call per paragraph for entity extraction). A full MEMORY.md ingest with 18 chunks takes about 5-7 minutes — fast enough for nightly cron.

---

## What It Enables

The question "what do I know about Lumi's server issues?" now returns:

> **Entity: Lumi's server (your-ec2-instance)**  
> - t3.medium, 4GB RAM (downgraded from t3.large 2026-02-10)
> - Runs OpenClaw gateway PID 985
> - Memory pressure: zombie gateway processes at 137% CPU (2026-02-28, 2026-03-03)
> - Fix: cleared delivery queue, added 2GB swap, set CPU burst to unlimited
> - Role: agent dispatch server for Sprint 14

That's a coherent entity with history — not just a list of chunks sorted by embedding distance.

For a personal AI agent that needs to maintain context across sessions, weeks, and projects, this architecture is the right foundation. Vector alone is a search engine. Graph + vector is memory.
