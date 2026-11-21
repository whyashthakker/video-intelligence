# Examples: Plain-English Camera Rule Translations

## 1. Loading Dock PPE + Discord Alert
**Request**: *"Set up a camera called Loading dock that alerts our Discord when anyone isn't wearing a helmet and hi-vis vest, and record 30s clips."*

**Confirm in one line**: Creating `Loading dock` (`ppe`, helmet + vest required, 5 min cooldown, 30s alert clips, Discord webhook attached).
```json
{
  "tool": "create_camera",
  "args": {
    "name": "Loading dock",
    "workflow": "ppe",
    "config": {
      "intervalSec": 10,
      "require": { "helmet": true, "vest": true, "gloves": false },
      "cooldownMin": 5,
      "response": { "clipSec": 30, "push": true, "alarm": false }
    },
    "discord_webhook_url": "https://discord.com/api/webhooks/..."
  }
}
```

## 2. Overnight Perimeter Intrusion (Left Half Only)
**Request**: *"Watch the back fence for people or vehicles between 10pm and 6am India time. Only the left half of the frame matters."*

```json
{
  "tool": "create_camera",
  "args": {
    "name": "Back fence",
    "workflow": "intrusion",
    "config": {
      "intervalSec": 4,
      "classes": ["person", "vehicle"],
      "schedule": { "mode": "hours", "from": "22:00", "to": "06:00", "tz": "Asia/Kolkata" },
      "cooldownMin": 5
    },
    "zone": { "x": 0, "y": 0, "w": 0.5, "h": 1 }
  }
}
```

## 3. Safe Partial Update (Always Merge Existing Config!)
**Request**: *"Raise the crowd limit on the lobby camera to 25 and pause the parking camera."*

1. Call `list_cameras({})` → find IDs for `Lobby` and `Parking`.
2. Call `get_camera({ camera_id: "<lobbyId>" })` → read existing `config` (e.g., `{ intervalSec: 10, maxPeople: 10, cooldownMin: 10, response: { clipSec: 20, push: true, alarm: true } }`).
3. Call `update_camera` merging existing keys so `alarm: true` and `cooldownMin` aren't reset:
   ```json
   {
     "camera_id": "<lobbyId>",
     "config": { "intervalSec": 10, "maxPeople": 25, "cooldownMin": 10, "response": { "clipSec": 20, "push": true, "alarm": true } }
   }
   ```
4. Pause parking camera:
   ```json
   { "camera_id": "<parkingId>", "enabled": false }
   ```

## 4. False-Alarm Tuning Playbook
- **Passing sidewalk traffic triggering `intrusion`**: Shrink `zone` inward by 10–15% away from the road edge.
- **Steam/glare triggering `hazards`**: Lower `sensitivity` to `"low"` and restrict `hazards` to `["fire", "smoke"]`.
- **Delivery drivers triggering `illegal_parking`**: Increase `graceMin` to `15` or add fleet plates via `update_plate_list`.
