# Examples: Enrolling Staff Faces & Configuring Plate Lists

## Example 1 — Enrol a Staff Member With Two Photos
```json
// Photo 1 (creates person, returns person.id)
{
  "tool": "add_camera_person",
  "args": {
    "camera_id": "cm_server_room_01",
    "name": "Asha Nair",
    "list": "staff",
    "image_base64": "<front_facing_jpeg_b64>"
  }
}

// Photo 2 (adds second angle using returned person_id)
{
  "tool": "add_camera_person",
  "args": {
    "camera_id": "cm_server_room_01",
    "person_id": "per_asha_01",
    "image_base64": "<glasses_angle_jpeg_b64>"
  }
}
```

## Example 2 — Read Plates From a Photo & Add to Residents List
1. Call `read_plates({ image_base64: "<lot_photo_b64>" })` → returns `["MH12AB1234", "KA01CD5678"]`.
2. Confirm the plate text with the user (OCR on angled plates can swap `O`/`0` or `I`/`1`).
3. Call `update_plate_list`:
```json
{
  "tool": "update_plate_list",
  "args": {
    "camera_id": "cm_gate_parking_01",
    "list": "Residents",
    "alert": false,
    "add": ["MH12AB1234", "KA01CD5678"]
  }
}
```

## Example 3 — Add an Instant-Alert Banned Plate
```json
{
  "tool": "update_plate_list",
  "args": {
    "camera_id": "cm_gate_parking_01",
    "list": "Banned",
    "alert": true,
    "add": ["DL04XY9999"]
  }
}
```
Verify rule evaluation immediately with `send_camera_frame({ camera_id: "cm_gate_parking_01", image_base64: "<test_frame_b64>" })`.
