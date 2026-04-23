# Coherence Pass Restructure Notes (2026-04-22)

These notes document every file move, rename, and new file created during the coherence pass. Serves as the commit-message proxy (project is not git-tracked) and as a reviewable artifact.

## File moves

| From | To | Rationale |
|------|----|-----------|
| `reference/carbon-os-mapping.md` | `implementations/carbon.md` | Spec §2.2 — Carbon is a reference implementation, not the canonical ontology. Mapping moves to a sibling directory to demote Carbon's structural role. Executed in Task 11. |

## New files

| Path | Purpose | Source |
|------|---------|--------|
| `implementations/README.md` | Orients readers to the reference-implementations directory. | Task 1 |
| `implementations/fulcrum.md` | the production system reference mapping — peer to `carbon.md`. Captures the manufacturer's system of record as-mapped to the ontology. | Task 11 |
| `implementations/_template.md` | Onboarding template for future implementations. | Task 11 |
| `reference/composition-archetypes.md` | Three walked archetype decomposition trees (single machined; outside-processed + FAI; multi-part assembly). | Tasks 2, 3, 4 |
| `reference/entity-inventory.yaml` | Machine-readable entity list (80 entities). | Task 5 |
| `reference/entity-matrix.md` | Entity × completeness matrix with state-of-the-ontology narrative. | Task 6 |
| `INDEX.md` | Repo-root entity catalog; one line per entity with links to schema / prose / matrix row. | Task 7 |
| `reference/relationship-graph.yaml` | Machine-readable enumeration of every edge (137 unique). | Task 8 |
| `reference/carbon-novelty-2026-04.md` | Findings from targeted Carbon novelty scan (3 adopt, 9 map, 6 ignore). | Task 9 |
| `reference/restructure-notes.md` | This document. | Task 10 |
| `README.md` | Repo-root orientation (≤500 tokens); canonical navigation starting point. | Task 12 |
| `docs/superpowers/plans/2026-04-22-ontology-deepening-pass.md` | Implementation plan for the deepening pass, authored against the matrix. | Task 13 |

## Anticipated follow-ups

Tracked in the deepening-pass plan (Task 13 output). Expect:
- Schema authoring for ~77 entities currently without a YAML schema
- State-machine formalization for 14 lifecycle-bearing entities
- Resolution of 9 taxonomic open questions flagged by the matrix
- Definition of 7 entities referenced only by UUID (customer, sales_order, user, item, inventory_location, bom_line, part_specification)
- De-Carbonization of `schemas/material.yaml` alloy_family/form enums (the one meaningful Carbon-framing residue identified by the matrix; the rest of the spec is already canonical)
- Evaluation of the NCR system-action taxonomy adopted from Carbon's recent shipping

## No deletions

No files deleted in this pass. Files currently referenced by other docs remain in place; de-Carbonization of individual entity framings (where needed) happens inline during the deepening pass rather than by removal.
