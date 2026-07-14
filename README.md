# Automated Video Creation Pipeline

**Role:** Lead Developer & Automation Architect  
**Tech Stack:** n8n, Google Gemini API (script gen + TTS), Node.js/Editly (FFmpeg rendering), Cloudflare R2 (S3-compatible storage)

**Files:**
- `Editly-Video-Pipeline.json` — Sanitized n8n workflow export (webhook trigger, 27 nodes)

## What It Does

Generates personalized marketing videos for businesses, fully automated from lead data to published video. Each video is customized with the company's name, website screenshot, industry-specific messaging, and AI-generated voiceover.

### Pipeline Flow

```
┌──────────────────────────────────────────────────────────────┐
│  STAGE 1: AI Script Generation (Gemini, ~3s)                 │
│  Generates a 30-second voiceover script customized to the    │
│  company's industry, angle, and contact name.                │
├──────────────────────────────────────────────────────────────┤
│  STAGE 2: Text-to-Speech (Google TTS, ~5s)                   │
│  Converts script to natural-sounding voiceover audio (PCM).  │
│  Configurable voice, pitch, and speed.                       │
├──────────────────────────────────────────────────────────────┤
│  STAGE 3: Video Rendering (Editly + FFmpeg, ~20-40s)         │
│  Composites: website screenshot + brand images + voiceover   │
│  + transitions + titles into 720x1280 vertical video.        │
│  5-clip structure: screenshot → images → CTA.                │
├──────────────────────────────────────────────────────────────┤
│  STAGE 4: Cloud Upload (S3-compatible, ~3s)                  │
│  Uploads rendered MP4 to object storage. Updates the CRM     │
│  record with the public video URL.                           │
└──────────────────────────────────────────────────────────────┘
```

### Multi-Angle Content Strategy

The pipeline supports multiple messaging angles, each with different assets and script prompts:

| Angle | Script Focus | Image Assets | Use Case |
|-------|-------------|--------------|----------|
| `value_proposition` | Cost savings, ROI | Hero image, benefits graphic | General outreach |
| `before_after` | Transformation proof | Before/after photos | Quality demonstration |
| `seasonal` | Urgency, limited-time | Seasonal imagery | Holiday/summer campaigns |
| `storm_damage` | Emergency response | Damage photos, insurance | Post-storm outreach |

### Key Design Decisions

**Dedicated render service:**
The FFmpeg/Editly rendering is handled by a separate Node.js microservice (not embedded in this Python pipeline). This keeps the orchestrator lightweight and allows horizontal scaling of render workers for concurrent video jobs.

**PCM → WAV Data URI for audio:**
Instead of passing audio as a file path (which requires shared filesystem), the pipeline converts raw PCM to a WAV Data URI that Editly can consume directly via HTTP. This eliminates server-side file management.

**Artifact validation:**
After rendering, the pipeline checks that the output is actually a valid MP4 (checks for `ftyp` box header). Renders can fail silently (e.g., missing FFmpeg codecs, bad input), and without validation you'd upload a corrupt file and not know until someone tries to play it.

## NDA Disclaimer

**Some implementation details have been modified** to comply with confidentiality obligations:
- Actual brand assets (image paths, logo URLs, background music) are replaced with placeholders
- Exact angle-specific script prompts have been generalized
- Background music selection logic has been removed
- Specific object storage bucket names and endpoints have been changed
- Company-specific video watermarking and branding logic is excluded

The 4-stage pipeline architecture, data flow, and integration patterns are accurate representations of the production system.

