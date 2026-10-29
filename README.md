---
id: REPO-README
type: readme
status: active
version: 1.0.0
depends_on: []
last_reviewed: 2026-10-30
owner_role: orchestrator-operator
---

Abstract functional specifications of publicly observable software capabilities, published clean-room with no reference to any origin product or owner.

This repository is machine-first: state lives in `state/`, taxonomy in `taxonomy/`, publications in `specs/`, grouped capability families in `families/`, and cycle reports in `reviews/`.

## Guarantees

1. Published specifications contain no product, company, brand, feature, or person names.
2. No code, assets, marketing text, visual identity, or databases are carried from any source.
3. Sources are limited to public documentation, observable lawful use, open standards, and public reviews.
4. Every published identifier maps to provenance held only in a separate, restricted private ledger.
5. Capabilities that may touch patent claims are held from publication until human review.

## Layout

| Path | Content |
|---|---|
| state/ | Configuration, coverage map, per-item queue, priority list |
| taxonomy/ | Categories, platforms, global capability taxonomy |
| specs/ | Published specification packages, sharded by identifier |
| families/ | Grouped similar specifications sharing common capabilities |
| reviews/ | Cycle reports and audits |
| schemas/ | Validation schemas for manifests and front matter |
| templates/ | Starting templates for new specifications |
| docs/ | Operating procedures and architecture decision records |

## Takedown

On a takedown request or legal notice: suspend the specification immediately, escalate to a human, and record the event in the private ledger. Contact route: repository owner role `orchestrator-operator`.
