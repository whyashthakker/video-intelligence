---
name: ceptory-cross-camera-tracking
description: Group Ceptory cameras by physical site or area, enable cross-camera person and vehicle re-identification (Re-ID) journeys, inspect sighting thumbnails, geocode site addresses, and configure camera coordinates, compass headings, and field-of-view (FOV) for automatic multi-camera event triangulation on the map. Use whenever a user wants to organize cameras into sites, trace where a person or vehicle went across cameras, follow a license plate across a campus, or pinpoint alert locations on a map.
---

# Ceptory Cross-Camera Tracking & Multi-Camera Geo-Triangulation

Turn isolated camera alerts into site-wide person/vehicle journeys and map-pinpointed events.

## Part 1: Organize Cameras Into Site Groups

Camera groups represent physical sites or connected areas (e.g., `"Plant 2"`, `"North site"`, `"Building A"`).

1. **Discover existing groups**: Call `list_camera_groups`.
2. **Create a site group**: Call `create_camera_group({ name, description })`.
3. **Assign cameras and enable tracking**: For every camera covering corridors, gates, lobbies, or parking lots in that site, call `update_camera`:
   ```json
   {
     "camera_id": "<cameraId>",
     "group_id": "<groupId>",
     "track_people": true
   }
   ```
   - `track_people: true` records appearance sightings for **both people and vehicles** (`car`, `truck`, `bus`, `motorcycle`, `bicycle`).
   - Sightings are retained for **30 days** and linked across different cameras within a **45-minute** inter-camera gap.
   - To ungroup a camera, pass `group_id: null`. Ask the user before calling `delete_camera_group` (cameras remain, ungrouped).

## Part 2: Querying Cross-Camera Journeys (`get_movement`)

Call `get_movement` to trace paths across a site or the whole workspace:
```json
{
  "group_id": "<groupId or 'all'>",
  "hours": 2,
  "kind": "person",
  "plate": "MH12AB1234"
}
```
- `group_id`: A specific group ID from `list_camera_groups`, or `"all"` for the entire workspace.
- `hours`: Look-back window from `0.25` to `72` hours.
- `kind`: Optional `"person"` or `"vehicle"`.
- `plate`: Optional number plate to follow a specific vehicle (normalized automatically).

### Inspecting Visual Evidence for Stops
Each journey contains ordered `stops` (`sightingId`, `cameraId`, `cameraName`, `latitude`, `longitude`, `firstAt`, `lastAt`, `match`).
- Call `get_sighting_image({ sighting_id })` to retrieve the cropped thumbnail for any stop and visually verify continuity before escalating an investigation.

### How Re-ID Matching Works & How to Communicate It
- Ceptory combines visual appearance embeddings (`0.75` CLIP similarity) and color histograms (`0.25`), upgraded when a readable license plate (`>= 0.99` on exact / edit-distance 1 match) or clear face embedding (`>= 0.95` when cosine `>= 0.5`) is present on both sightings.
- Cross-camera thresholds: `0.78` for people, `0.84` for vehicles.
- **Accuracy rule**: Always describe appearance-based cross-camera links as **"likely"** matches (e.g., *"Likely same person: Main Gate 10:12 → Lobby 10:14 → Parking B 10:18 (86% match)"*), unless confirmed by a visually verified sighting photo or exact license plate read.
- **Troubleshooting empty results**: If `get_movement` returns no multi-camera journeys, verify via `list_cameras` that at least two cameras in the group have `track_people: true` and are `online`.

## Part 3: Map Placement & Multi-Camera Event Triangulation

When two or more cameras report the same kind of alert (and same hazard type for `hazards`) within a **3-minute window**, Ceptory intersects their sight lines to compute an estimated GPS location (`{ lat, lng, accuracyM, method: "triangulated", cameras }`) up to **3,000 meters** from the cameras.

### Setup Steps
1. **Geocode the site address**:
   Call `geocode_address({ query: "123 Industrial Park Rd, Pune" })` to get baseline `latitude` and `longitude`.
2. **Place each camera with compass heading and FOV**:
   Call `update_camera` (or pass at `create_camera`):
   ```json
   {
     "camera_id": "<cameraId>",
     "latitude": 18.52043,
     "longitude": 73.85674,
     "heading": 135,
     "fov_deg": 90
   }
   ```
   - `latitude` (`-90` to `90`) and `longitude` (`-180` to `180`): Must be sent together (or both `null` to clear).
   - `heading` (`0`–`360` degrees): Compass direction of the center of the camera image (`0` = North, `90` = East, `180` = South, `270` = West).
   - `fov_deg` (`10`–`180` degrees): Horizontal field of view of the lens (typical fixed CCTV lenses: `70`–`110°`).
3. **Verify geometry**:
   - Two cameras covering an overlapping zone should view it from angled lines of sight (nearly parallel bearings are rejected to avoid inaccurate crossings).
   - Once configured, located alerts include `location: { lat, lng, accuracyM, cameras }` and automatically include OpenStreetMap links in Slack, Teams, Discord, and Email notifications.
