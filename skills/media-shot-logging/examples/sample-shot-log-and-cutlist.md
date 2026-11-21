# Example Output: Timecoded Shot Log & 60-Second Highlight Cutlist

**Source**: `product-keynote-2026-main.mp4` (`42:18`)

## 1. Shot Log
| Timecode In | Timecode Out | Title | Description | Suggested Use |
|---|---|---|---|---|
| `00:02:14` | `00:02:29` | Wide crowd applause opening | Wide crane shot of auditorium standing and clapping as keynote speaker enters. | trailer |
| `00:11:42` | `00:11:57` | Live search demo reveal | Close-up of screen showing instant natural-language match jumping to clip. | social clip |
| `00:18:05` | `00:18:24` | Latency benchmark slide | Speaker points to side-by-side latency chart; audience leans in and photographs slide. | recap |
| `00:29:50` | `00:30:12` | Live camera alert trigger | Stage screen triggers live PPE detection box with audible chime and crowd reaction. | social clip |
| `00:40:10` | `00:40:25` | Closing thank-you ovation | Speaker closes keynote under spotlight; standing applause. | recap |

## 2. Top Five Moments (Ranked)
1. `00:29:50–00:30:12` — Real-time visual demo triggers instant crowd applause (highest energy + clear product proof).
2. `00:11:42–00:11:57` — Clean 15-second self-contained product reveal ideal for social.
3. `00:18:05–00:18:24` — Concrete data visual audiences screenshot and share.
4. `00:02:14–00:02:29` — High-production-value opener for trailer pacing.
5. `00:40:10–00:40:25` — Natural emotional closer.

## 3. Verbatim Pull Quotes
- `00:11:46`: *"You shouldn't spend four hours scrubbing footage to answer a ten-second question."*
- `00:30:04`: *"No new cameras, no firewall holes—just point it at what you already have."*

## 4. 60-Second Highlight Cut Order
1. `00:02:14–00:02:22` (8s) — Opening energy
2. `00:11:42–00:11:57` (15s) — Core problem quote + search reveal
3. `00:29:50–00:30:12` (22s) — Live detection demo + crowd reaction
4. `00:40:10–00:40:25` (15s) — Closing applause + end card

## 5. Optional Rough-Cut Commands (`ffmpeg`)
The server logs and plays back moments; editors cut finals in their NLE or rough-cut clips via `ffmpeg`:
```bash
ffmpeg -ss 00:11:42 -to 00:11:57 -i product-keynote-2026-main.mp4 -c copy clip_02_search_reveal.mp4
ffmpeg -ss 00:29:50 -to 00:30:12 -i product-keynote-2026-main.mp4 -c copy clip_03_live_demo.mp4
```
