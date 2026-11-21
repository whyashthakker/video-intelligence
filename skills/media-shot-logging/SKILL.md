---
name: media-shot-logging
description: Build HH:MM:SS timecoded shot logs, extract verbatim spoken pull quotes, rank top highlight moments, generate 60-second edit cut orders, and standardize archive tags from long recordings (events, matches, interviews, webinars) using MCP video search and analysis. Use whenever a user wants to log raw footage without scrubbing, find spoken quotes or crowd reactions, or produce an edit decision sheet.
---

# Media Shot Logging, Highlight Extraction & Archive Search

Turn long unedited recordings—events, sports matches, interviews, product demos, webinars, and broadcast archives—into timecoded shot logs, verbatim pull quotes, and ready-to-cut highlight plans without manual scrubbing.

## Ingestion Best Practices for Media Workflows

1. **Upload full recordings, not pre-cuts**: Keep the full program feed (and separate isolated cameras if available) in one dedicated chat/index per production or series so moments aren't lost before search.
2. **File sizing**: Each upload supports up to **2 GB** (up to ~1 hour per recording recommended; split longer multi-hour events into numbered parts like `keynote-part1.mp4`, `keynote-part2.mp4` inside the same index so they remain searchable together).
3. **Preserve clean audio**: The index covers speech alongside visual frames and on-screen text—clean audio makes verbatim quote retrieval dramatically more accurate.
4. **Poll until `READY`**: Use `get_chat_videos` (MCP) or `GET /api/v1/indexes/{indexId}/videos/{videoId}` (REST) before searching or logging.

## High-Recall Media Search Patterns (`search_videos`)

- **Search the reaction, not only the event**: Crowd eruptions, applause, standing ovations, and close-up facial expressions often mark the strongest edit points (`"crowd cheering and standing up"`, `"audience laughing and clapping"`).
- **Combine visual cues with framing**: `"close-up of the product being demonstrated"`, `"wide shot of the stage with slides visible"`, `"speaker pointing at chart on screen"`.
- **Search spoken hooks verbatim**: `"someone says thank you everyone"`, `"speaker announces launch date"`, `"host asks about pricing"`.
- **Use `get_video` for full segment timelines**: Inspect the indexed chronological breakdown before drilling into specific timestamps with `ask_video`.

## Generating a Production Shot Log & Highlight Sheet

Run `analyze_video` (or `POST /api/v1/indexes/{indexId}/analyze`) with this structured assistant-editor prompt (tailor clip lengths to the target channel: **6–15s** for social vertical clips, **20–30s** for recaps/trailers):

> You are an assistant editor logging this recording. Write:
>
> 1. **Shot log**: a table with columns `Timecode in`, `Timecode out`, `Title` (five words or fewer), `Description`, and `Suggested use` (`social clip`, `trailer`, `recap`, `archive only`).
> 2. **Top five moments**, ranked, with one line on why each works.
> 3. **Quotes worth pulling**: verbatim lines with timecodes.
> 4. **60-second highlight cut**: a suggested edit order listing the exact timecodes to use.
> 5. **Archive tags**: topics, settings, and on-screen roles (host, speaker, panelist, player), without guessing personal names unless spoken aloud or shown on a lower-third/slide.
>
> Keep timecodes formatted as `HH:MM:SS` and keep every suggested clip between 5 and 30 seconds.

## Multi-Part Event Aggregation

When an event spans multiple uploaded recordings in the same chat/index:
1. Run `search_videos` across the whole chat/index to surface the highest-scoring moments across all parts.
2. Run `analyze_video` per `video_id` for each key part.
3. Combine the outputs into one master Edit Decision Note with columns `Source file`, `Timecode in`, `Timecode out`, `Title`, `Quote / Visual`, and `Target format`.

## Guardrails & Accurate Capabilities

- **The video tools log and play back moments; they do not render/export edited video timelines.** Deliver clean `HH:MM:SS` timecodes and cut orders that an editor (or downstream `ffmpeg` / NLE script) can cut directly.
- **No fabricated names or quotes**: Only attribute a speaker by name if their name is spoken or visibly displayed on screen; otherwise label by role (`Host`, `Speaker 1`, `Panelist`).
- **Consistent archive taxonomy**: Reuse a controlled vocabulary of topic and setting tags across uploads in the same index so future archive searches remain consistent.
