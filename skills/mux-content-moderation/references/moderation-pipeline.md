# Mux Moderation Pipeline Reference

Use this reference when implementing Trust and Safety workflows around Mux-hosted assets.

## Robots moderate workflow

Endpoint workflow: `moderate`

Create a job:

```http
POST /robots/v0/jobs/moderate
```

Example parameters:

```json
{
  "asset_id": "YOUR_ASSET_ID",
  "thresholds": {
    "sexual": 0.7,
    "violence": 0.8
  },
  "max_samples": 20
}
```

Authentication uses Mux HTTP Basic Auth with an access token that includes `robots:*` scope.

## Parameters

- `asset_id`: required Mux asset ID.
- `language_code`: for transcript analysis on audio-only assets; defaults to `en`.
- `thresholds.sexual`: score threshold from 0.0 to 1.0; default 0.7.
- `thresholds.violence`: score threshold from 0.0 to 1.0; default 0.8.
- `sampling_interval`: seconds between sampled thumbnails; minimum 5.
- `max_samples`: maximum thumbnails to sample, distributed across the video with first and last frames pinned.

Lower thresholds are stricter and increase human-review volume.

## Outputs

Completed jobs include:

- `thumbnail_scores[]`: per-thumbnail `sexual` and `violence` scores, with `time` for video assets.
- `max_scores.sexual`
- `max_scores.violence`
- `exceeds_threshold`

Persist raw outputs, threshold config, and final app decision. Threshold changes later should not make historical decisions impossible to audit.

## Webhooks and state

Use Robots webhooks:

- `robots.job.moderate.pending`
- `robots.job.moderate.processing`
- `robots.job.moderate.completed`
- `robots.job.moderate.errored`
- `robots.job.moderate.cancelled`

Use an idempotent handler:

1. Verify webhook signature.
2. Persist event by unique event ID.
3. Upsert moderation job state.
4. Apply app decision only once per job version.

## Review decisions

Map scores to product actions. Example:

- Below thresholds: allow or publish.
- Exceeds threshold by small margin: human review.
- Exceeds strict threshold: block or keep private pending appeal.
- Errored job: hold unpublished or retry, depending on product risk.

Store reviewer decisions separately from machine scores:

- `pending_review`
- `approved`
- `rejected`
- `age_gated`
- `limited_distribution`
- `appealed`
- `overturned`

## Strict mode

For high-risk products:

```json
{
  "thresholds": {
    "sexual": 0.3,
    "violence": 0.4
  },
  "max_samples": 50
}
```

Use strict mode with human review capacity planning, because false positives will rise.

## Database fields

Minimum moderation record:

- `id`
- `mux_asset_id`
- `robots_job_id`
- `workflow`
- `status`
- `thresholds`
- `sampling_interval`
- `max_samples`
- `max_scores`
- `exceeds_threshold`
- `thumbnail_scores`
- `decision`
- `reviewer_id`
- `reviewed_at`
- `created_at`
- `updated_at`
- `error`

For queue triage, index by decision, status, max scores, created time, owner, and visibility.
