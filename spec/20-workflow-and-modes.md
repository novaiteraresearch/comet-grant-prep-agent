# spec/20-workflow-and-modes.md — Workflow and Operational Modes

<!--
artifact: spec/20-workflow-and-modes.md
version: 1.0.0
status: stable
tags: [spec, workflow, modes, step-by-step]
-->

## Overview

The agent operates in three primary modes depending on the user's input and the state of the funding opportunity. All modes share the same global rules and stop conditions.

---

## Mode 1 — Funding Opportunity Page Analysis

**Trigger:** User provides a funding opportunity URL (FOA, BAA, RFP, RFI).

**Inputs:**
- Funding opportunity URL
- Optionally: user's DMP (for contextual matching)

**Steps:**

```
Step 1.1  Navigate to the provided URL.
Step 1.2  Extract structured data:
          - Program title and number
          - Sponsoring organization
          - Eligibility criteria (PI, institution, citizenship, etc.)
          - Submission deadlines (LOI, full proposal, registration)
          - Required proposal components (abstract, narrative, budget, etc.)
          - Page-level form fields (if online submission portal)
          - Attached RFP documents or linked FOA PDFs
Step 1.3  STOP. Present structured summary to user.
Step 1.4  Await user approval or modification before proceeding.
```

**Outputs:**
```yaml
program_title:
program_number:
organization:
eligibility:
  pi:
  institution:
  other:
deadlines:
  loi:
  full_proposal:
  registration:
required_components:
  - name:
    description:
    page_limit:
portal_fields:
  - label:
    type:
    required:
    selector:
notes:
```

**Guardrails:**
- Agent does not navigate away from the provided URL without user instruction.
- PDFs are read if accessible; if not, agent notes the link and requests user to provide extracted text.
- No DMP content is referenced in this mode unless the user explicitly requests matching.

---

## Mode 2 — Portal Form Draft and Preview Fill

**Trigger:** User approves Mode 1 output and provides their DMP for field matching.

**Inputs:**
- Mode 1 structured summary (approved by user)
- User's DMP content

**Steps:**

```
Step 2.1  Map each required field to a DMP section.
Step 2.2  For each field:
          a. Identify the DOM selector.
          b. Draft the proposed value from DMP content.
          c. Note the source DMP section.
          d. Flag any field with no DMP match as MISSING.
Step 2.3  STOP. Present full field-by-field draft table to user:
          | Field Label | Selector | Proposed Value | DMP Source | Status |
Step 2.4  Await user approval or corrections.
Step 2.5  For each approved field:
          a. Navigate to the form field.
          b. Insert the approved value.
          c. Log the action.
Step 2.6  STOP. Hard stop. Do not proceed to any submission step.
```

**Outputs (per field):**
```json
{
  "field_label": "Project Title",
  "selector": "input#project-title",
  "proposed_value": "AI Safety Evaluation Framework for Agentic Systems",
  "dmp_source": "DMP Section 1.2 — Project Overview",
  "status": "approved | missing | flagged"
}
```

**Guardrails:**
- Fields marked MISSING are surfaced explicitly; agent does not fabricate values.
- Agent never clicks Save Draft, Next, Continue, or Submit without explicit user instruction per field.
- If a form field is ambiguous (multiple elements match), agent presents both options and awaits user selection.

---

## Mode 3 — Delta / Update Pass

**Trigger:** User returns to a previously analyzed FOA to check for updates, or a prior draft session needs to be reconciled with a changed portal.

**Inputs:**
- Previous Mode 1 structured summary
- Current FOA URL
- Optionally: updated DMP

**Steps:**

```
Step 3.1  Navigate to the FOA URL.
Step 3.2  Re-extract the structured data.
Step 3.3  Diff against the prior summary:
          - New fields added
          - Fields removed
          - Deadline changes
          - Eligibility changes
Step 3.4  STOP. Present diff to user as a structured change report.
Step 3.5  Await user instruction to re-run Mode 2 if needed.
```

**Outputs:**
```yaml
changes_detected:
  - field: submission_deadline
    old_value: 2026-03-15
    new_value: 2026-04-01
    severity: high
  - field: page_limit_narrative
    old_value: 15
    new_value: 12
    severity: medium
no_changes:
  - program_title
  - eligibility.pi
```

**Guardrails:**
- If the page has changed substantially (structure, URL, or content), agent flags this and requests user confirmation before proceeding.
- Diff is presented as a human-readable table; no automated re-drafting without user approval.

---

## Mode Selection Logic

```
IF user provides URL only          -> Mode 1
IF user provides URL + DMP         -> Mode 1, then Mode 2 on approval
IF user references prior session   -> Mode 3, then Mode 2 on approval
```

---

*See also: [10-system-prompt.md](10-system-prompt.md) | [30-governance-and-constraints.md](30-governance-and-constraints.md)*
