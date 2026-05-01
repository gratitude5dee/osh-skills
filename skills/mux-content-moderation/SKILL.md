---
name: mux-content-moderation
description: Design content moderation workflows for Mux-hosted video using Mux Robots moderate jobs, thresholds, thumbnail sampling, transcripts, webhooks, policy gating, and human review. Use when building or reviewing Trust and Safety, NSFW or violence detection, pre-publish approval, post-publish takedown, age gating, creator review queues, or moderation pipelines for Mux VOD and live recordings.
---

# Mux Content Moderation

Use this skill to design moderation and review workflows for Mux-hosted video. Start from product policy, then map Mux signals into allow, block, review, age-gate, or takedown actions.

## Workflow

1. Define policy before implementation. Name the categories, thresholds, user-facing outcomes, reviewer states, retention rules, and appeal path.
2. Create or wait for a Mux asset. Use `video.asset.ready` before running job-based moderation unless the product has separately validated active live recording behavior.
3. Run a Robots `moderate` job for visual moderation. Tune `thresholds.sexual`, `thresholds.violence`, `sampling_interval`, and `max_samples` based on product risk and cost.
4. Persist the full moderation result: job ID, asset ID, threshold config, thumbnail scores, max scores, exceeds-threshold flag, status, errors, and units consumed.
5. Gate publication from stored moderation state, not from the raw Mux asset state alone.
6. Escalate ambiguous results to human review. Lower thresholds and denser sampling increase sensitivity but also false positives.
7. Use captions, transcripts, and `ask-questions` for policies outside thumbnail sexual or violence scoring, such as brand rules, disclosure checks, medical claims, or custom taxonomy.
8. For live real-time enforcement, design a separate delayed, buffered, or external frame/audio moderation path. Mux Robots moderation is job-oriented around assets.

## References

- For current Mux source material, start with `https://www.mux.com/llms.txt` and load only the relevant `.txt` or `.md` docs.
- Read `references/moderation-pipeline.md` for Robots moderation parameters, output shape, webhook events, and review queue design.
- Read `references/live-and-policy-notes.md` for live stream caveats, transcript-aware policy checks, and secure playback considerations.

## Output Standard

For a moderation plan, include:

- Product policy categories and decision outcomes.
- Asset and job state machines.
- Threshold defaults and stricter-mode changes.
- Reviewer queue schema and escalation criteria.
- Webhook events, idempotency, and retry behavior.
- Launch checks: false positive sampling, false negative review, abuse response, signed playback, and audit logs.
