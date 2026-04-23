# Taxonomic Decisions — Precision Manufacturing Ontology

Decisions resolving the nine open questions flagged by `reference/entity-matrix.md`
(see "State of the Ontology" narrative, top-5 priority #3). Each decision is an
append-only section. Reopening a decision requires a new dated section citing
the prior one.

Format per decision:

- **Question** — one-sentence statement of the open question
- **Options** — enumerated with their downstream consequences
- **Resolution** — the pick, with rationale grounded in archetype evidence, matrix notes, and Layer-1 standards
- **Impact** — list of Phase 2 schema/extension tasks that depend on this decision
- **Follow-ups** — edits to make in entity-inventory.yaml, entity-matrix.md, and affected extension docs

---

## Decision 1.1 — Assembly vs Recursive Part

**Dated:** 2026-04-22

**Question:** Is `assembly` a distinct entity type or a recursive `part` with an `is_assembly: bool` flag?

**Options:**

- **A** — `assembly` is a distinct entity with `contains_part[]` edges. Assembly-specific operations, kits, witness inspections, torque records, assembly-level FAI, and cert packages all have a natural home on `assembly` rather than forcing them onto `part`. This is what Archetype C displays: an `[Assembly]` node that is meaningfully different from its constituent `[Part]` nodes.
- **B** — `part` is recursive with `composed_of[]` self-edges and `is_assembly: bool`. Simpler entity count; risks cluttering `part` with assembly-only attributes (torque spec ref, witness signoff, kit record ref) that are nonsensical on a primitive machined part.
- **C** — Hybrid: `item` supertype with `part`, `assembly`, `material` as subtypes. Adds an `item` layer not grounded in any Layer-1 standard; rejected separately in Decision 1.9.

**Resolution:** **Option A — `assembly` is a distinct entity.**

Rationale:
1. Archetype C introduces `[Assembly]` as an unambiguous distinct node type with its own cert package, serial, shipment, FAI, and witness-sign-off chain. None of these appear on the constituent `[Part]` nodes in the same tree.
2. Assembly-specific operations (kit+stage, torque bolt-up, witness inspection) have no meaningful equivalents on a primitive machined part. Placing `torque_spec_ref`, `kit_record_ref`, and `witness_inspection_ref` on a generic `part` would make the schema incoherent.
3. STEP AP242 models assembly BOMs via a distinct assembly-usage relationship, not via recursive product-definition with a boolean flag.
4. `bom_line` is retained as a distinct many-to-many edge entity carrying qty, UOM, find-number, and effectivity — this is the STEP AP242 model and what production ERPs typically implement. A `bom_line` joins a parent (`part` or `assembly`) to a child (`part`, `assembly`, or `material`) with quantity semantics on the edge.
5. `part` and `assembly` share **no common parent type** in Layer 2 (no `item` supertype introduced). The `item` label stays a ERP-impl concept only, removed from canon per Decision 1.9.

**Impact:**
- `schemas/part.yaml` — authored in Phase 2 Parts & Items cluster; no `is_assembly` flag.
- `schemas/assembly.yaml` — new schema in Parts & Items cluster with assembly-specific attributes.
- `schemas/bom_line.yaml` — new schema in Parts & Items cluster; parent can be `part` or `assembly`.
- `schemas/routing.yaml` and `schemas/operation.yaml` — routing links to `part` or `assembly` via union ref (Decision 1.1b in Task 2.C.0 addresses whether `assembly_routing` and `assembly_operation` are distinct).
- Assembly-level FAI schema in Quality cluster: `first_article_inspection` references `assembly` as a valid `item` union target (replacing the `item_id` ref per Decision 1.9).

**Follow-ups applied:**
- `entity-inventory.yaml`: `assembly.notes`, `part.notes`, `bom_line.notes` updated to reference this decision.
- `entity-matrix.md`: `assembly.composition_rule` changed from 🔴 to 🟡 (schema authoring pending in Parts & Items cluster).

---

## Decision 1.2 — Fastener Scope

**Dated:** 2026-04-22

**Question:** Is `fastener` a subtype of `part`, a subtype of `material`, or a standalone entity with its own schema?

**Options:**

- **A** — Subtype of `part` (catalog fastener has a part number, revision, CAGE-controlled approval state, and its own cert chain — CoC, DFARS). Purchased complete, not machined from material.
- **B** — Subtype of `material` (fastener consumed at assembly like any consumable). Loses structural cert-chain and part-number identity.
- **C** — Standalone entity with NAS part #, material, plating, torque class fields. Separate schema; adds entity count without sufficient differentiation from Option A.

**Resolution:** **Option A — `fastener` is a subtype of `part`, modeled via `part_kind` enum.**

Rationale:
1. Archetype C uses NAS1351-3-12 as a concrete fastener example. It has a part number, is purchased from a catalog, has its own Certificate of Conformance (fastener CoC), and has DFARS compliance requirements. This is unambiguously part-like.
2. A fastener does not get "machined from" a material lot; it is received as a finished item. Modeling it as `material` would break the traceability chain (material lots have heat numbers and MTRs; fastener lots have lot-numbers and CoCs).
3. A standalone `fastener` schema adds no attributes that are not already covered by `part` + a `part_kind` discriminator. The NAS part number is the part number; the plating spec is a process specification reference.

Concrete shape: `part.part_kind` enum = `machined | fastener | raw_stock | purchased_assembly | consumable`. The `fastener` inventory row is retained as an archetype display alias pointing to "`part` with `part_kind: fastener`". No `schemas/fastener.yaml` is authored.

**Impact:**
- `schemas/part.yaml` in Parts & Items cluster: add `part_kind` enum attribute.
- Routing operations that reference fasteners: `assembly_operation.uses_fastener[]` targets `part` with `part_kind: fastener` constraint (enforced by guard, not schema type).
- `fastener` inventory row: marked as archetype alias; no standalone schema task.

**Follow-ups applied:**
- `entity-inventory.yaml`: `fastener.notes` updated.
- `entity-matrix.md`: `fastener.composition_rule` changed from 🔴 to 🟡 (resolved as part subtype; schema authoring pending via part.yaml).

---

## Decision 1.3 — Supplier / Vendor / AVL Aliasing

**Dated:** 2026-04-22

**Question:** What is the canonical name for the supplier/vendor entity, and is `approved_vendor_list_entry` (AVL entry) a distinct entity or merged into the supplier record?

**Options:**

- **A** — Canonical name `supplier`; `vendor` deprecated as synonym. `approved_vendor_list_entry` renamed `approved_supplier_list_entry` and kept distinct.
- **B** — Canonical name `vendor` (matches `extensions/supplier-approval.md` and `extensions/outside-processing.md` prose). `approved_vendor_list_entry` kept as-is.
- **C** — Canonical name `supplier`; collapse AVL entry into supplier record with inline approval fields.

**Resolution:** **Option A — canonical name `supplier`; `approved_supplier_list_entry` is distinct.**

Rationale:
1. `schemas/material.yaml` already uses `supplier` (not `vendor`) as the field name on `material_lot`. Renaming to `vendor` would break the one schema file in the repo that is canonical.
2. Archetype B display name is `[Supplier]`, not `[Vendor]`.
3. The distinction between `supplier` (contact/address/capability record) and `approved_supplier_list_entry` (structured approval record) mirrors the distinction between `material_lot` and `certification_document`: the base entity is the subject, the approval record is the attestation. The AVL entry is to `supplier` what a cert is to `material_lot`.
4. Collapsing approval fields into the supplier record (Option C) would deny the approval record its own lifecycle (an AVL entry has effective dates, audit history, and expiration — none of which belong on the flat supplier contact record).

Rename cascade:
- `extensions/supplier-approval.md`: rename `approved_vendor_list_entry` → `approved_supplier_list_entry`; rename `vendor_qualification_event` → `supplier_qualification_event`; replace all bare `vendor` with `supplier` except in compound nouns where the source standard uses "vendor" (e.g., X12 "vendor party").
- `extensions/outside-processing.md`: rename `vendor` → `supplier` throughout.
- Inventory rows: `supplier_approval` (archetype B display alias) marked as alias pointing to `approved_supplier_list_entry`.

**Impact:**
- Supply Chain cluster schemas: `schemas/supplier.yaml`, `schemas/approved_supplier_list_entry.yaml`, `schemas/supplier_qualification_event.yaml` authored with renamed entities.
- `schemas/material.yaml`: no change needed (already uses `supplier`).
- Extensions: rename pass applied in Supply Chain cluster Task 2.E.1.

**Follow-ups applied:**
- `entity-inventory.yaml`: `supplier.notes`, `approved_vendor_list_entry.notes`, `supplier_approval.notes` updated.
- `entity-matrix.md`: `supplier.canonical_framing` → ✅; `supplier_approval.canonical_framing` → 🟡 (alias will be collapsed when Supply Chain schemas land); `approved_vendor_list_entry.canonical_framing` → 🟡 (rename pending schema authoring).

---

## Decision 1.4 — Purchase Order Hierarchy

**Dated:** 2026-04-22

**Question:** Are `customer_purchase_order`, `sub_tier_purchase_order`, and a generic `purchase_order` three distinct entities, or is there a supertype/subtype hierarchy, or should they collapse with a discriminator?

**Options:**

- **A** — Generic `purchase_order` supertype with `po_kind: customer | sub_tier | material | opex` enum; `customer_purchase_order` and `sub_tier_purchase_order` become views/filters.
- **B** — Three distinct entities (no supertype). Avoids conflating inbound and outbound.
- **C** — Two-tier: `customer_purchase_order` stays distinct (inbound, different flowdown semantics); generic `purchase_order` covers all outbound (material, sub-tier, opex, etc.) with `po_kind` discriminator.

**Resolution:** **Option C — two-tier.**

Rationale:
1. `customer_purchase_order` is **inbound** (X12 850 from a prime to the manufacturer). Its role is the *origin* of the quality flowdown cascade: customer PO → job → sub-tier PO. It carries customer-imposed quality clause obligations. Conflating it with outbound POs would lose this directional semantics.
2. `sub_tier_purchase_order` and a generic material `purchase_order` are both **outbound** from the manufacturer to a supplier. They share structure (line items, supplier ref, delivery terms, quality clause refs) with the only distinction being `po_kind`. Maintaining three separate entities when two of them are structurally identical but for a discriminator is unnecessary complexity.
3. X12 850 maps to both directions; the resolution is that `customer_purchase_order` is the inbound X12 850 receipt, `purchase_order` is the outbound X12 850 issuance. Same transaction format, opposite directions, meaningfully different entity roles.

Concrete shape: `purchase_order` with `po_kind: material | sub_tier | services | opex`. `sub_tier_purchase_order` inventory row is retained as an archetype display alias; its canonical form is `purchase_order` with `po_kind: sub_tier`. `customer_purchase_order` stays distinct.

**Impact:**
- `schemas/purchase_order.yaml` in Supply Chain cluster: single schema with `po_kind` enum.
- `schemas/customer_purchase_order.yaml` in Supply Chain cluster: kept separate.
- `schemas/sub_tier_purchase_order.yaml`: **not authored**; alias of `purchase_order`.
- Finance & Lifecycle cluster: `sales_order` relationship to `customer_purchase_order` kept directional (sales_order is the manufacturer's internal commitment, customer_purchase_order is the customer document).

**Follow-ups applied:**
- `entity-inventory.yaml`: `purchase_order.notes`, `customer_purchase_order.notes`, `sub_tier_purchase_order.notes` updated.
- `entity-matrix.md`: `purchase_order.composition_rule`, `customer_purchase_order.composition_rule`, `sub_tier_purchase_order.composition_rule` updated.

---

## Decision 1.5 — Shipment Direction

**Dated:** 2026-04-22

**Question:** Is `shipment_inbound` a distinct entity, or is it `shipment` with a `direction` attribute?

**Options:**

- **A** — Generic `shipment` with `direction: outbound | inbound | return_to_supplier | return_from_supplier` enum. Both directions share carrier, tracking, dates, serials.
- **B** — Distinct `shipment` (outbound) and `shipment_inbound` entities. Explicit separation; duplicates many fields.
- **C** — Three-entity split: `outbound_shipment`, `inbound_shipment`, `receipt`. Overcomplicated for the data model.

**Resolution:** **Option A — generic `shipment` with `direction` enum.**

Rationale:
1. The attributes that differentiate inbound from outbound shipments (carrier, tracking number, manifest, included documents, date windows, certificate chain) are structurally identical. The direction determines the lifecycle states and integration touchpoints (X12 856 vs X12 861), not the schema shape.
2. The X12 856 vs X12 861 split is a *transport encoding* concern — it belongs in the integration layer (Layer 4), not the canonical ontology (Layer 2). The entity is "a physical movement of parts"; direction is an attribute of that movement.
3. Archetype B's `[Shipment Inbound]` display node (parts returning from Acme HT) has the same structure as the outbound `[Shipment]` to the customer: parts list, carrier ref, cert chain. One entity with a direction attribute serves both.
4. `receipt` is kept as a **distinct lifecycle event entity** (not a shipment subtype). A receipt is the goods-received event (triggers inventory movement, receiving inspection, cert logging). A shipment is the physical transit. These are different events in different lifecycle stages.

`shipment_inbound` inventory row is retained as an archetype display alias; canonical form is `shipment` with `direction: inbound`.

**Impact:**
- `schemas/shipment.yaml` in Supply Chain cluster: single schema with `direction` enum; two state-machine sections (one per direction).
- `schemas/receipt.yaml` in Supply Chain cluster: stays distinct.
- `schemas/shipment_inbound.yaml`: **not authored**; alias.

**Follow-ups applied:**
- `entity-inventory.yaml`: `shipment.notes`, `shipment_inbound.notes` updated.
- `entity-matrix.md`: `shipment.composition_rule`, `shipment_inbound.composition_rule` updated.

---

## Decision 1.6 — Certification Document Supertype

**Dated:** 2026-04-22

**Question:** Is `certification_document` an abstract supertype with typed subtypes (MTR, CoC, DFARS Declaration, etc.), or are those subtypes peer entities with no common parent?

**Options:**

- **A** — `certification_document` as abstract supertype with `cert_type` discriminator. Sub-schemas per cert type. Existing `extensions/material-certs.md` YAML already uses this shape.
- **B** — MTR, CoC, DFARS Declaration, PMI Report, ShelfLife, ConflictMineral as distinct first-class entities with no common parent. Higher entity count; complicates the cert package which must list documents of any type.
- **C** — `certification_document` as concrete base with union-typed `fields` bag; subtypes as reference data only. Loses the typed sub-schema structure that makes MTR chemistry queryable.

**Resolution:** **Option A — supertype with discriminator. Existing `material-certs.md` structure confirmed.**

Rationale:
1. `extensions/material-certs.md` already defines `certification_document` with a `cert_type` enum and `mtr_fields`, `dfars_fields`, `conflict_mineral_fields` sub-schemas. This is Option A already implemented in prose. Phase 2 moves it to `schemas/certification_document.yaml` without structural change.
2. The cert package (`certification_package.contains_documents[]`) must reference heterogeneous document types. A common supertype makes this relationship clean: the package contains `certification_document` entities regardless of cert type. Peer entities would require a union type on the package edge.
3. Carbon novelty scan approval-request pattern supports discriminated supertypes for similar reasons. Convergent evidence.
4. `export_compliance_declaration` (new in Archetypes B/C) is added to the `cert_type` enum as `ExportCompliance` — it has the same structure (issuer, date, references, recipient) and belongs on the same supertype.
5. `process_certification` stays **distinct** from `certification_document`. A process cert is issued by an outside processor attesting to process conformance (heat treat, plating, NDT). A material cert is issued by a mill or distributor attesting to material composition. Different issuers, different scopes, no useful shared base.

Resolved `cert_type` enum: `MTR | CoC | DFARS_Declaration | PMI_Report | ConflictMineral | ShelfLife | ExportCompliance`

Subtype rows in the inventory that are archetype display aliases with no standalone schema file:
- `mtr` → `certification_document` with `cert_type: MTR`
- `coc` → `certification_document` with `cert_type: CoC`
- `dfars_declaration` → `certification_document` with `cert_type: DFARS_Declaration`
- `pmi_report` → `certification_document` with `cert_type: PMI_Report`
- `conflict_minerals_declaration` → `certification_document` with `cert_type: ConflictMineral`
- `shelf_life_certification` → `certification_document` with `cert_type: ShelfLife`
- `export_compliance_declaration` → `certification_document` with `cert_type: ExportCompliance`

None of these seven get standalone schema files. They are discriminator values with per-cert-type `*_fields` sub-schemas within `schemas/certification_document.yaml`.

**Impact:**
- `schemas/certification_document.yaml` in Materials cluster: single schema with full `cert_type` enum and typed `*_fields` sub-objects per cert type.
- No `schemas/mtr.yaml`, `schemas/coc.yaml`, etc.
- `extensions/material-certs.md`: add "Canonical schema" cross-reference links per cert-type section.
- `fai_package.yaml` and `certification_package.yaml`: `contains_documents[]` types to `certification_document`.

**Follow-ups applied:**
- `entity-inventory.yaml`: seven subtype rows + `certification_document` row notes updated.
- `entity-matrix.md`: `certification_document.composition_rule` → ✅; seven subtype `composition_rule` rows → 🟡 (resolved as subtypes; schema authoring pending in Materials cluster).

---

## Decision 1.7 — String vs Entity: heat_number, uns_number, country, spec_body, nadcap_category

**Dated:** 2026-04-22

**Question:** For each of these five entities currently appearing as archetype nodes, should they be first-class entities (with queryable attributes beyond their string value) or validated string attributes on the entities that reference them?

**Governing rule:** If the entity has queryable sibling/parent data (registry lookup, alloy family membership, scope detail, audit body) beyond its string value, it is a first-class entity. If it is a pure coded string with no additional queryable facet, it is a validated string attribute.

**Per-item resolutions:**

**`heat_number` → validated string**
Mill's internal heat/melt identifier. The additional data associated with a heat (mill source, melt date, chemistry actuals) lives on `material_lot` and on the `mtr` cert document, not on the heat number string itself. Making `heat_number` an entity would create a record whose only content is what already exists on its parent. Canonical form: `material_lot.heat_number: string (required)` — a validated opaque identifier.

**`uns_number` → validated string**
Unified Numbering System code (e.g., `A97075`, `R56400`). Pattern: `^[A-Z]\d{5}$`. Alloy-family membership and equivalent-spec cross-references live on the `material` entity (which already has `alloy_family` and `spec_references[]` attributes). Making `uns_number` an entity would duplicate what `material` already knows. Canonical form: validated string field on `material` and `material_specification`.

**`country` → validated string**
ISO 3166-1 alpha-2 code (e.g., `US`, `DE`). The DFARS qualifying-countries membership is a reference list (maintained by DFARS 252.225-7009) applied as a lookup at compliance-checking time — not a per-country entity in the ontology. Canonical form: validated string (ISO 3166-1 alpha-2) on `material_lot.melt_country`, `material_lot.manufacture_country`, `supplier.country`.

**`spec_body` → enum**
Standards-issuing body. Already implemented as an enum in `schemas/material.yaml` (`SAE | ASTM | MIL | ASME | OEM | Internal`). The NASM body was missing from the enum; add it. A spec body has no queryable attributes beyond its name — it does not have a registry record, a certification scope, or an expiry. Canonical form: enum `SAE | ASTM | MIL | ASME | NASM | OEM | Internal` on `material_specification.spec_body` and `process_specification.spec_body`.

**`nadcap_category` → first-class entity**
Nadcap audit scope category (e.g., `AC7004` = Heat Treating, `AC7102` = Chemical Processing). A `nadcap_category` carries queryable attributes beyond its code: the checklist number (AC7xxx), subscriber primes, the PRI auditing body, and the scope definition. `approved_supplier_list_entry.nadcap_accreditations[]` references nadcap_category entries. Making it a first-class entity enables querying "which suppliers hold AC7004 accreditation" without a text-match on a string field.

**Impact:**
- `heat_number`, `uns_number`, `country` inventory rows: reclassified as validated string attributes; no schema files for these three.
- `spec_body` inventory row: reclassified as enum; update `schemas/material.yaml` to add `NASM` value.
- `nadcap_category` stays first-class; `schemas/nadcap_category.yaml` authored in Supply Chain cluster.
- Materials cluster schema authoring: drops four schema tasks; nadcap_category task moves to Supply Chain cluster.

**Follow-ups applied:**
- `entity-inventory.yaml`: five rows' notes updated.
- `entity-matrix.md`: `heat_number`, `uns_number`, `country`, `spec_body` composition_rule rows → ✅ (resolved as string/enum; no schema needed). `nadcap_category.composition_rule` → 🟡 (entity confirmed; schema authoring pending in Supply Chain cluster).

---

## Decision 1.8 — Quality Engineer as Entity vs Role

**Dated:** 2026-04-22

**Question:** Is `quality_engineer` a distinct entity type, a role on a generic `user` entity, or an attribute on `witness_inspection`?

**Options:**

- **A** — Distinct entity `quality_engineer` with its own schema (competency records, audit trail, certifications). High entity count; duplicates user management.
- **B** — Role on `user`: `user.roles[]` enum includes `quality_engineer`, `machinist`, `buyer`, etc. User entity defined as a canonical entity with role-bearing.
- **C** — Attribute on `witness_inspection`: `inspector_role` enum field. Loses the actor identity needed for ITAR citizenship tracking and AS9100 competency records.

**Resolution:** **Option B — `quality_engineer` is a role on `user`.**

Rationale:
1. AS9100 §7.2 competency requirements attach to the *person*, not to an abstract role entity. The record of "this person holds this competency" is a `user` attribute, not a standalone entity.
2. ITAR citizenship status (US Person / Foreign Person) applies to the individual, not to the role. A `quality_engineer` entity would need to carry the citizenship attribute anyway, at which point it is functionally a `user` with a filter.
3. Archetype C's `[Quality Engineer]` node connects to `witness_inspection.performed_by`. With Option B, this edge targets a `user` entity where the guard constraint is `user.roles contains quality_engineer`. The archetype display label `[Quality Engineer]` is preserved as a display alias.
4. Defines `user` as a canonical Layer-2 entity (it was in the "referenced but undefined" group). `user` carries: `id`, `name`, `email`, `roles[]` (enum), `citizenship_status` (US_Person | Foreign_Person), `training_records[]`, `active_flag`.

`quality_engineer` inventory row: retained as archetype display alias pointing to "user with quality_engineer role." No `schemas/quality_engineer.yaml`.

Roles enum: `quality_engineer | machinist | inspector | buyer | shipping | receiving | programmer | operator | supervisor | admin`

**Impact:**
- `schemas/user.yaml` in Finance & Lifecycle cluster: new entity definition (one of the seven "referenced but undefined" entities).
- `schemas/witness_inspection.yaml` in Routing & Operations cluster: `inspector_user_id` ref targets `user` with role guard.
- `schemas/nonconformance_report.yaml` in Quality cluster: `inspector_id` ref updated to `user`.
- `quality_engineer` inventory row: alias; no schema task.

**Follow-ups applied:**
- `entity-inventory.yaml`: `quality_engineer.notes`, `user.notes` updated.
- `entity-matrix.md`: `quality_engineer.composition_rule` → 🟡 (resolved as role alias; user schema pending). `user.composition_rule` → 🟡 (entity defined; schema authoring pending in Finance & Lifecycle cluster).

---

## Decision 1.9 — Cert-Package Nesting + Item Supertype

**Dated:** 2026-04-22

**Question (a):** Does an assembly-level `certification_package` nest child `certification_package` entities by reference, or is it a flat aggregation of all underlying documents?

**Question (b):** Is `item` a canonical supertype of `part`, `material`, and `fastener`, or is it an ERP-implementation concept removed from canon?

**Options — cert-package nesting:**

- **A** — Nested: `certification_package.contains_packages[]` holds child packages by reference. Each child package is still independently shippable and signable.
- **B** — Flat: assembly cert package is the union of all underlying documents with no nested package concept.

**Options — item supertype:**

- **A** — `item` is a canonical Layer-2 supertype (part, material, fastener as subtypes).
- **B** — `item` is an ERP-impl echo (Carbon also uses it). Remove from canon.

**Resolution — cert-package: Option A — nested.**

Rationale:
1. Archetype C explicitly shows a gimbal-bracket-assembly cert package that contains the cert packages of each of the three constituent brackets. Each constituent bracket cert package is separately signed, separately submittable, and separately identifiable. Flat aggregation would lose this traceability structure.
2. A nested cert package for an assembly is independently shippable (the sub-assembly cert can be accepted by the customer before the full assembly is complete). Flat aggregation prevents independent use.
3. Aerospace customers frequently require "traceability to sub-assembly cert package" as a distinct audit reference. The nesting makes this reference first-class rather than requiring document-list traversal.

Schema: `certification_package.contains_packages[]` (one-to-many, refs to child `certification_package` entities) + `contains_documents[]` (one-to-many, refs to `certification_document` entities).

**Resolution — item supertype: Option B — removed from canon.**

Rationale:
1. The canonical ontology has `part`, `material`, and `assembly` as distinct entities. There is no Layer-1 standard that introduces an "item" generalization that is useful in the manufacturer layer — STEP AP242 has "product" (not "item"), ISA-95 has "material class" (not "item"). The `item` vocabulary is specific to ERP item-master and ERP unified-item-master pattern.
2. Introducing `item` as a supertype would require every relationship that currently targets `part` or `material` to be retyped to `item`, degrading semantic precision. A `quality_flowdown_plan` doesn't attach to "an item" — it attaches to a `job`, which in turn produces a `part` (or `assembly`).
3. Where extensions currently use `item_id` (e.g., `first_article_inspection.item_id`, `nonconformance_report.item_id`), these are updated to reference `part | assembly` as a typed union in Phase 2 schema authoring.
4. `item` inventory row: marked "ERP-impl only, not canon." No `schemas/item.yaml`. Removed from Phase 2 entity-addition list.

**Impact:**
- `schemas/certification_package.yaml` in Finance & Lifecycle cluster: `contains_packages[]` and `contains_documents[]` both typed.
- `schemas/first_article_inspection.yaml` in Quality cluster: `item_id` → `part_id | assembly_id` typed union; same for `schemas/nonconformance_report.yaml`.
- `schemas/item.yaml`: **not authored**.
- Archetype display: `[Item]` display nodes in archetype walks remain as informal shortcuts; canonical references are to `part` or `assembly`.

**Follow-ups applied:**
- `entity-inventory.yaml`: `certification_package.notes`, `item.notes` updated.
- `entity-matrix.md`: `certification_package.composition_rule` → ✅ (nesting confirmed). `item.composition_rule` → ⬜ (removed from canon; N/A). `item.canonical_framing` → 🟡 updated to note ERP-impl only.
