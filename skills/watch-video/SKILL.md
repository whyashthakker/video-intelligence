---
name: watch-video
description: Let any LLM or AI agent actually watch a video (MP4, MOV, WebM) with its own vision. Skim the whole recording as one storyboard image of timestamped frames, zoom into the moments that matter, and pull full-size frames for small text, plates or UI errors. No embedding or indexing wait. Use semantic search, the spoken-audio timeline or local ffmpeg only when they fit better. Use whenever a user asks an agent to watch, look at, review or explain a video, debug a screen recording, check CCTV or dashcam footage, or find when something happens in a clip.
license: MIT
metadata:
  version: "2.0"
---

# Watch Video With Your Own Eyes

A video is frames plus audio. You can read images, so watch the frames directly. Skim first, then zoom in, the way a person scrubs a timeline. Use the embedding index only to find things across long or many videos, or to hear speech.

## Pick the path first

| The question is about… | Use | Why |
|---|---|---|
| What happens, when, what something looks like (most questions) | **Direct watch**: storyboard, then zoom, then frames | Works seconds after upload, free, and you see the pixels yourself |
| What someone **says**, or a narrated video | **Indexed**: `get_video`, `ask_video` | Frames are silent. The index includes the speech timeline |
| One moment in a recording over 30 minutes, or across many videos | **Indexed** `search_videos` to find it, then **direct watch** around each hit | Search narrows hours down to seconds. Your eyes confirm what is there |
| Footage that must not leave the machine, or no MCP server | **Local ffmpeg**: see [references/ffmpeg-frame-extraction.md](references/ffmpeg-frame-extraction.md) | Same storyboard method, nothing uploaded |

If the `get_video_*` tools are not in your tool list, use the indexed path or local ffmpeg. Do not guess at tool names.

## Direct watch protocol

1. **Get the video in.** If you already have a `video_id`, skip this step. Otherwise:
   - `create_chat({ title })`, or reuse a chat.
   - For an HTTPS link, call `add_video_from_url`. For a local file, call `upload_video_file` (stdio bridge), or `create_upload_url`, then `curl -X PUT`, then `complete_upload`.
   - Get the `video_id` from `get_chat_videos`. **Do not wait for `READY`.** Direct watch works once the upload finishes.
2. **Plan with `get_video_info({ video_id })`.** Note `durationSec`, the resolution and `hasAudio`. If `hasAudio` is true and the question depends on speech, also take the indexed path.
3. **Skim the whole video with `get_video_storyboard({ video_id })`.** You get one image of 16 frames spread evenly across the video. Tiles read row by row, left to right. The timestamps are **not** drawn on the image. Read them from the returned `tiles` list, where each entry has `tile`, `row`, `col`, `t` and `time`.
4. **Zoom in on change.** Find the two neighbouring tiles where the thing you care about appears, disappears or changes. Call the storyboard again with `start` and `end` set to those two tiles' `t` values, using `tiles: 6` to `9`. Each zoom cuts `secondsPerTile` by about 10 times. Stop when it is under about 0.5 s, or when you can name the moment.
5. **Inspect detail with `get_video_frames`.** Pass `times` for the exact seconds (up to 8), or `start`, `end` and `count` for a burst. Use the default `width: 768` for scenes. Use `width: 1280` only for small text: plates, error toasts, stack traces, signage.
6. **Answer with proof.** Cite every claim with the `time` value of the frame you saw it in. Say which frames show it. Mark anything you could not read clearly as `unconfirmed`.

The step-by-step zoom rules, token costs and response shapes are in [references/storyboard-zoom.md](references/storyboard-zoom.md).

## Token budget (Claude counts about width × height / 750 tokens per image)

- A full storyboard at the default 1280 px wide costs about **1,250 tokens** for 16 moments.
- A zoom storyboard with 6 tiles costs about **800 tokens**.
- One frame at 768 px costs about **440 tokens**. At 1280 px it costs about **1,230 tokens**.
- A typical "what happens and when" answer costs **2–5k tokens** in total, whatever the video length.
- Never ask for 8 full-size frames just to skim. That costs about 10k tokens and shows less than one storyboard.

## Hand off to other tools

- To find a moment across hours or many videos, use `search_videos({ chat_id, query, video_id, from_sec, to_sec })` (requires `READY`). Then storyboard around each hit with `start = hit.startSec - 5` and `end = hit.endSec + 5`.
- For speech, narration or a meeting, use `get_video` for the spoken timeline and `ask_video` for grounded Q&A. Both are paid and require `READY`.
- For counting, boxes, PPE, plates or faces on a frame, take the frame from `get_video_frames`, then call `run_engine({ engine_id, image_base64, annotate: true })`. Find engine IDs with `list_engines`.

## Grounding guardrails

- Never describe a moment you have not seen in a tile or frame. A storyboard samples the video, so events between tiles are invisible until you zoom in.
- Express time as `M:SS` (or `H:MM:SS`) using the tool's `time` field. Never estimate times from where a tile sits in the grid.
- Say when the camera moves (pans, handheld footage, cuts). Positions in different tiles may not be comparable.
- Do not invent unreadable text, plates or names. Do not identify people by face unless the user's workflow explicitly enrols them.
- Frames are free but rate limited (about 40 calls a minute). Indexing, `search_videos`, `ask_video` and `analyze_video` use prepaid balance, so check `get_account` before indexing long batches.

## Examples

- [examples/cctv-clip-storyboard-zoom.md](examples/cctv-clip-storyboard-zoom.md): a real trace on an 11-second parking clip, showing the skim, two zooms and a frame check.
- [examples/debug-screen-recording.md](examples/debug-screen-recording.md): reading the exact error text from a checkout bug recording.
- [examples/long-recording-hybrid.md](examples/long-recording-hybrid.md): a 2-hour warehouse recording where search finds the hits and storyboards confirm them.
- [examples/meeting-with-speech.md](examples/meeting-with-speech.md): a narrated demo where the answer is in the audio, so the indexed path is used.
- [examples/local-private-footage.md](examples/local-private-footage.md): the same method with local ffmpeg when footage cannot be uploaded.
