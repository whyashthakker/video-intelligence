# Example: Setting Up a Tracked Campus & Tracing a Vehicle

## 1. Create the Site Group & Geocode the Address
1. `create_camera_group({ name: "North Site", description: "Main gate, lobby, and parking" })` → `group_id: "grp_north"`.
2. `geocode_address({ query: "Plot 14, Chakan Industrial Area, Pune" })` → `{ lat: 18.7606, lng: 73.8636 }`.

## 2. Assign Cameras With Tracking & Compass Geometry
Call `update_camera` for each camera in the site:
```json
// Main Gate facing East (90 deg)
{ "camera_id": "cm_gate", "group_id": "grp_north", "track_people": true, "latitude": 18.7605, "longitude": 73.8631, "heading": 90, "fov_deg": 88 }

// Yard Corner facing North-West (315 deg) — angled for clean triangulation with Main Gate
{ "camera_id": "cm_yard", "group_id": "grp_north", "track_people": true, "latitude": 18.7601, "longitude": 73.8640, "heading": 315, "fov_deg": 95 }
```

## 3. Trace a Vehicle Plate Across the Site
```json
{
  "tool": "get_movement",
  "args": { "group_id": "grp_north", "hours": 4, "kind": "vehicle", "plate": "MH12AB1234" }
}
```
Response shows stops `Main Gate 09:12 → Yard Corner 09:14 → Loading Bay 09:17`.

## 4. Visually Confirm a Stop
Call `get_sighting_image({ sighting_id: "<stop.sightingId>" })` and report:
> **Vehicle `MH12AB1234`**: Main Gate (09:12) → Yard Corner (09:14, 99% plate match) → Loading Bay (09:17, visually confirmed via sighting thumbnail).
