# OSH Skills

Mux-focused Codex skills for video intelligence workflows.

## Included skills

- `overshoot`: real-time Mux live/video intelligence architecture.
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
