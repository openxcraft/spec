---
id: SPECS-README
type: readme
status: active
version: 1.0.0
depends_on: [SOP-CYCLE-001]
last_reviewed: 2026-11-02
owner_role: orchestrator-operator
---

Directory contract for published specification packages.
Only packages approved by the required firewall review belong here.

# Published specifications

The scan cycle authors packages beneath this directory using the sharded layout defined in `docs/SOP-CYCLE.md`.

Each package must include the manifest and required documents listed in that procedure. The manifest must conform to `schemas/spec.schema.json`, and every required gate result must be recorded before publication.

Do not store raw discovery notes, private provenance, credentials, or unpublished research material in this public directory.
