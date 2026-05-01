# Supasearch Architecture Reference

Use this reference to design video search over Mux assets, transcripts, and AI outputs.

## Search surfaces

Index multiple layers because each answers a different query type:

- Mux asset metadata: status, duration, aspect ratio, video quality, tracks, playback IDs.
- App metadata: title, description, creator, owner, visibility, tags, publish state.
- Captions and transcripts: spoken content and timecoded cues.
- Robots outputs: summaries, tags, chapters, answers, key moments, visual concepts.
- Moderation outputs: policy state and labels, but only expose internally unless product policy allows.
- Images: thumbnails and storyboards for previews, not primary textual search.

## Retrieval layers

Use exact filters for:

- Owner or creator.
- Visibility and publish state.
- Asset status.
- Duration ranges.
- Tags and categories.
- Playback policy.
- Moderation state.

Use full-text search for:

- Titles.
- Descriptions.
- Transcript text.
- Chapter titles.
- Key moment narratives and concepts.

Use vector or semantic search for:

- Natural-language video discovery.
- Similar moment retrieval.
- Concept search where the user does not know exact words.
- Cross-video recommendations.

Use hybrid ranking when both keyword precision and semantic recall matter.

## Timecoded result model

For transcript, chapter, and moment search, return time-aware results:

- `asset_id`
- `playback_id` or signed playback URL generated server-side.
- `match_type`: asset, transcript, chapter, moment, tag, or answer.
- `title`
- `snippet`
- `start_time` and `end_time` when available.
- `thumbnail_url`
- `score`
- `visibility`

For signed playback, do not place JWT-claim parameters in the final URL except `token`. Put time, clipping, thumbnail, or restriction parameters into the signed claims.

## Mux preview URLs

Public thumbnail:

```text
https://image.mux.com/{PLAYBACK_ID}/thumbnail.jpg?time={SECONDS}&width=640
```

Public playback:

```text
https://stream.mux.com/{PLAYBACK_ID}.m3u8
```

Instant VOD clip:

```text
https://stream.mux.com/{PLAYBACK_ID}.m3u8?asset_start_time={SECONDS}&asset_end_time={SECONDS}
```

Use signed playback for private or paid content.

## Query planning

For each new search feature, decide:

1. Which fields are authoritative.
2. Whether the result is an asset result or a moment result.
3. Whether search should include private user-owned content, public catalog content, or both.
4. Whether transcripts can contain sensitive text.
5. Whether AI outputs require review before becoming searchable.

## Common mistakes

- Searching only titles when transcript or chapter search is expected.
- Exposing playback URLs for private assets without signed URLs.
- Indexing assets before `video.asset.ready`.
- Losing timecodes during transcript chunking.
- Treating `asset_id` and `playback_id` as interchangeable.
- Using app-level search without syncing Mux webhooks and asset readiness.
