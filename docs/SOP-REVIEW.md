---
id: SOP-REVIEW-001
type: sop
status: active
version: 1.0.0
depends_on: [SOP-CYCLE-001, SOP-AUDIT-001]
last_reviewed: 2026-10-30
owner_role: orchestrator-operator
---

# SOP: Quarterly Review

Two lines: re-audit a random sample of published specifications with the strongest checks, verify platform limits against official documentation, and confirm takedown readiness. Run on the first day of January, April, July, and October at 09:00 (Asia/Baghdad).

## 1. Sample selection

1. Sample `review.quarterly_sample_size` (default 10) published specifications at random, or 20 percent of the catalog if smaller. Record the selection method and seed in the report; the sample list uses only public ids.
2. Include at least 2 specs from each of the highest-tier (T2) families when they exist.

## 2. Re-run strong checks per sampled spec

1. Reverse-identification: attempt to identify any origin product from the published specification alone. High-confidence identification fails the spec.
2. Similarity: compare each spec against its public sources again (source references live only in the private ledger; the reviewing role may read the ledger). Notable linguistic match fails the spec.
3. Neutrality and name-free re-check of every file in the package, including generated indexes.

## 3. Handling failures

1. Failing spec: set `status: suspended` immediately, remove it from generated indexes in the same commit, append to `/home/z/my-project/specs-ledger/escalations.yaml` with reasons.
2. If the failure reveals a systemic gate weakness, note the required firewall gate update and record it as an action item; on a confirmed leak, follow SOP-CYCLE stage 13 withdrawal procedure.

## 4. Platform limits verification

Verify current platform limits referenced in NFRs (for example maximum sizes, rates, and counts) against official documentation. Update affected NFRs with new numbers and a changelog line in the spec; record stale limits found in the report. This satisfies the quarterly platform-limits review cadence in config.

## 5. Legal posture

1. Review all `legal_flags` and open escalations; resolve or re-confirm each.
2. Review `specs-ledger/takedowns.yaml` handling: every event must show immediate suspension, escalation, and ledger recording.
3. Verify ledger completeness: every published spec has provenance with source link, access date, source type, extraction date, cycle id, gate result, legal risk, and checksum.

## 6. Close out

1. Write `reviews/QAR-YYYYQN.md`: sample list (public ids), check results, suspensions, NFR updates, legal posture, and actions for the next quarter.
2. Commit and run `scripts/push_github.sh "quarterly review YYYY-QN"`.
3. Send a summary via `scripts/notify_discord.py` and `scripts/notify_telegram.py`.
