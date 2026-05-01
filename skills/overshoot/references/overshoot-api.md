# Overshoot API Reference

Use this reference for Overshoot endpoint shape, lifecycle, authentication, and operational behavior.

## Source of truth

- Docs index: `https://docs.overshoot.ai/llms.txt`
- Base URL: `https://api.overshoot.ai/v1`
- OpenAPI spec: `https://docs.overshoot.ai/api-reference/openapi.yaml`

## Authentication

- API keys are bearer tokens prefixed with `ovs-`.
- Include `Authorization: Bearer $OVERSHOOT_API_KEY` on every authenticated request.
- Never expose API keys in browser code, mobile bundles, logs, telemetry, screenshots, or generated client examples.
- `GET /v1/models` is the only endpoint that does not require authentication.

## Endpoint surface

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| `GET` | `/v1/models` | No | List vision-language models that are ready or loading. |
| `POST` | `/v1/streams` | Yes | Create a leased stream and get LiveKit publish details. |
| `GET` | `/v1/streams/{stream_id}` | Yes | Inspect state, frame counts, recent FPS, and lease expiry. |
| `POST` | `/v1/streams/{stream_id}/keepalive` | Yes | Renew the lease and get a fresh LiveKit token. |
| `DELETE` | `/v1/streams/{stream_id}` | Yes | End a stream and release retained frames. |
| `POST` | `/v1/chat/completions` | Yes | Ask a model about stream frames or segments. |

## Session workflow

1. Call `GET /v1/models`; filter to `status == "ready"`.
2. Pick a model id that matches latency, quality, and context needs. Use the id exactly as returned.
3. Call `POST /v1/streams`.
4. Give the publisher the returned `publish.url` and `publish.token` for LiveKit.
5. Check `GET /v1/streams/{stream_id}` until frames arrive (`last_frame_at_ms` or `last_frame_index` is not null).
6. Use `/v1/chat/completions` with `ovs://` frame or segment references.
7. Renew with `/keepalive` before expiry and save the fresh `publish.token`.
8. Call `DELETE /v1/streams/{stream_id}` when the session ends.

## Stream creation response

`POST /v1/streams` returns:

- `id`: the `stream_id` used in path params and `ovs://streams/{stream_id}` references.
- `state`: `active` for a newly created stream.
- `publish.type`: currently `livekit`.
- `publish.url`: LiveKit room URL, usually `wss://...`.
- `publish.token`: short-lived JWT for the publisher.
- `expires_at_ms`: wall-clock Unix ms lease deadline.
- `ttl_seconds`: lease TTL, commonly `300` in the docs examples.

The stream is `active` before the first frame arrives. Until ingest starts, frame and timestamp fields can be `null`.

## Stream status fields

Use `GET /v1/streams/{stream_id}` to read:

- Wall-clock fields: `created_at_ms`, `first_frame_at_ms`, `last_frame_at_ms`, `first_available_frame_at_ms`, `expires_at_ms`, `ended_at_ms`.
- Stream-clock field: `stream_time_ms`, starting at `0` from the first frame.
- Frame-index fields: `first_available_frame_index`, `last_frame_index`, `retained_frame_count`, `evicted_frame_count`.
- Health fields: `recent_fps`, `state`, `end_reason`.

`state` moves forward from `active` to `ended` only. Ended streams cannot be resumed.

## Keepalive behavior

- Use `expires_at_ms` and `ttl_seconds` from the API response as the authoritative renewal schedule.
- Some Overshoot docs warn to call keepalive every 10-20 seconds and mention shorter expiry behavior. If product guidance or live testing shows that stricter interval, use it.
- Each successful keepalive returns a new `expires_at_ms`, `ttl_seconds`, current `stream_time_ms`, and fresh `publish.token`.
- A keepalive on an expired or ended stream returns `404`; create a new stream instead of trying to revive it.

## Model selection

- Do not hardcode model availability. Call `GET /v1/models` before each session.
- Use only models with `status == "ready"`.
- Treat `loading` as unavailable for production flows unless the user explicitly accepts waiting.
- A ready model can still return `503`; retry briefly or fall back to another ready model.
- Visual tokens dominate cost and latency. Reduce resolution, shorten segments, lower frame count, or lower `max_fps` before changing application behavior.

## Errors

Non-2xx responses generally use `{ "detail": "..." }`; validation errors can include `details`.

| Code | Meaning | Action |
| --- | --- | --- |
| `401`, `403` | Missing, invalid, or unauthorized API key. | Check server-side secret handling and account access. |
| `402` | Insufficient credits. | Stop renewing or creating streams; ask for billing action. |
| `404` | Stream not found, expired, ended, deleted, or not owned by key. | Create a new stream or fix stream ownership. |
| `422` | Request validation failed. | Check `ovs://` query grammar and request fields. |
| `503` | Model or ingest service unavailable. | Retry with backoff or use a fallback ready model. |

## Limits and cleanup

- Docs mention a 5 concurrent stream limit per API key.
- Stream buffers retain recent frames; older frame indices can clamp to the oldest available frame.
- Delete streams when finished rather than relying on lease expiry.
