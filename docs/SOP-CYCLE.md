---
id: SOP-CYCLE-001
type: sop
status: active
version: 1.0.0
depends_on: [ADR-0001]
last_reviewed: 2026-10-30
owner_role: orchestrator-operator
---

# SOP: Scan Cycle

Two lines: execute one full discovery-publication cycle for the clean-room specification repository. Every stage below is mandatory; quotas and thresholds come from `state/config.yaml`.

## 0. Identity and absolute rules

- You operate under clean-room rules. Published output describes WHAT a system does (functions, business rules, states, limits), never HOW it was implemented internally and never HOW it looks expressively.
- Permitted sources only: public product documentation, feature pages, public user guides, observable behavior from lawful use of publicly available copies without bypassing any barrier, open standards and publicly documented protocols, published reviews and comparisons.
- Strictly prohibited: unlicensed or leaked source code, decompilation or reverse engineering where forbidden, bypassing authentication, paywalls, technical protections, rate limits, robots files, or terms of service, collecting user or private data, copying names, trademarks, logos, icons, images, marketing text, distinctive visual design, or databases, and any information from a person bound by confidentiality.
- Writing rules for every public file: no names of people, organizations, products, or registered features; no emoji; no generation traces (no "as a model", no intros, apologies, closings); short declarative sentences; numbers instead of adjectives; terminology consistent with the spec glossary; every assumption recorded with an id, confidence, and verification method, tagged `unverified` until verified.
- Every file in the repository starts with YAML front matter: `id`, `type`, `status`, `version`, `depends_on`, `last_reviewed`, `owner_role`, plus a two-line summary at the top.
- Secrets live in `/home/z/my-project/.secrets` (mode 0600). Never print, echo, commit, or embed them. Use only the helper scripts in `/home/z/my-project/scripts/`.

## 1. Read state

1. Load `state/config.yaml` (quotas, thresholds), `state/coverage-map.yaml`, `state/priority.yaml`, and per-item queue files in `state/queue/`.
2. Determine the cycle id: `CYC-YYYYMMDD-N` where N is 1 for the 06:00 cycle and 2 for the 18:00 cycle (Asia/Baghdad).
3. Create the raw-notes zone `state/raw/<cycle-id>/` for this cycle only.

## 2. Select scan targets

1. Rank coverage-map cells: least-covered first (lowest `discovered_total`, then lowest `discoveries_last_7d`), then cells with the oldest or null `last_reviewed`.
2. Apply diversity constraints: no single category may exceed `max_category_share_per_cycle` of this cycle's output; the mix of well-known and lesser-known products must approach `known_vs_niche_target_ratio`.
3. Respect safe stop: any target with 3 consecutive recorded failures is suspended and escalated, not retried. Record suspensions in the queue file for the item.

## 3. Deep discovery (swarm angles)

Work each angle as an independent search pass, recording raw notes per product in `state/raw/<cycle-id>/`:

1. Category sweep for the selected cells.
2. Platform sweep (web, mobile, desktop, command line, browser extensions, API and services, embedded, cloud platforms, open-source tools).
3. Ecosystem-region sweep (products prominent outside the largest English-language markets).
4. Emerging products (publicly launched or publicly announced in the last 12 months).
5. Niche and lesser-known products.
6. Open-source alternatives in the category.
7. Publicly announced internal tools (only from public engineering blogs or conference talks; stop at the first sign of non-public detail).

For each product note: observable capabilities, documented limits and business rules, public source references (link plus access date), source type. Notes stay in the raw zone only.

## 4. Deduplicate

1. Build a capability fingerprint per candidate: the set of `CAP-####` ids from `taxonomy/capabilities.yaml` matching its observed functions. Extend the taxonomy with new generic capabilities when nothing fits; new ids append to the taxonomy in the same cycle.
2. Match candidates against published specs and queue items by fingerprint, never by name. Jaccard similarity at or above `fingerprint_similarity_threshold` (0.60) means duplicate or family candidate.
3. Duplicates do not produce new specs; they increment `discovered_total` on their cell and link into the existing family.

## 5. Evaluate and triage

Score each unique candidate: `functional_value` (0-5), `source_clarity` (0-5), `legal_risk` (0-5, higher is worse). Priority score = `functional_value + source_clarity + (5 - legal_risk)`.

- `legal_risk` 0-2: eligible for the automated pipeline.
- `legal_risk` 3-4: route to human review; append the item to `/home/z/my-project/specs-ledger/escalations.yaml` and skip publication this cycle.
- `legal_risk` 5: reject; record the aggregate reason in the cycle report without naming the product.

Write one queue file per processed item in `state/queue/` (status, cell, fingerprint, triage scores, safe-stop counter). Save state after this stage.

## 6. Extract and abstract (6 stages)

1. Capability extraction: list observed functions and behaviors, each tagged `publicly-documented`, `observed`, or `inferred`.
2. Generalization: replace every product-specific term with a generic functional term; remove commercial names, registered feature names, plan names, distinctive terminology.
3. Idea-expression separation: keep functional rules; drop wording, distinctive visual organization, and assets.
4. Recomposition: organize capabilities in a new structure that does not mirror the source's screen order or module names.
5. Design diversification: include independent design alternatives for sensitive components so implementation is not bound to imitating any source.
6. Patent screening: tag capabilities that may touch publicly known patent claims as `patent-review-required`; such specs are not published before human review (append to `specs-ledger/escalations.yaml`).

Delete nothing from the raw zone yet; the firewall review still needs it.

## 7. Firewall gate (all 8 checks must pass)

Run as an independent review pass over raw notes and abstractions (never self-certify without re-reading the artifacts):

1. Name-free: no product, company, brand, registered feature, or person name; no abbreviation that points to any of them.
2. Expression-free: no copied text; no visual description matching a distinctive identity; nothing inferable only from unauthorized access.
3. Asset-free: no logos, icons, brand colors, licensed fonts, or images.
4. Secret-free: no keys, credentials, internal addresses, or user data.
5. Similarity: automated text comparison of the abstraction against the public source; any notable linguistic match fails the gate.
6. Reverse-identification: attempt to identify the original product from the abstraction alone. High confidence means reject and return to stage 6.2 (generalization).
7. Sufficiency: an implementation plan can be built from the abstraction alone with no fundamental questions.
8. Neutrality: no promotion, preference, or comparison with any named product.

Record each check result per spec candidate. Any failure: return to the matching abstraction stage, fix, re-run the full gate. After the gate completes for the cycle, delete `state/raw/<cycle-id>/` (zone destruction, per ADR-0001).

## 8. Authoring (approved abstractions only)

Author the 12-part package at `specs/{id[0:3]}/{id[3:6]}/SPEC-{id9}/` with 9-digit id, exactly these files:

1. `spec.yaml` manifest: required fields per `templates/spec.yaml.template` and `schemas/spec.schema.json`; gates recorded with result and date; `provenance_ref` is a fresh opaque `PRV-XXXXXXXX`; `checksum` is the sha256 of the package computed after authoring.
2. `README.md`: two-line summary and package map.
3. `00-summary.md`: problem solved, user roles without names, in-scope and out-of-scope.
4. `01-requirements.md`: functional requirements `FR-###` with Given/When/Then acceptance criteria; non-functional requirements `NFR-###` with measurable numbers.
5. `02-domain-model.md`: entities, relationships, states, business rules, edge cases; Mermaid entity-relationship and state diagrams.
6. `03-flows.md`: user flows and visually neutral text wireframes; Mermaid sequence and context diagrams where useful.
7. `04-contracts.yaml`: OpenAPI or JSON Schema interface contracts designed from scratch.
8. `05-security-privacy.md`: threat model, data minimization, controls.
9. `06-architecture-options.md`: at least three alternatives with a weighted criteria matrix; impose no technology.
10. `07-test-strategy.md`: test strategy and traceability matrix to FR/NFR ids.
11. `08-plan.yaml`: small tasks with ids, dependencies, and a definition of done.
12. `glossary.md`: consistent terminology for the spec.

Respect `max_new_specs_per_cycle` (default 4). Assign each spec to a family (see stage 10) and record `family` and `related` in the manifest.

## 9. Verification

Separate pass over the published files only: confirm each spec suffices to build an independent system on its own, with no fundamental questions and no missing contract. Failures go back to authoring. Record the verifier result in the manifest gates.

## 10. Family grouping and indexing

1. Above the similarity threshold, merge specs into a family directory `families/FAM-######-<generic-slug>/` with `family.yaml`, `common-capabilities.md`, `variants.md`. Shared capabilities live once in the family; each spec holds only its differences.
2. Link similar specs through `related` in manifests.
3. Regenerate indexes: by category, platform, capability, family, tier, status, plus a full-text search index. Index pages hold 100 entries. Indexes are generated as release artifacts or `state/indexes/` content, never hand-edited.

## 11. Publish and notify

1. Commit with a neutral message (`cycle CYC-YYYYMMDD-N: n specs published`) and run `scripts/push_github.sh "<message>"` to push the public repository to GitHub.
2. Append one provenance entry per published spec to `/home/z/my-project/specs-ledger/ledger.yaml` (fields listed in `specs-ledger/README.md`). Never copy ledger content into the public repo.
3. Update `state/coverage-map.yaml` cells (discovered_total, specs_published, discoveries_last_7d, last_reviewed) and regenerate `state/priority.yaml`.
4. Send the cycle report summary to Discord (`scripts/notify_discord.py`) and to the Telegram topic (`scripts/notify_telegram.py`). Reports are name-free like everything else public.

## 12. Cycle report

Write `reviews/CYC-YYYYMMDD-N.md` with the 8 required sections:

1. Scan targets and coverage achieved.
2. Products discovered, duplicated, rejected (aggregate reasons only, no names).
3. Specifications added and promoted.
4. Gate results and rejection cases.
5. New or updated families.
6. Legal risks observed and escalated.
7. Metrics: coverage ratio, cycle duration, rejection rate, backlog size.
8. Next cycle plan.

## 13. Suspension and escalation

- Rejection rate = rejected gate attempts / total gate attempts. If the rate over the trailing `rejection_rate_window_cycles` (default 3) exceeds `rejection_rate_suspend_threshold` (default 0.50), state loudly in the report and the notifications that scheduling should be suspended pending human review, and stop producing new specs for the remainder of the cycle.
- On any legal notice or takedown request encountered: stop related processing immediately, set the affected spec `status: suspended`, append the event to `specs-ledger/takedowns.yaml`, and escalate in the report.
- On discovering a leaked name or asset in a published spec: set `status: withdrawn`, remove the content in the same commit, append an escalation entry, and note the required firewall gate update in the report.

## 14. Bounded execution

Work in bounded batches and save state after every stage (queue files, coverage map, reports). If the session ends mid-cycle, the next cycle resumes from state without repeating completed stages.
