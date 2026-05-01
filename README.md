# OSH Skills

Codex skills for Overshoot, Mux, video search, and moderation workflows.

## Included skills

- `overshoot`: Overshoot realtime vision streams, LiveKit publishing, and `ovs://` model queries.
- `mux-ai`: Mux Robots API, captions, transcripts, chapters, key moments, and AI enrichment.
- `supasearch`: searchable Mux video catalogs over metadata, transcripts, chapters, and moments.
- `mux-content-moderation`: Mux Robots moderation, policy gating, thresholds, and review queues.

## Install

Copy the skill folders into your Codex skills directory:

```sh
cp -R skills/* ~/.codex/skills/
```

Restart Codex or start a new session so the skill metadata is discovered.

## Validation

Each skill was created with the Codex `skill-creator` initializer and validated with `quick_validate.py`.
