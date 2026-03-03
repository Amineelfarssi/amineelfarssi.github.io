---
title: "The Gaming Server AI Lab: Running Production AI Workloads for €25/month"
date: 2026-03-01
description: "How I turned a gaming PC into a production AI server running OpenClaw, Neo4j, Docker, GPU rendering, and local TTS — for less than a cloud t3.large."
tags: ["AI Infrastructure", "Self-Hosted", "GPU", "Cost Optimization", "Home Lab"]
categories: ["Engineering"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Cloud is convenient. But when you already own a gaming PC with a 2080 Ti collecting dust, the math changes fast. Here's how I turned mine into a production AI server running everything from Neo4j to GPU rendering — at €25/month.
{{< /lead >}}

## The Hardware

I didn't buy anything new. This is the machine I had:

| Component | Spec |
|-----------|------|
| CPU | Intel i5-9600K — 6 cores / 6 threads @ 3.7GHz |
| RAM | 64GB DDR4 |
| GPU 1 | RTX 2080 Ti — 11GB VRAM (CUDA compute) |
| GPU 2 | GTX 1060 3GB (display + overflow) |
| OS | Ubuntu 22.04, CUDA 12.5 |
| Storage | NVMe SSD |

The RTX 2080 Ti is the key. 11GB of VRAM is enough to run serious local models, Blender GPU rendering with OptiX, and Kokoro TTS simultaneously.

---

## What's Running On It

{{< mermaid >}}
graph TD
    GW["🦞 OpenClaw Gateway\n(port 18789)"]
    NEO["🧠 Neo4j KG\n(port 7687, Docker)"]
    CHR["🔍 ChromaDB\n(vector memory)"]
    PLEX["🎬 Plex + *arr stack\n(ports 32400, 7878, 8989...)"]
    DOCKER["🐳 Docker (Colima)"]
    GPU1["⚡ RTX 2080 Ti\nBlender OptiX\nKokoro TTS\nOllama models"]
    XVFB["🖥️ Chrome on Xvfb:99\n(X autopilot, CDP:9222)"]
    NETDATA["📊 Netdata v2.8.5\n(monitoring)"]

    GW --> NEO
    GW --> CHR
    DOCKER --> NEO
    DOCKER --> PLEX
    GPU1 --> GW
{{< /mermaid >}}

**Always-on services:**
- OpenClaw gateway — the AI brain (WebSocket, port 18789)
- Neo4j — knowledge graph memory (Docker)
- ChromaDB — vector memory
- Plex + qBittorrent + Radarr + Sonarr + Prowlarr
- Netdata monitoring dashboard

**On-demand:**
- Blender GPU rendering (OptiX on RTX 2080 Ti)
- Kokoro TTS (local voice synthesis, zero cost)
- Chrome on Xvfb + CDP for automation
- Ollama local models

---

## Power & Cost Breakdown

{{< alert icon="bolt" cardColor="#1e3a5f" iconColor="#f59e0b" textColor="#f0f0f0" >}}
Belgium electricity rate: ~€0.35/kWh. At idle the server draws ~100W. With agents and rendering active, it can hit 500W.
{{< /alert >}}

| Scenario | Power Draw | Monthly Cost |
|----------|-----------|--------------|
| Idle (gateway + Docker) | ~100W | **€25** |
| Active AI agents | ~150-200W | **€35-40** |
| GPU rendering burst | ~500W | Peaks only, averages out |
| **Realistic average** | ~130W | **~€30/month** |

Now compare to what I'd pay on AWS:

| Cloud Option | Monthly Cost | GPU? |
|-------------|-------------|------|
| t3.medium (4GB RAM) | ~$30 | ❌ |
| t3.large (8GB RAM) | ~$60 | ❌ |
| g4dn.xlarge (T4 GPU) | ~$380 | ✅ (16GB) |
| Gaming server (gaming PC I own) | **€25-30** | ✅ (11GB VRAM) |

The gaming server wins on every axis except latency and uptime guarantees. For a dev/staging setup and personal AI infra, that's fine.

---

## Remote Access

The machine sits in my home but I access it from everywhere via **Tailscale**:

```bash
# From my Mac
ssh amine@amine-ms-7b17.taild14f8d.ts.net

# OpenClaw TUI
openclaw tui --url wss://amine-ms-7b17.taild14f8d.ts.net --token <token>
```

Tailscale gives me a stable hostname regardless of ISP changes, with WireGuard encryption. The gateway binds to `127.0.0.1` and Tailscale Serve proxies it over HTTPS — no port forwarding, no exposed ports.

---

## What Surprised Me

**The 64GB RAM matters more than the GPU.** Running 5 concurrent Claude sub-agents means 5 × ~400MB Node.js processes plus Neo4j (800MB) plus Docker containers. 16GB would OOM immediately. 64GB means I never think about it.

**No swap was a mistake.** I was running with zero swap on Ubuntu until an agent storm caused an OOM kill. Added 2GB swap as a safety net — it's never actually used but it prevents hard crashes.

**The GTX 1060 is useless for CUDA.** SM 6.1 isn't supported by modern PyTorch. It handles display output and that's it.

---

## Is This Worth It?

If you already own the hardware: absolutely yes. The break-even vs. renting a GPU cloud instance is immediate. I run:
- A production AI agent stack
- Local video rendering
- A media server
- All monitoring and backups

...for less than a t3.large with zero GPU. The tradeoff is uptime (home internet can hiccup) and maintenance time. For personal AI infrastructure, both are acceptable.

The real unlock was pairing it with Tailscale + OpenClaw. Now my phone and Mac both connect to the same AI brain running on this machine, and the brain can use the GPU.
