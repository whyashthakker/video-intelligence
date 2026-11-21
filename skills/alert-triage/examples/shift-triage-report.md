# Example: Morning Shift Alert Triage Session

## User Prompt
> "Which cameras are offline right now? Show me today's open alerts with their photos, acknowledge real ones, and resolve false alarms."

## Execution Trace
1. `list_cameras({})` → 6 cameras: 5 `online`, 1 `offline` (`East Perimeter`, last seen 02:14 UTC).
2. `list_alerts({ status: "open", limit: 20 })` → returns 3 open alerts (`al_1`, `al_2`, `al_3`).
3. Visually verify every open alert before changing status:
   - `get_alert_image({ alert_id: "al_1" })` — `Loading Dock` (`ppe`, warning): Worker in blue jacket clearly missing hard hat inside dock bay → **confirmed**. Call `update_alert({ alert_id: "al_1", status: "acknowledged" })`.
   - `get_alert_image({ alert_id: "al_2" })` — `Back Fence` (`intrusion`, warning): Headlight reflection on wet asphalt outside fence line, no person/vehicle inside perimeter → **false alarm**. Call `update_alert({ alert_id: "al_2", status: "resolved" })`.
   - `get_alert_image({ alert_id: "al_3" })` — `Fire Lane` (`illegal_parking`, critical): White box truck parked directly in striped fire lane past grace period → **confirmed**. Call `update_alert({ alert_id: "al_3", status: "acknowledged" })`.

## Output Format Returned to Operator
- **Fleet health**: 5/6 online. **Offline**: `East Perimeter` (last seen 02:14 UTC — check PoE switch / edge host).
- **Confirmed open incidents (acknowledged)**:
  1. `[CRITICAL]` `Fire Lane` (07:41) — White box truck in fire lane (`al_3`).
  2. `[WARNING]` `Loading Dock` (08:05) — Worker without helmet in bay 2 (`al_1`).
- **Resolved false alarms**:
  - `Back Fence` (06:18, `al_2`) — Road headlight glare outside fence. Recommendation: shrink `zone` bottom edge from `y: 0.0, h: 1.0` to `h: 0.82` via `update_camera`.
