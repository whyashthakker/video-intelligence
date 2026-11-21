# Example: When the Answer Is in the Audio

Storyboards and frames are silent. If `get_video_info` returns `hasAudio: true` and the question is about what was **said**, use the indexed tools for speech and frames for what was **shown**.

## User prompt
> "Here's the 40-minute product demo recording. What pricing did the presenter quote, and which slide was on screen at the time?"

## 1. Plan
`get_video_info` returns `durationSec: 2412` and `hasAudio: true`, with `indexStatus: "RUNNING"`. The price is spoken, so this question needs the index. Poll `get_chat_videos` until the status is `READY`.

## 2. Find what was said
```json
{ "tool": "search_videos", "args": { "chat_id": "…", "video_id": "vid_demo", "query": "presenter says the price per month or per seat" } }
```
The top hit is `1504–1512 s`. Then:
```json
{ "tool": "ask_video", "args": { "video_id": "vid_demo", "question": "What prices or plan names does the presenter say out loud, and at what times?" } }
```
The answer is "Team plan, $29 per seat per month, annual billing", with clip `1503–1514 s`.

## 3. See what was shown at that moment
```json
{ "tool": "get_video_frames", "args": { "video_id": "vid_demo", "times": [1506, 1511], "width": 1024 } }
```
The slide reads "Pricing: Starter free · Team $29/seat/mo · Enterprise contact us".

## Answer
> - **25:06**: the presenter says the Team plan is "$29 per seat per month, billed annually" (spoken, from `ask_video` clip 25:03–25:14).
> - The slide on screen at **25:06–25:11** reads: *Starter free · Team $29/seat/mo · Enterprise contact us*.
> - The spoken and on-screen prices match.

Label which facts came from speech and which came from frames. The quote's exact wording comes from the transcript-backed answer, so if `ask_video` paraphrased, say it is a paraphrase.
