---
name: supasearch
description: Design video search and retrieval over Mux assets using metadata, transcripts, captions, Robots API outputs, chapters, key moments, thumbnails, Supabase/Postgres search, and semantic indexing. Use when building searchable Mux video catalogs, media libraries, AI video retrieval, transcript search, moment search, or Supabase-backed video discovery.
---

# Supasearch

Use this skill to design search over Mux-backed video libraries. Treat Supasearch as a video search layer that combines Mux resource sync, app metadata, transcripts, generated AI outputs, and retrieval indexes.

## Workflow

1. Define the search surface: titles, descriptions, tags, creator IDs, external IDs, playback IDs, transcript text, chapter titles, key moments, moderation labels, and app-level visibility.
2. Sync Mux resources first. Use webhooks to keep assets, uploads, live stream recordings, tracks, and playback IDs current in the application database.
3. Normalize media-derived text:
   - Captions and transcripts for spoken content.
   - Robots summaries, tags, chapters, answers, and key moments for structured search facets.
   - App metadata for ownership, visibility, policy, and editorial curation.
4. Pick retrieval layers by need:
   - Exact filters for status, owner, visibility, duration, tags, and policy.
   - Postgres full-text search for titles, descriptions, transcripts, and chapter text.
   - Embeddings or vector search for semantic video and moment retrieval.
5. Return playable results safely. Only expose public playback URLs directly; signed playback IDs require server-generated JWTs. Use thumbnails, storyboards, or timecoded Mux Player links for previews.
6. For moment search, preserve time ranges from transcript cues, chapters, or key moments so results can jump to the relevant segment or create instant clips.

## References

- For current Mux source material, start with `https://www.mux.com/llms.txt` and load only the relevant `.txt` or `.md` docs.
- Read `references/search-architecture.md` for indexing strategy, retrieval design, and result shaping.
- Read `references/mux-supabase-schema.md` for Mux database tables, webhook sync, and Supabase integration notes.

## Output Standard

For a Supasearch plan, include:

- Query types supported: keyword, semantic, filter, transcript, chapter, moment, and hybrid search.
- Tables or indexes to create, including ownership and visibility gates.
- Sync events and backfill strategy.
- Result schema with asset ID, playback policy, display metadata, thumbnail, matched text, time range, and confidence or ranking score.
- Security rules for private assets, signed URLs, service-role database access, and PII in metadata.
