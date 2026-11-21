# Example: Debugging a Screen Recording

## User prompt
> "Watch `./checkout-bug.mp4` and tell me every step before the payment button breaks, including the exact error message."

## 1. Upload, without waiting for indexing
1. `create_chat({ title: "Checkout bug repro" })` returns `chat_id: "7f4c…1a"`.
2. `upload_video_file({ chat_id: "7f4c…1a", file_path: "./checkout-bug.mp4" })` (stdio bridge), or over HTTP: `create_upload_url`, then `curl -X PUT`, then `complete_upload`.
3. `get_chat_videos({ chat_id: "7f4c…1a" })` returns `video_id: "vid_9a2…"` with `status: "PROCESSING"`. **Continue straight away.**

## 2. Plan: `get_video_info`
This returns `durationSec: 55.2`, `1920×1080` and `hasAudio: false`. The recording is silent, so frames carry everything.

## 3. Skim: `get_video_storyboard({ video_id })`
A 4×4 grid with `secondsPerTile: 3.45`:
- Tiles 1–4 (0:01.7–0:12.1): cart page, quantity changed from 1 to 2.
- Tiles 5–9 (0:15.5–0:29.3): shipping form filled in, "Continue" clicked.
- Tiles 10–11 (0:32.8–0:36.2): card form, card number typed.
- Tile 12 (**0:39.7**): a red banner at the top of the form. The text is too small to read in a tile.
- Tiles 13–16 (0:43.1–0:53.5): the page reloads and the cart is empty.

## 4. Zoom on the failure: `get_video_storyboard`
```json
{ "video_id": "vid_9a2…", "start": 36.2, "end": 39.7, "tiles": 8 }
```
With `secondsPerTile: 0.44`, tile 4 (0:37.7) shows the "Pay $84.00" button pressed with a spinner. Tile 6 (0:38.6) is the first frame showing the banner.

## 5. Read the exact text: `get_video_frames`
```json
{ "video_id": "vid_9a2…", "times": [37.7, 38.6, 40.5], "width": 1280 }
```
At 1280 px the banner reads: `Payment failed: 422 card_declined (processing_error). Try again.` The DevTools console at 0:40.5 shows `POST /api/checkout/confirm 422`.

## Answer
A numbered repro, every step cited, for example "**0:37.7** clicked *Pay $84.00*", followed by the exact error string from 0:38.6 and the console line from 0:40.5. If part of a string was blurred in the frame, quote only what is readable and mark the rest `unconfirmed`.

## Notes
- No `search_videos` and no `ask_video` were needed. The fix took about 5k tokens, and no balance was used.
- If the recording had narration ("now I click pay…"), add `ask_video` once indexing is `READY`. See [meeting-with-speech.md](meeting-with-speech.md).
