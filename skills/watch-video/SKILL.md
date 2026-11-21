---
name: watch-video
description: Give any LLM or AI agent vision over local or remote video files (MP4, MOV, WebM). Upload or link a screen recording, bug repro, CCTV clip, drone flight, or product demo, read its chronological visual and audio timeline, search for exact moments in plain English, ask grounded visual questions, and zoom into individual frames with ffmpeg or vision detectors via MCP. Use whenever a user asks an agent to watch a video, look at a video, debug a screen recording, or explain what happens in footage.
license: MIT
metadata:
  version: "1.0"
---

# Watch & Inspect Video With Any LLM

Give any MCP-connected LLM fast, token-efficient vision over long videos without dumping raw video frames into context.

## 5-Step "Watch a Video" Protocol

1. **Ingest the video**:
   - Create or reuse a chat (`create_chat({ title })`).
   - **HTTPS URL**: call `add_video_from_url({ chat_id, url, file_name, duration_sec })`.
   - **Local file (up to 2 GB)**: call `upload_video_file` (via `ceptory-mcp` stdio bridge), or run `create_upload_url` → `curl -X PUT` → `complete_upload`.
   - Poll `get_chat_videos({ chat_id })` until `status === "READY"` (retry `FAILED` via `retry_video` — free).
2. **Read the chronological timeline first**: Call `get_video({ chat_id, video_id })` to map visual and spoken segments across the entire recording in minimal tokens.
3. **Jump to exact moments**: Call `search_videos({ chat_id, query, video_id, from_sec, to_sec })` using visual, spoken, or on-screen text descriptions.
4. **Ask grounded visual questions**:
   - Use `ask_video({ video_id, question })` for focused Q&A with returned time ranges.
   - Use `analyze_video({ chat_id, video_id, prompt })` for full structured breakdowns (summaries, repro steps, defect tables).
5. **Zoom into exact frames only when pixel detail matters**:
   - Read [references/ffmpeg-frame-extraction.md](references/ffmpeg-frame-extraction.md) when pulling crisp frames, bursts, or contact sheets at returned timestamps.
   - Optionally call `run_engine({ engine_id, image_base64, annotate: true })` (discover IDs via `list_engines`) to get bounding boxes on a specific frame.

## References & Examples

- Read [references/ffmpeg-frame-extraction.md](references/ffmpeg-frame-extraction.md) when extracting frames for tiny UI text, stack traces, or `< 8 MB` detector inputs.
- Read [examples/debug-screen-recording.md](examples/debug-screen-recording.md) for a worked trace of debugging a local `.mp4` screen recording.

## Grounding Guardrails

- Always cite exact `MM:SS` or `HH:MM:SS` timestamps returned by the tools.
- Never invent unreadable text, plates, or speaker names — mark unclear details `"unconfirmed"`.
- Check `get_account` before indexing large batches of long videos.
