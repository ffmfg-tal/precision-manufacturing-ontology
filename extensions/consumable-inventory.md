# Consumable Inventory Extension

## The Gap This Fills

A precision machine shop runs on more than raw stock and finished parts. Between the bar of 15-5 PH and the shipped assembly sit thousands of unit-tracked items that the standards stack does not model: the carbide endmill waiting in the tool crib drawer, the resin-bond grinding wheel hanging at the deburr bench, the 3/8-inch hex socket that lives with the torque wrench, the box of nitrile gloves at the inspection station, the cotton swabs by the cleaning sink, the AlNiCo magnets used to fixture small parts on the surface grinder.

These items are real inventory. They have part numbers. They have suppliers, on-hand counts, reorder thresholds, and physical locations. They have lifecycles — but those lifecycles are heterogeneous in a way that breaks the existing material model:

- A **glove** is issued and destroyed in a single use. Stock decrements by one when a pair is taken from the dispenser.
- A **carbide cutter** is consumed by machining but may be checked back in for re-sharpening, re-coating, or inspection, then re-issued as serviceable stock. Stock decrements on issue and may increment again on return.
- A **hex socket** is checked out to a job with the expectation that it returns intact at the end. Stock decrements on issue and increments on return; loss is the exception, not the rule.

None of these patterns fit `schemas/material.yaml`. `material` and `material_lot` model production raw stock — a heat-traceable bar of titanium, a lot-controlled drum of cutting fluid, a casting with an MTR. They carry alloy chemistry, DFARS compliance, melt country, PMI verification. That structure is correct for raw material and wrong for a hex socket.

The existing standards leave this gap unfilled:

- **STEP AP242** has no inventory concept at all.
- **MTConnect** describes assets that report to a machine controller, not the stockroom behind it.
- **QIF** is a measurement-data standard.
- **X12 EDI** carries purchase orders but no consumption model.
- **ISA-95 Part 2** defines storage zones and lot control — close to `inventory_location` — but does not address consumable lifecycle modes (consumed vs. recoverable vs. returnable) or the cross-departmental tracking pattern that cutters, gloves, and sockets share despite being conceptually different.

The result before this entity: `cutter_definition` carried a comment that physical cutter inventory "links to cutter_definition via the item master, not via a separate instance table" — but there was no item-master entity in the ontology to point to. Stock counts of grinding wheels and gloves had no home at all.

This extension defines a single entity, `consumable_inventory`, as the unit-tracked stock register for non-serialized tooling and shop-floor consumables. It is deliberately distinct from `material` and is the join target the cutter-stock-tracking gap was waiting for.

---

## Entity Definition

### consumable_inventory

A unit-tracked stock register. One row per stock-tracked unit per location. Carries description, kind, on-hand quantity, reorder thresholds, unit of issue, and a `consumption_mode` enum that disambiguates the lifecycle. When the unit is a cutter or insert, `cutter_definition_id` links to the catalog spec in `schemas/tool_room.yaml`; for everything else, identity is carried by `description` + `manufacturer` + `manufacturer_part_number`.

```yaml
consumable_inventory:
  id: uuid                          # UUID v4 per uuid-discipline.md
  description: string               # Required. Human-readable description.
                                    #   e.g., "0.500 4-flute carbide bull endmill, 0.030 corner radius"
                                    #         "6 inch resin-bond grinding wheel, 80 grit"
                                    #         "3/8 inch hex socket, 1/2 inch drive"
                                    #         "Nitrile glove, large, powder-free"
  consumable_kind: enum             # Required. Cutter | Insert | GrindingWheel | Socket | Bit |
                                    # Glove | Brush | Swab | Lubricant | CleaningSupply |
                                    # Hardware | Other
  cutter_definition_id: uuid        # Optional. FK → cutter_definition.id (schemas/tool_room.yaml).
                                    # Populated when consumable_kind ∈ {Cutter, Insert}.
  manufacturer: string              # Optional. e.g., "Kennametal", "Norton", "Snap-on", "Ansell"
  manufacturer_part_number: string  # Optional. Manufacturer catalog or SKU number.
  unit_of_issue: enum               # Required. Each | Box | Pair | Roll | Gallon | Pound |
                                    # Ounce | Foot | Meter
  inventory_location_id: uuid       # Optional. FK → inventory_location.id (schemas/inventory_location.yaml).
  quantity_on_hand: number          # Required. Current stock in unit_of_issue.
  min_quantity: number              # Optional. Reorder threshold; drives low-stock surface.
  max_quantity: number              # Optional. Reorder target ceiling.
  consumption_mode: enum            # Required. ConsumedDiscarded | ConsumedRecoverable | IssuedReturnable
  notes: string                     # Optional.
```

**Relationships:**

- `for_cutter` → `cutter_definition` (many-to-one). Populated when `consumable_kind ∈ {Cutter, Insert}`; null otherwise.
- `stored_at` → `inventory_location` (many-to-one). Where this stock physically lives.

---

## Distinction From `material`

| Concern | `material` / `material_lot` | `consumable_inventory` |
|---|---|---|
| What it models | Production raw stock (bar, plate, billet, casting) | Tooling and shop-floor consumables (cutters, gloves, sockets, wheels) |
| Consumed by | A job; usually 1:1 against a routing's material requirement | Many possible patterns; see `consumption_mode` below |
| Traceability requirement | Full: heat number, mill source, melt country, DFARS, MTR chain | Counted, not heat-traceable |
| Compliance fields | DFARS, PMI verification, conflict-mineral status, ITAR controlled flag | None — consumables do not carry compliance metadata |
| Cert package | Yes — `certification_document` chain | No |
| Lifecycle | State machine (Available → Reserved → In_Use → Consumed / Rejected / Quarantined) | None — register only; `consumption_mode` carries lifecycle semantics inline |
| Lot identity | `material_lot.heat_number` is the primary key for traceability | Not lot-tracked at this granularity |

The two models exist for different reasons and should not be merged. A titanium billet for a flight-critical part needs heat-number traceability or the cert package fails an audit. A pair of nitrile gloves does not. Overloading `material` to carry both would force one of two bad outcomes: either gloves and swabs acquire phantom DFARS and PMI fields they will never legitimately populate, or the cert-package generation logic gets harder to reason about because it must filter out non-traceable lots.

---

## Consumption Modes

The `consumption_mode` enum is the lifecycle discriminator. Three modes:

### ConsumedDiscarded

The unit is destroyed in use. Stock decrements on issue and never increments back. Workflows:

- Glove dispensed → `quantity_on_hand -= 1`
- Swab used to clean a part feature → `quantity_on_hand -= 1`
- Single-use brush thrown out → `quantity_on_hand -= 1`

Reorder logic: when `quantity_on_hand <= min_quantity`, the unit is flagged for purchasing. Most shop-floor PPE and cleaning supplies fall here.

### ConsumedRecoverable

The unit is consumed by production but may re-enter stock after re-sharpening, inspection, or refurbishment. The default workflow decrements on issue; a separate event may increment quantity back when the unit returns from a refurb supplier or in-house sharpening. Workflows:

- Carbide endmill issued to a job → `quantity_on_hand -= 1`
- Endmill removed at end of run, sent to outside sharpening → out of stock until return
- Sharpening vendor returns the cutter → `quantity_on_hand += 1` (typically tracked against a separate "re-sharpened" sub-stock if quality differs from new)
- Grinding wheel dressed in-house and re-mounted → no decrement, since the wheel never left the deburr bench; the wheel's remaining life is tracked by the operator informally

`ConsumedRecoverable` covers most non-disposable cutting tools and abrasives. The recovery path is workflow-specific, not modeled as a separate entity in this first cut. A future enhancement may add a `consumable_recovery_event` if the auditing need surfaces.

### IssuedReturnable

The unit is checked out and expected to return at end of job. Stock decrements on issue and increments on return. Loss is the exception. Workflows:

- 3/8" hex socket pulled from torque-wrench accessory tray for an assembly job → `quantity_on_hand -= 1`
- Job complete, socket returned to tray → `quantity_on_hand += 1`
- Socket lost or damaged → status note added; eventual disposition is `ConsumedDiscarded`-like (decrement, no return) with an investigation trail in `notes`

`IssuedReturnable` covers hand tools, calibrated torque accessories, and shared specialty equipment that is not severe enough to deserve a serialized `part` record (those go in `tool_room_allocation` against `part.id` with `part_kind: fixture`).

---

## Worked Examples

### Cutter as consumable

A 0.500" carbide bull endmill has a `cutter_definition` row (catalog spec: manufacturer, geometry, coating). The tool crib carries an opening stock of twelve. Each is a stock-tracked unit. The crib registers a single `consumable_inventory` row:

```yaml
- id: <uuid>
  description: "0.500 4-flute carbide bull endmill, 0.030 corner radius (Kennametal)"
  consumable_kind: Cutter
  cutter_definition_id: <kennametal-0500-endmill-uuid>
  manufacturer: Kennametal
  manufacturer_part_number: 12345-A1
  unit_of_issue: Each
  inventory_location_id: <tool-crib-drawer-3-uuid>
  quantity_on_hand: 12
  min_quantity: 4
  max_quantity: 20
  consumption_mode: ConsumedRecoverable
  notes: >
    Re-sharpenable up to three times; rebrand to "re-sharpened" stock after first
    sharpening if customer drawings require new-cutter cuts.
```

When a machinist pulls one for a job, `quantity_on_hand` drops to 11. The cutter is also tracked at the per-run level by `tool_life_record` (parts_run, tool_change_required) — that record is independent. When `tool_life_record.tool_change_required = true`, the operator either discards the cutter (decrement only) or routes it to the sharpening vendor (track separately; a return decrement-then-increment cycle).

### Grinding wheel as consumable

A 6-inch resin-bond grinding wheel at the deburr bench has no `cutter_definition` — grinding wheels are not currently in the tool-room catalog. The deburr bench tracks it directly:

```yaml
- id: <uuid>
  description: "6 inch resin-bond grinding wheel, 80 grit, for hand deburr"
  consumable_kind: GrindingWheel
  manufacturer: Norton
  manufacturer_part_number: 38A-80-K5-V
  unit_of_issue: Each
  inventory_location_id: <deburr-bench-cabinet-uuid>
  quantity_on_hand: 6
  min_quantity: 2
  max_quantity: 12
  consumption_mode: ConsumedRecoverable
  notes: >
    Dressed in place at the bench; replaced when remaining stock is below the
    flange marking. Used wheels are scrapped, not returned.
```

The wheel is `ConsumedRecoverable` because dressing extends life, but is functionally `ConsumedDiscarded` at end of life. The mode is chosen to match the dominant workflow.

### Glove as consumable

A box of large nitrile gloves at the inspection station:

```yaml
- id: <uuid>
  description: "Nitrile glove, large, powder-free"
  consumable_kind: Glove
  manufacturer: Ansell
  manufacturer_part_number: TouchNTuff-93-250
  unit_of_issue: Box
  inventory_location_id: <inspection-bench-dispenser-uuid>
  quantity_on_hand: 4
  min_quantity: 2
  max_quantity: 10
  consumption_mode: ConsumedDiscarded
  notes: >
    Each box contains 100 gloves. Reorder when 2 boxes remain.
```

`unit_of_issue: Box` keeps inventory transactions coarse-grained — the shop is not counting individual gloves. When a box runs out and someone opens the next, `quantity_on_hand -= 1`. Reorder fires at 2 boxes.

---

## Relationship to Other Tool-Room Entities

`consumable_inventory` complements, but does not replace, the existing tool-room entities:

| Concern | Entity | What it captures |
|---|---|---|
| Catalog spec for a cutter type | `cutter_definition` | Manufacturer, geometry, coating — abstract type |
| Per-job/work-order consumption record | `tool_life_record` | parts_run, tool_change_required for one cutter type on one run |
| Aggregate on-hand stock for the cutter type | `consumable_inventory` (new) | quantity_on_hand, min/max, location, mode |
| Serialized fixture/holder checkout | `tool_room_allocation` | Custody of a `part.id` with `part_kind: fixture` |
| Maintenance event on a fixture, holder, or gauge | `tool_maintenance_event` | Sharpen / Clean / Repair / Inspect / Recertify / Retire |

`tool_life_record` says how many parts a cutter type ran on a specific job. `consumable_inventory` says how many of that cutter type are in the crib right now. A future enhancement could close the loop — `tool_life_record.tool_change_required = true` could emit a stock decrement against the matching `consumable_inventory` row — but that linkage is workflow logic, not an ontology relationship, and is left out of this first cut.

The `Cutter` discriminator value on `tool_maintenance_event.asset_type` is still available for tracking maintenance events on individually identifiable cutter inventory where that level of detail is required (e.g., shop-internal sharpening tracking). For non-serialized cutters, `consumable_inventory` carries the aggregate counts and `tool_life_record` carries the per-run history.

---

## Layer-1 Gap Analysis

| Shop-floor inventory concern | STEP AP242 | MTConnect | QIF | X12 EDI | ISA-95 Part 2 | This Extension |
|---|---|---|---|---|---|---|
| Production raw material lot | No | No | No | Partial (PO line) | Yes (lot model) | covered by `material_lot` |
| Cutter stock counts | No | No | No | No | No | `consumable_inventory` |
| Insert stock counts | No | No | No | No | No | `consumable_inventory` |
| Grinding wheel stock | No | No | No | No | No | `consumable_inventory` |
| PPE (gloves, etc.) stock | No | No | No | No | No | `consumable_inventory` |
| Returnable hand tool tracking | No | No | No | No | No | `consumable_inventory` |
| Min/max reorder thresholds | No | No | No | No | Partial | `consumable_inventory` |
| Consumption-mode discriminator | No | No | No | No | No | `consumable_inventory.consumption_mode` |

The closest existing concept is the ISA-95 material lot, which models production stock under a single lifecycle. ISA-95 does not address the heterogeneous lifecycle of shop-floor consumables, the cross-departmental location pattern (a single SKU stocked at multiple benches), or the consumption-mode distinction that separates a glove from a hex socket from a re-sharpenable cutter. This extension fills that gap as a single first-class entity.

---

## Future Work

Out of scope for this first cut, but flagged for follow-on:

1. **Stock movement events.** A `consumable_inventory_event` log (issue, return, receive, adjust) would give an audit trail behind `quantity_on_hand`. Today the field is the running total; downstream systems may maintain their own event log.
2. **Tool-life-record → stock decrement link.** When `tool_life_record.tool_change_required = true` and the cutter is stock-tracked, the workflow should emit a stock decrement. This is a workflow rule, not an ontology relationship; a future schema pass may add an optional `consumable_inventory_id` FK on `tool_life_record` to anchor the rule.
3. **Sharpening-vendor recovery cycle.** `ConsumedRecoverable` cutters that go off-site for sharpening and return need a tracking pattern. Candidates: reuse `outside_process_operation` against the consumable, or add a dedicated `consumable_recovery_event`. Defer until the operational need surfaces.
4. **Multi-location sub-stock.** A single SKU stocked at three benches today requires three `consumable_inventory` rows (one per location). A future enhancement could introduce a `consumable_sku` parent entity that holds catalog identity and aggregates child rows per location, mirroring `cutter_definition → consumable_inventory` for non-cutter SKUs.
