# the manufacturer Digital Thread — Architecture and Domain Map

## Four-Layer Architecture

The digital thread at the shop is organized in four layers. Layers 1 and 2 are definitional — they specify what the data model is. Layers 3 and 4 are operational — they specify how the data model is implemented and accessed.

```
LAYER 1: STANDARDS
  The foundational data formats and protocols. Adopted without modification.
  - STEP AP242 (ISO 10303-242): Design and model-based definition
  - MTConnect (ANSI/MTC 1.1–2.2): Machine telemetry
  - QIF (ISO 23952 / ANSI QIF 3.0): Quality information
  - X12 EDI (ASC X12): Commercial transactions

LAYER 2: CM DATA MODEL
  manufacturer-defined entities and protocols that fill the gaps the standards leave at the
  manufacturer layer. This is a specification — portable across software implementations.
  - Material specifications, lots, MTRs, CoC chains, DFARS traceability
  - Outside process operations: hybrid routing-op + PO + cert collection event
  - Quality flowdowns: QA-001 through QA-017; AS9102 FAI three-form structure; NCR/MRB
  - Supplier approval hierarchy: customer-directed, ASL, Nadcap, internal qualified
  - UUID discipline: persistent artifact identity across all Layer 1 format boundaries

LAYER 3: DATA LAYER
  The operational system of record that implements the Layer 2 entity model.
  The spec is software-agnostic; the current implementation is the production system.
  Any conforming system must implement:
    - Layer 2 entity schema and field definitions
    - UUID assignment and propagation rules
    - Layer 1 format mappings defined in integration/
  Today: the production system (ERP/MES anchor)
  Future: Carbon OS or any conforming successor

LAYER 4: INTERFACE LAYER
  How operators, systems, and AI interact with Layer 3 data.
  - AI-native: Claude MCP over the production system (live)
  - Integration pipelines: MTConnect → production system; CMM → QIF → production system; production system → X12 ASN
  - Generated outputs: cert packages, FAI packages, OEE reports, pre-shipment alerts
  - Custom interfaces: any system that reads from or writes to the Layer 3 data layer
```

The Layer 2 Manufacturer data model is the novel contribution of this repo. Layers 1, 3, and 4 reference established standards and existing software; Layer 2 is what the shop defines.

---

## Eight Domains

The digital thread touches eight operational domains. For each domain:
- **Standard coverage** — which existing standard addresses it
- **manufacturer-layer gap** — what the standard doesn't handle at the manufacturer layer
- **manufacturer extension** — how this repo fills the gap

---

### 1. Materials

**Standard coverage:**
STEP AP242 carries material callouts as PMI text: `"7075-T7351 ALUMINUM ALLOY PER AMS 4078"`. This is unstructured — STEP has no schema for the AMS spec as a queryable entity, no field for DFARS compliance, no link to a physical material lot.

**manufacturer-layer gap:**
No standard defines the chain from drawing callout → specification → purchased lot → Mill Test Report → DFARS declaration. This chain is what makes a shipment certifiable and DFARS-auditable.

**manufacturer extension:**
`extensions/material-certs.md` defines the narrative and field shapes. Machine-readable schemas are in `schemas/material.yaml` (material, material_specification, material_lot) and `schemas/certification_document.yaml` (certification_document supertype). Supporting entities:
- `material_specification` (`schemas/material.yaml`) — AMS/ASTM spec as a first-class object with alloy family, UNS number, forms covered, conditions defined, and DFARS applicability
- `material_condition` (`schemas/material_condition.yaml`) — a single temper/condition within a spec, with minimum mechanical properties (tensile, yield, elongation, hardness)
- `material_lot` (`schemas/material.yaml`) — the physical lot received from a distributor: heat number, melt country, DFARS compliance flag, PMI verification, lot dimensions
- `certification_document` (`schemas/certification_document.yaml`) — discriminated supertype for MTR, CoC, DFARS_Declaration, PMI_Report, ConflictMineral, ShelfLife, ExportCompliance; each cert_type has a typed sub-field block
- `pmi_event` (`schemas/pmi_event.yaml`) — raw instrument record from XRF / OES / ICP Positive Material Identification testing at receiving inspection

---

### 2. Parts and Structures

**Standard coverage:**
STEP AP242 defines part geometry, BOM structure, and product identification (P/N, revision, CAGE code). It is the authoritative source for design intent.

**manufacturer-layer gap:**
STEP AP242 carries no approval state. It does not record whether the manufacturer is an approved source for this P/N, whether a First Article Inspection has been completed and approved, or whether source inspection is required. Configuration control without PLM requires a different approach.

**manufacturer extension:**
The item master in the production system Pro carries CAGE code, ITAR classification, customer approval status, and FAI status per P/N + revision. These fields are not defined by STEP — they are the manufacturer's Layer 2 additions that bridge the gap between design intent and approved production source. Machine-readable schemas for this domain:
- `part` and `assembly` (`schemas/part.yaml`) — part number + revision entities; assembly is a distinct type per Decision 1.1, carrying torque specs, assembly operations, and assembly-level FAI state that do not apply to individual parts
- `part_specification` (`schemas/part.yaml`) — the governing drawing (drawing number, revision, CAGE, STEP AP242 file ref, 2D PDF ref, MBD flag)
- `bom_line` (`schemas/part.yaml`) — BOM edge entity with quantity, UOM, find number, and effectivity; implements STEP AP242 NEXT_ASSEMBLY_USAGE_OCCURRENCE with manufacturer-layer additions
- `export_classification` (`schemas/export_classification.yaml`) — ITAR/EAR classification record per part or assembly; drives data handling rules and sub-tier PO eligibility
- `customer_approval` (`schemas/customer_approval.yaml`) — the manufacturer's approval status as a source for a given P/N+rev per customer; records FAI status and qualification method

---

### 3. Manufacturing

**Standard coverage:**
STEP AP238 (ISO 10303-238) defines CNC process plans in a neutral format. MTConnect streams machine state data from in-house CNC machines.

**manufacturer-layer gap:**
STEP AP238 has no schema for operations that leave the building. MTConnect produces data only for in-house CNC machines. Every outside processing operation — heat treat, plating, NDT, passivation — leaves a structural gap in the machine data timeline. Additionally, STEP AP238 adoption at the manufacturer layer is near zero; most CMs receive 2D PDF drawings.

**manufacturer extension:**
`extensions/outside-processing.md` defines the `outside_process_operation` as a hybrid entity: simultaneously a routing operation (with scheduling and status), a purchase order (with vendor, cost, and shipping), and a certification collection event (with documented conformance). The entity captures what MTConnect and STEP AP238 cannot: the physical location of parts during outside processing, the process specification being run, and the certifications that return with the parts. Machine-readable schemas for this domain:
- `routing` and `operation` (`schemas/routing.yaml`) — the process plan template and its in-house work steps; `operation` is the template, `work_order` (`schemas/job.yaml`) is the execution instance
- `work_center` (`schemas/routing.yaml`) — named machine or station where operations execute; MTConnect device ID links machine telemetry to the production system routing data
- `assembly_routing` and `assembly_operation` (`schemas/routing.yaml`) — assembly process plan with torque control, fastener traceability, and witness inspection additions that do not apply to machining operations
- `outside_process_operation` (`schemas/outside_process_operation.yaml`) — the hybrid entity bridging routing, procurement, and certification collection for off-site operations
- `process_specification` (`schemas/outside_process_operation.yaml`) — AMS/MIL/ASTM spec governing a special process; drives vendor qualification and cert requirements
- `process_certification` (`schemas/outside_process_operation.yaml`) — vendor conformance document (CoC, process chart, NDT report, thickness/hardness survey); distinct from `certification_document` which covers material certs
- `job`, `serial_number`, `work_order` (`schemas/job.yaml`) — the execution containers: job is the production run, serial_number is the individual unit, work_order is the scheduled and tracked instance of each routing operation

---

### 4. Quality

**Standard coverage:**
QIF (ISO 23952) defines the complete inspection lifecycle: measurement plans (QIF Plans), CMM results (QIF Results), and statistical analysis (QIF Statistics). QIF covers FAI Form 3 (Characteristic Accountability) completely.

**manufacturer-layer gap:**
QIF covers only dimensional measurement. AS9102 Rev C requires three FAI forms:
- Form 1 (Part Number Accountability) — outside QIF scope entirely
- Form 2 (Product Accountability — materials and processes) — outside QIF scope entirely
- Form 3 (Characteristic Accountability) — QIF covers this form

Additionally, QIF has no schema for nonconformance reports, MRB dispositions, or special process certifications.

**manufacturer extension:**
`extensions/quality-flowdowns.md` defines the narrative and field shapes. Machine-readable schemas are in `schemas/quality.yaml`:
- `quality_clause` (`schemas/quality.yaml`) — reusable quality obligation with category, cascade rules, and job-requirement triggers; covers QA-001 through QA-017 and customer-specific variants
- `quality_flowdown_plan` (`schemas/quality.yaml`) — the machine-readable quality addendum for a job: active clauses, derived requirements (FAI, CoC, MTR, DFARS, Nadcap), and sub-tier flowdown records
- `first_article_inspection` (`schemas/quality.yaml`) — AS9102 three-form FAI structure; `subject_id + subject_type` polymorphic FK for part or assembly (Decision 1.9); Form 3 characteristics link to QIF UUIDs; `as9102_form_1/2/3` are embedded sub-objects
- `nonconformance_report` (`schemas/quality.yaml`) — NCR and embedded MRB disposition workflow; `subject_id + subject_type` polymorphic FK; links to serial numbers, material lots, QIF result UUIDs, and rework jobs

---

### 5. Supply Chain

**Standard coverage:**
X12 EDI defines the commercial transaction envelope: 850 (PO), 855 (acknowledgment), 856 (ASN), 810 (invoice), 830 (forecast). The AIA implementation guide specifies aerospace-specific field usage.

**manufacturer-layer gap:**
X12 850 carries quality obligations only as unstructured `MSG` free-text or as PDF attachments outside the EDI transaction. There is no machine-readable representation of quality clause flowdowns, FAI requirements, or Nadcap requirements in X12. Sub-tier POs (outside process vendors) are entirely disconnected from the originating customer 850.

**manufacturer extension:**
`extensions/quality-flowdowns.md` defines the quality clause structure. `extensions/outside-processing.md` defines the sub-tier PO linkage. `extensions/uuid-discipline.md` defines how the production job UUID is carried in X12 REF segments to maintain cross-reference.

**Canonical schemas (Phase 2.E):**
- `schemas/supplier.yaml` — supplier contact and capability record (renamed from vendor per Decision 1.3)
- `schemas/approved_supplier_list_entry.yaml` — supplier qualification record: Nadcap certs, customer approvals, ITAR status
- `schemas/purchase_order.yaml` — outbound PO (material, sub-tier, services, opex); po_kind discriminator; state machine
- `schemas/customer_purchase_order.yaml` — inbound PO (X12 850); quality flowdown origin; state machine
- `schemas/shipment.yaml` — goods movement (outbound/inbound/return); X12 856 source; direction-discriminated state machines
- `schemas/receipt.yaml` — goods-received event; triggers inventory movement and receiving inspection
- `schemas/nadcap_certification.yaml` — PRI Nadcap certificate held by a supplier (standalone from approved_supplier_list_entry)
- `schemas/nadcap_category.yaml` — Nadcap process category reference entity (AC7xxx; first-class per Decision 1.7)
- `schemas/nadcap_certification.yaml`, `schemas/supplier_qualification_event.yaml` — vendor audit trail
- `schemas/customer_asl.yaml` — customer-provided Approved Source List; constrains outside process vendor selection

---

### 6. Sales and Quoting

**Standard coverage:**
Partial X12 coverage for PO receipt and acknowledgment. Quoting is pre-commercial and not covered by any standard.

**manufacturer-layer gap:**
Export control (ITAR/EAR) classification of the part drives which customers the manufacturer can quote, which vendors can perform outside processing, and how data must be handled. None of this is encoded in any standard. Additionally, the quality clause cascade from customer PO → production job → sub-tier PO begins at quote time when the customer's quality requirements are first understood.

**Integration:**
ITAR classification on the production system item master gates which quotes can include foreign entities and which data can flow through which systems. See `extensions/ip-boundary.md`.

**Canonical schemas (Phase 2.E):**
- `schemas/customer.yaml` — persistent customer record: name, CAGE, compliance flags (ITAR screening, DoD prime, DFARS), ASL references
- `schemas/customer_purchase_order.yaml` — inbound PO from customer; quality flowdown cascade origin
- `schemas/export_classification.yaml` — ITAR/EAR classification per part/assembly (Phase 2.B; applies to quoting gate)

**Canonical schemas (Phase 2.F):**
- `schemas/sales_order.yaml` — the manufacturer's internal acceptance record derived from customer_purchase_order; drives job creation and tracks delivery; state machine from Draft through Closed

---

### 7. Inventory

**Standard coverage:**
ISA-95 Part 2 (IEC 62264-2) defines resource management models for manufacturing, including material inventory. It establishes concepts for lot control and material identification.

**manufacturer-layer gap:**
ISA-95 defines the model but not the aerospace-specific attributes: heat number traceability to a specific mill, DFARS melt-country compliance, PMI (Positive Material Identification) verification at receiving, shelf-life controls for limited-life materials, and ITAR-controlled inventory location segregation.

**manufacturer extension:**
`extensions/material-certs.md` defines the `material_lot` entity with these attributes. the production system's inventory module carries the lot record; the extension defines what fields must be captured and how they link to the certification document chain.

**Canonical schemas (Phase 2.F):**
- `schemas/inventory_location.yaml` — named storage location with ISA-95-grounded type hierarchy; ITAR-segregated flag; hierarchical nesting via parent_location_id; Off_Site type tracks material at outside processors

---

### 8. Compliance and Governance

**Regulatory framework:**
- **ITAR (22 CFR 120-130):** Controls export of defense technical data; applies to most aerospace parts
- **DFARS 252.225-7009:** Specialty metals melt-source traceability for DoD contracts
- **CMMC:** Cybersecurity Maturity Model Certification for CUI handling (DoD contractors)
- **AS9100 Rev D:** Aerospace quality management system
- **AS9102 Rev C:** First Article Inspection

**manufacturer-layer gap:**
No standard integrates these regulatory requirements into the operational data model. ITAR classification is not in STEP. DFARS compliance is not in X12 (beyond text references). CMMC requirements for data residency are not addressed by any manufacturing standard.

**manufacturer extension:**
`extensions/ip-boundary.md` defines the data classification matrix, ITAR data handling rules, CMMC boundary assessment, and record retention requirements. This is the governance layer that overlays all seven operational domains.

**Canonical schemas (Phase 2.F):**
- `schemas/user.yaml` — shop floor and office users with citizenship_status for ITAR access control and training_records for quality signatory eligibility; roles[] for application-layer permissions
- `schemas/data_classification.yaml` — machine-readable governance framework implementing the ip-boundary.md Data Classification Matrix; classification_level enum, storage/transmission rules, ITAR flag, access_roles[], retention_years

---

### 9. Process Engineering / NRE

**What this domain covers:**
The engineering and prove-out work required to manufacture a part for the first time or after a significant revision: NC programming, CAM work, fixture design, CMM programming, inspection planning, and the first-piece prove-out that certifies the process is production-ready. NRE (Non-Recurring Engineering) is the cost and document set that locks together a specific drawing revision and routing revision.

**Layer-1 gap:**
STEP AP238 covers process plan structure for CNC but has no concept of the NC program file itself, the prove-out event, or the CAM-to-NC relationship. STEP AP242 covers PMI and key characteristics but not the inspection plan ordering. Neither standard has an NRE package as an organizing entity.

**manufacturer extension:**
`extensions/process-engineering-nre.md` defines the NRE package as the revision-lock envelope. Key architectural decisions: `cam_file → nc_program` is one-to-many (one CAM project posts to multiple machines); `nc_program` is per-operation × per-machine; `nre_package` locks `part_specification_id` + `routing_id` together so that either a drawing or routing revision change requires a new NRE package and re-approval.

**Entities:** `nre_package`, `nc_program`, `cam_file`, `setup_sheet`, `cmm_program`, `inspection_plan`, `prove_out_record`, `print_file`, `tooling_list`, `tooling_list_line`, `fixture_use`

**Canonical schema:** `schemas/process_engineering.yaml`

---

### 10. Tool Room

**What this domain covers:**
The definition, preparation, allocation, and lifecycle management of cutting tools, workholding fixtures, and measurement gauge catalog entries. Bridges NC program tool requirements to the physical tool room inventory. Physical serialized holders and fixtures are `part` entities with `part_kind: fixture`; this domain defines the catalog definitions and operational records that govern them.

**Layer-1 gap:**
MTConnect CuttingTool asset schema covers tool identity and asset ID but not the catalog hierarchy (cutter_definition × holder_definition = tool_assembly_definition), presetter measurement records (tool_preset), checkout/allocation system (tool_room_allocation), or a gauge catalog separate from the calibrated instrument (`tool_gauge`, which is in Domain 7). ISO 13399 covers cutting tool data exchange but not the holder+cutter assembly definition or the presetter measurement workflow.

**manufacturer extension:**
`extensions/tool-room.md` defines the two-level catalog model (definition vs. instance) and the full workflow from NC program authoring (tool_pocket_requirement) through preset measurement (tool_preset) through machine loading (tool_pocket_assignment). Physical holders and fixtures remain `part.id` records; the tool room tracks their allocation and maintenance without creating a separate instance entity.

**Entities:** `cutter_definition`, `holder_definition`, `tool_assembly_definition`, `gauge_definition`, `tool_preset`, `tool_pocket_requirement`, `tool_pocket_assignment`, `tool_room_allocation`, `tool_maintenance_event`, `tool_life_record`

**Note:** `tool_gauge` (the serialized calibrated instrument) and `calibration_record` are in Domain 7 / `schemas/inspection.yaml`. This domain's `gauge_definition` is the catalog entry; `tool_gauge` instances reference it conceptually.

**Canonical schema:** `schemas/tool_room.yaml`

---

### 11. Packaging

**What this domain covers:**
Packaging specification authoring and packaging execution evidence. Treats packaging as a traceable manufacturing operation per AS9100 §8.5.4. Covers MIL-STD-2073 preservation levels (A/B/C), ESD protection, desiccant and humidity indicator requirements, MIL-STD-129/130 labeling, and 3D-printed or machined packaging inserts (which are manufactured parts with `part_kind: packaging_insert`).

**Layer-1 gap:**
No Layer-1 manufacturing data standard defines packaging as a traceable operation with a data schema. MIL-STD-2073 defines preservation requirements in prose but has no data model. Shipment records (`schemas/shipment.yaml`) track logistics but not the packaging execution evidence required for AS9100 §8.5.4 preservation of conformity.

**manufacturer extension:**
`extensions/packaging.md` defines the three-entity packaging model: `packaging_specification` (the governing document per subject), `packing_record` (the immutable execution evidence for a specific shipment/serial/lot), and `packaging_item` (individual packaging components with material lot traceability). Packaging inserts are `part` entities and participate in the full part lifecycle.

**Entities:** `packaging_specification`, `packing_record`, `packaging_item`

**Canonical schema:** `schemas/packaging.yaml`

---

### 12. Change Management

**What this domain covers:**
Authorization of engineering changes (ECNs) and deviation/waiver requests. ECNs authorize revision changes to drawings and routings; they trigger cascades but do not model the cascade logic itself. Deviations/waivers authorize use of material or product that deviates from specification, with customer or DER approval.

**Layer-1 gap:**
No Layer-1 standard defines ECN or deviation/waiver as data entities. AS9100 §8.3.6 requires documented engineering change control; AS9100 §8.7.1 covers nonconforming output. The data structures that satisfy these clauses are pure manufacturer-layer entities.

**manufacturer extension:**
`extensions/change-management.md` covers both entities and their distinction from the internal quality records (`nonconformance_report`, which captures the nonconformance; `deviation_waiver`, which is the customer/DER authorization document that may reference an NCR). ECN lifecycle: Draft → UnderReview → Approved → Released. Deviation/waiver has expiry (date- and quantity-limited).

**Entities:** `engineering_change_notice`, `deviation_waiver`

**Canonical schema:** `schemas/change_management.yaml`

---

### Cross-Domain Extensions (Assembly Hardware and Operator Qualification)

**Assembly Hardware** — execution records for assembly operations involving threaded inserts (`insert_installation_record`) and press-fit components (`press_fit_record`). These are immutable quality records, analogous to `torque_readings_record` in Domain 5, that provide per-serial traceability for mechanical assembly operations. Defined in `schemas/assembly_hardware.yaml`; conceptually part of Domain 5 (Assembly) and Domain 7 (Quality).

**Operator Qualification** — records that a specific operator is trained and currently certified to perform a specific process, operation code, or equipment type. Distinct from `nadcap_certification` (facility-level Nadcap accreditation) — this tracks individual operator currency. Required for special processes (welding, NDT, torque-critical assembly, threaded insert installation). Defined in `schemas/operator_qualification.yaml`.

---

## UUID Discipline

UUID is the connective tissue across all layers. Every entity in the digital thread has a UUID v4 identifier, minted by the production system Pro, that persists as the entity moves through format translations:

```
production system Job UUID
  → carried in MTConnect WorkOrder assetId
  → carried in QIF document header (extension field)
  → carried in X12 856 ASN REF*ZZ segment

production system Serial UUID
  → carried in QIF SerialNumber header (extension field)
  → carried in X12 856 SN1 segment

Material Lot UUID
  → carried in MTConnect RawMaterial LotId (extension field)
  → links material_lot → certification_document chain
```

See `extensions/uuid-discipline.md` for the complete entity type list, UUID assignment rules, and format-translation survival requirements.
