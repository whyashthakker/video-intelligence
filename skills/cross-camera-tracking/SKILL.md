---
name: cross-camera-tracking
description: Group cameras by physical site or area, trace cross-camera person and vehicle re-identification (Re-ID) journeys with sighting thumbnails, geocode site addresses, and configure coordinates, compass headings, and field-of-view (FOV) for automatic multi-camera GPS event triangulation via MCP. Use whenever a user wants to organize cameras into sites, trace where a person or vehicle went across cameras, follow a license plate, or pinpoint alerts on a map.
license: MIT
metadata:
  version: "1.0"
---

# Cross-Camera Re-ID Tracking & Multi-Camera Geo-Triangulation

Turn isolated camera sightings into site-wide person/vehicle journeys and map-pinpointed events.

## Part 1: Group Cameras & Enable Tracking

1. Discover or create a site group: `list_camera_groups` → `create_camera_group({ name, description })`.
2. Assign cameras to the group and enable Re-ID via `update_camera({ camera_id, group_id, track_people: true })`.
   - `track_people: true` tracks **both people and vehicles** (`car`, `truck`, `bus`, `motorcycle`, `bicycle`), linked across cameras within 45 minutes (30-day retention).
   - Confirm with the user before `delete_camera_group` (cameras remain ungrouped).

## Part 2: Trace Journeys (`get_movement`)

1. Call `get_movement({ group_id, hours, kind, plate })` (`group_id` can be `"all"` for workspace-wide; `hours` `0.25`–`72`; optional `kind`: `"person"` | `"vehicle"`, optional `plate`).
2. Call `get_sighting_image({ sighting_id })` on key stops to visually verify continuity before escalating.
3. **Mandatory accuracy rule**: Always describe appearance-based cross-camera matches as **"likely"** unless confirmed by an exact plate read or visual thumbnail verification.
4. If no multi-camera journeys appear, verify via `list_cameras` that at least two cameras in the group have `track_people: true` and are `online`.

## Part 3: Map Placement & GPS Triangulation

1. Call `geocode_address({ query })` to get baseline `latitude` / `longitude`.
2. Set `latitude`, `longitude`, `heading` (`0`–`360°`, `0 = North`), and `fov_deg` (`10`–`180°`) via `update_camera` on overlapping cameras (send `latitude` and `longitude` together, or both `null` to clear).
3. Ensure overlapping cameras view shared zones from angled bearings (nearly parallel sight lines won't triangulate).

## References & Examples

- Read [references/reid-and-triangulation.md](references/reid-and-triangulation.md) for exact Re-ID similarity weights, thresholds, bearing math, and triangulation limits.
- Read [examples/campus-journey-trace.md](examples/campus-journey-trace.md) for a worked multi-camera site setup and plate trace.
