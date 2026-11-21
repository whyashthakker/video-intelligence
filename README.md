<div align="center">

# Video Intelligence Agent Skills

**Give Claude Code, Cursor, Codex, Gemini CLI, VS Code & any AI agent eyes on video and live CCTV.**

[![Skills](https://img.shields.io/badge/Agent%20Skills-11%20Included-111113?style=for-the-badge)](#all-11-video-intelligence-skills)
[![License: MIT](https://img.shields.io/badge/License-MIT-f3f1ec?style=for-the-badge)](LICENSE)

[**Showcase Page**](https://whyashthakker.github.io/video-intelligence/) · [**Skills**](#all-11-video-intelligence-skills) · [**Quickstart**](#2-minute-quickstart)

</div>

---

## What Is This Repository?

An open-source collection of **11 [Agent Skills](https://agentskills.io)** that teach AI agents how to work with video: watch recordings, search footage, run live camera rules, triage alerts, and write reports. The skills drive a video intelligence MCP server, so the agent gets real tools and the skills tell it how to use them well.

Install these skills so your AI agent can:
- **Actually watch & inspect video**: skim a whole recording as one storyboard image, zoom into the moments that matter and read exact frames with the model's own vision, with no indexing wait. Search scenes, speech and on-screen text when the video is long.
- **Connect a video MCP server in 2 minutes**: OAuth 2.1 browser sign-in or a stdio local-file bridge.
- **Configure 10 live camera workflows**: PPE, person down, fire & hazards, intrusion, face access, loitering, unattended bags, crowd limits, illegal parking, and rider helmets.
- **Triage alerts & route notifications**: inspect annotated alert photos before resolving false alarms, and verify Slack, Teams, Discord, Email, and HMAC-signed webhooks.
- **Trace movement & connect on-prem CCTV**: cross-camera Re-ID journeys, multi-camera GPS triangulation, a direct vision engines API, and RTSP/ONVIF setup for 15+ camera brands.
- **Generate structured operational & media reports**: Security Incident Reports, Daily CCTV Reviews, EHS Safety/PPE audits, Warehouse Shift Handovers, Construction Daily Reports, and `HH:MM:SS` media shot logs.

---

## 2-Minute Quickstart

### 1. Install all 11 skills

```bash
npx skills add whyashthakker/video-intelligence
```

*(Or install a single skill: `npx skills add whyashthakker/video-intelligence --skill watch-video`)*

### 2. Connect a video intelligence MCP server

```bash
# Claude Code (OAuth 2.1: approve in the browser, no key to copy)
claude mcp add --transport http video-intel https://mcp.ceptory.com/mcp
```

Works with **Claude Code**, **Claude Desktop**, **Cursor**, **VS Code**, **Codex**, **Gemini CLI**, **Windsurf**, **Antigravity**, **OpenAI Agents SDK**, **LangChain**, and **n8n**. Setup snippets for every client are in the [`video-mcp-setup`](skills/video-mcp-setup/SKILL.md) skill.

### 3. Ask your agent in plain language

> *"Watch `./checkout-bug.mp4`, give me a timestamped timeline of every user action and visible error state, and pull the exact frame where the crash happens."*

> *"Set up a camera called Loading Dock that alerts our Discord when anyone isn't wearing a helmet and hi-vis vest."*

> *"Which cameras are offline right now? Show me today's open alerts with their photos, then resolve confirmed false alarms."*

> *"Generate `edge.toml` for 4 Hikvision NVR channels with motion gating and clock ignore regions."*

---

## All 11 Video Intelligence Skills

| # | Skill | Purpose | Docs |
|---|---|---|---|
| 1 | [`video-mcp-setup`](skills/video-mcp-setup/SKILL.md) | Connect Claude Code, Claude Desktop, Cursor, VS Code, Codex, Gemini CLI, Windsurf, or custom frameworks to the video MCP server over OAuth or a stdio bridge, verify balance/RBAC, and troubleshoot auth. | [ceptory.com/mcp](https://www.ceptory.com/mcp) |
| 2 | [`watch-video`](skills/watch-video/SKILL.md) | Let any LLM watch video with its own vision: skim a storyboard of timestamped frames (`get_video_storyboard`), zoom in, and read exact frames (`get_video_frames`), with no indexing wait. Falls back to search (`search_videos`), the spoken timeline (`get_video`, `ask_video`) or local ffmpeg. | [ceptory.com](https://www.ceptory.com) |
| 3 | [`video-search`](skills/video-search/SKILL.md) | Natural-language video indexing, multimodal search across scenes/speech/OCR, RAG Q&A, and programmatic REST API v1 pipelines (`/api/v1/indexes`). | [Video Search](https://www.ceptory.com/en/features/search) |
| 4 | [`cctv-workflows`](skills/cctv-workflows/SKILL.md) | Translate plain-English monitoring requests into 10 live camera workflows (`ppe`, `fall`, `hazards`, `intrusion`, `unauthorized_access`, `loitering`, `abandoned_object`, `crowd`, `illegal_parking`, `traffic_helmet`), IANA schedules, and ROI zones. | [Live Monitoring](https://www.ceptory.com/en/features/monitor) |
| 5 | [`alert-triage`](skills/alert-triage/SKILL.md) | Fleet health checks, paginated open-alert triage with visual verification (`get_alert_image`), and Slack, Teams, Discord, Email, and HMAC-SHA256 webhook routing. | [MCP Server](https://www.ceptory.com/mcp) |
| 6 | [`face-and-plate-access`](skills/face-and-plate-access/SKILL.md) | Face access lists (`staff`, `watchlist`, custom `allow:` / `alert:`) on `unauthorized_access` cameras and ANPR plate lists (`Authorized`, `Residents`, `Banned`, `read_plates`) on `illegal_parking` cameras. | [Detection](https://www.ceptory.com/en/features/detect) |
| 7 | [`cross-camera-tracking`](skills/cross-camera-tracking/SKILL.md) | Organize cameras into site groups, trace cross-camera person/vehicle Re-ID journeys (`get_movement`, `get_sighting_image`), geocode addresses, and configure heading/FOV for multi-camera GPS triangulation. | [Tracking](https://www.ceptory.com/en/features/track) |
| 8 | [`vision-engines`](skills/vision-engines/SKILL.md) | Discover and invoke the AI detection engine catalog directly on single images or stateful multi-frame loops (`list_engines`, `run_engine`, `/api/v1/engines/{engineId}`) with annotated JPEG output. | [Video Intelligence API](https://www.ceptory.com/en/video-intelligence-api) |
| 9 | [`rtsp-camera-setup`](skills/rtsp-camera-setup/SKILL.md) | Connect existing IP cameras, NVRs, DVRs, and analog CCTV over outbound HTTPS using an edge agent (`edge.toml`, local motion gating) with RTSP templates for 15+ brands (Hikvision, Dahua, CP Plus, Axis, Reolink, Uniview, UniFi Protect, ONVIF). | [Connect CCTV](https://www.ceptory.com/en/integrations/cameras) |
| 10 | [`video-incident-reports`](skills/video-incident-reports/SKILL.md) | Generate evidence-backed Security Incident Reports, Daily CCTV Reviews, Workplace Safety/PPE Observation Reports, Warehouse Shift Handovers, and Construction Daily Reports from indexed footage. | [Report Guides](https://www.ceptory.com/guides) |
| 11 | [`media-shot-logging`](skills/media-shot-logging/SKILL.md) | Build `HH:MM:SS` timecoded shot logs, extract verbatim spoken pull quotes, rank top moments, build 60-second highlight cut orders, and tag media archives without scrubbing. | [Video Logging Guide](https://www.ceptory.com/guides/log-video-footage-with-ai) |

---

## Skill Structure

Every skill follows the [agentskills.io](https://agentskills.io) progressive-disclosure layout to keep agent context lean:

```text
skills/<skill-name>/
├── SKILL.md           # Lean core workflow, guardrails & conditional triggers (< 50 lines)
├── references/        # Deep API specs, schemas, RTSP brand tables, webhook verification code
└── examples/          # Concrete transcripts, tool-call payloads, edge.toml & sample reports
```

---

## Manual Installation

Copy any skill directory from `skills/` into your agent's skills folder (e.g. `.agents/skills/` or `.claude/skills/`):

```bash
git clone https://github.com/whyashthakker/video-intelligence.git
cp -r video-intelligence/skills/* .agents/skills/
```

---

## Links

- **Showcase page**: [whyashthakker.github.io/video-intelligence](https://whyashthakker.github.io/video-intelligence/)
- **Ceptory MCP Server guide**: [whyashthakker.github.io/video-intelligence/ceptory-mcp](https://whyashthakker.github.io/video-intelligence/ceptory-mcp/)
- **Agent Skills spec**: [agentskills.io](https://agentskills.io)
- **Default MCP server docs**: [ceptory.com/mcp](https://www.ceptory.com/mcp)

## License

MIT — see [LICENSE](LICENSE).
