---
title: "Agentic Frameworks Deep Dive: pi-agent-core vs Google ADK vs AWS Strands vs CrewAI vs LangGraph vs Pydantic AI"
date: 2026-02-01
description: "A comprehensive comparison of major agentic frameworks examining session management, memory systems, webhooks, agent loops, and replay capabilities."
tags: ["AI Agents", "Frameworks", "Architecture", "LangGraph", "CrewAI", "AWS Strands", "Google ADK", "Pydantic AI", "OpenClaw"]
categories: ["Technical Deep Dive"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Building production AI agents requires choosing the right framework. This analysis examines **pi-agent-core** (OpenClaw's runtime), **Google ADK**, **AWS Strands**, **CrewAI**, **LangGraph**, and **Pydantic AI** across critical dimensions: sessions, memory, webhooks, agent loops, and replay support.
{{< /lead >}}

---

## Executive Summary

{{< alert icon="lightbulb" cardColor="#1e3a5f" iconColor="#60a5fa" textColor="#f0f0f0" >}}
**Key Finding:** All frameworks converge on similar patterns — the agent loop, tool calling, and state management — but differ significantly in their **abstraction level**, **deployment story**, and **enterprise readiness**.
{{< /alert >}}

| Framework | Primary Use Case | Abstraction Level | Session Support | Memory System |
|-----------|-----------------|-------------------|-----------------|---------------|
| **pi-agent-core** | Multi-channel orchestration | High | ⭐⭐⭐⭐⭐ Full | Workspace-based |
| **Google ADK** | GCP-native agents | Medium | ⭐⭐⭐⭐ Native | Session + Memory services |
| **AWS Strands** | AWS-native agents | Medium | ⭐⭐⭐⭐ Native | AgentCore Memory |
| **LangGraph** | Custom agent workflows | Low | ⭐⭐⭐⭐⭐ Full | Checkpointing |
| **CrewAI** | Multi-agent teams | High | ⭐⭐⭐ Basic | Short/Long-term + Entity |
| **Pydantic AI** | Type-safe agents | Medium | ⭐⭐⭐ Manual | Message history |

---

## The Frameworks at a Glance

{{< mermaid >}}
flowchart TB
    subgraph HIGH["High Abstraction"]
        OC[pi-agent-core/OpenClaw]
        CR[CrewAI]
    end
    
    subgraph MED["Medium Abstraction"]
        ADK[Google ADK]
        STR[AWS Strands]
        PYD[Pydantic AI]
    end
    
    subgraph LOW["Low Abstraction"]
        LG[LangGraph]
    end
    
    HIGH --> MED
    MED --> LOW
{{< /mermaid >}}

---

## 1. Session Management

### pi-agent-core (OpenClaw)

The most sophisticated session system with structured keys and comprehensive lifecycle management:

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

```javascript
// OpenClaw session lifecycle
session: {
  dmScope: "main",           // main | per-peer | per-channel-peer
  reset: {
    mode: "daily",
    atHour: 4,
    idleMinutes: 120
  }
}
```

### Google ADK

Clean separation of Session, State, and Memory:

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
- ✅ In-memory implementations for testing
- ✅ Cloud-based backends for production
- ⚠️ Simpler key structure than pi-agent-core

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

# Compile with checkpointer
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
- ⚠️ Requires manual session key design

### CrewAI

Implicit session management focused on crew executions:

```python
crew = Crew(
    agents=[...],
    tasks=[...],
    memory=True,  # Enables memory system
)

# Storage locations
# macOS: ~/Library/Application Support/CrewAI/{project}/
# Linux: ~/.local/share/CrewAI/{project}/
```

**Features:**
- ✅ Automatic storage location handling
- ✅ Project-scoped isolation
- ⚠️ Less explicit session control
- ⚠️ Crew-centric, not conversation-centric

### Pydantic AI

Manual message history management:

```python
result = agent.run_sync("Hello", message_history=previous_messages)

# Access messages
all_messages = result.all_messages()
new_messages = result.new_messages()
```

**Features:**
- ✅ Type-safe message handling
- ✅ Simple append-based continuation
- ⚠️ No built-in persistence
- ⚠️ Manual session tracking required

---

## 2. Memory Systems

{{< chart >}}
type: 'radar',
data: {
  labels: ['Short-term', 'Long-term', 'Entity', 'Semantic Search', 'Cross-session', 'Compaction'],
  datasets: [{
    label: 'pi-agent-core',
    data: [95, 90, 80, 95, 95, 95],
    fill: true,
    backgroundColor: 'rgba(34, 197, 94, 0.2)',
    borderColor: 'rgb(34, 197, 94)',
  }, {
    label: 'CrewAI',
    data: [90, 85, 90, 80, 75, 60],
    fill: true,
    backgroundColor: 'rgba(59, 130, 246, 0.2)',
    borderColor: 'rgb(59, 130, 246)',
  }, {
    label: 'LangGraph',
    data: [95, 80, 60, 70, 90, 85],
    fill: true,
    backgroundColor: 'rgba(168, 85, 247, 0.2)',
    borderColor: 'rgb(168, 85, 247)',
  }, {
    label: 'Google ADK',
    data: [85, 85, 70, 80, 85, 70],
    fill: true,
    backgroundColor: 'rgba(249, 115, 22, 0.2)',
    borderColor: 'rgb(249, 115, 22)',
  }]
},
options: {
  plugins: {
    title: { display: true, text: 'Memory Capabilities by Framework' }
  }
}
{{< /chart >}}

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
- ✅ Session transcript indexing (experimental)
- ✅ `memory_search` and `memory_get` tools

### CrewAI Memory

**Four-layer memory architecture:**

| Layer | Description | Storage |
|-------|-------------|---------|
| Short-Term | Current context (RAG) | ChromaDB |
| Long-Term | Past insights | SQLite |
| Entity | People/places/concepts | ChromaDB (RAG) |
| Contextual | Combined view | Aggregated |

```python
crew = Crew(
    memory=True,
    embedder={
        "provider": "ollama",
        "config": {"model": "mxbai-embed-large"}
    }
)
```

### LangGraph Memory

**Checkpoint-based with flexible backends:**

```python
# Short-term: within graph state
# Long-term: external store integration

from langgraph.store.memory import InMemoryStore
store = InMemoryStore()

# Store can persist across sessions
graph = workflow.compile(checkpointer=memory, store=store)
```

### Google ADK Memory

**Service-based architecture:**

```python
# MemoryService handles:
# - Ingesting from completed sessions
# - Cross-session search
# - Knowledge base management

memory_service = CloudMemoryService()
results = memory_service.search(query="previous decisions")
```

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

| Framework | Loop Style | Streaming | Tool Execution | Max Iterations |
|-----------|-----------|-----------|----------------|----------------|
| **pi-agent-core** | Event-driven | ✅ Full | Async queued | Configurable |
| **Google ADK** | Service-based | ✅ Async iterators | Concurrent/Sequential | Configurable |
| **AWS Strands** | Event loop | ✅ Callback handlers | Executor pattern | Configurable |
| **LangGraph** | Graph traversal | ✅ Native | Node execution | Graph structure |
| **CrewAI** | Task delegation | ✅ Verbose mode | Agent-based | Task completion |
| **Pydantic AI** | Request/response | ✅ run_stream | Sync/async | Result-based |

### pi-agent-core Loop Details

```
1. agent RPC validates, resolves session → returns {runId, acceptedAt}
2. agentCommand:
   - resolves model + thinking/verbose
   - loads skills snapshot
   - calls runEmbeddedPiAgent
3. runEmbeddedPiAgent:
   - serializes via per-session + global queues
   - subscribes to pi events
   - streams assistant/tool deltas
   - enforces timeout
4. Events emitted:
   - tool → start/update/end
   - assistant → text deltas
   - lifecycle → start/end/error
```

### AWS Strands Loop

```python
from strands import Agent

agent = Agent(
    model=bedrock_model,
    tools=[...],
    callback_handler=StreamHandler()  # For streaming
)

# Event loop handles:
# - Tool orchestration
# - State management
# - Streaming responses
result = agent("Process this request")
```

---

## 4. Webhooks & External Integration

### pi-agent-core (OpenClaw)

**Most comprehensive channel support:**

| Channel | Protocol | Real-time |
|---------|----------|-----------|
| WhatsApp | Baileys (WebSocket) | ✅ |
| Telegram | grammY (Long-poll/Webhook) | ✅ |
| Discord | discord.js (WebSocket) | ✅ |
| Slack | Bolt (Socket Mode) | ✅ |
| Signal | signal-cli (dbus) | ✅ |
| Webhook | HTTP POST | ✅ |

```javascript
// Gateway WebSocket protocol
{"type": "req", "id": "1", "method": "agent", "params": {...}}
{"type": "event", "event": "agent", "payload": {"stream": "assistant"}}
```

### AWS Strands

**Native AWS integration + external protocols:**

- AG-UI protocol support
- A2A (Agent-to-Agent) protocol
- Telegram integration (community)
- Teams integration (community)
- UTCP tool protocol

### Google ADK

**Multi-language webhook support:**

- Python/Go/Java/TypeScript SDKs
- HTTP endpoints
- gRPC support
- Cloud Run integration

### LangGraph

**LangServe for deployment:**

```python
from langserve import add_routes
add_routes(app, graph, path="/agent")

# Endpoints:
# POST /agent/invoke
# POST /agent/stream
# POST /agent/batch
```

---

## 5. Session Replay & Debugging

### pi-agent-core

**JSONL transcripts enable full replay:**

```bash
# Session transcripts stored as:
~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl

# Each line is a complete message event
{"role": "user", "content": "...", "timestamp": "..."}
{"role": "assistant", "content": "...", "tool_calls": [...]}
```

**Features:**
- ✅ Full conversation replay from JSONL
- ✅ Time-travel through session history
- ✅ Compaction summaries preserved
- ✅ Tool results in transcript

### LangGraph

**Best-in-class debugging with LangSmith:**

```python
# Time-travel debugging
for state in graph.get_state_history(config):
    print(state.values, state.next)

# Replay from any checkpoint
graph.update_state(config, new_values)
```

**Features:**
- ✅ Checkpoint-based replay
- ✅ State history traversal
- ✅ Human-in-the-loop interrupts
- ✅ LangSmith visualization

### AWS Strands

**Comprehensive observability:**

```python
# OpenTelemetry integration
from strands.telemetry import configure_tracing
configure_tracing(service_name="my-agent")

# Metrics, traces, logs
# → CloudWatch / X-Ray / Custom backends
```

**Features:**
- ✅ OTEL traces for replay analysis
- ✅ Strands Evals SDK for trajectory evaluation
- ✅ Goal success rate tracking
- ✅ Tool selection accuracy metrics

### CrewAI

**Task-based replay:**

```python
# Long-term memory stores task results
# Can be queried for past executions

from crewai.utilities.paths import db_storage_path
# SQLite: long_term_memory_storage.db
```

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

### Tool Definition Convergence

All frameworks use similar tool schemas:

```python
# Universal pattern
{
    "name": "get_weather",
    "description": "Get weather for a city",
    "parameters": {
        "type": "object",
        "properties": {
            "city": {"type": "string"}
        },
        "required": ["city"]
    }
}
```

### Multi-Agent Patterns

| Pattern | Strands | CrewAI | LangGraph | ADK |
|---------|---------|--------|-----------|-----|
| Agents as Tools | ✅ | ✅ | ✅ | ✅ |
| Swarm | ✅ | - | ✅ | - |
| Graph/DAG | ✅ | - | ✅ | - |
| Hierarchical | ✅ | ✅ | ✅ | ✅ |
| A2A Protocol | ✅ | - | - | - |

---

## 7. Framework Selection Guide

{{< mermaid >}}
flowchart TB
    START[Need an Agent Framework?] --> Q1{Multi-channel<br/>messaging?}
    Q1 -->|Yes| OC[pi-agent-core/OpenClaw]
    Q1 -->|No| Q2{AWS native?}
    Q2 -->|Yes| STR[AWS Strands]
    Q2 -->|No| Q3{GCP native?}
    Q3 -->|Yes| ADK[Google ADK]
    Q3 -->|No| Q4{Complex workflows?}
    Q4 -->|Yes| LG[LangGraph]
    Q4 -->|No| Q5{Multi-agent teams?}
    Q5 -->|Yes| CR[CrewAI]
    Q5 -->|No| PYD[Pydantic AI]
{{< /mermaid >}}

### Recommendations

| Use Case | Recommended Framework |
|----------|----------------------|
| Personal AI assistant (multi-channel) | pi-agent-core/OpenClaw |
| AWS enterprise deployment | AWS Strands + AgentCore |
| GCP enterprise deployment | Google ADK |
| Custom complex workflows | LangGraph |
| Autonomous research teams | CrewAI |
| Type-safe simple agents | Pydantic AI |
| Learning/prototyping | Pydantic AI or LangGraph |

---

## Conclusion

The agentic framework landscape is converging on common patterns while differentiating on:

1. **Abstraction level** — High (OpenClaw, CrewAI) vs Low (LangGraph)
2. **Cloud integration** — AWS (Strands), GCP (ADK), agnostic (others)
3. **Session sophistication** — Full lifecycle (OpenClaw, LangGraph) vs basic (Pydantic AI)
4. **Memory architecture** — Workspace files (OpenClaw), RAG layers (CrewAI), checkpoints (LangGraph)
5. **Multi-agent support** — Built-in (Strands, CrewAI) vs compose yourself (others)

**For production AI agents** that need multi-channel messaging, sophisticated session management, and workspace-based memory, **pi-agent-core (OpenClaw)** offers the most complete solution. For **AWS-native deployments**, **Strands** with AgentCore Runtime provides excellent integration. For **maximum control** over agent behavior, **LangGraph** remains the gold standard.

---

*This comparison reflects the state of these frameworks as of February 2026. The agentic AI space evolves rapidly.*

{{< badge >}}
Written by Amine El Farssi — Building production AI agents at KBC
{{< /badge >}}
