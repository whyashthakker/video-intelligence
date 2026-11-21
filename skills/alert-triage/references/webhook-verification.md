# Notification Channels & Webhook Signature Verification

## Table of Contents
- [Workspace Channel Rules](#workspace-channel-rules)
- [Webhook JSON Payload Schema](#webhook-json-payload-schema)
- [HMAC-SHA256 Verification (TypeScript / Node.js)](#hmac-sha256-verification-typescript--nodejs)
- [HMAC-SHA256 Verification (Python)](#hmac-sha256-verification-python)

## Workspace Channel Rules
Endpoint: `GET /api/cameras/channels` and `PUT /api/cameras/channels` (up to **20 channels** per workspace).

| `type` | `target` Requirement | Routing Fields |
|---|---|---|
| `slack` | `https://hooks.slack.com/(services\|workflows\|triggers)/...` | `minSeverity` (`info`\|`warning`\|`critical`), `groupIds`, `cameraIds`, `enabled` |
| `teams` | `https://` on `webhook.office.com`, `logic.azure.com`, or `environment.api.powerplatform.com` | Same |
| `discord` | `https://discord.com/api/webhooks/...` (test via MCP `test_discord_webhook`) | Same |
| `email` | Up to 10 comma-separated email addresses | Same |
| `webhook` | Public `https://` hostname (no raw IPs, `localhost`, `.local`, `.internal`, or private DNS ranges) | Auto-generates per-channel HMAC `secret` |

Empty `groupIds` and `cameraIds` match every camera in the workspace.

## Webhook JSON Payload Schema
Headers sent on every `POST`:
- `Content-Type: application/json`
- `User-Agent: Ceptory-Alerts/1.0`
- `X-Ceptory-Timestamp`: Unix epoch seconds (string)
- `X-Ceptory-Signature`: `sha256=<hex>` where signature is `HMAC-SHA256(secret, "${timestamp}.${rawBody}")`

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

## HMAC-SHA256 Verification (TypeScript / Node.js)
```ts
import { createHmac, timingSafeEqual } from "node:crypto";

export function verifyAlertWebhook(rawBody: string, headers: Headers, secret: string): boolean {
  const ts = headers.get("x-ceptory-timestamp");
  const sig = headers.get("x-ceptory-signature");
  if (!ts || !sig?.startsWith("sha256=")) return false;
  if (Math.abs(Date.now() / 1000 - Number(ts)) > 300) return false; // 5-min replay window

  const expected = createHmac("sha256", secret).update(`${ts}.${rawBody}`).digest("hex");
  const actual = sig.slice("sha256=".length);
  if (expected.length !== actual.length) return false;
  return timingSafeEqual(Buffer.from(expected, "hex"), Buffer.from(actual, "hex"));
}
```

## HMAC-SHA256 Verification (Python)
```python
import hashlib, hmac, time

def verify_alert_webhook(raw_body: bytes, headers: dict, secret: str) -> bool:
    ts = headers.get("X-Ceptory-Timestamp") or headers.get("x-ceptory-timestamp")
    sig = headers.get("X-Ceptory-Signature") or headers.get("x-ceptory-signature")
    if not ts or not sig or not sig.startswith("sha256="):
        return False
    if abs(time.time() - int(ts)) > 300:
        return False
    expected = hmac.new(secret.encode(), f"{ts}.".encode() + raw_body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, sig[len("sha256="):])
```
