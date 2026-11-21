# Video Intelligence Skills for Ceptory

A collection of **10 agent skills** for [Ceptory](https://www.ceptory.com) — the enterprise video intelligence platform for live camera workflows, natural-language video search, cross-camera tracking, AI detection engines, edge CCTV ingestion, and automated operational reporting.

Built to work with any [Agent Skills](https://agentskills.io)-compatible coding or operations agent (Claude Code, Cursor, Codex, Gemini CLI, VS Code, Windsurf, Antigravity, and custom MCP agents).

## Included Skills

| # | Skill | What It Covers |
|---|---|---|
| 1 | [`ceptory-mcp-setup`](skills/ceptory-mcp-setup/SKILL.md) | Connect Claude, Cursor, VS Code, Codex, Gemini CLI, Windsurf, or custom agents to Ceptory's MCP server (`https://mcp.ceptory.com/mcp` or `npx -y ceptory-mcp`), verify workspace permissions and balance, and route requests across all MCP tools. |
| 2 | [`ceptory-video-search`](skills/ceptory-video-search/SKILL.md) | Index recorded videos via URL or local upload (up to 2 GB), search scenes/speech/on-screen text in plain English, run RAG video Q&A (`ask_video`), and automate indexing/search via REST API v1 (`/api/v1/indexes`). |
| 3 | [`ceptory-camera-workflows`](skills/ceptory-camera-workflows/SKILL.md) | Translate plain-English monitoring requests into Ceptory's 10 live camera workflows (`ppe`, `fall`, `hazards`, `intrusion`, `unauthorized_access`, `loitering`, `abandoned_object`, `crowd`, `illegal_parking`, `traffic_helmet`) with IANA schedules, ROI zones, and alert responses. |
| 4 | [`ceptory-alert-triage`](skills/ceptory-alert-triage/SKILL.md) | Inspect fleet health, paginate open alerts, visually verify annotated alert images before resolving false alarms, and configure Slack, Microsoft Teams, Discord, Email, and HMAC-SHA256-signed custom webhooks. |
| 5 | [`ceptory-access-and-plates`](skills/ceptory-access-and-plates/SKILL.md) | Manage face access lists (`staff`, `watchlist`, custom `allow:` / `alert:` lists) on `unauthorized_access` cameras and ANPR license plate lists (`Authorized`, `Residents`, `Banned`, `read_plates`) on `illegal_parking` cameras. |
| 6 | [`ceptory-cross-camera-tracking`](skills/ceptory-cross-camera-tracking/SKILL.md) | Group cameras by site, enable cross-camera person and vehicle Re-ID journeys (`get_movement`, `get_sighting_image`), geocode sites, and configure coordinates/heading/FOV for automatic multi-camera GPS triangulation. |
| 7 | [`ceptory-ai-engines`](skills/ceptory-ai-engines/SKILL.md) | Discover and invoke Ceptory's AI detection engine catalog directly on single images or sequential frames (`list_engines`, `run_engine`, `/api/v1/engines`), including stateful multi-frame tracking loops and annotated JPEG outputs. |
| 8 | [`ceptory-edge-cctv-setup`](skills/ceptory-edge-cctv-setup/SKILL.md) | Connect on-prem IP cameras, NVRs, and DVRs via Ceptory Edge (`edge.toml`, outbound HTTPS only, motion gating) with RTSP sub-stream templates for 15+ brands (Hikvision, Dahua, CP Plus, Axis, Reolink, Uniview, UniFi Protect, ONVIF, etc.) and `/api/v1/edge`. |
| 9 | [`ceptory-incident-and-ops-reports`](skills/ceptory-incident-and-ops-reports/SKILL.md) | Generate evidence-backed Security Incident Reports, Daily CCTV Reviews, Workplace Safety / PPE Observation Reports, Warehouse End-of-Shift Handovers, and Construction Daily Reports from indexed footage. |
| 10 | [`ceptory-media-logging`](skills/ceptory-media-logging/SKILL.md) | Build `HH:MM:SS` timecoded shot logs, extract verbatim spoken pull quotes, rank top moments, generate 60-second highlight cut orders, and tag media/broadcast archives without scrubbing. |

## Installation

### Install all skills with the Skills CLI

```bash
npx skills add whyashthakker/video-intelligence
```

### Install a single skill

```bash
npx skills add whyashthakker/video-intelligence --skill ceptory-camera-workflows
```

### Manual installation

Copy any folder from `skills/` into your agent's skills directory (for example `.agents/skills/` or `.claude/skills/`).

## Connecting Your Agent to Ceptory

Most of these skills pair directly with Ceptory's live MCP server:

- **Streamable HTTP URL**: `https://mcp.ceptory.com/mcp` (OAuth 2.1 browser sign-in, or `Authorization: Bearer <token>`)
- **Stdio bridge**: `CEPTORY_TOKEN=<token> npx -y ceptory-mcp`
- **REST API v1**: `https://www.ceptory.com/api/v1`
- **Docs**: [ceptory.com/mcp](https://www.ceptory.com/mcp) · [ceptory.com/mcp/llms.txt](https://www.ceptory.com/mcp/llms.txt) · [ceptory.com/en/video-intelligence-api](https://www.ceptory.com/en/video-intelligence-api)

## License

MIT
