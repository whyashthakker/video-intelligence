# Extracting Frames with ffmpeg for Pixel-Level Inspection

Use these commands when the footage must stay on the user's machine, when no MCP server is connected, or when you have the file locally and want frames faster than an upload. The hosted `get_video_storyboard` and `get_video_frames` tools do the same thing server-side.

Always seek with `-ss` **before** `-i`. This is a fast keyframe seek followed by an accurate decode, so even a remote URL only reads a few seconds around the timestamp.

## 0. Storyboard: the same grid as `get_video_storyboard`, fully local
Mid-slice sampling (`t_i = S + (i + 0.5) * (E - S) / N`), one fast seek per tile, then `tile`. Set `tile=` to match `N`: 2x2 for 4, 3x2 for 6, 3x3 for 9, 4x2 for 8, 4x4 for 16.
```bash
V=input.mp4; S=0; E=$(ffprobe -v error -show_entries format=duration -of csv=p=0 "$V"); N=16
mkdir -p /tmp/sb && cd /tmp/sb && rm -f ??.jpg          # stale tiles would be picked up by %02d
for i in $(seq 0 $((N-1))); do
  t=$(awk -v i=$i -v s=$S -v e=$E -v n=$N 'BEGIN { printf "%.2f", s + (i + 0.5) * (e - s) / n }')
  ffmpeg -v error -y -ss "$t" -i "$V" -frames:v 1 -vf "scale='min(314,iw)':-2" -q:v 4 "$(printf %02d $i).jpg"
  echo "tile $((i+1))  t=$t"
done
ffmpeg -v error -y -framerate 1 -i %02d.jpg -vf "tile=4x4:padding=4:margin=4:color=0x111111" -frames:v 1 storyboard.jpg
```
Use `awk` for the arithmetic, not `bc`. `bc` prints `.35`, and ffmpeg rejects a duration without a leading zero. Timestamps are not drawn on the tiles, so keep the printed `tile → t` list and cite from it. Most ffmpeg builds lack `drawtext`, so don't rely on burning text in.

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
