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
