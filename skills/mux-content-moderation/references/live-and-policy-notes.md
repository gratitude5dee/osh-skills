# Live and Policy Notes

Use this reference for moderation caveats outside simple VOD thumbnail scoring.

## Live stream caveat

Mux Robots moderation is job-oriented around Mux assets. For live real-time enforcement, do not assume Robots can block unsafe content before viewers see it.

Possible live strategies:

- Moderate the completed live recording before making it available as VOD.
- Run a delayed broadcast buffer with an external real-time moderation system.
- Sample frames or audio outside Mux before or during ingest.
- Use human operators for high-risk live events.
- Disable or complete streams based on out-of-band policy signals.

Mux live webhooks help with stream state, not policy classification.

## Transcript-aware policy checks

Robots `moderate` focuses on sexual and violence scores from sampled thumbnails. For other policies, combine:

- Generated captions or transcripts.
- `ask-questions` with constrained answer options.
- Keyword or rule checks.
- Human review for ambiguous context.

Good `ask-questions` examples:

- "Does this video contain medical claims?" with `["yes", "no"]`.
- "Is a required sponsor disclosure spoken or visible?" with `["yes", "no"]`.
- "What age rating best fits this video?" with `["general", "teen", "mature", "unknown"]`.
- "Does this content show dangerous instructions?" with `["yes", "no", "unclear"]`.

Always handle `skipped`, null answers, low confidence, and reasoning.

## Privacy and metadata

- Do not put PII in Mux `meta` fields that may become browser-visible.
- Avoid exposing reviewer notes or moderation labels to public clients unless intended.
- Keep audit logs for decisions that affect user accounts or publishing.

## Secure playback during review

For unreviewed or flagged content:

- Use signed playback IDs.
- Generate short-lived playback JWTs server-side.
- Consider playback restrictions for web embed domains.
- Do not publish public playback IDs until policy permits public access.

## Human review queue

A practical queue item should show:

- Video thumbnail and playback preview.
- Asset ID and app video ID.
- Max sexual and violence scores.
- Highest-risk timestamp.
- Transcript snippets around reported moments if available.
- Creator, upload time, prior decisions, and policy history.
- Clear action buttons for approve, reject, age-gate, limited distribution, and escalate.

## Acceptance tests

- A ready safe asset publishes after a completed non-threshold-exceeding job.
- An exceeding-threshold job creates a human review item and blocks public publishing.
- An errored job does not accidentally publish high-risk content.
- Duplicate webhooks do not duplicate queue items.
- Threshold changes are recorded with each job.
- Private assets never return public playback URLs during review.
