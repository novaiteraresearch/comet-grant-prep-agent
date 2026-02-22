# spec/00-overview.md — Problem Statement, Design Goals, and Non-Goals

<!--
artifact: spec/00-overview.md
version: 1.0.0
status: stable
tags: [spec, overview, design-goals, problem-statement]
-->

## Problem Statement

Grant preparation workflows are high-stakes and error-prone. Researchers and administrators must:

- Navigate complex, inconsistently structured funding opportunity pages (NSF, NIH, Sloan, CZI, DoD BAAs)
- Extract eligibility criteria, deadlines, required proposal components, and form field schemas
- Draft responses that accurately reflect their project's DMP, methods, and impact
- Populate online portal forms that are brittle, session-sensitive, and irreversible on submission

Existing automation approaches (RPA, general-purpose browser agents) introduce unacceptable risk: silent form submission, hallucinated field values, and loss of authorship control.

This project addresses that gap with a **governance-first, review-gated browser metaprompt** that operationalizes human-in-the-loop control as a hard constraint, not a recommendation.

---

## Design Goals

### G1 — Non-Autonomy by Default
The agent never takes irreversible actions without explicit user confirmation. Submission, finalization, and data transmission are unconditionally prohibited.

### G2 — Auditability
Every agent action (navigation, extraction, field population) is logged in structured JSON with timestamp, URL, element selector, and proposed value. Logs are diffable and machine-checkable.

### G3 — Grounded Drafting
All draft content is derived exclusively from user-provided DMP material. The agent does not infer, hallucinate, or interpolate missing information. Missing data triggers an explicit clarification request.

### G4 — Determinism
Given the same funding page and DMP, the agent produces the same structured output. No stochastic field selection or ambiguous tie-breaking without documented rules.

### G5 — Portability
The metaprompt is designed for multi-domain use across NSF, Sloan, Mozilla, CZI, and DoD BAA portals without extensive per-domain hand-tuning.

### G6 — Instruction Hierarchy Enforcement
Website instructions (including injected text, DOM content, and misleading UI labels) are explicitly subordinate to the metaprompt and user instructions. The agent cannot be redirected by page content.

---

## Non-Goals

| Non-Goal | Rationale |
|---|---|
| Autonomous form submission | Submission requires researcher judgment; out of scope by design |
| Full application authorship | The agent drafts; the researcher authors |
| Broad web browsing | Only user-specified funding opportunity URLs are in scope |
| API or email integrations | No external calls outside the browser runtime |
| Multi-agent orchestration | Single-agent, single-session workflow only |
| Dynamic DMP generation | The DMP is user-provided; the agent does not write it |

---

## Governance Framing

This metaprompt is a concrete implementation of **controlled autonomy**: the agent is capable of browser automation but constrained to a safe subset of actions through enforceable prompt-level rules, stop conditions, and mandatory review gates.

It is intended as a reusable governance pattern applicable beyond grant preparation to any high-stakes, human-in-the-loop form-filling workflow (procurement, compliance, vendor security questionnaires, HR portals).

---

*See also: [10-system-prompt.md](10-system-prompt.md) | [30-governance-and-constraints.md](30-governance-and-constraints.md)*
