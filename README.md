# Precision Manufacturing Ontology

A specification for the entity model behind aerospace precision manufacturing. Defines how a machined, finished, assembled, and certifiable part is composed — from drawing callout to material lot to outside-processing operation to FAI to shipment cert package — in a way that's software-agnostic and agent-consumable.

## The Problem

NIST and the MTConnect / QIF / STEP / X12 consortia have defined a four-standard stack for the digital thread at prime contractors. The standards stop at the contract-manufacturer boundary. On the other side — where parts are actually made — nobody has published the entity model. This repo is that model.

## Layered Architecture

| Layer | What | Status |
|---|---|---|
| **Layer 1 — Standards** | STEP AP242, MTConnect, QIF, X12 EDI | Adopted unmodified. See [`standards/`](standards/). |
| **Layer 2 — Manufacturer Data Model** | manufacturer-defined entities that fill the manufacturer-layer gap | Canonical. See [`extensions/`](extensions/) and [`schemas/`](schemas/). |
| **Layer 3 — Data Layer** | A conforming system of record | Reference implementations in [`implementations/`](implementations/). |
| **Layer 4 — Interface Layer** | How operators and AI interact with Layer 3 | Out of scope for the spec. |

## Organizing Principle

The ontology is measured by composition, not by domain polish. Success means: any real complex aerospace part — machined, finished, assembled, inspected under AS9102, shipped with a full cert package — can be described by walking its decomposition tree against the spec. Domains (materials, quality, supply chain, …) are the filing system. The composition test is the measuring stick.

## Navigating This Repo

**Start here** (humans and agents):
- [`INDEX.md`](INDEX.md) — every entity, alphabetical, one line each, linking to schema + prose + matrix row.
- [`reference/composition-archetypes.md`](reference/composition-archetypes.md) — three walked example part decompositions (single machined; outside-processed + FAI; multi-part assembly).
- [`reference/entity-matrix.md`](reference/entity-matrix.md) — every entity × every completeness dimension, classified.

**The canon** (Layer 2):
- [`reference/domain-map.md`](reference/domain-map.md) — twelve operational domains plus cross-domain extensions and their manufacturer-layer gaps.
- [`extensions/`](extensions/) — per-gap specifications (material certs, outside processing, quality flowdowns, supplier approval, IP boundary, UUID discipline, process engineering/NRE, tool room, packaging, change management, inspection/calibration).
- [`schemas/`](schemas/) — machine-readable entity definitions (YAML).

**References:**
- [`standards/`](standards/) — Layer 1 briefs.
- [`reference/relationship-graph.yaml`](reference/relationship-graph.yaml) — machine-readable cross-entity links (stale at 137 edges; pending update for Phase 2 + InspectAI entities).
- [`implementations/`](implementations/) — reference mappings to Carbon, Fulcrum, and a template for future systems.
- [`reference/carbon-novelty-2026-04.md`](reference/carbon-novelty-2026-04.md) — latest Carbon scan findings.

## Contributing

A new contributor — human or agent — can reach any entity in ≤2 hops from here. Start with [`INDEX.md`](INDEX.md). To propose changes, consult the archetype walks first: if your change doesn't improve a representative part's decomposition, it's a filing-system change, not a canon change.

## Status

Active. Phase 2 (12 domains, 117 entities) completed 2026-04-24; InspectAI-derived quality entities added same day. Schema coverage: 94 of 123 entities fully specified; remaining are intentional (external standards not authored here, string/enum type decisions, aliases). Two machine-readable reference files stale pending update: `reference/entity-inventory.yaml` (still 82 entries) and `reference/relationship-graph.yaml` (still 137 edges). See [`reference/residual-gaps.md`](reference/residual-gaps.md) for deferred items and architectural decisions.

## License

Apache 2.0. See [LICENSE](LICENSE).
