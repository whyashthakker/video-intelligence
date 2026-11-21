# Re-ID Matching & Multi-Camera Triangulation Reference

## Cross-Camera Re-Identification (`camera-reid`)
- **Tracked classes**: `person` and vehicles (`car`, `truck`, `bus`, `motorcycle`, `bicycle`) whenever `track_people: true` is enabled on the camera.
- **Similarity formula**:
  - **Plate override (vehicles)**: If both sightings have a readable plate (score `≥ 0.5`), exact match or edit-distance `≤ 1` (plate length `≥ 5`) scores `0.99`; different plates score `0.20`.
  - **Appearance score**: `0.75 * cosine(CLIP_embedding) + 0.25 * color_histogram_overlap`.
  - **Face boost (people)**: If both sightings have a face embedding, cosine `≥ 0.5` boosts score to `≥ 0.95`; cosine `< 0.2` caps score at `≤ 0.50`.
- **Thresholds & windows**:
  - Same-camera merge: `0.82` (person), `0.86` (vehicle) within `30 seconds`.
  - Cross-camera link: `0.78` (person), `0.84` (vehicle) across different cameras within `45 minutes`.
  - Sighting retention: `30 days`.

## Multi-Camera Geo-Triangulation (`camera-geo` / `camera-locate`)
- **Camera fields required**: `latitude` (`-90`..`90`), `longitude` (`-180`..`180`), `heading` (`0`..`360`°, where `0° = North`, `90° = East`), and `fov_deg` (`10`..`180`°).
- **Bearing math**: Detection box horizontal center `cx ∈ [0, 1]` maps to `bearing = (heading + (cx - 0.5) * fov_deg) mod 360`.
- **Triangulation trigger**: Two or more placed cameras report the same alert `kind` (and same `hazard` for `hazards`) within **3 minutes**.
- **Rejection guards**:
  - Nearly parallel sight lines (`|det| < 0.05`) are discarded.
  - Intersections behind a camera or farther than **3,000 meters** are discarded.
- **Result stored on alerts**: `{ lat, lng, accuracyM, method: "triangulated", cameras }` — automatically rendered as OpenStreetMap links in Slack, Teams, Discord, and Email alerts.
