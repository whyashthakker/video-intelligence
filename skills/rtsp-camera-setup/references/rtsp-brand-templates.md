# Brand-Specific H.264 Sub-Stream RTSP URL Directory

## Table of Contents
- [Sub-Stream Encoding Rules](#sub-stream-encoding-rules)
- [RTSP URL Templates (15+ Brands)](#rtsp-url-templates-15-brands)
- [Brand-Specific Notes & Gotchas](#brand-specific-notes--gotchas)

## Sub-Stream Encoding Rules
- Always configure the **Sub Stream** (extra stream) to plain **H.264**, ~`640×360` or `640×480`, `10–15 fps`, I-frame interval `2× fps`.
- **Disable Smart Codec** (`H.264+`, `H.265+`, `U-Code`, `Smart Codec`) — long keyframe gaps cause clips to start late.
- **URL-encode password characters**: `@` → `%40`, `:` → `%3A`, `/` → `%2F`, `#` → `%23`.
- Create a dedicated viewer-only user instead of using `admin` whenever supported.

## RTSP URL Templates (15+ Brands)

| Brand (& OEMs) | Port / Default User | Sub-Stream RTSP URL | Main Stream |
|---|---|---|---|
| **Hikvision** (HiLook, Prama, NVRs/DVRs) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/Streaming/Channels/{ch}02` | `.../Channels/{ch}01` |
| **Dahua** (Amcrest, Lorex, Imou, NVRs/XVRs) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/cam/realmonitor?channel={ch}&subtype=1` | `...&subtype=0` |
| **CP Plus** (DVRs, NVRs, HDCVI via DVR) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/cam/realmonitor?channel={ch}&subtype=1` | `...&subtype=0` |
| **Axis** | `554` / `ceptory` | `rtsp://{user}:{pass}@{host}:{port}/axis-media/media.amp?videocodec=h264&resolution=640x360&fps=15` | `...media.amp?videocodec=h264` |
| **Reolink** (PoE / wired Wi-Fi / NVRs) | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/h264Preview_{ch2}_sub` | `..._{ch2}_main` |
| **Uniview (UNV)** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/media/video2` (NVR: `/unicast/c{ch}/s1/live`) | `/media/video1` |
| **UniFi Protect** | `7447` / *(none)* | `rtsp://{console_ip}:7447/{token}` | Same (pick quality in Protect) |
| **EZVIZ** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/h264/ch1/sub/av_stream` | `.../main/av_stream` |
| **TP-Link Tapo / VIGI** | `554` / *(camera acct)* | `rtsp://{user}:{pass}@{host}:{port}/stream2` | `/stream1` |
| **Hanwha Vision (Wisenet)** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:554/profile{N}/media.smp` | Pick profile via `discover` |
| **Bosch** | `554` / `service` | `rtsp://{user}:{pass}@{host}:{port}/?inst=2` | `/?inst=1` |
| **i-PRO (Panasonic)** | `554` / `admin` | `rtsp://{user}:{pass}@{host}:{port}/MediaInput/h264/stream_2` | `.../stream_1` |
| **Vivotek** | `554` / `root` | `rtsp://{user}:{pass}@{host}:{port}/live2.sdp` | `/live.sdp` |
| **Foscam** | `554` (old: `88`) / `admin` | `rtsp://{user}:{pass}@{host}:{port}/videoSub` | `/videoMain` |
| **XMEye / NetSurveillance** | `554` / `admin` | `rtsp://{host}:{port}/user={user}&password={pass}&channel={ch}&stream=1.sdp?real_stream` | `...stream=0.sdp?...` |
| **Any ONVIF Profile S** | `554` / `admin` | Run `python3 -m ceptory_edge discover --username admin` on camera VLAN | Listed by `discover` |

## Brand-Specific Notes & Gotchas
- **Hikvision / CP Plus**: Repeated wrong passwords lock logins for ~30 minutes; wait for lock expiry before retrying. For ONVIF discovery on Hikvision, enable Integration Protocol and add a separate ONVIF user.
- **CP Plus mobile app (gCMOB)**: Cloud P2P relay does not provide RTSP; The edge agent must run on the recorder's local LAN using its TCP/IP address.
- **Reolink**: RTSP ships disabled on newer firmware — turn it on in `Settings → Network → Advanced → Port Settings`. `ch2` is zero-padded (`01`, `02`). Battery models typically lack RTSP.
- **UniFi Protect**: Host is the UniFi Console (UDM/Cloud Key/NVR), not the camera. Strip `rtsps://...:7441/<token>?enableSrtp` down to plain LAN `rtsp://<console>:7447/<token>`.
- **Tapo**: Create a dedicated Camera Account in Tapo app (`Settings → Advanced Settings → Camera Account`); do not use your TP-Link cloud login.
- **Analog CCTV (HD-TVI/CVI/AHD)**: Point `edge.toml` at the DVR's local IP with one `[[cameras]]` block per channel number.
