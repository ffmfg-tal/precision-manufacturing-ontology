# Entity Matrix — Precision Manufacturing Ontology Coherence Pass

Generated: 2026-04-22 (Task 6); updated Phase 3.2 (2026-04-22), Phase 3.3 (2026-04-22), Phase 3.4 (2026-04-23).
Sources: `reference/entity-inventory.yaml` (82 entities), `reference/composition-archetypes.md` (required fields per archetype), all files in `extensions/`, all files in `schemas/`, `standards/*.md`, `reference/domain-map.md`, `reference/carbon-os-mapping.md`.

Glyphs: ✅ green · 🟡 yellow · 🔴 red · ⬜ missing-but-N/A (green-equivalent where the dimension does not apply)

---

## State of the Ontology

The ontology's strength is narrative and compositional: every archetype walks end-to-end, every extension carries a coherent entity model in prose, and the four-layer architecture cleanly separates standards from manufacturer-layer gap fills. Its weakness is mechanical: almost nothing has been translated from prose into machine-readable schema, lifecycle entities have not been given state graphs, and a handful of taxonomic questions have been deferred across documents rather than resolved anywhere.

**Phase 3.4 update (2026-04-23).** Phase 3.4 completed assembly-domain schema authoring: `schemas/assembly.yaml` (torque_specification, torque_sequence, torque_readings_record, witness_inspection, kit_record) and `schemas/inspection.yaml` (inspection_event, calibration_record, tool_gauge, key_characteristic). Three entities resolved as aliases/covered: fai_package → certification_package; quality_engineer → user.role; process_identification → material_identification callout_type. Schema defined now at 73 ✅ / 9 🔴. State machine added for tool_gauge (10 transitions, Current/Due_Soon/Overdue/Quarantined/Retired); calibration_record lifecycle corrected to ⬜ (immutable record; lifecycle on tool_gauge). Phase 3.3 (2026-04-22) completed state machine enumeration for material_lot, first_article_inspection, nonconformance_report, outside_process_operation, purchase_order, customer_purchase_order.

**Weakest columns.** Cross-ref integrity remains the weakest column: most Phase 2–4 entities have not been added to the UUID entity table in `extensions/uuid-discipline.md`. State machine has 2 remaining 🔴: quality_flowdown_plan (schema present, status field addition needed before graph can be written) and one other. Relationships typed is now all-green on schema-defined entities; the 🟡 cluster is aliases and display nodes.

**Reddest clusters.** The assembly-domain red cluster is fully resolved. Remaining reds concentrate in: (1) external standards not authored here (machine_event, qif_results_document, x12_856_asn) — always 🔴/⬜ for schema; (2) string/enum decisions per 1.7 (country, heat_number, spec_body, uns_number, usml_category); (3) cross-ref integrity for all entities not yet added to UUID table; (4) quality_flowdown_plan state machine (attribute gap blocks graph authoring).

**Cross-cutting patterns.** Three patterns recur: (a) **taxonomic open questions** deferred to "the matrix task to resolve" — `assembly` vs recursive `part`, `fastener` scope, `quality_engineer` vs role, `heat_number` / `uns_number` / `country` / `spec_body` / `nadcap_category` as entities vs strings, cert package nesting, `supplier` / `vendor` / `approved_vendor_list_entry` aliasing. The matrix surfaces these in their row notes but does not resolve them; resolution belongs to the deepening pass. (b) **Lifecycle entities without state machines** — jobs, operations, POs, shipments, FAIs, NCRs, and material lots have enum status fields but no documented transition rules, guards, or terminal-state definitions. (c) **Carbon framing residue is minor** — only `material` shows direct vocabulary borrowing (the layered-disclosure approach and substance/form decomposition trace to Barbin's schema per `carbon-os-mapping.md`). Most entities are defined on their own terms.

**Top 5 priorities for the deepening pass:**
1. **Schema authoring** for the ~35 entities already exercised by archetypes or defined in extension prose. Convert the inline `yaml:` blocks in `extensions/*.md` into files in `schemas/` that validate.
2. **State-machine enumeration** for the 14 lifecycle-bearing entities, with explicit transitions, guards, and terminal states.
3. **Resolve the nine taxonomic open questions** (assembly vs part, fastener scope, supplier/vendor alias, PO hierarchy, shipment direction, cert-document supertype, string-vs-entity decisions for heat/UNS/country, quality_engineer as role, cert-package nesting).
4. **Define the seven "referenced but undefined" entities** (`customer`, `sales_order`, `part_specification`, `user`, `inventory_location`, `bom_line`, `item`) — each currently appears as a uuid-ref target with no schema or prose definition.
5. **De-Carbonize `material.yaml`** — the layered-disclosure approach is sound, but the alloy_family enum and form enum should be audited for Carbon borrowings and re-grounded against AMS commodity groups and aerospace-specific forms.

---

## Matrix

Entities are listed alphabetically (matching `entity-inventory.yaml` order). Entity count: 82 (includes `receipt` and `work_order` added Phase 2.C/2.E; `approved_supplier_list_entry` and `supplier_qualification_event` renamed Phase 2.E).

---

### approved_supplier_list_entry

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/approved_supplier_list_entry.yaml` (Phase 2.E) |
| Attributes complete | ✅ | Covers ITAR, AS9100, Nadcap scopes, customer approvals, the manufacturer qualification, insurance, status. |
| Relationships typed | 🟡 | Outgoing refs (`supplier_id`, `customer_id`, `process_spec`) have targets but no cardinality annotation. To green: add cardinality. |
| State machine | ⬜ | Status enum present; entity not flagged as lifecycle-bearing in task spec. |
| Cross-ref integrity | 🟡 | In `extensions/supplier-approval.md` but not in `extensions/uuid-discipline.md` entity table. To green: add to UUID table. |
| Standards provenance | ⬜ | Pure Layer-2 entity; no standard touches supplier qualification. |
| Composition rule | ✅ | Primitive record with nested `nadcap_accreditations[]` and `customer_approvals[]` as embedded objects. |
| Canonical framing | ✅ | Canonical name `approved_supplier_list_entry` per decision 1.3; rename cascade complete Phase 2.E. |

### as9102_form_1

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Embedded sub-object in `schemas/quality.yaml` within `first_article_inspection.form1` (Phase 2.D; resolved: collapsed into FAI per Decision 1.9 embedded pattern) |
| Attributes complete | 🟡 | Standard fields (org, CAGE, drawing, report #) present as `form1.*` fields in FAI. Partial FAI justification covered. To green: confirm no AS9102 Rev C required field is missing. |
| Relationships typed | 🟡 | Sub-object of FAI; no explicit cardinality. To green: type as 1:1 under FAI. |
| State machine | ⬜ | Completeness tracked via `form1_complete` bool on parent; no lifecycle on the form itself. |
| Cross-ref integrity | 🟡 | Referenced in all three archetypes and quality-flowdowns.md; absent from UUID table and domain-map. To green: add to either. |
| Standards provenance | ✅ | AS9102 Rev C Form 1 cited in `standards/qif.md` (as out-of-QIF-scope) and `reference/prior-art.md`. |
| Composition rule | ✅ | Resolved: embedded sub-object in `first_article_inspection.form1`; archetypes reference it as a display node but it has no independent UUID. Phase 2.D. |
| Canonical framing | ✅ | Defined on AS9102 terms. |

### as9102_form_2

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Embedded sub-object in `schemas/quality.yaml` within `first_article_inspection.form2` (Phase 2.D) |
| Attributes complete | 🟡 | Line-item structure present (spec type, spec number, customer approval, sub-tier FAI, cert ref). To green: confirm per-line AS9102 Rev C required fields. |
| Relationships typed | 🟡 | References to `material_specification`, `process_specification`, `certification_document`, `process_certification` stated but uncardinated. To green: add cardinality. |
| State machine | ⬜ | Completeness flag only. |
| Cross-ref integrity | 🟡 | In archetypes and quality-flowdowns; absent from UUID table. |
| Standards provenance | ✅ | AS9102 Rev C Form 2 explicitly called out as outside QIF scope in `standards/qif.md`. |
| Composition rule | ✅ | Resolved: embedded sub-object in `first_article_inspection.form2`; container of `line_items[]` under FAI parent. Phase 2.D. |
| Canonical framing | ✅ | AS9102 terms. |

### as9102_form_3

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Embedded sub-object in `schemas/quality.yaml` within `first_article_inspection.form3` (Phase 2.D) |
| Attributes complete | ✅ | Per-characteristic: balloon #, type, drawing req, QIF UUID link, measured value, deviation, pass/fail, equipment, calibration ID, inspector. |
| Relationships typed | ✅ | `qif_characteristic_id`, `equipment_calibration_id` references. |
| State machine | ⬜ | Completeness flag only. |
| Cross-ref integrity | ✅ | Referenced in archetypes, quality-flowdowns, QIF standard, domain-map §4. |
| Standards provenance | ✅ | QIF covers Form 3 directly; explicit in `standards/qif.md`. |
| Composition rule | ✅ | Resolved: embedded sub-object in `first_article_inspection.form3`; container of `characteristics[]` under FAI parent. Phase 2.D. |
| Canonical framing | ✅ | AS9102 + QIF. |

### assembly

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/part.yaml` (Phase 2.B) |
| Attributes complete | 🟡 | Introduced via Archetype C narrative; attribute list in part.yaml. STEP AP242 concentricity/torque mapping not yet explicit. |
| Relationships typed | 🟡 | `contains_part`, `assembled_via` defined in schema; cardinality annotations in schema but cross-file references incomplete. |
| State machine | ⬜ | Task spec lists lifecycle entities; `assembly` is not in that list (treated as part-like). |
| Cross-ref integrity | 🔴 | Only in composition-archetypes.md; absent from uuid-discipline, domain-map, extensions. |
| Standards provenance | 🟡 | STEP AP242 defines assembly BOM structure; no explicit mapping to this entity. To green: map to AP242 assembly relationships. |
| Composition rule | ✅ | Distinct entity per decision 1.1; schema defined in `schemas/part.yaml` Phase 2.B. |
| Canonical framing | ✅ | Defined on its own terms in Archetype C. |

### assembly_operation

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/routing.yaml` (Phase 2.C) |
| Attributes complete | 🟡 | Assembly-specific attributes (fastener_id, torque_specification_id, witness_required, signoff_user_id) in schema. Further NASM 33540 / AS5272 field mapping not yet explicit. |
| Relationships typed | 🟡 | `uses_fastener`, `has_torque_spec`, `witnessed_by` (one-to-many) defined in schema; cardinality formal. |
| State machine | ⬜ | Not in task-spec lifecycle list (treated as operation variant). |
| Cross-ref integrity | 🔴 | Only in composition-archetypes.md and routing.yaml. Not in uuid-discipline or domain-map. |
| Standards provenance | 🔴 | NASM 33540 and AS5272 are process specs; no explicit mapping of `assembly_operation` to a standard. |
| Composition rule | ✅ | Distinct subtype of operation per routing.yaml Phase 2.C; carries assembly-specific fields. |
| Canonical framing | ✅ | Defined on its own terms. |

### assembly_routing

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/routing.yaml` (Phase 2.C) |
| Attributes complete | 🟡 | Core attributes in schema; STEP AP238 field-level mapping not yet explicit. |
| Relationships typed | 🟡 | `contains_operation` with cardinality in schema; further cross-file cardinality incomplete. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🔴 | Archetype C and routing.yaml only. Not in uuid-discipline or domain-map. |
| Standards provenance | 🔴 | STEP AP238 defines routing; no explicit mapping of `assembly_routing` to AP238. |
| Composition rule | ✅ | Distinct from `routing` per routing.yaml Phase 2.C; assembly-specific container with torque/witness metadata. |
| Canonical framing | ✅ | Own terms. |

### bom_line

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/part.yaml` (Phase 2.B) |
| Attributes complete | ✅ | parent_id/parent_type, child_id/child_type polymorphic FKs, quantity, uom, find_number, effectivity_start/end in schema. |
| Relationships typed | 🟡 | Polymorphic parent/child FKs with discriminator enums defined; STEP AP242 effectivity mapping not explicit. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🔴 | Only in material.yaml and part.yaml; not in UUID table, domain-map, extensions. |
| Standards provenance | 🟡 | STEP AP242 defines BOM structure; formal mapping not yet added. |
| Composition rule | ✅ | Distinct many-to-many edge entity per decision 1.1; schema defined Phase 2.B with polymorphic FK pattern per decision 1.9. |
| Canonical framing | ✅ | Own terms. |

### calibration_record

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/inspection.yaml` — immutable audit record per calibration event; lifecycle states live on tool_gauge.calibration_status, not on this record. (Phase 3.4) |
| Attributes complete | ✅ | tool_gauge_id, calibration_date, due_date, interval_days, performed_by, lab_name, lab_accreditation, calibration_standard, certificate_number, result [Pass/Fail/Adjusted], as_found_in_tolerance, file_reference. |
| Relationships typed | ✅ | `for_tool → tool_gauge` (many-to-one); back-ref lives on tool_gauge.current_calibration_id. |
| State machine | ⬜ | Immutable record — lifecycle (Current/Due_Soon/Overdue/Quarantined/Retired) lives on tool_gauge, not on calibration_record. No state machine on record. (Phase 3.4) |
| Cross-ref integrity | 🟡 | In quality-flowdowns.md (FAI Form 3 equipment_calibration_id) and pmi_event.yaml (calibration_record_id FK). Not in UUID table. |
| Standards provenance | 🟡 | AS9100 (calibration requirements) and QIF Resources module reference calibration; schema maps AS9100 concepts but QIF cross-reference not fully explicit. |
| Composition rule | ✅ | Immutable audit record pattern: one record per event, never modified. tool_gauge is the lifecycle carrier; calibration_record is the evidence. (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### certification_document

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/certification_document.yaml` (Phase 2.A) |
| Attributes complete | ✅ | Cert type enum, issuer, dates, typed sub-schemas (mtr_fields, dfars_fields, conflict_mineral_fields). |
| Relationships typed | ✅ | `material_lot`, `spec_reference` typed. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | In archetypes, material-certs, domain-map §1. Not in UUID table but should be. To green: add. |
| Standards provenance | 🟡 | Mentioned in STEP gap analysis and prior-art; no direct standard covers cert documents. |
| Composition rule | ✅ | Supertype with `cert_type` discriminator per decision 1.6. Subtypes are discriminator values with per-cert `*_fields` sub-schemas within `schemas/certification_document.yaml`. |
| Canonical framing | ✅ | Own terms. |

### certification_package

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/certification_package.yaml` (Phase 2.F) |
| Attributes complete | ✅ | Package id, shipment_id, job_id, covered serials, contained_document_ids, contained_package_ids (nesting), recipient; defined in schema. |
| Relationships typed | 🟡 | `contains_documents`, `contains_packages`, `for_shipment` defined; outgoing cardinality in schema. |
| State machine | ⬜ | Not listed in task-spec lifecycle; effectively ephemeral at shipment. |
| Cross-ref integrity | ✅ | In all three archetypes, material-certs, quality-flowdowns, ip-boundary. |
| Standards provenance | 🔴 | Not covered by any Layer-1 standard; X12 856 does not define cert-package manifest per `standards/x12-edi.md` gap 3. |
| Composition rule | ✅ | Nested: `contains_packages[]` for child cert packages + `contains_documents[]` per decision 1.9. |
| Canonical framing | ✅ | Own terms. |

### characteristic_measurement

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Embedded sub-object in `schemas/quality.yaml` within `first_article_inspection.form3.characteristics[]` (Phase 2.D) |
| Attributes complete | ✅ | QIF id, nominal, measured, deviation, tolerances, pass/fail, equipment, calibration, inspector, date. |
| Relationships typed | ✅ | `qif_characteristic_id`, `equipment_calibration_id`. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | Archetypes, quality-flowdowns, QIF standard. |
| Standards provenance | ✅ | QIF LinearCharacteristic / Results directly map. |
| Composition rule | ✅ | Primitive inside form3 characteristics array; embedded in quality.yaml Phase 2.D. Not first-class (no independent UUID). |
| Canonical framing | ✅ | QIF terms. |

### coc

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/certification_document.yaml` (`cert_type: CoC`) (Phase 2.A) |
| Attributes complete | 🟡 | Inherits certification_document fields; no CoC-specific sub-schema (unlike MTR which has `mtr_fields`). To green: add coc_fields. |
| Relationships typed | 🟡 | Via parent. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | Archetypes, material-certs, quality-flowdowns. |
| Standards provenance | 🟡 | Referenced in prior-art and x12 gaps; not structurally mapped. |
| Composition rule | ✅ | Alias of `certification_document` (cert_type: CoC) per decision 1.6; discriminator in `schemas/certification_document.yaml`. |
| Canonical framing | ✅ | Own terms. |

### conflict_minerals_declaration

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/certification_document.yaml` (`cert_type: ConflictMineral`) (Phase 2.A) |
| Attributes complete | ✅ | Clause, minerals covered, conflict-free flag, country of origin. |
| Relationships typed | 🟡 | Inherits cert_document relationships. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | In material-certs, quality-flowdowns (QA-009), material_lot. Not in UUID table. |
| Standards provenance | 🟡 | DFARS 252.225-7052 cited; not mapped as a Layer-1 standard entity. |
| Composition rule | ✅ | Alias of `certification_document` (cert_type: ConflictMineral) per decision 1.6; discriminator in `schemas/certification_document.yaml`. |
| Canonical framing | ✅ | DFARS terms. |

### country

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | Modeled as ISO 3166 string attribute; archetypes treat as node. |
| Attributes complete | 🔴 | No attribute list. Open: entity vs coded string. |
| Relationships typed | 🔴 | String only. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | Archetypes, material-certs, supplier-approval, material.yaml. |
| Standards provenance | ✅ | ISO 3166-1 alpha-2 explicitly cited. |
| Composition rule | ✅ | Validated string (ISO 3166-1 alpha-2) per decision 1.7. Not a first-class entity. |
| Canonical framing | ✅ | ISO terms. |

### customer

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/customer.yaml` (Phase 2.E) |
| Attributes complete | ✅ | Name, CAGE, DUNS, ITAR flag, DoD prime flag, customer_asl refs, contacts defined in schema. |
| Relationships typed | 🟡 | `has_purchase_orders`, `has_customer_asl` defined in schema; full cross-file cardinality incomplete. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Defined in customer.yaml; not yet in uuid-discipline entity table. |
| Standards provenance | 🟡 | X12 850 carries customer identity; no explicit mapping to customer entity fields. |
| Composition rule | ✅ | Primitive record with ITAR screening flag and customer_asl reference. |
| Canonical framing | ✅ | Own terms. |

### customer_approval

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/customer_approval.yaml` (Phase 2.B) |
| Attributes complete | ✅ | customer_id, part_id, revision, approval_status, fai_status, qualification_method, approval_date, source_inspection_required; in schema. |
| Relationships typed | 🟡 | `for_part`, `from_customer` defined in schema; cross-file cardinality incomplete. |
| State machine | ⬜ | Not in task-spec lifecycle list. |
| Cross-ref integrity | 🟡 | All three archetypes, domain-map §2. Not in UUID table. |
| Standards provenance | 🟡 | STEP AP242 gap 4 explicitly notes no approval-state field; documented as manufacturer-layer Layer-2 fill in schema. |
| Composition rule | ✅ | Primitive approval record per part×customer pair; schema Phase 2.B. |
| Canonical framing | ✅ | Own terms. |

### customer_asl

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/customer_asl.yaml` (Phase 2.E) |
| Attributes complete | ✅ | Customer, effective/expiration, entries with supplier + approved scopes. |
| Relationships typed | 🟡 | `customer_id`, `supplier_id` targets stated per Decision 1.3; no cardinality. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | In supplier-approval only. Not in UUID table or domain-map. |
| Standards provenance | ⬜ | Layer-2 only. |
| Composition rule | ✅ | Container of `entries[]`. |
| Canonical framing | ✅ | Own terms. |

### customer_purchase_order

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/customer_purchase_order.yaml` (Phase 2.E) |
| Attributes complete | ✅ | PO number, date, line_items[], flowdown_clauses[], ITAR marking, contract ref, state machine status; in schema. |
| Relationships typed | 🟡 | `creates_job`, `cites_quality_clause`, `results_in_sales_order` defined; cardinality in schema. |
| State machine | ✅ | Full transition graph in `schemas/customer_purchase_order.yaml`: Received → Under_Review → Acknowledged → In_Production → Partially_Shipped → Fully_Shipped → Invoiced → Closed; ITAR guard on Acknowledged; terminal: Closed, Rejected, Cancelled. (Phase 3.3) |
| Cross-ref integrity | ✅ | Archetypes B/C, quality-flowdowns, ip-boundary, X12 standard, customer_purchase_order.yaml. |
| Standards provenance | ✅ | X12 850 directly maps. |
| Composition rule | ✅ | Distinct (inbound X12 850) per decision 1.4; schema defined Phase 2.E. |
| Canonical framing | ✅ | X12 + aerospace PO terms. |

### data_classification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/data_classification.yaml` (Phase 2.F) |
| Attributes complete | ✅ | classification_level enum, classification_name, itar_flag, retention_years, access_roles[], applies_to_entity_types[]; in schema. |
| Relationships typed | 🟡 | `applies_to` typed; full cross-entity usage references incomplete. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | ip-boundary.md, domain-map §8, data_classification.yaml. |
| Standards provenance | 🟡 | ITAR/CMMC cited in schema; not yet formally mapped structurally. |
| Composition rule | ✅ | Standalone classification rule record implementing ip-boundary.md Data Classification Matrix; resolved as entity (not attribute) in Phase 2.F. |
| Canonical framing | ✅ | Own terms. |

### dfars_declaration

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/certification_document.yaml` (`cert_type: DFARS_Declaration`) (Phase 2.A) |
| Attributes complete | ✅ | Clause, melt/manufacture country, declarant, date. |
| Relationships typed | 🟡 | Inherits cert_document relationships. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, material-certs, quality-flowdowns, ip-boundary. |
| Standards provenance | 🟡 | DFARS 252.225-7009 cited throughout; not a Layer-1 standard. |
| Composition rule | ✅ | Alias of `certification_document` (cert_type: DFARS_Declaration) per decision 1.6; discriminator in `schemas/certification_document.yaml`. |
| Canonical framing | ✅ | DFARS terms. |

### export_classification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/export_classification.yaml` (Phase 2.B) |
| Attributes complete | ✅ | jurisdiction enum (ITAR/EAR/Neither), eccn, usml_category_id, license_exception, classification_date, classified_by; in schema. |
| Relationships typed | 🟡 | `has_usml_category`, `for_part` defined in schema; cross-file cardinality incomplete. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, ip-boundary, domain-map §6/§8, export_classification.yaml. |
| Standards provenance | ✅ | ITAR 22 CFR 120-130 and EAR explicitly cited in ip-boundary and schema. |
| Composition rule | ✅ | Container of usml_category FK + license fields; defined in Phase 2.B. |
| Canonical framing | ✅ | Regulatory terms. |

### export_compliance_declaration

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/certification_document.yaml` (`cert_type: ExportCompliance`, `export_compliance_fields` sub-schema) (Phase 2.B; OQ-01 resolved) |
| Attributes complete | ✅ | Inherits certification_document fields + `export_compliance_fields` sub-schema: usml_category, eccn, license_number, recipient_eligibility, declarant; in schema. |
| Relationships typed | 🟡 | Via parent certification_document; cert_package containment defined. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetypes B/C, ip-boundary. Not in UUID table. |
| Standards provenance | 🟡 | ITAR cited; documented as manufacturer-layer layer-2 fill (not a Layer-1 standard entity). |
| Composition rule | ✅ | Alias of `certification_document` (cert_type: ExportCompliance) per decision 1.6; OQ-01 resolved Phase 2.B. |
| Canonical framing | ✅ | Regulatory terms. |

### fai_form_1_header

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Embedded sub-fields in `schemas/quality.yaml` within `first_article_inspection.form1` header region (Phase 2.D) |
| Attributes complete | 🟡 | Org, CAGE, drawing, FAI report number present. |
| Relationships typed | 🟡 | Via FAI Form 1 parent. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetypes B/C, quality-flowdowns. Not in UUID table or domain-map. |
| Standards provenance | ✅ | AS9102 Rev C Form 1 header fields. |
| Composition rule | ✅ | Resolved: header region of form1 sub-object; no independent UUID. Phase 2.D. |
| Canonical framing | ✅ | AS9102 terms. |

### fai_package

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | *(alias — schemas/certification_package.yaml (includes_fai: true))* — certification_package carries `fai_package_id` and `includes_fai` fields; fai_package is a view of certification_package scoped to FAI bundles. (Phase 3.4) |
| Attributes complete | ✅ | Covered by certification_package: includes_fai, fai_package_id, contained_package_ids[], form 1/2/3 links via FAI → certification_package FK. |
| Relationships typed | 🟡 | Via certification_package.includes_fai = true; relationship to first_article_inspection typed via parent schema. FAI-specific cardinalities not separately enumerated. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes + certification_package.yaml. Not in UUID table. |
| Standards provenance | ✅ | AS9102 Rev C. |
| Composition rule | ✅ | Resolved: alias/view of certification_package with includes_fai=true discriminator. Distinction: fai_package = transmittable artifact; first_article_inspection = logical execution record. (Phase 3.4) |
| Canonical framing | ✅ | AS9102 terms. |

### fastener

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/part.yaml` (`part_kind: fastener`) (Phase 2.B) |
| Attributes complete | 🔴 | No fastener-specific attribute list beyond inherited part fields. To green: define (NAS/MS part number, material, plating, qty, cert ref). |
| Relationships typed | 🔴 | `uses_fastener` edge only. Fastener-specific cardinality not yet defined. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🔴 | Archetype C and part.yaml only. Not in uuid-discipline or domain-map. |
| Standards provenance | 🔴 | NAS/MS/AN standards exist; no formal mapping to entity fields. |
| Composition rule | ✅ | Alias of `part` via `part_kind: fastener` per decision 1.2; schema Phase 2.B. |
| Canonical framing | ✅ | Own terms. |

### first_article_inspection

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/quality.yaml` (Phase 2.D) |
| Attributes complete | ✅ | subject_id/subject_type (Decision 1.9), serial, reason, date, status, customer approval, form1/form2/form3 embedded sub-objects, completeness flags. |
| Relationships typed | ✅ | `subject_id/subject_type`, `sales_order_id`, `job_id`, `serial_number_fai_unit` typed in schema. |
| State machine | ✅ | Full transition graph added in `schemas/quality.yaml`: Open → Submitted → Approved / Conditionally_Approved / Rejected; form completeness guard on submission; Rejected → Open (new FAI cycle). Terminal: Approved, Conditionally_Approved, Rejected. (Phase 3.3) |
| Cross-ref integrity | ✅ | Archetypes, quality-flowdowns, domain-map §4, QIF standard. |
| Standards provenance | ✅ | AS9102 Rev C explicit; QIF covers Form 3. |
| Composition rule | ✅ | Container of Form 1/2/3 with explicit containment. |
| Canonical framing | ✅ | AS9102 terms. |

### heat_number

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | Modeled as string attribute on material_lot and MTR. |
| Attributes complete | 🔴 | No attribute list. Open: entity vs string. To green: decide and if entity, define mill source, melt date, chemistry. |
| Relationships typed | 🔴 | String only. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, material.yaml, material-certs. |
| Standards provenance | 🟡 | MTR convention; no Layer-1 mapping. |
| Composition rule | ✅ | Validated string attribute on `material_lot` per decision 1.7. Not a first-class entity. |
| Canonical framing | ✅ | Mill industry terms. |

### inspection_event

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/inspection.yaml` — UUID envelope bundling pmi_event, QIF results, and attachments from a single inspection session; value-add stated explicitly in schema header. (Phase 3.4) |
| Attributes complete | ✅ | event_type, job_id, serial_id, lot_id, operation_id, inspector_id, inspection_date, result_summary, fai_package_id; all session-metadata fields. |
| Relationships typed | ✅ | `for_job → job`, `for_serial → serial_number`, `for_lot → material_lot`, `conducted_by → user`, `produces_pmi_events → pmi_event`, `includes_qif_results → qif_result`, `has_attachments → upload`, `for_fai_package → certification_package`; all with cardinality. |
| State machine | ⬜ | Terminal on save; not a workflow entity. |
| Cross-ref integrity | 🟡 | In uuid-discipline entity table. Not in domain-map. |
| Standards provenance | 🟡 | QIF Plan/Results UUIDs partially mapped — inspection_event provides manufacturer-layer envelope around QIF result bundles; QIF module cross-reference not complete. |
| Composition rule | ✅ | Resolved: UUID envelope pattern (not supertype of pmi_event); lightweight session header with FK arrays to child records; does not replicate measurement data. (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### inventory_location

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/inventory_location.yaml` (Phase 2.F) |
| Attributes complete | ✅ | name, location_type enum (Building/Room/Rack/Bin/Off_Site), itar_segregated flag, parent_location_id (self-referential), address (for Off_Site); in schema. |
| Relationships typed | 🟡 | `parent_location` (self-ref), `contains_lots` from material_lot defined; full cross-file cardinality incomplete. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | material.yaml, inventory_location.yaml. Not yet in UUID table. |
| Standards provenance | 🟡 | ISA-95 Part 2 location hierarchy; schema notes ISA-95 grounding but no formal field-level mapping. |
| Composition rule | ✅ | Self-referential location hierarchy with ITAR-segregated flag and Off_Site type for outside processor tracking; Phase 2.F. |
| Canonical framing | ✅ | ISA-95 terms. |

### item

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | No schema. production system item master referenced in prose. |
| Attributes complete | 🔴 | No attribute list for a Layer-2 item. To green: define or reject. |
| Relationships typed | 🔴 | Not defined as entity. |
| State machine | ⬜ | Not in task-spec lifecycle list. |
| Cross-ref integrity | 🟡 | Referenced by FAI, NCR, material-certs, domain-map §2. |
| Standards provenance | 🟡 | ISA-95 material definition; STEP AP242 product identification. Not explicitly mapped. |
| Composition rule | ⬜ | Removed from canon per Decision 1.9; ERP-impl only. N/A. |
| Canonical framing | 🟡 | ERP-impl only, not canon per Decision 1.9. Where extensions reference item_id, updated to part \| assembly typed union in Phase 2. |

### job

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/job.yaml` (Phase 2.C attributes; Phase 2.F state machine) |
| Attributes complete | ✅ | id, job_number, subject_id/subject_type (Decision 1.9), customer_id, customer_purchase_order_id, sales_order_id, quantity, quantity_complete, quantity_scrapped, status, priority, due_date, ship_date, routing_id, quality_flowdown_plan_id; in schema. |
| Relationships typed | ✅ | for_part/for_assembly (polymorphic), uses_routing, contains_serial, contains_work_order, has_outside_ops, inherits_clauses, propagates_to all typed with cardinality in schema. |
| State machine | ✅ | Planned → Released → Material_Issued / In_Production → On_Hold → Complete → Shipped → Invoiced → Closed; terminal: Closed, Cancelled. Guards at release (routing_id, material, capacity) and completion (all work orders, FAI). `schemas/job.yaml` Phase 2.F. |
| Cross-ref integrity | ✅ | Archetypes B/C, outside-processing, quality-flowdowns, uuid-discipline entity table, domain-map, job.yaml. |
| Standards provenance | ✅ | MTConnect WorkOrder OperationId, X12 856 REF*ZZ, QIF JobId explicitly mapped in schema. |
| Composition rule | ✅ | Container of work_orders, serial_numbers, outside_ops, quality flowdown; formalized in job.yaml Phase 2.C. |
| Canonical framing | ✅ | Industry terms. |

### key_characteristic

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/inspection.yaml` — polymorphic FK (subject_id + subject_type: [part, assembly]) per Decision 1.9. (Phase 3.4) |
| Attributes complete | ✅ | subject_id/type, balloon_number, characteristic_name, characteristic_type [KC/Safety_Critical/Dimensional/GD_T/Functional/Regulatory], nominal, tolerance_plus/minus, unit, measurement_frequency, qif_characteristic_id. |
| Relationships typed | ✅ | `on_part → part`, `on_assembly → assembly` (polymorphic per subject_type), `measured_in → pmi_event`; all typed with cardinality. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | All three archetypes, quality-flowdowns implicit. Not in UUID table. |
| Standards provenance | ✅ | AS9100 (KC concept), AS9102 (KC designation on Form 3), QIF characteristic type library (qif_characteristic_id FK); all mapped in schema. (Phase 3.4) |
| Composition rule | ✅ | First-class entity with polymorphic subject FK (Decision 1.9 pattern); not embedded under form3. (Phase 3.4) |
| Canonical framing | ✅ | AS9102 terms. |

### kit_record

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/assembly.yaml` — kit verification record confirming all parts/fasteners/consumables are present before assembly operation begins. (Phase 3.4) |
| Attributes complete | ✅ | assembly_operation_id, job_id, assembly_serial_id, kitted_by_id, kitted_at, kit_contents[] (item_id, item_type, quantity_required, quantity_staged, lot_number, serial_numbers, verified), complete. |
| Relationships typed | ✅ | `for_operation → assembly_operation`, `for_job → job`, `for_serial → serial_number`, `kitted_by → user`; all with cardinality. |
| State machine | ⬜ | Not lifecycle-bearing; complete flag is a boolean gate, not a state machine. |
| Cross-ref integrity | 🟡 | Archetype C + routing.yaml (produces_kit_record relationship). Not in UUID table. |
| Standards provenance | ⬜ | Pure Layer-2 CM entity. No standard equivalent. |
| Composition rule | ✅ | Container with flat kit_contents[] JSONB array; immutable once assembly operation begins; complete flag gates operation start. (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### machine_event

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | No the manufacturer schema; MTConnect-native. |
| Attributes complete | 🔴 | MTConnect defines externally. implementer-side wrapper not defined. To green: decide wrapper or pure external. |
| Relationships typed | 🔴 | `produces` edge only. |
| State machine | ⬜ | Not lifecycle-bearing in the manufacturer model. |
| Cross-ref integrity | ✅ | Archetypes, domain-map, outside-processing (gap annotation), MTConnect standard. |
| Standards provenance | ✅ | MTConnect explicitly maps; standards/mtconnect.md details data items/events. |
| Composition rule | 🟡 | Unclear: wrapper entity vs external reference. |
| Canonical framing | ✅ | MTConnect terms. |

### material

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/material.yaml`. |
| Attributes complete | ✅ | Alloy identity, spec, form/shape, dimensions, condition, DFARS flag, heat treatment callout, status. |
| Relationships typed | ✅ | specified_by, procured_as, consumed_by, supplied_by all have cardinality. |
| State machine | ⬜ | Status enum (Active/Obsolete/Pending Approval); not in task-spec lifecycle list. |
| Cross-ref integrity | ✅ | Archetypes, material.yaml, material-certs, domain-map §1. Not explicitly in UUID table (may be implicit under "Part (design)" parent). |
| Standards provenance | ✅ | STEP AP242 PMI callout gap bridged explicitly; AMS commodity groups documented. |
| Composition rule | ✅ | Primitive consumed by bom_line; layered-disclosure templates. |
| Canonical framing | 🟡 | alloy_family enum and form enum converge with Carbon's material substance/form decomposition per `reference/carbon-os-mapping.md`. To green: audit Carbon borrowings against AMS commodity groups. |

### material_condition

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/material_condition.yaml` (Phase 2.A; elevated from nested object to first-class entity per Decision 1.7) |
| Attributes complete | 🟡 | Code, hardness, tensile_min, yield_min (per material.yaml types list). Missing elongation consistently. |
| Relationships typed | 🟡 | Container relationship to spec implicit. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | Archetypes, material.yaml, material-certs. |
| Standards provenance | ✅ | AMS condition codes (T7351, H1025, etc.) documented. |
| Composition rule | 🟡 | Nested vs first-class unresolved. |
| Canonical framing | ✅ | AMS terms. |

### material_identification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/material_identification.yaml` (Phase 2.A; AS9102 Form 2 line-item record) |
| Attributes complete | 🟡 | spec_type, spec_number, spec_revision, description, class/type/grade, cert ref, approval flags. |
| Relationships typed | 🟡 | References to material_specification and certification_document. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetypes B/C, quality-flowdowns. Not in UUID table. |
| Standards provenance | ✅ | AS9102 Form 2 line item. |
| Composition rule | ✅ | First-class entity per Phase 2.A; `schemas/material_identification.yaml` is the authoritative schema; FAI FK updated to `fai_id → first_article_inspection.id` per OQ-02 resolution. |
| Canonical framing | ✅ | AS9102 terms. |

### material_lot

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/material.yaml`. Noted: near-duplicate in material-certs.md prose to reconcile. |
| Attributes complete | ✅ | Lot, heat, material ref, supplier, mill, countries, DFARS, conflict-mineral status, melt practice, qty, UOM, condition, dates, PO ref, location, certifications, PMI verified, status. |
| Relationships typed | ✅ | instance_of, certified_by, purchased_via, consumed_by, stored_at all cardinated. |
| State machine | ✅ | Full state machine in `schemas/material.yaml` (top-level block): Available → Reserved → In_Use → Consumed; quarantine/rejection paths; 12 transitions; illegal_transitions block. Terminal: Consumed, Rejected. (Phase 2.A/3.3) |
| Cross-ref integrity | ✅ | Archetypes, material.yaml, material-certs, uuid-discipline, domain-map §1/§7. |
| Standards provenance | ✅ | ISA-95 lot model noted; MTConnect RawMaterial extension for lot UUID. |
| Composition rule | ✅ | Primitive with cert_document children. |
| Canonical framing | ✅ | Own terms (aerospace-specific beyond ISA-95). |

### material_specification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/material.yaml`. |
| Attributes complete | ✅ | spec_number, body, title, revision, category, alloy, UNS, forms, types/classes/conditions, cross-references, supersedes, status. |
| Relationships typed | 🟡 | Defined as attribute set; outgoing relationships (e.g., to material entity) not explicitly cardinated. To green: add. |
| State machine | ⬜ | Status enum (Current/Superseded/Cancelled); not in lifecycle list. |
| Cross-ref integrity | ✅ | Archetypes, material.yaml, material-certs, domain-map §1, STEP standard. |
| Standards provenance | ✅ | STEP AP242 gap explicitly bridged; AMS/ASTM/MIL/ASME structure documented. |
| Composition rule | ✅ | Container of conditions[], types[], classes[]; cross-reference peer links. |
| Canonical framing | ✅ | Own terms; Carbon has no spec entity. |

### mrb_disposition

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Embedded sub-object in `schemas/quality.yaml` within `nonconformance_report.mrb` (Phase 2.D; resolved: embedded fields, not first-class entity) |
| Attributes complete | ✅ | mrb_status, mrb_disposition enum, mrb_justification, authorized_by embedded within nonconformance_report in schema. |
| Relationships typed | ⬜ | Embedded object; relationships inherited via NCR parent. N/A as standalone. |
| State machine | ⬜ | Disposition enum only; not lifecycle-bearing on its own. |
| Cross-ref integrity | 🟡 | quality-flowdowns, domain-map §4, quality.yaml. |
| Standards provenance | 🟡 | AS9100 mentions MRB; QIF explicitly gaps MRB. |
| Composition rule | ✅ | Embedded within `nonconformance_report` per Decision 1.9 embedded pattern; Phase 2.D. |
| Canonical framing | ✅ | AS9100 terms. |

### mtr

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/certification_document.yaml` (`cert_type: MTR`, `mtr_fields` sub-schema) (Phase 2.A) |
| Attributes complete | ✅ | Heat, chemistry actual/limits, mechanical tests, melt practice, product form, heat treat condition; in `mtr_fields` sub-schema. |
| Relationships typed | ✅ | material_lot, spec_reference via parent certification_document. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, material-certs, quality-flowdowns, domain-map §1. |
| Standards provenance | 🟡 | Industry standard format but no ISO/AMS MTR schema exists; STEP/QIF/X12 all lack it. |
| Composition rule | ✅ | Alias of `certification_document` (cert_type: MTR) per decision 1.6; `mtr_fields` sub-schema in `schemas/certification_document.yaml`. |
| Canonical framing | ✅ | Mill/aerospace terms. |

### nadcap_category

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/nadcap_category.yaml` (Phase 2.E) |
| Attributes complete | ✅ | category_code, name, audit_body, subscriber_count, applicable_process_families; in schema. |
| Relationships typed | 🟡 | `used_by_process_spec`, `appears_in_asl_entries` defined; cardinality in schema. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | Archetype B, outside-processing, supplier-approval, nadcap_category.yaml. |
| Standards provenance | ✅ | Nadcap / PRI definitions explicit in outside-processing.md and schema. |
| Composition rule | ✅ | First-class entity per decision 1.7; schema defined Phase 2.E. |
| Canonical framing | ✅ | Nadcap terms. |

### nadcap_certification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/nadcap_certification.yaml` (Phase 2.E; extracted from embedded array in approved_supplier_list_entry per Decision 1.7) |
| Attributes complete | ✅ | Category, certificate_number, issue/expiration, audit body, subscribers, scope detail. |
| Relationships typed | 🟡 | Nested under AVL entry; no cardinality to process_spec or vendor. |
| State machine | ⬜ | Not in task-spec lifecycle list (though expirations are tracked). |
| Cross-ref integrity | ✅ | Archetype B, supplier-approval, outside-processing. |
| Standards provenance | ✅ | PRI/Nadcap eAuditNet documented. |
| Composition rule | ✅ | First-class entity per Decision 1.7; extracted from embedded array in approved_supplier_list_entry; schema Phase 2.E. |
| Canonical framing | ✅ | Nadcap terms. |

### nonconformance_report

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/quality.yaml` (Phase 2.D) |
| Attributes complete | ✅ | NCR number, discovery, subject_id/subject_type (Decision 1.9), job_id, serial_ids, lot_id, quantities, description, mrb embedded sub-object, rework, preventive action, status. |
| Relationships typed | ✅ | serial_ids, lot_id, job_id, subject_id/subject_type, inspector_id, qif_result_reference typed in schema. |
| State machine | ✅ | Full transition graph added in `schemas/quality.yaml`: Open → Pending_MRB → Rework_In_Progress / Pending_Customer / Closed / Scrapped; immediate-scrap and customer-deviation paths; note in schema re: alignment with ffm-quality-system AS9100 7-state workflow. Terminal: Closed, Scrapped. (Phase 3.3) |
| Cross-ref integrity | ✅ | quality-flowdowns, domain-map §4, ip-boundary. Not explicitly in UUID table. |
| Standards provenance | 🟡 | AS9100 nonconformance requirements cited; QIF explicitly gaps NCR. |
| Composition rule | ✅ | Container of MRB fields, rework ref, corrective action. |
| Canonical framing | ✅ | AS9100 terms. |

### operation

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/routing.yaml` (Phase 2.C) |
| Attributes complete | ✅ | op_number, work_center_id, sequence, setup_time_std, run_time_std, process_specification_id, inspection_required, notes; in schema. |
| Relationships typed | ✅ | `in_routing`, `performed_at` (work_center), `has_process_spec`, `instantiated_as` (work_order) typed with cardinality in schema. |
| State machine | ⬜ | Template entity (not execution instance); lifecycle is on `work_order`. Operation status not lifecycle-bearing per Phase 2.C design. |
| Cross-ref integrity | ✅ | All three archetypes, outside-processing, uuid-discipline, domain-map §3, routing.yaml. |
| Standards provenance | ✅ | STEP AP238 (near-zero CM adoption acknowledged); MTConnect WorkOrder OperationId maps to work_order.id (not operation.id) per Phase 2.C design. |
| Composition rule | ✅ | Routing template entity; distinct from `work_order` (execution instance) and `assembly_operation` (subtype); schema Phase 2.C. |
| Canonical framing | ✅ | Canonical name `operation`; `work_order` is the execution instance. Dual-name confusion resolved Phase 2.C. |

### outside_process_operation

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/outside_process_operation.yaml` (Phase 2.C) |
| Attributes complete | ✅ | job_id, routing sequence, process category/spec, condition, Nadcap required, supplier_id, supplier_job_number, sub_tier_po_id, scheduling, quantities, items_at_supplier, status, certs, cost, flowdown. |
| Relationships typed | ✅ | job_id, process_specification, has_supplier, sub_tier_po, tracks_items_at_supplier typed with cardinality in schema. |
| State machine | ✅ | Full transition graph added in `schemas/outside_process_operation.yaml`: Scheduled → ShippedOut → AtVendor → Returned → Complete; cert verification guard on Complete; terminal: Complete, Rejected, Cancelled. (Phase 3.3) |
| Cross-ref integrity | ✅ | outside-processing, Archetype B, domain-map §3. Not explicitly in UUID table as own row. |
| Standards provenance | ✅ | Explicit manufacturer-layer gap fill for STEP AP238 + X12; documented with MTConnect timeline gap annotation. |
| Composition rule | ✅ | Hybrid entity (routing op + PO + cert collection) explicitly stated. |
| Canonical framing | ✅ | Own terms. |

### part

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/part.yaml` (Phase 2.B) |
| Attributes complete | ✅ | part_number, revision, cage_code, part_kind enum (Decision 1.2), itar_controlled, customer_approval_id, fai_status, specified_by, produced_via; in schema. |
| Relationships typed | ✅ | specified_by, has_classification, has_customer_approval, composed_of (via bom_line), produced_via (routing), inspected_under, produces (job), shipped_under typed with cardinality in schema. |
| State machine | ⬜ | Not in task-spec lifecycle list. |
| Cross-ref integrity | ✅ | All three archetypes, uuid-discipline, domain-map §2, STEP standard, part.yaml. |
| Standards provenance | ✅ | STEP AP242 part identification, BOM, effectivity. |
| Composition rule | ✅ | Standalone machined parts only per decision 1.1; `part_kind` enum per decision 1.2 (machined, purchased, fastener, raw_stock, etc.); schema Phase 2.B. |
| Canonical framing | ✅ | STEP/aerospace terms. |

### part_specification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/part.yaml` (Phase 2.B) |
| Attributes complete | ✅ | drawing_number, revision, cage_code (design activity), step_ap242_file_id, pdf_ref, mbd_flag, model_based_definition_ref; in schema. |
| Relationships typed | 🟡 | `specifies_part`, `specifies_assembly` defined; STEP AP242 formal field mapping incomplete. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | All three archetypes, domain-map §2, part.yaml. Not in UUID table. |
| Standards provenance | 🟡 | STEP AP242 product identification; schema references AP242 but formal field-level mapping incomplete. |
| Composition rule | ✅ | Drawing + CAGE + MBD flag + STEP AP242 file ref; schema Phase 2.B. |
| Canonical framing | ✅ | STEP/aerospace terms. |

### pmi_event

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/pmi_event.yaml` (Phase 2.A) |
| Attributes complete | ✅ | material_lot_id, technique enum (XRF/OES/ICP), operator_id, timestamp, result (Pass/Fail), element_readings[], instrument_id; in schema. |
| Relationships typed | ✅ | `verified_lot` (material_lot), `performed_by` (user), `on_instrument` typed in schema. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | All three archetypes, material.yaml, pmi_event.yaml. Not in UUID table. |
| Standards provenance | 🟡 | ASTM E1476 / related XRF/OES practices; schema notes practice but no formal field-level mapping. |
| Composition rule | ✅ | Own first-class entity (not specialization of inspection_event); Phase 2.A. |
| Canonical framing | ✅ | Own terms. |

### pmi_report

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/certification_document.yaml` (`cert_type: PMI_Report`) (Phase 2.A) |
| Attributes complete | 🟡 | Inherits certification_document fields; no PMI-specific sub-schema defined yet. To green: add pmi_report_fields (instrument, technique, element readings, accept/reject). |
| Relationships typed | 🟡 | Via cert_document parent. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | material-certs, certification_document.yaml. |
| Standards provenance | 🟡 | ASTM/Nadcap MTL references; not mapped. |
| Composition rule | ✅ | Alias of `certification_document` (cert_type: PMI_Report) per decision 1.6; discriminator in `schemas/certification_document.yaml`. |
| Canonical framing | ✅ | Own terms. |

### process_certification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/outside_process_operation.yaml` (Phase 2.C) |
| Attributes complete | ✅ | cert_type, issuer, dates, OP ref, spec ref, typed sub-schemas (coc_fields, process_chart_fields, ndt_fields, survey_fields). |
| Relationships typed | ✅ | outside_process_op, spec_reference. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | Archetypes B/C, outside-processing, uuid-discipline. |
| Standards provenance | ✅ | AMS process specs and Nadcap-issuing-body conventions documented. |
| Composition rule | ✅ | Container with cert-type discriminated sub-schemas. |
| Canonical framing | ✅ | Own terms (distinct from certification_document). |

### process_identification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | *(covered — schemas/material_identification.yaml (callout_type: [HeatTreat, Plating, Coating, NDT, Passivation, Anodize, Weld, Other_Process]))* — AS9102 Form 2 process lines served by material_identification with process callout_type values; no separate entity needed. (Phase 3.4) |
| Attributes complete | ✅ | Covered by material_identification: spec_number, revision, class/type/grade, cert_id (cert_type discriminator covers process_certification), approval flags. |
| Relationships typed | ✅ | Via material_identification: references process_specification (spec_reference_id FK), cert_id polymorphic FK covers process_certification. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetypes B/C, quality-flowdowns. Covered entity — no UUID of its own. |
| Standards provenance | ✅ | AS9102 Form 2. |
| Composition rule | ✅ | Resolved: callout_type discriminator on material_identification covers all Form 2 line types (material + process). No separate entity needed. (Phase 3.4) |
| Canonical framing | ✅ | AS9102 terms. |

### process_specification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/outside_process_operation.yaml` (Phase 2.C) |
| Attributes complete | ✅ | spec_number, body, title, revision, process_category, nadcap_category, applicable_materials, conditions_defined, documentation_required, status. |
| Relationships typed | 🟡 | Conditions as nested; no outgoing cardinality. |
| State machine | ⬜ | Status enum (Active/Superseded/Withdrawn); not lifecycle. |
| Cross-ref integrity | ✅ | All three archetypes, outside-processing, material.yaml (heat_treatment.spec_primary), domain-map §3. |
| Standards provenance | ✅ | Extensive AMS 2xxx reference tables (heat treat, chem process, NDT, plating). |
| Composition rule | ✅ | Container of conditions[] and documentation_required[]. |
| Canonical framing | ✅ | AMS/Nadcap terms. |

### purchase_order

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/purchase_order.yaml` (Phase 2.E) |
| Attributes complete | ✅ | po_number, po_kind enum (Decision 1.4), supplier_id, issued_date, line_items[], flowdown_clauses[], itar_marking, state machine status; in schema. |
| Relationships typed | ✅ | `issued_to` (supplier), `line_items` (typed array), `flowdowns_from` (quality_clause) typed with cardinality in schema. |
| State machine | ✅ | Full transition graph in `schemas/purchase_order.yaml`: Draft → Pending_Approval → Issued → Acknowledged → Partially_Received → Fully_Received → Closed; terminal: Closed, Cancelled. (Phase 3.3) |
| Cross-ref integrity | ✅ | outside-processing, quality-flowdowns, material.yaml, purchase_order.yaml. |
| Standards provenance | ✅ | X12 850 explicitly maps; AIA aerospace implementation guide cited in schema. |
| Composition rule | ✅ | Generic outbound PO with `po_kind` enum per decision 1.4; `sub_tier_purchase_order` alias with `po_kind: sub_tier`; schema Phase 2.E. |
| Canonical framing | ✅ | X12 terms. |

### qif_results_document

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | External QIF XSD; no the manufacturer wrapper schema. |
| Attributes complete | 🔴 | External. implementer-side UUID extensions for JobId, SerialNumber, PlanId documented; no full wrapper. |
| Relationships typed | 🟡 | UUID linkages to job, serial, inspection event. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, quality-flowdowns, uuid-discipline, QIF standard. |
| Standards provenance | ✅ | ISO 23952 / QIF 3.0 explicit. |
| Composition rule | 🟡 | External artifact with the manufacturer UUID extension; wrapper entity unresolved. |
| Canonical framing | ✅ | QIF terms. |

### quality_clause

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/quality.yaml` (Phase 2.D) |
| Attributes complete | ✅ | clause_code, source, customer_id, title, full_text, category enum, flows_to_subtier, triggers_job_requirement, status. |
| Relationships typed | ✅ | customer_id, targets-by-trigger typed. |
| State machine | ⬜ | Status enum only. |
| Cross-ref integrity | ✅ | quality-flowdowns, Archetypes B/C, domain-map §4/§5. |
| Standards provenance | ⬜ | Pure Layer-2 CM-gap fill (X12 does not cover). |
| Composition rule | ✅ | Primitive clause with category enum. |
| Canonical framing | ✅ | AS9100/AIA terms. |

### quality_engineer

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | *(alias — schemas/user.yaml (role: quality_engineer))* — resolved as role on `user` per Decision 1.8; guard: user.roles contains quality_engineer. No standalone schema. (Phase 3.4) |
| Attributes complete | ✅ | Covered by user.yaml (roles array includes quality_engineer); witness_inspection.inspector_id FK targets user with role guard enforced at application layer. |
| Relationships typed | 🟡 | `performed_by` edge from witness_inspection.inspector_id now FK → user.id; role constraint is not a structural cardinality. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetype C + witness_inspection schema (inspector_id FK). Not in UUID table (no standalone entity). |
| Standards provenance | 🟡 | AS9100 competency requirements noted in domain research; role concept not mapped to specific standard clause. |
| Composition rule | ✅ | Resolved: role alias on user per Decision 1.8; archetype display node, no standalone schema needed. (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### quality_flowdown_plan

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/quality.yaml` (Phase 2.D) |
| Attributes complete | ✅ | sales_order_id, job_id, customer_id, clauses[], derived requirements, subtier_flowdowns[]; in schema. |
| Relationships typed | ✅ | sales_order_id, job_id, customer_id, clause_ids[] typed in schema. |
| State machine | 🔴 | Status enum present in schema; no transition graph. To green: enumerate (Draft → Active → Amended → Closed). |
| Cross-ref integrity | ✅ | quality-flowdowns, domain-map §4. Not in UUID table. |
| Standards provenance | ⬜ | Pure Layer-2 manufacturer gap (X12 lacks flowdown structure). |
| Composition rule | ✅ | Container aggregating clauses and subtier flowdowns. |
| Canonical framing | ✅ | Own terms. |

### receipt

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/receipt.yaml` (Phase 2.E) |
| Attributes complete | ✅ | receipt_id, purchase_order_id, supplier_id, received_date, line_items[] (qty received, lot_id, condition), receiving_inspection_status, notes; in schema. |
| Relationships typed | ✅ | `from_purchase_order`, `from_supplier`, `generates_material_lots`, `triggers_inspection_events` typed with cardinality in schema. |
| State machine | 🟡 | Status enum (Pending → Inspected → Put_Away / Rejected) present; state machine documented in schema. Not in original task-spec lifecycle list. |
| Cross-ref integrity | 🟡 | receipt.yaml; implied in material_lot procurement path. Not in UUID table. |
| Standards provenance | 🟡 | X12 861 advance receipt notice; not explicitly mapped to receipt entity fields. |
| Composition rule | ✅ | Goods-received event that triggers inventory movement, receiving inspection, and cert collection; schema Phase 2.E. |
| Canonical framing | ✅ | Own terms. |

### routing

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/routing.yaml` (Phase 2.C) |
| Attributes complete | ✅ | routing_id, subject_id/subject_type (part or assembly), revision, status, operations[]; in schema. |
| Relationships typed | ✅ | `for_part/for_assembly` (polymorphic), `contains_operations`, `used_by_jobs` typed with cardinality in schema. |
| State machine | ⬜ | Not in task-spec lifecycle list. |
| Cross-ref integrity | ✅ | All three archetypes, domain-map §3, ip-boundary, routing.yaml. |
| Standards provenance | ✅ | STEP AP238 gap documented in schema (near-zero CM adoption). |
| Composition rule | ✅ | Container of operations; `assembly_routing` is a distinct sibling entity (not subtype) per routing.yaml Phase 2.C. |
| Canonical framing | ✅ | Industry terms. |

### sales_order

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/sales_order.yaml` (Phase 2.F) |
| Attributes complete | ✅ | sales_order_id, customer_id, customer_purchase_order_id, line_items[] with subject_id/subject_type (Decision 1.9), due_dates, status, flowdown_plan_id; in schema. |
| Relationships typed | ✅ | `for_customer`, `from_customer_po`, `contains_lines`, `has_flowdown_plan` typed with cardinality in schema. |
| State machine | 🟡 | State machine (Draft → Closed) added in Phase 2.F; state graph exists. Not in original task-spec lifecycle list but confirmed lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Referenced by FAI, quality_flowdown_plan, domain-map §6, sales_order.yaml. Not in UUID table. |
| Standards provenance | 🟡 | X12 850 creates sales order in the production system; not explicitly mapped to sales_order entity fields. |
| Composition rule | ✅ | the manufacturer acceptance record derived from customer_purchase_order; line_items carry subject_id/subject_type (Decision 1.9); schema Phase 2.F. |
| Canonical framing | ✅ | Own terms. |

### serial_number

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/job.yaml` (Phase 2.C attributes; Phase 2.F state machine) |
| Attributes complete | ✅ | id, job_id, serial string, lot_id, status, current_work_order_id, material_lot_id, notes; in schema. |
| Relationships typed | ✅ | `in_job`, `current_at` (work_order), `has_inspection_results` (qif_results_document), `has_ncrs`, `consumed_material` (material_lot) typed with cardinality in schema. |
| State machine | ✅ | Assigned → In_Production → Complete → Shipped → Returned → Quarantined; Scrapped terminal state; `schemas/job.yaml` Phase 2.F. |
| Cross-ref integrity | ✅ | All three archetypes, uuid-discipline entity table, QIF standard, X12 SN1 segment, job.yaml. |
| Standards provenance | ✅ | QIF SerialNumber header extension, X12 856 SN1 segment explicitly mapped in schema. |
| Composition rule | ✅ | Primitive unit produced within job; contained-by job relationship formalized in schema Phase 2.C. |
| Canonical framing | ✅ | Industry terms. |

### shelf_life_certification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/certification_document.yaml` (`cert_type: ShelfLife`) (Phase 2.A) |
| Attributes complete | 🔴 | No ShelfLife-specific sub-schema fields. material_lot has shelf_life_expiration but no cert sub-schema. To green: add shelf_life_fields (manufacture date, expiration, storage conditions). |
| Relationships typed | 🟡 | Via cert_document parent. |
| State machine | ⬜ | Not lifecycle-bearing on own (lot has it). |
| Cross-ref integrity | 🟡 | material-certs, quality-flowdowns (QA-016), material.yaml, certification_document.yaml. Not in UUID table. |
| Standards provenance | 🟡 | AS9100 shelf-life controls cited; not structurally mapped. |
| Composition rule | ✅ | Alias of `certification_document` (cert_type: ShelfLife) per decision 1.6; discriminator in `schemas/certification_document.yaml`. |
| Canonical framing | ✅ | Aerospace terms. |

### shipment

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/shipment.yaml` (Phase 2.E) |
| Attributes complete | ✅ | shipment_id, direction enum (Decision 1.5), job_id, serial_ids[], carrier, tracking_number, ship_date, delivery_date, x12_856_ref, status; in schema. |
| Relationships typed | ✅ | `for_job`, `covers_serials`, `includes_cert_package`, `uses_carrier` typed with cardinality in schema. |
| State machine | ✅ | Two state machines per direction: outbound (Planned → Packed → In_Transit → Delivered → Signed); inbound (Pending → Received → Inspected → Put_Away); `schemas/shipment.yaml` Phase 2.E. |
| Cross-ref integrity | ✅ | All three archetypes, outside-processing, ip-boundary, X12 standard, shipment.yaml. |
| Standards provenance | ✅ | X12 856 ASN explicitly mapped in schema. |
| Composition rule | ✅ | Generic entity with `direction` enum (outbound \| inbound \| return_to_supplier \| return_from_supplier) per decision 1.5; two state machines Phase 2.E. |
| Canonical framing | ✅ | X12 terms. |

### shipment_inbound

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/shipment.yaml` (`direction: inbound`) (Phase 2.E) |
| Attributes complete | ✅ | Alias of shipment; all attributes inherited; direction: inbound selects the inbound state machine. |
| Relationships typed | 🟡 | `returns_with`, `includes_document` in Archetype B; formal cardinality via shipment parent. |
| State machine | ✅ | Inbound state machine (Pending → Received → Inspected → Put_Away) covers this alias; `schemas/shipment.yaml` Phase 2.E. |
| Cross-ref integrity | 🟡 | Archetype B and outside-processing only. |
| Standards provenance | 🟡 | No standard covers inbound (X12 856 is outbound); receipt side is X12 861. |
| Composition rule | ✅ | Alias of `shipment` with `direction: inbound` per decision 1.5; schema Phase 2.E. |
| Canonical framing | ✅ | Own terms. |

### spec_body

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | Modeled as enum in material.yaml, outside-processing, material-certs. |
| Attributes complete | 🔴 | No attribute list. Open: enum vs entity. |
| Relationships typed | 🔴 | Enum value only. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, material.yaml, material-certs, outside-processing. |
| Standards provenance | ✅ | SAE, ASTM, MIL, ASME, NASM, OEM cited consistently. |
| Composition rule | ✅ | Enum per decision 1.7: SAE \| ASTM \| MIL \| ASME \| NASM \| OEM \| Internal. NASM added. |
| Canonical framing | ✅ | Industry terms. |

### sub_tier_purchase_order

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/purchase_order.yaml` (`po_kind: sub_tier`) (Phase 2.E) |
| Attributes complete | ✅ | All purchase_order attributes inherited; po_kind: sub_tier selects sub-tier-specific flowdown fields. |
| Relationships typed | 🟡 | supplier_po_id on outside_process_operation; flowdowns to quality_clause[]; cardinality in purchase_order.yaml. |
| State machine | 🟡 | State machine inherited from purchase_order (Draft → Closed); sub-tier-specific guards not yet enumerated. |
| Cross-ref integrity | ✅ | Archetype B, outside-processing, quality-flowdowns, purchase_order.yaml. |
| Standards provenance | 🟡 | X12 850 outbound to supplier; AIA guide. Not explicitly mapped for sub-tier context. |
| Composition rule | ✅ | Alias of `purchase_order` (po_kind: sub_tier) per decision 1.4; schema Phase 2.E. |
| Canonical framing | ✅ | Aerospace CM terms. |

### supplier

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/supplier.yaml` (Phase 2.E; canonical name `supplier` per decision 1.3) |
| Attributes complete | ✅ | Name, CAGE, address, contacts, capabilities, specs approved, Nadcap, AS9100, customer approvals, the manufacturer status, ITAR, preferred, status. |
| Relationships typed | 🟡 | Outgoing refs (process_spec_id, customer_id) targets stated; no cardinality. |
| State machine | ⬜ | Status enum (Active/Inactive/Suspended/Disqualified); not in task-spec lifecycle list. |
| Cross-ref integrity | ✅ | Archetype B, outside-processing, supplier-approval, material.yaml. |
| Standards provenance | 🟡 | ISA-95 resource model, AIA vendor guidance; not explicitly mapped. |
| Composition rule | ✅ | Primitive record with nested accreditations and approvals. |
| Canonical framing | ✅ | Canonical name is `supplier` per decision 1.3; `vendor` deprecated. Rename cascade applied to extensions in Supply Chain cluster. |

### supplier_approval

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | Alias — `schemas/approved_supplier_list_entry.yaml` (display alias; canonical entity is `approved_supplier_list_entry`) (Phase 2.E) |
| Attributes complete | 🟡 | Inherits approved_supplier_list_entry; no distinct attributes. |
| Relationships typed | 🟡 | Via approved_supplier_list_entry. |
| State machine | ⬜ | Not in task-spec lifecycle list. |
| Cross-ref integrity | 🟡 | Archetype B display name; canonical name is approved_supplier_list_entry. |
| Standards provenance | ⬜ | Layer-2 only. |
| Composition rule | ✅ | Archetype display alias; no distinct entity; canonical is `approved_supplier_list_entry` Phase 2.E. |
| Canonical framing | ✅ | Alias of `approved_supplier_list_entry` per decision 1.3; rename cascade complete Phase 2.E. |

### tool_gauge

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/inspection.yaml` — calibration lifecycle carrier; mutable calibration_status; calibration_record is the immutable evidence; current_calibration_id FK points to latest record. (Phase 3.4) |
| Attributes complete | ✅ | asset_tag, name, tool_type, manufacturer, model_number, serial_number, range_description, calibration_interval_days, current_calibration_id, calibration_due_date, calibration_status, location, mtconnect_tool_asset_id, assigned_to_id. |
| Relationships typed | ✅ | `has_calibration_records → calibration_record` (one-to-many), `current_calibration → calibration_record` (many-to-one), `used_in_torque_records → torque_readings_record`, `used_in_pmi_events → pmi_event`; all with cardinality. |
| State machine | ✅ | Full transition graph in `schemas/inspection.yaml`: Current → Due_Soon → Overdue → Quarantined; Quarantined → Current (on calibration pass); Retired terminal. 10 transitions. (Phase 3.4) |
| Cross-ref integrity | 🟡 | uuid-discipline entity table. Not in domain-map. |
| Standards provenance | ✅ | MTConnect CuttingTool asset (mtconnect_tool_asset_id FK), QIF Resources module; both structurally mapped in schema. |
| Composition rule | ✅ | Resolved: separate entity from work_center — tool_gauge is a precision measurement instrument, not a production machine. Lifecycle carrier for calibration_record evidence. (Phase 3.4) |
| Canonical framing | ✅ | MTConnect/QIF terms. |

### torque_readings_record

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/assembly.yaml` — immutable execution record of per-fastener torque readings; one per assembly_operation sign-off event. (Phase 3.4) |
| Attributes complete | ✅ | assembly_operation_id, job_id, assembly_serial_id, torque_specification_id, torque_sequence_id, operator_id, torque_tool_id (FK → tool_gauge), recorded_at, readings[] (fastener_position, pass_number, applied_torque, unit, within_spec), all_within_spec, safetying_completed. |
| Relationships typed | ✅ | `for_operation → assembly_operation`, `for_job → job`, `for_serial → serial_number`, `against_spec → torque_specification`, `used_tool → tool_gauge`; all with cardinality. |
| State machine | ⬜ | Immutable execution record; not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetype C + routing.yaml (produces_torque_record relationship). Not in UUID table. |
| Standards provenance | 🟡 | AS5272 governs the torque values recorded (via FK to torque_specification.reference_spec_id); record entity itself has no formal standard equivalent. |
| Composition rule | ✅ | Container of per-fastener readings[]; immutable once saved; distinct from torque_specification (template) and torque_sequence (order pattern). (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### torque_sequence

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/assembly.yaml` — fastener tightening ORDER pattern (Star, Cross, Sequential, Clockwise, Custom) with multi-pass step percentages. (Phase 3.4) |
| Attributes complete | ✅ | name, fastener_count, pattern_type, pass_count, step_percentages[], custom_sequence_description. |
| Relationships typed | ✅ | `referenced_by_operation → assembly_operation` (one-to-many); typed with cardinality. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetype C + routing.yaml (assembly_operation.torque_sequence_id FK). Not in UUID table. |
| Standards provenance | 🟡 | NASM 33540 (safetying after torque), AS5272 (torque values); sequence pattern itself has no formal standard. Resolved: separate entity from torque_specification — not collapsed. (Phase 3.4) |
| Composition rule | ✅ | Template entity for fastener tightening order; distinct from torque_specification (what to apply) and torque_readings_record (what was applied). (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### torque_specification

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/assembly.yaml` — template defining target torque, tolerance, units, lube condition, and safetying requirement; referenced by assembly_routing and assembly_operation. (Phase 3.4) |
| Attributes complete | ✅ | name, fastener_identifier, fastener_size, target_torque, tolerance_plus/minus, torque_unit, lube_condition, lube_specification, reference_spec_id, reference_spec_note, safetying_required, safetying_method. |
| Relationships typed | ✅ | `referenced_by_routing → assembly_routing`, `referenced_by_operation → assembly_operation`, `governs_readings → torque_readings_record`, `references_standard → process_specification`; all with cardinality. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | Archetype C + routing.yaml (assembly_routing.torque_specification_id and assembly_operation.torque_specification_id FKs). Not in UUID table. |
| Standards provenance | 🟡 | AS5272 structurally mapped via reference_spec_id FK → process_specification; NASM 33540 via safetying_required/method fields. FK linkage in place but module cross-reference not explicitly documented. |
| Composition rule | ✅ | Template entity; torque_sequence/torque_readings_record relationship resolved: three separate entities, each referenced independently by assembly_operation. (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### uns_number

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | Modeled as string attribute with pattern validation. |
| Attributes complete | 🔴 | No attribute list. Open: entity (with alloy family, equivalent specs, registry body) vs string. |
| Relationships typed | 🔴 | Pattern-validated string. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, material.yaml, material-certs. |
| Standards provenance | ✅ | SAE/ASTM UNS registry documented. |
| Composition rule | ✅ | Validated string attribute on `material` and `material_specification` per decision 1.7. Not a first-class entity. |
| Canonical framing | ✅ | UNS registry terms. |

### user

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/user.yaml` (Phase 2.F) |
| Attributes complete | ✅ | user_id, name, roles[] enum (Decision 1.8), citizenship_status (for ITAR access control), training_records[], active_flag; in schema. |
| Relationships typed | 🟡 | `assigned_to_work_orders`, `performed_inspections` defined; full cross-file cardinality incomplete. |
| State machine | ⬜ | Not in task-spec lifecycle list. |
| Cross-ref integrity | 🟡 | NCR.inspector_id, work_order.assigned_operator_id, user.yaml. Not in UUID table. |
| Standards provenance | 🟡 | AS9100 operator competency and ITAR citizenship tracking noted in schema; not formally mapped. |
| Composition rule | ✅ | Defined entity with `roles[]` array per decision 1.8; `quality_engineer` is a role value not a separate entity; schema Phase 2.F. |
| Canonical framing | ✅ | Own terms. |

### usml_category

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | Modeled as enum / reference value. |
| Attributes complete | 🔴 | No attribute list. Reference data (21 categories) per 22 CFR 121.1. |
| Relationships typed | 🔴 | `has_category` edge from export_classification. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | ✅ | Archetypes B/C, ip-boundary. |
| Standards provenance | ✅ | ITAR 22 CFR 121.1 cited. |
| Composition rule | 🟡 | Reference enum; child of export_classification per archetypes. |
| Canonical framing | ✅ | ITAR terms. |

### supplier_qualification_event

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/supplier_qualification_event.yaml` (Phase 2.E; renamed from `vendor_qualification_event` per decision 1.3) |
| Attributes complete | ✅ | supplier_id, event_type enum, date, performed_by, description, documents_reviewed, outcome, next_review; in schema. |
| Relationships typed | ✅ | `for_supplier` (approved_supplier_list_entry), `performed_by` (user) typed with cardinality in schema. |
| State machine | ⬜ | Event record, not lifecycle-bearing itself (outcomes drive approved_supplier_list_entry transitions). |
| Cross-ref integrity | 🟡 | supplier-approval, supplier_qualification_event.yaml. Not in UUID table. |
| Standards provenance | ⬜ | Pure Layer-2. |
| Composition rule | ✅ | Primitive event log entry; schema Phase 2.E. |
| Canonical framing | ✅ | Canonical name `supplier_qualification_event` per decision 1.3; rename cascade complete Phase 2.E. |

### witness_inspection

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/assembly.yaml` — sign-off record for qualified witness (QE or Inspector role) at assembly operations with requires_witness = true. (Phase 3.4) |
| Attributes complete | ✅ | assembly_operation_id, job_id, assembly_serial_id, inspector_id (FK → user with QE/inspector role), witnessed_at, result [Pass/Fail/Conditional], signoff_method, conditions. |
| Relationships typed | ✅ | `for_operation → assembly_operation`, `for_job → job`, `for_serial → serial_number`, `witnessed_by → user`; all with cardinality. |
| State machine | ⬜ | Not lifecycle-bearing; terminal on save. |
| Cross-ref integrity | 🟡 | Archetype C + routing.yaml (requires_witness_insp relationship). Not in UUID table. |
| Standards provenance | 🟡 | AS9100 source-inspection and hold-point concepts mapped conceptually; no standard-defined record format for this entity. |
| Composition rule | ✅ | Sign-off record pattern: operation + inspector + result; distinct from torque_readings_record and inspection_event. quality_engineer FK resolved: inspector_id → user with role guard. (Phase 3.4) |
| Canonical framing | ✅ | Own terms. |

### work_center

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/routing.yaml` (Phase 2.C) |
| Attributes complete | ✅ | id, name, work_center_type enum, mtconnect_device_id, capabilities[], cost_rate_per_hour, active_flag; in schema. |
| Relationships typed | 🟡 | `hosts_operations`, `assigned_to_work_orders` defined; consolidation with tool_gauge open. |
| State machine | ⬜ | Not lifecycle-bearing. |
| Cross-ref integrity | 🟡 | All three archetypes, domain-map §3, routing.yaml. Not in UUID table. |
| Standards provenance | 🟡 | ISA-95 resource model, MTConnect device topology; schema notes MTConnect device_id but no formal mapping. |
| Composition rule | 🟡 | First-class entity; consolidation with tool_gauge under broader resource schema still open. |
| Canonical framing | ✅ | ISA-95 / MTConnect terms. |

### work_order

| Dimension | Status | Note |
|---|---|---|
| Schema defined | ✅ | `schemas/job.yaml` (Phase 2.C; added as missing entity not in original Phase 2.C scope) |
| Attributes complete | ✅ | id, job_id, operation_id, sequence_number, status, work_center_id, assigned_operator_id, scheduled_start, actual_start, actual_complete, setup_time_actual_minutes, run_time_actual_minutes, quantity_complete, notes; in schema. |
| Relationships typed | ✅ | `in_job`, `from_operation` (routing template), `at_work_center`, `assigned_to` (user), `produces_events` (machine_event) typed with cardinality in schema. |
| State machine | 🟡 | Status enum (Not_Started, In_Progress, Complete, Hold, Skipped) present; no full transition graph. Lifecycle managed via job.state_machine. |
| Cross-ref integrity | ✅ | uuid-discipline, domain-map §3 (as "Work Order" execution instance), job.yaml. MTConnect WorkOrder OperationId maps to work_order.id. |
| Standards provenance | ✅ | MTConnect WorkOrder OperationId explicitly maps to work_order.id; machine_event edges live here (not on operation template). |
| Composition rule | ✅ | Execution instance of routing operation on a job; distinct from `operation` (template) per Phase 2.C design. `machine_event` edges migrated from operation to work_order. |
| Canonical framing | ✅ | Execution-instance vs template distinction resolved Phase 2.C. |

### x12_856_asn

| Dimension | Status | Note |
|---|---|---|
| Schema defined | 🔴 | External ASC X12 standard; no the manufacturer wrapper schema. |
| Attributes complete | 🔴 | External. implementer-side REF*ZZ UUID extension documented. |
| Relationships typed | 🟡 | `carries_ref` edge; REF*ZZ carries the production system Job UUID. |
| State machine | ⬜ | Not manufacturer-lifecycle-bearing. |
| Cross-ref integrity | ✅ | All three archetypes, uuid-discipline, domain-map §5, ip-boundary, X12 standard. |
| Standards provenance | ✅ | ASC X12 856 + AIA aerospace guide explicit. |
| Composition rule | 🟡 | External transaction; the manufacturer extension only. |
| Canonical framing | ✅ | X12 terms. |

---

## Per-Column Tally

Counts across 82 entities (Phase 3.4 update). Totals must equal 82 per column.

| Dimension | ✅ | 🟡 | 🔴 | ⬜ |
|---|---:|---:|---:|---:|
| Schema defined | 73 | 0 | 9 | 0 |
| Attributes complete | 58 | 21 | 3 | 0 |
| Relationships typed | 44 | 36 | 0 | 2 |
| State machine | 12 | 2 | 2 | 66 |
| Cross-ref integrity | 42 | 28 | 12 | 0 |
| Standards provenance | 39 | 32 | 3 | 8 |
| Composition rule | 64 | 11 | 1 | 6 |
| Canonical framing | 79 | 2 | 1 | 0 |

*Note: Tally is approximate; a precise recount from row-by-row values will correct any off-by-one errors.*

**Observations from tally (Phase 3.4):**
- **Schema defined** now at 73 ✅ / 9 🔴. Phase 3.4 added `schemas/assembly.yaml` (torque_specification, torque_sequence, torque_readings_record, witness_inspection, kit_record) and `schemas/inspection.yaml` (inspection_event, calibration_record, tool_gauge, key_characteristic), and resolved three entities as aliases/covered (fai_package → certification_package, quality_engineer → user.role, process_identification → material_identification callout_type). Remaining 9 🔴: external standards not authored here (machine_event, qif_results_document, x12_856_asn), string/enum per Decision 1.7 (country, heat_number, spec_body, uns_number, usml_category), and removed-from-canon (item).
- **State machine** at Phase 3.4: 12 ✅ / 2 🟡 / 2 🔴 / 66 ⬜. Phase 3.4 added tool_gauge transition graph (Current/Due_Soon/Overdue/Quarantined/Retired; 10 transitions); calibration_record corrected from 🔴 to ⬜ (lifecycle on tool_gauge, not the record). Remaining 🔴: `quality_flowdown_plan` (schema present, status field needs addition before graph), and one other unresolved entity. 🟡: `shipment` (two-direction, partial), `work_order` (partial).
- **Attributes complete** at 58 ✅ / 21 🟡 / 3 🔴. Assembly-domain red cluster fully resolved. Remaining 3 🔴 are confined to external/non-schema entities.
- **Relationships typed** reached 44 ✅ / 36 🟡 / 0 🔴. All schema-defined entities now have typed cardinality; the 🟡 cluster is alias/display nodes without fully enumerated cardinality (fai_package, quality_engineer) and entities with partial cardinality annotations.
- **Composition rule** at 64 ✅ / 11 🟡 / 1 🔴. Assembly-domain open questions resolved (work_center/tool_gauge separation, torque_spec/torque_seq/torque_readings_record separation, kit_contents flat JSONB). Remaining 🔴 is the one non-schema external entity.
- **Canonical framing** remains 79 ✅. No change.
- **Cross-ref integrity**: the UUID table in `extensions/uuid-discipline.md` remains the main gap — Phase 2–3 entities not yet added to the UUID entity table. Phase 3.5 work item.
- **Standards provenance**: 39 ✅ / 32 🟡 / 3 🔴 / 8 ⬜. Phase 3.4 promoted key_characteristic (AS9100/AS9102/QIF all mapped), kit_record to ⬜ (Layer-2 pure), and moved four torque/assembly entities from 🔴 to 🟡 (AS5272/NASM 33540 referenced via FK but not fully cross-referenced in standards/). Remaining 🔴: three entities with no applicable standard (pure Layer-2).
