<div align="center">

# Ceptory Video Intelligence — Agent Skills

**Give Claude Code, Cursor, Codex, Gemini CLI, VS Code & any AI agent eyes on video and live CCTV — powered by [ceptory.com](https://www.ceptory.com).**

[![Website](https://img.shields.io/badge/ceptory.com-Video%20Intelligence-d9485f?style=for-the-badge)](https://www.ceptory.com)
[![MCP Server](https://img.shields.io/badge/MCP-mcp.ceptory.com%2Fmcp-111113?style=for-the-badge)](https://www.ceptory.com/mcp)
[![Skills](https://img.shields.io/badge/Agent%20Skills-11%20Included-111113?style=for-the-badge)](https://www.ceptory.com/skills)
[![License: MIT](https://img.shields.io/badge/License-MIT-f3f1ec?style=for-the-badge)](LICENSE)

[**Try Ceptory Free**](https://www.ceptory.com/signup) · [**Live Showcase Page**](https://whyashthakker.github.io/video-intelligence/) · [**ceptory.com/skills**](https://www.ceptory.com/skills) · [**MCP Docs**](https://www.ceptory.com/mcp) · [**REST API**](https://www.ceptory.com/en/video-intelligence-api) · [**Pricing**](https://www.ceptory.com/pricing)

</div>

---

## What Is This Repository?

This is the official open-source collection of **11 [Agent Skills](https://agentskills.io)** for **[Ceptory](https://www.ceptory.com)** — the enterprise video intelligence platform that turns recorded video and existing CCTV cameras into searchable, chat-able, alert-ready intelligence.

Install these skills so your AI agent can:
- **Actually watch & inspect video** — ingest recordings up to 2 GB, read chronological timelines, search visual scenes + speech + on-screen text, and zoom into exact frames.
- **Connect to Ceptory MCP in 2 minutes** — OAuth 2.1 browser sign-in (`https://mcp.ceptory.com/mcp`) or stdio local-file bridge (`npx -y ceptory-mcp`).
- **Configure all 10 live camera workflows** — PPE, person down, fire & hazards, intrusion, face access, loitering, unattended bags, crowd limits, illegal parking, and rider helmets.
- **Triage alerts & route notifications** — inspect annotated alert photos before resolving false alarms, and verify Slack, Teams, Discord, Email, and HMAC-signed webhooks.
- **Trace movement & connect on-prem CCTV** — cross-camera Re-ID journeys, multi-camera GPS triangulation, direct AI vision engines API, and RTSP/ONVIF setup for 15+ camera brands via Ceptory Edge.
- **Generate structured operational & media reports** — Security Incident Reports, Daily CCTV Reviews, EHS Safety/PPE audits, Warehouse Shift Handovers, Construction Daily Reports, and `HH:MM:SS` Media Shot Logs.

---

## 2-Minute Quickstart

### 1. Install all 11 skills

```bash
npx skills add whyashthakker/video-intelligence
```

*(Or install a single skill: `npx skills add whyashthakker/video-intelligence --skill ceptory-llm-video-vision`)*

### 2. Connect Ceptory's MCP server

```bash
# Claude Code (OAuth 2.1 — opens ceptory.com to approve, no key to copy)
claude mcp add --transport http ceptory https://mcp.ceptory.com/mcp
```

Works with **Claude Code**, **Claude Desktop**, **Cursor**, **VS Code**, **Codex**, **Gemini CLI**, **Windsurf**, **Antigravity**, **OpenAI Agents SDK**, **LangChain**, and **n8n**. Full setup snippets at **[ceptory.com/mcp](https://www.ceptory.com/mcp)**.

### 3. Ask your agent in plain language

> *"Watch `./checkout-bug.mp4`, give me a timestamped timeline of every user action and visible error state, and pull the exact frame where the crash happens."*

> *"Set up a camera called Loading Dock that alerts our Discord when anyone isn't wearing a helmet and hi-vis vest."*

> *"Which cameras are offline right now? Show me today's open alerts with their photos, then resolve confirmed false alarms."*

> *"Generate `edge.toml` for 4 Hikvision NVR channels with motion gating and clock ignore regions."*

---

## All 11 Video Intelligence Skills

| # | Skill | Purpose | Ceptory Docs |
|---|---|---|---|
| 1 | [`ceptory-mcp-setup`](skills/ceptory-mcp-setup/SKILL.md) | Connect Claude Code, Claude Desktop, Cursor, VS Code, Codex, Gemini CLI, Windsurf, or custom frameworks to `https://mcp.ceptory.com/mcp` or `npx -y ceptory-mcp`, verify balance/RBAC, and troubleshoot auth. | [ceptory.com/mcp](https://www.ceptory.com/mcp) |
| 2 | [`ceptory-llm-video-vision`](skills/ceptory-llm-video-vision/SKILL.md) | Give any LLM vision over local or remote video files: upload/link video, read chronological visual+audio timelines (`get_video`), jump to moments (`search_videos`), ask grounded visual questions (`ask_video`), and inspect exact frames. | [ceptory.com](https://www.ceptory.com) |
| 3 | [`ceptory-video-search`](skills/ceptory-video-search/SKILL.md) | Natural-language video indexing, multimodal search across scenes/speech/OCR, RAG Q&A, and programmatic REST API v1 pipelines (`/api/v1/indexes`). | [Video Search](https://www.ceptory.com/en/features/search) |
| 4 | [`ceptory-camera-workflows`](skills/ceptory-camera-workflows/SKILL.md) | Translate plain-English monitoring requests into Ceptory's 10 live camera workflows (`ppe`, `fall`, `hazards`, `intrusion`, `unauthorized_access`, `loitering`, `abandoned_object`, `crowd`, `illegal_parking`, `traffic_helmet`), IANA schedules, and ROI zones. | [Live Monitoring](https://www.ceptory.com/en/features/monitor) |
| 5 | [`ceptory-alert-triage`](skills/ceptory-alert-triage/SKILL.md) | Fleet health checks, paginated open-alert triage with visual verification (`get_alert_image`), and Slack, Teams, Discord, Email, and HMAC-SHA256 webhook routing. | [MCP Server](https://www.ceptory.com/mcp) |
| 6 | [`ceptory-access-and-plates`](skills/ceptory-access-and-plates/SKILL.md) | Face access lists (`staff`, `watchlist`, custom `allow:` / `alert:`) on `unauthorized_access` cameras and ANPR plate lists (`Authorized`, `Residents`, `Banned`, `read_plates`) on `illegal_parking` cameras. | [Detection](https://www.ceptory.com/en/features/detect) |
| 7 | [`ceptory-cross-camera-tracking`](skills/ceptory-cross-camera-tracking/SKILL.md) | Organize cameras into site groups, trace cross-camera person/vehicle Re-ID journeys (`get_movement`, `get_sighting_image`), geocode addresses, and configure heading/FOV for multi-camera GPS triangulation. | [Tracking](https://www.ceptory.com/en/features/track) |
| 8 | [`ceptory-ai-engines`](skills/ceptory-ai-engines/SKILL.md) | Discover and invoke Ceptory's AI detection engine catalog directly on single images or stateful multi-frame loops (`list_engines`, `run_engine`, `/api/v1/engines/{engineId}`) with annotated JPEG output. | [Video Intelligence API](https://www.ceptory.com/en/video-intelligence-api) |
| 9 | [`ceptory-edge-cctv-setup`](skills/ceptory-edge-cctv-setup/SKILL.md) | Connect existing IP cameras, NVRs, DVRs, and analog CCTV over outbound HTTPS using Ceptory Edge (`edge.toml`, local motion gating) with RTSP templates for 15+ brands (Hikvision, Dahua, CP Plus, Axis, Reolink, Uniview, UniFi Protect, ONVIF). | [Connect CCTV](https://www.ceptory.com/en/integrations/cameras) |
| 10 | [`ceptory-incident-and-ops-reports`](skills/ceptory-incident-and-ops-reports/SKILL.md) | Generate evidence-backed Security Incident Reports, Daily CCTV Reviews, Workplace Safety/PPE Observation Reports, Warehouse Shift Handovers, and Construction Daily Reports from indexed footage. | [Report Guides](https://www.ceptory.com/guides) |
| 11 | [`ceptory-media-logging`](skills/ceptory-media-logging/SKILL.md) | Build `HH:MM:SS` timecoded shot logs, extract verbatim spoken pull quotes, rank top moments, build 60-second highlight cut orders, and tag media archives without scrubbing. | [Video Logging Guide](https://www.ceptory.com/guides/log-video-footage-with-ai) |

---

## Why Teams Use [Ceptory](https://www.ceptory.com)

- **Works with existing cameras & footage** — No proprietary cameras or hardware refresh. Upload MP4/MOV/WebM clips, connect RTSP/ONVIF streams via Ceptory Edge, or stream via Ceptory Desktop.
- **Native MCP + REST API** — Built for both human operators and AI agents (`https://mcp.ceptory.com/mcp`, `https://www.ceptory.com/api/v1`, `https://www.ceptory.com/llms.txt`).
- **Transparent, usage-based pricing** — Free workspace indexing allowance to start, then `$0.042/min` video indexing + `$0.0015/min` embedding infra and `$4 / 1,000` search queries ([ceptory.com/pricing](https://www.ceptory.com/pricing)).
- **Enterprise deployment options** — Cloud SaaS, private cloud, or on-prem / hybrid edge deployments ([Book a demo](https://www.ceptory.com/demo)).

---

## Manual Installation

Copy any skill directory from `skills/` into your agent's skills folder (e.g. `.agents/skills/` or `.claude/skills/`):

```bash
git clone https://github.com/whyashthakker/video-intelligence.git
cp -r video-intelligence/skills/* .agents/skills/
```

---

## Links

- **Website**: [https://www.ceptory.com](https://www.ceptory.com)
- **Skills Page**: [https://www.ceptory.com/skills](https://www.ceptory.com/skills)
- **GitHub Pages Showcase**: [https://whyashthakker.github.io/video-intelligence/](https://whyashthakker.github.io/video-intelligence/)
- **MCP Server & Agent Docs**: [https://www.ceptory.com/mcp](https://www.ceptory.com/mcp) · [llms.txt](https://www.ceptory.com/mcp/llms.txt)
- **Video Intelligence API**: [https://www.ceptory.com/en/video-intelligence-api](https://www.ceptory.com/en/video-intelligence-api)
- **Free Signup**: [https://www.ceptory.com/signup](https://www.ceptory.com/signup)
- **Book a Demo**: [https://www.ceptory.com/demo](https://www.ceptory.com/demo)

## License

MIT — see [LICENSE](LICENSE).
