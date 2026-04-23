<!-- Moved from reference/carbon-os-mapping.md on 2026-04-22 as part of the coherence pass.
     Carbon is a reference implementation, not the canonical ontology.
     See ../docs/superpowers/specs/2026-04-22-ontology-comprehensive-pass-design.md §2.2 and §8.
     Also see ../reference/carbon-novelty-2026-04.md for post-snapshot novelty findings. -->

# Carbon OS — Reference Implementation Mapping

Cross-reference between this ontology and Carbon OS (https://github.com/crbnos/carbon) data model. Based on analysis of Carbon's migration files. Carbon is one reference implementation among several; the canonical ontology lives in `../reference/`, `../extensions/`, and `../schemas/`.

## Related Work: Structured Raw Material Schema (Barbin, 2024)

Brad Barbin (Carbon OS maintainer) produced a [PostgreSQL schema for structured raw material management](https://gist.github.com/barbinbrad/b47f1100065e3650b5ecbc2aca50bc7a) in a general manufacturing ERP context. It decomposes a material into six normalized lookup tables: substance (aluminum, steel, titanium...), grade (7075, 4140, 17-4...), form (round bar, plate, sheet...), type (hot rolled, seamless, extruded...), finish, and dimension -- the same composable model this ontology independently arrives at from the compliance-specification direction. The convergence is informative: Brad's schema addresses the catalog and inventory lookup problem well; this ontology adds specification governance (AMS/ASTM/MIL as first-class entities), lot traceability with heat number and certification chains, and the DFARS compliance layer that aerospace procurement requires.

---

## Architecture Overview

**Carbon OS stack**: PostgreSQL (via Supabase), TypeScript, Row-Level Security, multi-tenant (company-scoped)

**Key design patterns**:
1. **Unified Item Master** -- single `item` table with `itemType` enum, extended by domain-specific tables
2. **Three-Layer BOM ("Methods")** -- `makeMethod` -> `methodOperation` -> `methodMaterial`, with recursive sub-assembly support
3. **Triple Mirroring** -- Method (design-time) / Quote (estimating) / Job (production) all have parallel structures
4. **JSONB Custom Fields** -- every major entity supports extensible custom fields
5. **Revision Control** -- composite uniqueness on (readableId, revision, companyId, type)

---

## Domain Mapping

### Materials

| Ontology Concept | Carbon OS Entity | Carbon OS Fields | Gap? |
|-----------------|-----------------|-----------------|------|
| Material (as item) | `item` + `material` | `item.type = 'Material'`, `material.materialForm`, `material.materialSubstance`, `material.grade`, `material.dimensions`, `material.finish` | |
| Material Form | `materialForm` (enum/lookup) | Angle, Channel, Beam, Bar, Plate, Sheet, Tube, Wire, etc. | |
| Material Substance | `materialSubstance` (enum/lookup) | Aluminum, Steel, Stainless Steel, Titanium, Copper, Brass | Limited -- no UNS, no spec-level detail |
| Material Specification | -- | -- | **GAP** -- no first-class spec entity |
| Material Lot | `batchNumber` | Lot/batch tracking with manufacturing dates, JSONB properties | DFARS/compliance via custom fields |
| Heat Treatment Spec | -- | -- | **GAP** -- referenced as text only |
| UNS Number | -- | -- | **GAP** -- no cross-reference system |
| AMS/ASTM cross-ref | -- | -- | **GAP** |
| Condition codes | `material.grade` (partially) | Grade field stores condition info | Minimal structure |

**Assessment**: Carbon models materials as inventory items with basic form/substance classification. The specification-level detail (AMS numbers, types, classes, conditions, chemistry limits, cross-references) that aerospace shops need is not modeled. This is the biggest domain gap.

### Items & BOMs

| Ontology Concept | Carbon OS Entity | Carbon OS Fields | Gap? |
|-----------------|-----------------|-----------------|------|
| Item (master) | `item` | readableId, revision, name, type, trackingType, replenishmentSystem, unitOfMeasureCode | |
| Item Types | `itemType` enum | Part, Material, Tool, Service, Consumable, Fixture | Missing: Phantom, Outside Process, Raw Material (as distinct type) |
| Part extension | `part` | approved, approvedBy, customFields | |
| Revision tracking | `item` | readableId + revision composite, readableIdWithRevision generated | |
| CAGE Code | -- | -- | **GAP** |
| Export Control | -- | -- | **GAP** |
| BOM | `makeMethod` | itemId, version | |
| BOM Line | `methodMaterial` | itemId, quantity, unitOfMeasureCode, unitCost, kit flag | |
| Recursive BOM | `methodMaterial.materialMakeMethodId` | FK to another makeMethod for sub-assemblies | Nice recursive CTE support |
| Routing | `makeMethod` (unified with BOM) | | Good pattern: BOM + routing unified |
| Operation | `methodOperation` | processId, workCenterId, setupTime, laborTime, machineTime, outsourced | |

**Assessment**: Strong BOM/routing model. The unified "Methods" approach with recursive sub-assemblies is a good pattern. Missing aerospace-specific fields (CAGE, export control, drawing references as structured data).

### Manufacturing Execution

| Ontology Concept | Carbon OS Entity | Carbon OS Fields | Gap? |
|-----------------|-----------------|-----------------|------|
| Process | `process` | name (simple) | Minimal -- no special process classification |
| Work Center | `workCenter` | name, locationId, abilityId, rates, active | |
| Work Center capabilities | `workCenterProcess` | Junction table linking WC to processes | |
| Job | `job` | jobId, itemId, status (7 states), quantity, scrapQuantity, deadlineType, dueDate | |
| Job Operation | `jobOperation` | All routing fields + targetQuantity, startDate, dueDate | |
| Job Material | `jobMaterial` | itemId, quantity, methodType (Buy/Make/Pick) | |
| Operation Dependencies | `operationDependency` | DAG-style sequencing | Nice -- goes beyond simple linear |
| Work Instructions | `procedure`, `procedureStep` | Versioned, typed steps (Value, Measurement, Checkbox, etc.) | |
| Actual data capture | `jobOperationAttributeRecord` | Captures production data per step | |
| Time factor flexibility | `factor` enum | Hours/Piece, Minutes/Piece, Pieces/Hour, etc. | Very flexible |

**Assessment**: Solid manufacturing execution model. The triple-mirroring (method->quote->job) preserves planned vs. estimated vs. actual across contexts. Operation dependencies support complex sequencing. Missing Nadcap classification and process specification tracking.

### Quality

| Ontology Concept | Carbon OS Entity | Carbon OS Fields | Gap? |
|-----------------|-----------------|-----------------|------|
| NCR | `nonConformance` | Status, quantity, assignment, tags, customFields | |
| NCR Types | `nonConformanceType` | 10 pre-seeded types | |
| NCR Workflow | `nonConformanceWorkflow` | Templates with priority, source, required actions, approvals | |
| NCR Tasks | `nonConformanceInvestigationTask`, `nonConformanceActionTask`, `nonConformanceApprovalTask` | Full task management | |
| NCR Linkage | Junction tables | Links to suppliers, customers, jobs, POs, SOs, receipts, shipments | Comprehensive |
| FAI (AS9102) | -- | -- | **GAP** -- no FAI model |
| Inspection Record | `jobOperationAttributeRecord` | Embedded in operation workflow | No standalone inspection entity |
| Gauge Management | `gauge`, `gaugeType`, `gaugeCalibrationRecord` | 20+ gauge types, serial tracking, cal intervals, pass/fail records | Strong |
| Key Characteristics | -- | -- | **GAP** |
| SPC / Statistical | -- | -- | **GAP** |

**Assessment**: Strong NCR and calibration management. Significant gaps in FAI (critical for aerospace), standalone inspection records, key characteristic tracking, and SPC capabilities.

### Supply Chain

| Ontology Concept | Carbon OS Entity | Carbon OS Fields | Gap? |
|-----------------|-----------------|-----------------|------|
| Supplier | `supplier` | name, type, status, customFields | |
| Supplier Contacts | `supplierContact` | Junction table | |
| Supplier Certifications | -- | -- | **GAP** -- no structured cert tracking |
| OEM Approvals | -- | -- | **GAP** |
| Nadcap scope tracking | -- | -- | **GAP** |
| Supplier-Item link | `supplierPart` (fka `buyMethod`) | Pricing, lead time per supplier-item combo | |
| Purchase Order | `purchaseOrder` | Full lifecycle, multi-currency | |
| PO Lines | `purchaseOrderLine` | All item types, UOM conversion | |
| Quality Clauses | -- | -- | **GAP** -- no structured clause system |
| DFARS tracking | -- | -- | **GAP** |
| Receipt | `receipt`, `receiptLine` | Source document, status, quantities | |
| Outside Processing | `methodOperation.outsourced` flag | Boolean on operations | Minimal -- no integrated PO generation |

**Assessment**: Basic supplier management and purchasing. Missing the aerospace-specific overlays: certifications, OEM approvals, quality clause flowdowns, DFARS compliance tracking, and robust outside processing integration.

### Sales & Quoting

| Ontology Concept | Carbon OS Entity | Carbon OS Fields | Gap? |
|-----------------|-----------------|-----------------|------|
| Customer | `customer` | name, type, status, customFields | |
| Sales Order | `salesOrder` | Full lifecycle with revision tracking, multi-currency | |
| SO Lines | `salesOrderLine` | All item types, requiresInspection flag | |
| Quote | `quote` | Full quote system with line-level estimating | |
| Quote Methods | `quoteMakeMethod`, `quoteOperation`, `quoteMaterial` | Mirrors BOM/routing for estimating | Strong |
| Opportunity | `opportunity` | Pipeline connecting RFQ -> Quote -> SO | |
| Contract/Program | -- | -- | **GAP** -- no contract entity |
| Export Control | -- | -- | **GAP** |
| Quality flowdowns | -- | -- | **GAP** |

### Inventory

| Ontology Concept | Carbon OS Entity | Carbon OS Fields | Gap? |
|-----------------|-----------------|-----------------|------|
| Location | `location` (fka `warehouse`) | Physical sites | |
| Sub-location | `shelf` | Divisions within location | |
| Item Ledger | `itemLedger` | Every movement with entry/document types, cost | |
| Quantities view | `itemQuantities` | On-hand, on-PO, on-SO, on-job, available | |
| Serial Tracking | `serialNumber` | Status, source, expiration | |
| Batch Tracking | `batchNumber` | Properties, dates | |
| Unified Tracking | `itemTracking` | Cross-document serial/batch tracking | Good design |
| Transfers | `warehouseTransfer`, `warehouseTransferLine` | 6-status lifecycle, shelf-level | |
| MRP/Planning | `demandForecast`, `supplyForecast`, etc. | Period-based planning with confidence | |
| Quarantine | -- | -- | **GAP** -- no quarantine location designation |

---

## Patterns Worth Adopting from Carbon OS

1. **Unified Item Master with type extensions** -- single source of truth for all trackable things
2. **Integrated BOM + Routing ("Methods")** -- materials know which operation consumes them
3. **Triple mirroring (Method -> Quote -> Job)** -- preserves context across lifecycle
4. **Recursive BOM traversal** -- `WITH RECURSIVE` CTE for multi-level explosion
5. **Flexible time factors** -- 9 different time calculation modes for operations
6. **Operation dependencies** -- DAG-style, not just linear sequencing
7. **JSONB custom fields + metadata registry** -- extensible without schema changes
8. **Unified item tracking** -- single table for serial/batch across all document types

## Key Gaps to Fill for Aerospace

1. **Material specification as first-class entity** -- AMS/ASTM/MIL specs with types, conditions, cross-references
2. **Process specification entity** -- H/T, plating, NDT specs with Nadcap categorization
3. **First Article Inspection (AS9102)** -- 3-form structure, trigger tracking, status per part/revision
4. **Export control classification** -- ITAR/EAR marking, access control, audit trail
5. **Supplier certification tracking** -- ISO, AS9100, Nadcap scopes, OEM approvals
6. **Quality clause flowdown system** -- Structured clause codes on POs and SOs
7. **Outside processing integration** -- Operations that generate POs and track round-trip
8. **Key characteristic / SPC tracking** -- Statistical process control for critical dimensions
9. **Contract/program entity** -- Long-term agreements, pricing, flowdowns
10. **Data retention/governance rules** -- Per-record retention classification
