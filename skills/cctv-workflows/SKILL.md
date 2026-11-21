---
name: cctv-workflows
description: Translate plain-English monitoring requirements into live CCTV camera workflows (PPE checks, person down, fire and smoke hazards, intrusion, face access, loitering, unattended bags, crowd limits, illegal parking, and rider helmets), including IANA timezone schedules, normalized ROI zones, clip recording, and alert tuning via MCP. Use whenever a user asks to set up a camera rule, configure CCTV detections, pause or resume cameras, or reduce false alarms.
---

# Live CCTV Monitoring Workflows & Rules

Turn plain-language monitoring requirements into exact camera configurations using the 10 built-in workflows.

## Standard Setup Procedure

Whenever a user asks to watch a camera for something in plain English:

1. Call `list_camera_workflows` to confirm the latest workflow definitions and defaults.
2. Select the matching `workflow` ID from the catalog below.
3. Map the user's request onto valid `config` keys (schedules, required PPE, hazard types, dwell minutes, crowd limits, cooldowns, and alert response).
4. Map any spatial instructions ("left half of the frame", "loading dock door") onto a normalized `zone` rectangle (`{ x, y, w, h }` with values `0`–`1`, minimum width/height `0.05`). Omit `zone` (or pass `null` in `update_camera`) to watch the whole frame.
5. **Confirm the plan in one concise line** (camera name, workflow, key thresholds, schedule, zone, alert channel), then call `create_camera` or `update_camera`.
6. Optionally test the rule immediately with `send_camera_frame({ camera_id, image_base64 })`, or inspect the live view with `get_camera_snapshot({ camera_id })`.

> Note: `update_camera` **replaces** `config` (unknown keys are dropped). Always call `get_camera` first and merge existing keys before patching `config`.

## The 10 Built-In Camera Workflows

| Category | Workflow ID | Label | Motion-Gated | Key `config` Fields & Defaults |
|---|---|---|---|---|
| **Workplace safety** | `ppe` | PPE check | No | `intervalSec: 10`, `require: { helmet: true, vest: true, gloves: false }`, `cooldownMin: 5` |
| **Workplace safety** | `fall` | Person down | Yes | `intervalSec: 3`, `cooldownMin: 5` |
| **Fire & hazards** | `hazards` | Fire & hazards | No | `intervalSec: 15`, `hazards: ["fire","smoke","flood","storm"]`, `sensitivity: "normal"` (`low`\|`normal`\|`high`), `cooldownMin: 10` |
| **Security** | `intrusion` | Intrusion | Yes | `intervalSec: 4`, `classes: ["person"]` (`person`\|`vehicle`\|`animal`), `schedule: { mode: "always" }`, `cooldownMin: 5` |
| **Security** | `unauthorized_access` | Face access | Yes | `intervalSec: 3`, `schedule: { mode: "always" }`, `matchThreshold: 0.5` (`0.3`–`0.9`), `alertUnknown: true`, `alertUnverified: true`, `cooldownMin: 5` |
| **Security** | `loitering` | Loitering | No | `intervalSec: 10`, `minutes: 5` (`1`–`1440`) |
| **Security** | `abandoned_object` | Unattended bags | No | `intervalSec: 10`, `minutes: 2` (`1`–`1440`) |
| **Security** | `crowd` | Crowd limit | No | `intervalSec: 10`, `maxPeople: 10` (`1`–`500`), `cooldownMin: 10` |
| **Traffic & parking** | `illegal_parking` | Illegal parking | No | `intervalSec: 30`, `authorizedPlates: []`, `plateLists: []`, `graceMin: 10` (`0`–`1440`) |
| **Traffic & parking** | `traffic_helmet` | Rider helmets | Yes | `intervalSec: 4`, `tripleRiding: true`, `cooldownMin: 2` |

## Shared Configuration Objects

### Sampling Interval
- `intervalSec`: Seconds between analyzed frames, clamped to `2`–`300`.

### Active Schedules (`intrusion`, `unauthorized_access`)
- Always active: `{ "mode": "always" }`
- Scheduled hours (supports overnight windows where `from > to`, e.g., `22:00` to `06:00`):
  ```json
  {
    "mode": "hours",
    "from": "22:00",
    "to": "06:00",
    "tz": "Asia/Kolkata"
  }
  ```
  Always use a valid IANA timezone (`UTC`, `America/New_York`, `Europe/London`, `Asia/Kolkata`, `Asia/Dubai`, `Asia/Singapore`).

### Alert Response (`config.response`)
Controls what happens when an alert fires (in addition to storing the alert and annotated photo):
```json
{
  "clipSec": 20,
  "push": true,
  "alarm": false
}
```
- `clipSec`: `0` (photo only), `10`, `20`, or `30` seconds recorded around the alert (half pre-roll, half post-roll).
- `push`: Send mobile app push notifications to the workspace.
- `alarm`: Sound an audible alarm on computers displaying the monitoring desk.

### Normalized ROI Zone (`zone`)
Fractions of the frame (`0`–`1`):
- Whole frame: omit `zone` (or `null` on update).
- Left half: `{ "x": 0, "y": 0, "w": 0.5, "h": 1 }`
- Right half: `{ "x": 0.5, "y": 0, "w": 0.5, "h": 1 }`
- Lower dock floor: `{ "x": 0.1, "y": 0.45, "w": 0.8, "h": 0.55 }`

## Plain-English Translation Examples

1. **"Alert our Discord when anyone at Loading Dock isn't wearing a helmet and hi-vis vest."**
   - `create_camera`: `name: "Loading dock"`, `workflow: "ppe"`, `config: { intervalSec: 10, require: { helmet: true, vest: true, gloves: false }, cooldownMin: 5 }`, `discord_webhook_url: "<url>"`.
2. **"Watch the back fence for people or vehicles between 10pm and 6am India time, left half only."**
   - `create_camera`: `name: "Back fence"`, `workflow: "intrusion"`, `config: { intervalSec: 4, classes: ["person", "vehicle"], schedule: { mode: "hours", from: "22:00", to: "06:00", tz: "Asia/Kolkata" }, cooldownMin: 5 }`, `zone: { x: 0, y: 0, w: 0.5, h: 1 }`.
3. **"Raise the crowd limit on the lobby camera to 25 and pause the parking camera."**
   - Call `list_cameras`, `get_camera` on Lobby to preserve existing config keys, `update_camera({ camera_id: lobbyId, config: { ...existing, maxPeople: 25 } })`, and `update_camera({ camera_id: parkingId, enabled: false })`.

## Operational Tips

- **Where live frames come from**: After `create_camera`, tell the user to attach the camera stream in the **desktop app** or the browser dashboard ("Use this computer"), or feed frames programmatically via `send_camera_frame`.
- **Reducing alert fatigue**: Tighten the `zone` to exclude public sidewalks/roads, raise `cooldownMin`, add an after-hours `schedule`, or tune `sensitivity` (`"low"` on `hazards`) / `minutes` (`loitering`, `abandoned_object`) / `graceMin` (`illegal_parking`).
- **Destructive check**: Always confirm with the user before calling `delete_camera`, which permanently removes the camera, its alerts, and enrolled people.
