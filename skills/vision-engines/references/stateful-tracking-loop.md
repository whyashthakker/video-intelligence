# Stateful Multi-Frame Engine Tracking Loops (TypeScript & Python)

Use this pattern whenever `list_engines` reports `stateful: true` for an engine (tracking, dwell, persistence across frames).

## Step 1 — Extract Frames at a Fixed FPS
```bash
ffmpeg -i clip.mp4 -vf "fps=1,scale='min(1280,iw)':-2" -q:v 3 frames/frame_%04d.jpg
```

## TypeScript / Node.js Loop
```ts
import { readdirSync, readFileSync, writeFileSync } from "node:fs";
import { join } from "node:path";

const API_KEY = process.env.CEPTORY_API_KEY!;
const ENGINE_ID = "ppe"; // pick exact id from GET /api/v1/engines
const FPS = 1;

const files = readdirSync("frames").filter((f) => f.endsWith(".jpg")).sort();
let state: Record<string, unknown> | null = null;

for (let i = 0; i < files.length; i++) {
  const image = readFileSync(join("frames", files[i]!)).toString("base64");
  const res = await fetch(`https://www.ceptory.com/api/v1/engines/${ENGINE_ID}`, {
    method: "POST",
    headers: { Authorization: `Bearer ${API_KEY}`, "Content-Type": "application/json" },
    body: JSON.stringify({
      image,
      zone: { x: 0, y: 0, w: 1, h: 1 },
      params: {},
      state,
      timestamp: i / FPS,
      annotate: false,
    }),
  });
  if (res.status === 429) {
    const waitSec = Number(res.headers.get("retry-after") ?? 60);
    await new Promise((r) => setTimeout(r, waitSec * 1000));
    i--;
    continue;
  }
  const data = await res.json();
  state = data.state ?? state;
  console.log(`t=${i / FPS}s`, data.detections?.length ?? 0, "detections");
}
```

## Python Loop (Standard Library Only)
```python
import base64, glob, json, os, time, urllib.request

API_KEY = os.environ["CEPTORY_API_KEY"]
ENGINE_ID = "ppe"
state = None

for idx, path in enumerate(sorted(glob.glob("frames/*.jpg"))):
    with open(path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode()
    payload = json.dumps({
        "image": img_b64,
        "zone": {"x": 0, "y": 0, "w": 1, "h": 1},
        "params": {},
        "state": state,
        "timestamp": float(idx),
        "annotate": False,
    }).encode()
    req = urllib.request.Request(
        f"https://www.ceptory.com/api/v1/engines/{ENGINE_ID}",
        data=payload,
        headers={"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"},
    )
    with urllib.request.urlopen(req, timeout=50) as resp:
        data = json.loads(resp.read())
    state = data.get("state", state)
    print(f"t={idx}s -> {len(data.get('detections', []))} detections")
    time.sleep(0.55)  # stay comfortably under 120 req/min
```
