---
name: video-search
description: Index recorded videos from HTTPS URLs or local uploads (up to 2 GB), search across visual scenes, spoken audio, and on-screen text in natural language, run grounded video Q&A, and automate video indexing pipelines via MCP or the REST API (/api/v1/indexes). Use whenever a user wants to upload, index, semantically search, query, or summarize recorded video footage.
license: MIT
metadata:
  version: "1.0"
---

# Natural-Language Video Search & Q&A

Index recorded video and retrieve timestamped moments across scenes, speech, and on-screen text via MCP or the REST API v1.

## Choose Your Interface

- **Interactive agent tasks (MCP)**: Use `create_chat` → `add_video_from_url` (or `create_upload_url` + `PUT` + `complete_upload`) → poll `get_chat_videos` until `READY` → `search_videos` / `ask_video` / `analyze_video` / `get_video`.
- **Scripts & backend pipelines (REST API v1)**: Read [references/rest-api-v1.md](references/rest-api-v1.md) for `/api/v1/indexes`, `/videos`, `/search`, and `/analyze` schemas and Node.js examples (call HTTP directly; no client SDK).

## Core Workflow

1. **Keep one index/chat per site or workflow** so unrelated footage doesn't dilute search ranking. Preserve camera names and start timestamps in `file_name`.
2. **Ingest & poll**: Wait until `status === "READY"`. On `FAILED`, call `retry_video` (not billed again).
3. **Run atomic semantic searches**:
   - Describe **appearance + action** (`"person in dark jacket carrying box near rear door"`), spoken phrases (`"speaker mentions safety stop"`), or visible signage — never guess personal identities.
   - Use `min_score` (`0`–`1`), `from_sec`, `to_sec`, and `video_id` to narrow results.
4. **Synthesize with proof**: Follow top search hits with `ask_video` or `analyze_video`, citing every returned `startSec`–`endSec` range.

## References & Examples

- Read [references/rest-api-v1.md](references/rest-api-v1.md) when writing code or `curl` commands against `https://www.ceptory.com/api/v1`.
- Read [examples/search-and-rag-session.md](examples/search-and-rag-session.md) for strong vs weak query patterns and a multi-clip investigation trace.

## Pricing & Limits

- Metered usage: `$0.042/min` indexing + `$0.0015/min` embedding infra; `$4/1,000` search queries (workspaces include a free starter allowance; check `get_account`).
- Uploads up to **2 GB** (MP4/MOV/WebM); URL ingest requires `https://`; `402` means prepaid balance is depleted.
