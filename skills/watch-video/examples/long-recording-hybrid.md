# Example: A 4-Hour Recording, Search First, Then Confirm by Eye

Uniform skimming is the wrong first step for a long video with a brief target. At 16 tiles, a 4-hour video has 15 minutes between tiles, and a 5-second door opening falls between them almost every time.

## User prompt
> "This is last night's loading-dock recording, 22:00 to 02:00. Did anyone open the side door after 1 am, and was a forklift moving at the time?"

## 1. Index, then search the right window
1. `add_video_from_url({ chat_id, url, file_name: "dock-cam_2026-11-20T22-00.mp4" })`. Keep the camera name and start time in the file name: it is how you convert video seconds to clock time.
2. `get_video_info` returns `durationSec: 14402`, which matches 4 hours. Poll `get_chat_videos` until the status is `READY`.
3. 01:00 is 3 h into the file, which is 10,800 s:
```json
{ "tool": "search_videos", "args": { "chat_id": "…", "video_id": "vid_dock", "query": "person opening the side door", "from_sec": 10800 } }
```
The hits are `11612–11618 s` (score 0.83) and `13231–13235 s` (score 0.61).

## 2. Confirm every hit by eye
Storyboard around each hit with about 5 s on either side:
```json
{ "tool": "get_video_storyboard", "args": { "video_id": "vid_dock", "start": 11607, "end": 11623, "tiles": 8 } }
```
With `secondsPerTile: 2`:
- Tile 3 (3:13:32.0): the door is closed.
- Tile 4 (**3:13:34.0**): the door is open, and a person in a hi-vis vest steps through.
- Tile 7 (3:13:40.0): the door is closed again.

The second hit (13226–13240 s) shows a shadow sweeping across the door, which stays shut. That is a false positive, so drop it and say so.

## 3. Answer the second question from the same frames
The forklift is visible in tiles 2–6 of the first storyboard and shifts about a third of the frame between tile 2 and tile 5, so it is moving. If the motion is borderline, check with a short burst:
```json
{ "tool": "get_video_frames", "args": { "video_id": "vid_dock", "start": 11612, "end": 11618, "count": 4, "width": 768 } }
```

## Answer
> - **01:13:34** (video 3:13:34): a person in a hi-vis vest opens the side door. It is closed again by 01:13:40.
> - A forklift is moving in the dock lane at the same time (storyboard tiles 2–5).
> - Search also flagged 01:40:31, but the frames show a shadow and the door stays shut.

Clock time = file start (22:00) + video time. If the file name doesn't encode the start time, give video times only and say that clock times are `unconfirmed`.
