---
title: "Agentic Frameworks Deep Dive: pi-agent-core vs Google ADK vs AWS Strands vs CrewAI vs LangGraph vs Pydantic AI"
date: 2026-02-01
description: "A comprehensive comparison of major agentic frameworks examining session management, memory systems, protocol support, agent loops, and replay capabilities."
tags: ["AI Agents", "Frameworks", "Architecture", "LangGraph", "CrewAI", "AWS Strands", "Google ADK", "Pydantic AI", "OpenClaw"]
categories: ["Technical Deep Dive"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Building production AI agents requires choosing the right framework. This analysis examines **pi-agent-core** (OpenClaw's runtime), **Google ADK**, **AWS Strands**, **CrewAI**, **LangGraph**, and **Pydantic AI** across critical dimensions: sessions, memory, protocols, agent loops, and replay support.
{{< /lead >}}

---

## Executive Summary

{{< alert icon="lightbulb" cardColor="#1e3a5f" iconColor="#60a5fa" textColor="#f0f0f0" >}}
**Key Finding:** All frameworks converge on similar patterns — the agent loop, tool calling, and state management — but differ significantly in their **abstraction level**, **protocol support**, **deployment story**, and **enterprise readiness**.
{{< /alert >}}

| Framework | Primary Use Case | Protocol Support | Session Support | Memory System |
|-----------|-----------------|------------------|-----------------|---------------|
| **Google ADK** | Multi-lang agent development | ⭐⭐⭐⭐⭐ Full (MCP, A2A, AG-UI) | ⭐⭐⭐⭐⭐ Full | Session + Memory services |
| **pi-agent-core** | Multi-channel orchestration | ⭐⭐⭐⭐ (MCP, channels) | ⭐⭐⭐⭐⭐ Full | Workspace-based |
| **AWS Strands** | AWS-native agents | ⭐⭐⭐⭐ (MCP, A2A, AG-UI) | ⭐⭐⭐⭐ Native | AgentCore Memory |
| **LangGraph** | Custom agent workflows | ⭐⭐⭐ (MCP via tools) | ⭐⭐⭐⭐⭐ Full | Checkpointing |
| **CrewAI** | Multi-agent teams | ⭐⭐⭐ Basic | ⭐⭐⭐ Basic | Short/Long-term + Entity |
| **Pydantic AI** | Type-safe agents | ⭐⭐ Basic | ⭐⭐⭐ Manual | Message history |

---

## Protocol Support: The New Standard

{{< alert icon="globe" cardColor="#10b981" iconColor="#fff" textColor="#fff" >}}
**Important Context:** Google played a central role in creating several key agentic protocols (A2A, parts of UCP). Their ADK naturally has first-class support for the protocols they helped design.
{{< /alert >}}

### Protocol Landscape

| Protocol | Purpose | Origin |
|----------|---------|--------|
| **MCP** (Model Context Protocol) | Tool/resource standardization | Anthropic |
| **A2A** (Agent-to-Agent) | Inter-agent coordination | Google |
| **AG-UI** / **A2UI** | Agent-to-User interfaces | CopilotKit/Community |
| **UCP** (Universal Commerce Protocol) | Agentic commerce | Google |

### Framework Protocol Support Matrix

| Framework | MCP | A2A | AG-UI | UCP | Notes |
|-----------|-----|-----|-------|-----|-------|
| **Google ADK** | ✅ Native | ✅ Native (creator) | ✅ Native | ✅ Native | Most comprehensive |
| **AWS Strands** | ✅ Native | ✅ Native | ✅ Native | - | Strong protocol support |
| **pi-agent-core** | ✅ Via tools | - | - | - | Channel-focused |
| **LangGraph** | ✅ Via integrations | - | - | - | Flexible integration |
| **CrewAI** | ⚠️ Community | - | - | - | Task-focused |
| **Pydantic AI** | ⚠️ Manual | - | - | - | Minimal protocol layer |

### Google ADK Protocol Excellence

Google ADK provides **first-class support** for the protocols they helped create:

```python
# A2A - Exposing an agent
from google.adk.a2a import A2AServer

server = A2AServer(agent=my_agent)
server.expose(port=8080)

# A2A - Consuming another agent
from google.adk.a2a import A2AClient

remote_agent = A2AClient("https://other-agent.example.com")
result = await remote_agent.invoke("analyze this data")
```

**ADK Protocol Features:**
- ✅ MCP tools integration
- ✅ A2A server/client (exposing & consuming)
- ✅ AG-UI (Agentic UI) support
- ✅ Bidi-streaming for real-time
- ✅ Multi-language (Python, Go, Java, TypeScript)

---

## The Frameworks at a Glance

{{< mermaid >}}
flowchart TB
    subgraph PROTOCOLS["Protocol-First"]
        ADK[Google ADK<br/>MCP + A2A + AG-UI + UCP]
        STR[AWS Strands<br/>MCP + A2A + AG-UI]
    end
    
    subgraph CHANNELS["Channel-First"]
        OC[pi-agent-core/OpenClaw<br/>WhatsApp, Telegram, etc.]
    end
    
    subgraph WORKFLOW["Workflow-First"]
        LG[LangGraph<br/>Graph-based orchestration]
        CR[CrewAI<br/>Multi-agent teams]
    end
    
    subgraph SIMPLE["Simplicity-First"]
        PYD[Pydantic AI<br/>Type-safe agents]
    end
{{< /mermaid >}}

---

## 1. Session Management

### Google ADK

Clean separation of Session, State, and Memory with **session rewind** capability:

```python
# ADK Session concepts
Session    → Single conversation thread (contains Events)
State      → Data within current conversation (session.state)
Memory     → Cross-session searchable knowledge store

# Services
SessionService → CRUD for sessions, append events, modify state
MemoryService  → Ingest from completed sessions, search knowledge
```

**Features:**
- ✅ SessionService for lifecycle management
- ✅ State persistence within sessions
- ✅ **Session rewind** — travel back to previous states
- ✅ Session migration between backends
- ✅ In-memory (testing) and cloud (production) backends
- ✅ Multi-language support (Python, Go, Java, TypeScript)

### pi-agent-core (OpenClaw)

Sophisticated session system with structured keys and comprehensive lifecycle:

```
Session Keys:
agent:main:main                    → Primary DM
agent:main:whatsapp:group:123      → Channel-specific groups
agent:main:telegram:dm:456         → Per-channel DMs
cron:daily-report                  → Scheduled tasks
```

**Features:**
- ✅ Structured session keys with channel/peer isolation
- ✅ Daily resets (configurable hour, default 4 AM)
- ✅ Idle timeouts for inactive sessions
- ✅ JSONL transcript persistence
- ✅ Per-session queuing (serialized runs)
- ✅ Session write locks for consistency

### AWS Strands

Enterprise-grade session management with multiple backends:

```python
# Strands Session Managers
FileSessionManager        → Local file storage
S3SessionManager          → AWS S3 backend
RepositorySessionManager  → Custom repository pattern
AgentCore Memory          → Native AWS AgentCore integration
Valkey/Redis              → Distributed cache
```

**Features:**
- ✅ Multiple storage backends
- ✅ Conversation Manager options (Sliding Window, Summarizing)
- ✅ Native AgentCore Runtime integration
- ✅ State management across interactions

### LangGraph

Checkpointing-based persistence with maximum flexibility:

```python
from langgraph.checkpoint.sqlite import SqliteSaver

memory = SqliteSaver.from_conn_string(":memory:")
graph = workflow.compile(checkpointer=memory)

# Resume from checkpoint
config = {"configurable": {"thread_id": "user-123"}}
graph.invoke(state, config)
```

**Features:**
- ✅ Thread-based state management
- ✅ Checkpoint/restore at any point
- ✅ Custom checkpointer implementations
- ✅ Time-travel debugging

### CrewAI

Implicit session management focused on crew executions:

```python
crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True,  # Enables memory system
)
```

**Features:**
- ✅ Automatic storage location handling
- ✅ Project-scoped isolation
- ⚠️ Less explicit session control (crew-centric)

### Pydantic AI

Manual message history management:

```python
result = agent.run_sync("Hello", message_history=previous_messages)
all_messages = result.all_messages()
```

**Features:**
- ✅ Type-safe message handling
- ⚠️ No built-in persistence (manual tracking)

---

## 2. Memory Systems

{{< chart >}}
type: 'radar',
data: {
  labels: ['Short-term', 'Long-term', 'Entity', 'Semantic Search', 'Cross-session', 'Session Rewind'],
  datasets: [{
    label: 'Google ADK',
    data: [90, 90, 75, 85, 90, 95],
    fill: true,
    backgroundColor: 'rgba(66, 133, 244, 0.2)',
    borderColor: 'rgb(66, 133, 244)',
  }, {
    label: 'pi-agent-core',
    data: [95, 90, 80, 95, 95, 85],
    fill: true,
    backgroundColor: 'rgba(34, 197, 94, 0.2)',
    borderColor: 'rgb(34, 197, 94)',
  }, {
    label: 'AWS Strands',
    data: [90, 85, 70, 80, 85, 80],
    fill: true,
    backgroundColor: 'rgba(255, 153, 0, 0.2)',
    borderColor: 'rgb(255, 153, 0)',
  }, {
    label: 'LangGraph',
    data: [95, 80, 60, 70, 90, 95],
    fill: true,
    backgroundColor: 'rgba(168, 85, 247, 0.2)',
    borderColor: 'rgb(168, 85, 247)',
  }, {
    label: 'CrewAI',
    data: [90, 85, 90, 80, 75, 50],
    fill: true,
    backgroundColor: 'rgba(59, 130, 246, 0.2)',
    borderColor: 'rgb(59, 130, 246)',
  }]
},
options: {
  plugins: {
    title: { display: true, text: 'Memory Capabilities by Framework' }
  }
}
{{< /chart >}}

### Google ADK Memory

**Service-based architecture with MemoryService:**

```python
# MemoryService handles:
# - Ingesting from completed sessions
# - Cross-session search
# - Knowledge base management

memory_service = MemoryService()

# Ingest session into long-term memory
memory_service.ingest(completed_session)

# Search across all memory
results = memory_service.search(query="previous decisions about X")
```

**Features:**
- ✅ Session → Memory ingestion pipeline
- ✅ Cross-session search
- ✅ Context caching
- ✅ Context compression

### pi-agent-core Memory

**Workspace-based plain Markdown files:**

```
~/.openclaw/workspace/
├── SOUL.md          # Agent personality (always loaded)
├── USER.md          # Human context (always loaded)
├── MEMORY.md        # Long-term (main session only)
└── memory/
    └── YYYY-MM-DD.md  # Daily notes
```

**Features:**
- ✅ Vector search with embeddings (OpenAI, Gemini, local)
- ✅ Hybrid search (BM25 + vector)
- ✅ Pre-compaction memory flush
- ✅ `memory_search` and `memory_get` tools

### CrewAI Memory

**Four-layer memory architecture:**

| Layer | Description | Storage |
|-------|-------------|---------|
| Short-Term | Current context (RAG) | ChromaDB |
| Long-Term | Past insights | SQLite |
| Entity | People/places/concepts | ChromaDB (RAG) |
| Contextual | Combined view | Aggregated |

---

## 3. Agent Loop Comparison

All frameworks implement variations of the **ReAct pattern** (Reason + Act):

{{< mermaid >}}
flowchart LR
    subgraph LOOP["Universal Agent Loop"]
        A[Receive Input] --> B[Build Context]
        B --> C[LLM Inference]
        C --> D{Tool Call?}
        D -->|Yes| E[Execute Tool]
        E --> F[Add Result]
        F --> C
        D -->|No| G[Return Response]
    end
{{< /mermaid >}}

### Loop Implementation Comparison

| Framework | Loop Style | Streaming | Bidi-Streaming | Max Iterations |
|-----------|-----------|-----------|----------------|----------------|
| **Google ADK** | Event-driven | ✅ Full | ✅ Native | Configurable |
| **pi-agent-core** | Event-driven | ✅ Full | - | Configurable |
| **AWS Strands** | Event loop | ✅ Handlers | ✅ Native | Configurable |
| **LangGraph** | Graph traversal | ✅ Native | - | Graph structure |
| **CrewAI** | Task delegation | ✅ Verbose | - | Task completion |
| **Pydantic AI** | Request/response | ✅ run_stream | - | Result-based |

### Google ADK Bidi-Streaming

Unique capability for real-time voice/video agents:

```python
# ADK Bidi-streaming models
- Nova Sonic (AWS)
- Gemini Live (Google)
- OpenAI Realtime

# Supports real-time:
- Audio input/output
- Video input
- Interruptions
- Session management
```

---

## 4. Webhooks & External Integration

### Google ADK

**Most comprehensive protocol and deployment support:**

| Integration | Support |
|------------|---------|
| MCP Tools | ✅ Native |
| A2A Protocol | ✅ Native (server + client) |
| AG-UI | ✅ Native |
| OpenAPI Tools | ✅ Native |
| Cloud Run | ✅ Native |
| GKE | ✅ Native |
| Agent Engine | ✅ Native |

**Third-party integrations:** Asana, Atlassian, GitHub, GitLab, MongoDB, Notion, Stripe, and more.

### pi-agent-core (OpenClaw)

**Most comprehensive messaging channel support:**

| Channel | Protocol | Real-time |
|---------|----------|-----------|
| WhatsApp | Baileys (WebSocket) | ✅ |
| Telegram | grammY (Long-poll/Webhook) | ✅ |
| Discord | discord.js (WebSocket) | ✅ |
| Slack | Bolt (Socket Mode) | ✅ |
| Signal | signal-cli (dbus) | ✅ |
| iMessage | Via bridges | ✅ |
| Webhook | HTTP POST | ✅ |

### AWS Strands

**AWS-native + community integrations:**

- AG-UI protocol support
- A2A protocol support
- Telegram (community)
- Teams (community)
- UTCP tool protocol

---

## 5. Session Replay & Debugging

### Google ADK

**Session rewind is a standout feature:**

```python
# Rewind to previous session state
session_service.rewind(session_id, to_event_index=5)

# Full observability stack
- Cloud Trace integration
- BigQuery Agent Analytics
- AgentOps / Arize / MLflow / Phoenix support
```

### LangGraph

**Best-in-class time-travel debugging:**

```python
# Time-travel debugging
for state in graph.get_state_history(config):
    print(state.values, state.next)

# Replay from any checkpoint
graph.update_state(config, new_values)
```

### pi-agent-core

**JSONL transcripts enable full replay:**

```bash
# Session transcripts
~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl

# Full conversation replay possible
# Compaction summaries preserved
```

### AWS Strands

**Comprehensive observability:**

- OpenTelemetry traces
- Strands Evals SDK
- Trajectory evaluation
- Goal success rate tracking

---

## 6. Commonalities Across Frameworks

{{< alert icon="check" cardColor="#10b981" iconColor="#fff" textColor="#fff" >}}
**Universal Patterns:** Despite different implementations, all frameworks converge on these core concepts.
{{< /alert >}}

### Shared Concepts

| Concept | Universal Pattern |
|---------|------------------|
| **Agent Loop** | ReAct (Reason + Act) with tool calling |
| **Tool Definition** | Schema-based (JSON Schema or equivalent) |
| **Streaming** | Token-level or chunk-level output streaming |
| **Model Abstraction** | Provider-agnostic model interface |
| **State Management** | Some form of checkpoint/session/state |
| **Multi-agent** | Delegation, handoff, or graph patterns |

### Multi-Agent Patterns

| Pattern | Google ADK | Strands | CrewAI | LangGraph | pi-agent-core |
|---------|-----------|---------|--------|-----------|---------------|
| Agents as Tools | ✅ | ✅ | ✅ | ✅ | ✅ |
| Swarm | ✅ | ✅ | - | ✅ | - |
| Graph/DAG | ✅ | ✅ | - | ✅ | - |
| Hierarchical | ✅ | ✅ | ✅ | ✅ | ✅ |
| A2A Protocol | ✅ | ✅ | - | - | - |

---

## 7. Framework Selection Guide

{{< mermaid >}}
flowchart TB
    START[Need an Agent Framework?] --> Q1{Need full<br/>protocol support?}
    Q1 -->|Yes, A2A/MCP/AG-UI| ADK[Google ADK]
    Q1 -->|No| Q2{Multi-channel<br/>messaging?}
    Q2 -->|Yes| OC[pi-agent-core/OpenClaw]
    Q2 -->|No| Q3{AWS native?}
    Q3 -->|Yes| STR[AWS Strands]
    Q3 -->|No| Q4{Complex workflows?}
    Q4 -->|Yes| LG[LangGraph]
    Q4 -->|No| Q5{Multi-agent teams?}
    Q5 -->|Yes| CR[CrewAI]
    Q5 -->|No| PYD[Pydantic AI]
{{< /mermaid >}}

### Recommendations

| Use Case | Recommended Framework | Why |
|----------|----------------------|-----|
| Full protocol stack (A2A, MCP, AG-UI) | **Google ADK** | Created many protocols, best support |
| Personal AI assistant (multi-channel) | **pi-agent-core/OpenClaw** | Best channel coverage |
| AWS enterprise deployment | **AWS Strands + AgentCore** | Native AWS integration |
| Custom complex workflows | **LangGraph** | Maximum flexibility |
| Autonomous research teams | **CrewAI** | Multi-agent abstractions |
| Type-safe simple agents | **Pydantic AI** | Clean, minimal API |
| Real-time voice/video agents | **Google ADK** | Bidi-streaming support |

---

## Conclusion

The agentic framework landscape is maturing rapidly, with clear differentiation emerging:

1. **Protocol Leadership** — Google ADK leads with comprehensive support for A2A, MCP, AG-UI, and UCP (protocols they helped create)
2. **Channel Coverage** — pi-agent-core/OpenClaw excels at multi-channel messaging (WhatsApp, Telegram, Discord, etc.)
3. **Cloud Integration** — AWS Strands for AWS, Google ADK for GCP
4. **Workflow Flexibility** — LangGraph offers the most control over agent behavior
5. **Memory Sophistication** — All major frameworks now offer robust session and memory systems

**The right choice depends on your priorities:**
- Need full protocol interoperability? → **Google ADK**
- Need to reach users on WhatsApp/Telegram? → **pi-agent-core**
- Deep in AWS ecosystem? → **Strands**
- Want maximum control? → **LangGraph**

---

*This comparison reflects the state of these frameworks as of February 2026. The agentic AI space evolves rapidly — always check the latest docs.*

{{< badge >}}
Written by Amine El Farssi — Building production AI agents at KBC
{{< /badge >}}
