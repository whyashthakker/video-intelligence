---
name: ceptory-incident-and-ops-reports
description: Generate structured, evidence-backed operational reports from indexed CCTV, workplace, dock, and drone video footage in Ceptory: Security Incident Reports, Daily CCTV Reviews, Workplace Safety / PPE Observation Reports, Warehouse End-of-Shift Handover Reports, and Construction Daily Reports. Use whenever a user asks to write an incident report, daily security review, safety/PPE audit report, warehouse shift handover, or construction progress log from video footage in Ceptory.
---

# Ceptory Structured Security, Safety & Operations Reports

Convert indexed CCTV, warehouse, plant, or drone footage into standardized, review-ready operational reports using Ceptory's search + analyze pipeline.

## End-to-End Execution Pipeline

1. **Scope the footage**: Keep one index / video chat per site or incident window. Preserve camera names and start times in file names (e.g., `cam-rear-door-2026-11-20-2130.mp4`). For incidents, request ~30 minutes before and after the event across every camera covering the location and exit routes.
2. **Confirm indexing status**: Poll `get_chat_videos` (or `GET /api/v1/indexes/{indexId}/videos/{videoId}`) until `READY`.
3. **Run targeted `search_videos` queries first**: Search entry, event, exit, hazards, or vehicles individually to collect exact candidate timestamps before drafting.
4. **Run `analyze_video` (or `POST /api/v1/indexes/{indexId}/analyze`)** with the matching structured prompt below.
   - Note: `analyze_video` runs on one `READY` video at a time (the latest ready video unless `video_id` is specified). For multi-camera incidents, run per relevant video and merge into one chronological timeline, noting any camera clock offsets once at the top.
5. **Cross-check before delivery**: Verify every cited timestamp and mark anything unclear as `"unconfirmed"` or `"not visible"`. Always remind the user that AI drafts should be confirmed by a human reviewer before filing.

## Evidentiary Integrity Rules (Mandatory)

- **Only report what is visible or audible** in the footage.
- **Never guess identities or names**: Describe people by clothing, color, carried items, and direction of travel.
- **Never invent compliance certifications**: Document what cameras saw; do not claim OSHA/HSE or legal compliance certification.
- **Separate "not visible" from "not worn"**: If angle, distance, or occlusion hides PPE (gloves, eye protection, harness), record `"not visible"` — never log it as a violation.
- **State timezone once** at the top of the report and keep timestamps in `HH:MM:SS`.

---

## Report Templates & Starter Search Queries

### 1. Security Incident Report
- **Starter searches**: `"person entering through the rear door"`, `"someone carrying a large box out of the building"`, `"vehicle stopping at the loading dock"`, `"person in a dark hoodie near the gate"`.
- **Analyze prompt**:
  > You are preparing an incident report for a security investigation. Using the footage in this index, write:
  > 1. **Summary**: three or four sentences describing what happened, where, and when.
  > 2. **Timeline**: a table with columns `Timestamp (HH:MM:SS)`, `Camera or clip`, and `Observed action`, in chronological order.
  > 3. **People and vehicles**: describe each distinct person or vehicle by clothing, color, and direction of travel. Do not guess identities.
  > 4. **Evidence clips**: the moments an investigator should review first, with timestamps and one line on why each matters.
  > 5. **Open questions**: what the footage does not show or cannot confirm.
  >
  > Only describe what is visible or audible. Mark anything uncertain as "unconfirmed".

### 2. Daily CCTV Review
- **Starter searches**: `"person on site after hours"`, `"door propped open with nobody around"`, `"vehicle parked in the fire lane"`, `"people loitering near the entrance"`.
- **Analyze prompt**:
  > You are a security operator writing the daily CCTV review from the footage in this index. Write:
  > 1. **Summary**: three sentences on the period covered. Was it quiet, busy, or unusual?
  > 2. **Notable events**: a table with columns `Timestamp`, `Camera`, and `Event`, in chronological order. Include arrivals, departures, deliveries, and anything that stands out.
  > 3. **After-hours activity**: people or vehicles present outside normal operating hours, with timestamps.
  > 4. **Anomalies and conditions**: doors or gates left open, blocked exits, unattended items, lights or cameras that look obstructed or dark.
  > 5. **To investigate**: up to five clips a person should watch, ranked by urgency, each with a one-line reason.
  >
  > Describe people by clothing and behaviour, never by identity. Mark anything uncertain as "unconfirmed".

### 3. Workplace Safety & PPE Observation Report
- **Starter searches**: `"worker without a hard hat"`, `"person not wearing a high-visibility vest"`, `"forklift moving close to pedestrians"`, `"someone inside the marked restricted zone"`.
- **Analyze prompt**:
  > You are a safety officer writing a safety observation report from shift footage. Write:
  > 1. **Observation summary**: the area covered, the time window, and the overall safety picture in three sentences.
  > 2. **Observations**: a table with columns `Timestamp`, `Area or camera`, `Category` (`safe act`, `at-risk act`, `rule violation`, `unsafe condition`), `Detail` (for PPE name the item: hard hat, high-visibility vest, gloves, eye protection, safety boots, harness), and `Severity` (`low`, `medium`, `high`).
  > 3. **Counts by category and by PPE item**.
  > 4. **Safe acts worth recognising** (for the next toolbox talk).
  > 5. **Corrective actions**, most urgent first, each with the timestamp that shows the issue.
  >
  > Only report what is clearly visible. If PPE cannot be seen because of angle or distance, record it as "not visible", never as a violation.

### 4. Warehouse & Dock End-of-Shift Handover Report
- **Starter searches**: `"truck backing into a dock door"`, `"pallet dropped or damaged"`, `"forklift idle for a long time"`, `"trailer door left open with nobody around"`.
- **Analyze prompt**:
  > You are a warehouse supervisor writing the end-of-shift handover report from dock and yard camera footage. Write:
  > 1. **Shift summary**: three sentences on volume, flow, and anything unusual.
  > 2. **Arrivals and departures**: a table with columns `Time`, `Dock door or area`, `Vehicle description` (type, color, visible trailer markings only), and `Arrival or departure`.
  > 3. **Handling events**: loading and unloading activity by dock door, with timestamps.
  > 4. **Exceptions**: damaged or dropped goods, unsafe handling, forklifts or trailers idle for a long time, doors left open.
  > 5. **Handover notes**: what the next shift should check or finish.
  >
  > Only include what is visible in the footage, and give a timestamp for every row.

### 5. Construction Daily Report (Fixed Cameras & Drone Flights)
- **Starter searches**: `"concrete truck arriving at the site"`, `"crane lifting steel beams"`, `"crew working on the roof"`, `"excavator digging trench"`.
- **Analyze prompt**:
  > You are a construction superintendent writing the daily jobsite report from site cameras and/or drone footage. Write:
  > 1. **Day summary**: what moved forward today, plus visible weather and site conditions.
  > 2. **Work performed**: activity grouped by area or trade, with timestamps.
  > 3. **Crews and equipment**: approximate visible headcount and equipment seen working or idle.
  > 4. **Deliveries**: trucks and materials arriving at the gate or laydown area, with timestamps.
  > 5. **Safety and delays**: visible safety observations, stoppages, and open follow-up items.
  >
  > Only describe what is visible in the footage. Mark anything uncertain as "unconfirmed".
