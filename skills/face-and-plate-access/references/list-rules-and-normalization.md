# People Lists & Plate List Rules Reference

## Face Access (`unauthorized_access`)

### List Naming Syntax (`add_camera_person`)
| `list` Value | Behavior |
|---|---|
| `"staff"` *(default)* | Allowed person — suppresses unknown-face alert. |
| `"watchlist"` | Critical alert immediately when matched (ignores `alertUnknown: false`). |
| `"allow:<List name>"` | Custom allowed list (name ≤ 40 chars, e.g. `"allow:Contractors"`). |
| `"alert:<List name>"` | Custom alert-on-sight list (e.g. `"alert:Banned"`). |

### Photo & Threshold Guidelines
- Up to **5 photos per person**: pass `person_id` on subsequent `add_camera_person` calls to add front, slight left/right profile, and glasses/no-glasses angles.
- Image format: JPEG/PNG base64 or `data:` URL, max **8 MB**. Only the face embedding is used for matching.
- `matchThreshold` calibration (`0.3`–`0.9`, default `0.5`):
  - `0.45`: Poor lighting or angled doorway cameras where staff are missed too often.
  - `0.50`: Balanced default for well-lit indoor entrances.
  - `0.60`–`0.70`: High-security doors (server rooms, vaults) requiring strict match confidence.

## ANPR Parking (`illegal_parking`)

### Plate Normalization
- Uppercased and stripped of non-alphanumeric characters (`[^A-Z0-9]`): `"MH-12 AB 1234"` → `"MH12AB1234"`.
- Valid normalized length: `2`–`12` characters (shorter/longer entries are ignored).

### List Limits & Behavior (`update_plate_list`)
- **`"Authorized"`** (built-in default): Always allowed; cannot be deleted (`delete_list: true` fails — use `remove: [...]` instead).
- **Named lists**: Up to **10** named lists per camera, up to **500** plates per list.
  - `alert: false` *(default on creation)*: Allowed vehicles (e.g. `"Residents"`, `"Fleet"`).
  - `alert: true`: Fires a **critical alert on sight**, bypassing `graceMin` (e.g. `"Banned"`, `"Stolen"`).
- `graceMin`: `0`–`1440` minutes (default `10`). Set `0` for fire lanes or tow-away bays.
