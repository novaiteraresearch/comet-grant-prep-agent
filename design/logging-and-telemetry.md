# design/logging-and-telemetry.md — Action Log Format and Sink Specification

<!--
artifact: design/logging-and-telemetry.md
version: 1.0.0
status: stable
tags: [logging, telemetry, action-log, schema, audit]
-->

## Overview

The action log is the primary audit artifact produced by the agent. Every agent action must produce a log entry before or at the time of execution. This document defines the full log format, field semantics, sink options, and retention requirements.

---

## Log Entry Schema

Full schema: [`tooling/schema/action-log.schema.json`](../tooling/schema/action-log.schema.json)

```json
{
  "session_id": "uuid-v4",
  "entry_index": 1,
  "timestamp": "2026-02-22T10:00:00.000Z",
  "action": "navigate",
  "url": "https://www.research.gov/grfp/AwardeeList.do",
  "element": null,
  "extracted_text": null,
  "proposed_value": null,
  "user_approved": null,
  "stop_reason": null,
  "question": null,
  "dmp_source": null,
  "field_label": null,
  "status": null
}
```

---

## Field Definitions

| Field | Type | Required For | Description |
|---|---|---|---|
| `session_id` | string (UUID) | All entries | Unique identifier for the browser session |
| `entry_index` | integer | All entries | Monotonically increasing index within the session |
| `timestamp` | string (ISO 8601) | All entries | UTC timestamp of when the action was taken |
| `action` | enum | All entries | Action type (see below) |
| `url` | string (URI) | All entries | URL of the current page at time of action |
| `element` | string or null | extract, preview_fill | CSS selector or aria-label of the target element |
| `extracted_text` | string or null | extract | Raw text extracted from the page element |
| `proposed_value` | string or null | draft, preview_fill | Value proposed for the field, from DMP |
| `user_approved` | boolean or null | preview_fill | Whether the user explicitly approved this value |
| `stop_reason` | string or null | stop | Human-readable description of why the agent stopped |
| `question` | string or null | ask_user | The clarification question posed to the user |
| `dmp_source` | string or null | draft, preview_fill | DMP section reference (e.g., "Section 1.2") |
| `field_label` | string or null | draft, preview_fill | Human-readable label of the form field |
| `status` | enum or null | draft, preview_fill | "approved", "missing", or "flagged" |

---

## Action Type Definitions

| Action | Description | Required Fields |
|---|---|---|
| `navigate` | Agent navigated to a URL | timestamp, url |
| `extract` | Agent extracted text from a page element | timestamp, url, element, extracted_text |
| `draft` | Agent generated a proposed field value from DMP | timestamp, field_label, proposed_value, dmp_source, status |
| `preview_fill` | Agent inserted an approved value into a form field | timestamp, url, element, proposed_value, user_approved |
| `ask_user` | Agent surfaced a clarification question | timestamp, question |
| `stop` | Agent halted due to a stop condition | timestamp, stop_reason |

---

## Log Sink Options

### Option A — In-Session JSON Array (Minimum)

- Log entries are accumulated in memory during the session.
- At session end, the full array is returned to the user for export.
- **Limitation:** Log is lost if the session crashes.

### Option B — File Sink (Recommended)

- Each log entry is appended to a newline-delimited JSON file (`session-{id}.jsonl`) in real time.
- File is closed at session end.
- **Supports:** Real-time monitoring, crash recovery, diff-based regression testing.

### Option C — Database Sink (Enterprise)

- Each log entry is written to a structured database (Postgres, Supabase, etc.) in real time.
- Session ID is the partition key.
- **Supports:** Cross-session querying, team-level audit dashboards, automated regression assertions.

---

## Retention and Redaction

- **Retention minimum:** 90 days per session log.
- **Redaction:** Use `tooling/scripts/redact-output.py` to remove PII before sharing logs externally.
- **Redactable fields:** `extracted_text`, `proposed_value`, `question` (may contain DMP content or personal information).
- **Non-redactable fields:** `timestamp`, `action`, `url`, `entry_index`, `stop_reason` (required for audit integrity).

---

## Example Log Sequence

```jsonl
{"session_id": "a1b2c3d4", "entry_index": 1, "timestamp": "2026-02-22T18:00:01Z", "action": "navigate", "url": "https://www.research.gov/grfp/AwardeeList.do", "element": null, "extracted_text": null, "proposed_value": null, "user_approved": null, "stop_reason": null}
{"session_id": "a1b2c3d4", "entry_index": 2, "timestamp": "2026-02-22T18:00:05Z", "action": "extract", "url": "https://www.research.gov/grfp/AwardeeList.do", "element": "h1.program-title", "extracted_text": "CISE Research Initiation Initiative (CRII)", "proposed_value": null, "user_approved": null, "stop_reason": null}
{"session_id": "a1b2c3d4", "entry_index": 3, "timestamp": "2026-02-22T18:00:10Z", "action": "stop", "url": "https://www.research.gov/grfp/AwardeeList.do", "element": null, "extracted_text": null, "proposed_value": null, "user_approved": null, "stop_reason": "gate_1_post_extraction_review"}
```

---

*See also: [`tooling/schema/action-log.schema.json`](../tooling/schema/action-log.schema.json) | [`tooling/scripts/validate-action-log.py`](../tooling/scripts/validate-action-log.py)*
