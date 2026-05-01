# Overshoot Stream URI Reference

Use this reference for `ovs://` stream references inside OpenAI-compatible chat completions.

## Mental model

Overshoot chat completions accept normal message content plus stream references in `image_url` or `video_url` parts:

```text
ovs://streams/{stream_id}?<query>
```

The `ovs://` URI is not fetchable. Overshoot parses the `stream_id` and query server-side, then resolves retained frames internally.

## Latest-frame question

Use `image_url` for a single frame. The common "right now" reference is:

```json
{
  "type": "image_url",
  "image_url": {
    "url": "ovs://streams/{stream_id}?frame_index=-1"
  }
}
```

Image references require exactly one anchor:

| Param | Meaning |
| --- | --- |
| `frame_index` | Lifetime frame index. Negative values are relative to the live edge; `-1` is latest. |
| `timestamp_ms` | Absolute stream-clock ms since the first frame. |
| `offset_ms` | Offset from now at request time. Negative values are in the past. |

Optional image controls:

- `tolerance_ms`: positive integer snap tolerance, default `100`; ignored with `frame_index`.
- `direction`: `nearest`, `forward`, or `backward`; default `nearest`; ignored with `frame_index`.

## Last-N-seconds analysis

Use `video_url` for a window of frames. The common "last 5 seconds" reference is:

```json
{
  "type": "video_url",
  "video_url": {
    "url": "ovs://streams/{stream_id}?start_offset_ms=-5000"
  }
}
```

Video references require exactly one start anchor:

| Param | Meaning |
| --- | --- |
| `start_frame_index` | Lifetime frame index where the segment begins. |
| `start_timestamp_ms` | Stream-clock ms where the segment begins. |
| `start_offset_ms` | Offset from now where the segment begins; negative means past. |

Video references accept at most one optional end anchor:

| Param | Meaning |
| --- | --- |
| `end_frame_index` | Lifetime frame index where the segment ends. |
| `end_timestamp_ms` | Stream-clock ms where the segment ends. |
| `end_offset_ms` | Offset from now where the segment ends. |

If no end anchor is supplied, the segment ends at the live edge.

Optional video control:

- `max_fps`: positive float, default `1.0`. Lowering `max_fps` reduces sampled frames and visual tokens.

Start and end anchor types may differ, for example `start_offset_ms=-30000&end_frame_index=523`.

## Request examples

Single latest-frame request:

```json
{
  "model": "Qwen/Qwen3.5-9B",
  "messages": [{
    "role": "user",
    "content": [
      { "type": "text", "text": "What is the person doing right now?" },
      { "type": "image_url", "image_url": {
        "url": "ovs://streams/{stream_id}?frame_index=-1"
      }}
    ]
  }]
}
```

Last-5-seconds request:

```json
{
  "model": "google/gemma-4-E4B-it",
  "messages": [{
    "role": "user",
    "content": [
      { "type": "text", "text": "Did anything important happen in the last 5 seconds?" },
      { "type": "video_url", "video_url": {
        "url": "ovs://streams/{stream_id}?start_offset_ms=-5000&max_fps=1"
      }}
    ]
  }]
}
```

Bounded segment request:

```json
{
  "type": "video_url",
  "video_url": {
    "url": "ovs://streams/{stream_id}?start_timestamp_ms=10000&end_timestamp_ms=20000&max_fps=2"
  }
}
```

## Resolution and validation rules

- Negative `frame_index`, `offset_ms`, `start_offset_ms`, and `end_offset_ms` are evaluated at request time against the live edge.
- Old `frame_index` or `start_frame_index` values clamp up to `first_available_frame_index`; the request can succeed on the oldest retained frame.
- Future frame references fail with `422`; Overshoot does not wait for future frames.
- Duplicate query keys fail with `422`.
- Multiple anchors of the same kind fail with `422`, such as `frame_index` plus `timestamp_ms`, or two `start_*` anchors.
- Use `GET /v1/streams/{stream_id}` when debugging frame windows: compare `first_available_frame_index`, `last_frame_index`, `stream_time_ms`, and `last_frame_at_ms`.

## Cost and latency controls

- Prefer `image_url` for instantaneous state or object checks.
- Prefer short `video_url` windows for motion, actions, and "did anything happen" questions.
- Lower `max_fps` first when a video segment is too slow or expensive.
- Reduce camera resolution or publisher frame rate when visual tokens dominate prompt size.
- Keep prompts explicit about the time window: "right now", "last 5 seconds", or "between 10s and 20s".
