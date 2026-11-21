# Video Intelligence REST API v1 Reference

## Table of Contents
- [Base URL, Auth & Rate Limits](#base-url-auth--rate-limits)
- [List & Create Indexes](#list--create-indexes)
- [Ingest Video by HTTPS URL](#ingest-video-by-https-url)
- [Poll Video Status](#poll-video-status)
- [Semantic Search](#semantic-search)
- [Multimodal Analysis](#multimodal-analysis)
- [Node.js / TypeScript Pipeline Recipe](#nodejs--typescript-pipeline-recipe)

## Base URL, Auth & Rate Limits
- **Base URL**: `https://www.ceptory.com/api/v1`
- **Auth header**: `Authorization: Bearer <API_KEY>` (create keys in Dashboard → API Keys; **no client SDKs** — call HTTP directly).
- **Scopes**: `READ` (list indexes, poll video, search), `WRITE` (create index, add video, analyze), `ADMIN` (all).
- **Rate limit**: `120 requests / minute` per key (`429` includes `Retry-After` and `X-RateLimit-Reset`).
- **Billing**: `$0.042/min` indexing + `$0.0015/min` embedding infra; `$4/1,000` search queries (`402` if prepaid balance is insufficient).

## List & Create Indexes
- `GET /api/v1/indexes` (`READ`) → `{ indexes: [{ id, name, description, status, videoCount, createdAt, updatedAt }] }`
- `POST /api/v1/indexes` (`WRITE`) → `201 Created` (`409` if duplicate name in workspace)
  ```json
  { "name": "North Gate Footage", "description": "Daily NVR exports" }
  ```
  Limits: `name` ≤ 80 chars, `description` ≤ 500 chars.

## Ingest Video by HTTPS URL
- `POST /api/v1/indexes/{indexId}/videos` (`WRITE`) → `201 Created`
  ```json
  {
    "url": "https://cdn.example.com/gate-2026-11-20.mp4",
    "fileName": "gate-2026-11-20.mp4",
    "durationSec": 300
  }
  ```
  - `url` must start with `https://`.
  - Indexed in 2-second chunks (1s overlap, 6 fps) with audio speech transcription enabled.
  - Returns `{ video: { id, title, status, indexStatus, sourceUrl } }`.

## Poll Video Status
- `GET /api/v1/indexes/{indexId}/videos/{videoId}` (`READ`)
  - Poll until `video.status === "READY"` (or `"FAILED"`).

## Semantic Search
- `POST /api/v1/indexes/{indexId}/search` (`READ`, billed as 1 search query)
  ```json
  { "query": "white delivery van reversing toward dock door 3" }
  ```
  - Returns top **6** reranked results:
    ```json
    {
      "results": [
        {
          "videoId": "vid_...",
          "videoTitle": "gate-2026-11-20",
          "startSec": 112,
          "endSec": 118,
          "score": 0.87,
          "snippet": "A white cargo van backs slowly toward bay 3..."
        }
      ]
    }
    ```

## Multimodal Analysis
- `POST /api/v1/indexes/{indexId}/analyze` (`WRITE`)
  ```json
  { "prompt": "List every vehicle arrival with timestamp, color, and dock door." }
  ```
  - Runs on the most recently `READY` video in the index (max output 4,000 chars).
  - Returns `{ runId, videoId, response }`.

## Node.js / TypeScript Pipeline Recipe
```ts
const BASE = "https://www.ceptory.com/api/v1";
const headers = {
  Authorization: `Bearer ${process.env.CEPTORY_API_KEY}`,
  "Content-Type": "application/json",
};

const { id: indexId } = await fetch(`${BASE}/indexes`, {
  method: "POST",
  headers,
  body: JSON.stringify({ name: "Dock Audit Nov 20" }),
}).then((r) => r.json());

const { video } = await fetch(`${BASE}/indexes/${indexId}/videos`, {
  method: "POST",
  headers,
  body: JSON.stringify({
    url: "https://storage.example.com/dock-shift.mp4",
    fileName: "dock-shift.mp4",
    durationSec: 600,
  }),
}).then((r) => r.json());

// Poll until READY
let status = video.status;
while (status === "PROCESSING") {
  await new Promise((r) => setTimeout(r, 5000));
  const poll = await fetch(`${BASE}/indexes/${indexId}/videos/${video.id}`, { headers }).then((r) => r.json());
  status = poll.video.status;
}

const search = await fetch(`${BASE}/indexes/${indexId}/search`, {
  method: "POST",
  headers,
  body: JSON.stringify({ query: "forklift carrying double-stacked pallets near aisle 2" }),
}).then((r) => r.json());
```
