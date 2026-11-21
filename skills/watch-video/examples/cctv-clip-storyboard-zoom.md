# Example: Skim → Zoom → Frame on a Short Parking Clip

A real trace on an 11-second 640×360 phone clip of a basement car park. The tile times below are the actual values the tools returned.

## User prompt
> "When does the scooter come in, what does the rider look like, and where do they go?"

## 1. Plan: `get_video_info`
```json
{ "video_id": "cm9…park" }
```
```json
{ "durationSec": 11.27, "duration": "0:11.3", "width": 640, "height": 360, "fps": 15, "hasAudio": false, "status": "PROCESSING", "indexStatus": "RUNNING" }
```
The video is still indexing, which doesn't matter here. With no audio, direct watch is the whole job.

## 2. Skim: `get_video_storyboard`
```json
{ "video_id": "cm9…park" }
```
This returns a 4×4 grid with `secondsPerTile: 0.7`. From the `tiles` list:

| Tile | Time | Seen |
|---|---|---|
| 1–5 | 0:00.4 – 0:03.2 | Empty car park, pillar centre-left, orange bollards |
| 6 | 0:03.9 | Rider on a dark blue and white scooter, close to the camera, right side |
| 7–8 | 0:04.6 – 0:05.3 | Same rider crossing left to right. **The camera pans**: the red car and the back row come into view |
| 9–16 | 0:06.0 – 0:10.9 | The rider gets smaller, heading for the far row of pillars |

The target appears between tile 5 (3.17 s, absent) and tile 6 (3.87 s, present).

## 3. Zoom on the entry: `get_video_storyboard`
```json
{ "video_id": "cm9…park", "start": 2.8, "end": 4.2, "tiles": 6 }
```
This returns a 3×2 grid with `secondsPerTile: 0.23` and tiles at `0:02.9, 0:03.2, 0:03.4, 0:03.6, 0:03.9, 0:04.1`.
- Tiles 1–3 (to 0:03.4): no scooter.
- Tile 4 (**0:03.6**): the front wheel and handlebar come in at the bottom-right edge.
- Tiles 5–6: the rider is fully in frame, wearing a vertical-striped shirt and light trousers, **with no helmet visible**.

The time step is now under 0.5 s, so stop zooming.

## 4. Check the end: `get_video_frames`
```json
{ "video_id": "cm9…park", "times": [11.0], "width": 640 }
```
At 0:11.0 the rider is small, near the back row beside the white SUV and the yellow-and-black striped pillars. It isn't clear whether they have stopped.

## Answer
> - **0:03.6**: the scooter enters from the bottom-right (zoom tile 4). Before 0:03.4 the car park is empty.
> - **Rider (0:03.9–0:04.1)**: striped shirt, light trousers, dark blue and white scooter, no helmet visible.
> - **0:04.6–0:10.9**: they ride across and away from the camera towards the far row. The camera pans left during this, so positions shift between frames.
> - **0:11.0** (last frame): near the back row beside the white SUV. Whether they parked is **unconfirmed**, because the clip ends first.

Cost: 1 info call, 2 storyboards and 1 frame, about 2.3k image tokens. No indexing wait and no balance used.
