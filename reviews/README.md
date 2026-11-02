---
id: REVIEWS-README
type: readme
status: active
version: 1.0.0
depends_on: [SOP-BOOTSTRAP-001, SOP-CYCLE-001]
last_reviewed: 2026-11-02
owner_role: orchestrator-operator
---

Directory contract for bootstrap, cycle, and audit reports.
Reports record verification evidence, gate outcomes, deviations, and follow-up actions.

# Review reports

Bootstrap reports use the `BOOT-YYYYMMDD.md` naming convention described in `docs/SOP-BOOTSTRAP.md`. Cycle and audit reports must follow the corresponding procedure and include enough evidence to reproduce the reported checks.

Reports must not include secret values, private-ledger contents, user data, or raw discovery notes. Keep restricted provenance and escalation records outside this public repository.
