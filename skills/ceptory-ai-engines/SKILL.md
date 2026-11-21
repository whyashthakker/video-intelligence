---
name: ceptory-ai-engines
description: Discover and invoke Ceptory's AI detection engines catalog (PPE, fire/smoke, license plates, faces, crowd counting, vehicles, and more) on individual images or sequential video frames via MCP (list_engines, run_engine) or REST API (/api/v1/engines), including stateful multi-frame tracking loops, ROI zone filtering, and annotated JPEG output. Use whenever a user wants to run a specific computer-vision detector on an image, test frame-by-frame detections in a script, or build a custom detection pipeline on Ceptory's Engines API.
---

# Ceptory Direct AI Detection Engines API

Run individual Ceptory computer-vision detection engines directly on still images or sequential frames—without setting up a full live camera workflow.

## Step 1: Discover Available Engines

Always query the live catalog first instead of guessing engine IDs or parameters, because the catalog exposes each engine's exact `id`, `category`, `maturity` (`stable` | `beta` | `experimental`), `params`, `stateful` flag, and `needs_zone` flag.

- **MCP**: Call `list_engines({})`.
- **REST API** (requires `READ` scope):
  ```bash
  curl https://www.ceptory.com/api/v1/engines \
    -H "Authorization: Bearer $CEPTORY_API_KEY"
  ```

## Step 2: Run an Engine on a Single Image (Stateless)

### Via MCP (`run_engine`)
```json
{
  "engine_id": "<id from list_engines>",
  "image_base64": "<base64 JPEG/PNG or data:image/jpeg;base64,...>",
  "zone": { "x": 0.1, "y": 0.2, "w": 0.8, "h": 0.7 },
  "params": {},
  "annotate": true
}
```
- Setting `annotate: true` returns the annotated JPEG image with bounding boxes drawn directly in the MCP response alongside the JSON detections.
- If using the `ceptory-mcp` stdio bridge, you can pass a local `image_path` instead of base64.

### Via REST API (`POST /api/v1/engines/{engineId}`)
Requires a `WRITE` (or `ADMIN`) API key or signed-in session.

```bash
IMAGE_B64=$(base64 < frame.jpg | tr -d '\n')

curl -X POST https://www.ceptory.com/api/v1/engines/$ENGINE_ID \
  -H "Authorization: Bearer $CEPTORY_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"image\": \"$IMAGE_B64\",
    \"zone\": {\"x\": 0, \"y\": 0, \"w\": 1, \"h\": 1},
    \"params\": {},
    \"annotate\": true
  }"
```

## Step 3: Multi-Frame Tracking Loop (Stateful Engines)

For engines where `stateful: true` (such as dwell/loitering, tracking, or multi-frame persistence detectors):
1. Extract sequential frames from a video at a steady interval (e.g., `ffmpeg -i input.mp4 -vf fps=1 frame_%04d.jpg`).
2. On frame `0`, send `state: null` and `timestamp: 0.0`.
3. On every subsequent frame `k`, pass back the `state` object returned by the previous response together with the frame's `timestamp` in seconds.

```ts
let state: Record<string, unknown> | null = null;

for (const frame of frames) {
  const res = await fetch(`https://www.ceptory.com/api/v1/engines/${engineId}`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.CEPTORY_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      image: frame.jpegBase64,
      zone: { x: 0.0, y: 0.0, w: 1.0, h: 1.0 },
      params: engineParams,
      state,
      timestamp: frame.timestampSec,
      annotate: false,
    }),
  });
  const data = await res.json();
  state = data.state ?? state;
  // Inspect detections / alerts in `data`
}
```

## Constraints, Limits & Best Practices

- **Engine ID format**: Lowercase letters and underscores (`^[a-z][a-z_]{1,40}$`).
- **Image constraints**: JPEG or PNG encoded as raw base64 or a `data:` URL; max **8 MB** decoded image size (max 12 MB HTTP JSON body).
- **Zone coordinates**: Normalized fractions `0`–`1` (`x`, `y`, `w >= 0.05`, `h >= 0.05`). Required when `needs_zone: true`.
- **Rate limit & timeout**: **120 requests/minute** per API key; per-call timeout is 45 seconds (`504` on timeout).
- **Billing**: Engine calls draw from the workspace's prepaid balance (check `get_account` before running large batch frame loops).
- **Annotations**: Disable `annotate: false` during high-throughput frame processing and enable `annotate: true` only on flagged frames to save bandwidth and response size.
- **No SDKs**: Use standard `fetch`, Python `requests`/`urllib`, or `curl` directly against `https://www.ceptory.com/api/v1/engines`.
