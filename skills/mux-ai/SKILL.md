---
name: mux-ai
description: Design AI workflows for Mux-hosted video using the Mux Robots API, captions, transcripts, structured job outputs, webhook delivery, summarization, chapter generation, ask-questions, key moments, and translated captions. Use when building or reviewing Mux video analysis, metadata generation, transcript processing, video understanding, or AI enrichment pipelines.
---

# Mux AI

Use this skill to turn Mux assets into structured AI outputs. Prefer Mux Robots API for batteries-included video understanding, and use captions or transcripts when custom processing is needed.

## Workflow

1. Start with a Mux asset ID. If the video is still `preparing`, wait for `video.asset.ready`.
2. Add or verify captions before transcript-heavy workflows. Generated captions can be enabled during asset creation, direct upload creation, or later from an audio track.
3. Pick the smallest Robots workflow that satisfies the task:
   - `summarize`: title, description, and tags.
   - `generate-chapters`: timestamped table of contents.
   - `ask-questions`: constrained classification or extraction.
   - `find-key-moments`: highlight candidates with time ranges and rationale.
   - `translate-captions`: translated VTT track creation.
4. Create one job per workflow and use Robots webhooks for completion. Poll only for low-volume prototypes.
5. Store job IDs, workflow, parameters, status, outputs, errors, units consumed, and related asset IDs.
6. Keep prompt overrides narrow and product-specific. Use them to enforce output style, vocabulary, title rules, chapter density, or controlled tags.
7. If the request is specifically about Trust and Safety, NSFW detection, violence thresholds, or publish gating, use `$mux-content-moderation` instead.

## References

- For current Mux source material, start with `https://www.mux.com/llms.txt` and load only the relevant `.txt` or `.md` docs.
- Read `references/robots-api.md` for Robots job lifecycle, workflow parameters, webhook events, and output shapes.
- Read `references/captions-transcripts.md` for generated captions, track readiness, transcript URLs, and translation prerequisites.

## Output Standard

For a Mux AI plan, include:

- Required input asset state and caption requirements.
- Exact Robots workflow endpoint names and parameters.
- Webhook events to listen for and database fields to persist.
- Output consumption plan: where title, tags, chapters, answers, moments, tracks, and temporary URLs go.
- Failure behavior for missing captions, skipped answers, errored jobs, expired URLs, and delayed caption generation.
