# Mux and Supabase Schema Reference

Use this reference when storing Mux resources in Supabase or Postgres for video search.

## Recommended tables

Core Mux mirror tables:

- `mux_assets`: asset status, playback IDs, duration, aspect ratio, tracks, quality, upload ID, live stream ID, passthrough.
- `mux_uploads`: direct upload status, timeout, and resulting asset ID.
- `mux_webhook_events`: event ID, type, object type, object ID, payload, and received timestamp.
- `mux_live_streams`: live stream status, playback IDs, active asset ID, recent asset IDs, latency mode, reconnect window.

App metadata table:

- `videos`: app-owned title, description, user ID, visibility, tags, and link to `mux_asset_id`.

Search enrichment tables or columns:

- `video_transcripts`: asset ID, track ID, language, transcript text, cue JSON or separate cue rows.
- `video_chapters`: asset ID, start time, title, source job ID.
- `video_key_moments`: asset ID, start/end milliseconds, title, narratives, concepts, score, source job ID.
- `video_ai_tags`: asset ID, tag, source workflow, confidence if available.
- `video_embeddings`: asset ID or moment ID, embedding vector, source text, source type.

## Webhook sync

Persist webhook events before applying state changes so processing is idempotent and auditable.

Important events:

- `video.asset.created`: upsert asset as preparing.
- `video.asset.ready`: mark ready and store playback IDs, duration, tracks, aspect ratio, quality.
- `video.asset.errored`: mark errored and store error context.
- `video.upload.asset_created` or upload asset-ready event: connect upload to asset.
- `video.live_stream.*`: update live stream state and active asset IDs.
- `robots.job.*.completed`: store structured AI outputs.
- `robots.job.*.errored`: store errors for retry or reviewer action.

Use `mux_event_id` uniqueness to avoid processing duplicates.

## Supabase integration notes

Mux offers an `@mux/supabase` package that can initialize a `mux` schema, create an edge function, and backfill existing Mux data.

Operational flow:

1. Run Supabase init if needed.
2. Run `npx @mux/supabase init`.
3. Set `MUX_TOKEN_ID`, `MUX_TOKEN_SECRET`, and `MUX_WEBHOOK_SECRET`.
4. Deploy the webhook function.
5. Configure the Mux Dashboard webhook URL.
6. Backfill existing assets with `npx @mux/supabase backfill`.

The generated `mux` schema blocks anon and authenticated roles by default. Query with service role from server-side code or add explicit RLS policies before client access.

## Indexing strategy

Postgres:

- B-tree indexes for status, owner, visibility, asset ID, upload ID, live stream ID, and created time.
- GIN full-text indexes for title, description, transcript, chapter, and moment text.
- JSONB indexes only for fields queried often.
- Partial indexes for common states such as ready and public videos.

Supabase vector search:

- Use embeddings for semantic queries over transcript chunks, chapters, moments, or summaries.
- Keep chunk IDs and time ranges attached to embeddings.
- Filter by visibility and ownership before or during vector retrieval.

## Access control

- Keep Mux API credentials and Supabase service role keys server-side.
- Enforce app visibility separately from Mux playback policy.
- Public search can return public asset metadata and public playback IDs.
- Private search can return metadata to authorized users but must generate signed playback URLs server-side.
- Avoid storing PII in Mux asset metadata because some metadata may be browser-visible.
