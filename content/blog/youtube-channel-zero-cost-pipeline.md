---
title: "Running a YouTube Channel for $0: Manim + Local TTS + FFmpeg + YouTube API"
date: 2026-03-02
description: "The full pipeline behind the DPO channel — AI/ML technical Shorts built with Manim animations, Kokoro local TTS, FFmpeg editing, and YouTube Data API v3. Total monthly cost: ~$0.05 per video."
tags: ["YouTube", "Automation", "Manim", "TTS", "FFmpeg", "Content Creation"]
categories: ["Engineering"]
showTableOfContents: true
showReadingTime: true
showWordCount: true
---

{{< lead >}}
The DPO channel (<a href="https://youtube.com/@DPO-AI">@DPO-AI</a>) publishes AI/ML technical Shorts. 7 videos uploaded so far, covering agent memory, HNSW indexing, and agentic protocols. The entire production pipeline costs less than a coffee per video.
{{< /lead >}}

## Why Build This

I wanted to publish technical AI content that goes beyond surface-level explanations — real system architecture, real algorithms, real trade-offs. And I wanted it to be visually compelling, not just a talking head.

The constraint: I have zero budget for production software and zero patience for manual video editing.

The solution: automate the entire pipeline from script to upload.

---

## The Pipeline

{{< mermaid >}}
flowchart TD
    A["💡 Topic"] --> B["🤖 Research<br/>Kimi K2.5 (~$0.001)"]
    B --> C["✍️ Script<br/>Claude Opus (~$0.05)<br/>~500 chars = 35s speech"]
    C --> D["🎬 Animation<br/>Manim on RTX 2080 Ti<br/>OptiX GPU rendering"]
    D --> E["🎙️ Narration<br/>Kokoro TTS local<br/>am_michael voice"]
    E --> F["✂️ Edit<br/>FFmpeg: sync + speed<br/>max 1.2x"]
    F --> G["📤 Upload<br/>YouTube Data API v3<br/>OAuth2 token"]
    G --> H["📱 Short published<br/>< 60s, vertical 9:16"]
    style D fill:#1e3a5f,color:#fff
    style E fill:#10b981,color:#fff
{{< /mermaid >}}

---

## Step 1: Research (Kimi K2.5)

Kimi K2.5 is cheap and has a massive context window. I use it to research the topic, pull together relevant papers, and draft a content brief. A full research pass costs literally fractions of a cent.

The brief includes: core concept, key visual metaphors, 3-5 key points to hit, suggested Manim scene structure.

---

## Step 2: Script (Claude Opus)

The script is the most important part. The constraint is strict:

{{< alert icon="calculator" cardColor="#1e3a5f" iconColor="#60a5fa" textColor="#f0f0f0" >}}
**~500 characters = ~35 seconds of natural speech.** YouTube Shorts must be under 60 seconds. That gives you roughly 800 characters max, and you want breathing room.
{{< /alert >}}

The script has to be dense but not rushed — every word has to earn its place. Claude Opus at ~$0.05 per script is worth it for the quality jump over cheaper models. Sonnet works too but needs more prompting.

---

## Step 3: Manim Animation

[Manim](https://manim.community) is the same library 3Blue1Brown uses for his math videos. It's pure Python — you describe animations programmatically and render them.

```python
class AgentMemoryScene(ThreeDScene):
    def construct(self):
        # Create 3D surface representing embedding space
        surface = Surface(
            lambda u, v: np.array([u, v, np.sin(u)*np.cos(v)]),
            u_range=[-3, 3], v_range=[-3, 3],
            resolution=(30, 30)
        )
        self.play(Create(surface))
        # ... animate memory retrieval path
```

The RTX 2080 Ti renders with **OptiX** — GPU-accelerated ray tracing. A 30-second 720p animation renders in about 45 seconds. Without GPU it would take 10+ minutes.

For the DPO channel, scenes include: HNSW graph traversal, agent memory hierarchy, protocol stack diagrams, gradient descent visualization.

---

## Step 4: Kokoro TTS (Local Voice)

Kokoro is a local TTS model that runs on the gaming server. The `am_michael` voice is clear, natural, and works well for technical content.

```python
import kokoro

pipeline = kokoro.KPipeline(lang_code='a')  # American English
audio, sample_rate = pipeline(script, voice='am_michael')
```

No API calls, no rate limits, no cost. The RTX 2080 Ti generates speech faster than real-time.

The only limitation: no emotional range. Kokoro is clear and neutral — great for explanation content, not for hype. For the occasional premium video, I switch to ElevenLabs Lily.

---

## Step 5: FFmpeg Editing

The sync step is simple but fussy:

```bash
# Merge audio + video, extend if needed
ffmpeg -i animation.mp4 -i narration.wav \
  -c:v copy -c:a aac \
  -shortest output.mp4

# Speed up slightly if over 60s (max 1.2x sounds natural)
ffmpeg -i output.mp4 -filter:v "setpts=0.85*PTS" \
  -filter:a "atempo=1.18" final.mp4
```

{{< alert icon="triangle-exclamation" cardColor="#7c2d12" iconColor="#f97316" textColor="#f0f0f0" >}}
**1.2x is the ceiling for natural-sounding audio speedup.** Above that, speech starts sounding chipmunk-ish. If the script is too long, cut it — don't speed it up.
{{< /alert >}}

---

## Step 6: YouTube Data API v3

```javascript
// node upload.js --file final.mp4 --title "..." --description "..."
const youtube = google.youtube({ version: 'v3', auth: oauth2Client });
await youtube.videos.insert({
  part: ['snippet', 'status'],
  requestBody: { snippet: { title, description, tags }, 
                 status: { privacyStatus: 'public' } },
  media: { body: fs.createReadStream('final.mp4') }
});
```

OAuth2 token stored at `x-autopilot/yt-token.json`. The same Node.js infrastructure that handles X/Twitter posting handles YouTube uploads — shared auth layer.

---

## Cost Per Video

| Step | Tool | Cost |
|------|------|------|
| Research | Kimi K2.5 | ~$0.001 |
| Script | Claude Opus | ~$0.05 |
| Animation | Manim (local) | $0 |
| Voice | Kokoro TTS (local) | $0 |
| Editing | FFmpeg (local) | $0 |
| Upload | YouTube API | $0 |
| **Total** | | **~$0.05** |

Seven Shorts published. Total cost: about 35 cents in API calls.

The channel is still small — it's more of a proof-of-concept than a growth vehicle right now. The real value is the pipeline: once a video format works, producing more is nearly free.
