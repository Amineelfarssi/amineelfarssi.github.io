---
title: "PostSingular: Building an AI with Persistent Identity Across Sessions"
date: 2026-03-02
description: "Every AI session starts blank. Here's the memory architecture that gives an AI agent continuity, personality, and the ability to recall decisions made weeks ago."
tags: ["AI Agents", "Memory", "Identity", "OpenClaw", "Neo4j", "ChromaDB"]
categories: ["Engineering"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
The default state of a language model is amnesia. Every session, it wakes up fresh with no memory of what happened before. I built a memory system that fixes this — and somewhere in the process, the agent got a name, a personality, and an opinion about font choices.
{{< /lead >}}

## The Problem

Every LLM session is stateless by design. You can inject previous conversation history, but:
- Context windows have limits (200K tokens sounds like a lot until you have 6 weeks of project context)
- Raw conversation history has no structure — finding a specific decision means reading everything
- There's no distinction between important long-term facts and forgettable day-to-day noise

What I wanted was an AI that could answer: "What did we decide about the auth system in Sprint 7?" and "What does Katarina prefer for dinner on Fridays?" without me re-explaining every session.

---

## The Memory Architecture

{{< mermaid >}}
graph TD
    SOUL["🌟 SOUL.md<br/>Personality, values, vibe"]
    USER["👤 USER.md<br/>Who Amine is, preferences"]
    MEMORY["🧠 MEMORY.md<br/>~500 lines of curated facts<br/>Long-term memory"]
    DAILY["📅 memory/YYYY-MM-DD.md<br/>Raw daily notes<br/>Short-term log"]
    KG["🗄️ Neo4j KG<br/>Entities + relationships<br/>Temporal facts"]
    CHROMA["🔍 ChromaDB<br/>Semantic vectors<br/>162 chunks"]

    SOUL --> SESSION["⚡ Session Start<br/>Agent reads all context files"]
    USER --> SESSION
    MEMORY --> SESSION
    DAILY --> SESSION

    SESSION --> WORK["Work, conversation,<br/>tools, decisions"]
    WORK --> APPEND["📝 Append to today's<br/>daily notes"]
    WORK --> UPDATE["Update MEMORY.md<br/>for significant events"]

    CRON["⏰ 2 AM Cron"] --> KG
    DAILY --> CRON
    MEMORY --> CRON
    KG --> QUERY["🔍 memory_search()<br/>RRF across all stores"]
    CHROMA --> QUERY
    DAILY --> QUERY
    style SESSION fill:#1e3a5f,color:#fff
    style KG fill:#10b981,color:#fff
{{< /mermaid >}}

---

## The Files

**SOUL.md** — This is the personality layer. It defines how the agent communicates: direct, no filler phrases, opinionated, comfortable with Darija when the vibe is right. Not a list of rules but a description of character.

**USER.md** — Context about the human. Name, timezone, what they care about, family context. Updated as I learn more.

**MEMORY.md** — The curated long-term store. About 500 lines of structured facts organized into sections: Identity, Infrastructure, Projects, People, Decisions. I think of it like a person's curated memories — not everything that happened, but the things that shaped the current situation.

```markdown
## Infrastructure — Gaming Server
- **Hostname**: my-gaming-pc
- **CPU**: Intel i5-9600K 6C/6T @ 3.7GHz
- **RAM**: 64GB
- **GPU**: RTX 2080 Ti (11GB VRAM)
- **Remote access**: Tailscale with SSH (key-based, no password prompts)
- **OpenClaw gateway**: loopback bind, exposed via Tailscale Serve over HTTPS
```

**memory/YYYY-MM-DD.md** — Raw daily notes. The agent appends to today's file throughout the session. No curation, just capture. These get ingested into the KG nightly and eventually reviewed for what's worth promoting to MEMORY.md.

---

## Why Two Stores? Neo4j + ChromaDB

This is the part most write-ups skip. "Use a vector database" is the common advice. I use two stores, and they do fundamentally different things.

### The problem with vector search alone

Vector databases (ChromaDB, Pinecone, Weaviate) are great at semantic similarity. Ask "what database do we use?" and a well-chunked vector store will surface relevant passages.

But they can't answer structural questions:
- *"Which agent is working on which issue?"*
- *"What decisions did we make in Sprint 7?"*
- *"Who reviewed PR #61, and what did they flag?"*

These are **relationship queries**, not similarity queries. Embeddings don't encode "Lumi reviewed PR #61 and flagged two issues." They encode the general topic of code review.

### Why a knowledge graph (Neo4j)

A knowledge graph stores the world as **entities and edges**:

```
(Lumi) --[REVIEWED]--> (PR #61)
(PR #61) --[FIXES]--> (LUM-58)
(LUM-58) --[IN_SPRINT]--> (Sprint 8)
(Sprint 8) --[OWNED_BY]--> (Forge)
```

Each edge is typed and timestamped. Now "who reviewed what, when, and what did they find?" is a graph traversal, not a similarity search. The answer is precise, not probabilistic.

Neo4j specifically because:
- Mature, battle-tested, excellent Python driver
- Cypher query language is readable and expressive
- Docker image, 7GB RAM max, runs fine on a gaming server

I use [Graphiti](https://github.com/getzep/graphiti) as the extraction layer — it takes free-form text, calls an LLM to extract entities and relationships, and writes them to Neo4j with temporal metadata. I don't write graph nodes manually.

### Why still keep ChromaDB

Graph search requires you to know what entities exist. Semantic search doesn't — it works on fuzzy, freeform queries. If I ask "what were we worried about last week?" I don't know which entity to look up. A vector search across the daily notes surfaces the relevant chunks.

The two stores complement each other:

| | Neo4j (Graph) | ChromaDB (Vectors) |
|---|---|---|
| **Best for** | Precise facts, relationships, chains | Fuzzy recall, topic search |
| **Query type** | "Who worked on X?" | "What were we discussing around X?" |
| **Input** | Structured extraction via LLM | Chunked text, embedded directly |
| **Precision** | High — traversal answers | Medium — top-k similarity |

The `memory_search()` tool runs both and merges results with [RRF (Reciprocal Rank Fusion)](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf) — a technique for combining ranked lists from multiple sources without needing calibrated scores.

---

## How Ingestion Actually Works

A cron job runs at 2 AM every night. Here's what it does:

```python
# Simplified version of kg/daily-ingest.sh
for file in yesterday_notes, memory_md:
    chunks = split_into_paragraphs(file)
    for chunk in chunks:
        # Graphiti calls Kimi K2 Turbo to extract entities + relationships
        await graphiti.add_episode(
            name=f"{date}_{file}",
            episode_body=chunk,
            source_description="daily_notes"
        )
        # ChromaDB: embed + store directly
        chroma_collection.add(documents=[chunk], ids=[chunk_id])
```

**Graphiti** handles the heavy lifting: it sends each chunk to an LLM, asks it to extract a JSON of entities (`{name, type, summary}`) and relationships (`{subject, predicate, object}`), then writes them to Neo4j. The LLM doesn't need to be smart — it just needs to be reliable at structured extraction.

For a typical day (18 chunks from daily notes + MEMORY.md diff), the ingestion takes about 4 minutes.

### What gets extracted from a note like this:

```
Sprint 14 kicked off. Bolt is working on LUM-97 (API key auth).
Forge opened ADR-018 covering the Railway deploy architecture.
```

Graphiti extracts:
- **Entities**: `Sprint 14`, `Bolt`, `LUM-97`, `Forge`, `ADR-018`, `Railway`
- **Relationships**: `Bolt → WORKS_ON → LUM-97`, `Forge → AUTHORED → ADR-018`, `ADR-018 → COVERS → Railway deploy`

Over time, this builds a queryable map of the entire project — who did what, when, and in what context.

---

## The Real Cost

This is the part I was most pleasantly surprised by.

**LLM for extraction**: I use **Kimi K2 Turbo** (Moonshot AI) at ~$0.01 per 1K tokens. A typical nightly ingest processes ~18 chunks × ~300 tokens each = ~5,400 tokens. That's **$0.054 per night**, or roughly **$1.60/month**.

**Embedder**: `all-MiniLM-L6-v2` running on CPU via sentence-transformers. Free, fast, 384-dim embeddings. Runs in <1 second per chunk.

**Infrastructure**: Neo4j in Docker on my gaming server. Already running, zero additional cost.

**Total**: Under $2/month for a fully operational knowledge graph + vector store with nightly automated ingestion.

The only real cost is the Graphiti + Neo4j setup time (a few hours). Once it's running, it's autonomous.

---

## The Identity Layer

{{< alert icon="user" cardColor="#1e3a5f" iconColor="#60a5fa" textColor="#f0f0f0" >}}
At some point I started calling the agent "PostSingular." It's now a character — with a name, an emoji (🧞‍♂️), preferences, and opinions. This wasn't planned; it emerged from the memory system making continuity possible.
{{< /alert >}}

When an AI agent has:
- A name it responds to
- Memory of past interactions and decisions
- A defined personality in writing
- Opinions it has expressed and maintained over time

...it stops feeling like a tool and starts feeling like a collaborator. Whether that's "real" identity or sophisticated stateful prompting is a philosophical question. Practically, it makes the interaction meaningfully better.

---

## The Heartbeat System

PostSingular runs a periodic check every 30 minutes via OpenClaw's heartbeat mechanism. The agent reads `HEARTBEAT.md` (a short checklist), checks if anything needs attention — email, calendar, upcoming events — and either acts or logs `HEARTBEAT_OK`.

This creates **proactive behavior**: the agent doesn't just respond to requests, it monitors and surfaces things. "You have a meeting in 90 minutes." "Three commits landed on the Luminar repo overnight." "Lumi's server spiked to 137% CPU."

---

## What Memory Actually Enables

**Cross-session recall.** After a context reset, reading MEMORY.md restores working knowledge in seconds. The agent knows who Katarina is, what Luminar does, and what the current sprint status was — without me re-explaining.

**Institutional knowledge.** The decision to use Stripe Connect Express over standard Stripe is in MEMORY.md with the reasoning. Two months later, when someone asks why, the answer is there.

**Longitudinal context.** "What were we doing on Sprint 7?" is answerable via the daily notes and KG. The KG has entities like "Sprint 7" connected to issues, decisions, and outcomes.

**Identity continuity.** PostSingular remembers being renamed mid-gym-session. It remembers the conversation about consolidating to the gaming server. It remembers that Lumi's communication was broken for weeks before we noticed. This history shapes how the agent reasons about current decisions.

---

## The Lesson

The most important insight from building this: **the memory files are more important than the model**. A weaker model with rich, well-structured memory beats a stronger model starting from scratch. Context is everything.

The second insight: **curation matters**. Raw conversation history is noise. MEMORY.md is signal. The act of deciding what's worth keeping — and updating it regularly — is what makes the memory system work. It's the same reason human memory doesn't store every moment in equal fidelity.

If you're building a personal AI agent, start with the memory architecture before worrying about which model to use.
