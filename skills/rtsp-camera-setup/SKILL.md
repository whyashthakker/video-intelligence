---
name: rtsp-camera-setup
description: Connect existing on-premises IP cameras, NVRs, DVRs, and analog CCTV to cloud video search over outbound HTTPS (no port forwarding, local credentials, local motion gating) using an edge agent. Includes H.264 sub-stream RTSP URL templates for 15+ brands (Hikvision, Dahua, CP Plus, Axis, Reolink, Uniview, UniFi Protect, EZVIZ, Tapo, Hanwha, Bosch, ONVIF) and edge.toml generation. Use whenever a user wants to connect CCTV or an NVR/DVR, find RTSP URLs, or troubleshoot ONVIF streams.
license: MIT
metadata:
  version: "1.0"
---

# On-Prem RTSP & ONVIF CCTV Camera Integration

Connect existing LAN cameras, NVRs, and DVRs to cloud video search using the edge agent (`ceptory_edge`, Python 3.11+ stdlib + `ffmpeg`, or Docker) with outbound HTTPS only and local motion gating.

## Architecture Guardrails

- **Never recommend port forwarding or exposing RTSP to the internet.** The edge agent runs on a host on the camera LAN (mini-PC, NAS, Raspberry Pi 5, server) and makes outbound HTTPS requests only.
- **Keep credentials local**: Store RTSP passwords in environment variables interpolated in `edge.toml` (`${CAMERA_PASS}`), using a dedicated viewer-only camera user where supported.
- **Always use the H.264 sub stream** (~`640×360`, `10–15 fps`, Smart Codec / `H.264+` off) and mask on-screen clocks via `ignore_regions`.

## Setup Procedure

1. **Identify camera brand & build RTSP sub-stream URL**: Read [references/rtsp-brand-templates.md](references/rtsp-brand-templates.md) for exact templates across 15+ brands (Hikvision, Dahua, CP Plus, Axis, Reolink, Uniview, UniFi Protect, EZVIZ, Tapo/VIGI, Hanwha, Bosch, i-PRO, Vivotek, Foscam, XMEye, ONVIF).
2. **Generate `edge.toml`**: Start from [examples/edge-warehouse.toml](examples/edge-warehouse.toml) (`url = "test:pattern"` works with no camera for pipeline testing).
3. **Discover, test, validate, and run**:
   ```bash
   python3 -m ceptory_edge discover --username admin
   python3 -m ceptory_edge test-camera "<rtsp_url>" --snapshot test.jpg
   python3 -m ceptory_edge check --config edge.toml
   python3 -m ceptory_edge run --config edge.toml
   ```
   Or in Docker with `--network host` (required for LAN RTSP and ONVIF multicast discovery).
4. **Custom edge agents**: Read [references/edge-api.md](references/edge-api.md) when implementing `/api/v1/edge/heartbeat`, `/clips/presign`, and `/clips` directly.

## Troubleshooting

- **Stream fails in `check`**: Test the URL in VLC from the edge box; verify port `554`, URL-encoded password characters (`@`→`%40`, `:`→`%3A`, `/`→`%2F`), and Hikvision/CP Plus ~30-min login lockouts.
- **Online, no clips**: Lower `motion_sensitivity` (e.g. `0.01`), check `upload_mode = "motion"`, or mask ticking OSD clocks in `ignore_regions`.
- **ONVIF `discover` empty**: Ensure ONVIF is enabled, camera NTP clocks are synced, and edge host/Docker is on the camera VLAN with `--network host`.

## References & Examples

- Read [references/rtsp-brand-templates.md](references/rtsp-brand-templates.md) when looking up brand RTSP URLs or sub-stream quirks.
- Read [references/edge-api.md](references/edge-api.md) when integrating custom hardware with `/api/v1/edge`.
- Read [examples/edge-warehouse.toml](examples/edge-warehouse.toml) when generating a multi-camera `edge.toml`.
