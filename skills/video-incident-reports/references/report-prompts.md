# Structured Report Prompts & Starter Search Queries

## Table of Contents
- [1. Security Incident Report](#1-security-incident-report)
- [2. Daily CCTV Review](#2-daily-cctv-review)
- [3. Workplace Safety & PPE Observation Report](#3-workplace-safety--ppe-observation-report)
- [4. Warehouse & Dock End-of-Shift Handover Report](#4-warehouse--dock-end-of-shift-handover-report)
- [5. Construction Daily Report (Cameras & Drones)](#5-construction-daily-report-cameras--drones)

## 1. Security Incident Report
- **Starter searches**: `"person entering through the rear door"`, `"someone carrying a large box out of the building"`, `"vehicle stopping at the loading dock"`, `"person in a dark hoodie near the gate"`.
- **Analyze prompt**:
```text
You are preparing an incident report for a security investigation. Using the footage in this index, write:

1. Summary: three or four sentences describing what happened, where, and when.
2. Timeline: a table with columns Timestamp (HH:MM:SS), Camera or clip, and Observed action, in chronological order.
3. People and vehicles: describe each distinct person or vehicle by clothing, color, and direction of travel. Do not guess identities.
4. Evidence clips: the moments an investigator should review first, with timestamps and one line on why each matters.
5. Open questions: what the footage does not show or cannot confirm.

Only describe what is visible or audible. Mark anything uncertain as "unconfirmed".
```

## 2. Daily CCTV Review
- **Starter searches**: `"person on site after hours"`, `"door propped open with nobody around"`, `"vehicle parked in the fire lane"`, `"people loitering near the entrance"`.
- **Analyze prompt**:
```text
You are a security operator writing the daily CCTV review from the footage in this index. Write:

1. Summary: three sentences on the period covered. Was it quiet, busy, or unusual?
2. Notable events: a table with columns Timestamp, Camera, and Event, in chronological order. Include arrivals, departures, deliveries, and anything that stands out.
3. After-hours activity: people or vehicles present outside normal operating hours, with timestamps.
4. Anomalies and conditions: doors or gates left open, blocked exits, unattended items, lights or cameras that look obstructed or dark.
5. To investigate: up to five clips a person should watch, ranked by urgency, each with a one-line reason.

Describe people by clothing and behaviour, never by identity. Mark anything uncertain as "unconfirmed".
```

## 3. Workplace Safety & PPE Observation Report
- **Starter searches**: `"worker without a hard hat"`, `"person not wearing a high-visibility vest"`, `"forklift moving close to pedestrians"`, `"someone inside the marked restricted zone"`.
- **Analyze prompt**:
```text
You are a safety officer writing a safety observation report from shift footage. Write:

1. Observation summary: the area covered, the time window, and the overall safety picture in three sentences.
2. Observations: a table with columns Timestamp, Area or camera, Category (safe act, at-risk act, rule violation, unsafe condition), Detail (for PPE name the item: hard hat, high-visibility vest, gloves, eye protection, safety boots, harness), and Severity (low, medium, high).
3. Counts by category and by PPE item.
4. Safe acts worth recognising.
5. Corrective actions, most urgent first, each with the timestamp that shows the issue.

Only report what is clearly visible. If PPE cannot be seen because of angle or distance, record it as "not visible", never as a violation.
```

## 4. Warehouse & Dock End-of-Shift Handover Report
- **Starter searches**: `"truck backing into a dock door"`, `"pallet dropped or damaged"`, `"forklift idle for a long time"`, `"trailer door left open with nobody around"`.
- **Analyze prompt**:
```text
You are a warehouse supervisor writing the end-of-shift handover report from dock and yard camera footage. Write:

1. Shift summary: three sentences on volume, flow, and anything unusual.
2. Arrivals and departures: a table with columns Time, Dock door or area, Vehicle description (type, color, trailer markings if visible), and Arrival or departure.
3. Handling events: loading and unloading activity by dock door, with timestamps.
4. Exceptions: damaged or dropped goods, unsafe handling, forklifts or trailers idle for a long time, doors left open.
5. Handover notes: what the next shift should check or finish.

Only include what is visible in the footage, and give a timestamp for every row.
```

## 5. Construction Daily Report (Cameras & Drones)
- **Starter searches**: `"concrete truck arriving at the site"`, `"crane lifting steel beams"`, `"crew working on the roof"`, `"excavator digging trench"`.
- **Analyze prompt**:
```text
You are a construction superintendent writing the daily jobsite report from site cameras and/or drone footage. Write:

1. Day summary: what moved forward today, plus visible weather and site conditions.
2. Work performed: activity grouped by area or trade, with timestamps.
3. Crews and equipment: approximate visible headcount and equipment seen working or idle.
4. Deliveries: trucks and materials arriving at the gate or laydown area, with timestamps.
5. Safety and delays: visible safety observations, stoppages, and open follow-up items.

Only describe what is visible in the footage. Mark anything uncertain as "unconfirmed".
```
