---
name: vision-engines
description: Discover and run computer-vision detection engines (PPE, fire/smoke, license plates, faces, crowd counting, vehicles, and more) on single images or stateful multi-frame tracking loops with ROI zone filtering and annotated JPEG output via MCP or /api/v1/engines. Use whenever a user wants to run a detector on a still image, process video frames in a script, or build a custom vision pipeline.
license: MIT
metadata:
  version: "1.0"
---

# Computer-Vision Detection Engines API (Stateless & Stateful)

Invoke individual computer-vision detectors directly on still images or sequential video frames without configuring a live camera.

## Workflow

1. **Always discover the live catalog first** (never guess engine IDs or params):
   - MCP: `list_engines({})`
   - REST (`READ` scope): `GET https://www.ceptory.com/api/v1/engines`
   - Inspect each engine's `id`, `maturity` (`stable` | `beta` | `experimental`), `params`, `stateful`, and `needs_zone`.
2. **Run on a single image (stateless)**:
   - MCP: `run_engine({ engine_id, image_base64, zone, params, annotate: true })` (stdio `ceptory-mcp` also accepts `image_path`).
   - REST (`WRITE` scope): `POST https://www.ceptory.com/api/v1/engines/{engineId}` with `{ image, zone, params, annotate }`.
3. **Run across video frames (stateful)**:
   - When `stateful: true`, extract sequential frames at fixed FPS and pass back the previous response's `state` plus `timestamp` (seconds) on every frame.
   - Read [references/stateful-tracking-loop.md](references/stateful-tracking-loop.md) for complete TypeScript and Python loops with `429` retry handling.

## Constraints & Guardrails

- **Image limits**: Base64 JPEG/PNG or `data:` URL, max **8 MB** decoded (12 MB JSON body).
- **Rate limit & timeout**: **120 req/min** per key; 45s per-call timeout (`504`).
- **Performance**: Keep `annotate: false` during bulk frame loops and request `annotate: true` only on flagged frames.
- **Billing & HTTP**: Draws from prepaid workspace balance (`get_account`). Call HTTP directly — no client SDKs.

## References & Examples

- Read [references/stateful-tracking-loop.md](references/stateful-tracking-loop.md) when writing a multi-frame script in TypeScript or Python.
- Read [examples/inspect-frame-with-engine.md](examples/inspect-frame-with-engine.md) for a single-frame MCP inspection walkthrough.
