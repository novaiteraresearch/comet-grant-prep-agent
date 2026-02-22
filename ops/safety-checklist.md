# ops/safety-checklist.md — Pre-Deployment Safety Checklist

<!--
artifact: ops/safety-checklist.md
version: 1.0.0
status: stable
tags: [ops, safety, checklist, deployment, pre-flight]
-->

> Complete this checklist before deploying the metaprompt in any new runtime environment. All items must be checked before a session is considered production-ready.

---

## Section 1 — Prompt Integrity

- [ ] The SYSTEM prompt matches `prompts/comet-grant-agent.metaprompt.md` Section 1 verbatim
- [ ] No additional SYSTEM prompt sections have been added that grant broader permissions
- [ ] The metaprompt version in use is documented (check `Section 3 - Version History` in the prompt file)
- [ ] The metaprompt has not been modified without a corresponding changelog entry

---

## Section 2 — No-Submit Enforcement

- [ ] The runtime does not expose autonomous form submission capabilities to the agent
- [ ] Stop conditions are tested: agent halts on Submit, Finalize, Send, Confirm, Apply, and equivalent buttons
- [ ] A golden-case test (GC-03 from evaluation-plan.md) has been run and passed in this runtime
- [ ] No timeout-based auto-proceed has been added to review gates

---

## Section 3 — Logging

- [ ] Action logging is wired to a persistent sink (file, database, or equivalent) before session start
- [ ] Log schema matches `tooling/schema/action-log.schema.json`
- [ ] Logs from the test run have been validated with `tooling/scripts/validate-action-log.py`
- [ ] Log retention policy is documented and meets the 90-day minimum
- [ ] Redaction policy is applied before any logs are shared externally

---

## Section 4 — Instruction Hierarchy

- [ ] The runtime does not inject additional instructions between the SYSTEM prompt and user messages
- [ ] An adversarial test (AC-01 from evaluation-plan.md) has been run and passed
- [ ] The domain allowlist has not been extended without a spec update and changelog entry

---

## Section 5 — Data Handling

- [ ] The agent cannot access browser storage, history, bookmarks, or saved credentials in this runtime
- [ ] No DMP content is persisted outside the session without user consent
- [ ] PII redaction is applied to logs before any third-party sharing
- [ ] Cross-session memory is disabled

---

## Section 6 — Human Review Gates

- [ ] Gate 1 (post-extraction) is confirmed functional: agent halts and presents summary before Mode 2
- [ ] Gate 2 (pre-fill) is confirmed functional: agent halts and presents full field table before insertion
- [ ] Neither gate has a bypass, auto-proceed, or timeout mechanism
- [ ] User approval language is documented: what constitutes valid approval vs. ambiguous response

---

## Section 7 — Versioning and Governance

- [ ] This deployment corresponds to a specific metaprompt version (document the version here: ___)
- [ ] If any spec file was modified since the last deployment, a changelog entry has been created
- [ ] Safety-classified changes (`change_type: safety`) have been reviewed before deployment
- [ ] The changelog is accessible to all team members with access to this runtime

---

## Sign-Off

```
Checklist completed by: ___________________
Date: ___________________
Runtime environment: ___________________
Metaprompt version deployed: ___________________
Notes: ___________________
```

---

*See also: [`ops/deployment-notes.md`](deployment-notes.md) | [`design/evaluation-plan.md`](../design/evaluation-plan.md)*
