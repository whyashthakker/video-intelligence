---
name: ceptory-access-and-plates
description: Manage face access control lists (staff, watchlist, custom allow/alert lists) on Ceptory unauthorized_access cameras and ANPR license plate lists (Authorized, Residents, Banned, instant critical alerts, and photo plate reading) on illegal_parking cameras. Use whenever a user wants to enroll or remove authorized staff or watchlisted individuals, tune face match strictness, extract number plates from photos, or configure allowed and banned vehicle plate lists.
---

# Ceptory Face Access Control & License Plate (ANPR) Enforcement

Operate Ceptory's identity-based (`unauthorized_access`) and vehicle-plate-based (`illegal_parking`) camera rules accurately and safely.

## Part 1: Face Access Control (`unauthorized_access`)

### Camera Setup & Tuning
Create or update a camera with `workflow: "unauthorized_access"`:
```json
{
  "name": "Server room door",
  "workflow": "unauthorized_access",
  "config": {
    "intervalSec": 3,
    "schedule": { "mode": "always" },
    "matchThreshold": 0.5,
    "alertUnknown": true,
    "alertUnverified": true,
    "cooldownMin": 5
  }
}
```
- `matchThreshold` (`0.3`–`0.9`, default `0.5`): Higher = stricter matching (fewer false matches, more unknown alerts); lower = more lenient.
- `alertUnknown`: Fire alerts for faces not on an allowed list (watchlist / `alert:` list matches **always** alert regardless of this setting).
- `alertUnverified`: Fire alerts when a person enters the zone but their face is obscured or angled away.
- `schedule`: Restrict enforcement to after-hours if desired (`{ mode: "hours", from: "19:00", to: "08:00", tz: "Asia/Kolkata" }`).

### People Lists & Naming Convention
Pass `list` to `add_camera_person`:
- `"staff"` (default): Allowed personnel.
- `"watchlist"`: Alert immediately on sight.
- `"allow:<List name>"`: Custom allowed list (name up to 40 chars, e.g., `"allow:Contractors"`, `"allow:VIP Visitors"`).
- `"alert:<List name>"`: Custom alert-on-sight list (e.g., `"alert:Banned"`, `"alert:Ex-Staff"`).

### Enrolling & Managing People
- **Add a new person**:
  ```json
  {
    "tool": "add_camera_person",
    "args": {
      "camera_id": "<cameraId>",
      "name": "Asha Nair",
      "list": "staff",
      "image_base64": "<clear front-facing JPEG/PNG base64 or data URL, max 8 MB>"
    }
  }
  ```
- **Improve accuracy with multiple angles/lighting**: Pass the returned `person_id` with `image_base64` (up to **5 photos per person**). Only the face embedding is used for matching.
- **Inspect enrolled people**: Call `get_camera({ camera_id })`.
- **Remove a person**: Confirm with the user first (destructive action), then call `remove_camera_person({ camera_id, person_id })`.

### Privacy & Compliance Guardrails
- Face recognition must be opt-in with appropriate site signage, consent, and retention policies (e.g., GDPR, India DPDP Act, BIPA depending on jurisdiction).
- Never enroll people from blurry crowd frames; require a clear, well-lit, single-face photo.

---

## Part 2: Illegal Parking & ANPR Plate Lists (`illegal_parking`)

### Camera Setup
Create or update a camera with `workflow: "illegal_parking"`:
```json
{
  "name": "Main gate no-parking",
  "workflow": "illegal_parking",
  "config": {
    "intervalSec": 30,
    "graceMin": 10,
    "authorizedPlates": [],
    "plateLists": []
  }
}
```
- `graceMin` (`0`–`1440`, default `10`): Minutes an unauthorized vehicle may stay in the watched `zone` before alerting. Use `0` for instant alerts in fire lanes or emergency bays.

### Reading Plates from a Photo (`read_plates`)
When a user shares an image of parked cars or a registration sticker:
1. Call `read_plates({ image_base64 })` — reads up to 10 visible number plates from the photo.
2. Confirm the recognized plate strings with the user before adding them to an allowlist or alertlist.

### Managing Plate Lists (`update_plate_list`)
Prefer `update_plate_list` over `update_camera` so other camera settings stay untouched:
- **Default `"Authorized"` list**: Always an allowed list (cannot be deleted; remove plates instead).
- **Up to 10 named lists** (up to 500 plates per list):
  - `alert: false` (default for new lists): Allowed vehicles, e.g., `"Residents"`, `"Deliveries"`, `"Fleet"`.
  - `alert: true`: **Critical alert on sight** (bypasses parking grace period), e.g., `"Banned"`, `"Stolen"`, `"Watchlist"`.
- **Automatic plate normalization**: Plates are normalized to uppercase alphanumeric (`A-Z0-9`, length `2`–`12`); spaces and hyphens are stripped (`"MH-12-AB-1234"` → `"MH12AB1234"`).

#### Examples
```json
// Add residents to a named allowlist
{
  "tool": "update_plate_list",
  "args": {
    "camera_id": "<cameraId>",
    "list": "Residents",
    "alert": false,
    "add": ["MH12AB1234", "KA01CD5678"]
  }
}

// Create a critical alert-on-sight banned list
{
  "tool": "update_plate_list",
  "args": {
    "camera_id": "<cameraId>",
    "list": "Banned",
    "alert": true,
    "add": ["DL04XY9999"]
  }
}

// Remove a plate or delete a named list
{
  "tool": "update_plate_list",
  "args": {
    "camera_id": "<cameraId>",
    "list": "Deliveries",
    "delete_list": true
  }
}
```

### Verification
Run `send_camera_frame({ camera_id, image_base64 })` with a sample parking image to confirm plate detection and rule evaluation before relying on live alerts.
