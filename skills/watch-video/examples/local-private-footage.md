# Example: Watching Footage That Must Stay on This Machine

The user says the footage cannot be uploaded (an HR investigation, a customer's CCTV), or no MCP server is connected. Run the same storyboard, zoom and frame method locally with ffmpeg, then read the images with your own vision.

## User prompt
> "Don't upload this. Look at `~/Desktop/storeroom-0312.mp4` and tell me when the shelf gets emptied."

## 1. Probe
```bash
ffprobe -v error -show_entries format=duration:stream=width,height,r_frame_rate,codec_type \
  -of default=noprint_wrappers=1 ~/Desktop/storeroom-0312.mp4
```
This returns `duration=1830.4` (30.5 min) and 1280×720.

## 2. Skim with a 16-tile storyboard
Use mid-slice times, `t_i = S + (i + 0.5) * (E - S) / N`, which gives 57.20 s, 171.60 s … 1773.20 s. Grab each frame with a fast seek (`-ss` before `-i`), then tile:
```bash
V=~/Desktop/storeroom-0312.mp4; S=0; E=1830.4; N=16      # range and tile count
mkdir -p /tmp/sb && cd /tmp/sb && rm -f ??.jpg
for i in $(seq 0 $((N-1))); do
  # awk, not bc: bc prints ".35" without the leading zero and ffmpeg rejects it
  t=$(awk -v i=$i -v s=$S -v e=$E -v n=$N 'BEGIN { printf "%.2f", s + (i + 0.5) * (e - s) / n }')
  ffmpeg -v error -y -ss "$t" -i "$V" -frames:v 1 -vf "scale='min(314,iw)':-2" -q:v 4 "$(printf %02d $i).jpg"
  echo "tile $((i+1))  t=$t"
done
ffmpeg -v error -y -framerate 1 -i %02d.jpg -vf "tile=4x4:padding=4:margin=4:color=0x111111" -frames:v 1 storyboard.jpg
```
Read `storyboard.jpg`, then use the printed `tile → t` list for timestamps. Tile 9 (t=972.4) shows a full shelf and tile 10 (t=1086.8) shows it empty.

## 3. Zoom between tiles 9 and 10
Run the same loop with `S=972.4; E=1086.8; N=8` and `tile=4x2`. Tile 3 (1008.2 s) shows a person lifting the boxes. Tile 4 (1022.5 s) shows the shelf half empty.

## 4. Detail frames
```bash
ffmpeg -v error -ss 1008.2 -i ~/Desktop/storeroom-0312.mp4 -frames:v 1 -q:v 2 f_1008.jpg
ffmpeg -v error -ss 1022.5 -i ~/Desktop/storeroom-0312.mp4 -frames:v 1 -q:v 2 f_1022.jpg
```

## Answer
> The shelf is emptied between **16:48** and **17:03** (1008–1023 s). At 16:48 one person in a grey hoodie is carrying boxes from it. No faces were identified. Nothing was uploaded.

Delete `/tmp/sb` afterwards if the footage is sensitive.
