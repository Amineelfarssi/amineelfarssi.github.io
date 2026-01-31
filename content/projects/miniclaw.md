---
title: "MiniClaw: Python Agent Framework"
description: "A minimal agent orchestration framework implementing OpenClaw's architecture patterns"
tags: ["Python", "AI Agents", "Open Source", "LLM", "Architecture"]
weight: 0
---

{{< badge >}}Python{{< /badge >}}
{{< badge >}}OpenAI{{< /badge >}}
{{< badge >}}Anthropic{{< /badge >}}
{{< badge >}}Open Source{{< /badge >}}

## Overview

**MiniClaw** is a ~2,800 line Python implementation of the core patterns from [OpenClaw](https://github.com/openclaw/openclaw), the open-source agent orchestration gateway. Built as both a learning tool and a functional framework.

{{< alert icon="code" cardColor="#10b981" iconColor="#fff" textColor="#fff" >}}
**Goal:** Understand agent architecture by building it from scratch — not by hiding behind a library.
{{< /alert >}}

## Architecture

{{< mermaid >}}
flowchart TB
    subgraph CHANNELS["Channels"]
        CLI[CLI]
        WH[Webhook]
        EXT[Extensible...]
    end
    
    subgraph GATEWAY["Gateway"]
        ROUTER[Message Router]
        EVENTS[Event System]
    end
    
    subgraph CORE["Core"]
        SESS[Session Store<br/>Keys, JSONL, Resets]
        MEM[Workspace Memory<br/>SOUL.md, USER.md]
        AGENT[Agent Loop<br/>Think → Act → Observe]
        TOOLS[Tool Registry]
    end
    
    CHANNELS --> GATEWAY
    GATEWAY --> CORE
{{< /mermaid >}}

## Key Features

### OpenClaw-Style Sessions
Session keys following the OpenClaw convention:
```
agent:main:main                    → Primary DM
agent:main:whatsapp:group:123      → Group chat
agent:main:dm:+1234567890          → Per-peer DM
```

Features:
- **Daily resets** at configurable hour (default 4 AM)
- **JSONL transcripts** for full history
- **Compaction** when nearing token limits
- **Idle timeouts** for inactive sessions

### Workspace Memory
Plain Markdown files as the agent's memory:

| File | Purpose | When Loaded |
|------|---------|-------------|
| `SOUL.md` | Agent personality | Always |
| `USER.md` | Human context | Always |
| `MEMORY.md` | Long-term memories | Main session only |
| `memory/YYYY-MM-DD.md` | Daily notes | Today + yesterday |

### Multi-Provider Support
Same code works across providers:
```python
# OpenAI
agent = Agent(AgentConfig(model="gpt-4o", provider="openai"))

# Anthropic
agent = Agent(AgentConfig(model="claude-3-5-sonnet", provider="anthropic"))

# Local (Ollama)
agent = Agent(AgentConfig(model="llama3.2", provider="ollama"))
```

### TUI Interface
Rich terminal interface with:
- Formatted message display
- Tool call visualization
- Session status bar
- Slash commands (`/help`, `/status`, `/new`)

## Code Structure

```
miniclaw/
├── __init__.py     # Package exports
├── tools.py        # @tool decorator, registry, built-ins
├── session.py      # Sessions, keys, JSONL, compaction
├── memory.py       # SOUL.md, USER.md, MEMORY.md
├── agent.py        # The agent loop
├── gateway.py      # Orchestrator + channels
├── tui.py          # Rich terminal UI
└── cli.py          # CLI entry point
```

## Usage

```bash
# Install
pip install -e .

# Run CLI
miniclaw

# Run TUI
miniclaw tui

# One-shot
miniclaw --one-shot "List files in current directory"
```

## Why Build This?

Most agent frameworks are either:
- **Too simple** — just wrappers around API calls
- **Too complex** — hundreds of abstractions

MiniClaw sits in the middle: **fully functional** but **completely readable**. Every pattern is explicit and documented.

{{< alert icon="lightbulb" >}}
**Use MiniClaw to learn.** Use OpenClaw for production.
{{< /alert >}}

## Related

- [OpenClaw Architecture Deep Dive](/blog/openclaw-architecture-deep-dive/) — The blog explaining these patterns
- [Agentic Protocols](/blog/agentic-protocols-a2ui-a2a-mcp-ucp/) — MCP, A2A, A2UI, UCP

{{< button href="https://github.com/amineelfarssi" target="_blank" >}}
View on GitHub
{{< /button >}}
