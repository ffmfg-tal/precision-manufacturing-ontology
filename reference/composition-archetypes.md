# Composition Archetypes

## Purpose

The ontology's success criterion is compositional: given a real complex aerospace part, can every node and edge of its decomposition be described unambiguously from the spec? Archetypes are worked examples that anchor that test. They serve three purposes:

1. **Generate the entity inventory.** Every node and edge on an archetype tree becomes a row in the entity matrix (`reference/entity-matrix.md`).
2. **Stress-test the ontology.** Gaps — unnamed entities, missing relationships, fuzzy state machines — surface when you try to walk a tree end-to-end.
3. **Onboard contributors.** A new reader grounds their understanding of the ontology by reading a walked archetype, not by scanning thirty extension docs in order.

Three archetypes cover the intended span. A is the spine. B adds outside processing and full AS9102 FAI. C adds assembly and quality-flowdown propagation.

---

## Archetype A — Single Machined Part, In-House Only

**Concrete example:** the manufacturer P/N MFG-7075-BRK-001, Rev B. Aluminum 7075-T7351 structural bracket machined from plate per AMS 4078. In-house 5-axis CNC only; no outside processing. Shipped with MTR + CoC + DFARS declaration. Basic FAI required (AS9102 Form 1 + Form 3, Form 2 N/A because no outside processes or multiple materials).

### Decomposition tree

(node types are in square brackets; edges are labeled)

- `[Part]` MFG-7075-BRK-001 Rev B
  - `specified_by` → `[Part Specification]` customer CAGE 98765 drawing 123-456 Rev B
  - `has_classification` → `[Export Classification]` EAR99 / not ITAR
  - `has_approval_state` → `[Customer Approval]` approved source; FAI on file
  - `composed_of` → `[Material]` 7075-T7351 plate, 1.000" thick
    - `specified_by` → `[Material Specification]` AMS 4078
      - `references` → `[UNS Number]` A97075
      - `has_condition` → `[Material Condition]` T7351
      - `has_spec_body` → `[Spec Body]` SAE
    - `instance_of` → `[Material Lot]` LOT-20250412-7075PL-14
      - `certified_by` → `[Certification Document]` MTR-XX-1234 (mill test report)
      - `certified_by` → `[Certification Document]` COC-YYY-5678 (certificate of conformance)
      - `certified_by` → `[Certification Document]` DFARS-DECL-ZZZ-901 (DFARS declaration)
      - `has_heat` → `[Heat Number]` H-20250115-A
      - `has_melt_country` → `[Country]` US
      - `verified_by` → `[PMI Event]` XRF at receiving
  - `produced_via` → `[Routing]` R-MFG-7075-BRK-001-v3
    - `contains_operation` → `[Operation]` OP-010 Saw cut blank
      - `performed_at` → `[Work Center]` Saw cell
      - `produces` → `[Machine Event]` MTConnect work order segment
    - `contains_operation` → `[Operation]` OP-020 5-axis rough
      - `performed_at` → `[Work Center]` Mazak 5-axis
      - `runs` → `[Process Specification]` internal programming procedure v7
      - `produces` → `[Machine Event]` MTConnect work order segment
    - `contains_operation` → `[Operation]` OP-030 5-axis finish
      - `performed_at` → `[Work Center]` Mazak 5-axis
      - `produces` → `[Machine Event]` MTConnect work order segment
    - `contains_operation` → `[Operation]` OP-040 Deburr + inspect
      - `performed_at` → `[Work Center]` Inspection bench
  - `inspected_under` → `[First Article Inspection]` FAI-MFG-7075-BRK-001-B
    - `includes_form` → `[AS9102 Form 1]` Part Number Accountability
    - `includes_form` → `[AS9102 Form 3]` Characteristic Accountability
      - `references` → `[QIF Results Document]` CMM results for serial 0001
        - `records` → `[Characteristic Measurement]` each KC
          - `measures` → `[Key Characteristic]` (per drawing)
  - `produces` → `[Serial Number]` MFG-7075-BRK-001-B-0001 (one unit this archetype)
  - `shipped_under` → `[Shipment]` SHP-20250420-001
    - `includes_document` → `[Certification Package]` assembled bundle
      - `contains` → `[MTR]`
      - `contains` → `[CoC]`
      - `contains` → `[DFARS Declaration]`
      - `contains` → `[FAI Package]`
    - `carries_ref` → `[X12 856 ASN]` with the production system Job UUID in REF*ZZ

### Entities introduced by Archetype A

Deduplicated list of every unique `[Entity]` node type in the tree above. This list seeds the entity inventory (Task 5).

- `[AS9102 Form 1]`
- `[AS9102 Form 3]`
- `[Certification Document]`
- `[Certification Package]`
- `[Characteristic Measurement]`
- `[CoC]`
- `[Country]`
- `[Customer Approval]`
- `[DFARS Declaration]`
- `[Export Classification]`
- `[FAI Package]`
- `[First Article Inspection]`
- `[Heat Number]`
- `[Key Characteristic]`
- `[Machine Event]`
- `[Material]`
- `[Material Condition]`
- `[Material Lot]`
- `[Material Specification]`
- `[MTR]`
- `[Operation]`
- `[Part]`
- `[Part Specification]`
- `[PMI Event]`
- `[Process Specification]`
- `[QIF Results Document]`
- `[Routing]`
- `[Serial Number]`
- `[Shipment]`
- `[Spec Body]`
- `[UNS Number]`
- `[Work Center]`
- `[X12 856 ASN]`

---

## Archetype B — Machined Part with Outside Processing and Full FAI

**Concrete example:** the manufacturer P/N MFG-TI64-FIT-007, Rev A. Titanium 6Al-4V structural fitting machined from AMS 4911 plate. In-house 5-axis CNC, then out to Nadcap-approved vendor for heat treat per AMS 2774, back in-house for rough finish, back out for passivation per AMS 2700, back in for final inspection and FAI. AS9102 Rev C three-form FAI. ITAR-controlled. Customer PO carries quality clauses QA-004, QA-007, QA-011, QA-013.

### Decomposition tree

(node types are in square brackets; edges are labeled)

- `[Part]` MFG-TI64-FIT-007 Rev A
  - `specified_by` → `[Part Specification]` customer CAGE 54321 drawing 987-654 Rev A
  - `has_classification` → `[Export Classification]` ITAR
    - `has_category` → `[USML Category]` Category VIII (aircraft and associated equipment)
  - `has_approval_state` → `[Customer Approval]` approved source; FAI pending
  - `composed_of` → `[Material]` Ti 6Al-4V plate, 0.750" thick
    - `specified_by` → `[Material Specification]` AMS 4911
      - `references` → `[UNS Number]` R56400
      - `has_condition` → `[Material Condition]` annealed
      - `has_spec_body` → `[Spec Body]` SAE
    - `instance_of` → `[Material Lot]` LOT-20260301-TI64-03
      - `certified_by` → `[Certification Document]` MTR-TI-9981 (mill test report)
      - `certified_by` → `[Certification Document]` COC-TI-9981 (certificate of conformance)
      - `certified_by` → `[Certification Document]` DFARS-DECL-TI-9981 (DFARS specialty metals declaration)
      - `has_heat` → `[Heat Number]` H-20260115-TI-B
      - `has_melt_country` → `[Country]` US
      - `verified_by` → `[PMI Event]` XRF at receiving
  - `produced_via` → `[Routing]` R-MFG-TI64-FIT-007-v2
    - `contains_operation` → `[Operation]` OP-010 Saw cut blank
      - `performed_at` → `[Work Center]` Saw cell
      - `produces` → `[Machine Event]` MTConnect work order segment
    - `contains_operation` → `[Operation]` OP-020 5-axis rough
      - `performed_at` → `[Work Center]` Mazak 5-axis
      - `runs` → `[Process Specification]` internal programming procedure v9
      - `produces` → `[Machine Event]` MTConnect work order segment
    - `contains_operation` → `[Outside Process Operation]` OP-030 Heat treat per AMS 2774
      - `has_vendor` → `[Supplier]` Acme Heat Treat (CAGE 0ABC1)
        - `approved_via` → `[Supplier Approval]` the manufacturer AVL entry for Acme HT
          - `scope_certified_by` → `[Nadcap Certification]` HT-prime cert no. NDCP-HT-1047
            - `has_nadcap_category` → `[Nadcap Category]` HT (Heat Treating)
          - `approved_for` → `[Process Specification]` AMS 2774
      - `runs` → `[Process Specification]` AMS 2774
        - `has_nadcap_category` → `[Nadcap Category]` HT (Heat Treating)
        - `has_spec_body` → `[Spec Body]` SAE
      - `issued_via` → `[Sub-Tier Purchase Order]` PO-20260315-AHT-042
        - `flows_down` → `[Quality Clause]` QA-004 (MTR required)
        - `flows_down` → `[Quality Clause]` QA-007 (subtier control)
        - `flows_down` → `[Quality Clause]` QA-011 (ITAR compliance)
        - `flows_down` → `[Quality Clause]` QA-013 (Nadcap for special processes)
      - `ships_out_via` → `[Shipment]` SHP-OUT-20260316-AHT (parts to Acme)
      - `results_in` → `[Process Certification]` Acme HT cert AHT-CERT-88421 for lot
      - `returns_with` → `[Shipment Inbound]` SHP-IN-20260322-AHT (parts + certs back to the shop)
        - `includes_document` → `[Process Certification]` Acme HT cert AHT-CERT-88421
    - `contains_operation` → `[Operation]` OP-040 Finish machining
      - `performed_at` → `[Work Center]` Mazak 5-axis
      - `runs` → `[Process Specification]` internal programming procedure v9
      - `produces` → `[Machine Event]` MTConnect work order segment
    - `contains_operation` → `[Outside Process Operation]` OP-050 Passivation per AMS 2700
      - `has_vendor` → `[Supplier]` Sierra Passivation (CAGE 3SPS2)
        - `approved_via` → `[Supplier Approval]` the manufacturer AVL entry for Sierra Passivation
          - `scope_certified_by` → `[Nadcap Certification]` CP-prime cert no. NDCP-CP-2318
            - `has_nadcap_category` → `[Nadcap Category]` CP (Chemical Processing)
          - `approved_for` → `[Process Specification]` AMS 2700
      - `runs` → `[Process Specification]` AMS 2700
        - `has_nadcap_category` → `[Nadcap Category]` CP (Chemical Processing)
        - `has_spec_body` → `[Spec Body]` SAE
      - `issued_via` → `[Sub-Tier Purchase Order]` PO-20260328-SPS-015
        - `flows_down` → `[Quality Clause]` QA-004 (MTR required)
        - `flows_down` → `[Quality Clause]` QA-007 (subtier control)
        - `flows_down` → `[Quality Clause]` QA-011 (ITAR compliance)
        - `flows_down` → `[Quality Clause]` QA-013 (Nadcap for special processes)
      - `ships_out_via` → `[Shipment]` SHP-OUT-20260329-SPS (parts to Sierra)
      - `results_in` → `[Process Certification]` Sierra passivation cert SPS-CERT-44910 for lot
      - `returns_with` → `[Shipment Inbound]` SHP-IN-20260403-SPS (parts + certs back to the shop)
        - `includes_document` → `[Process Certification]` Sierra passivation cert SPS-CERT-44910
    - `contains_operation` → `[Operation]` OP-060 Final inspect + FAI
      - `performed_at` → `[Work Center]` Inspection bench
      - `produces` → `[Machine Event]` CMM session record
  - `inspected_under` → `[First Article Inspection]` FAI-MFG-TI64-FIT-007-A
    - `includes_form` → `[AS9102 Form 1]` Part Number Accountability
      - `records` → `[FAI Form 1 Header]` part identity, supplier identity, FAI report number
    - `includes_form` → `[AS9102 Form 2]` Product Accountability
      - `records` → `[Material Identification]` Ti 6Al-4V plate per AMS 4911, heat H-20260115-TI-B
        - `references` → `[Material Specification]` AMS 4911
        - `references` → `[Certification Document]` MTR-TI-9981
      - `records` → `[Process Identification]` Heat treat anneal per AMS 2774
        - `references` → `[Process Specification]` AMS 2774
        - `references` → `[Process Certification]` Acme HT cert AHT-CERT-88421
      - `records` → `[Process Identification]` Passivation per AMS 2700
        - `references` → `[Process Specification]` AMS 2700
        - `references` → `[Process Certification]` Sierra passivation cert SPS-CERT-44910
    - `includes_form` → `[AS9102 Form 3]` Characteristic Accountability
      - `references` → `[QIF Results Document]` CMM results per serial
        - `records` → `[Characteristic Measurement]` each KC
          - `measures` → `[Key Characteristic]` (per drawing)
  - `produces` → `[Serial Number]` MFG-TI64-FIT-007-A-0001 through -0010 (ten units this archetype)
  - `shipped_under` → `[Shipment]` SHP-20260408-007
    - `includes_document` → `[Certification Package]` assembled bundle
      - `contains` → `[MTR]`
      - `contains` → `[CoC]`
      - `contains` → `[DFARS Declaration]`
      - `contains` → `[Process Certification]` Acme HT cert AHT-CERT-88421
      - `contains` → `[Process Certification]` Sierra passivation cert SPS-CERT-44910
      - `contains` → `[FAI Package]` three forms (Form 1, Form 2, Form 3)
      - `contains` → `[Export Compliance Declaration]` ITAR / USML Cat VIII declaration
    - `carries_ref` → `[X12 856 ASN]` with the production system Job UUID in REF*ZZ

### Flowdown origin chain

The quality clauses on each sub-tier PO originate at the customer PO. The chain is:

- `[Customer Purchase Order]` PO-CUST-20260210-TI64-007 — cites clauses QA-004, QA-007, QA-011, QA-013
  - `creates` → `[Job]` production job 26-0307 for the manufacturer-TI64-FIT-007 Rev A
    - `inherits_clauses` → `[Quality Clause]` QA-004
    - `inherits_clauses` → `[Quality Clause]` QA-007
    - `inherits_clauses` → `[Quality Clause]` QA-011
    - `inherits_clauses` → `[Quality Clause]` QA-013
    - `propagates_to` → `[Sub-Tier Purchase Order]` PO-20260315-AHT-042 (heat treat)
    - `propagates_to` → `[Sub-Tier Purchase Order]` PO-20260328-SPS-015 (passivation)

### Entities introduced by Archetype B (not in A)

Deduplicated list of every unique `[Entity]` node type used in the Archetype B tree above that is not already on the Archetype A list. This list extends the entity inventory.

- `[AS9102 Form 2]`
- `[Customer Purchase Order]`
- `[Export Compliance Declaration]`
- `[FAI Form 1 Header]`
- `[Job]`
- `[Material Identification]`
- `[Nadcap Category]`
- `[Nadcap Certification]`
- `[Outside Process Operation]`
- `[Process Certification]`
- `[Process Identification]`
- `[Quality Clause]`
- `[Shipment Inbound]`
- `[Sub-Tier Purchase Order]`
- `[Supplier]`
- `[Supplier Approval]`
- `[USML Category]`

---

## Archetype C — Multi-Part Assembly

**Concrete example:** the manufacturer P/N MFG-GMB-BKT-ASSY-012, Rev A. A three-piece gimbal mount bracket sub-assembly for a satellite attitude-control system. Consists of MFG-TI64-FIT-007 Rev A (the archetype B titanium fitting), MFG-AL7075-PLT-003 Rev A (a 7075-T7351 mounting plate, archetype-A-like), and MFG-17-4PH-SHAFT-005 Rev A (a 17-4 PH H1025 stainless pivot pin, archetype-A-like but simpler). Assembled in-house via torque-controlled bolt-up with a witnessed torque sequence. ITAR-controlled — the classification propagates up from the titanium fitting to the assembly. Customer requires assembly-level AS9102 Rev C three-form FAI. Customer PO PO-CUST-20260215-GMB-012 carries quality clauses QA-001, QA-002, QA-005, QA-010, QA-011, QA-013.

### Framing choice: `[Assembly]` as a distinct entity type

This archetype models the top of the tree as `[Assembly]` rather than as a recursive `[Part]` whose `composed_of` children are themselves `[Part]` nodes. Aerospace assemblies carry attributes that do not apply to individual parts — torque specs, fastener sequences, concentricity across the joined parts, witness-inspection signoff — and introducing them as fields on `[Part]` muddies the part definition. An `[Assembly]` node with its own characteristic set, its own `[Assembly Operation]` children, and `contains_part` edges to the constituent `[Part]` nodes is structurally cleaner and lets an agent walking the tree distinguish "this node is a joined-up product of other parts" from "this node is a machined article from raw material." The constituent parts remain `[Part]` nodes with their archetype A or B decomposition intact. The assembly's identity, FAI, certification package, serial record, and shipment all hang off the `[Assembly]` node.

The alternative framing — recursive `[Part]` with `composed_of` children that are themselves `[Part]` nodes — is flagged below as an open question for the entity-matrix task to resolve. A single `[Part]` type with a boolean `is_assembly` flag or a subtype relation would be equivalent in expressive power and would avoid introducing a new top-level entity type; the matrix review should weigh the ergonomic clarity of a distinct `[Assembly]` against the ontological parsimony of a recursive `[Part]`.

### Decomposition tree

(node types are in square brackets; edges are labeled)

- `[Assembly]` MFG-GMB-BKT-ASSY-012 Rev A
  - `specified_by` → `[Part Specification]` customer CAGE 54321 drawing 987-700 Rev A (assembly drawing)
  - `has_classification` → `[Export Classification]` ITAR
    - `has_category` → `[USML Category]` Category XV (spacecraft and related articles)
    - `inherits_classification` → `[Export Classification]` ITAR on MFG-TI64-FIT-007 Rev A (child `[Part]`)
  - `has_approval_state` → `[Customer Approval]` approved source; assembly FAI pending
  - `contains_part` → `[Part]` MFG-TI64-FIT-007 Rev A — titanium fitting, qty 1
    - `has_classification` → `[Export Classification]` ITAR
      - `has_category` → `[USML Category]` Category VIII
    - `composed_of` → `[Material]` Ti 6Al-4V plate per AMS 4911
    - `produced_via` → `[Routing]` R-MFG-TI64-FIT-007-v2
    - `inspected_under` → `[First Article Inspection]` FAI-MFG-TI64-FIT-007-A
    - `consumes_serial` → `[Serial Number]` MFG-TI64-FIT-007-A-0003 (this build consumes one unit)
    - (full decomposition in Archetype B; only the top-level edges are shown here)
  - `contains_part` → `[Part]` MFG-AL7075-PLT-003 Rev A — 7075-T7351 mounting plate, qty 1
    - `has_classification` → `[Export Classification]` EAR99 / not ITAR
    - `composed_of` → `[Material]` 7075-T7351 plate per AMS 4078
    - `produced_via` → `[Routing]` R-MFG-AL7075-PLT-003-v1 (in-house 3-axis + deburr; archetype-A-like)
    - `inspected_under` → `[First Article Inspection]` FAI-MFG-AL7075-PLT-003-A (basic: Forms 1 + 3)
    - `consumes_serial` → `[Serial Number]` MFG-AL7075-PLT-003-A-0007 (this build consumes one unit)
    - (decomposition is substantively identical to Archetype A — see there for full walk)
  - `contains_part` → `[Part]` MFG-17-4PH-SHAFT-005 Rev A — 17-4 PH H1025 pivot pin, qty 2
    - `specified_by` → `[Part Specification]` customer CAGE 54321 drawing 987-701 Rev A
    - `has_classification` → `[Export Classification]` EAR99 / not ITAR
    - `has_approval_state` → `[Customer Approval]` approved source; FAI on file
    - `composed_of` → `[Material]` 17-4 PH stainless bar, 0.500" dia
      - `specified_by` → `[Material Specification]` AMS 5643
        - `references` → `[UNS Number]` S17400
        - `has_condition` → `[Material Condition]` H1025
        - `has_spec_body` → `[Spec Body]` SAE
      - `instance_of` → `[Material Lot]` LOT-20260220-174PH-11
        - `certified_by` → `[Certification Document]` MTR-174-5521
        - `certified_by` → `[Certification Document]` COC-174-5521
        - `certified_by` → `[Certification Document]` DFARS-DECL-174-5521
        - `has_heat` → `[Heat Number]` H-20260108-174-C
        - `has_melt_country` → `[Country]` US
        - `verified_by` → `[PMI Event]` XRF at receiving
    - `produced_via` → `[Routing]` R-MFG-17-4PH-SHAFT-005-v1
      - `contains_operation` → `[Operation]` OP-010 Saw cut blank
        - `performed_at` → `[Work Center]` Saw cell
        - `produces` → `[Machine Event]` MTConnect work order segment
      - `contains_operation` → `[Operation]` OP-020 Turn + mill complete
        - `performed_at` → `[Work Center]` Mazak Integrex mill-turn
        - `runs` → `[Process Specification]` internal programming procedure v4
        - `produces` → `[Machine Event]` MTConnect work order segment
      - `contains_operation` → `[Operation]` OP-030 Deburr + inspect
        - `performed_at` → `[Work Center]` Inspection bench
    - `inspected_under` → `[First Article Inspection]` FAI-MFG-17-4PH-SHAFT-005-A (basic: Forms 1 + 3; no outside processing)
      - `includes_form` → `[AS9102 Form 1]` Part Number Accountability
      - `includes_form` → `[AS9102 Form 3]` Characteristic Accountability
    - `consumes_serial` → `[Serial Number]` MFG-17-4PH-SHAFT-005-A-0014 and -0015 (two units consumed)
  - `assembled_via` → `[Assembly Routing]` R-MFG-GMB-BKT-ASSY-012-v1
    - `contains_operation` → `[Assembly Operation]` OP-A010 Kit + stage sub-components
      - `performed_at` → `[Work Center]` Assembly cell
      - `produces` → `[Kit Record]` KIT-20260415-GMB-012-001 (sub-component serials binned to assembly S/N)
    - `contains_operation` → `[Assembly Operation]` OP-A020 Bolt-up per NASM 33540
      - `performed_at` → `[Work Center]` Assembly cell
      - `runs` → `[Process Specification]` NASM 33540 (safetying general practices)
        - `has_spec_body` → `[Spec Body]` NASM
      - `uses_fastener` → `[Fastener]` NAS1351-3-12 socket head cap screw, qty 6 (A286, silver-plated)
        - `certified_by` → `[Certification Document]` fastener CoC from approved distributor
      - `has_torque_spec` → `[Torque Specification]` 55 +/- 3 in-lb per AS5272 Table IV-A, lubricated
        - `references` → `[Process Specification]` AS5272 (torque values for threaded fasteners)
      - `has_torque_sequence` → `[Torque Sequence]` cross-pattern two-pass: 50% preload (all six), then final torque (all six)
      - `witnessed_by` → `[Witness Inspection]` WI-20260416-GMB-012-001
        - `performed_by` → `[Quality Engineer]` QE signoff (named)
        - `records` → `[Torque Readings Record]` TRR-20260416-GMB-012-001 (six readings, all within spec)
    - `contains_operation` → `[Assembly Operation]` OP-A030 Final assembly inspect + concentricity check
      - `performed_at` → `[Work Center]` Inspection bench
      - `produces` → `[QIF Results Document]` assembly-level CMM results (concentricity across pin-fitting-plate axis)
  - `inspected_under` → `[First Article Inspection]` FAI-MFG-GMB-BKT-ASSY-012-A (assembly-level, full three forms)
    - `includes_form` → `[AS9102 Form 1]` Part Number Accountability
      - `records` → `[FAI Form 1 Header]` assembly P/N, the manufacturer CAGE, assembly drawing number, FAI report number
    - `includes_form` → `[AS9102 Form 2]` Product Accountability
      - `records` → `[Material Identification]` Ti 6Al-4V fitting — references child `[Part]` MFG-TI64-FIT-007 Rev A and its Form 2 entries
      - `records` → `[Material Identification]` 7075-T7351 plate — references child `[Part]` MFG-AL7075-PLT-003 Rev A and its Form 1/3 FAI
      - `records` → `[Material Identification]` 17-4 PH H1025 pin — references child `[Part]` MFG-17-4PH-SHAFT-005 Rev A and its Form 1/3 FAI
      - `records` → `[Process Identification]` Assembly bolt-up per NASM 33540 with torque per AS5272
        - `references` → `[Process Specification]` NASM 33540
        - `references` → `[Process Specification]` AS5272
        - `references` → `[Torque Readings Record]` TRR-20260416-GMB-012-001
      - `includes_fai` → `[First Article Inspection]` FAI-MFG-TI64-FIT-007-A (child part FAI rolled into Form 2)
      - `includes_fai` → `[First Article Inspection]` FAI-MFG-AL7075-PLT-003-A
      - `includes_fai` → `[First Article Inspection]` FAI-MFG-17-4PH-SHAFT-005-A
    - `includes_form` → `[AS9102 Form 3]` Characteristic Accountability
      - `references` → `[QIF Results Document]` assembly-level CMM results
        - `records` → `[Characteristic Measurement]` concentricity across pin-fitting-plate axis
          - `measures` → `[Key Characteristic]` assembly concentricity (per assembly drawing)
        - `records` → `[Characteristic Measurement]` fastener torque values (six readings)
          - `measures` → `[Key Characteristic]` bolt preload (per assembly drawing)
  - `produces` → `[Serial Number]` MFG-GMB-BKT-ASSY-012-A-0001 (one assembly serial for this build)
  - `shipped_under` → `[Shipment]` SHP-20260420-012
    - `includes_document` → `[Certification Package]` assembly-level aggregated bundle (child cert packages referenced below aggregate their own documents — MTR, CoC, DFARS declarations, process certs, Form 1/2/3 FAI, and ITAR/USML declarations — per their respective archetypes. The assembly cert package includes these child packages by reference rather than re-enumerating their contents.)
      - `contains` → `[Certification Package]` child cert package for the manufacturer-TI64-FIT-007 Rev A
      - `contains` → `[Certification Package]` child cert package for the manufacturer-AL7075-PLT-003 Rev A
      - `contains` → `[Certification Package]` child cert package for the manufacturer-17-4PH-SHAFT-005 Rev A
      - `contains` → `[FAI Package]` assembly-level FAI — Form 1, Form 2, Form 3
      - `contains` → `[Torque Readings Record]` TRR-20260416-GMB-012-001
      - `contains` → `[Certification Document]` witness inspection signoff WI-20260416-GMB-012-001
      - `contains` → `[Certification Document]` fastener CoC for NAS1351-3-12 lot
      - `contains` → `[Export Compliance Declaration]` assembly-level ITAR / USML Cat XV declaration
    - `carries_ref` → `[X12 856 ASN]` with the production system Job UUID in REF*ZZ

### Flowdown origin chain (assembly level)

The assembly's customer PO carries its own clause set. Sub-component POs that existed at the part level (the Ti fitting's sub-tier heat-treat and passivation POs in Archetype B) remain rooted in the child part's job; they do not re-flow from the assembly job. The assembly job inherits its clauses from the assembly PO and propagates nothing downstream because assembly itself is in-house.

- `[Customer Purchase Order]` PO-CUST-20260215-GMB-012 — cites clauses QA-001, QA-002, QA-005, QA-010, QA-011, QA-013
  - `creates` → `[Job]` production job 26-0419 for the manufacturer-GMB-BKT-ASSY-012 Rev A
    - `inherits_clauses` → `[Quality Clause]` QA-001 (QMS / AS9100)
    - `inherits_clauses` → `[Quality Clause]` QA-002 (FAI per AS9102)
    - `inherits_clauses` → `[Quality Clause]` QA-005 (right of access / source inspection)
    - `inherits_clauses` → `[Quality Clause]` QA-010 (record retention)
    - `inherits_clauses` → `[Quality Clause]` QA-011 (ITAR)
    - `inherits_clauses` → `[Quality Clause]` QA-013 (Nadcap for special processes — satisfied at child part level by the manufacturer-TI64-FIT-007's sub-tier POs)
    - `references_child_job` → `[Job]` production job 26-0307 for the manufacturer-TI64-FIT-007 Rev A (Archetype B's job)
    - `references_child_job` → `[Job]` child job for the manufacturer-AL7075-PLT-003 Rev A
    - `references_child_job` → `[Job]` child job for the manufacturer-17-4PH-SHAFT-005 Rev A

### Entities introduced by Archetype C (not in A or B)

Deduplicated list of every unique `[Entity]` node type used in the Archetype C tree above that is not already on the Archetype A or Archetype B list. This list extends the entity inventory.

- `[Assembly]`
- `[Assembly Operation]`
- `[Assembly Routing]`
- `[Fastener]`
- `[Kit Record]`
- `[Quality Engineer]`
- `[Torque Readings Record]`
- `[Torque Sequence]`
- `[Torque Specification]`
- `[Witness Inspection]`

### Open questions introduced by Archetype C (for the entity-matrix task)

- **`[Assembly]` vs recursive `[Part]`.** Archetype C uses `[Assembly]` as a distinct entity type. The alternative — treating an assembly as a `[Part]` whose `composed_of` children are `[Part]` nodes, with an `is_assembly` flag or subtype relation — is equivalently expressive and would avoid introducing a new top-level type. The matrix task should pick one and reconcile references across extensions.
- **`[Assembly Routing]` vs reuse of `[Routing]`.** A routing that contains `[Assembly Operation]` nodes may not need a distinct type from a routing that contains `[Operation]` nodes; the type could collapse if `[Operation]` and `[Assembly Operation]` share a parent type.
- **`[Quality Engineer]` vs a generic `[Person]` or `[User]` role entity.** Witness inspection signoff requires a typed identity. Whether that identity is its own entity type, a role on a general person entity, or an attribute on `[Witness Inspection]` is unresolved.
- **`[Fastener]` scope.** Fasteners are COTS purchased items with their own cert chain (MTR, CoC). Whether a `[Fastener]` is a specialization of `[Part]`, a specialization of `[Material]`, or its own type depends on whether the manufacturer treats fasteners as inventory items or as consumables rolled into an assembly op. The matrix should decide.
- **Child cert package nesting.** Archetype C nests `[Certification Package]` inside `[Certification Package]`. Whether the assembly-level cert package is literally a bundle of child cert packages, or a flat aggregation of the underlying documents with references back to child parts, is an implementation choice the matrix should call out.

---

## Walk 4 — Calibration Recall

**Scenario:** Inspector Maria uses ring gauge CAL-0042 (`tool_gauge`, `calibration_status: Current`) on 2026-04-10 to perform `Final_Inspection` on SN-003 and SN-004 of Job J-2026-189. On 2026-04-15 she notices the gauge face is chipped. Quality quarantines it. The shop must determine whether the measurements from 2026-04-10 are still valid.

**What this walk tests:** the calibration-recall traceability chain. Without `calibration_record` (immutable evidence of the last calibration event) and `inspection_event` (the UUID envelope linking inspector + gauge + session), there is no systematic way to identify which measurement results are at risk and bound the review scope.

### Step-by-step walkthrough

**Step 1 — Quarantine the gauge.**

Entity: `tool_gauge` (asset_tag: CAL-0042)

Data at this node:
- `calibration_status`: Current → **Quarantined** (update)
- `location`: "QC Lab — Quarantine" (update)
- `current_calibration_id`: FK → calibration_record (unchanged — last valid calibration still on record)
- `calibration_due_date`: 2026-06-15 (unchanged)

Without `tool_gauge.calibration_status`, the shop has no single field to flip that signals "this instrument is out of service" to every downstream query; the quarantine would have to be enforced by convention rather than data.

**Step 2 — Retrieve the last calibration record.**

Entity: `calibration_record` (query: `tool_gauge_id = CAL-0042`, sort by `calibration_date` desc)

Data at the most-recent record:
- `calibration_date`: 2026-03-15
- `result`: Pass
- `as_found_in_tolerance`: true
- `certificate_number`: CAL-CERT-2026-0315-042
- `performed_by`: "Acme Calibration Lab"
- `lab_accreditation`: "A2LA accreditation #4821"

Finding: the gauge was in tolerance as of 2026-03-15. All measurements taken between 2026-03-15 and the quarantine event on 2026-04-15 were performed while the gauge was within its calibration interval.

Without `calibration_record`, determining the "last known-good" calibration date requires digging through paper logs; the query cannot be automated.

**Step 3 — Find all inspection sessions using this gauge in the at-risk window.**

Entity: `inspection_event` (query: records where the gauge appears in session metadata between 2026-03-15 and 2026-04-15)

Matching record — `inspection_event` IE-0089:
- `event_type`: Final_Inspection
- `job_id`: J-2026-189
- `serial_ids`: [SN-003, SN-004]
- `inspection_date`: 2026-04-10
- `inspector_id`: Maria (user FK)
- `result_summary`: Pass

Without `inspection_event` as a first-class UUID envelope, there is no indexed record linking a gauge to a group of measurement results; the recall scope cannot be determined by query.

**Step 4 — Retrieve all measurements from that session.**

Entity: `pmi_event` (query: `inspection_event_id = IE-0089`)

14 records returned. Each record holds:
- `technique`: ring_gauge (OD measurement) or depth_gauge (depth measurement)
- `element_readings[]`: measured value, nominal, deviation, pass/fail
- `instrument_id`: FK → tool_gauge.id

Result: 14 measurements total — mix of OD and depth readings on SN-003 and SN-004.

**Step 5 — Check which measurements are key characteristics.**

Entity: `key_characteristic` (query: records where `balloon_number` matches any `pmi_event` in IE-0089)

2 of the 14 measurements correspond to KCs:
- KC-7832-001-K01: depth dimension (depth gauge, not ring gauge)
- KC-7832-001-K02: depth dimension (depth gauge, not ring gauge)

Both KC measurements used the depth gauge, not CAL-0042. The chipped ring gauge was used only for OD measurements (7 of 14).

**Step 6 — Engineering disposition.**

The quality engineer reviews the damage pattern: the chipped face on CAL-0042 affects the OD-measurement contact faces only. The 2 KC measurements are depth measurements, unaffected. The 7 OD measurements used CAL-0042, but OD dimensions on 7832-001 are not KC-designated; tolerances are standard bilateral and the gauge was within calibration.

Disposition: OD measurements remain valid. No recall action required.

**Step 7 — Document in nonconformance report.**

Entity: `nonconformance_report` (NCR-2026-0041)

Key fields:
- `subject_id/subject_type`: tool_gauge / CAL-0042
- `description`: "Gauge CAL-0042 found with chipped ring face on 2026-04-15."
- `mrb_disposition`: Use_As_Is (for the measured parts — no rework)
- `mrb_justification`: "Measurements IE-0089 reviewed; no impact — damage pattern does not affect measurement axes used for KC or critical dimensions. OD readings remain valid: gauge was within calibration interval and damage does not affect OD-measurement geometry for the tolerances specified."
- `status`: Closed
- Free-text note: "Gauge CAL-0042 quarantined. Measurements IE-0089 reviewed; no impact — damage pattern does not affect measurement axes used."

**Step 8 — Record maintenance event.**

Entity: `tool_maintenance_event`

Key fields:
- `asset_type`: Gauge
- `tool_gauge_id`: FK → CAL-0042
- `event_type`: Inspect
- `result`: RetiredAfterService
- `description`: "Chip on ring face deemed unrepairable. Gauge retired."
- `event_date`: 2026-04-16

**Step 9 — Retire the gauge.**

Entity: `tool_gauge` (CAL-0042)

Update:
- `calibration_status`: Quarantined → **Retired**
- `location`: "QC Lab — Retired Instruments"

### Digital thread break analysis

If `inspection_event` did not exist as a UUID envelope, Step 3 would require manual cross-reference of paper logs to link a gauge serial number to a set of measurement results. If `calibration_record` did not exist as a queryable entity, Step 2 would require reviewing physical calibration certificates to establish the last in-tolerance date. If `key_characteristic` balloon references were not recorded in `pmi_event`, Step 5 would require manual review of each measurement against the drawing to assess KC impact.

### Entity touchpoints

`tool_gauge` · `calibration_record` · `inspection_event` · `pmi_event` · `key_characteristic` · `nonconformance_report` · `tool_maintenance_event`

---

## Walk 5 — ECN Mid-Production

**Scenario:** Engineering issues ECN-2026-0042 for part 7832-001: drawing changes from Rev B to Rev C (tightened tolerance on one bore). At the time of approval (2026-04-20), 3 jobs are running against Rev B: Job J-2026-180 (qty 5, 2 complete), J-2026-185 (qty 3, 0 complete), J-2026-192 (qty 10, 7 complete). ECN effective date: 2026-05-01 for new orders; existing open orders may complete Rev B.

**What this walk tests:** the change management chain across `engineering_change_notice`, `part_specification`, `routing`, `nre_package`, and in-flight jobs. Without `engineering_change_notice` as a first-class entity, the audit trail linking a drawing revision change to in-flight job dispositions cannot be queried.

### Step-by-step walkthrough

**Step 1 — Create and approve the ECN.**

Entity: `engineering_change_notice` (ECN-2026-0042)

Data at this node:
- `ecn_number`: ECN-2026-0042
- `change_type`: Drawing
- `status`: Draft → **Approved** (on 2026-04-20)
- `affected_part_id`: FK → part 7832-001
- `old_part_specification_id`: FK → part_specification (7832-001 Rev B)
- `new_part_specification_id`: FK → part_specification (7832-001 Rev C — being created, see Step 2)
- `effective_date`: 2026-05-01
- `customer_notification_required`: true
- `customer_approval_required`: false (drawing change within design authority)

**Step 2 — Create the new part_specification; preserve the old.**

Entity: `part_specification` (two records)

New record — 7832-001 Rev C:
- `drawing_number`: 7832-001
- `revision`: C
- `cage_code`: customer CAGE
- `pdf_file_ref`: path to new PDF
- Status: active

Old record — 7832-001 Rev B:
- Not deleted; retained as historical record
- `routing_revision_status` on its associated routing: Released → **Obsolete** (effective 2026-05-01)

Without `part_specification` as a versioned, immutable record distinct from the part entity, there is no place to anchor "the drawing revision in force on this job"; jobs carry only a `part_id` and the question "which revision was produced?" has no queryable answer.

**Step 3 — Transition routings.**

Entity: `routing` (two records for 7832-001)

Rev B routing:
- `status`: Released → **Obsolete** (effective 2026-05-01)

Rev C routing (new record):
- `status`: Draft → **Released** (concurrent with ECN approval)
- `routing_revision`: Rev C
- Contains updated operations reflecting the tightened tolerance bore (inspection step updated)

**Step 4 — Create new NRE package for Rev C.**

Entity: `nre_package`

New record:
- `part_id`: FK → 7832-001
- `part_specification_id`: FK → part_specification Rev C (immutable lock)
- `routing_id`: FK → routing Rev C (immutable lock)
- `status`: Draft (NC programs, setup sheets, CMM program updates needed before Approved)

Without `nre_package` as the revision-lock envelope, there is no single record confirming "all process engineering documents are current to Rev C and approved for production."

**Step 5 — Disposition in-flight jobs.**

**Job J-2026-180** (qty 5, 2 complete):
- Customer notified; customer approves completing remaining 3 under Rev B.
- No `deviation_waiver` required: existing in-flight job under original PO, customer explicitly approved.
- Job continues under original routing Rev B. No job record changes.

**Job J-2026-185** (qty 3, 0 complete):
- No work started. Job cancelled; reissued as new Job J-2026-198 against routing Rev C.
- Original job: `status` → Cancelled.
- New job J-2026-198: `routing_id` → routing Rev C; `part_specification_id` → Rev C.

**Job J-2026-192** (qty 10, 7 complete):
- 7 already-complete units measured against the tightened Rev C bore tolerance.
- 6 of 7 pass the new tolerance.
- 1 of 7 fails the new tolerance.
- `nonconformance_report` opened for the 1 out-of-tolerance unit.
- `deviation_waiver` (WAI-2026-0005) created for that 1 unit (see Step 6).
- Remaining 3 units completed under routing Rev C.

**Step 6 — Create deviation waiver for the out-of-tolerance unit.**

Entity: `deviation_waiver` (WAI-2026-0005)

Key fields:
- `document_type`: Waiver
- `dw_number`: WAI-2026-0005
- `part_id`: FK → 7832-001
- `part_specification_id`: FK → 7832-001 Rev C
- `serial_id`: FK → the 1 failing serial number
- `quantity_affected`: 1
- `ncr_id`: FK → nonconformance_report for the out-of-tolerance bore
- `nonconformance_description`: "Bore diameter 0.0003 outside new Rev C tolerance; within prior Rev B tolerance."
- `proposed_disposition`: "Use as-is — customer functional analysis confirms margin adequate for intended bore application."
- `status`: Draft → **Submitted** → **Approved**
- `approved_by`: "John Smith, SpaceCo DER-0042"
- `approved_date`: 2026-04-28

Without `deviation_waiver`, the authorization from SpaceCo's DER has no data entity to anchor to; it exists only as an email or paper document with no FK link to the affected serial number and NCR.

**Step 7 — Release the ECN.**

Entity: `engineering_change_notice` (ECN-2026-0042)

Update:
- `status`: Approved → **Released** (after 2026-05-01 effective date passes and all in-flight job dispositions are documented)

### Digital thread break analysis

If `engineering_change_notice` did not link `old_part_specification_id` and `new_part_specification_id` with typed FKs, the change record would be a text field describing what changed rather than a queryable link. Finding all jobs impacted by the revision change would require a full-text search rather than a FK join. If `nre_package` did not lock `routing_id` immutably, an operator could inadvertently run Rev B NC programs against a Rev C job without any schema-layer guard.

### Entity touchpoints

`engineering_change_notice` · `part_specification` · `routing` · `nre_package` · `job` · `work_order` · `deviation_waiver` · `nonconformance_report` · `customer_purchase_order`

---

## Walk 6 — Fixture Full Lifecycle

**Scenario:** FFMFG designs a dedicated soft-jaw fixture for part 7832-001 Op 10 (5-axis rough mill). The fixture has its own part number FIX-7832-001-01, goes through internal manufacture, qualification prove-out, active production use, a maintenance event, and eventual retirement.

**What this walk tests:** the fixture lifecycle chain across `part` (as fixture), `routing`, `job`, `fixture_use`, `nc_program`, `prove_out_record`, `tool_room_allocation`, `tool_maintenance_event`. Without `fixture_use` as a first-class join entity, the link between a routing operation and the fixture required to run it has no queryable representation.

### Step-by-step walkthrough

**Step 1 — Register the fixture in the item master.**

Entity: `part` (FIX-7832-001-01, Rev A)

Key fields:
- `part_number`: FIX-7832-001-01
- `revision`: A
- `part_kind`: fixture
- `fai_status`: Not_Required (internal tooling; not a customer-deliverable part)
- `itar_controlled`: false
- `active`: true

**Step 2 — Create the fixture drawing.**

Entity: `part_specification` (FIX-7832-001-01 Rev A)

Key fields:
- `drawing_number`: FIX-7832-001-01
- `revision`: A
- `pdf_file_ref`: path to fixture drawing PDF
- `cage_code`: FFMFG internal CAGE

**Step 3 — Create the manufacturing routing for the fixture.**

Entity: `routing` (FIX-7832-001-01 Rev A)

Key fields:
- `routing_revision`: A
- `status`: Released
- Operations:
  - Op 10: MILL — 3-axis mill the jaw profiles (Work Center: Haas VF-3)
  - Op 20: INSPECT — CMM verify jaw geometry (Work Center: Zeiss Contura)
  - Op 30: CLEAN — deburr and clean

**Step 4 — Issue a job to build the fixture.**

Entity: `job` (J-2026-155)

Key fields:
- `subject_id`: FK → part FIX-7832-001-01
- `subject_type`: part
- `routing_id`: FK → fixture routing Rev A
- `quantity`: 1
- `job_type`: stock build (internal, not for a customer PO)

**Step 5 — Execute work orders and inspection.**

Entities: `work_order` (three records, one per operation)

Op 10 (MILL): machined; `status` → Complete.
Op 20 (INSPECT): CMM inspection of jaw geometry.

Entity: `inspection_event` (IE-0101)

Key fields:
- `event_type`: Final_Inspection
- `job_id`: FK → J-2026-155
- `serial_ids`: [FIX-SN-001]
- `result_summary`: Pass — jaw geometry within tolerance

Op 30 (CLEAN): deburr and clean; `status` → Complete.

Job J-2026-155: `status` → Complete.

**Step 6 — Create the fixture_use record linking the fixture to 7832-001 Op 10.**

Entity: `fixture_use`

Key fields:
- `operation_id`: FK → operation Op 10 in routing for 7832-001 Rev C
- `fixture_part_id`: FK → part FIX-7832-001-01
- `description`: "primary workholding — soft jaw set for 5-axis rough mill"
- `required`: true

Without `fixture_use`, the relationship between the operation and the required fixture exists only in setup sheet prose. A scheduler or tool crib operator cannot query "which fixture does Op 10 of job J-2026-189 need?" by FK.

**Step 7 — NC program references the fixture.**

Entity: `nc_program` (7832-001-OP10-MAZAK1)

Key fields:
- `operation_id`: FK → Op 10 of 7832-001 routing
- `work_center_id`: FK → Mazak 5-axis
- `status`: Draft → Released (after prove-out)
- `program_number`: 7832-001-OP10-MAZAK1

The associated `setup_sheet` shows FIX-7832-001-01 as the primary fixture; the `tooling_list` specifies tools needed with the fixture installed.

**Step 8 — Prove-out record: first part machined with the fixture.**

Entity: `prove_out_record`

Key fields:
- `nc_program_id`: FK → 7832-001-OP10-MAZAK1
- `work_center_id`: FK → Mazak 5-axis
- `operator_id`: FK → user (machinist)
- `prove_out_date`: 2026-03-20
- `first_piece_result`: Pass
- `approved_by_id`: FK → user (engineering)
- `approved_date`: 2026-03-21

NC program status advances: Draft → Proven → Released.

**Step 9 — Fixture checked out for production job.**

Entity: `tool_room_allocation` (for Job J-2026-189)

Key fields:
- `asset_part_id`: FK → part FIX-7832-001-01
- `job_id`: FK → J-2026-189
- `operation_id`: FK → Op 10
- `checked_out_by_id`: FK → user (operator Alex)
- `checked_out_date`: 2026-04-08T07:30:00Z
- `returned_date`: 2026-04-10T15:45:00Z
- `status`: CheckedOut → **Returned**

**Step 10 — Scheduled maintenance inspection after 50 uses.**

Entity: `tool_maintenance_event`

Key fields:
- `asset_type`: Fixture
- `asset_part_id`: FK → FIX-7832-001-01
- `event_type`: Inspect
- `event_date`: 2026-04-20
- `result`: Pass
- `description`: "Jaw geometry re-verified after 50 uses; within tolerance per CMM check."

**Step 11 — Wear-out: jaw worn beyond regrind limit after 200 uses.**

Entity: `tool_maintenance_event` (second record)

Key fields:
- `asset_type`: Fixture
- `asset_part_id`: FK → FIX-7832-001-01
- `event_type`: Inspect
- `result`: Fail
- `description`: "Jaw geometry outside tolerance after 200 production uses. Jaw worn beyond regrind limit. Retire."
- `event_date`: 2026-09-15

**Step 12 — Retire the fixture.**

Entity: `part` (FIX-7832-001-01)

Update:
- `active`: false

The `fixture_use` record linking Op 10 → FIX-7832-001-01 remains in place as historical record; a replacement fixture (FIX-7832-001-02) would create a new `fixture_use` or the existing one's `fixture_part_id` would be updated to point to the replacement part.

### Digital thread break analysis

Without `fixture_use`, there is no queryable link between Op 10 of job J-2026-189 and FIX-7832-001-01. Without `tool_room_allocation`, there is no audit trail of which jobs used the fixture and when it was returned, making it impossible to systematically trace "which jobs ran with this fixture between its last inspection and retirement?" Without `prove_out_record`, the NC program's transition from Draft to Released has no immutable evidence record; prove-out approval exists only as a signoff in the shop floor log.

### Entity touchpoints

`part` (fixture) · `part_specification` · `routing` · `job` · `work_order` · `inspection_event` · `fixture_use` · `nc_program` · `setup_sheet` · `prove_out_record` · `tool_room_allocation` · `tool_maintenance_event`

---

## Walk 7 — Concurrent Revision Production

**Scenario:** FFMFG has two open customer purchase orders for part 7832-001: CPO-101 ordered Rev B (legacy, partially shipped — 15 of 20 shipped), CPO-107 ordered Rev C (new order, 10 pieces). Rev C FAI has not been completed. Both are in production simultaneously.

**What this walk tests:** concurrent multi-revision production, where two jobs for the same part number run simultaneously against different revisions, each with its own routing, NRE package, and FAI status. Without separate `part` records per revision (or a revision-disambiguating FK on `job`), the production system cannot prevent Rev B NC programs from running on a Rev C job.

### Step-by-step walkthrough

**Step 1 — Part master: two records, one per revision.**

Entity: `part` (two records)

Record A — 7832-001 Rev B:
- `part_number`: 7832-001
- `revision`: B
- `fai_status`: Approved (existing, proven)
- `active`: true

Record B — 7832-001 Rev C:
- `part_number`: 7832-001
- `revision`: C
- `fai_status`: In_Progress (FAI not yet complete)
- `active`: true
- UUID distinct from Rev B record

**Step 2 — Part specifications: two records.**

Entity: `part_specification` (two records)
- 7832-001 Rev B: `revision` = B; established drawing
- 7832-001 Rev C: `revision` = C; tightened bore tolerance

**Step 3 — Routings: two records.**

Entity: `routing` (two records)
- Rev B routing: `status` = Released (established, proven)
- Rev C routing: `status` = Released (new version per ECN-2026-0042)

**Step 4 — NRE packages: two records.**

Entity: `nre_package` (two records)
- Rev B package: `status` = Approved (existing, all documents current)
- Rev C package: `status` = InReview (NC programs written, CMM program updated; not yet prove-out approved; blocks Rev C production from claiming Approved status)

**Step 5 — Customer purchase orders: two records.**

Entity: `customer_purchase_order` (two records)

CPO-101 (Rev B):
- `part_id`: FK → part 7832-001 Rev B
- `quantity`: 20
- `quantity_shipped`: 15
- `open_quantity`: 5
- `status`: Partially_Shipped

CPO-107 (Rev C):
- `part_id`: FK → part 7832-001 Rev C
- `quantity`: 10
- `quantity_shipped`: 0
- `open_quantity`: 10
- `status`: In_Production
- `quality_clause_ids`: includes FAI_Required (triggers first-article constraint on the job)

**Step 6 — Jobs: two records.**

Entity: `job` (two records)

J-2026-200 (CPO-101, Rev B, 5 remaining):
- `routing_id`: FK → routing Rev B
- `customer_purchase_order_id`: FK → CPO-101
- `serial_numbers`: SN-016 through SN-020
- `status`: In_Production

J-2026-201 (CPO-107, Rev C, 10 pieces):
- `routing_id`: FK → routing Rev C
- `customer_purchase_order_id`: FK → CPO-107
- `serial_numbers`: SN-021 through SN-030
- `status`: In_Production

The separate `routing_id` FKs on each job are the guard that prevents Rev B NC programs from being issued to J-2026-201.

**Step 7 — Rev C FAI: first article inspection on SN-021.**

Entity: `inspection_event` (IE-0110)

Key fields:
- `event_type`: First_Article
- `job_id`: FK → J-2026-201
- `serial_ids`: [SN-021]
- `fai_package_id`: FK → certification_package (new, `includes_fai = true`)
- `result_summary`: Pending

Entity: `as9102_form_1` (embedded in `first_article_inspection`)

Populated for SN-021: all part numbers (7832-001 Rev C + all sub-components) accounted for; CAGE codes, drawing numbers, FAI report number recorded.

**Step 8 — Certification package assembled for SN-021.**

Entity: `certification_package`

Key fields:
- `includes_fai`: true
- `fai_package_id`: this package UUID
- `covered_serial_ids`: [SN-021]
- `contained_document_ids`: [FAI forms 1/2/3, material certs, process certs for Rev C routing]
- `status`: Submitted (to SpaceCo customer)

Part 7832-001 Rev C: `fai_status` → Submitted.

**Step 9 — Rev B parallel shipment proceeds.**

Entity: `shipment` (outbound, CPO-101)

Serial SN-016 ships under CPO-101 while Rev C FAI is in progress. Rev B cert package assembled from established Rev B material certs and process certs. No FAI required on SN-016 (FAI already Approved on Rev B).

**Step 10 — Key characteristics verified for SN-021.**

Entity: `key_characteristic`

All KC records for 7832-001 Rev C verified against `inspection_plan` entries. Balloon numbers match Form 3 characteristic entries. Two KCs on Rev C drawing include the tightened bore tolerance (the bore that changed from Rev B). SN-021 passes both KCs.

**Step 11 — Customer approves Rev C FAI.**

Entity: `part` (7832-001 Rev C)

Update:
- `fai_status`: Submitted → **Approved**

Entity: `nre_package` (Rev C)

Update:
- `status`: InReview → **Approved** (prove-out and FAI both complete)

Remaining serial numbers SN-022 through SN-030 released for production under the approved Rev C NRE package.

### Digital thread break analysis

Without separate `part` records per revision, the `job.routing_id` FK would have no revision-specific anchor; the production system could not enforce that J-2026-201 runs Rev C operations. Without `nre_package` carrying `fai_status`-awareness (via the `part` FK), the Rev C `nre_package.status = InReview` would have no queryable signal blocking production use. Without `inspection_plan` with ballot numbers linked to `key_characteristic`, the Form 3 KC verification on SN-021 would be a manual cross-reference exercise rather than a queryable pass/fail record.

### Entities introduced by Walks 4–7 (not in Archetypes A, B, or C)

Deduplicated list of every unique `[Entity]` node type introduced by these four scenario walks that is not already on the Archetype A, B, or C entity lists.

- `[Calibration Record]`
- `[Deviation Waiver]`
- `[Engineering Change Notice]`
- `[Fixture Use]`
- `[Inspection Event]` (first-class UUID envelope, distinct from `[PMI Event]`)
- `[Inspection Plan]`
- `[NC Program]`
- `[NRE Package]`
- `[Prove-Out Record]`
- `[Tool Maintenance Event]`
- `[Tool Room Allocation]`
