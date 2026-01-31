---
title: "Agentic AI Skills"
description: "Deep expertise in building production AI agents and autonomous systems"
showDate: false
showReadingTime: false
showWordCount: false
showAuthor: false
sharingLinks: false
showTableOfContents: true
---

{{< lead >}}
Building AI that doesn't just think — it **acts**. From enterprise agents at scale to open-source frameworks.
{{< /lead >}}

---

## The Agentic Stack

I work across the full agentic AI stack, from foundation models to production deployment:

{{< mermaid >}}
flowchart TB
    subgraph LAYER4["🎨 Presentation Layer"]
        AGUI[AG-UI Protocol]
        A2UI[A2UI Blueprints]
    end
    
    subgraph LAYER3["🤝 Coordination Layer"]
        A2A[A2A Protocol]
        MULTI[Multi-Agent Orchestration]
    end
    
    subgraph LAYER2["🔧 Tools & Data Layer"]
        MCP[MCP Protocol]
        RAG[RAG Systems]
        TOOLS[Tool Integration]
    end
    
    subgraph LAYER1["🧠 Foundation Layer"]
        LLM[Foundation Models]
        EMB[Embeddings]
    end
    
    LAYER4 --> LAYER3
    LAYER3 --> LAYER2
    LAYER2 --> LAYER1
{{< /mermaid >}}

---

## Core Competencies

### 🤖 Agent Development

{{< alert icon="robot" cardColor="#1e3a5f" iconColor="#60a5fa" textColor="#f0f0f0" >}}
**Production Experience:** Building enterprise AI agents at Belgium's largest bank
{{< /alert >}}

| Skill | Tools & Frameworks | Experience |
|-------|-------------------|------------|
| Agent Orchestration | LangGraph, AWS Bedrock Agents, CrewAI | Production |
| Tool Use & Function Calling | OpenAI, Claude, Bedrock | Production |
| Memory & State Management | Session stores, vector DBs, compaction | Production |
| Multi-Agent Systems | A2A protocol, hierarchical agents | Advanced |
| Agent Evaluation | AgentOps, LangSmith, custom evals | Production |

### 🔗 Agentic Protocols

Deep understanding of the emerging protocol stack:

**MCP (Model Context Protocol)**
- Tool and resource standardization
- Server/client architecture
- Enterprise security patterns

**A2A (Agent-to-Agent)**
- Capability discovery
- Task delegation and lifecycle
- Cross-agent coordination

**AG-UI / A2UI**
- Declarative UI blueprints
- Safe agent-to-user interfaces
- Event-based streaming

### 🏗️ Agent Infrastructure

```
┌─────────────────────────────────────────────────────────┐
│                  Production Agent Stack                  │
├─────────────────────────────────────────────────────────┤
│  Observability    │ AgentOps, LangSmith, CloudWatch     │
│  Guardrails       │ Bedrock Guardrails, custom filters  │
│  Evaluation       │ LLM-as-judge, automated evals       │
│  Deployment       │ Lambda, ECS, SageMaker endpoints    │
│  State Management │ DynamoDB, Redis, JSONL transcripts  │
│  Vector Storage   │ OpenSearch, Pinecone, pgvector      │
└─────────────────────────────────────────────────────────┘
```

---

## Framework Proficiency

### Cloud Platforms

{{< badge >}}AWS Bedrock{{< /badge >}}
{{< badge >}}AWS SageMaker{{< /badge >}}
{{< badge >}}Azure OpenAI{{< /badge >}}
{{< badge >}}Google Vertex AI{{< /badge >}}

### Agent Frameworks

{{< badge >}}LangChain{{< /badge >}}
{{< badge >}}LangGraph{{< /badge >}}
{{< badge >}}AWS AgentCore Runtime{{< /badge >}}
{{< badge >}}CrewAI{{< /badge >}}
{{< badge >}}AWS Strands{{< /badge >}}
{{< badge >}}Google ADK{{< /badge >}}
{{< badge >}}AutoGen{{< /badge >}}

### Foundation Models

{{< badge >}}Claude{{< /badge >}}
{{< badge >}}OpenAI{{< /badge >}}
{{< badge >}}Gemini{{< /badge >}}
{{< badge >}}Llama{{< /badge >}}
{{< badge >}}Mistral{{< /badge >}}

---

## Real-World Applications

### Enterprise AI Agents
Building agents that handle real banking workflows:
- Document processing and extraction
- Customer query routing
- Compliance checking
- Internal knowledge retrieval

### RAG-Enhanced Agents
Combining retrieval with reasoning:
- Hybrid search (semantic + keyword)
- Chunking strategies for domain documents
- Citation and source tracking
- Incremental index updates

### Agentic Evaluation
Ensuring agents work reliably:
- Trajectory evaluation
- Tool use accuracy
- Hallucination detection
- Latency optimization

---

## Open Source Contributions

### MiniClaw
A minimal agent orchestration framework in Python demonstrating core patterns:
- OpenClaw-style session management
- Workspace-based memory (SOUL.md, MEMORY.md)
- Multi-provider support (OpenAI, Anthropic, Ollama)
- ~2,800 lines of readable, educational code

{{< button href="/projects/miniclaw" >}}
View Project
{{< /button >}}

---

## Certifications & Training

- AWS Certified Solutions Architect
- Deep Learning Specialization (Coursera)
- MLOps specialization (ongoing)

---

{{< alert icon="graduation-cap" >}}
**Want to discuss agentic AI?** I'm always happy to chat about agent architectures, production challenges, or collaboration opportunities.
{{< /alert >}}
