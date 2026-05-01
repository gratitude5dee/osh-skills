# Mux Live Intelligence Reference

Use this reference when planning real-time video AI systems with Mux as the media layer.

## Core resource model

- Organization: top-level billing and team container.
- Environment: isolated resources for assets, live streams, tokens, signing keys, and webhooks.
- Access token: Token ID plus Token Secret for server-to-Mux API requests. Keep server-side.
- Asset: processed video or audio file used for on-demand playback and API management.
- Playback ID: identifier used with `stream.mux.com` or `image.mux.com` to play or preview content.
- Live stream: reusable broadcast resource that receives RTMP, RTMPS, or SRT input.
- Stream key: secret credential for broadcasting to a live stream.
- Signing key: key pair used to create JWTs for signed playback and restricted access.

## Basic VOD flow

1. Create an asset from a public input URL or create a direct upload URL server-side.
2. Request playback policy: `public` for open playback, `signed` for JWT-protected playback.
3. Wait for `video.asset.ready`; do not build production systems around repeated asset polling.
4. Play with `https://stream.mux.com/{PLAYBACK_ID}.m3u8`.
5. Preview with thumbnails, storyboards, or `https://stream.new/v/{PLAYBACK_ID}`.

## Live stream flow

1. Create a live stream with `playback_policies` and `new_asset_settings`.
2. Return stream key and ingest URL only to the trusted broadcaster.
3. Broadcaster sends RTMP/RTMPS or SRT to Mux.
4. Use the live stream playback ID for non-DVR playback.
5. Use the active asset playback ID for DVR or the resulting recording.
6. Use webhooks to update UI and database state.

Important webhook events:

- `video.live_stream.connected`: encoder connected; not necessarily playable.
- `video.live_stream.recording`: Mux is recording incoming content.
- `video.live_stream.active`: live playback is available.
- `video.live_stream.disconnected`: encoder disconnected.
- `video.live_stream.idle`: reconnect window expired or stream finished.
- `video.asset.ready`: asset is ready for playback.
- `video.asset.live_stream_completed`: recording finalized as VOD.

## Latency choices

- `standard`: most resilient; typical HLS latency is higher.
- `reduced`: lower latency when encoder and network are controlled.
- `low`: LL-HLS path with the lowest latency target; requires compatible players and stable ingest.

Only choose `reduced` or `low` when the product controls encoder hardware, encoder software, network quality, and supported players.

## Instant clipping and time windows

Use instant clipping when a new encoded asset is not required:

- Live or live-origin content: `program_start_time` and `program_end_time` use epoch timestamps.
- VOD content: `asset_start_time` and `asset_end_time` use seconds relative to asset start.
- Accuracy is segment-level, not frame-level.
- For signed playback, include clipping parameters in JWT claims rather than URL query parameters.

Use asset-based clipping when frame accuracy, a standalone asset, downloadable MP4s, text tracks, or watermarks are required.

## AI handoff patterns

- Use asset IDs to manage and enrich media through APIs.
- Use playback IDs to construct playback, thumbnail, storyboard, and transcript URLs.
- Use active asset IDs from live streams to connect live sessions to recordings.
- Use captions and transcript tracks for speech-aware AI.
- Use Robots API jobs for structured AI outputs when lower-level custom inference is not required.
- Preserve timecodes so AI outputs can drive chapter jumps, key moment previews, or instant clips.

## Security boundaries

- Mux API calls must originate from trusted servers.
- Do not expose Mux API credentials, token secrets, signing private keys, stream keys, or service-role database credentials to clients.
- Use signed playback IDs and playback restrictions for private or paid content.
- Public playback IDs should never be passed a `token` query parameter.
