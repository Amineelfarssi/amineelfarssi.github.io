---
title: "Graph RAG in Practice: How I Wired Neo4j Into My AI Agent's Memory"
date: 2026-03-23
description: "Vector search finds similar text. Graph RAG finds connected facts. Here is how I built a hybrid retrieval system using Neo4j, Graphiti, and OpenClaw that actually remembers context across sessions."
tags: ["Graph RAG", "Neo4j", "Knowledge Graph", "OpenClaw", "Graphiti", "RAG", "AI Memory"]
categories: ["Engineering"]
showTableOfContents: false
showReadingTime: true
showWordCount: true
externalUrl: "/graph-rag/"
---

{{< lead >}}
Vector RAG retrieves documents. Graph RAG retrieves relationships. When your agent needs to reason across entities, timelines, and decisions, the graph wins.
{{< /lead >}}

This article is an interactive deep dive — animated pipeline diagrams, side-by-side comparisons, and real code from my setup.

**[Open the interactive article →](/graph-rag/)**

---

### What is covered

- Why vector memory alone breaks for agents with months of context
- The full stack: Neo4j + Graphiti + ChromaDB + RRF fusion
- The KimiClient patch that Graphiti needs for Moonshot API
- Real query: vector returns flat chunks, graph returns causal chains
- Wiring it all into OpenClaw as a first-class memory tool
- What I would do differently

**Stack:** Neo4j 5 · Graphiti v0.28.1 · ChromaDB · all-MiniLM-L6-v2 · Kimi K2 Turbo · OpenClaw
