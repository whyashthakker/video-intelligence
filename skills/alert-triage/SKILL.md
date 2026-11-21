---
name: alert-triage
description: Triage live security and safety camera alerts by checking fleet health (online/offline/error), paginating open alerts, visually inspecting annotated evidence images before resolving false alarms, and configuring or verifying Slack, Microsoft Teams, Discord, Email, and HMAC-SHA256 webhook channels via MCP. Use whenever a user asks to review open alerts, run a shift check on camera health, clear false alarms, or verify alert webhook signatures.
license: MIT
metadata:
  version: "1.0"
---

# Security & Safety Alert Triage & Notification Routing

Run disciplined camera fleet checks, visually verify every alert before closing it, and integrate signed multi-channel alert delivery.

## 4-Step Shift Triage Workflow (MCP)

1. **Check fleet health**: Call `list_cameras` to flag any `offline` or `error` cameras (separate from intentionally `paused` ones) and spot alert spikes. Inspect questionable views with `get_camera_snapshot({ camera_id })`.
2. **Fetch open alerts**: Call `list_alerts({ status: "open", camera_id, limit: 40 })`. Paginate older pages by passing `before` set to the last alert's `createdAt` ISO timestamp.
3. **Visually verify before changing status**:
   - **Never bulk-resolve alerts blind.** Call `get_alert_image({ alert_id })` to inspect the annotated evidence image.
   - **Confirmed incident**: Call `update_alert({ alert_id, status: "acknowledged" })` and include camera, timestamp, severity, evidence summary, and any triangulated GPS location.
   - **False alarm**: Call `update_alert({ alert_id, status: "resolved" })` and propose a concrete tuning adjustment (shrink `zone`, raise `cooldownMin`, adjust `schedule` or `sensitivity`).
4. **Deliver a concise shift summary**: Fleet counts + unhealthy camera names, confirmed incidents sorted `critical → warning → info`, and resolved false alarms with tuning notes.

## Notification Channels & Webhooks

- Workspace channels (`GET`/`PUT /api/cameras/channels`, up to 20 channels): `slack`, `teams`, `discord`, `email`, and custom HMAC-signed `webhook`, each scoped by `minSeverity`, `groupIds`, and `cameraIds`.
- Test Discord webhooks via MCP `test_discord_webhook({ url, camera_name })`.
- **When implementing or debugging a webhook receiver**, read [references/webhook-verification.md](references/webhook-verification.md) for the exact `camera.alert` payload and constant-time HMAC-SHA256 verification (`X-Ceptory-Timestamp` + `X-Ceptory-Signature`) in TypeScript and Python.

## References & Examples

- Read [references/webhook-verification.md](references/webhook-verification.md) when configuring notification channels or verifying webhook signatures.
- Read [examples/shift-triage-report.md](examples/shift-triage-report.md) for a complete shift triage transcript and report format.
