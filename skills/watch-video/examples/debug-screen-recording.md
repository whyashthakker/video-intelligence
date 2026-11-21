# Example: Watching & Debugging a Local Screen Recording

## User Prompt
> "Watch `./checkout-bug.mp4` and tell me every step before the payment button breaks, including the exact error message."

## Step 1 — Create a Chat & Upload the Video
1. `create_chat({ title: "Checkout bug repro" })` → `chat_id: "7f4c...1a"`
2. If using `ceptory-mcp` stdio bridge:
   `upload_video_file({ chat_id: "7f4c...1a", file_path: "./checkout-bug.mp4" })`
   *(Or over HTTP MCP: `create_upload_url` → `curl -X PUT` → `complete_upload`.)*
3. Poll `get_chat_videos({ chat_id: "7f4c...1a" })` until `status === "READY"` → `video_id: "vid_9a2..."`.

## Step 2 — Read the Timeline First (`get_video`)
`get_video({ chat_id: "7f4c...1a", video_id: "vid_9a2..." })`

Returns indexed segments (e.g., `0–14s` cart page, `14–32s` shipping form, `32–48s` card entry and red banner error, `48–55s` page reload).

## Step 3 — Pinpoint the Failure Moment (`search_videos` + `ask_video`)
```json
{
  "tool": "search_videos",
  "args": {
    "chat_id": "7f4c...1a",
    "video_id": "vid_9a2...",
    "query": "red error banner or failed payment toast after clicking pay"
  }
}
```
Hit returned: `38.0s–44.0s` (score `0.91`).

Follow up with:
```json
{
  "tool": "ask_video",
  "args": {
    "video_id": "vid_9a2...",
    "question": "List every user click, form field entered, and visible error state between 00:00 and 00:50 with timestamps."
  }
}
```

## Step 4 — Zoom Into the Exact Frame for Tiny UI Text
Because toast banners or console stack traces can be small, pull the frame at `40s`:
```bash
ffmpeg -ss 00:00:40 -i ./checkout-bug.mp4 -frames:v 1 -q:v 2 crash_40s.jpg
```
Inspect `crash_40s.jpg` to read the exact HTTP status / error string verbatim, then deliver a chronological repro report with exact `MM:SS` timestamps.
