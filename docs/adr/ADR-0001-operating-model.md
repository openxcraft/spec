---
id: ADR-0001
type: adr
status: accepted
version: 1.0.0
depends_on: []
last_reviewed: 2026-10-30
owner_role: orchestrator-operator
---

# ADR-0001: Initial operating model

Two lines: this record fixes the first operating decisions of the orchestrator and the known limitations accepted at start.

## Decisions

1. Identifier space: 9-digit identifiers, `SPEC-000000001` onward, sharded `specs/AAA/BBB/SPEC-AAA-BBBBBBB` with a cap of 1,000 entries per directory. New shard opens as `specs-catalog-NNN` with a follow-up ADR.
2. Ledger isolation: provenance lives in a physically separate directory `/home/z/my-project/specs-ledger` with mode 0700, outside the public repository. The public side carries only `provenance_ref` (opaque `PRV-XXXXXXXX`).
3. Secret isolation: credentials live in `/home/z/my-project/.secrets` with mode 0700/0600. Agents load them only through helper scripts. Values never enter prompts, reports, or the repository.
4. Zone separation at start: a single scheduled agent executes stages sequentially. Raw notes are written under `state/raw/<cycle-id>/`, and the directory is deleted after the firewall gate of the same cycle. Stage discipline is procedural, not filesystem-enforced. When multiple agent runtimes become available, enforce raw-note read denial with file permissions and revisit this ADR.
5. Notification channels: Discord webhook and Telegram topic for reports, GitHub `openxcraft/spec` for publication. GitLab push is pending a credential and stays disabled in configuration.
6. Schedule: two scan cycles per day at 06:00 and 18:00 Asia/Baghdad, a weekly coverage audit on Mondays 10:00, and a quarterly review on the first day of January, April, July, October at 09:00.
