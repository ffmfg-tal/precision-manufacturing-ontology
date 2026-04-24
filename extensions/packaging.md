# Packaging Extension

## The Gap This Fills

AS9100 §8.5.4 requires preservation of conformity during internal processing and delivery, including packaging. MIL-STD-2073 defines preservation and packaging levels (A, B, C) for military hardware — specifying moisture barrier materials, desiccant quantities, humidity indicator cards, and outer container requirements. Neither standard provides a data schema for packaging as a traceable operation.

The existing standards stack leaves three specific gaps:

- **MIL-STD-2073** defines the physical requirements for each preservation level but is a requirements document, not a data model. It has no machine-readable representation of what a compliant package must contain.
- **AS9100 §8.5.4** mandates preservation of conformity and requires documented procedures, but specifies no record structure. The audit evidence is a procedure document, not a per-part execution record.
- **`schemas/shipment.yaml`** tracks logistics: carrier, tracking number, ship date, serialized units in transit. It does not capture what packaging materials were used, who applied them, whether ESD protection was installed, or whether a quality inspector verified the package before sealing.
- **No Layer-1 standard** (STEP, QIF, MTConnect, X12 EDI) defines packaging as a traceable manufacturing operation with a linkage from part serial number → packaging specification → packaging execution evidence → shipment.

The result: the digital thread terminates at shipment dispatch. There is no structured record connecting the part's traceability chain to the specific bag, desiccant unit, humidity indicator card, and box used to protect it. For aerospace customers who require packaging compliance evidence (and for DCMA audits of MIL-STD-2073 compliance), this gap means the cert package cannot include packaging execution evidence in machine-readable form.

This extension defines three entities that close the gap:

1. **`packaging_specification`** — the governing document defining HOW a part or assembly must be packaged: preservation level, moisture barrier, ESD, desiccant, labeling standard, and optional manufactured packaging insert.
2. **`packing_record`** — the execution instance: evidence that a specific serial number or lot was packaged according to its specification, with packed-by and inspected-by traceability. Immutable after inspector sign-off.
3. **`packaging_item`** — a single packaging component line within a packing record: the specific bag, desiccant unit, humidity indicator, ESD bag, box, label, or custom insert used, with lot/batch traceability of the packaging material itself.

---

## Entity Definitions

### Packaging Specification

The governing document for how a part or assembly must be packaged. Defines the required preservation level (MIL-STD-2073 Level A, B, C, or commercial), moisture barrier requirements, ESD protection, desiccant, humidity indicator, quantity limits, and labeling standard. One specification per part+revision or assembly+revision; updated by issuing a new revision of the spec.

The subject is polymorphic: a packaging specification may govern a part or an assembly, discriminated by `subject_type` per Decision 1.9 (the same pattern used by `first_article_inspection` and `nonconformance_report`).

When the packaging design requires a custom formed insert — a 3D-printed or machined tray that protects the part geometry — that insert is itself a part entity with `part_kind: packaging_insert` (see Decision 2.1). The specification references it via `packaging_insert_part_id`.

```yaml
packaging_specification:
  id: uuid                          # UUID v4 — minted when spec is created
  subject_id: uuid                  # FK → part.id or assembly.id (discriminated by subject_type)
  subject_type: enum                # part | assembly
  revision: string                  # Spec revision; advance independently of part revision
  preservation_level: enum          # A | B | C | None
    # A = MIL-STD-2073 Level A: heat-sealed poly bag, desiccant (MIL-D-3464), humidity
    #     indicator card (HIC), sealed moisture barrier bag, outer box
    # B = Level B: moderate storage protection, less stringent than Level A
    # C = Level C: spot-pack, domestic short-duration storage or transit
    # None = commercial packaging per customer-specified instruction
  moisture_barrier_required: boolean
  esd_protection_required: boolean  # ESD shielding bag required (static-sensitive parts)
  desiccant_required: boolean       # Desiccant unit required (type per MIL-D-3464)
  humidity_indicator_required: boolean  # HIC required inside moisture barrier
  max_pieces_per_bag: integer       # optional — maximum quantity per inner bag
  max_pieces_per_box: integer       # optional — maximum quantity per shipping box
  labeling_standard: string         # optional — e.g., "MIL-STD-129", "MIL-STD-130", "customer spec"
  special_handling_notes: string    # optional — e.g., "Do not stack", "Keep dry", "Fragile fins"
  customer_packaging_spec_ref: string  # optional — customer-supplied packaging spec document reference
  packaging_insert_part_id: uuid    # optional — FK → part.id where part_kind = packaging_insert
  approved_by_id: uuid              # optional — FK → user.id (quality engineer who approved this spec)
  approved_date: date               # optional
  notes: string                     # optional
```

---

### Packing Record

The execution instance: evidence that a specific serialized unit or lot was packaged in conformance with its packaging specification. Created when packaging begins; finalized (and made immutable) when an inspector signs off. Links to the outbound shipment it supports.

One packing record per serial number or lot per shipment. A shipment of five serialized parts produces five packing records (one per serial) unless the specification allows multiple pieces per bag and a single bag is confirmed.

```yaml
packing_record:
  id: uuid                          # UUID v4 — minted when packaging begins
  shipment_id: uuid                 # FK → shipment.id (the outbound shipment this packaging supports)
  packaging_specification_id: uuid  # FK → packaging_specification.id (governing spec)
  subject_id: uuid                  # FK → part.id or assembly.id
  subject_type: enum                # part | assembly — matches packaging_specification.subject_type
  serial_id: uuid                   # optional — FK → serial_number.id (if serialized)
  lot_id: uuid                      # optional — FK → material_lot.id (if lot-controlled)
  quantity_packed: integer          # Number of pieces covered by this record
  packed_by_id: uuid                # FK → user.id (operator who performed packaging)
  packed_date: date                 # Date packaging was performed
  inspected_by_id: uuid             # optional — FK → user.id (quality inspector who verified)
  inspection_date: date             # optional — date of quality inspector sign-off
  result: enum                      # Pass | Fail | Rework
  notes: string                     # optional
```

**Immutability:** Once `inspected_by_id` and `inspection_date` are populated and `result` is `Pass`, the packing record is immutable. Any subsequent packaging nonconformance that requires rework generates a new packing record linked to the same shipment, with `result: Rework` on the original and `result: Pass` on the replacement.

---

### Packaging Item

A single packaging component used in a packing record — one line per distinct component type. Provides material-level traceability: which specific bag, desiccant unit, humidity indicator card, ESD bag, box, label, or manufactured insert was consumed, including lot or batch number of the packaging material.

When `item_type` is `CustomInsert`, the `part_id` field links to the manufactured packaging insert part entity (part_kind: packaging_insert), providing full part-number traceability for the insert.

```yaml
packaging_item:
  id: uuid                          # UUID v4
  packing_record_id: uuid           # FK → packing_record.id
  item_type: enum                   # Bag | Box | DesiccantUnit | HumidityIndicator |
                                    # VCIBag | ESDShielding | Label | Foam | CustomInsert | Other
  part_id: uuid                     # optional — FK → part.id
                                    # populated when item_type = CustomInsert or other manufactured packaging component
  description: string               # e.g., "4×8 heat-sealed poly bag", "2-unit desiccant per MIL-D-3464",
                                    # "humidity indicator card, 3-spot, 10/20/30%", "24×18×6 cardboard outer box"
  quantity: integer                 # Number of this item used
  lot_or_batch: string              # optional — lot/batch number of the packaging material
  notes: string                     # optional
```

**MIL-STD-2073 Level A Packaging Item Composition (reference):**

| Item Type | Description | Spec Reference |
|-----------|-------------|----------------|
| Bag | Heat-sealed poly inner bag | MIL-PRF-22191 or MIL-DTL-117 |
| DesiccantUnit | Desiccant unit(s) per humidity level | MIL-D-3464 |
| HumidityIndicator | Humidity indicator card (3-spot or 5-spot) | MIL-I-8835 |
| Bag | Heat-sealed moisture barrier outer bag | MIL-PRF-22191 Type I |
| Box | Outer shipping container | ASTM D4169 or customer spec |
| Label | MIL-STD-129 shipping label | MIL-STD-129 |

---

## MIL-STD-2073 Preservation Level Reference

| Level | Preservation Requirements | Typical Application |
|-------|---------------------------|---------------------|
| A | Heat-sealed poly bag, desiccant (MIL-D-3464), humidity indicator card, sealed moisture barrier bag (MIL-PRF-22191), outer box. Designed for extended storage and shipment through severe environmental exposure. | Long-term storage, overseas shipment, DoD supply depots |
| B | Moderate storage protection. Inner wrap or bag, no desiccant required unless specified, outer container. | Medium-term domestic storage, standard DoD procurement |
| C | Spot-pack. Commercial-grade packaging sufficient for short-duration domestic transit. Minimum physical protection only. | Short-haul domestic, non-critical commercial delivery |
| None | Commercial packaging per customer instruction. No MIL-STD-2073 requirements apply. | Commercial aerospace customers with own packaging specs |

---

## Integration with the Digital Thread

| Packaging Data | From | Linked To |
|----------------|------|-----------|
| packaging_specification.id | Quality-authored spec | Part/assembly master record |
| packing_record.shipment_id | Outbound shipment | X12 856 ASN, cert package |
| packing_record.serial_id | Serialized part traceability | Job → serial → packing record chain |
| packaging_item.part_id | Manufactured insert | Part master, drawing, routing |
| packaging_item.lot_or_batch | Packaging material lot | Supplier traceability for packaging materials |
| packing_record.inspected_by_id | Quality inspector sign-off | AS9100 §8.5.4 conformance evidence |

**Digital Thread Sequence:**

```
PART COMPLETION
  | Part passes final CMM inspection
  | packing_record minted for serial S/N 001
  | packing_record.packaging_specification_id → spec for this P/N
  v
PACKAGING EXECUTION
  | Operator selects per-spec materials:
  |   packaging_item: Bag (lot 24-B-0042), qty 1
  |   packaging_item: DesiccantUnit (lot MDD-2024-08), qty 2
  |   packaging_item: HumidityIndicator (lot HIC-2024-11), qty 1
  |   packaging_item: Bag (MBB, lot PRF-24-0091), qty 1
  |   packaging_item: Box (24×18×6 outer), qty 1
  | packed_by_id + packed_date recorded
  v
QUALITY INSPECTION
  | Inspector verifies materials, desiccant placement, HIC, seal integrity
  | inspected_by_id + inspection_date populated
  | result → Pass
  | packing_record becomes immutable
  v
SHIPMENT DISPATCH
  | packing_record.shipment_id → shipment.id
  | Shipment status → In_Transit
  | X12 856 ASN generated from shipment record
  v
CERT PACKAGE ASSEMBLY
  | packing_record UUID included as packaging conformance evidence
  | AS9100 §8.5.4 traceability: serial → spec → record → inspector
```

---

## Decision Log

**Decision 2.1 — Packaging inserts are parts, not a separate entity type.**
A 3D-printed or machined packaging insert (a formed tray that cradles a part in the box) has a part number, a drawing, a material, and potentially its own routing. It is manufactured by the same processes as production parts. Treating it as a distinct entity would fragment the digital thread: the insert would lose access to part-number search, FAI status, routing, and revision control that part entities already carry. The insert is therefore a `part` entity with `part_kind: packaging_insert`. This enum value is a companion change required in `schemas/part.yaml` — it is not currently present in the `[machined, fastener, raw_stock, purchased_assembly, consumable]` enum. The `packaging_specification.packaging_insert_part_id` FK references `part.id` and relies on `part_kind = packaging_insert` as a runtime constraint, not a schema-enforced discriminator.

**Decision 2.2 — Subject polymorphism via `subject_type` discriminator (follows Decision 1.9).**
Both `packaging_specification` and `packing_record` carry `subject_id` + `subject_type: enum [part, assembly]`. This is the same pattern established in Decision 1.9 and used by `first_article_inspection`, `nonconformance_report`, and similar quality entities. A shared supertype was rejected (Decision 1.1); a separate entity per subject type was rejected as redundant. The discriminator pattern is the established idiom for the typed union throughout this ontology.

**Decision 2.3 — `packing_record` is immutable after inspector sign-off; rework produces a new record.**
A packing record after quality inspection sign-off is evidence of conformance. Mutating it post-sign-off would invalidate the AS9100 §8.5.4 audit trail. Rework (repackage after failed inspection or customer return) creates a new packing record with `result: Rework` on the original and `result: Pass` on the replacement. Both records are retained and linked to the same shipment, providing a complete history of packaging attempts.

**Decision 2.4 — `packaging_item` is a first-class entity (line entity), not a nested array.**
The alternative was to embed packaging items as a YAML array on `packing_record`. A first-class entity was chosen because each item needs independent lot/batch traceability (the desiccant lot and the barrier bag lot are different records), an optional FK to a manufactured insert part, and queryability ("find all shipments that used bag lot 24-B-0042"). A nested array makes none of those possible without extracting the entity anyway.

**Decision 2.5 — `packing_record` links to `shipment.id`, not to `job.id` directly.**
Packaging happens at the point of shipment, not at job completion. A job may produce a batch that ships in multiple shipments (split delivery, partial acceptance). Linking packing records to the shipment — which is already linked to the job via `shipment.job_id` — preserves the full chain without adding a redundant FK. The job traceability is available transitively: packing_record → shipment → job.

**Decision 2.6 — Preservation level `None` is an explicit enum value, not a null.**
Commercial customers may specify their own packaging requirements that are not MIL-STD-2073 levels. Using `None` as an explicit enum value (rather than nullable `preservation_level`) makes the intent explicit in the data: this spec is not a MIL-STD-2073 spec, it is a commercial spec governed by `customer_packaging_spec_ref`. A null value would be ambiguous between "commercial" and "not yet specified."
