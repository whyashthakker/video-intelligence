# Example: Discovering & Running a Detector With Annotated Output

## User Prompt
> "Check this warehouse photo for fire or smoke and show me the boxes."

## Step 1 — Discover Exact Engine ID & Params
Call `list_engines({})` and locate the fire/smoke engine entry (check `id`, `maturity`, `params`, and `needs_zone`).

## Step 2 — Run Engine With `annotate: true`
```json
{
  "tool": "run_engine",
  "args": {
    "engine_id": "<exact_id_from_list_engines>",
    "image_base64": "<jpeg_base64_or_data_url>",
    "annotate": true
  }
}
```
*(If using `ceptory-mcp` stdio bridge, pass `image_path` instead of `image_base64`.)*

## Step 3 — Interpret & Report
- MCP returns both the annotated JPEG image (boxes drawn) and the structured JSON detections.
- Summarize: number of detections, confidence scores, normalized bounding box positions, and whether any area of the image was occluded or low-light.
- Keep `annotate: false` for bulk scripts and re-run with `annotate: true` only on frames where detections fired.
