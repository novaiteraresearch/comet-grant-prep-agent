<!--
artifact: README.md
repo: novaiteraresearch/comet-grant-prep-agent
version: 1.0.0
author: novaiteraresearch
created: 2026-02-22
license: MIT
status: stable
tags: [metaprompt, browser-agent, grant-prep, governance, ai-safety, human-in-the-loop]
-->

# Comet Grant Prep Agent

> **Review-gated, non-autonomous browser metaprompt for grant preparation and form population.**
> Governance-first. Human-in-the-loop by design.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status: Stable](https://img.shields.io/badge/Status-Stable-green.svg)]()
[![Autonomy: Non-Autonomous](https://img.shields.io/badge/Autonomy-Non--Autonomous-red.svg)](spec/30-governance-and-constraints.md)
[![Runtime: Comet](https://img.shields.io/badge/Runtime-Comet%20Browser%20Agent-blueviolet.svg)]()

---

## Overview

This repository contains a deterministic, review-gated metaprompt designed for use with browser-enabled AI assistants (e.g., Comet) capable of navigating funding opportunity websites. The metaprompt supports the **preparation phase** of grant and sponsorship applications by:

- Extracting required fields from funding opportunity announcements (FOAs) and online grant portals
- Generating draft responses grounded in a user-provided Data Management Plan (DMP)
- Populating form fields **for preview only**
- Logging all actions with timestamps, URLs, and DOM selectors
- Enforcing a hard stop before any submission, confirmation, or transmission step

It is explicitly designed to **avoid autonomous submission or irreversible actions**.

---

## Key Properties

| Property | Value |
|---|---|
| Autonomy level | Non-autonomous |
| Submission behavior | Prohibited (hard constraint) |
| Human review gates | Mandatory before field population and before any form action |
| Data source policy | User-provided DMP only; no hallucination or inference |
| Action logging | Structured JSON per action (timestamp, URL, element, value) |
| Determinism | Same inputs produce same outputs |
| Portability | Multi-domain (NSF, Sloan, Mozilla, CZI, ONR/AFOSR/ARO) |

---

## Repository Structure

```text
comet-grant-prep-agent/
├── README.md                        # This file
├── LICENSE                          # MIT License
├── spec/
│   ├── 00-overview.md               # Problem statement, design goals, non-goals
│   ├── 10-system-prompt.md          # Formal system prompt specification
│   ├── 20-workflow-and-modes.md     # Step-by-step workflow per mode
│   ├── 30-governance-and-constraints.md  # Rules, scope, data handling
│   └── 40-threat-model-and-failure-modes.md  # Threats and mitigations
├── prompts/
│   ├── comet-grant-agent.metaprompt.md   # Primary deployable metaprompt
│   ├── changelog-autofill.metaprompt.md  # Companion governance prompt
│   └── prompt-snippets.md               # Reusable composable safety clauses
├── design/
│   ├── design-notes.md              # Rationale and rejected alternatives
│   ├── evaluation-plan.md           # Test protocol: golden, adversarial, metrics
│   └── logging-and-telemetry.md     # Action log format and sink specification
├── examples/
│   ├── nsf-funding-opportunity/     # Worked NSF FOA example flow
│   └── generic-foundation-rfp/     # Worked generic RFP example flow
├── ops/
│   ├── deployment-notes.md          # Runtime integration guide
│   ├── safety-checklist.md          # Pre-deployment safety checklist
│   └── change-log-template.md       # Governance changelog template
└── tooling/
    ├── schema/
    │   └── action-log.schema.json   # JSON Schema for action logs
    └── scripts/
        ├── validate-action-log.py   # Log schema validator
        └── redact-output.py         # PII redaction helper
```

---

## Intended Use Cases

- Academic and industry **grant preparation** (NSF, NIH, Sloan, CZI, Mozilla, DARPA BAAs)
- **RFP / RFI / procurement** form drafting
- **Vendor security questionnaire** and compliance portal workflows
- Any high-friction, high-stakes form + unstructured guidance page workflow where mis-submission is unacceptable

---

## Workflow Summary

```
[User provides FOA URL + DMP]
        |
        v
[Agent navigates to funding page]
        |
        v
[Extract: title, eligibility, deadlines, required fields]
        |
        v
[STOP: Present structured summary to user]
        |
        v
[User approves -> Generate draft responses from DMP]
        |
        v
[Preview-fill each field: label + selector + proposed value + DMP source]
        |
        v
[STOP: User reviews all proposed values]
        |
        v
[User approves -> Insert values into form fields]
        |
        v
[STOP: Hard stop. No submission. No confirmation click.]
```

---

## Safety and Compliance

The metaprompt enforces the following hard constraints:

1. **No submission or irreversible actions** — agent will never click Submit, Finalize, Send, Confirm, Apply, or equivalents
2. **No autonomous data transmission** — all field values require explicit user approval
3. **No hallucination or inference** — only user-provided DMP content is used; missing data triggers a clarification request
4. **No instruction hijacking** — website instructions are explicitly subordinate to the metaprompt and user instructions
5. **Mandatory human review checkpoints** — two required stops before any form interaction
6. **Full action logging** — every navigation, extraction, and field population event is logged in structured JSON
7. **Deterministic behavior** — same input state produces same output state

See [`spec/30-governance-and-constraints.md`](spec/30-governance-and-constraints.md) and [`spec/40-threat-model-and-failure-modes.md`](spec/40-threat-model-and-failure-modes.md) for full details.

---

## Supported Funding Domains

| Organization | Entry Point |
|---|---|
| NSF | research.gov → Funding Search |
| Sloan Foundation | sloan.org → Grants |
| Mozilla Foundation | mozilla.org → Grants |
| Chan Zuckerberg Initiative | chanzuckerberg.com → Science → Grants |
| ONR / AFOSR / ARO BAAs | navy.mil, afrl.af.mil, army.mil |

---

## Evaluation

See [`design/evaluation-plan.md`](design/evaluation-plan.md) for the full test protocol, including:

- **Golden cases**: clean FOA + well-formed portal, expected structured output
- **Adversarial cases**: prompt injection text on page, misleading button labels, conflicting instructions
- **Metrics**: unsafe action rate, ambiguity flag accuracy, no-submit adherence, log completeness

---

## Quick Start

1. Copy the contents of [`prompts/comet-grant-agent.metaprompt.md`](prompts/comet-grant-agent.metaprompt.md)
2. Inject as the SYSTEM prompt in your Comet-compatible browser agent runtime
3. Provide the agent with:
   - A funding opportunity URL
   - Your Data Management Plan (DMP) as context
4. Review the structured field extraction output
5. Approve or modify proposed values at each checkpoint
6. The agent will populate form fields for your final review — it will not submit

---

## Related Artifacts

- [Gist: Original Metaprompt](https://gist.github.com/novaiteraresearch/10da52e3dc698456798340992077b6e4) — canonical source this repo was derived from
- `prompts/changelog-autofill.metaprompt.md` — companion prompt for governance changelog population

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

*Maintained by [Nova Itera Research Group](https://github.com/novaiteraresearch)*
