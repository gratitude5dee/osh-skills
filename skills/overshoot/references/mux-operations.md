# Mux Operations Reference

Use this reference for live stream reliability, ingest setup, captions, simulcast, and operational health.

## Ingest options

Common RTMP/RTMPS ingest:

- Standard RTMP: `rtmp://global-live.mux.com:5222/app`
- Secure RTMPS: `rtmps://global-live.mux.com:443/app`
- Some encoders ask for one URL: `rtmp://global-live.mux.com:5222/app/{STREAM_KEY}`.

SRT ingest:

- URL format: `srt://global-live.mux.com:6001?streamid={STREAM_KEY}&passphrase={SRT_PASSPHRASE}`
- Encoder mode should be caller when required.
- Mux exposes `srt_passphrase` on live stream resources.
- SRT can support HEVC ingest, but simulcast destinations must support the source codec.

Regional ingest URLs exist for U.S. East, U.S. West, and Europe when manual routing or failover is needed.

## Encoder recommendations

- Video codec: H.264 Main Profile.
- Audio codec: AAC.
- Keyframe interval: usually 2 seconds.
- 1080p30 start point: about 5000 kbps.
- 720p30 start point: about 3500 kbps.
- Keep encoder bitrate below available upload bandwidth; roughly half is a safer starting point.
- Prefer constant bitrate for live reliability.

## Reconnects and slates

- `reconnect_window` controls how long Mux waits for an unexpected reconnect before finalizing the stream.
- Default is commonly 60 seconds for standard latency and 0 for lower latency modes, but can be configured up to 1800 seconds.
- `reconnect_slate_url` can provide a slate image during disconnects.
- A custom slate must be downloadable when recording starts; otherwise Mux falls back and sends warning webhooks.
- `signal live stream complete` finalizes faster but does not instantly close encoder connections.
- `disable live stream` stops accepting encoder connections.

## Live health signals

Use live health data to explain streamer-facing problems:

- Excellent: stream drift deviation at or below 500 ms.
- Good: at or below 1 second but greater than 500 ms.
- Poor: greater than 1 second.
- Unknown: inactive or insufficient recent data.

Common issue patterns:

- High video bitrate variance: reduce bitrate and use constant bitrate.
- Intermittent loss: improve network reliability and reduce competing bandwidth.
- Spiky audio/video bitrate: encoder may be CPU-bound.
- Spiky frame rate: unstable network, encoder settings, or hardware limits.

## Simulcast

- Simulcast targets can be added when the live stream is not active.
- Each target needs a URL and stream key from the destination platform.
- Common destinations include YouTube, Facebook Live, Twitch, Crowdcast, and Vimeo.
- Instagram is not supported through generic RTMP simulcast.
- Treat third-party stream keys as secrets.

## Live captions

Manual embedded live captions:

- Mux supports CEA-608 embedded captions for a single language.
- Configure `embedded_subtitles` before the stream is active.
- Live caption changes require the stream to be idle.

Auto-generated live captions:

- Supported languages include English, Spanish, Italian, Portuguese, German, and French.
- Use transcription vocabularies for product names and technical terms.
- Live captions currently do not apply to low-latency live streams in the cited Mux guide.

## Local development helpers

- Mux CLI can create assets, uploads, live streams, signing keys, and signed URLs.
- CLI webhook forwarding can replay or trigger local webhook events, but it is for local development only.
- Mux MCP can expose Mux Video and Data APIs to AI tools when configured.
