---
name: alert-triage
description: Triage live security and safety camera alerts by checking fleet health (online/offline/error), paginating open alerts, visually inspecting annotated evidence images before resolving false alarms, and configuring or verifying Slack, Microsoft Teams, Discord, Email, and HMAC-SHA256 webhook channels via Ceptory. Use whenever a user asks to review open alerts, run a shift check on camera health, clear false alarms, or verify alert webhook signatures.
---

# Security & Safety Alert Triage & Notification Routing

Run systematic security/ops alert triage with visual verification and configure reliable multi-channel alert delivery.

## Part 1: Alert Triage & Shift Health Workflow (MCP)

### 1. Check Fleet Health First
Call `list_cameras` to identify:
- Cameras in `offline` or `error` status (separate from intentionally `paused` cameras).
- Cameras with high `openAlerts` or `alertsToday` spikes (may indicate misconfigured zones, lighting glare, or an active incident).
- For any suspicious camera, call `get_camera_snapshot({ camera_id })` to verify the lens is unobstructed and well-lit.

### 2. Fetch Open Alerts (Newest First)
Call `list_alerts`:
```json
{
  "status": "open",
  "camera_id": "<optional cameraId>",
  "limit": 40
}
```
- Paginate older alerts by passing `before` set to the ISO `createdAt` timestamp of the last returned alert.
- Valid `status` filters: `"open"`, `"acknowledged"`, `"resolved"`, `"all"` (default `"all"`).

### 3. Visually Verify Before Resolving
- **Never bulk-resolve alerts blindly.** Call `get_alert_image({ alert_id })` to inspect the annotated frame captured when the alert fired (returns image content + a 10-minute signed link).
- Classify each alert:
  - **Confirmed incident**: Call `update_alert({ alert_id, status: "acknowledged" })` while under active response; summarize camera, time, severity, observed evidence, and any triangulated GPS location.
  - **False alarm** (e.g., reflection, shadow, authorized personnel outside schedule, non-target object): Call `update_alert({ alert_id, status: "resolved" })` and recommend a concrete camera tuning fix (adjust `zone`, `cooldownMin`, `sensitivity`, or `schedule` via `update_camera`).
  - **Re-open**: If resolved prematurely, set `status: "open"`.

### 4. Produce a Shift Triage Summary
Report:
1. Camera fleet status (`online`, `offline`, `paused`, `error` counts + names of unhealthy cameras).
2. Open / acknowledged confirmed incidents ordered by severity (`critical` → `warning` → `info`) and timestamp.
3. Resolved false alarms with recommended config adjustments.

## Part 2: Workspace Notification Channels

Ceptory supports up to **20 workspace-wide notification channels** configured via `GET /api/cameras/channels` and `PUT /api/cameras/channels` (or in the Ceptory dashboard), plus per-camera Discord webhooks via `create_camera` / `update_camera` and `test_discord_webhook`.

### Supported Channel Types

| `type` | `target` Format | Notes |
|---|---|---|
| `slack` | `https://hooks.slack.com/(services\|workflows\|triggers)/...` | Sends rich Block Kit message with camera, workflow, severity, image, Ceptory link, and OpenStreetMap link if geo-located. |
| `teams` | `https://<host>` on `webhook.office.com`, `logic.azure.com`, or `environment.api.powerplatform.com` | Sends Adaptive Card v1.4 with facts, image, and OpenUrl actions. |
| `discord` | `https://discord.com/api/webhooks/...` | Testable via MCP tool `test_discord_webhook({ url, camera_name })`. |
| `email` | Up to 10 comma-separated email addresses | Formatted HTML + plain-text alert with snapshot and map link. |
| `webhook` | Public `https://` hostname (no raw IPs, `localhost`, `.local`, or `.internal`) | Signs payload with HMAC-SHA256 using an auto-generated per-channel `secret`. |

### Scoping & Severity Routing
Each channel object supports:
- `minSeverity`: `"info"` | `"warning"` | `"critical"` (default `"warning"`).
- `groupIds`: Array of site/group IDs (empty = all groups).
- `cameraIds`: Array of camera IDs (empty = all cameras).
- `enabled`: Boolean toggle without deleting the channel.

## Part 3: Verifying Custom Ceptory Alert Webhooks

When `type` is `"webhook"`, Ceptory sends `POST` with `Content-Type: application/json`, `User-Agent: Ceptory-Alerts/1.0`, and two signature headers:
- `X-Ceptory-Timestamp`: Unix epoch seconds (string)
- `X-Ceptory-Signature`: `sha256=<hex>` computed as `HMAC-SHA256(secret, "${timestamp}.${rawBody}")`

### Webhook JSON Payload
```json
{
  "event": "camera.alert",
  "alert": {
    "id": "cm...",
    "title": "PPE missing",
    "detail": "Worker without helmet in Loading dock",
    "severity": "warning",
    "kind": "ppe",
    "createdAt": "2026-11-20T14:22:05.000Z",
    "url": "https://www.ceptory.com/dashboard/...",
    "imageUrl": "https://...signed-7d-url...",
    "location": { "lat": 19.021, "lng": 73.018, "accuracyM": 25 },
    "data": {}
  },
  "camera": {
    "id": "cm...",
    "name": "Loading dock",
    "workflow": "PPE check",
    "groupId": "grp...",
    "group": "Plant 2"
  }
}
```

### Signature Verification Example (Node.js / TypeScript)
```ts
import { createHmac, timingSafeEqual } from "node:crypto";

export function verifyCeptoryWebhook(rawBody: string, headers: Headers, secret: string): boolean {
  const ts = headers.get("x-ceptory-timestamp");
  const sig = headers.get("x-ceptory-signature");
  if (!ts || !sig?.startsWith("sha256=")) return false;
  // Reject replays older than 5 minutes
  if (Math.abs(Date.now() / 1000 - Number(ts)) > 300) return false;

  const expected = createHmac("sha256", secret).update(`${ts}.${rawBody}`).digest("hex");
  const actual = sig.slice("sha256=".length);
  if (expected.length !== actual.length) return false;
  return timingSafeEqual(Buffer.from(expected, "hex"), Buffer.from(actual, "hex"));
}
```

Always verify against the **raw request body** before JSON parsing and use constant-time comparison (`timingSafeEqual`).
