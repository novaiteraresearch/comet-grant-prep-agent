# design/design-notes.md — Design Rationale and Rejected Alternatives

<!--
artifact: design/design-notes.md
version: 1.0.0
status: stable
tags: [design, rationale, decisions, rejected-alternatives]
-->

## Why Review-Gated Instead of Fully Autonomous

Grant preparation is a high-stakes, authorship-sensitive workflow. Autonomous form submission would:

1. **Eliminate authorship control.** A researcher is legally and ethically responsible for submitted materials. An agent submitting on their behalf removes the deliberate review step that constitutes authorship.
2. **Introduce irreversible errors.** Grant portals frequently disable editing after submission. A mis-submission may require withdrawal, re-application, or disqualification.
3. **Create liability.** Automated submission of inaccurate DMP content (e.g., from a hallucinated field) could constitute misrepresentation to a federal funding agency.
4. **Undermine auditability.** Post-hoc review of what was submitted requires the agent to have logged every action — but autonomous submission removes the human verification checkpoint that catches log errors.

**Decision:** Non-autonomous by design. Review gates are hard constraints, not configurable options.

---

## Why Separate Spec from Deployable Prompt

Combining the annotated specification and the deployable prompt in a single file creates:

- **Token bloat:** Annotations add tokens that consume context window without contributing to agent behavior.
- **Mutation risk:** Developers editing the spec accidentally modify the prompt.
- **Versioning confusion:** The spec and the deployable prompt should version independently (the spec may gain documentation without the prompt changing).

**Decision:** `spec/10-system-prompt.md` is the annotated, human-readable specification. `prompts/comet-grant-agent.metaprompt.md` is the deployable artifact.

---

## Why a Fixed Domain Allowlist

Alternatives considered:

| Alternative | Rejected Because |
|---|---|
| Allow any URL the user provides | Increases prompt injection surface; attacker-controlled pages become in-scope |
| Dynamic allowlist (user adds domains) | Requires a separate governance flow; out of scope for v1 |
| No allowlist (agent judges per-URL) | Non-deterministic; agent cannot reliably distinguish safe from unsafe URLs |

**Decision:** Fixed allowlist of 5 funding domains. Extension requires a spec update with changelog entry.

---

## Why Action Logging is Mandatory (Not Optional)

Logging was considered as an optional feature ("log if requested"):

- **Optional logging breaks reproducibility.** If a session is not logged, it cannot be audited or replicated.
- **Optional logging is a security regression.** An attacker-influenced page could instruct the agent to disable logging before a harmful action.
- **Optional logging shifts burden to the user.** Users shouldn't need to remember to enable logging before each session.

**Decision:** Logging is always-on and cannot be disabled by page content, user instruction, or configuration.

---

## Why No Cross-Session Memory

Cross-session memory was considered to allow the agent to remember previously extracted FOA data:

- **Stale data risk.** FOA requirements change (deadlines, page limits, eligibility). Cached data from a prior session may be silently incorrect.
- **Privacy risk.** DMP content from session A should not be accessible in session B without explicit user re-provision.
- **Complexity without proportionate value.** The extraction step (Mode 1) is fast and deterministic; re-running it is cheaper than managing stale cache.

**Decision:** No cross-session memory. Each session starts fresh from user-provided context.

---

## Why the Metaprompt Does Not Generate DMP Content

Automatic DMP generation was considered ("generate a DMP from the funding opportunity and user's project description"):

- **Authorship integrity.** A DMP is a primary research document. AI-generated DMPs submitted without researcher authorship raise integrity concerns.
- **Scope creep.** DMP generation is a different, larger task that would require its own spec, governance, and review process.
- **Hallucination risk amplification.** Generating DMP content from a brief project description introduces compounding hallucination risk: the agent could fabricate methods, outputs, and data handling plans that don't reflect the researcher's actual project.

**Decision:** DMP content is always user-provided. The agent matches and formats; it does not author.

---

*See also: [`spec/00-overview.md`](../spec/00-overview.md) | [`spec/30-governance-and-constraints.md`](../spec/30-governance-and-constraints.md)*
