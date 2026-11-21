---
name: media-shot-logging
description: Build HH:MM:SS timecoded shot logs, extract verbatim spoken pull quotes, rank top highlight moments, generate 60-second edit cut orders, and standardize archive tags from long recordings (events, matches, interviews, webinars) using video search and analysis via MCP. Use whenever a user wants to log raw footage without scrubbing, find spoken quotes or crowd reactions, or produce an edit decision sheet.
license: MIT
metadata:
  version: "1.0"
---

# Media Shot Logging, Highlight Extraction & Archive Search

Turn long unedited recordings—events, sports, interviews, product demos, webinars, and archives—into `HH:MM:SS` shot logs, verbatim pull quotes, and ready-to-cut highlight orders without scrubbing.

## Workflow

1. **Ingest full recordings**: Upload the full program feed (up to 2 GB / ~1 hour per file; split multi-hour events into numbered parts in the same chat/index so they stay searchable together). Preserve clean audio for accurate quote retrieval. Poll `get_chat_videos` until `READY`.
2. **Search high-recall cues (`search_videos` + `get_video`)**:
   - **Search reactions, not only events**: `"crowd cheering and standing up"`, `"audience laughing and clapping"`.
   - **Search framing + visuals**: `"close-up of product being demonstrated"`, `"wide shot of stage with slides"`.
   - **Search verbatim speech hooks**: `"speaker announces launch date"`, `"host asks about pricing"`.
3. **Generate the structured log (`analyze_video`)**: Read [references/shot-log-prompts.md](references/shot-log-prompts.md) for the assistant-editor prompt, channel cut-length presets (6–15s vertical social, 20–30s recap/trailer), and archive tag taxonomy.
4. **Aggregate multi-part recordings**: For split recordings, run `analyze_video` per `video_id` and merge into a single master Edit Decision Note (`Source file`, `Timecode in`, `Timecode out`, `Title`, `Quote / Visual`, `Target format`).

## Guardrails

- **The tools log and play back moments; they do not render final edited video.** Deliver crisp `HH:MM:SS` timecodes, cut orders, and optional `ffmpeg -ss ... -to ... -c copy` rough-cut commands.
- **No fabricated names or quotes**: Only name a speaker if their name is spoken aloud or visibly shown on screen; otherwise label by role (`Host`, `Speaker 1`, `Panelist`).

## References & Examples

- Read [references/shot-log-prompts.md](references/shot-log-prompts.md) for the full logging prompt, social/recap presets, and archive taxonomy.
- Read [examples/sample-shot-log-and-cutlist.md](examples/sample-shot-log-and-cutlist.md) for a complete output template with `ffmpeg` rough-cut commands.
