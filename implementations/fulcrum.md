<!-- Created 2026-04-22 as part of the coherence pass.
     Fulcrum is a reference implementation, not the canonical ontology.
     See ../docs/superpowers/specs/2026-04-22-ontology-comprehensive-pass-design.md §8. -->

# Fulcrum Pro — Reference Implementation Mapping

Cross-reference between this ontology and Fulcrum Pro, a SaaS ERP for precision manufacturers. Based on the Fulcrum Pro public API (see `fulcrum-pro-mcp/docs/fulcrum-api-reference.md`) and operational experience running Fulcrum as the system of record for jobs, routing, time tracking, inventory, costing, and quality records.

Fulcrum Pro is one reference implementation among several; the canonical ontology lives in `../reference/`, `../extensions/`, and `../schemas/`. Where the ontology demands an entity Fulcrum does not model, the implementer stores the attribute in JSONB custom fields per the discipline in `../extensions/uuid-discipline.md` §"UUID Storage in Fulcrum Pro". Where even custom fields do not fit, the implementer attaches PDFs or maintains external records.

## Architecture Overview

- **Stack:** Hosted multi-tenant SaaS. REST API (OpenAPI-described). No webhooks — integrations poll. API list endpoints capped at 50 rows per page.
- **Storage pattern for ontology extensions:** `customFields` JSONB object on major entities (Item, Job, Job Operation, Inventory Lot, Receipt, Quote, Quote Line, Sales Order, Sales Order Line, Purchase Order, Shipment). Custom fields are defined via a registry (`/api/custom-fields`) scoped to a module (Item, Job, etc.) and rendered inline in the Fulcrum UI. the implementer namespaces ontology UUIDs with a shop prefix (e.g. `shop_job_uuid`).
- **Notable architectural properties:** Fulcrum is the system of record for production and costing. It is the Layer 2 anchor for UUID discipline — job and job-operation UUIDs flow from Fulcrum into MTConnect and QIF. No native revision graph on items (revisions are separate item records linked through an item-revision reference rather than through a normalized version table). No native row-level security — access is scoped by user role and company tenant.

## Domain Mapping

Keep this structure in sync with `../reference/domain-map.md` — twelve domains (Domains 1–8 original; Domains 9–12 added 2026-04-24: Process Engineering/NRE, Tool Room, Packaging, Change Management). Compliance and Governance is covered inside the Sales and Supply Chain sections because Fulcrum has no dedicated compliance module. Domains 9–12 are not yet mapped in this file — Fulcrum coverage for these domains is minimal.

### Materials

| Ontology Entity | Fulcrum Entity | Fulcrum Fields | Gaps |
|---|---|---|---|
| material | `Material` (`MaterialShapeDto`) | `name`, `form`, `materialReferenceId`, `specification` (free-text), `subSpecification`, `finish`, `grade`, `dimension`, `density`, `type`, `version`, `vendors[]` | `specification` is a string, not a link to a structured spec entity |
| material (as inventory) | `Item` with `materialDetails` (`ItemMaterialDetailsDto`) | `materialCodeId`, `gradeId`, `shapeId`, `gaugeId`, `form`, `materialWidth`, `materialLength`, `materialThickness`, `isRemnant` | Lookup tables for grade/shape/gauge are flat strings; no UNS, AMS, ASTM cross-reference |
| material_specification | -- | -- | **GAP** — no first-class spec entity. AMS/ASTM/MIL numbers ride on free-text `specification` or on the item number |
| material_condition | -- | -- | **GAP** — tempers like T7351, H1025, Annealed are embedded in grade or item description |
| uns_number | -- | -- | **GAP** — no UNS registry; cross-reference is ad-hoc |
| material_lot | `Inventory Lot` (`InventoryLotDto`) | `name` (lot identifier), `itemId`, `quantity`, `expirationDate`, `custom` | `customFields` JSONB: `ffm_lot_uuid`, `ffm_heat_number`, `ffm_melt_country`, `ffm_dfars_qualifying`, `ffm_manufacturer` |
| heat_number | Inventory Lot JSONB (`ffm_heat_number`) | -- | Not a first-class entity; string attribute on the lot |
| certification_document | `Attachment` on Inventory Lot or Receipt | PDF with media-format tag | No structured MTR/CoC/DFARS fields; PDF is the source of truth |
| pmi_event / pmi_report | -- | -- | **GAP** — XRF/OES results logged as attachment + NCR if out-of-spec; no structured event |
| shelf_life_certification | Inventory Lot `expirationDate` + attachment | -- | Expiration date only; the certifying document is an attached PDF |

**Assessment:** Fulcrum's `Material` entity is close to Brad Barbin's structured-material schema (form + grade + dimension composition) and is richer than most SaaS ERPs. It stops at the catalog and purchasing layer — the aerospace specification layer (AMS number, type, class, condition, chemistry limits, DFARS qualifying origin) is not modeled. the implementer carries that layer in Inventory Lot custom fields and supporting PDFs. Specification governance — "what is AMS 4045 Rev T and what are its valid conditions?" — has no home in Fulcrum.

### Items & BOMs

| Ontology Entity | Fulcrum Entity | Fulcrum Fields | Gaps |
|---|---|---|---|
| item / part | `Item` (`ItemDto`) | `number`, `description`, `itemOrigin`, `unitOfMeasureName`, `revision`, `barCodeNumber`, `isSellable`, `isLotTracked`, `categoryId`, `materialDetails`, `qualityPlanStatus`, `customFields` | No native itemType enum for Raw Material / Phantom / Outside-Processing-as-Item |
| item revision | `Item` with `revision` (`ItemRevisionDto`) + `/items/{itemId}/revision` endpoint | Each revision is a separate Item record linked through revision reference | No revision graph; supersession tracking is a manual attribute |
| part (ontology UUID) | Item JSONB (`ffm_part_uuid`, `ffm_customer_part_ref`) | -- | Customer CAGE + P/N stored as custom field string |
| item_category | `Item Category` (`/api/item-categories`) | `name` | Flat taxonomy; no hierarchy |
| item_class | `Item Class` (`/api/item-classes`) | `name` | Flat classification |
| CAGE code | -- | -- | **GAP** — shop CAGE and customer CAGE live in JSONB or the item description |
| export_classification | -- | -- | **GAP** — ITAR / EAR99 / USML designation is a custom field string; no access-control enforcement |
| bom_line / input_material | `Item Routing Input Material` (`/api/items/{itemId}/routing/input-materials`) | `costing` (`CommonEnumMaterialRequirementCostingEnum`), nesting definitions, quantity per parent | Materials are attached to operations, which is good — mirrors the "Methods" pattern |
| bom_line / input_item | `Item Routing Input Item` (`/api/items/{itemId}/routing/input-items`) | Links sub-items consumed by a routing step | Supports multi-level assemblies through recursive references |
| routing | `Item Routing` (`/api/items/{itemId}/routing`) | Attaches operations, input items, input materials to an item | Unified BOM+routing is a strength |
| routing operation (definition) | `Item Routing Operation` (`/api/items/{itemId}/routing/operations`) | `setupTime`, `laborTime`, `machineTime`, `isOutsideProcessing`, `defaultVendorId`, `outsideProcessingCost`, `leadDays`, `instructions` | Instructions are free-text only; no versioned procedure entity |
| part_specification (drawing) | `Attachment` on Item | PDF drawing linked via attachment | No structured drawing-number + revision + design-activity CAGE entity |

**Assessment:** Fulcrum has a working BOM+routing model and its nesting support for sheet and bar stock is the best we have seen from a SaaS ERP. Revision handling is the weakest link: each revision is a separate item, and supersession is tribal knowledge rather than a first-class relationship. For aerospace, export classification and CAGE codes have no proper home — they live as strings in custom fields and are unenforceable at the data layer.

### Manufacturing Execution

| Ontology Entity | Fulcrum Entity | Fulcrum Fields | Gaps |
|---|---|---|---|
| job | `Job` (`JobDto`) | `id`, `number`, `parentItemId`, `quantityToMake`, `productionDueDate`, `salesOrderId`, `status` (9-state enum), `priority`, `workOrderIds`, `customFields`, `revenue` | JSONB: `ffm_job_uuid`, `ffm_customer_part_ref`, flowdown plan references |
| operation (per-job) | `Job Operation` (`/api/jobs/{jobId}/operations/list`, `get_job_operations`) | Inherits fields from the routing operation plus per-unit completion tracking | JSONB: `ffm_op_uuid` — the MTConnect `<WorkOrderId>` anchor |
| job_material (issue) | `Job Routing Input Material` / `Job Routing Input Item` | Material issues logged against a specific job routing step | |
| serial_number | `Inventory Lot` in single-unit mode | Lot-as-serial pattern; `name` = SN string; JSONB: `ffm_serial_uuid` | No distinct serial entity; serialization overloads the lot table |
| machine_event | -- (external MTConnect agent) | Fulcrum records job timer events (`/api/timers`, `/api/job-tracking-timers`) but not spindle-level telemetry | MTConnect stream lives outside Fulcrum and is joined via `ffm_op_uuid` |
| work_center | `Work Center` (`/api/work-centers`) | `name` and display metadata | No structured capability attributes; capability matching is tribal |
| equipment | `Equipment` (`/api/equipment`) | Individual machines; links to work centers | |
| routing operation (in-process tracking) | `In-Process Tracking Field` (`/api/in-process-tracking-field-types`) and per-job IPTs | Per-operation data capture fields (checklists, measurements) | Limited type system compared with Carbon's `procedureStep` typing |
| operation dependencies | Fulcrum operation sequence (ordered list) | Linear sequence via operation order | **GAP** — no DAG, only linear |
| procedure / procedure step | `instructions` free-text + attached PDF | -- | **GAP** — no structured work-instruction versioning |
| process_specification | -- | -- | **GAP** — in-house programming procedures and outside AMS specs live as attached PDFs and free-text spec references |
| outside_process_operation | `Item Routing Operation` with `isOutsideProcessing = true` + `Purchase Order Outside Processing Line Item` | Operation flags outside work; the linked PO line carries the vendor and cost | Integration is functional but the ontology's hybrid-entity view (operation + PO + cert collection event) is not unified — three separate Fulcrum records |

**Assessment:** Fulcrum's manufacturing execution covers the shop-floor timing, completion, and cost-rollup cases well. Job and operation UUIDs are stable and serve as the canonical MTConnect work-order anchors. What is missing is the aerospace structure on top: process specifications as entities, Nadcap categorization, structured procedures, and a DAG-sequenced routing. the implementer compensates by placing in-house SOPs in SharePoint and pointing Fulcrum operations at them by number.

### Quality

| Ontology Entity | Fulcrum Entity | Fulcrum Fields | Gaps |
|---|---|---|---|
| nonconformance_report | `NCR` (`NCRNcrDto`) | `number`, `type`, `cause`, `reason`, `detail`, `status`, `departmentId`, `vendorId`, `workCenters`, `scheduledEquipment`, `operator`, `quantityImpacted`, `incrementalCost`, `jobIds`, `customFields`, `impacts[]` | Types/causes are free-text strings, not enum-constrained |
| mrb_disposition | NCR JSONB (`ffm_mrb_status`, `ffm_mrb_disposition`, `ffm_mrb_authorized_by`) | -- | **GAP** — no MRB event entity with attendees, minutes, customer deviation references |
| capa | `CAPA` (`CAPACapaDto`) | `capaType`, `problemStatement`, `rootCauseAnalysis`, `correctiveAction`, `ncrId`, `dueDate`, `closedUtc` | Free-text fields; no structured 8D or 5-why workflow |
| first_article_inspection | -- | -- | **GAP** — no FAI entity. FAI records live as attached PDFs on the Job plus JSONB fields |
| as9102_form_1 / form_2 / form_3 | -- | -- | **GAP** — no structured AS9102 forms |
| fai_package | `Attachment` on Job | PDF bundle | No linkage from FAI unit serial to characteristic results beyond the attached PDF |
| key_characteristic | Quote Line or Item JSONB | -- | **GAP** — KC balloons are captured in drawings and QIF files, not as structured records |
| characteristic_measurement | QIF results file (attachment) + JSONB on Job | -- | **GAP** — measurements are PDFs or QIF XML; individual characteristic rows are not queryable |
| qif_results_document | `Attachment` on Job | File reference; UUID carried via `ffm_inspection_event_uuid` | Fulcrum does not parse QIF |
| inspection_event | `Attachment` + Job JSONB | -- | **GAP** — no inspection-event entity; CMM sessions are logged as attachments |
| calibration_record / tool_gauge | -- | -- | **GAP** — gauge calibration is tracked in a spreadsheet outside Fulcrum |

**Assessment:** Fulcrum has an NCR and CAPA table but not an aerospace quality record layer. FAIs, inspection events, characteristic measurements, key characteristics, and gauge calibration all rely on PDFs, QIF files, and a spreadsheet. This is the implementer's largest structural gap against the canonical ontology. The ontology's `first_article_inspection` extension is where the structure should live; Fulcrum just holds the attachment and the JSONB UUID pointers.

### Supply Chain

| Ontology Entity | Fulcrum Entity | Fulcrum Fields | Gaps |
|---|---|---|---|
| vendor / supplier | `Vendor` (`VendorDto`) | `name`, addresses, contacts, payment terms | Flat contact record |
| vendor_contact | `Vendor Contact` (`/api/vendors/{vendorId}/contacts`) | `name`, `email`, `phone` | |
| approved_vendor_list_entry / supplier_approval | Vendor JSONB (`ffm_approval_status`, `ffm_approval_scope`, `ffm_approval_expiry`) | -- | **GAP** — no structured AVL record. Nadcap scopes, AS9100 cert, customer approvals, DFARS qualification all ride on JSONB or attachments |
| nadcap_certification | Attachment on Vendor | PDF | **GAP** — no category, certificate number, expiration, audit body, subscribing primes as structured fields |
| customer_asl | -- | -- | **GAP** — customer-supplied ASLs are not modeled; vendor selection relies on buyer knowledge |
| vendor_qualification_event | `Note` on Vendor | Free-text timeline | No audit-trail entity for approval, annual review, suspension, reinstatement |
| purchase_order | `Purchase Order` (`PurchaseOrderDto`) | `type` (`PurchaseOrderTypeEnum`), `status`, vendor, payment terms, `customFields` | JSONB: `ffm_quality_clause_codes[]`, `ffm_dfars_required`, `ffm_nadcap_required` |
| customer_purchase_order | `Sales Order` (inbound PO context) | Customer PO number carried as a reference on Sales Order | **GAP** — customer PO is not a distinct entity; its number lives as a field on the Sales Order |
| sub_tier_purchase_order | `Purchase Order` with outside-processing line items | Same PO entity with `PurchaseOrderOutsideProcessingLineItemDto` | The ontology distinguishes; Fulcrum merges |
| PO line (part) | `Purchase Order Part Line Item` | Item ref, quantity, unit price, delivery date | |
| PO line (outside processing) | `Purchase Order Outside Processing Line Item` | Links to source Job and operation | Enables operation-to-PO-line round-trip |
| quality_clause | -- | -- | **GAP** — clause codes (QA-001…QA-017) attach as prose on the PO or as a custom field string array |
| quality_flowdown_plan | -- | -- | **GAP** — no flowdown-plan container; flowdown is a tribal mapping from customer clause to sub-tier clause |
| receipt | `Receipt` (`/api/receiving/receipts`) | `status`, source PO ref, receipt line items, `customFields` | JSONB: `ffm_inspection_event_uuid`, `ffm_receiving_disposition` |
| receipt_line | `Receipt Line Item` | Quantity received, lot assignment | |
| process_certification | `Attachment` on Receipt or Job | PDF | **GAP** — outside-processor CoC / test report / process chart / hardness survey / NDT report / Nadcap cert copy live as attached PDFs; no structured fields |

**Assessment:** Fulcrum handles the mechanical supply-chain workflow — vendors, POs, receipts — cleanly, and the outside-processing PO line item is a genuine feature. What is missing is everything the aerospace overlay requires: structured AVL records with Nadcap scopes, quality clause catalogs, flowdown plans, DFARS qualification tracking, and process certifications as structured records. These all collapse into attachments and custom-field strings, which is workable for a small shop but does not scale to audit-on-demand.

### Sales & Quoting

| Ontology Entity | Fulcrum Entity | Fulcrum Fields | Gaps |
|---|---|---|---|
| customer | `Customer` (`CustomerDto`) | `name`, addresses, contacts, customer tier, payment terms, `customFields` | JSONB: `ffm_customer_cage`, `ffm_customer_approval_status` |
| customer_contact | `Customer Contact` (`/api/customers/{customerId}/contacts`) | `name`, `email`, `phone`, `title` | |
| customer_purchase_order | `Sales Order` (`SalesOrderDto`) with PO-number field | Carries customer PO reference | No distinct customer-PO entity |
| sales_order | `Sales Order` | `number`, `status`, line items, `customFields` | JSONB: flowdown plan references, ITAR flag |
| sales_order_line | `Sales Order Part Line Item` | Item, quantity, price, delivery; per-line routing mirror | Line-level routing mirror enables estimating-to-production fidelity |
| blanket_order | `Sales Order Blanket Line Item` | Long-running blanket with draws | |
| quote | `Quote` (`QuoteDto`) | `number`, `status`, line items, per-line routing mirror, `customFields` | JSONB: `ffm_quote_uuid`, `ffm_export_classification`, `ffm_quality_clauses[]` |
| quote_line | `Quote Part Line Item` | Item ref, quoted price, per-line cost breakdown, per-line routing mirror | Triple-mirror (Quote routing → Sales Order routing → Job routing) is supported |
| opportunity / rfq | -- | -- | **GAP** — no opportunity pipeline; RFQ-to-quote is tribal |
| customer_approval (implementer as approved source) | Item JSONB or Customer JSONB | -- | **GAP** — approval per part-revision not structured |
| export_classification | -- | -- | **GAP** — EAR99 / ITAR / USML designation stored as JSONB strings with no enforcement |
| contract / program | -- | -- | **GAP** — long-term agreements and program identifiers live in spreadsheets |

**Assessment:** Fulcrum's quote-to-job flow is strong and the per-line routing mirror preserves the estimating context all the way to the job. The gaps are on the compliance side: export classification, program-level contracts, and customer-approval state have no structured home. For a shop doing ITAR work, this means the access boundary lives in naming conventions and user training rather than in the data model.

### Inventory

| Ontology Entity | Fulcrum Entity | Fulcrum Fields | Gaps |
|---|---|---|---|
| inventory (on-hand) | `Inventory` (`/api/inventory`) | `itemId`, `lotId`, `locationId`, `quantity` | Ledger-style view available via inventory transactions |
| inventory_event | `Inventory Event` (`InventoryEventDto`) | Receive / pick / override / transfer events, cost fields | Event type enum covers common operations |
| inventory_transaction | `Inventory Transaction` | Each movement with cost tracking | |
| inventory_lot | `Inventory Lot` (`InventoryLotDto`) | `name`, `itemId`, `quantity`, `expirationDate`, `system`, `custom` | JSONB: `ffm_lot_uuid`, `ffm_heat_number`, `ffm_melt_country`, `ffm_dfars_qualifying` |
| material_lot (physical) | Inventory Lot with item type = Material | Same as above | |
| heat_number | Inventory Lot JSONB (`ffm_heat_number`) | -- | Not a first-class entity |
| inventory_location | `Location` (`/api/locations`) | Physical site with addresses | |
| sub-location / bin | -- | -- | **GAP** — no shelf/bin below Location; lot moves between locations only |
| tool_gauge | `Equipment` (partial) | Machines tracked; cutting tools and gauges tracked outside Fulcrum | **GAP** — no gauge table; cal records in spreadsheet |
| quarantine location | Named `Location` ("MRB Hold") | Convention, not a flag | **GAP** — no quarantine designation on Location; flag is the location name |
| serial tracking | Inventory Lot in single-unit mode | Lot name = serial number | No distinct serial entity |
| inventory pick / issue | `/api/inventory/pick`, `InventoryPickDto` | `itemId`, `quantity`, `lotId`, `locationId`, `secondaryType` (pick / consume / scrap / stockAdjustmentDecrease / otherDecrease) | |
| inventory receive | `/api/inventory/receive`, `InventoryReceiveDto` | Same structure; labor / machine / outside-processing / material value fields for cost tracking | |
| mrp / planning | -- | -- | **GAP** — MRP runs as nightly job outside the API surface; demand and supply forecasts not exposed as entities |

**Assessment:** Fulcrum's inventory event and cost-carrying receive/pick are solid. The gaps are below the Location level (no bins), in resource tracking (no gauge table), and in MRP exposure. the implementer relies on the Inventory Lot custom-field discipline to carry heat number and DFARS status — that is the ontology's traceability chain holding together through JSONB.

## Gaps Fulcrum Doesn't Cover Natively

Entities the ontology requires that have no representation in Fulcrum beyond JSONB, prose, attachments, or external integrations:

- **material_specification as a first-class entity.** AMS, ASTM, MIL specs with types, classes, conditions, chemistry limits, and cross-references. Fulcrum exposes a free-text `specification` string on Material and nothing more.
- **quality_flowdown_plan and quality_clause.** No structured clause catalog, no clause-to-PO attachment, no customer-clause-to-sub-tier-clause mapping. Clauses ride as custom-field string arrays and as prose on the PO.
- **outside_process_operation as a hybrid entity.** Fulcrum has the pieces (operation flagged outside + PO outside-processing line item + receipt + attachment) but they are three records linked by references rather than one hybrid node with cert-collection semantics.
- **first_article_inspection and AS9102 Form 1/2/3.** No structured FAI; records live as attached PDFs on the Job with JSONB pointers.
- **nadcap_certification and nadcap_category.** No structured audit scope records. Categories (HT/CP/NDT/WLD/CT/SEAL/FSS/E) and subscribing primes are not reference data.
- **quality_clause catalog.** The QA-001 through QA-017 catalog has no table in Fulcrum; clause codes are strings.
- **dfars_declaration and DFARS qualification tracking.** The DFARS 252.225-7009 specialty-metals declaration lives as a JSONB boolean on the Inventory Lot plus an attached PDF; no structured declaration entity.
- **export_classification.** ITAR, EAR99, USML category designations are JSONB strings on Quote, Sales Order, and Item. No access-control enforcement at the data layer — enforcement is by user role and naming convention.
- **approved_vendor_list_entry.** No structured AVL. Nadcap scopes, AS9100 cert, customer approvals, implementer internal qualification, and expiration dates are JSONB custom fields.
- **process_specification.** In-house programming procedures and outside AMS process specs (AMS 2759/3, AMS 2700, NASM 33540, AS5272) are attached PDFs referenced by free-text spec numbers.
- **process_certification.** Outside-processor CoC, test report, process chart, hardness survey, thickness report, NDT report, and Nadcap cert copies are attached PDFs on Receipt or Job.
- **key_characteristic and characteristic_measurement.** KC balloons and per-characteristic measurements live in drawings and QIF files; Fulcrum does not parse them.
- **calibration_record and tool_gauge.** No gauge table; calibration in a spreadsheet.
- **inspection_event and qif_results_document (structured).** Fulcrum stores the QIF file as an attachment and carries the UUID in JSONB but does not parse the QIF or surface characteristic-level queries.
- **data_classification and ip_boundary governance.** Customer-owned, implementer-owned, and quality-record classifications have no structured attribute; enforcement relies on SharePoint and folder conventions.

## Patterns Worth Preserving From the Fulcrum Experience

1. **JSONB custom fields scale further than expected.** Every major Fulcrum entity (Item, Job, Job Operation, Inventory Lot, Receipt, Quote, Sales Order, Purchase Order, Shipment) supports `customFields`. the implementer carries roughly 30 ontology extension attributes this way — UUIDs, heat numbers, melt country, DFARS flags, export classifications, quality clause arrays, flowdown plan references. The pattern is good enough to run production on and leaves room for a future structured-table migration. A canonical ontology implementation should assume custom-field escape hatches will be used for the aerospace extensions on day one.
2. **Polling architecture forces explicit snapshot boundaries.** Fulcrum has no webhooks. Every integration polls `/api/jobs/list`, `/api/inventory-events/list`, `/api/purchase-orders/list`, etc., and tracks the last-seen `modifiedUtc` or event timestamp. This is painful in isolation but has a hidden benefit: it forces every consumer (shop-floor MES, MTConnect correlator, scheduler) to define a snapshot boundary and a reconciliation strategy. Webhook-driven systems tend to skip that discipline. A canonical implementation should still prefer webhooks — but it should model the polling-compatible snapshot boundary as a first-class concept.
3. **The 50-row pagination cap is a real constraint.** Every list endpoint caps at 50 per page. Large fetches require pagination loops with stable sort. This shaped how `fulcrum-pro-mcp` MCP tools are written and how scheduled jobs batch their reads. The operational lesson: assume pagination caps exist for any SaaS ERP and design ingestion around them from day one.
4. **Per-line routing mirror (Quote → Sales Order → Job) preserves estimating context to production.** Each Quote Part Line Item has its own routing tree, which is cloned into the Sales Order Part Line Item routing and into the Job routing. Variance between quoted and actual cost is measurable at the operation level because the operations match by identity. This mirrors Carbon's triple-mirroring pattern and is worth preserving in the canonical ontology — the `quote_line`, `sales_order_line`, and `job` entities should share a routing shape.
5. **Outside-processing line items are unified with the PO, not separated.** Fulcrum's `PurchaseOrderOutsideProcessingLineItemDto` lets a single PO carry both part-purchase lines and outside-processing-service lines, with the outside-processing line holding a reference to the originating Job and operation. This preserves the operation ↔ PO round-trip without needing a separate "service order" entity. The ontology's `outside_process_operation` hybrid entity fits this shape naturally.
6. **Job status as a 9-state enum captures the lifecycle without custom states.** `draft → needsReview → approved → engineering → scheduled → inProgress → complete / cancelled / hold` covers the production lifecycle cleanly. this implementer has not needed to invent custom states. The canonical ontology's job lifecycle should borrow this shape.

## Known Deviations From the Canonical Ontology

Places where Fulcrum's model conflicts with — not just omits — canonical-ontology entities:

- **Customer purchase order is collapsed into Sales Order.** The ontology treats `customer_purchase_order` (inbound X12 850) and `sales_order` (the implementer's internal commitment record) as distinct. Fulcrum collapses them: the Sales Order carries a customer-PO-number field. Cleanly separating them would require a second entity Fulcrum does not expose.
- **Serial number overloads Inventory Lot.** The ontology has a distinct `serial_number` entity. Fulcrum represents a serial as a single-quantity Inventory Lot. Serial-specific attributes (first-operation timestamp, customer-assigned SN, scrap-replacement chain) therefore ride on the Lot entity.
- **Item revision is a separate item, not a revision record.** The ontology's `part` has revisions as a supersession chain. In Fulcrum, Rev B and Rev C are distinct Item records linked through a revision reference. A canonical-first implementation would store revisions as records in a revision table attached to a part identity, not as sibling items.
- **Sub-tier PO is the same entity as customer-facing PO.** The ontology distinguishes `purchase_order` (generic), `customer_purchase_order` (inbound), and `sub_tier_purchase_order` (outbound to outside processor). Fulcrum uses one Purchase Order entity for outbound; inbound does not exist as a PO at all (see first bullet).
- **MRB disposition is fields on the NCR, not an event entity.** The ontology's `mrb_disposition` is a first-class event with attendees, minutes, and customer-deviation references. In Fulcrum, MRB status and disposition are text fields on the NCR.
- **Quality clauses are strings, not a catalog.** The ontology defines `quality_clause` as a structured catalog entry (category enum, clause body, flowdown behavior). Fulcrum has no catalog; clause codes ride as JSONB string arrays on POs and SOs. Two shops using different Fulcrum tenants cannot share a clause catalog.
- **Outside processing is three records, not one hybrid.** The ontology's `outside_process_operation` is a hybrid entity (operation + PO + cert-collection event). Fulcrum splits this into the Job Operation (flagged outside), the Purchase Order Outside Processing Line Item, and the Receipt + attached process certifications. Functional, but the hybrid entity's cert-collection semantics are not first-class.

## Status

- **Mapped against ontology version:** 2026-04-22 coherence pass (Task 11)
- **Coverage snapshot:** Roughly 60 of the ~80 canonical entities have either a native Fulcrum entity or a documented JSONB custom-field pattern. The remainder (FAI forms, key characteristics, characteristic measurements, process specifications, quality clauses, flowdown plans, AVL records, Nadcap scopes, export classifications, calibration records) rely on PDFs or external systems.
- **Source references:** `../../fulcrumpro-mcp/docs/fulcrum-api-reference.md` (API surface), `../extensions/uuid-discipline.md` §"UUID Storage in Fulcrum Pro" (JSONB UUID discipline), `../extensions/outside-processing.md` (outside-processing hybrid), `../extensions/material-certs.md` and `../schemas/material.yaml` (material lot and certification structure), `../reference/entity-inventory.yaml` (canonical entity list).
- **Last refreshed:** 2026-04-22
