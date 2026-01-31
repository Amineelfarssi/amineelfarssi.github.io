---
title: "AI Engineer"
description: "Building production AI agents at KBC Bank & Insurance"
date: 2023-01-01
tags: ["AI Agents", "AWS Bedrock", "LangChain", "RAG", "AgentOps"]
weight: 1
---

{{< badge >}}KBC Bank & Insurance{{< /badge >}}
{{< badge >}}Leuven, Belgium{{< /badge >}}
{{< badge >}}2023 - Present{{< /badge >}}

## Overview

Building the bank's first production AI agents — from architecture design to deployment and monitoring. Active participant in architecture design boards, driving technical decisions for enterprise-scale agentic systems.

{{< alert icon="building" cardColor="#1e3a5f" iconColor="#60a5fa" textColor="#f0f0f0" >}}
**Scale:** One of Belgium's largest banks with 12M+ customers
{{< /alert >}}

## Key Achievements

### 🤖 Production AI Agents
- **First internal AI agents** deployed to production using AWS Bedrock and custom AgentCore framework
- **Multi-agent orchestration** for complex banking workflows (document processing, customer routing, compliance)
- **Tool integration** with internal banking APIs, document stores, and knowledge bases
- **Memory management** with session persistence and context windowing

### 🏗️ Architecture Leadership
- Active participant in **architecture design boards** — driving technical decisions for AI infrastructure
- Designed **event-driven agent pipelines** using EventBridge, SQS, and Lambda
- Established **guardrails and safety patterns** using Bedrock Guardrails
- Built **evaluation frameworks** for continuous agent quality monitoring

### 📚 RAG Systems
- Enterprise-scale **Retrieval-Augmented Generation** for internal knowledge management
- **Hybrid search** combining semantic embeddings with keyword matching
- **Chunking strategies** optimized for banking documents (contracts, policies, procedures)
- **Incremental indexing** for real-time updates to knowledge base

### 📊 AgentOps & Observability
- Implemented **agent tracing** and **trajectory evaluation** for debugging complex agent behavior
- **LLM-as-judge evaluation** for response quality and hallucination detection
- **Cost tracking** and optimization for foundation model usage
- **Latency monitoring** and performance optimization

## Technical Stack

### Agent Infrastructure
{{< badge >}}AWS Bedrock{{< /badge >}}
{{< badge >}}LangChain{{< /badge >}}
{{< badge >}}LangGraph{{< /badge >}}
{{< badge >}}AgentCore{{< /badge >}}

### Cloud & DevOps
{{< badge >}}AWS Lambda{{< /badge >}}
{{< badge >}}EventBridge{{< /badge >}}
{{< badge >}}SQS{{< /badge >}}
{{< badge >}}DynamoDB{{< /badge >}}
{{< badge >}}SageMaker{{< /badge >}}

### Evaluation & Monitoring
{{< badge >}}AgentOps{{< /badge >}}
{{< badge >}}CloudWatch{{< /badge >}}
{{< badge >}}Custom Evals{{< /badge >}}

### Foundation Models
{{< badge >}}Claude 3.5{{< /badge >}}
{{< badge >}}GPT-4{{< /badge >}}
{{< badge >}}Titan Embeddings{{< /badge >}}

## Architecture Highlight

{{< mermaid >}}
flowchart LR
    subgraph INPUT["Ingestion"]
        API[API Gateway]
        EB[EventBridge]
    end
    
    subgraph AGENT["Agent Layer"]
        ORCH[Orchestrator Agent]
        DOC[Document Agent]
        KNOW[Knowledge Agent]
    end
    
    subgraph TOOLS["Tools & Data"]
        RAG[RAG System]
        APIS[Banking APIs]
        GUARD[Guardrails]
    end
    
    subgraph OBS["Observability"]
        TRACE[Tracing]
        EVAL[Evaluation]
    end
    
    INPUT --> AGENT
    AGENT --> TOOLS
    AGENT --> OBS
{{< /mermaid >}}

## Impact

- **50%+ reduction** in manual document processing time
- **Production-grade reliability** with <1% error rate on agent tasks
- **Scalable architecture** handling thousands of daily agent interactions
- **Knowledge democratization** — internal knowledge accessible via natural language
