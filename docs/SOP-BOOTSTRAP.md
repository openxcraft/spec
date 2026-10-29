---
id: SOP-BOOTSTRAP-001
type: sop
status: active
version: 1.0.0
depends_on: [SOP-CYCLE-001]
last_reviewed: 2026-10-30
owner_role: orchestrator-operator
---

# SOP: Bootstrap and Pilot Cycle

Two lines: verify the environment end to end, then run one small pilot cycle and review firewall gate results manually. Limits for this run override config: 2 products maximum, 1 specification maximum.

## 1. Verify environment

1. Confirm the tree: `state/` (config.yaml, coverage-map.yaml, priority.yaml, queue/), `taxonomy/` (categories, platforms, capabilities), `specs/`, `families/`, `reviews/`, `schemas/`, `templates/`, `docs/`.
2. Confirm `/home/z/my-project/specs-ledger/` exists with mode 0700 and is not inside the public repository.
3. Confirm `/home/z/my-project/.secrets/` files exist with mode 0600. Verify presence with `ls -l` only; never read or print secret values directly.
4. Confirm helper scripts are executable: `notify_discord.py`, `notify_telegram.py`, `push_github.sh` under `/home/z/my-project/scripts/`.
5. Send test notifications: one short line to Discord and one to the Telegram topic, for example `bootstrap check: channel verification from spec-orchestrator`. Do not include secret values or internal paths in messages.
6. Verify git in `/home/z/my-project/specs-catalog`: branch `main`, remote origin `https://github.com/openxcraft/spec.git`. If the initial push has not happened, run `scripts/push_github.sh "init: repository structure, taxonomy, state, docs"`.

## 2. Run the pilot cycle

Execute `docs/SOP-CYCLE.md` stages 1 through 12 with these overrides:

- Discover 2 products maximum across different category-platform cells.
- Publish 1 specification maximum.
- All 8 firewall gate checks are mandatory; the reverse-identification and similarity checks must be executed explicitly and their results recorded in the manifest.

## 3. Manual gate review

Before finishing, produce a gate review table in the pilot report listing each of the 8 checks, its result, and the evidence inspected. If any check is ambiguous, hold the spec in `status: draft`, append the concern to `/home/z/my-project/specs-ledger/escalations.yaml`, and state the hold in the notifications.

## 4. Close out

1. Write `reviews/BOOT-YYYYMMDD.md` with: environment verification results, pilot cycle metrics, gate review table, deviations from SOPs, and recommendations before full-rate operation.
2. Update ledger, coverage map, priority list, and push the repository.
3. Send a summary to Discord and the Telegram topic: what was verified, what was published, what is on hold, and the recommended next step.

## 5. Escalate immediately if

- The GitHub push fails for credential or permission reasons.
- Either notification channel is unreachable.
- The pilot spec cannot pass the reverse-identification check after two generalization attempts.
