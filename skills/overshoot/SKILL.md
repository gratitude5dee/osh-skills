---
name: overshoot
description: "Design real-time video intelligence workflows on Mux for Overshoot-style products: live stream ingestion, low-latency playback, RTMP/SRT encoder setup, webhook state machines, live stream health, instant clipping, stream recordings, and AI analysis handoff. Use when planning or reviewing low-latency video AI systems, livestream ops, video stream capture, or Mux-backed real-time multimodal pipelines."
---

# Overshoot

Use this skill to design real-time or near-real-time video intelligence systems on top of Mux. Treat Mux as the media transport and asset control plane, then hand stable asset, playback, transcript, or time-window signals to AI systems.

## Workflow

1. Identify the media mode: user upload, hosted file URL, reusable live stream, mobile camera stream, screen capture, simulated live, or third-party restream.
2. Keep Mux credentials and stream keys server-side. Never expose API access tokens in clients, logs, mobile bundles, or browser code.
3. Choose the ingest path:
   - VOD or AI-generated file: create an asset from a public URL or direct upload.
   - Browser/mobile user upload: create a direct upload URL server-side.
   - Real-time stream: create a Mux live stream and provide RTMP/RTMPS/SRT connection details only to the broadcaster.
4. Choose latency and resilience deliberately. Use `standard` for reliability, `reduced` or `low` only when encoder hardware, network, and player support are controlled.
5. Model state with webhooks, not polling. Persist asset IDs, live stream IDs, active asset IDs, playback IDs, and terminal states.
6. For AI handoff, prefer stable artifacts first: asset metadata, generated captions, transcript URLs, Robots API outputs, instant clipping ranges, and active or completed live stream recordings.
7. Add signed playback, playback restrictions, and short-lived JWTs when content access matters.

## References

- For current Mux source material, start with `https://www.mux.com/llms.txt` and load only the relevant `.txt` or `.md` docs.
- Read `references/mux-live-intelligence.md` for live stream architecture, state, ingest, instant clipping, and secure playback.
- Read `references/mux-operations.md` for encoder settings, reconnect behavior, live health, simulcast, SRT, and live captions.

## Output Standard

When planning an Overshoot-style workflow, include:

- Resource model: Mux assets, live streams, playback IDs, uploads, tracks, and database records.
- State machine: webhook events, allowed transitions, retry/recovery behavior, and failure states.
- Security boundaries: where credentials, stream keys, signing keys, and JWTs live.
- AI handoff: which Mux artifact or URL is passed to AI, when it becomes available, and how timecodes are preserved.
- Acceptance checks: readiness webhooks, playback URL behavior, latency target, health metrics, and moderation or privacy gates if relevant.
