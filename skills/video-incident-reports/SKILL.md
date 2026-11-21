---
name: video-incident-reports
description: Generate structured, evidence-backed operational reports from indexed CCTV, workplace, dock, and drone video: Security Incident Reports, Daily CCTV Reviews, Workplace Safety / PPE Observation Reports, Warehouse End-of-Shift Handovers, and Construction Daily Reports via MCP. Use whenever a user asks to write an incident report, daily security review, safety/PPE audit, warehouse handover, or construction log from video footage.
license: MIT
metadata:
  version: "1.0"
---

# Structured Security, Safety & Operations Reports From Video

Turn indexed CCTV, warehouse, plant, or drone footage into standardized, review-ready operational reports using the search + analyze workflow.

## 5-Step Reporting Pipeline

1. **Scope & name clips cleanly**: Keep one chat/index per site or incident window (~30 minutes before and after an incident across all relevant cameras). Preserve camera names and start times in file names.
2. **Confirm `READY` status**: Poll `get_chat_videos` (or `GET /api/v1/indexes/{indexId}/videos/{videoId}`).
3. **Run targeted `search_videos` queries first**: Collect exact candidate timestamps for entry, action, exit, PPE/hazards, or vehicles before drafting.
4. **Generate the structured draft with `analyze_video`**: Read [references/report-prompts.md](references/report-prompts.md) and select the exact prompt for the requested report type:
   - Security Incident Report
   - Daily CCTV Review
   - Workplace Safety & PPE Observation Report
   - Warehouse & Dock End-of-Shift Handover Report
   - Construction Daily Report (fixed cameras & drones)
   *(Because `analyze_video` runs on one `READY` video per call, run per relevant video on multi-camera incidents and merge chronologically, noting any camera clock offset once at the top.)*
5. **Flag for human confirmation**: Remind the user to confirm high-severity rows and timestamps against original camera exports before filing.

## Mandatory Evidentiary Rules

- **Only describe what is visible or audible.** Mark anything unclear as `"unconfirmed"`.
- **Never guess identities or personal names**: Describe people by clothing, color, carried items, and direction of travel.
- **Separate `"not visible"` from `"not worn"`**: If camera angle or distance obscures PPE (gloves, eye protection, harness), mark `"not visible"` — never log a violation.
- **Never claim legal/OSHA/HSE compliance certification**: Document camera observations only.
- **State timezone once** at the top and format all timestamps as `HH:MM:SS`.

## References & Examples

- Read [references/report-prompts.md](references/report-prompts.md) to copy the exact report prompt and starter `search_videos` queries for the requested report type.
- Read [examples/sample-security-incident-report.md](examples/sample-security-incident-report.md) for the expected formatting, timeline table, and evidentiary phrasing.
