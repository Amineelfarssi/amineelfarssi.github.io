---
title: "9 AI Agents Building My Startup: How I Run a Software Team with $0 Salaries"
date: 2026-03-02
description: "How I built a multi-merchant commerce platform (Luminar) using 9 specialized AI agents — each with a defined role, persona, and domain — dispatched via Linear webhooks."
tags: ["AI Agents", "Multi-Agent Systems", "Software Engineering", "LangGraph", "Automation"]
categories: ["Engineering"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Luminar has 173 source files, 21,586 lines of production code, 43 API endpoints, and 155+ tests. It was built almost entirely by AI agents. Here's the team structure, the workflow, and the honest truth about what breaks.
{{< /lead >}}

## The Team

I didn't want generic agents. I wanted specialists — each with a clear domain, sharp ownership boundaries, and a persona that shapes how they approach problems.

| Agent | Role | Model | What They Own |
|-------|------|-------|---------------|
| 🏗️ **Forge** | Platform Architect | Opus | ADRs, system topology, tech decisions |
| 🧠 **Sage** | AI Engineer | Opus | LiteLLM gateway, model routing, evals |
| ⚡ **Bolt** | Backend Engineer | Opus | FastAPI, PostgreSQL, Stripe, integrations |
| 🔌 **Wire** | Frontend Engineer | Opus | Next.js 16, console UI, Vercel deploy |
| ⚡ **Flux** | QA Engineer | Sonnet | Test suites, E2E tests, documentation |
| 🎨 **Muse** | Design Lead | Opus | Design system, component specs |
| 🌊 **Drift** | DevOps Engineer | Sonnet | CI/CD, Docker, GitHub Actions |
| 💎 **Prism** | Business Architect | Opus | Pricing, GTM, investor narrative |
| 🔭 **Scout** | Researcher | Sonnet | Protocol deep-dives, library evaluations |

The key was the **ownership model**. Forge doesn't write code. Sage doesn't touch UI. Bolt doesn't make architecture decisions. When an agent hits a boundary, they defer — not freelance outside their domain.

---

## The Dispatch Workflow

{{< mermaid >}}
flowchart LR
    A["📋 Linear Issue<br/>created"] --> B["🏷️ Label:<br/>agent:bolt"]
    B --> C["🔔 Webhook fires<br/>/opt/linear-webhook/handler.py"]
    C --> D{"Route by label"}
    D --> E["⚡ Bolt spawned<br/>on Lumi's server"]
    E --> F["🔨 Implement<br/>+ commit"]
    F --> G["📬 PR opened<br/>→ develop"]
    G --> H["👁️ Lumi reviews"]
    H --> I["✅ Merged"]
    style C fill:#1e3a5f,color:#fff
    style E fill:#10b981,color:#fff
{{< /mermaid >}}

Each agent gets triggered by a Linear label. The webhook handler on my AWS EC2 (Lumi, the project manager AI) reads the label and spawns the right agent via OpenClaw sessions. The agent reads the Linear issue, checks the existing codebase via GitHub, implements the change, and opens a PR to `develop`.

```python
AGENT_MAP = {
    "agent:bolt": "bolt",
    "agent:sage": "sage",
    "agent:wire": "wire",
    "agent:flux": "flux",
    "agent:forge": "forge",
    "agent:drift": "drift",
}
```

No manual dispatch. Add the label, the agent starts working.

---

## What's Been Built

After 14 sprints:

- **Backend**: FastAPI + LangGraph + Pydantic AI, 9 PostgreSQL tables
- **Protocols**: UCP (Google), ACP (OpenAI/Stripe), A2A (Google), MCP (Anthropic), TAP (Visa), AP2
- **Payments**: Stripe Connect Express with Separate Charges and Transfers
- **Search**: Pydantic AI buyer agent with A2A protocol
- **Auth**: API keys + OAuth 2.0 for Shopify
- **LLM**: AWS Bedrock Claude Sonnet via LiteLLM gateway
- **Tests**: 155+ tests, 23 production evals as launch gate

The codebase grew from 0 to 21k lines over roughly 6 weeks. Each sprint lasted 2-3 days.

---

## What Actually Works

{{< alert icon="check" cardColor="#14532d" iconColor="#4ade80" textColor="#f0f0f0" >}}
**Clear ownership eliminates merge conflicts.** When Bolt owns the backend and Wire owns the frontend, they rarely touch the same files. 90% of PRs merge cleanly.
{{< /alert >}}

**ADR-first architecture** saved us multiple times. Forge writes an Architecture Decision Record before any major feature. When an agent makes a wrong assumption, the ADR is the source of truth. Agents can reference ADR-017 and understand _why_ LiteLLM was chosen over direct Bedrock calls.

**Specialist context** beats generalist context. A "senior backend engineer" prompt who owns auth and integrations knows to check the existing API key middleware before adding a new one. A generic "code assistant" prompt won't.

---

## What Actually Breaks

{{< alert icon="triangle-exclamation" cardColor="#7c2d12" iconColor="#f97316" textColor="#f0f0f0" >}}
**The t3.medium (4GB RAM) can't run 5 concurrent Opus agents.** Each session is ~400MB. 5 agents = 2GB just for processes, plus Neo4j, the gateway, and Docker. OOM kills are frequent.
{{< /alert >}}

**Context resets are the main pain point.** When Lumi's server restarts mid-sprint, agents lose their session state. They restart from scratch, sometimes re-implementing work that was already done, sometimes conflicting with merged PRs.

**Webhook timing is fragile.** Labels applied before the webhook handler restarts = lost events. I've had to re-trigger dispatch manually by removing and re-adding labels more than once.

**Agents don't review each other's work well.** Bolt reviewing Wire's frontend PR doesn't catch CSS bugs. I eventually gave Lumi the review responsibility — she has broader context about what "done" looks like.

---

## The Economics

| Cost | Monthly |
|------|---------|
| Anthropic API (Opus + Sonnet) | ~$40-80 depending on sprint activity |
| AWS t3.medium (Lumi's server) | ~$30 |
| Bedrock (production LLM) | $0 so far (credits) |
| Everything else | $0 |

Per feature, the cost is roughly $5-15 in API calls. A full sprint (8-10 features) costs about $60-80. That's a few hours of a junior developer's time — for a week of parallel work across 9 specialists.

The ROI isn't mainly cost — it's **speed** and **scale**. Five agents working in parallel, 24/7, with no context-switching overhead.
