# Residual Gaps and Deferred Work

Tracked items that are intentionally deferred, not oversights.

---

## Phase 3.3 — State Machine Completion

**Complete as of 2026-04-22**

State machine transition graphs added for all lifecycle-bearing entities that had status enums but no transition graph. Confirmed existing state machines for `purchase_order` and `customer_purchase_order` (already authored in Phase 2.E). New state machines authored:

- `outside_process_operation` (`schemas/outside_process_operation.yaml`): Scheduled → ShippedOut → AtVendor → Returned → Complete; terminal: Rejected, Cancelled. Cert verification guard on Returned → Complete.
- `first_article_inspection` (`schemas/quality.yaml`): Open → Submitted → Approved / Conditionally_Approved / Rejected; form completeness guard on submission. Terminal: Approved, Conditionally_Approved, Rejected.
- `nonconformance_report` (`schemas/quality.yaml`): Open → Pending_MRB → Rework_In_Progress / Pending_Customer / Closed / Scrapped; immediate-scrap path; customer-deviation path. Terminal: Closed, Scrapped.

Material_lot state machine was confirmed complete as a top-level block in `schemas/material.yaml` (12 transitions, illegal_transitions block).

**Matrix updated:** State machine column tally revised to 11 ✅ / 2 🟡 / 3 🔴 / 66 ⬜.

**Remaining 🔴 (deferred to Phase 3.4+):**
- `calibration_record`: assembly-domain entity, schema deferred to Phase 3.4
- `quality_flowdown_plan`: schema has no top-level `status` field yet; matrix note says "enumerate (Draft → Active → Amended → Closed)" — requires attribute addition before state machine

---

## Phase 3.4 — Assembly Domain Schema Authoring

**Complete as of 2026-04-23**

All 12 unscoped assembly-domain entities have been resolved: 9 received new schemas, 3 resolved as aliases or covered.

**New schemas authored:**

`schemas/assembly.yaml`:
- `torque_specification` — template defining target torque, tolerance, torque_unit [in_lb/ft_lb/Nm/cNm], lube_condition, safetying_required/method (AS5272 / NASM 33540 FKs)
- `torque_sequence` — fastener tightening order pattern (Star, Cross, Sequential, Clockwise, Custom); pass_count and step_percentages[]
- `torque_readings_record` — immutable execution record; per-fastener readings[] array; FK to tool_gauge (torque wrench used)
- `witness_inspection` — sign-off record (assembly_operation + inspector user FK + result + signoff_method); no state machine (terminal on save)
- `kit_record` — pre-assembly kit verification; kit_contents[] JSONB; complete flag gates operation start

`schemas/inspection.yaml`:
- `inspection_event` — UUID envelope for multi-evidence inspection sessions (bundles pmi_event + QIF results + attachments); value stated in schema header
- `calibration_record` — immutable audit record per calibration event; no state machine (lifecycle on tool_gauge); result [Pass/Fail/Adjusted]; as_found_in_tolerance field
- `tool_gauge` — calibration lifecycle carrier; state machine: Current → Due_Soon → Overdue → Quarantined → Retired (10 transitions); mtconnect_tool_asset_id bridge
- `key_characteristic` — polymorphic FK (subject_id + subject_type: [part, assembly]) per Decision 1.9; characteristic_type [KC/Safety_Critical/Dimensional/GD_T/Functional/Regulatory]; qif_characteristic_id FK

**Resolved as aliases/covered:**
- `fai_package` → *(alias — schemas/certification_package.yaml (includes_fai: true))*; certification_package already carries fai_package_id field
- `quality_engineer` → *(alias — schemas/user.yaml (role: quality_engineer))*; per Decision 1.8; role guard enforced at application layer
- `process_identification` → *(covered — schemas/material_identification.yaml)*; callout_type enum covers all AS9102 Form 2 process lines (HeatTreat, Plating, Coating, NDT, Passivation, Anodize, Weld, Other_Process)

**Architectural decisions recorded in schema headers:**
- calibration_record is IMMUTABLE; lifecycle belongs on tool_gauge.calibration_status, not on the record
- inspection_event is a UUID ENVELOPE (not a supertype of pmi_event); its value is providing a shared parent FK for multi-evidence sessions
- torque_specification, torque_sequence, and torque_readings_record are three distinct entities, each referenced independently by assembly_operation

**Matrix updated:** Schema defined 61 → 73 ✅ / 21 → 9 🔴. State machine 11 → 12 ✅; calibration_record corrected 🔴 → ⬜.

**Remaining deferred (Phase 3.5 candidates):**
- `quality_flowdown_plan` state machine: schema present but no `status` field yet; requires attribute addition then transition graph
- CAPA standalone entity: currently embedded in NCR; AS9100-conforming QMS implementations typically have a standalone CAPA table (see AS9100 QMS Alignment Review below)
- SDR (Supplier Deviation Request, AS9100 8.7.2) standalone entity: not modeled in ontology
- Cross-ref integrity: Phase 2–4 entities not yet added to UUID entity table in `extensions/uuid-discipline.md`

---

## AS9100 QMS Alignment Review — Deferred

**Deferred — pending review**

Alignment gaps identified between the ontology's Layer 2 model and a conforming AS9100D QMS implementation reviewed in April 2026.

**Alignment gaps identified:**

| Ontology Entity | Ontology Status Model | AS9100 QMS Pattern | Gap |
|---|---|---|---|
| `nonconformance_report` | Open → Pending_MRB → Rework_In_Progress → Closed / Scrapped | draft → containment → investigation → disposition → corrective_action → verification → closed | Different granularity; QMS follows AS9100 process flow; ontology follows MRB disposition flow. Both valid for their layers. |
| `nonconformance_report` | `mrb_status` embedded sub-object | CAPA entity as separate table | Ontology embeds MRB; a conforming QMS typically has a standalone CAPA entity. Consider whether CAPA needs a Phase 3.5 standalone entity. |
| `nonconformance_report` | `corrective_action_verified` boolean | `corrective_actions` junction table (links NC or CAPA) | AS9100-compliant implementations often use a richer corrective action model. |
| Missing entity | — | `sdr` (Supplier Deviation Request — AS9100 8.7.2) | SDR is not modeled in the ontology. Candidate for Phase 3.5. |
| Missing entity | — | `rma` | `shipment` with direction=return covers inbound; a full RMA entity adds richer customer contact / resolution tracking fields. May warrant a dedicated RMA entity or `rma_record` embedded in `shipment`. |

**Action items (not yet scheduled):**
1. Decide whether `CAPA` should be a standalone ontology entity (currently embedded in NCR).
2. Decide whether `SDR` (Supplier Deviation Request, AS9100 8.7.2) needs a standalone entity.
3. Review `nonconformance_report.status` enum alignment — ontology and conforming QMS implementations use different state models at different granularities. Decision: keep as-is at Layer 2 with annotation, or align enums to 7-state AS9100 process model.
4. Check whether a standalone RMA entity adds material coverage beyond `shipment` (direction=return).

---

## Phase 3.1 — De-Carbonize material.yaml

**Complete as of 2026-04-22**

`schemas/material.yaml` alloy_family enum replaced with AMS commodity group–grounded values. Changes:
- `alloy_family`: old Carbon-influenced values replaced with AMS range–grounded family names (Aluminum Alloy, Magnesium Alloy, Titanium Alloy, Precipitation Hardening Stainless, Austenitic Stainless, Martensitic Stainless, Nickel Alloy, Cobalt Alloy, Alloy Steel, Carbon Steel, Tool Steel, Copper Alloy, Beryllium Copper, Composite, Polymer). Each value annotated with AMS spec number range in the description field.
- `form` enum: added "Billet" with description distinguishing from Bar.
- `spec_body` enum: NASM added to both `material` and `material_specification` entities per Decision 1.7.

---

## Phase 3.2 — Matrix Cell Regeneration

**Complete as of 2026-04-22**

`reference/entity-matrix.md` updated to reflect all Phase 2.A–2.F schema additions. Schema defined cells updated from 🔴 to ✅ for ~60 newly schematized entities (standalones, aliases, and embedded sub-objects). Entity count corrected from 80 to 82 (receipt and work_order added as missing sections; approved_supplier_list_entry and supplier_qualification_event renamed). Tally updated. Phase 3.2 section headings annotated with completion dates.

---

## Matrix Cell Regeneration — Phase 2.A Entities

**Complete — Phase 3.2**

The entity matrix has been updated. The following entities had new/upgraded schema definitions (see Phase 3.2 summary above):

| Entity | Schema | What Changed |
|--------|--------|--------------|
| `certification_document` | `schemas/certification_document.yaml` | New discriminated supertype; replaces mtr/coc/dfars_declaration/pmi_report/conflict_minerals_declaration/shelf_life_certification as separate entities (Decision 1.6) |
| `material_condition` | `schemas/material_condition.yaml` | Elevated from string/enum to first-class entity (Decision 1.7); full mechanical property fields |
| `material_identification` | `schemas/material_identification.yaml` | New schema; AS9102 Form 2 line-item record |
| `pmi_event` | `schemas/pmi_event.yaml` | New schema; XRF/OES/ICP instrument record at receiving inspection |
| `material_lot` | `schemas/material.yaml` | State machine appended; new relationships (verified_by pmi_event, supplied_by supplier) |
| `material_specification` | `schemas/material.yaml` | New relationships block (has_condition material_condition) |

**Why deferred:** Matrix cell content (layer coverage, completeness dimensions, open questions) requires a full regen pass across all 80 entities. Doing a partial regen mid-cluster risks inconsistency. Phase 3.2 is the dedicated matrix regen pass after all Phase 2 schemas are complete.

---

## Alias Entities — No Standalone Schema Needed

Per Decision 1.6, the following entities are `cert_type` values within `certification_document`, not standalone entities. They carry `*(alias …)*` in the INDEX schema column and do not need their own schema files:

- `mtr` (cert_type: MTR)
- `coc` (cert_type: CoC)
- `dfars_declaration` (cert_type: DFARS_Declaration)
- `pmi_report` (cert_type: PMI_Report)
- `conflict_minerals_declaration` (cert_type: ConflictMineral)
- `shelf_life_certification` (cert_type: ShelfLife)

---

## Matrix Cell Regeneration — Phase 2.B Entities

**Complete — Phase 3.2**

| Entity | Schema | What Changed |
|--------|--------|--------------|
| `part` | `schemas/part.yaml` | New schema; part_kind enum (Decision 1.2); explicit ITAR/export fields |
| `assembly` | `schemas/part.yaml` | New schema; distinct from part (Decision 1.1); torque_specification and assembly_routing FKs |
| `part_specification` | `schemas/part.yaml` | New schema; drawing number + CAGE + MBD flag + STEP AP242 file ref |
| `bom_line` | `schemas/part.yaml` | New schema; polymorphic parent/child FKs with discriminator enums; find number and effectivity |
| `export_classification` | `schemas/export_classification.yaml` | New schema; ITAR/EAR classification with ECCN, USML category, exemptions |
| `customer_approval` | `schemas/customer_approval.yaml` | New schema; approval_status and fai_status enums; qualification_method |

---

## Matrix Cell Regeneration — Phase 2.C Entities

**Complete — Phase 3.2**

| Entity | Schema | What Changed |
|--------|--------|--------------|
| `routing` | `schemas/routing.yaml` | New schema; routing template for a part's process plan |
| `operation` | `schemas/routing.yaml` | New schema; in-house routing operation template (distinct from work_order execution instance) |
| `work_center` | `schemas/routing.yaml` | New schema; named machine/station with MTConnect device ID linkage |
| `assembly_routing` | `schemas/routing.yaml` | New schema; assembly process plan with torque/witness fields |
| `assembly_operation` | `schemas/routing.yaml` | New schema; assembly step enriched with fastener, torque, witness attributes; `witnessed_by` cardinality updated to one-to-many |
| `outside_process_operation` | `schemas/outside_process_operation.yaml` | New schema; hybrid routing-op/PO/cert-collection entity |
| `process_specification` | `schemas/outside_process_operation.yaml` | New schema; AMS/MIL/ASTM process spec as first-class entity |
| `process_certification` | `schemas/outside_process_operation.yaml` | New schema; vendor process conformance cert (distinct from material certification_document) |
| `job` | `schemas/job.yaml` | New schema; production run container; subject_id/subject_type polymorphic FK pattern |
| `serial_number` | `schemas/job.yaml` | New schema; individual unit with lifecycle, material lot traceability, QIF/NCR links |
| `work_order` | `schemas/job.yaml` | New schema; execution instance of routing operation on a job; machine_event edges migrated here from `operation` template |

**Scope note:** Phase 2.C original scope was 6 entities (outside_process_operation, routing, operation, work_center, job, serial_number). Phase 2.C delivered 11: assembly_routing and assembly_operation added because they belong in routing.yaml alongside routing/operation; work_order added because it was missing from the initial INDEX inventory despite being in uuid-discipline.md.

**Graph migration:** `job → operation (contains_job_operation)` edge and `operation → machine_event (produces)` edge migrated in Phase 2.C. `job → work_order (contains_work_order)` and `work_order → machine_event (produces_events)` are the replacements. The old archetype-source edges are annotated with comments in relationship-graph.yaml.

---

## Matrix Cell Regeneration — Phase 2.D Entities

**Complete — Phase 3.2**

| Entity | Schema | What Changed |
|--------|--------|--------------|
| `quality_clause` | `schemas/quality.yaml` | New schema; reusable quality obligation with category enum, subtier cascade flag, and job-requirement triggers |
| `quality_flowdown_plan` | `schemas/quality.yaml` | New schema; machine-readable quality addendum for a job; drives cert package and subtier PO flowdowns |
| `first_article_inspection` | `schemas/quality.yaml` | New schema; AS9102 three-form FAI; subject_id/subject_type replaces item_id (Decision 1.9); as9102_form_1/2/3 embedded as sub-objects |
| `nonconformance_report` | `schemas/quality.yaml` | New schema; NCR + embedded MRB disposition; subject_id/subject_type replaces item_id (Decision 1.9) |

**item_id migration:** `extensions/quality-flowdowns.md` `item_id` fields in both entities replaced with `subject_id + subject_type: enum [part, assembly]`. The pending fix noted in the item-entity removal section is now resolved.

**Embedded sub-objects (no standalone UUIDs):** `as9102_form_1`, `as9102_form_2`, `as9102_form_3` are sub-objects within `first_article_inspection.form1/form2/form3`. `mrb_disposition` is embedded within `nonconformance_report`. INDEX entries for these four now reference `schemas/quality.yaml` with embedded notation.

---

## Matrix Cell Regeneration — Phase 2.E Entities

**Complete — Phase 3.2**

| Entity | Schema | What Changed |
|--------|--------|--------------|
| `supplier` | `schemas/supplier.yaml` | New schema; contact/capability record; renamed from `vendor` per Decision 1.3 |
| `approved_supplier_list_entry` | `schemas/approved_supplier_list_entry.yaml` | New schema; supplier qualification record with Nadcap refs, customer approvals, ITAR status; renamed from `approved_vendor_list_entry` per Decision 1.3 |
| `customer` | `schemas/customer.yaml` | New schema; persistent customer record with ITAR screening, DoD prime flag, customer_asl refs |
| `purchase_order` | `schemas/purchase_order.yaml` | New schema; outbound PO with po_kind discriminator and state machine; sub_tier_purchase_order is alias with po_kind: sub_tier per Decision 1.4 |
| `customer_purchase_order` | `schemas/customer_purchase_order.yaml` | New schema; inbound X12 850 with state machine; quality flowdown cascade origin |
| `shipment` | `schemas/shipment.yaml` | New schema; direction-discriminated entity (outbound/inbound/return) with two state machines per Decision 1.5 |
| `receipt` | `schemas/receipt.yaml` | New schema; goods-received event; triggers inventory movement, receiving inspection, cert collection |
| `nadcap_certification` | `schemas/nadcap_certification.yaml` | New schema; standalone PRI Nadcap certificate; extracted from embedded array in approved_supplier_list_entry |
| `nadcap_category` | `schemas/nadcap_category.yaml` | New schema; first-class reference entity per Decision 1.7; replaces free-text string in outside_process_operation |
| `supplier_qualification_event` | `schemas/supplier_qualification_event.yaml` | New schema; audit-trail entity for supplier approval lifecycle events; renamed from `vendor_qualification_event` per Decision 1.3 |
| `customer_asl` | `schemas/customer_asl.yaml` | New schema; customer-provided Approved Source List; resolves OQ-03 (customer_asl is the customer's document; approved_supplier_list_entry is the manufacturer's per-vendor verification record) |

**OQ-03 resolution:** `customer_asl` represents the customer's authoritative ASL document (imported from PDF, portal, or contract attachment). the manufacturer's per-vendor approval records live in `approved_supplier_list_entry.customer_approvals[]`. The two cross-reference via `customer_asl.entries[].supplier_id → supplier.id`.

**Rename cascade status (complete as of Phase 2.E close-out):** All `vendor_` field names and relationship verb fragments have been cascaded to `supplier_` equivalents across schema files and the relationship graph. Specific fields renamed:
- `schemas/outside_process_operation.yaml`: `vendor_job_number` → `supplier_job_number`; relationship verbs `has_vendor` → `has_supplier`, `tracks_items_at_vendor` → `tracks_items_at_supplier`
- `schemas/supplier.yaml`: `vendor_procedure` → `supplier_procedure`
- `schemas/quality.yaml`: `vendor_id` → `supplier_id` (in subtier_flowdowns sub-object)
- `extensions/outside-processing.md`: `items_at_vendor` → `items_at_supplier`; `flowed_to_vendor_po` → `flowed_to_supplier_po`; `vendor_procedure` → `supplier_procedure`
- `extensions/quality-flowdowns.md`: `vendor_id` → `supplier_id`; prose "sent to vendor" → "sent to supplier"
- `reference/relationship-graph.yaml`: verb fragments `has_vendor` → `has_supplier`, `tracks_items_at_vendor` → `tracks_items_at_supplier`, `qualifies_vendor` → `qualifies_supplier`, `flows_to_subtier_vendor` → `flows_to_subtier_supplier`

Intentional retentions: extension prose narrative text uses "vendor" in human-readable descriptions (acceptable — narrative context, not entity or field names). `entity-matrix.md` and `entity-inventory.yaml` still carry pre-rename entries; deferred to Phase 3.2 matrix regen. `implementations/fulcrum.md` maps Layer 2 canonical names to the production system Layer 3 names; deferred to Phase 3.2.

**process_certification note:** The plan listed `schemas/process_certification.yaml` as a new file. Process_certification is already defined in `schemas/outside_process_operation.yaml` (authored in Phase 2.C). No standalone file created — existing schema is authoritative.

---

## Matrix Cell Regeneration — Phase 2.F Entities

**Complete — Phase 3.2**

| Entity | Schema | What Changed |
|--------|--------|--------------|
| `sales_order` | `schemas/sales_order.yaml` | New schema; the manufacturer acceptance record derived from customer_purchase_order; state machine Draft → Closed; line_items[] with subject_id/subject_type (Decision 1.9) |
| `certification_package` | `schemas/certification_package.yaml` | New schema; machine-readable cert document assembly for a shipment; supports assembly nesting via contained_package_ids[]; links material_certs, process_certs, FAI, CMM results, the manufacturer CoC |
| `inventory_location` | `schemas/inventory_location.yaml` | New schema; ISA-95-grounded location hierarchy; ITAR-segregated flag; Off_Site type for outside processor tracking; self-referential parent_location_id |
| `user` | `schemas/user.yaml` | New schema; shop floor and office users; citizenship_status for ITAR access control; roles[] enum; training_records[] for quality signatory eligibility |
| `data_classification` | `schemas/data_classification.yaml` | New schema; classification rule record implementing ip-boundary.md Data Classification Matrix; classification_level enum; retention_years; ITAR flag; access_roles[] |
| `job` | `schemas/job.yaml` | State machine added (Planned → Closed); guards document release readiness and completion requirements; no attribute changes |
| `serial_number` | `schemas/job.yaml` | State machine added (Assigned → Scrapped); RMA return path Shipped → Returned → Quarantined; no attribute changes |

**Plan scope note:** The plan listed `schemas/job.yaml` and `schemas/serial_number.yaml` as new files in Phase 2.F. Both were authored in Phase 2.C. Phase 2.F delivered their state machines and the five new schema files above.

---

## item Entity — Removed from Canon

Per Decision 1.9, `item` is removed from the canonical Layer 2 ontology. It is an ERP implementation concept with no Layer-1 justification. Extensions that reference `item_id` must be updated to use `part | assembly` as a typed union.

**Fixed in Phase 2.D:** `extensions/quality-flowdowns.md` `item_id` fields in `first_article_inspection` and `nonconformance_report` replaced with `subject_id` + `subject_type: enum [part, assembly]`.

---

## Stale Machine-Readable Files — RESOLVED 2026-04-24

Both machine-readable source files updated 2026-04-24.

### `reference/entity-inventory.yaml`

**Resolved.** Updated from 82 to 117 entries. Phase 2 added 29 new entity entries (Domains 9–12, Assembly Hardware, Operator Qualification); InspectAI added 6 (characteristic, measurement_result, fmea_item, control_plan_item, spc_result, ppap_submission). Nine stale `has_schema: null` entries corrected: calibration_record, inspection_event, key_characteristic, tool_gauge (→ schemas/inspection.yaml); kit_record, torque_readings_record, torque_sequence, torque_specification, witness_inspection (→ schemas/assembly.yaml).

**Note on count:** True entity count is 117 (82 original + 29 Phase 2 + 6 InspectAI), not 123 as temporarily stated in prior session notes. The 35-entity Phase 2 figure was inflated; the actual new-entity count is 29 (the other 6 were entities already in the original 82 that received schemas in Phase 2).

### `reference/relationship-graph.yaml`

**Resolved.** Updated from 137 to 355 edges (+218). Edges extracted from all Phase 2 and InspectAI schema `relationships:` blocks across 11 schema files. Ten new sections added covering: Process Engineering/NRE, Tool Room, Packaging, Change Management, Assembly Hardware, Operator Qualification, Assembly Phase 2 entities, Inspection Phase 2 entities, AIAG Quality Tools, and Dimensional Inspection (InspectAI).

---

## Open Questions — Carry Forward

| ID | Entity | Question | Status |
|----|--------|----------|--------|
| OQ-01 | `export_compliance_declaration` | Does this need a schema or does `certification_document` cert_type: ExportCompliance cover it fully? | **Resolved (Phase 2.B):** `certification_document` cert_type: ExportCompliance covers it. `export_compliance_fields` sub-schema added in `schemas/certification_document.yaml`. `schemas/export_classification.yaml` header explicitly notes the distinction: export_classification classifies an article; the certification_document records a shipment-level compliance declaration. No standalone schema needed. |
| OQ-02 | `material_identification` | Should `form_2_id` be a FK to a `fai_form_2` entity, or is the FAI package the top-level anchor? | **Resolved in Phase 2.D (schema fixed):** FAI package is the top-level anchor. `as9102_form_2` is an embedded sub-object with no independent UUID. `schemas/material_identification.yaml` field renamed `form_2_id` → `fai_id`; FK now points to `first_article_inspection.id`. Relationship-graph edge updated from `as9102_form_2 → material_identification` to `first_article_inspection → material_identification (has_form2_line)`. |
| OQ-03 | `customer_asl` | Is `customer_asl` the customer's view of an ASL or the manufacturer's internal ASL keyed to a customer? Supertype relationship to `approved_supplier_list_entry` unclear. | **Resolved in Phase 2.E:** `customer_asl` is the customer's document (ASL document or portal reference). `approved_supplier_list_entry.customer_approvals[]` is the manufacturer's per-vendor verification record. Cross-reference: `customer_asl.entries[].supplier_id → supplier.id`. See `schemas/customer_asl.yaml` and `schemas/approved_supplier_list_entry.yaml`. |
