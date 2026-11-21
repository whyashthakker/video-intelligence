---
name: ceptory-video-search
description: Index recorded videos (from public HTTPS URLs or local files up to 2 GB), search footage in natural language across visual scenes, speech, and on-screen text, ask grounded questions about videos, and run multimodal analysis via Ceptory MCP tools or REST API v1 (/api/v1/indexes). Use whenever a user wants to upload, index, semantically search, query, or summarize recorded video footage in Ceptory.
---

# Ceptory Natural-Language Video Search & Q&A

Index recorded video footage into Ceptory and retrieve exact timestamped moments using natural language, visual + audio RAG Q&A, and structured multimodal analysis.

## Choose Your Interface

- **Interactive agent workflows (MCP)**: Uses video chats (`create_chat`, `add_video_from_url`, `create_upload_url` + `complete_upload`, `search_videos`, `ask_video`, `analyze_video`, `get_video`).
- **Backend / programmatic pipelines (REST API v1)**: Uses `https://www.ceptory.com/api/v1/indexes` with `Authorization: Bearer <API_KEY>` (no client SDK; use standard HTTP / `fetch` / `curl`).

## Workflow A: Via Ceptory MCP

### 1. Create or Pick a Video Chat
- Call `list_chats` to reuse an existing workspace chat, or `create_chat({ title })` to keep one workflow/site per chat (mixing unrelated footage reduces search precision).
- Call `get_account` first if unsure about workspace balance.

### 2. Ingest Video Footage
- **From a public HTTPS URL** (direct file or presigned S3/GCS URL):
  ```json
  {
    "tool": "add_video_from_url",
    "args": {
      "chat_id": "<uuid>",
      "url": "https://example.com/gate-cam-2026-11-20.mp4",
      "file_name": "gate-cam-2026-11-20.mp4",
      "duration_sec": 600
    }
  }
  ```
- **From a local file (up to 2 GB, MP4/MOV/WebM)**:
  1. Call `create_upload_url({ chat_id, file_name, mime_type: "video/mp4", file_size_bytes })` → returns `presignedUrl` (valid 1 hour) and `storage_key`.
  2. Upload bytes directly to storage:
     ```bash
     curl -X PUT -H "Content-Type: video/mp4" --upload-file ./clip.mp4 "<presignedUrl>"
     ```
  3. Call `complete_upload({ chat_id, storage_key, file_name, mime_type: "video/mp4", file_size_bytes, duration_sec })`.
  *(If using the `ceptory-mcp` stdio bridge, `upload_video_file` performs these steps in one call.)*

### 3. Wait for Indexing
- Poll `get_chat_videos({ chat_id })` until each video moves from `PROCESSING` to `READY`.
- If a video ends in `FAILED`, call `retry_video({ chat_id, video_id })` — retries are **not billed again**.
- Inspect the indexed segment timeline anytime with `get_video({ chat_id, video_id })`.

### 4. Search, Ask & Analyze
- **Semantic moment search (`search_videos`)**: Returns up to 6 reranked matches with `videoId`, `startSec`, `endSec`, `score`, and `snippet`. Supports optional `video_id`, `min_score` (`0`–`1`), `from_sec`, and `to_sec`.
- **Single-video Q&A (`ask_video`)**: Pass `video_id` and `question`; Ceptory inspects relevant clips and answers with the time ranges it used.
- **Full written analysis (`analyze_video`)**: Pass `chat_id`, optional `video_id` (defaults to the latest `READY` video), and a custom `prompt` (up to 4,000 chars). Without a prompt, summarizes scene, people, objects, and events. Can take up to 60 seconds.

## Workflow B: Via REST API v1

Base URL: `https://www.ceptory.com/api/v1`  
Auth: `Authorization: Bearer <API_KEY>` (scopes: `READ` for listing/search, `WRITE` for creating indexes, ingesting videos, and running analyze). Rate limit: **120 requests/minute per key**.

```bash
# 1. Create an index (name max 80 chars, description max 500 chars)
curl -X POST https://www.ceptory.com/api/v1/indexes \
  -H "Authorization: Bearer $CEPTORY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"North Gate Footage","description":"Daily security exports"}'

# 2. Add a video by HTTPS URL (chunked at 2s / 1s overlap / 6 fps + audio)
curl -X POST https://www.ceptory.com/api/v1/indexes/$INDEX_ID/videos \
  -H "Authorization: Bearer $CEPTORY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/clip.mp4","fileName":"dock-1.mp4","durationSec":180}'

# 3. Poll video indexing status
curl https://www.ceptory.com/api/v1/indexes/$INDEX_ID/videos/$VIDEO_ID \
  -H "Authorization: Bearer $CEPTORY_API_KEY"

# 4. Natural-language semantic search (returns top 6 reranked hits)
curl -X POST https://www.ceptory.com/api/v1/indexes/$INDEX_ID/search \
  -H "Authorization: Bearer $CEPTORY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"white delivery van reversing toward dock door 3"}'

# 5. Multimodal analysis on the latest READY video in the index
curl -X POST https://www.ceptory.com/api/v1/indexes/$INDEX_ID/analyze \
  -H "Authorization: Bearer $CEPTORY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"List every vehicle arrival with timestamp, vehicle color, and dock door."}'
```

## Query Phrasing Best Practices

- **Describe appearance + action, not identities**: Use `"person in a red hoodie climbing the fence"` or `"worker without a hard hat near forklift"`, never a person's name unless spoken or printed on screen.
- **Break compound questions into focused searches**: Run separate queries for entry, action, exit, and vehicles, then synthesize with `ask_video` or `analyze_video`.
- **Leverage multimodal indexing**: Speech and on-screen text are indexed alongside visual scenes—search spoken phrases (`"someone says safety stop"`) or visible signage/labels.
- **Keep original file names**: Include camera name and recording start timestamp in `file_name` so returned results trace back cleanly to source footage.

## Pricing & Limits Awareness

- **Metered rates**: `$0.042 / minute` video indexing + `$0.0015 / minute` embedding infrastructure; `$4 / 1,000` search queries (workspaces include a free indexing allowance; check `get_account` or `/pricing`).
- **Limits**: Files up to 2 GB; HTTPS URLs required for URL ingest; `402` returned when prepaid balance is insufficient.
- Always report returned timestamps (`startSec`–`endSec` formatted as `HH:MM:SS`) and clearly distinguish observed facts from anything unconfirmed.
