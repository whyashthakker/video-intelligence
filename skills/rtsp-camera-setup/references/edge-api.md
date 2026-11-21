# Edge Agent REST API Reference (`/api/v1/edge`)

All endpoints require an API key with `WRITE` or `ADMIN` scope (`Authorization: Bearer <API_KEY>`). Rate limit: `120 req/min`.

## 1. Heartbeat — `POST /api/v1/edge/heartbeat`
Send roughly every 60 seconds (cameras show `OFFLINE` after **3 minutes** without a heartbeat). Max **64 cameras** per request; optional base64 JPEG `snapshot` max **256 KB**.

```json
{
  "gateway": { "name": "warehouse-edge-1", "version": "1.0.0" },
  "cameras": [
    {
      "id": "dock-3",
      "name": "Dock 3",
      "location": "Warehouse A",
      "indexId": "<indexId>",
      "status": "online",
      "detail": null,
      "stream": { "codec": "h264", "width": 640, "height": 360, "fps": 15 },
      "snapshot": "<optional_base64_jpeg_under_256kb>"
    }
  ]
}
```
Camera `id` regex: `^[A-Za-z0-9][A-Za-z0-9_.-]{0,63}$`.

## 2. Presign Clip Upload — `POST /api/v1/edge/clips/presign`
The camera must have sent at least one heartbeat first (`404` otherwise). Max clip size: **512 MB**.
```json
{ "cameraId": "dock-3", "fileSizeBytes": 4200000 }
```
Response: `{ "presignedUrl": "https://...", "storageKey": "videos/<indexId>/cameras/<id>/...", "expiresIn": 3600 }`.
Upload the MP4 bytes directly via `PUT` with `Content-Type: video/mp4`.

## 3. Register Clip — `POST /api/v1/edge/clips`
Turns the uploaded MP4 into a searchable `Video` in the camera's index (default `standard` 5s-chunk profile for cost-efficient continuous camera footage).
```json
{
  "cameraId": "dock-3",
  "storageKey": "videos/<indexId>/cameras/<id>/1732110000-abc.mp4",
  "capturedAt": "2026-11-20T14:00:00Z",
  "durationSec": 30,
  "fileSizeBytes": 4200000,
  "motionScore": 0.08
}
```
Constraints:
- `durationSec`: `0 < durationSec ≤ 600` seconds.
- `capturedAt`: ISO 8601 within last **30 days** and ≤ **5 minutes** in the future.
- **Idempotent on retry**: Re-submitting the same `storageKey` returns `{ video, duplicate: true }` without double-billing.
