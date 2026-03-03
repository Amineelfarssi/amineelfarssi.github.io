---
title: "Voice Cloning Reality Check: F5-TTS With 12 Seconds of Audio"
date: 2026-03-01
description: "I tried to clone a voice for AI content using F5-TTS and a Telegram audio clip. Here's what happened, why it failed, and what you'd actually need to make it work."
tags: ["TTS", "Voice Cloning", "F5-TTS", "AI Audio", "Experiment"]
categories: ["Experiments"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
Premise: use F5-TTS to clone a voice from a short reference clip and generate high-quality narration for AI content. Reality: mediocre output, weird artifacts, wrong prosody. Here's the honest post-mortem.
{{< /lead >}}

## The Setup

F5-TTS is a non-autoregressive TTS model that uses flow matching for zero-shot voice cloning. You give it:
1. A reference audio clip (the voice to clone)
2. The transcript of that reference audio
3. The text to generate

In theory, it should replicate the voice from just a few seconds of audio. The model runs locally — perfect for a GPU server.

**My setup:**
- RTX 2080 Ti (11GB VRAM, CUDA 12.5)
- F5-TTS with all deps: torch 2.10.0, torchaudio, torchcodec
- Reference audio: 12-second clip extracted from a Telegram voice message
- Target: narration for DPO YouTube Shorts

```bash
# Install (took a while to sort out the torch version)
pip install f5-tts torch==2.10.0 torchaudio torchcodec

# Run
f5-tts_infer-cli \
  --model F5TTS_v1_Base \
  --ref_audio reference.wav \
  --ref_text "This is what I said in the reference clip." \
  --gen_text "This is the text I want to generate." \
  --output_dir output/
```

---

## What Actually Came Out

{{< alert icon="triangle-exclamation" cardColor="#7c2d12" iconColor="#f97316" textColor="#f0f0f0" >}}
The output was... recognizable as voice. But the prosody was wrong — flat where it should rise, clipped consonants, and a faint metallic undertone throughout. Not usable for content.
{{< /alert >}}

Three specific issues:

**1. Prosody collapse.** The generated speech had no sentence-level intonation. Questions sounded flat. Emphasis was random. This is a known F5-TTS limitation with short reference audio — the model can't capture the speaker's prosodic patterns from 12 seconds.

**2. Telegram compression artifacts.** Telegram uses Opus codec at a variable bitrate optimized for voice calls, not audio fidelity. The reference audio had encoding artifacts that F5-TTS interpreted as features of the voice — and faithfully reproduced them in the output.

**3. Background noise bleed.** The reference clip had faint ambient sound. Again, F5-TTS treated it as part of the voice signature.

---

## What You Actually Need

| Parameter | My Setup | What Works |
|-----------|----------|------------|
| Reference length | 12 seconds | **2-3 minutes** |
| Recording method | Telegram voice message | **Phone's native recorder** |
| Environment | Unclear | **Quiet room, consistent distance** |
| Format | Telegram Opus | **WAV, 44.1kHz, uncompressed** |

The minimum viable reference clip for decent zero-shot cloning is probably 90 seconds. For good results, 2-3 minutes. At 12 seconds, the model is interpolating more than it's copying.

---

## What I Use Instead

While waiting to record a proper reference, I use two alternatives:

**Kokoro TTS** for the DPO channel:
- Runs fully local on the gaming server
- `am_michael` voice — American male, clear and natural
- Cost: $0/month
- Quality: good enough for 30-60s YouTube Shorts
- Speed: generates faster than real-time on the RTX 2080 Ti

**ElevenLabs Lily** for premium outputs:
- Velvety British actress voice (`pFZP5JQG7iQjIQuC4Bku`)
- `eleven_multilingual_v2`, stability=0.5, similarity_boost=0.75
- Cost: ~177 chars remaining on free tier (resets monthly)
- Quality: noticeably better than local TTS

---

## Is F5-TTS Worth It?

Yes — but not for what I tried to use it for. Zero-shot cloning from compressed audio is optimistic. The actual sweet spot for F5-TTS is:

- You have a clean, long reference recording
- You want to generate consistent narration in that voice over time
- You don't want to pay per character (ElevenLabs)
- You're OK with occasional prosody quirks

The model is genuinely impressive given it runs locally. My experiment just highlighted that "zero-shot" doesn't mean "zero effort" — the reference audio quality matters as much as the model quality.

Next time I try it: 3 minutes recorded in my home office on a phone held at face distance. No Telegram, no background noise. I'll write a follow-up.

---

## Verdict

{{< alert icon="check" cardColor="#14532d" iconColor="#4ade80" textColor="#f0f0f0" >}}
**Would use again?** Yes, with proper reference audio.  
**Recommendation:** Record 2-3 min of clean audio before even installing F5-TTS. The tooling is fine; the bottleneck is the reference quality.
{{< /alert >}}
