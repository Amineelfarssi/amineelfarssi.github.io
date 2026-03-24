---
title: "Graph RAG in Practice: How I Wired Neo4j Into My AI Agent's Memory"
date: 2026-03-23
description: "Vector search finds similar text. Graph RAG finds connected facts. Here is how I built a hybrid retrieval system using Neo4j, Graphiti, and OpenClaw that actually remembers context across sessions."
tags: ["Graph RAG", "Neo4j", "Knowledge Graph", "OpenClaw", "Graphiti", "RAG", "AI Memory"]
categories: ["Engineering"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Vector RAG retrieves documents. Graph RAG retrieves relationships. When your agent needs to reason across entities, timelines, and decisions, the graph wins.
{{< /lead >}}

{{< button href="/graph-rag/" target="_self" >}}
Open Interactive Version →
{{< /button >}}

---

## The Problem I Was Trying to Solve

My AI agent PostSingular, running on OpenClaw, talks to me every day. It helps me build Luminar, manages my YouTube channel, and tracks infrastructure decisions across sessions.

But memory was broken in a subtle way.

I asked it: **"Which decision did we make about the auth system last month, and why?"**

ChromaDB returned 6 chunks. All semantically similar. None of them connected to each other. No timeline. No causal chain. Just floating text.

The problem is not retrieval quality. The problem is **structural**. Real knowledge has relationships. A vector store does not model that.

---

## What Graph RAG Actually Means

Standard RAG:


Graph RAG:


Vectors find text that *looks like* your query. Graphs find entities that are *structurally connected* to it.

---

## The Stack

Three retrieval layers, fused with **RRF (Reciprocal Rank Fusion)**:

| Layer | Tool | When It Wins |
|-------|------|-------------|
| Neo4j BFS | Graphiti v0.28.1 | Entity relationships, causal chains, timelines |
| ChromaDB | all-MiniLM-L6-v2 | Semantic similarity, topic recall |
| grep | ripgrep | Exact IDs, issue numbers, code snippets |

After a few months of daily ingestion: **133 entities, 117 relationships, 23 episodes, 162 semantic chunks.**

---

## The Key Patch: KimiClient for Graphiti

Graphiti uses  JSON schema by default. Moonshot API does not support it. I had to patch it:

json
{schema_str}


The fix is 20 lines. The debugging was 4 hours.

---

## Real Query Comparison

**Query:** "What infrastructure decisions did we make in February, and why?"

**Vector RAG result:** 5 chunks about infrastructure. No ordering. No causal relationships. No "why."

**Graph RAG result:**



The graph knows *why* the decision was made and *what* it connects to.

---

## OpenClaw Integration

The whole thing runs inside OpenClaw as a first-class  tool — called automatically before every response.



---

## What I Would Do Differently

1. **Define entity types upfront** — Graphiti auto-extracts but explicit schemas give cleaner traversals
2. **Use a strong LLM for extraction** — Qwen3:8b quality was noticeably worse than Kimi K2 Turbo
3. **Ingest daily, not in bulk** — bulk ingesting 3 months triggers rate limits and duplicate edges
4. **Plan for deduplication** — "Luminar" vs "luminar-labs" vs "LuminarLabs" become 3 separate entities

---

*Running on a gaming server: i5-9600K, 64GB RAM, RTX 2080 Ti. Neo4j browser at localhost:7474.*
