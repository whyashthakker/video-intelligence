# Extracting Frames with ffmpeg for Pixel-Level Inspection

Use these commands **after** the search tools return exact timestamps (`startSec` / `endSec`) so you only extract the frames that matter.

## 1. Extract a Single Frame at an Exact Timestamp
```bash
# Fast seek (-ss before -i) to 00:01:42.500 and save high-quality JPEG
ffmpeg -ss 00:01:42.5 -i input.mp4 -frames:v 1 -q:v 2 frame_102s.jpg
```

## 2. Extract a Short Burst Around a Fast Event (e.g., Crash or Impact)
```bash
# 2 frames per second from 00:01:40 for 6 seconds (12 frames total)
ffmpeg -ss 00:01:40 -t 6 -i input.mp4 -vf fps=2 -q:v 2 burst_%02d.jpg
```

## 3. Build a 3×2 Contact Sheet for a 12-Second Window
Useful when inspecting a short visual sequence in a single image turn:
```bash
ffmpeg -ss 00:01:40 -t 12 -i input.mp4 \
  -vf "fps=0.5,scale=640:-1,tile=3x2" -frames:v 1 -q:v 3 contact_sheet.jpg
```

## 4. Keep Images Under the 8 MB Engine Limit
If passing a frame to `run_engine` or `send_camera_frame` (max 8 MB JPEG/PNG):
```bash
ffmpeg -ss 00:01:42 -i input.mp4 -frames:v 1 -vf "scale='min(1920,iw)':-2" -q:v 3 frame_1080p.jpg
```

## 5. Probe Video Duration & Codec Before Uploading
Passing accurate `duration_sec` and `file_size_bytes` avoids billing/upload surprises:
```bash
ffprobe -v error -show_entries format=duration,size -of default=noprint_wrappers=1 input.mp4
```
