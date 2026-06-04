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
- [`reference/composition-archetypes.md`](reference/composition-archetypes.md) — eight walked part decompositions: three core archetypes (single machined; outside-processed + FAI; multi-part assembly) plus five lifecycle walks (calibration recall, ECN mid-production, fixture lifecycle, concurrent-revision production, and design ingestion / CAM programming).
- [`reference/entity-matrix.md`](reference/entity-matrix.md) — every entity × every completeness dimension, classified.

**The canon** (Layer 2):
- [`reference/domain-map.md`](reference/domain-map.md) — twelve operational domains plus cross-domain extensions and their manufacturer-layer gaps.
- [`extensions/`](extensions/) — per-gap specifications (material certs, outside processing, quality flowdowns, supplier approval, IP boundary, UUID discipline, provenance & verification, process engineering/NRE, tool room, packaging, change management, inspection/calibration).
- [`extensions/provenance-and-verification.md`](extensions/provenance-and-verification.md) — the cross-cutting epistemic substrate: how every asserted fact carries method, confidence, source, and human-verification state, plus the design-ingestion entities (`cad_model`, `derived_view`, `manufacturing_feature`) at the front of the thread. The discipline that makes AI-in-the-loop manufacturing definition auditable and certifiable.
- [`schemas/`](schemas/) — machine-readable entity definitions (YAML).

**References:**
- [`standards/`](standards/) — Layer 1 briefs.
- [`reference/relationship-graph.yaml`](reference/relationship-graph.yaml) — machine-readable cross-entity links (409 edges; current as of 2026-05-31).
- [`implementations/`](implementations/) — reference mappings to Carbon, Fulcrum, and a template for future systems.
- [`reference/carbon-novelty-2026-04.md`](reference/carbon-novelty-2026-04.md) — latest Carbon scan findings.
- [`docs/department-head-briefing.md`](docs/department-head-briefing.md) — slide-shaped briefing for explaining the four-layer model, business-object audit history, queues, and the new department-head role.

## Contributing

A new contributor — human or agent — can reach any entity in ≤2 hops from here. Start with [`INDEX.md`](INDEX.md). To propose changes, consult the archetype walks first: if your change doesn't improve a representative part's decomposition, it's a filing-system change, not a canon change.

## Status

**Active and substantially complete.** The spec covers the contract-manufacturer layer end to end — 12 operational domains, **132 entities**, with the cross-cutting provenance & verification substrate in place as of 2026-05-31. What remains is a short, deliberate residual list, not unfinished core work.

**Composition coverage — the real completeness signal.** The ontology is measured by part-decomposition, not entity count (see *Organizing Principle*). Eight worked walks in [`reference/composition-archetypes.md`](reference/composition-archetypes.md) now decompose cleanly against the spec: the three core archetypes (single machined part; outside-processed part with full AS9102 FAI; multi-part assembly) plus five lifecycle walks — calibration recall, ECN mid-production, fixture full lifecycle, concurrent-revision production, and design ingestion / CAM programming. Walk 8 closes the front of the digital thread: model + drawing → structured features and requirements. A representative complex aerospace part can be walked from drawing callout to shipped cert package without hitting a gap.

**Entity & schema coverage.** 132 entities; 120 carry a standalone YAML schema. The remaining 12 are intentionally schema-less — external standards modeled but not re-authored here (`machine_event`/MTConnect, `qif_results_document`/QIF, `x12_856_asn`/X12, `usml_category`/ITAR), string/enum value types (`country`, `heat_number`, `spec_body`, `uns_number`), aliases (`fai_package`, `process_identification`), an embedded sub-object (`fai_form_1_header`), and one entity removed from canon (`item`, Decision 1.9).

**Machine-readable reference files are current** as of 2026-05-31: [`reference/entity-inventory.yaml`](reference/entity-inventory.yaml) holds all 132 entries and [`reference/relationship-graph.yaml`](reference/relationship-graph.yaml) holds 409 edges. No inventory/graph backlog remains.

**The provenance & verification substrate** (Decision 1.10) is the spec's epistemic layer and its most recent major addition. Every asserted fact — extracted, inferred, computed, or measured — carries how it was established, with what confidence, from what source, and whether a human has verified it (the `provenance` value-object mixin, plus `cad_model`, `derived_view`, `manufacturing_feature`, and `verification_event`). This is the precondition for putting AI in the loop on regulated work and the basis for the platform's **human-supported-AI** trajectory: AI runs the manufacturing-engineering loop; humans verify, judge, and own. Prose: [`extensions/provenance-and-verification.md`](extensions/provenance-and-verification.md).

**Remaining residuals — intentional and minor.** Tracked in full in [`reference/residual-gaps.md`](reference/residual-gaps.md):
- **SDR** (Supplier Deviation Request, AS9100 §8.7.2) and **RMA** standalone entities are deferred; both have partial coverage today (`deviation_waiver` and `shipment` with `direction=return` respectively).
- The Phase 2–4 + QMS entities are not yet cascaded into the UUID entity table in `extensions/uuid-discipline.md`.
- Staged field additions on `characteristic`, `part_specification`, `measurement_result`, and `operation` (specified in the provenance extension + Decision 1.10) land in their schemas in lockstep with implementation codegen, to avoid drift between spec and generated code.
- `nonconformance_report.status` keeps the MRB-disposition flow at the ontology layer; the 7-state AS9100 process flow lives in the quality system implementation. Both are valid at different layers (architectural note in `schemas/quality.yaml`).

## License

Apache 2.0. See [LICENSE](LICENSE).
