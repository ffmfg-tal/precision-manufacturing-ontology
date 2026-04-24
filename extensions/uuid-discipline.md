# UUID Discipline — Artifact Identity Across the Digital Thread

## Why This Matters

The digital thread is only a thread if the same physical artifact is recognizable as itself across every system it passes through. A part designed in STEP AP242, machined under an MTConnect work order, inspected via a QIF measurement plan, and shipped under an X12 ASN is one object — but without an external identity discipline, it appears as four unrelated records in four different systems.

UUID persistence is the connective tissue that makes the thread coherent. It enables:
- Linking machine data to a specific job and serial number
- Linking CMM results to the same artifact the machine touched
- Answering cross-domain queries ("show me all jobs where spindle overload events correlated with CMM out-of-tolerance results")
- Supporting the AI-native query layer with ground-truth identity

This approach mirrors the federated digital thread architecture developed in DMDII research (Hardwick, RPI), where UUID persistence across format boundaries was identified as the primary enabler of multi-system traceability.

---

## Entity Types Requiring UUIDs

These are the artifact types that cross system boundaries or require long-term identity:

| Entity | Description | Where UUID Is Used |
|--------|-------------|-------------------|
| **Part (design)** | A specific part number + revision combination | STEP AP242 file reference, production system item master, QIF measurement plan target. Schema: `schemas/part.yaml` — `part` entity |
| **Assembly** | A joined product composed of multiple parts; distinct from part per Decision 1.1 | production system item master; assembly FAI package; cert package; X12 ASN line item. Schema: `schemas/part.yaml` — `assembly` entity |
| **Part Specification** | A governing drawing (drawing number + revision + CAGE) | production system item master; STEP AP242 file reference; QIF measurement plan. Schema: `schemas/part.yaml` — `part_specification` entity |
| **BOM Line** | A parent–child relationship edge in the product structure (with qty, UOM, find number) | the production system item routing; BOM traversal queries; assembly work instructions. Schema: `schemas/part.yaml` — `bom_line` entity |
| **Export Classification** | The ITAR/EAR classification record for a part or assembly | production system item master; PO screening; shipment export declaration. Schema: `schemas/export_classification.yaml` |
| **Customer Approval** | the manufacturer's approval status as a source for a given P/N+rev from a specific customer | production system item master; FAI package link; source inspection planning. Schema: `schemas/customer_approval.yaml` |
| **Job** | A production run of a specific part or assembly (subject_type discriminator per Decision 1.9) | production job ID, MTConnect work order reference, QIF plan/results association, X12 ASN REF*ZZ. Schema: `schemas/job.yaml` — `job` entity |
| **Serial Number** | A single physical unit within a job; carries material lot traceability and QIF/NCR links | the production system serial record, QIF SerialNumber header, X12 ASN SN1 segment, cert package. Schema: `schemas/job.yaml` — `serial_number` entity |
| **Work Order** | The execution instance of a routing operation on a specific job; distinct from operation (the template) | MTConnect WorkOrder OperationId (correlates machine events to job operation), production job operation UUID. Schema: `schemas/job.yaml` — `work_order` entity |
| **Routing** | The process plan template for a part — ordered sequence of operations | the production system item routing; job creation; BOM traversal. Schema: `schemas/routing.yaml` — `routing` entity |
| **Outside Process Operation** | A hybrid entity: routing operation + PO + cert collection for off-site vendor work | production job operation (Outside_Process type), PO record, cert attachment. Schema: `schemas/outside_process_operation.yaml` |
| **Process Certification** | A vendor conformance document for a special process (CoC, process chart, NDT report) | Cert package assembly, the production system cert attachment, job traceability chain. Schema: `schemas/outside_process_operation.yaml` |
| **Material Lot** | A specific purchased lot of raw material (tied to heat number) | the production system material lot, cert package assembly, traceability chain |
| **Inspection Event** | A single CMM or dimensional inspection run | QIF results file, the production system inspection record |
| **Tool / Gauge** | A cutting tool or measurement device | Calibration records, MTConnect tool reference, QIF resource |
| **Certification Document** | A material or process conformance cert (MTR, CoC, DFARS_Declaration, PMI_Report, ConflictMineral, ShelfLife, ExportCompliance) | the production system cert attachment record; cert package assembly; material_lot traceability chain. Schema: `schemas/certification_document.yaml` |
| **Material Specification** | An AMS/ASTM/MIL spec as a versioned queryable entity | the production system material spec library; material_lot.spec_reference; AS9102 Form 2 line-item. Schema: `schemas/material.yaml` |
| **PMI Event** | A Positive Material Identification test run (XRF / OES / ICP) at receiving inspection | the production system receiving inspection record; material_lot.pmi_verified flag; pmi_cert link. Schema: `schemas/pmi_event.yaml` |
| **Material Condition** | A temper or treatment condition within a material_specification (e.g., H1025 in AMS 5659) | material_specification entity authoring; MTR conformance validation; AS9102 Form 2. Schema: `schemas/material_condition.yaml` |
| **Material Identification** | An AS9102 Form 2 line-item record — one drawing callout with its substantiating cert and approval code | AS9102 FAI Form 2 package; cert package assembly; traceability to material_lot and certification_document. Schema: `schemas/material_identification.yaml` |
| **Quality Clause** | A reusable quality obligation (QA-001 through QA-017, customer variants) that cascades to jobs and sub-tier POs | Quality flowdown plans; sub-tier PO text generation; cert package requirement derivation. Schema: `schemas/quality.yaml` |
| **Quality Flowdown Plan** | The machine-readable quality addendum for a job: active clauses, derived requirements, sub-tier flowdowns | production job quality plan; sub-tier PO generation; cert package checklist. Schema: `schemas/quality.yaml` |
| **First Article Inspection** | An AS9102 Rev C three-form FAI record (Form 1 part accountability, Form 2 material/process, Form 3 characteristics) | FAI package; item master FAI status; QIF Form 3 linkage; customer approval. Schema: `schemas/quality.yaml` |
| **Nonconformance Report** | A quality event record for any article failing a specified requirement; includes embedded MRB disposition | NCR log; MRB workflow; rework job linkage; QIF out-of-tolerance trigger; cert package quality evidence. Schema: `schemas/quality.yaml` |
| **Supplier** | Contact, address, and process-capability record for an outside processor or material vendor; renamed from vendor per Decision 1.3 | the production system vendor record; outside process PO FK; approved_supplier_list_entry FK. Schema: `schemas/supplier.yaml` |
| **Approved Supplier List Entry** | the manufacturer's qualification record for a supplier: Nadcap certs, customer approvals, ITAR status, performance rating; renamed from approved_vendor_list_entry per Decision 1.3 | the production system vendor qualification record; vendor selection decision logic; supplier_qualification_event audit chain. Schema: `schemas/approved_supplier_list_entry.yaml` |
| **Customer** | Persistent record for an aerospace customer: name, CAGE, compliance flags, ASL references | the production system customer record; sales order FK; quality clause FK; customer_asl FK; customer_approval FK. Schema: `schemas/customer.yaml` |
| **Purchase Order** | Outbound PO from the manufacturer to a supplier (material, sub-tier, services, or opex); sub_tier_purchase_order is an alias with po_kind: sub_tier per Decision 1.4 | the production system PO record; outside_process_operation FK; quality flowdown subtier flowdown FK; receipt FK. Schema: `schemas/purchase_order.yaml` |
| **Customer Purchase Order** | Inbound PO from an aerospace customer (X12 850); origin of the quality flowdown cascade | X12 850 EDI transmission; the production system sales order source; quality_flowdown_plan.sales_order_id chain. Schema: `schemas/customer_purchase_order.yaml` |
| **Shipment** | Logistics record for goods movement in any direction (outbound/inbound/return); X12 856 ASN generated from outbound shipments; shipment_inbound is an alias per Decision 1.5 | X12 856 ASN; the production system shipment record; serial_number and material_lot traceability; cert package link. Schema: `schemas/shipment.yaml` |
| **Receipt** | Goods-received event at the shop dock — triggers inventory movement, receiving inspection, and cert collection | the production system receiving record; material_lot creation; PMI event trigger; outside process operation return closure. Schema: `schemas/receipt.yaml` |
| **Nadcap Certification** | A PRI Nadcap accreditation certificate for a specific process category held by a supplier | the production system vendor qualification; outside process vendor selection (Nadcap required check); approved_supplier_list_entry.nadcap_certification_ids[]. Schema: `schemas/nadcap_certification.yaml` |
| **Nadcap Category** | A Nadcap process category (AC7xxx audit scope) — the queryable reference entity for category codes like HT, CP, NDT; first-class entity per Decision 1.7 | outside_process_operation.nadcap_category FK; nadcap_certification.category_id FK; vendor selection filter. Schema: `schemas/nadcap_category.yaml` |
| **Supplier Qualification Event** | Timestamped audit-trail record for a supplier approval lifecycle action (initial approval, review, suspension, reinstatement); renamed from vendor_qualification_event per Decision 1.3 | the production system vendor audit trail; AS9100 §8.4 supplier evaluation evidence. Schema: `schemas/supplier_qualification_event.yaml` |
| **Customer ASL** | A customer's Approved Source List document imported into the manufacturer — constrains outside process vendor selection when QA-013 is active | the production system vendor selection constraint; customer_asl.entries[] cross-ref to approved_supplier_list_entry. Schema: `schemas/customer_asl.yaml` |
| **Sales Order** | the manufacturer's internal acceptance record derived from a customer_purchase_order; source of truth for due dates and delivery commitments | the production system sales order record; job creation FK; quality_flowdown_plan FK; X12 855 acknowledgment origin. Schema: `schemas/sales_order.yaml` |
| **Certification Package** | Machine-readable assembly of cert documents (material certs, process certs, FAI, CMM results, the manufacturer CoC) transmitted with a delivery | Cert package transmittal tracking; compliance audit; customer portal upload. Schema: `schemas/certification_package.yaml` |
| **Inventory Location** | A named storage location (warehouse, work cell, quarantine, off-site) with ITAR-segregation flag and hierarchical nesting support | the production system inventory location master; material_lot.current_location_id FK; ITAR segregation enforcement. Schema: `schemas/inventory_location.yaml` |
| **User** | A person interacting with the manufacturer digital thread systems; carries citizenship_status for ITAR access control and training_records for quality signatory eligibility | the production system user record; work_order.assigned_operator_id FK; NCR.originator_id FK; FAI signatory. Schema: `schemas/user.yaml` |
| **Data Classification** | A classification rule record defining storage, transmission, and access requirements for a data category; implements ip-boundary.md governance framework | production system data governance; ITAR/CMMC boundary enforcement; Claude MCP data handling rules. Schema: `schemas/data_classification.yaml` |
| **Engineering Change Notice** | Authorization record for a change to a part drawing revision, routing version, material, process, or governing specification; AS9100 §8.3.6 compliance record | production system ECN record; part_specification old/new revision FK; routing_id old/new FK; customer portal change notification. Schema: `schemas/change_management.yaml` — `engineering_change_notice` entity |
| **Deviation Waiver** | Customer or DER authorization permitting use of nonconforming material (Deviation = pre-production, Waiver = post-production); AS9100 §8.7.1 record | production system quality record; NCR FK (waiver case); customer approval system; DER certificate traceability. Schema: `schemas/change_management.yaml` — `deviation_waiver` entity |
| **NRE Package** | Revision-lock envelope for all process engineering documents (NC programs, CAM files, setup sheets, CMM programs, inspection plans, tooling lists) for a specific drawing + routing revision pair | production system NRE record for billing; PLM revision-lock reference; nre_package_id FK on nc_program, cam_file, cmm_program, inspection_plan, setup_sheet. Schema: `schemas/process_engineering.yaml` — `nre_package` entity |
| **NC Program** | Machine-specific CNC G-code program for one operation at one work center; lifecycle: Draft → Proven → Released → Obsolete | CAM system output file identity; production system program library (only Released programs issued); CNC controller load record; MTConnect work order program reference. Schema: `schemas/process_engineering.yaml` — `nc_program` entity |
| **CAM File** | CAM project file that is the engineering source for one or more NC programs; one file posts to multiple nc_programs via different post-processors | PLM/CAM system file identity; production system NRE document set; cam_file.id FK on nc_program for post-processor traceability. Schema: `schemas/process_engineering.yaml` — `cam_file` entity |
| **Inspection Plan** | Method-agnostic pre-defined plan specifying which characteristics to check, in what order, and with what gauge type; governs CMM, hand gauge, and vision system alike | CMM software reference (plan governs program authoring); quality system inspection record; inspection_plan_id FK on cmm_program. Schema: `schemas/process_engineering.yaml` — `inspection_plan` entity |
| **CMM Program** | Machine-executable CMM program loaded on a coordinate measuring machine for a specific inspection operation; distinct from inspection_plan (method-agnostic governance) | CMM controller program load record (PC-DMIS, Calypso, etc.); production system NRE document set; inspection_plan_id FK for plan traceability. Schema: `schemas/process_engineering.yaml` — `cmm_program` entity |
| **Operator Qualification** | Record that a specific operator is qualified to perform a specific operation type, process specification, or equipment class; required for special processes per AS9100 §8.5.1 | QMS qualification record (welding, NDT, torque-critical assembly, helicoil installation); training record systems; work_order operator eligibility check. Schema: `schemas/operator_qualification.yaml` |
| **Packaging Specification** | Governing document defining how a part or assembly must be packaged (MIL-STD-2073 preservation level, moisture barrier, ESD, desiccant, labeling) | shipment documentation reference; customer portal packaging requirement; packaging_specification_id FK on packing_record. Schema: `schemas/packaging.yaml` — `packaging_specification` entity |
| **Packing Record** | Execution instance of a packaging_specification for a specific shipment; immutable after inspector sign-off | X12 856 ASN line-item traceability; shipment FK; cert package quality evidence; packing_record_id referenced in shipment documentation. Schema: `schemas/packaging.yaml` — `packing_record` entity |
| **Insert Installation Record** | Execution record for installing threaded inserts (helicoils, keenserts, nutplates) into a parent part; immutable after inspector sign-off | QMS AS9100 §8.5.1 traceability record; assembly_operation FK; cert package assembly evidence. Schema: `schemas/assembly_hardware.yaml` — `insert_installation_record` entity |
| **Press Fit Record** | Execution record for pressing interference-fit components (dowel pins, bearing races, bushings) into a parent part; records press force and displacement with pass/fail | QMS AS9100 §8.5.1 traceability record; assembly_operation FK; cert package assembly evidence; critical quality characteristic record. Schema: `schemas/assembly_hardware.yaml` — `press_fit_record` entity |
| **Characteristic** | Base drawing callout entity — every dimension, GD&T callout, thread note, surface finish, datum, and note extracted from a part_specification revision; source includes AI_Extracted (InspectAI pipeline) | QIF measurement plan characteristic ID reference; CMM program characteristic mapping; FAI Form 3 characteristic_id FK; InspectAI characteristic ID exported to control plans and PPAP. Schema: `schemas/characteristic.yaml` |
| **Measurement Result** | Per-reading per-sample per-characteristic dimensional measurement result from a single inspection event; carries actual_value, deviation, conformance, and equipment/operator traceability | QIF results record linkage (one result per characteristic per sample); job traceability chain; cert package measurement evidence; InspectAI measurement audit trail. Schema: `schemas/measurement_result.yaml` |
| **PPAP Submission** | AIAG PPAP 4th Edition submission package tracking ppap_level, submission_reason, status lifecycle, and 18-element AIAG checklist; links to first_article_inspection and customer_approval | Customer portal PPAP submission tracking (InspectAI workflow); first_article_inspection FK; customer_approval FK; APQP quality record. Schema: `schemas/quality_tools.yaml` — `ppap_submission` entity |
| **FMEA Item** | Single row in a PFMEA or DFMEA per AIAG FMEA 4th Edition; carries severity/occurrence/detection ratings and RPN with revised ratings after corrective actions | AIAG-standard FMEA documents shared with OEM customers; control_plan_item FK (FMEA row drives control plan); ppap_submission FMEA element; InspectAI FMEA workflow. Schema: `schemas/quality_tools.yaml` — `fmea_item` entity |
| **Control Plan Item** | Single row in an AIAG APQP control plan specifying how one characteristic at one process step is monitored during production | OEM customer APQP control plan documents; characteristic.id FK; fmea_item FK; ppap_submission control plan element; InspectAI control plan workflow. Schema: `schemas/quality_tools.yaml` — `control_plan_item` entity |
| **SPC Result** | Cached SPC statistics record for one characteristic over one measurement sample set (Cp, Cpk, UCL/LCL, Western Electric rule flags); derived from measurement_result records | Quality management system SPC dashboard; characteristic FK; InspectAI SPC workflow; process capability reporting to customers. Schema: `schemas/quality_tools.yaml` — `spc_result` entity |

---

## UUID Assignment Rules

### System of Record: the production system

the production system is the Layer 2 integration anchor. All UUIDs for the manufacturer-originated artifacts are minted by the production system Pro. This means:
- When a job is created in the production system, its UUID is established there
- When a serial number is assigned (at job open or first operation), its UUID is established in the production system
- When a material lot is received, its UUID is established in the production system receiving

UUIDs for customer-supplied artifacts (part designs in STEP packages) may be provided by the customer's PLM. the manufacturer captures and preserves those IDs in the production system alongside its own job UUID.

### Minting Points (When UUIDs Are Created)

| Entity | UUID Minted When |
|--------|-----------------|
| Part (design) | Customer-assigned (from STEP file or customer PO reference); stored in the production system item master on first use |
| Assembly | Customer-assigned (from STEP file or customer PO); stored in the production system item master on first receipt |
| Part Specification | production system item master entry created when drawing first received from customer |
| BOM Line | the production system item routing entry created when routing is configured for the parent part or assembly |
| Export Classification | Classification record created in the production system item master when part is first entered; reviewed on any design change |
| Customer Approval | production system item master record created when customer approval tracking begins for this P/N+rev |
| Job | Job created in the production system; UUID minted at job creation and immediately propagated to MTConnect WorkOrder |
| Serial Number | First operation started on that unit (or at job open for serialized jobs) |
| Work Order | production job operation created when routing is instantiated on a job (each routing operation becomes one work_order with its own UUID) |
| Routing | Routing record created in the production system item routing when process plan is authored |
| Outside Process Operation | Minted in the production system at job creation when the routing (containing an outside-process step) is assigned to the job |
| Process Certification | Outside process cert received at the production system receiving inspection; UUID minted when cert is logged as an attachment |
| Material Lot | Receiving inspection in the production system |
| Inspection Event | CMM run initiated; UUID recorded before measurement begins |
| Tool / Gauge | Tool crib entry / calibration record created in the production system |
| Certification Document | Cert PDF received and logged in the production system as cert attachment record |
| Material Specification | Spec ingested into the manufacturer material specification library (production system item master) |
| PMI Event | XRF/OES/ICP run initiated at receiving inspection |
| Material Condition | Material_specification entity authored or imported into library |
| Material Identification | AS9102 Form 2 line-item record created during FAI authoring (one per drawing callout) |
| Quality Clause | Clause definition record authored in the manufacturer quality clause library (one-time; reused across jobs) |
| Quality Flowdown Plan | Quality plan record created when a job is opened from a sales order with quality clauses active |
| First Article Inspection | FAI record created in the production system when FAI workflow is initiated for a job |
| Nonconformance Report | NCR record created in the production system when nonconformance is identified (receiving, in-process, or final inspection) |
| Supplier | Supplier record created in the production system vendor master when supplier is first added (initial qualification or PO issuance) |
| Approved Supplier List Entry | the manufacturer qualification record created in the production system when supplier is first qualified; updated as Nadcap certs and customer approvals change |
| Customer | Customer record created in the production system when customer relationship begins (first PO received) |
| Purchase Order | PO created in the production system when buyer issues an outbound PO to a supplier |
| Customer Purchase Order | production system record created when inbound X12 850 is received and logged from the customer |
| Shipment | the production system shipment record created when a shipment is planned (outbound) or a delivery is announced/arrives (inbound) |
| Receipt | the production system receiving record created when goods physically arrive at the shop dock |
| Nadcap Certification | Certificate record created in the production system when Nadcap cert is received from supplier or verified on eAuditNet |
| Nadcap Category | Category record authored in the manufacturer Nadcap category library (one-time; stable reference data per PRI audit program) |
| Supplier Qualification Event | Event record created in the production system when a supplier qualification action occurs (approval, review, suspension) |
| Customer ASL | ASL record created in the production system when customer provides an Approved Source List document or portal reference |
| Sales Order | production system record created when customer_purchase_order is acknowledged and accepted (X12 855 sent) |
| Certification Package | Cert package record created in the production system when shipment is being prepared; UUID used for cert package transmittal tracking |
| Inventory Location | Location record created in the production system location master when the location is established (one-time; stable reference) |
| User | User record created in the production system when employee account is provisioned |
| Data Classification | Classification record authored in the manufacturer data governance library (one-time; stable reference data) |
| Engineering Change Notice | ECN record created in the production system when an engineering change is initiated and authorized |
| Deviation Waiver | Record created in the production system QMS when a deviation or waiver request is submitted and authorized (Deviation: pre-production; Waiver: post-production after NCR disposition) |
| NRE Package | Package record created in the production system when process engineering work begins for a new part_specification + routing revision pair; UUID immutable after creation |
| NC Program | Program record created in the production system NRE workflow when programming begins for an operation; status advances Draft → Proven → Released after prove-out sign-off |
| CAM File | CAM file record created in the production system NRE workflow when the CAM project is initiated; cam_file.id minted at creation and propagated to resulting nc_program records |
| Inspection Plan | Plan record created in the production system NRE workflow when inspection planning begins; minted before CMM program authoring so cmm_program can carry inspection_plan_id FK |
| CMM Program | Program record created in the production system NRE workflow when CMM programming begins; references inspection_plan_id and cmm_system at creation |
| Operator Qualification | Qualification record created in the QMS when a training or qualification event occurs and is signed off; minted by the QMS system-of-record for operator records |
| Packaging Specification | Spec record created in the production system when packaging requirements are defined for a part or assembly; new UUID minted for each revision |
| Packing Record | Record created in the production system when packaging is executed for an outbound shipment; minted when packing begins and frozen on inspector sign-off |
| Insert Installation Record | Record created in the production system at the time of insert installation; minted at operation start and immutable after inspector sign-off |
| Press Fit Record | Record created in the production system at the time of press-fit operation; minted at operation start and immutable after inspector sign-off |
| Characteristic | Characteristic record minted in InspectAI when a drawing callout is extracted from a part_specification revision (AI-extracted or manually entered); UUID shared with QIF plan |
| Measurement Result | Result record minted in InspectAI when a measurement reading is recorded for an inspection event; UUID propagated to QIF results document |
| PPAP Submission | Submission record minted in InspectAI when a PPAP package is initiated for a part number; UUID used for customer portal submission tracking |
| FMEA Item | FMEA row record minted in InspectAI when a PFMEA or DFMEA is authored; UUID referenced by control_plan_item and ppap_submission elements |
| Control Plan Item | Control plan row record minted in InspectAI when a control plan is authored; UUID referenced in APQP documents submitted to OEM customers |
| SPC Result | SPC statistics record minted in InspectAI when an SPC calculation run is completed for a characteristic; recalculated (new record) when measurement data is updated |

### Format: Standard UUID v4

All UUIDs follow RFC 4122 v4 (randomly generated). Format: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`.

Example: `a3f7c2d1-8b4e-4a2f-9e1c-0d6b5a3f8c91`

---

## Translation Survival: How UUIDs Cross Format Boundaries

Each standard has its own identity scheme. The UUID must be carried explicitly as a cross-reference field — not replacing the native ID, but supplementing it.

### STEP AP242
STEP files use internal instance IDs (`#12345`) that are file-local. Customer STEP packages may include a product UUID in the `PRODUCT` entity. the manufacturer captures:
- The customer's part identifier (CAGE code + P/N + revision) as the canonical part reference
- Customer-assigned UUID if present in the STEP file
- Mapped to production system item master UUID on first receipt

### MTConnect
MTConnect work orders are referenced via `<WorkOrder>` elements in the MTConnect device XML. the manufacturer's MTConnect agent configuration maps:
```
<WorkOrder>
  <WorkOrderId>[Job UUID]</WorkOrderId>
  <PartNumber>[Part Number]</PartNumber>
  <Revision>[Revision]</Revision>
  <SerialNumber>[Serial UUID]</SerialNumber>
  <OperationId>[Job Operation UUID]</OperationId>
</WorkOrder>
```

This makes production job UUIDs the native work order reference in the machine data stream.

### QIF
QIF measurement plans and results include `<Id>` elements throughout the schema. the manufacturer's QIF output maps:
- `<MeasurementPlanId>` → the production system Inspection Event UUID (minted before the CMM run begins)
- `<JobId>` (QIF extension field) → the production system Job UUID
- `<SerialNumber>` → the production system Serial Number UUID
- `<PartDesignationId>` → Customer part reference from STEP AP242

### X12 EDI
X12 transaction sets use PO numbers and line item numbers as primary keys. The production job UUID is carried in:
- **X12 856 (ASN):** `REF*ZZ*[Job UUID]` reference qualifier segment
- **X12 810 (Invoice):** `REF*ZZ*[Job UUID]` reference qualifier segment
- **X12 850 inbound (PO):** customer's PO number is stored in the production system and linked to the job UUID

---

## UUID Storage in the production system Pro

the production system's custom fields (JSONB) are the primary storage mechanism for UUID cross-references until the production system exposes a native UUID field at each entity level. Field naming convention:

| the production system Entity | Custom Field Name | Stores |
|---------------|------------------|--------|
| Job | `job_uuid` | Canonical job UUID (= production system's internal job ID where possible) |
| Job Operation | `op_uuid` | Operation UUID for MTConnect work order reference |
| Serial/Lot | `ffm_serial_uuid` | Serial number UUID |
| Item | `ffm_part_uuid` | Part design UUID; `ffm_customer_part_ref` for customer's CAGE+P/N |
| Inventory Lot | `ffm_lot_uuid` | Material lot UUID |
| Receipt | `ffm_inspection_event_uuid` | Receiving inspection UUID |

---

## Validation Rules

1. A MTConnect work order record with no matching production job UUID is an orphan — flag for investigation.
2. A QIF result set with no matching Inspection Event UUID in the production system cannot be linked to a serial number — block cert package assembly.
3. A cert package cannot be finalized unless all serial numbers in the shipment have QIF result UUIDs resolving to passing inspection events.
4. Material lot UUIDs must be present on all job material issues — traceability chain is broken without them.

---

## Reference

- Hardwick, M. et al. (RPI / DMDII) — Federated digital thread research on UUID persistence across STEP, MTConnect, and QIF format boundaries. DMDII Project 14-05-02.
- NIST AMS 300-1 (2017) — Identifies persistent artifact identity as a requirement for digital thread coherence.
- MTConnect 2.2 specification — `<WorkOrder>` element definition.
- ISO 23952:2020 (QIF 3.0) — `<Id>` element usage throughout measurement plan and results schemas.
