# Live CCTV Workflow Schemas Reference

## Table of Contents
- [Shared Settings (intervalSec, zone, schedule, response)](#shared-settings)
- [Workplace Safety (ppe, fall)](#workplace-safety)
- [Fire & Hazards (hazards)](#fire--hazards)
- [Security (intrusion, unauthorized_access, loitering, abandoned_object, crowd)](#security)
- [Traffic & Parking (illegal_parking, traffic_helmet)](#traffic--parking)

## Shared Settings

- `intervalSec`: Integer `2`–`300` (seconds between analyzed frames).
- `zone`: Optional normalized ROI `{ x, y, w, h }` with `x, y ∈ [0, 1]`, `w, h ∈ [0.05, 1]`. Omit (or pass `null` on `update_camera`) to watch the entire frame.
- `schedule` (used by `intrusion` and `unauthorized_access`):
  - Always on: `{ "mode": "always" }`
  - Time window (overnight supported when `from > to`):
    ```json
    { "mode": "hours", "from": "22:00", "to": "06:00", "tz": "Asia/Kolkata" }
    ```
    Use valid IANA timezones (`UTC`, `America/New_York`, `Europe/London`, `Asia/Kolkata`, `Asia/Dubai`, `Asia/Singapore`).
- `response` (nested inside `config.response`):
  ```json
  { "clipSec": 20, "push": true, "alarm": false }
  ```
  - `clipSec`: `0` (photo only), `10`, `20`, or `30` (half pre-roll, half post-roll).
  - `push`: Mobile push notifications to workspace members.
  - `alarm`: Audible monitoring-desk alarm.

## Workplace Safety

### `ppe` (PPE check — motion-gated: false)
- Defaults: `{ intervalSec: 10, require: { helmet: true, vest: true, gloves: false }, cooldownMin: 5 }`
- `require`: `{ helmet: boolean, vest: boolean, gloves: boolean }` — required gear for every person in the zone.
- `cooldownMin`: Minutes between repeat alerts.

### `fall` (Person down — motion-gated: true)
- Defaults: `{ intervalSec: 3, cooldownMin: 5 }`

## Fire & Hazards

### `hazards` (Fire & hazards — motion-gated: false)
- Defaults: `{ intervalSec: 15, hazards: ["fire", "smoke", "flood", "storm"], sensitivity: "normal", cooldownMin: 10 }`
- `hazards`: Subset of `["fire", "smoke", "flood", "storm"]`.
- `sensitivity`: `"low"` | `"normal"` | `"high"`.

## Security

### `intrusion` (Intrusion — motion-gated: true)
- Defaults: `{ intervalSec: 4, classes: ["person"], schedule: { mode: "always" }, cooldownMin: 5 }`
- `classes`: Subset of `["person", "vehicle", "animal"]`.

### `unauthorized_access` (Face access — motion-gated: true)
- Defaults: `{ intervalSec: 3, schedule: { mode: "always" }, cooldownMin: 5, matchThreshold: 0.5, alertUnverified: true, alertUnknown: true }`
- `matchThreshold`: `0.3`–`0.9` (higher = stricter).
- `alertUnknown`: Alert on faces not on an allowed list (watchlist matches always alert).
- `alertUnverified`: Alert when a person's face is obscured.

### `loitering` (Loitering — motion-gated: false)
- Defaults: `{ intervalSec: 10, minutes: 5 }` (`minutes`: `1`–`1440`).

### `abandoned_object` (Unattended bags — motion-gated: false)
- Defaults: `{ intervalSec: 10, minutes: 2 }` (`minutes`: `1`–`1440`).

### `crowd` (Crowd limit — motion-gated: false)
- Defaults: `{ intervalSec: 10, maxPeople: 10, cooldownMin: 10 }` (`maxPeople`: `1`–`500`).

## Traffic & Parking

### `illegal_parking` (Illegal parking — motion-gated: false)
- Defaults: `{ intervalSec: 30, authorizedPlates: [], graceMin: 10 }` (`graceMin`: `0`–`1440`).
- `plateLists`: Up to 10 named lists `[{ name, alert, plates }]` (prefer `update_plate_list` tool).

### `traffic_helmet` (Rider helmets — motion-gated: true)
- Defaults: `{ intervalSec: 4, tripleRiding: true, cooldownMin: 2 }`
- `tripleRiding`: Also alert on three riders on one two-wheeler.
