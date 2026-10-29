---
id: SOP-AUDIT-001
type: sop
status: active
version: 1.0.0
depends_on: [SOP-CYCLE-001]
last_reviewed: 2026-10-30
owner_role: orchestrator-operator
---

# SOP: Weekly Coverage Audit

Two lines: verify quotas, repository integrity, and regenerate the priority list for the coming cycles. Run weekly; produce a name-free metrics report.

## 1. Quota checks (from `state/config.yaml`)

1. Per-cell minimum: every active coverage-map cell must reach at least `min_discoveries_per_active_cell_per_week` (default 2) discoveries per week. Use `discoveries_last_7d` and ledger timestamps. Cells below minimum become top priority entries.
2. Staleness: cells untouched for more than `cell_staleness_days` (default 30) get a priority boost and a note.
3. Category share: no category may exceed `max_category_share_per_cycle` of output averaged over the last 7 days; flag violations for the next cycles' target selection.
4. Known vs niche ratio: compare against `known_vs_niche_target_ratio`; note the correction needed in the next cycles.

## 2. Repository integrity

1. Validate every `spec.yaml` against `schemas/spec.schema.json`; list failures with ids.
2. Recompute the sha256 checksum of each published package and compare with the manifest `checksum` and the private ledger `spec_checksum`. Mismatches are a security event: suspend the spec, append to `/home/z/my-project/specs-ledger/escalations.yaml`.
3. Confirm every manifest `provenance_ref` has a matching ledger entry and vice versa. Orphans escalate.
4. Confirm `state/raw/` holds no leftovers from completed cycles (zone destruction rule).
5. Regenerate all indexes (category, platform, capability, family, tier, status, full-text) and confirm pages hold at most 100 entries.

## 3. Priority list

Rebuild `state/priority.yaml`: order cells by (below-minimum discoveries, staleness, lowest coverage) so the next cycles pick targets directly from it. Include the safe-stop suspensions so suspended targets are not selected.

## 4. Rejection-rate watch

Compute the rejection rate over the trailing `rejection_rate_window_cycles`. If it exceeds `rejection_rate_suspend_threshold`, state loudly in the report and notifications that scheduling should be suspended pending human review.

## 5. Close out

1. Write `reviews/AUD-YYYYMMDD.md`: quota results, integrity findings, priority changes, metrics (coverage ratio, backlog size, rejection rate, family count, spec count), and next-week focus.
2. Commit and run `scripts/push_github.sh "weekly audit YYYY-MM-DD"`.
3. Send the report summary via `scripts/notify_discord.py` and `scripts/notify_telegram.py`. Name-free, no secrets, no internal paths.
