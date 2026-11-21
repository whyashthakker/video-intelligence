---
name: face-and-plate-access
description: Manage face access control lists (staff, watchlist, custom allow/alert lists) and ANPR license plate lists (Authorized, Residents, Banned, instant critical alerts, and reading plates from photos) on live cameras via MCP. Use whenever a user wants to enroll or remove authorized people or watchlists, tune face match strictness, read number plates from an image, or configure allowed and banned parking plate lists.
license: MIT
metadata:
  version: "1.0"
---

# Face Access Control & License Plate (ANPR) Enforcement

Operate identity-based (`unauthorized_access`) and vehicle-plate-based (`illegal_parking`) camera lists cleanly and safely.

## Part 1: Face Access (`unauthorized_access`)

1. Ensure the camera runs `workflow: "unauthorized_access"` (configure `schedule`, `matchThreshold` `0.3`–`0.9`, `alertUnknown`, `alertUnverified`, `cooldownMin`).
2. Enroll people with `add_camera_person({ camera_id, name, list, image_base64 })` using a clear, well-lit single-face photo. Add up to **5 photos per person** by passing `person_id` on follow-up calls.
3. Inspect enrolled people with `get_camera({ camera_id })`. Always confirm with the user before calling `remove_camera_person`.
4. **Privacy guardrail**: Face access must be opt-in with site signage, consent, and retention rules appropriate to local law (GDPR, India DPDP Act, BIPA, etc.). Never enroll from blurry crowd crops.

## Part 2: Illegal Parking & ANPR (`illegal_parking`)

1. Ensure the camera runs `workflow: "illegal_parking"` (set `graceMin` `0`–`1440`, default `10`; use `0` for fire lanes).
2. Extract plates from a shared photo via `read_plates({ image_base64 })` (up to 10 plates) and confirm the OCR strings with the user before saving.
3. Update lists with `update_plate_list` (preserves all other camera settings):
   - `"Authorized"`: default allowlist (cannot be deleted).
   - Named allowlists (`alert: false`, e.g. `"Residents"`): vehicles allowed to park.
   - Named alertlists (`alert: true`, e.g. `"Banned"`): **critical alert on sight**, bypassing `graceMin`.
4. Verify live rule evaluation with `send_camera_frame({ camera_id, image_base64 })`.

## References & Examples

- Read [references/list-rules-and-normalization.md](references/list-rules-and-normalization.md) for `allow:`/`alert:` list syntax, `matchThreshold` tuning, plate normalization, and list size limits.
- Read [examples/enrol-staff-and-plates.md](examples/enrol-staff-and-plates.md) for multi-photo enrollment and `read_plates` → `update_plate_list` tool calls.
