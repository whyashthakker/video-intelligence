# Storyboard → Zoom → Frames: Exact Rules

Read this when you use direct watch with `get_video_info`, `get_video_storyboard` and `get_video_frames`.

## How frames are sampled

A request for `n` tiles over `[start, end]` takes each frame from the **middle** of its slice:

```
step = (end - start) / n
t_i  = start + step * (i + 0.5)      for i = 0 … n-1
```

So a 16-tile storyboard of a 60 s video shows 1.9 s, 5.6 s … 58.1 s. The first and last seconds are never shown exactly. If the answer may be at the very start or end, ask for exact frames with `get_video_frames({ times: [0, durationSec - 0.1] })`.

Each tile stands for `secondsPerTile = (end - start) / n` seconds. **Anything shorter than `secondsPerTile` can fall between tiles.** Fast events like a punch, a dropped item or a flash of a toast need a zoom before you can say they did not happen.

## Grid shapes

| `tiles` | Grid (cols × rows) | Notes |
|---|---|---|
| 4 | 2×2 | |
| 5–6 | 3×2 | Unused cells are dark |
| 7–9 | 3×3 | |
| 10–12 | 4×3 | |
| 13–16 | 4×4 | Default |

Tiles are read row by row, left to right. The image has **no timestamps drawn on it**. Always map a tile to its time through the `tiles` list in the response:

```json
{
  "durationSec": 11.27, "duration": "0:11.3", "start": 0, "end": 11.27, "secondsPerTile": 0.7,
  "grid": { "cols": 4, "rows": 4, "order": "row by row, left to right, top to bottom" },
  "tiles": [ { "tile": 1, "row": 1, "col": 1, "t": 0.35, "time": "0:00.4" }, "…" ]
}
```

If a frame cannot be decoded, for example right at the end of a file, its tile is left out and the rest shift up. Trust the `tiles` list, not your own `t_i` arithmetic.

## The zoom loop

```
range = [0, duration], tiles = 16
loop:
  storyboard(range, tiles)
  find the first tile pair (A, B) where the target changes (appears, disappears, moves, opens…)
  if none:    the target is absent at this sampling rate. Zoom into suspect gaps, or switch to search_videos
  if secondsPerTile <= 0.5 s or the moment is clear: break
  range = [A.t, B.t], tiles = 6–9
get_video_frames(times = [the decisive moments]) for detail
```

- **Bracket, don't centre.** Zoom between the last tile *without* the target and the first tile *with* it. That pins the start of the event.
- **One question per zoom.** "When does the van arrive?" and "When does it leave?" are two zooms on two different tile pairs.
- **Two zooms are usually enough.** A 2-hour video at 16 tiles is 7.5 min per tile. One 8-tile zoom gives about 56 s per tile, and a second gives about 7 s. From there, `get_video_frames` with a `count: 8` burst gives about 1 s steps.
- For recordings over about 30 minutes where the target is small or brief, **search first** with `search_videos`, then zoom around each hit. Uniform skimming can miss a 3-second event in 2 hours.

## Choosing frame width

| Need | `width` | Tokens per image (about) |
|---|---|---|
| Scene layout, people, vehicles, actions | 512–768 | 250–440 |
| UI screens, dashboards, slides | 1024 | 790 |
| Small text: plates, error toasts, stack traces, timestamps burned into CCTV | 1280 | 1,230 |

Frames are never upscaled. Asking for 1280 px from a 640 px source returns 640 px. Check `width` in `get_video_info` first. If the source is too small to read something, say so instead of guessing.

## Response shapes

- `get_video_info` returns `durationSec`, `duration`, `width`, `height`, `fps`, `hasAudio`, `videoCodec`, `status` and `indexStatus`.
- `get_video_storyboard` returns one image, then the JSON above.
- `get_video_frames` returns `Frame at M:SS (t s)` text followed by its image for each frame, then a JSON summary.

## Errors

| Error | Meaning | What to do |
|---|---|---|
| `still uploading` (409) | The file isn't in storage yet | Wait for `complete_upload` or the URL import to finish, then retry |
| `Could not read this video file` (422) | Unsupported or corrupt file | Report it. Indexing will likely fail too |
| `every time must be within it` (400) | A requested time is past `durationSec` | Clamp times to `durationSec - 0.1` |
| `length is unknown` (400) | No duration in the container (some live WebM files) | Pass an explicit `end` |
| 429 | Too many frame calls | Wait about a minute. Use storyboards instead of many single frames |
