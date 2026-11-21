---
name: ceptory-edge-cctv-setup
description: Connect existing on-premises IP cameras, NVRs, DVRs, and analog CCTV systems to Ceptory using Ceptory Edge (outbound HTTPS only, no port forwarding, local RTSP credentials, local motion gating) and the Edge REST API (/api/v1/edge). Includes brand-specific H.264 sub-stream RTSP URL templates for Hikvision, Dahua, CP Plus, Axis, Reolink, Uniview, UniFi Protect, EZVIZ, TP-Link Tapo/VIGI, Hanwha, Bosch, i-PRO, Vivotek, Foscam, XMEye, and ONVIF discovery. Use whenever a user wants to connect CCTV cameras or an NVR/DVR to Ceptory, generate edge.toml, troubleshoot RTSP/ONVIF streams, or integrate custom edge hardware.
---

# Ceptory Edge & RTSP/ONVIF CCTV Integration

Connect existing LAN cameras, NVRs, and DVRs to Ceptory without replacing hardware, opening firewall ports, or exposing camera passwords to the internet.

## Architecture Principles

- **Outbound HTTPS only**: Ceptory Edge runs on a local machine on the camera LAN (mini-PC, NAS with Docker, Raspberry Pi 5, or Linux/Windows/Mac host) and makes outbound HTTPS calls to Ceptory and S3. **Never recommend port forwarding.**
- **Local credentials**: RTSP URLs and camera passwords stay in `edge.toml` on the edge machine (use `${ENV_VAR}` interpolation).
- **Motion-gated cost control**: Cuts RTSP streams into 10-second segments (stream copy, low CPU), checks local greyscale motion, and uploads only active clips + pre/post-roll so you don't pay to index empty hours.
- **Resilient outbox**: Clips queue locally (`max_outbox_mb`) and upload reliably after internet outages.

## Brand RTSP Sub-Stream URL Reference

Always use the **H.264 sub stream** (~640×360 or 640×480, 10–15 fps, Smart Codec / H.264+ / H.265+ disabled, I-frame interval ~2× fps) to minimize bandwidth and ensure crisp clip start boundaries. URL-encode special password characters (`@` → `%40`, `:` → `%3A`, `/` → `%2F`).

| Brand (and OEMs) | Default Port / User | Sub-Stream RTSP URL Template |
|---|---|---|
| **Hikvision** (HiLook, Prama, NVRs/DVRs) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/Streaming/Channels/{ch}02` (`{ch}01` = main) |
| **Dahua** (Amcrest, Lorex, Imou, NVRs/XVRs) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/cam/realmonitor?channel={ch}&subtype=1` (`subtype=0` = main) |
| **CP Plus** (DVRs, NVRs, HDCVI via DVR) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/cam/realmonitor?channel={ch}&subtype=1` |
| **Axis** | `554` / `ceptory` | `rtsp://{user}:{pass}@{host}:{port}/axis-media/media.amp?videocodec=h264&resolution=640x360&fps=15` |
| **Reolink** (PoE / wired Wi-Fi / NVRs) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/h264Preview_{ch2}_sub` (`ch2` = 2-digit, e.g. `01`; enable RTSP in Network → Port Settings) |
| **Uniview (UNV)** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/media/video2` (NVRs: `/unicast/c{ch}/s1/live`) |
| **UniFi Protect** | `7447` / *(none)* | `rtsp://{console_ip}:7447/{token}` (host = UniFi console; extract token from Protect RTSPS URL) |
| **EZVIZ** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/h264/ch1/sub/av_stream` (`pass` = 6-letter verification code) |
| **TP-Link Tapo / VIGI** | `554` / *(camera acct)* | `rtsp://{user}:{pass}@{host}:{port}/stream2` (`stream1` = main) |
| **Hanwha Vision (Wisenet)** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:554/profile{N}/media.smp` (use `ceptory-edge discover` for profile N) |
| **Bosch** | `554` / `service` | `rtsp://{user}:{pass}@{host}:{port}/?inst=2` |
| **i-PRO (Panasonic)** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/MediaInput/h264/stream_2` |
| **Vivotek** | `554` / `root` | `rtsp://{user}:{pass}@{host}:{port}/live2.sdp` |
| **Foscam** | `554` (older: `88`) / `admin` | `rtsp://{user}:{pass}@{host}:{port}/videoSub` |
| **XMEye / NetSurveillance** | `554` / `admin` | `rtsp://{host}:{port}/user={user}&password={pass}&channel={ch}&stream=1.sdp?real_stream` |
| **Any ONVIF Profile S camera** | `554` / `admin` | Run `ceptory-edge discover --username admin` on the same VLAN |

> For analog CCTV (HD-TVI/CVI/AHD), connect to the DVR's local IP and add one `[[cameras]]` block per DVR channel. Use a dedicated viewer-only account instead of `admin` whenever supported.

## Generating `edge.toml`

```toml
[ceptory]
url = "https://www.ceptory.com"
api_key = "${CEPTORY_API_KEY}"       # WRITE API key from Dashboard -> API keys
gateway_name = "warehouse-edge-1"
heartbeat_seconds = 60

[storage]
data_dir = "/data"
keep_uploaded_clips = false
max_outbox_mb = 5120

[defaults]
index_id = "<index_id>"              # From /dashboard/indexes/<index_id>
upload_mode = "motion"               # "motion" (cheapest) | "continuous" | "off"
segment_seconds = 10
pre_roll_segments = 1                # 10s pre-roll
post_roll_segments = 1               # 10s post-roll
max_event_seconds = 60
motion_sensitivity = 0.02            # 2% changed pixels triggers clip
pixel_threshold = 25                 # brightness delta (0-255)
sample_fps = 2
transcode = false                    # set true only if camera forces H.265

[[cameras]]
id = "dock-3"                        # 1-64 chars: A-Za-z0-9_.-
name = "Dock 3"
location = "Warehouse A"
url = "rtsp://viewer:${DOCK3_PASS}@192.168.1.64:554/Streaming/Channels/102"
# Mask on-screen timestamp clock so ticking seconds don't trigger motion: [x, y, w, h]
ignore_regions = [[0.0, 0.0, 0.45, 0.08]]
```

For testing without a physical camera, set `url = "test:pattern"` to emit a synthetic motion clip every 5 minutes.

## CLI & Docker Commands

```bash
# 1. Discover ONVIF cameras on the LAN (needs same VLAN; --network host in Docker)
python3 -m ceptory_edge discover --username admin

# 2. Test a single RTSP stream and inspect codec/resolution/fps
python3 -m ceptory_edge test-camera "rtsp://viewer:pass@192.168.1.64:554/Streaming/Channels/102" --snapshot test.jpg

# 3. Validate config, API key, index_id, and every camera stream (uploads nothing)
python3 -m ceptory_edge check --config edge.toml

# 4. Run continuously (or via Docker)
python3 -m ceptory_edge run --config edge.toml

docker run -d --name ceptory-edge --restart unless-stopped --network host \
  -e CEPTORY_API_KEY="$CEPTORY_API_KEY" \
  -v "$PWD/edge.toml:/config/edge.toml:ro" \
  -v ceptory-edge-data:/data ceptory-edge
```

## Custom Edge Integration API (`/api/v1/edge`)

Requires a `WRITE` (or `ADMIN`) workspace API key (`Authorization: Bearer <API_KEY>`).

1. **Heartbeat** (`POST /api/v1/edge/heartbeat`): Send ~every 60 seconds (cameras mark `OFFLINE` after 3 minutes without heartbeat). Up to **64 cameras** per heartbeat; optional base64 JPEG `snapshot` up to **256 KB**.
   ```json
   {
     "gateway": { "name": "site-1", "version": "1.0.0" },
     "cameras": [
       { "id": "dock-3", "name": "Dock 3", "location": "Warehouse A", "indexId": "<indexId>", "status": "online", "stream": { "codec": "h264", "width": 640, "height": 360, "fps": 15 } }
     ]
   }
   ```
2. **Presign Clip Upload** (`POST /api/v1/edge/clips/presign`): Body `{ "cameraId": "dock-3", "fileSizeBytes": 4200000 }` (max **512 MB**). Returns `{ presignedUrl, storageKey, expiresIn }`. `PUT` MP4 bytes to `presignedUrl` with `Content-Type: video/mp4`.
3. **Register Clip** (`POST /api/v1/edge/clips`): Body `{ "cameraId": "dock-3", "storageKey": "...", "capturedAt": "2026-11-20T14:00:00Z", "durationSec": 30, "fileSizeBytes": 4200000, "motionScore": 0.08 }`. Max clip length **600 seconds**; `capturedAt` within last 30 days (max 5 min future skew). Idempotent on retry (`duplicate: true`).

## Troubleshooting Checklist

- **Stream fails in `check`**: Test the exact URL in VLC from the edge host; check port 554 reachability, special-character URL encoding, and account lockout (Hikvision/CP Plus lock logins ~30 min after failed attempts).
- **Camera online, no clips arriving**: Lower `motion_sensitivity` (e.g., `0.01`), confirm `upload_mode = "motion"`, or mask ticking clocks with `ignore_regions`.
- **ONVIF `discover` finds nothing**: Verify ONVIF is enabled on the camera, NTP clocks are synced (ONVIF digests reject skewed clocks), and Docker uses `--network host` on the camera VLAN.
