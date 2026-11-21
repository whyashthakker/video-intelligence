---
name: watch-video
description: Give any LLM or AI agent vision over local or remote video files (MP4, MOV, WebM). Upload or link a screen recording, bug repro, CCTV clip, drone flight, or product demo, read its chronological visual and audio timeline, search for exact moments in plain English, ask grounded visual questions, and zoom into individual frames with ffmpeg or vision detectors via MCP. Use whenever a user asks an agent to watch a video, look at a video, debug a screen recording, or explain what happens in footage.
---

# Watch & Inspect Video With Any LLM

Most LLMs cannot natively watch long video files without blowing through context limits or missing transient visual and spoken details. This skill gives any MCP-connected LLM a fast, accurate, token-efficient protocol to "watch" and reason over any video through a video intelligence MCP server.

## Prerequisites

- A video intelligence MCP server connected (e.g. `https://mcp.ceptory.com/mcp` via OAuth 2.1, or `CEPTORY_TOKEN=<token> npx -y ceptory-mcp` for direct local file uploads) — see the `video-mcp-setup` skill or [ceptory.com/mcp](https://www.ceptory.com/mcp).
- Or a REST API key (`https://www.ceptory.com/api/v1`) for scriptable pipelines.
- Free workspaces include starter video indexing minutes; check balance anytime with `get_account`.

## The 5-Step "Watch a Video" Protocol

### Step 1: Ingest the Video
Create or reuse a video chat (`create_chat({ title })`), then add the video:
- **Public HTTPS URL** (direct `.mp4`/`.mov`/`.webm` or presigned cloud URL):
  ```json
  {
    "tool": "add_video_from_url",
    "args": {
      "chat_id": "<chat_uuid>",
      "url": "https://example.com/demo-recording.mp4",
      "file_name": "demo-recording.mp4"
    }
  }
  ```
- **Local video file (up to 2 GB)**:
  - If running the `ceptory-mcp` stdio bridge: call `upload_video_file` directly with the local path.
  - Over remote Streamable HTTP MCP:
    1. Call `create_upload_url({ chat_id, file_name, mime_type: "video/mp4", file_size_bytes })`.
    2. Upload bytes: `curl -X PUT -H "Content-Type: video/mp4" --upload-file /path/to/video.mp4 "<presignedUrl>"`.
    3. Call `complete_upload({ chat_id, storage_key, file_name, mime_type: "video/mp4", file_size_bytes })`.

Poll `get_chat_videos({ chat_id })` until `status` is `READY` (if `FAILED`, call `retry_video` — retries are not billed again).

### Step 2: Read the Chronological Timeline (`get_video`)
Before asking open-ended questions, call `get_video({ chat_id, video_id })`.
- Returns the indexed timeline of segments with `startSec`, `endSec`, and what happens visually and audibly in each slice.
- Gives the LLM an immediate map of the entire recording—even for 30–60 minute videos—in a fraction of the tokens raw frames would cost.

### Step 3: Jump Directly to Relevant Moments (`search_videos`)
When looking for a specific action, bug, object, quote, or on-screen text:
```json
{
  "tool": "search_videos",
  "args": {
    "chat_id": "<chat_uuid>",
    "video_id": "<optional video_id>",
    "query": "error toast appears after clicking checkout button",
    "from_sec": 0,
    "to_sec": 600
  }
}
```
- Searches visual scenes, spoken audio, and on-screen text simultaneously.
- Returns up to 6 reranked timestamped hits (`startSec`, `endSec`, `score`, `snippet`).

### Step 4: Ask Visual Questions & Synthesize (`ask_video` / `analyze_video`)
- **Targeted Q&A (`ask_video`)**: Ask a concrete question about one video (`ask_video({ video_id, question })`). The server inspects the relevant video clips and returns a grounded answer with the exact time ranges it watched.
- **Full structured breakdown (`analyze_video`)**: Pass `chat_id`, `video_id`, and a structured `prompt` (up to 4,000 chars) to extract step-by-step timelines, defect lists, scene changes, or executive summaries.

### Step 5: Optional Pixel-Level Frame Inspection (When Exact Details Matter)
If the user needs exact UI text, fine print, small defects, or visual object boxes at a timestamp the search identified:
1. Extract the exact frame locally with `ffmpeg`:
   ```bash
   ffmpeg -ss 00:01:42 -i /path/to/video.mp4 -frames:v 1 -q:v 2 frame_102s.jpg
   ```
2. Either inspect the extracted image directly in a multimodal LLM turn, OR run a specialized detector via `run_engine({ engine_id, image_base64, annotate: true })` (discover available detectors with `list_engines`) to get structured bounding boxes and an annotated JPEG.

## High-Value Use Cases

| Video Type | Best Query & Analysis Pattern |
|---|---|
| **Bug repro / screen recording** | Call `get_video` → `search_videos("error message or modal dialog")` → `ask_video("List every click, URL, and visible error state before the crash with timestamps")` → `ffmpeg` zoom on the crash frame. |
| **CCTV / security clip** | `search_videos` separately for entry, action, exit, and vehicles → `ask_video` for clothing, direction of travel, and timestamps (never guess identities). |
| **Product demo / user interview** | `analyze_video` with a prompt requesting timestamped feature moments, friction points, and verbatim user quotes. |
| **Drone / field inspection** | `search_videos("damaged roof panel or standing water")` → `ask_video` per anomaly → `run_engine` for hazard/PPE/vehicle counts on key frames. |
| **Multi-video comparison** | Upload multiple clips to the same chat, run `search_videos` across all of them, then call `ask_video` per matched `video_id`. |

## Accuracy & Grounding Guardrails

- **Always cite timestamps (`MM:SS` or `HH:MM:SS`)** returned by the tools so the user can verify every claim in seconds.
- **Never hallucinate unseen details**: If a label, plate, or screen element is unreadable due to resolution or motion blur, say so explicitly and mark it `"unconfirmed"`.
- **Mind workspace balance**: Indexing, search, analysis, and engine calls use workspace allowance/prepaid balance (`get_account`); confirm before indexing very large batches of long videos.
- **MCP server docs**: [ceptory.com/mcp](https://www.ceptory.com/mcp)
