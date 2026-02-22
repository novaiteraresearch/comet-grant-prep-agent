<!--
artifact: prompts/changelog-autofill.metaprompt.md
version: 1.0.0
status: stable
purpose: Governance companion prompt for populating Change Log properties
target: Notion database property | any append-only changelog field
tags: [changelog, governance, autofill, audit, versioning]
-->

# Changelog Autofill Metaprompt

> **Purpose:** Companion governance prompt for deterministically populating a **Change Log** property in Notion (or any structured changelog field) when this repo's artifacts are modified.

> **Usage:** Paste as the instruction prompt when asking an AI to fill a Change Log property. Provide current page content and version history as context.

---

## What a Changelog Is

A **changelog** is a chronological, append-only record of material changes to an artifact (document, prompt, spec, repo, dataset). It answers:

- What changed?
- When did it change?
- Why did it change?
- Who/what authorized it?
- What is the impact (including safety/regression risk)?

**Purpose:** Auditability, accountability, reproducibility, risk control, and operational clarity.

---

## Autofill Prompt (Copy Verbatim)

```
SYSTEM / INSTRUCTION

You are generating content for a property named "Change Log" for a single
artifact page.

INPUTS YOU MAY USE
- The current page content.
- The page's version history if provided.
- The page's database properties (Title, Author, Created Date, Last Updated,
  etc.) if provided.

OUTPUT FORMAT (MUST)
Return ONLY a changelog entry in the following YAML block.
No extra prose before or after.

CHANGELOG
- date:
  time_local: America/Los_Angeles
  version:
  author:
  change_type:         # cleanup | docs | feature | governance | safety
  scope:               # which section(s) changed
  summary:             # 1-2 sentences, factual, testable
  rationale:           # 1 sentence
  safety_impact:       # none | low | medium | high
  breaking_change:     # true | false
  verification:        # not_verified | diff_confirmed | reviewed_by:<name>
  references:
    -

RULES (HARD CONSTRAINTS)
1. Do not dilute: describe actual changes only. No recommendations.
2. No hallucinations: if author, time, or specifics are unknown, set to null
   or use verification: not_verified.
3. Prefer material changes over cosmetic edits. If only formatting changed:
   change_type: cleanup, safety_impact: none.
4. If the edit modified safety constraints, stop conditions, allowed domains,
   data handling, or review gates:
   - change_type: safety
   - safety_impact: medium or high (choose conservatively)
5. summary MUST be testable: what would differ before vs. after?
6. Keep entries append-only: new entries at the top (most recent first).

DEFAULTS
- version: null unless an explicit version identifier is present.
- verification: not_verified unless diffs or snapshots were provided.

STOP
Do not include anything besides the YAML block.
```

---

## Integration Notes

- This prompt is designed for use alongside [`ops/change-log-template.md`](../ops/change-log-template.md).
- When a spec file in `spec/` is modified, the `change_type` should be evaluated against the safety classification rules above.
- The `references` field should point to the commit hash, PR, or Notion page that authorized the change.

---

*See also: [`ops/change-log-template.md`](../ops/change-log-template.md) | [`spec/30-governance-and-constraints.md`](../spec/30-governance-and-constraints.md)*
