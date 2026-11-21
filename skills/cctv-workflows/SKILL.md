---
name: cctv-workflows
description: Translate plain-English monitoring requirements into live CCTV camera workflows (PPE checks, person down, fire and smoke hazards, intrusion, face access, loitering, unattended bags, crowd limits, illegal parking, and rider helmets), including IANA timezone schedules, normalized ROI zones, clip recording, and alert tuning via MCP. Use whenever a user asks to set up a camera rule, configure CCTV detections, pause or resume cameras, or reduce false alarms.
license: MIT
metadata:
  version: "1.0"
---

# Live CCTV Monitoring Workflows & Rules

Map plain-English monitoring requests onto the 10 built-in live camera workflows accurately on the first try.

## Setup Procedure

1. Call `list_camera_workflows` to confirm available workflow IDs and defaults.
2. Pick the matching `workflow`:
   - **Workplace safety**: `ppe`, `fall`
   - **Fire & hazards**: `hazards`
   - **Security**: `intrusion`, `unauthorized_access`, `loitering`, `abandoned_object`, `crowd`
   - **Traffic & parking**: `illegal_parking`, `traffic_helmet`
3. Read [references/workflow-schemas.md](references/workflow-schemas.md) to map thresholds, IANA schedule (`from > to` spans midnight), normalized ROI `zone: { x, y, w, h }`, and `config.response` (`clipSec`, `push`, `alarm`).
4. **Confirm the configuration in one concise line**, then call `create_camera` or `update_camera`.
5. Optionally verify with `send_camera_frame({ camera_id, image_base64 })` or inspect live view via `get_camera_snapshot({ camera_id })`.

## Critical Rules

- **`update_camera` replaces `config`**: Always call `get_camera({ camera_id })` first and merge existing keys before patching `config` so you don't wipe schedules, cooldowns, or alert responses.
- **Live stream attachment**: After `create_camera`, remind the user to bind the feed from the desktop app or browser dashboard ("Use this computer"), or push frames via `send_camera_frame`.
- **Destructive check**: Always confirm with the user before calling `delete_camera`.

## References & Examples

- Read [references/workflow-schemas.md](references/workflow-schemas.md) for every workflow's exact `config` keys, defaults, value ranges, and motion-gating flags.
- Read [examples/plain-english-translations.md](examples/plain-english-translations.md) for worked plain-English → JSON translations, safe config merging, and false-alarm tuning recipes.
