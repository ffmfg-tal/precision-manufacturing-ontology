# Precision Manufacturing Ontology

**The published data model for how an aerospace part gets made — from engineering drawing to certified, flight-ready hardware.**

Every precision part travels the same path: a customer releases a drawing, and that drawing has to become a physical object — machined, heat-treated, finished, inspected, certified, packaged, and shipped. At [Final Frontier Manufacturing](https://ffmfg.com) we call that path **[Print to Part](https://ffmfg.com/perspectives/print-to-part/)**. This repository is the precise, software-agnostic map of everything that path touches.

## Why This Exists

Modern aerospace runs on a *digital thread* — an unbroken chain of structured data that follows a part from design to delivery, so nothing is lost and everything is traceable. The major standards bodies (NIST and the MTConnect / QIF / STEP / X12 consortia) have defined that thread for the prime contractors at the top of the supply chain.

**The thread goes dark exactly where parts are actually made.** At the contract manufacturer — the shop that takes the drawing and produces the hardware — no one had published the underlying model. Every shop reinvents it privately, and information leaks out at every handoff between machining, outside processing, inspection, and certification.

This repo closes that gap. It is an **ontology**: a complete, precise vocabulary and data model — the shared language that lets software, machines, and AI agents describe a real manufactured part without ambiguity. Publishing it is itself the point. Most of the industry treats this knowledge as a private moat; we've written it down.

## How This Fits the Bigger Picture

This is one of three public documents describing what FFMFG is building:

| Document | What it covers |
|---|---|
| [**Print to Part**](https://ffmfg.com/perspectives/print-to-part/) | The macro thesis — why American precision manufacturing is reindustrializing, and the opportunity to close the Print-to-Part gap at scale. |
| [**The Platform**](https://ffmfg.com/perspectives/platform/) | The operating model — an engineering-led, AI-integrated manufacturing platform where *the factory is the product*. |
| [**The Manifesto**](https://ffmfg.com/perspectives/manifesto/) | The operating principles and culture behind the work. |

The Manifesto puts it plainly: *"data is the product, too."* **This repository is that data model made concrete** — the canonical layer the rest of the platform is built on. It's not a whitepaper promise; it's the actual schema, published and verifiable.

## What's In Here

A complete entity model for contract-manufacturing precision work:

- **132 entities** across **12 operational domains** — materials, quality, supply chain, inspection, assembly, tooling, change management, and more.
- **Machine-readable schemas** for every entity that needs one, in plain YAML.
- **Eight worked examples** that walk real parts — from a single machined part, to an outside-processed part with full first-article inspection, to a multi-part assembly — decomposing each one against the model end to end.
- **A provenance and verification layer** so every fact a system asserts carries how it was established, how confident it is, where it came from, and whether a human has checked it. This is the groundwork that makes it safe to put AI in the loop on regulated aerospace work — *AI runs the manufacturing-engineering loop; humans verify, judge, and own.*

You don't need to be an engineer to see the shape of it. Start with the [eight worked examples](reference/composition-archetypes.md): each one reads as a story of how a part is built, with every step grounded in the model.

## The Four Layers

The model is organized into four layers, from open standards up to the screens an operator touches:

| Layer | What it is | Status |
|---|---|---|
| **1 — Standards** | Existing open standards (STEP, MTConnect, QIF, X12) | Adopted as-is. See [`standards/`](standards/). |
| **2 — Manufacturer Data Model** | The entities that fill the gap the standards leave | **This repo's canon.** See [`extensions/`](extensions/) and [`schemas/`](schemas/). |
| **3 — Data Layer** | A real system that stores it | Reference mappings in [`implementations/`](implementations/). |
| **4 — Interface Layer** | How people and AI interact with it | Out of scope for this spec. |

## For Builders — Navigating the Repo

A new contributor, human or agent, can reach any entity in two hops from here.

**Start here:**
- [`INDEX.md`](INDEX.md) — every entity, alphabetical, one line each, linking to its schema, prose, and matrix row.
- [`reference/composition-archetypes.md`](reference/composition-archetypes.md) — the eight worked walks (three core archetypes plus five lifecycle walks: calibration recall, ECN mid-production, fixture lifecycle, concurrent-revision production, and design ingestion / CAM programming).
- [`reference/entity-matrix.md`](reference/entity-matrix.md) — every entity × every completeness dimension, classified.

**The canon (Layer 2):**
- [`reference/domain-map.md`](reference/domain-map.md) — the twelve operational domains and the manufacturer-layer gaps each one fills.
- [`extensions/`](extensions/) — per-gap specifications (material certs, outside processing, quality flowdowns, supplier approval, IP boundary, UUID discipline, provenance & verification, process engineering/NRE, tool room, packaging, change management, inspection/calibration).
- [`extensions/provenance-and-verification.md`](extensions/provenance-and-verification.md) — the cross-cutting epistemic substrate plus the design-ingestion entities (`cad_model`, `derived_view`, `manufacturing_feature`) at the front of the thread.
- [`schemas/`](schemas/) — machine-readable entity definitions (YAML).

**References:**
- [`standards/`](standards/) — Layer 1 briefs.
- [`reference/relationship-graph.yaml`](reference/relationship-graph.yaml) — machine-readable cross-entity links (409 edges; current as of 2026-05-31).
- [`implementations/`](implementations/) — reference mappings to Carbon, Fulcrum, and a template for future systems.
- [`reference/carbon-novelty-2026-04.md`](reference/carbon-novelty-2026-04.md) — latest Carbon scan findings.
- [`docs/department-head-briefing.md`](docs/department-head-briefing.md) — slide-shaped briefing on the four-layer model, audit history, and queues.

**The measuring stick:** the ontology is judged by *composition*, not by entity count. Success means any real complex aerospace part — machined, finished, assembled, inspected under AS9102, shipped with a full cert package — can be described by walking its decomposition tree against the spec. The domains are the filing system; the worked walks are the test. To propose a change, consult the walks first: if your change doesn't improve a representative part's decomposition, it's a filing-system change, not a canon change.

## Status

**Active and substantially complete.** The spec covers the contract-manufacturer layer end to end — 12 domains, 132 entities, with the provenance & verification substrate in place as of 2026-05-31. What remains is a short, deliberate residual list, not unfinished core work.

- **Composition coverage** — all eight worked walks decompose cleanly against the spec, including Walk 8 (design ingestion), which closes the front of the thread: model + drawing → structured features and requirements. A representative complex aerospace part can be walked from drawing callout to shipped cert package without hitting a gap.
- **Entity & schema coverage** — 132 entities; 120 carry a standalone YAML schema. The remaining 12 are intentionally schema-less: external standards modeled but not re-authored here (`machine_event`/MTConnect, `qif_results_document`/QIF, `x12_856_asn`/X12, `usml_category`/ITAR), string/enum value types (`country`, `heat_number`, `spec_body`, `uns_number`), aliases (`fai_package`, `process_identification`), an embedded sub-object (`fai_form_1_header`), and one entity removed from canon (`item`, Decision 1.9).
- **Machine-readable reference files are current** as of 2026-05-31: [`reference/entity-inventory.yaml`](reference/entity-inventory.yaml) holds all 132 entries and [`reference/relationship-graph.yaml`](reference/relationship-graph.yaml) holds 409 edges. No inventory or graph backlog remains.

**Remaining residuals — intentional and minor.** Tracked in full in [`reference/residual-gaps.md`](reference/residual-gaps.md):
- **SDR** (Supplier Deviation Request, AS9100 §8.7.2) and **RMA** standalone entities are deferred; both have partial coverage today (`deviation_waiver` and `shipment` with `direction=return` respectively).
- The Phase 2–4 + QMS entities are not yet cascaded into the UUID entity table in `extensions/uuid-discipline.md`.
- Staged field additions on `characteristic`, `part_specification`, `measurement_result`, and `operation` (specified in the provenance extension + Decision 1.10) land in their schemas in lockstep with implementation codegen, to avoid drift between spec and generated code.
- `nonconformance_report.status` keeps the MRB-disposition flow at the ontology layer; the 7-state AS9100 process flow lives in the quality system implementation. Both are valid at different layers (architectural note in `schemas/quality.yaml`).

## License

Apache 2.0. See [LICENSE](LICENSE).
