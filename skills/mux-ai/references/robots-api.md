# Mux Robots API Reference

Use this reference when designing Mux AI jobs for structured video understanding.

## Job model

Robots API endpoints live under `/robots/v0/`. Each workflow creates a job.

Common flow:

1. POST to `/robots/v0/jobs/{workflow}` with `parameters`.
2. Wait for job status: `pending`, `processing`, `completed`, `errored`, or `cancelled`.
3. Prefer webhook delivery over polling.
4. Read outputs from the completed job.

Common job fields:

- `id`: job ID.
- `workflow`: workflow name.
- `status`: current lifecycle state.
- `created_at` and `updated_at`: Unix timestamps.
- `passthrough`: caller-provided string returned unchanged.
- `parameters`: workflow-specific input.
- `outputs`: workflow-specific result when completed.
- `units_consumed`: consumed Mux AI units.
- `errors`: details when errored.
- `resources`: related Mux resources.

Authentication uses Mux HTTP Basic Auth with a token ID and token secret. The access token must include `robots:*` scope.

## Webhooks

Robots webhooks follow:

`robots.job.{workflow}.{status}`

Examples:

- `robots.job.summarize.completed`
- `robots.job.generate_chapters.completed`
- `robots.job.ask_questions.completed`
- `robots.job.find_key_moments.completed`
- `robots.job.translate_captions.completed`

Multi-word workflow names use underscores in webhook event types even when API paths use hyphens. The webhook `data` contains the full job object, including outputs.

## Summarize

Endpoint workflow: `summarize`

Use for:

- CMS title and description generation.
- Tags for search and discovery.
- Social or product metadata.

Key parameters:

- `asset_id` required.
- `tone`: `neutral`, `playful`, or `professional`.
- `title_length`, `description_length`, and `tag_count`.
- `language_code` for the caption track.
- `output_language_code`.
- `prompt_overrides` for task, title, description, keywords, and quality guidelines.

Outputs:

- `title`
- `description`
- `tags`

Summarization works best when the asset has a text track.

## Generate chapters

Endpoint workflow: `generate-chapters`

Use for:

- Long-form navigation.
- Table of contents.
- Search facets and playback jumps.

Key parameters:

- `asset_id` required.
- `language_code`.
- `output_language_code`.
- `prompt_overrides` for task, output format, chapter guidelines, and title guidelines.

Outputs:

- `chapters[]`
- `chapters[].start_time`
- `chapters[].title`

Chapter generation depends on caption tracks.

## Ask questions

Endpoint workflow: `ask-questions`

Use for:

- Classification.
- Policy or taxonomy checks.
- Structured extraction where answer choices are known.

Key parameters:

- `asset_id` required.
- `questions[]` required.
- `questions[].question`
- `questions[].answer_options`, defaulting to `["yes", "no"]`.
- `language_code`.

Outputs:

- `answers[]`
- `question`
- `answer` or null.
- `skipped`
- `confidence`
- `reasoning`

Handle skipped answers. Confidence above 0.9 is strong; below 0.5 is weak or uncertain.

## Find key moments

Endpoint workflow: `find-key-moments`

Use for:

- Highlight reels.
- Clip candidates.
- Search result jump points.
- Social previews.

Key parameters:

- `asset_id` required.
- `max_moments` from 1 to 10, default 5.
- `target_duration_ms.min` and `target_duration_ms.max`.

Outputs:

- `moments[]`
- `start_ms` and `end_ms`
- `overall_score`
- `title`
- `audible_narrative`
- `notable_audible_concepts`
- `visual_narrative` for video assets.
- `notable_visual_concepts`
- `cues[]` with transcript spans.

This workflow relies on transcript cues.

## Translate captions

Endpoint workflow: `translate-captions`

Use for:

- Multilingual caption tracks.
- Localized accessibility.

Key parameters:

- `asset_id` required.
- `track_id` required.
- `to_language_code` required.
- `upload_to_mux`, default `true`.

Outputs:

- `track_id`
- `uploaded_track_id` when uploaded to Mux.
- `temporary_vtt_url` when available.

Translation requires an existing caption track.
