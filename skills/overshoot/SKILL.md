---
name: overshoot
description: "Design realtime vision workflows with Overshoot: /v1/streams sessions, LiveKit video publishing, ovs:// frame and segment references, /v1/models selection, /v1/chat/completions requests, keepalive scheduling, stream lifecycle handling, model fallback, and low-latency video AI architecture. Use when planning or reviewing Overshoot live video understanding, webcam-to-model flows, stream Q&A, or realtime multimodal inference systems."
---

# Overshoot

Use this skill to design realtime vision systems on Overshoot. Overshoot creates leased live-video streams, accepts video through LiveKit/WebRTC, and lets OpenAI-compatible chat completion requests reference frames or segments with `ovs://streams/{stream_id}` URLs.

## Workflow

1. Start from the current docs index at `https://docs.overshoot.ai/llms.txt` when exact endpoint behavior matters.
2. Keep `ovs-...` API keys server-side. Do not expose bearer tokens in browser code, mobile bundles, logs, or generated client snippets. `GET /v1/models` is the only unauthenticated endpoint.
3. List models before each session with `GET /v1/models`; filter to `status == "ready"` and keep at least one fallback because `/v1/chat/completions` can still return `503`.
4. Create the stream with `POST /v1/streams`. Store the returned `stream_id`, `publish.url`, `publish.token`, `expires_at_ms`, and `ttl_seconds`.
5. Connect the publisher through LiveKit using the returned room URL and token. Treat the stream as active immediately but wait for `last_frame_at_ms` or `last_frame_index` before asking frame-specific questions.
6. Ask the model with `POST /v1/chat/completions`:
   - Use `image_url` with `ovs://streams/{stream_id}?frame_index=-1` for latest-frame Q&A.
   - Use `video_url` with `ovs://streams/{stream_id}?start_offset_ms=-5000` for last-N-seconds analysis.
   - Use bounded `start_*` and `end_*` anchors for review of a specific retained window.
7. Schedule keepalives from `expires_at_ms` and `ttl_seconds`; save the fresh `publish.token` returned by each keepalive. If docs or product guidance require 10-20 second renewal, follow that stricter interval.
8. Delete streams with `DELETE /v1/streams/{stream_id}` when done. Ended streams cannot be resumed; create a new stream after expiry, deletion, or reaping.

## References

- Read `references/overshoot-api.md` for base URL, auth, endpoints, lifecycle, errors, keepalive behavior, model listing, and stream operations.
- Read `references/overshoot-stream-uris.md` for `ovs://` URL grammar, image and video content parts, anchor validation, examples, and token/cost controls.
- Read `references/mux-live-intelligence.md` only when Mux is part of the surrounding media system for recording, VOD playback, asset storage, clipping, transcripts, or playback IDs.
- Read `references/mux-operations.md` only when integrating Overshoot with Mux live operations, encoder setup, RTMP/SRT ingest, simulcast, captions, or live health workflows.

## Output Standard

When planning an Overshoot workflow, include:

- Resource model: `stream_id`, LiveKit room URL/token, model id, retained frame window, and any app-side session records.
- Stream lifecycle: create, publisher connect, first-frame readiness, keepalive cadence, terminal `ended` state, delete behavior, and recovery path.
- Prompting plan: when to use latest frame, relative offset, bounded segment, `max_fps`, tolerance, and model fallback.
- Security boundaries: server-side API key handling, token refresh, client-visible LiveKit token handling, and log redaction.
- Failure handling: `401/403`, `402`, `404`, `422`, `503`, no frames yet, expired streams, future-frame references, and old-frame clamping.
- Acceptance checks: ready model selected, stream receives frames, expected `ovs://` URL resolves, keepalive updates `expires_at_ms`, retries/fallbacks work, and streams are deleted after use.
