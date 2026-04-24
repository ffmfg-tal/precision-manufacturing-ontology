# Tool Room Extension

## The Gap This Fills

The tool room is the physical and informational bridge between engineering (NC programs, process plans) and the machine floor (loaded pockets, running cutters). Industry standards leave most of this bridge undefined:

- **MTConnect CuttingTool asset schema** covers tool identity and asset ID on a connected machine — serial number, life counts, cutting item geometry. It does not model the catalog hierarchy that precedes a physical tool (cutter type × holder type = assembly spec), the presetter measurement record that produces the offsets, the checkout system that tracks who has a fixture, or the gauge catalog separate from the calibrated instrument.
- **STEP AP238** defines machining operations with tool references but has no standard model for the tool room itself: no presetter record, no holder checkout, no cutter consumption tracking.
- **ISO 13399** defines cutting tool representation and exchange (CuttingToolRepresentation) for catalog interchange between manufacturers and shops. It covers cutter geometry and catalog identity but not the assembled tool spec (holder + cutter + stickout), not the measured preset, and not any shop-floor lifecycle records.
- **ISO 1832** and **ANSI B212.12** cover nomenclature and designation for indexable inserts; they are classification standards, not data schemas.

The result: the digital thread from NC program → tooling list → presetter → loaded pocket → consumption event → cutter life tracking is assembled from disconnected records in the ERP, the presetter software, and the tool crib. None of these are natively linked. Fixture checkout is typically tracked on paper or in a spreadsheet. Cutter life events are tribal knowledge.

This extension defines the **tool room** as a set of connected entities spanning three layers:

1. **Catalog layer** — what types of tools exist (`cutter_definition`, `holder_definition`, `tool_assembly_definition`, `gauge_definition`)
2. **Measurement and assignment layer** — what was actually measured and loaded (`tool_preset`, `tool_pocket_requirement`, `tool_pocket_assignment`)
3. **Lifecycle and operations layer** — checkout, maintenance, and consumption tracking (`tool_room_allocation`, `tool_maintenance_event`, `tool_life_record`)

---

## Entity Definitions

### cutter_definition

The catalog entry for a cutter type. `cutter_definition` is the abstract specification — manufacturer, part number, geometry, coating. It is not a physical item. Cutters are consumable and are not individually serialized by default; a single cutter_definition may represent hundreds of physical cutters consumed over time. Consumption and life are tracked via `tool_life_record`. Physical cutter inventory (purchase orders, stock counts) links to cutter_definition via the item master, not via a separate instance table.

```yaml
cutter_definition:
  id: uuid                          # UUID v4 — minted when the cutter type is registered in the catalog
  manufacturer: string              # e.g., "Kennametal", "Seco", "Iscar"
  manufacturer_part_number: string  # Manufacturer catalog number
  description: string               # e.g., "0.500 4-flute carbide bull endmill, 0.030 corner radius"
  tool_type: enum                   # EndMill | FaceMill | Drill | Reamer | Tap | BoreTool |
                                    # ChamferMill | ThreadMill | TurningInsert | GrooveInsert | Other
  diameter_in: number               # Cutting diameter in inches (optional)
  corner_radius_in: number          # Corner radius in inches; 0 for sharp corners (optional)
  flute_count: integer              # Number of flutes/cutting edges (optional)
  overall_length_in: number         # Overall cutter length in inches (optional)
  material: enum                    # Carbide | HSS | Ceramic | CBN | Diamond | Other (optional)
  coating: string                   # e.g., "TiAlN", "AlCrN", "uncoated" (optional)
  notes: string                     # (optional)
```

---

### holder_definition

The catalog entry for a holder type — the specification for a holder style (collet, hydro, shrink fit), taper standard, and bore size. `holder_definition` is not a physical item. Physical holders are serialized parts with `part_kind: fixture` in `schemas/part.yaml`; the `holder_definition` is the spec those parts conform to. One `holder_definition` may correspond to many serialized holder parts checked out and tracked via `tool_room_allocation`.

```yaml
holder_definition:
  id: uuid                          # UUID v4 — minted when the holder type is registered in the catalog
  manufacturer: string              # e.g., "BIG-PLUS", "Haimer", "Kennametal"
  manufacturer_part_number: string  # Manufacturer catalog number
  description: string               # e.g., "ER20 collet chuck, CAT40 taper, 4.00 gauge length"
  holder_style: enum                # Collet | HydroChuck | ShrinkFit | MechanicalChuck | BoreTool |
                                    # TurningBlock | FaceGroove | Other
  taper_standard: enum              # CAT40 | CAT50 | BT30 | BT40 | BT50 | HSK63A | HSK100A |
                                    # CAPTO_C6 | Other
  collet_series: string             # e.g., "ER20", "ER32" — null if not a collet holder (optional)
  bore_diameter_in: number          # Cutter bore or collet range maximum in inches (optional)
  gauge_length_in: number           # Holder gauge line to nose in inches (optional)
  notes: string                     # (optional)
```

---

### tool_assembly_definition

A specific combination of one `cutter_definition` plus one `holder_definition` at a nominal stickout — the abstract spec for a complete assembled tool. This is the entity that tooling lists and NC programs reference; it represents what a machinist should build when setting up tool number T#N.

`tool_assembly_definition` is the SPEC (abstract). The corresponding physical reality is a `tool_preset` — a specific serialized holder (`part.id`) with a cutter loaded to a measured stickout as recorded on the presetter on a specific date. One `tool_assembly_definition` may have many `tool_preset` records over time (different holders, different dates, remeasurements after cutter changes).

```yaml
tool_assembly_definition:
  id: uuid                          # UUID v4
  cutter_definition_id: uuid        # FK → cutter_definition.id
  holder_definition_id: uuid        # FK → holder_definition.id
  description: string               # e.g., "T#3 — 0.500 chamfer mill, ER20 holder, 3.125 stickout"
  nominal_stickout_in: number       # Nominal cutter stickout from holder gauge line, inches
  stickout_tolerance_in: number     # Allowed stickout tolerance ±, inches (optional)
  notes: string                     # (optional)
```

---

### gauge_definition

The catalog entry for a gauge type — the specification for a class of measurement instrument (presetter model, caliper model, CMM probe type). `gauge_definition` defines what a gauge *is* (manufacturer, model, measurement range, calibration interval); it is not a physical instrument.

Physical calibrated instruments are `tool_gauge` records defined in `schemas/inspection.yaml`. A `tool_gauge` is the serialized, asset-tagged, calibration-tracked instance of a gauge type. The relationship between `gauge_definition` and `tool_gauge` is currently implicit — the shop knows which presetter model corresponds to which asset tag. A future schema pass may add a `gauge_definition_id` FK to `tool_gauge` to make this link explicit without modifying the existing calibration tracking structure.

```yaml
gauge_definition:
  id: uuid                          # UUID v4 — minted when the gauge type is registered in the catalog
  name: string                      # e.g., "Zoller Venturion 450 Presetter"
  gauge_type: enum                  # Presetter | Caliper | Micrometer | CMM_Probe | Go_NoGo |
                                    # Ring_Gauge | Pin_Gauge | Torque_Wrench | Height_Gauge |
                                    # Surface_Plate | Optical_Comparator | XRF_Gun | Other
  manufacturer: string              # (optional)
  manufacturer_model: string        # (optional)
  measurement_range: string         # e.g., "0–6 in, 0.0001 in resolution" (optional)
  calibration_interval_days: integer # Default calibration interval for instances of this type (optional)
  notes: string                     # (optional)
```

Note: `gauge_definition` has no direct relationship to `tool_gauge` in the current schema. The link is a planned FK addition; until then, correlation is by name and type.

---

### tool_preset

A measured offset record for a specific physical tool assembly — a named holder (serialized `part.id`) with a cutter loaded to a measured stickout, measured on the presetter on a specific date. One `tool_preset` per measurement event. When a cutter is changed or remeasured, a new `tool_preset` is created; old records are preserved as historical measurement evidence.

The presetter is identified via `presetter_id` → `tool_gauge.id` (defined in `schemas/inspection.yaml`). This ensures the measurement instrument used is calibration-tracked and traceable.

```yaml
tool_preset:
  id: uuid                          # UUID v4 — minted at the presetter measurement event
  tool_assembly_definition_id: uuid # FK → tool_assembly_definition.id — which assembly spec this satisfies
  holder_part_id: uuid              # FK → part.id — the specific serialized holder used (part_kind = fixture)
  presetter_id: uuid                # FK → tool_gauge.id — the presetter used for measurement
                                    #   (tool_gauge defined in schemas/inspection.yaml)
  measured_by_id: uuid              # FK → user.id — operator who performed the measurement
  measured_date: date               # Date the preset was measured
  measured_length_in: number        # Measured tool length offset, inches
  measured_diameter_in: number      # Measured tool diameter, inches
  stickout_actual_in: number        # Actual stickout achieved, inches (optional)
  within_tolerance: boolean         # True if stickout_actual_in is within tool_assembly_definition tolerances
  notes: string                     # (optional)
```

---

### tool_pocket_requirement

The tool requirement encoded in an NC program: which pocket number requires which tool assembly. This is the SPEC — what the program needs. It lives on the NC program side of the ledger and drives what must be loaded before a work order can run.

`nc_program` is defined in `schemas/process_engineering.yaml`. This entity references it by FK only.

```yaml
tool_pocket_requirement:
  id: uuid                          # UUID v4
  nc_program_id: uuid               # FK → nc_program.id (defined in schemas/process_engineering.yaml)
  pocket_number: integer            # Tool pocket / turret station number as programmed
  tool_assembly_definition_id: uuid # FK → tool_assembly_definition.id — the required tool assembly
  description: string               # e.g., "T03 — chamfer mill" (optional)
  notes: string                     # (optional)
```

---

### tool_pocket_assignment

The ACTUAL assignment for a specific work order execution: which `tool_preset` (a measured, physical tool assembly) was loaded into which machine pocket, by whom, and when. This bridges the abstract program requirement (`tool_pocket_requirement`) to the physical event (`tool_preset`).

One `tool_pocket_assignment` per pocket per work order. If a cutter is changed mid-run, a new assignment record is created for that pocket when the replacement preset is loaded.

```yaml
tool_pocket_assignment:
  id: uuid                          # UUID v4
  work_order_id: uuid               # FK → work_order.id (defined in schemas/job.yaml)
  pocket_number: integer            # Machine table pocket number loaded
  tool_preset_id: uuid              # FK → tool_preset.id — the measured tool assembly loaded
  loaded_by_id: uuid                # FK → user.id — machinist who loaded the tool
  loaded_date: datetime             # When the tool was loaded (ISO 8601 datetime)
  removed_date: datetime            # When the tool was removed; null while still loaded (optional)
  notes: string                     # (optional)
```

---

### tool_room_allocation

A checkout record for a physical fixture or holder (a part with `part_kind: fixture` in `schemas/part.yaml`) checked out to a job or operation. Tracks custody: who took it, when, expected return, actual return, and outcome.

Physical holders and fixtures are serialized parts — not a separate entity. `tool_room_allocation` references `part.id` as its asset reference, relying on `part.part_kind = fixture` as the constraint. The status field tracks the disposition lifecycle.

```yaml
tool_room_allocation:
  id: uuid                          # UUID v4
  asset_part_id: uuid               # FK → part.id — the physical fixture or holder (part_kind = fixture)
  job_id: uuid                      # FK → job.id (defined in schemas/job.yaml)
  operation_id: uuid                # FK → operation.id — specific operation requiring it (optional)
                                    #   (operation defined in schemas/routing.yaml)
  checked_out_by_id: uuid           # FK → user.id — who checked it out
  checked_out_date: datetime        # Checkout datetime (ISO 8601)
  expected_return_date: date        # Expected return date (optional)
  returned_date: datetime           # Actual return datetime; null while checked out (optional)
  status: enum                      # CheckedOut | Returned | Lost | Damaged
  notes: string                     # (optional)
```

---

### tool_maintenance_event

A service event performed on a physical tool or instrument: sharpening, cleaning, repair, inspection, recertification, or retirement. Covers two asset types via a discriminator pattern:

- When `asset_type` is `Fixture` or `Holder`: `asset_part_id` → `part.id` is populated; `tool_gauge_id` is null.
- When `asset_type` is `Gauge`: `tool_gauge_id` → `tool_gauge.id` (defined in `schemas/inspection.yaml`) is populated; `asset_part_id` is null.
- `Cutter` asset type is included as a discriminator value for tracking events on specific cutter inventory where relevant, but cutters are not individually serialized by default; see `tool_life_record` for typical cutter lifecycle tracking.

```yaml
tool_maintenance_event:
  id: uuid                          # UUID v4
  asset_type: enum                  # Fixture | Holder | Gauge | Cutter
                                    #   Discriminator — determines which FK below is populated
  asset_part_id: uuid               # FK → part.id — populated when asset_type = Fixture or Holder (optional)
  tool_gauge_id: uuid               # FK → tool_gauge.id — populated when asset_type = Gauge (optional)
                                    #   (tool_gauge defined in schemas/inspection.yaml)
  event_type: enum                  # Sharpen | Clean | Repair | Inspect | Recertify | Retire
  performed_by_id: uuid             # FK → user.id
  event_date: date                  # Date the maintenance was performed
  description: string               # Description of work performed
  result: enum                      # Pass | Fail | RetiredAfterService
  notes: string                     # (optional)
```

---

### tool_life_record

Cumulative usage tracking for a cutter type on a specific job or work order run. Records how many parts were produced, optionally the running total, and whether a tool change was triggered. This is the primary mechanism for cutter consumption tracking in the absence of individual cutter serialization.

Over time, `tool_life_record` entries for a given `cutter_definition_id` build a usage history that can inform tool life expectations, optimal change intervals, and purchasing decisions.

```yaml
tool_life_record:
  id: uuid                          # UUID v4
  cutter_definition_id: uuid        # FK → cutter_definition.id — which cutter type was used
  tool_assembly_definition_id: uuid # FK → tool_assembly_definition.id — which assembly spec (optional)
  job_id: uuid                      # FK → job.id (defined in schemas/job.yaml)
  work_order_id: uuid               # FK → work_order.id — specific work order run (optional)
                                    #   (work_order defined in schemas/job.yaml)
  parts_run: integer                # Number of parts produced in this run
  cumulative_parts_run: integer     # Total parts run on this cutter across all runs (optional)
                                    #   Populated when the cutter is individually tracked across jobs
  tool_change_required: boolean     # True if the cutter was replaced after this run
  notes: string                     # (optional)
```

---

## Tool Room Workflow

The complete sequence from NC program authoring through cutter consumption tracking:

```
NC PROGRAM AUTHORING
  | Programmer defines tooling list: T01=face mill, T03=chamfer mill, T05=0.500 endmill
  | tool_pocket_requirement minted per pocket:
  |   T01 → tool_assembly_definition (face mill assy)
  |   T03 → tool_assembly_definition (chamfer mill assy)
  |   T05 → tool_assembly_definition (0.500 endmill assy)
  v
FIXTURE / HOLDER CHECKOUT (tool_room_allocation)
  | Machinist checks out Fixture-A (custom angle plate, part.id = UUID)
  |   → tool_room_allocation: asset_part_id = Fixture-A, job_id, status = CheckedOut
  | Machinist checks out Holder-7 (ER20 collet chuck, part.id = UUID)
  |   → tool_room_allocation: asset_part_id = Holder-7, job_id, status = CheckedOut
  v
PRESETTER MEASUREMENT (tool_preset)
  | Machinist loads cutter into Holder-7, sets stickout
  | Measures on Zoller presetter (tool_gauge.id = UUID, calibration_status = Current)
  |   → tool_preset: holder_part_id = Holder-7, presetter_id = Zoller-UUID
  |                  measured_length_in = 4.2510, measured_diameter_in = 0.4998
  |                  within_tolerance = true
  v
POCKET LOADING (tool_pocket_assignment)
  | Machinist loads T05 pocket with Holder-7 + cutter as measured
  |   → tool_pocket_assignment: work_order_id, pocket_number = 5, tool_preset_id = preset-UUID
  |                              loaded_by_id, loaded_date
  v
MACHINING RUN
  | Work order executes; parts are produced
  v
CUTTER LIFE RECORDING (tool_life_record)
  | After run: machinist records cutter consumption
  |   → tool_life_record: cutter_definition_id = 0.500-endmill-UUID, job_id
  |                        parts_run = 12, tool_change_required = false
  | Subsequent job: parts_run = 10, tool_change_required = true
  |   → cutter replaced; new tool_preset created for replacement cutter
  v
CHECKOUT RETURN (tool_room_allocation update)
  | Fixture-A and Holder-7 returned to tool crib
  |   → tool_room_allocation.status = Returned; returned_date recorded
  v
MAINTENANCE EVENT (tool_maintenance_event)
  | Holder-7 shows wear; sent for inspection
  |   → tool_maintenance_event: asset_type = Holder, asset_part_id = Holder-7
  |                              event_type = Inspect, result = Pass
  | Presetter (Zoller) due for calibration
  |   → calibration_record (in schemas/inspection.yaml); tool_gauge.calibration_status = Current
```

---

## Layer-1 Gap Analysis

| Tool Room Function | MTConnect CuttingTool | ISO 13399 | STEP AP238 | This Extension |
|---|---|---|---|---|
| Cutter type catalog | Partial (asset identity) | Yes (geometry/designation) | No | cutter_definition |
| Holder type catalog | Partial (holder identity) | Partial | No | holder_definition |
| Assembled tool spec (cutter+holder+stickout) | No | No | No | tool_assembly_definition |
| Gauge type catalog | No | No | No | gauge_definition |
| Presetter measurement record | No | No | No | tool_preset |
| NC program → pocket → tool spec | No | No | Partial | tool_pocket_requirement |
| Actual pocket loading per work order | Partial (asset ID) | No | No | tool_pocket_assignment |
| Fixture/holder checkout tracking | No | No | No | tool_room_allocation |
| Tool maintenance history | No | No | No | tool_maintenance_event |
| Cutter consumption / life tracking | Partial (life count) | No | No | tool_life_record |

The MTConnect CuttingTool asset schema is the closest existing standard. It covers the machine-side view of a loaded tool (asset ID, life counts, cutting item geometry on the controller), but it does not cover the shop-side workflow that assembles, measures, approves, and tracks that tool before and after it is on the machine. This extension fills that gap.

---

## Integration with the Digital Thread

| Tool Room Data | From | Linked To |
|---|---|---|
| cutter_definition.id | Tool room catalog | tool_assembly_definition, tool_life_record |
| holder_definition.id | Tool room catalog | tool_assembly_definition |
| tool_assembly_definition.id | Process engineering | tool_pocket_requirement, tool_preset |
| tool_preset.id | Presetter measurement | tool_pocket_assignment |
| tool_preset.presetter_id | tool_gauge (inspection.yaml) | Calibration traceability |
| tool_pocket_assignment.work_order_id | Job execution | work_order (job.yaml) |
| tool_room_allocation.asset_part_id | Fixture checkout | part (part.yaml, part_kind = fixture) |
| tool_life_record.cutter_definition_id | Cutter consumption | Purchasing, tool life optimization |

**Calibration Traceability:**

Every `tool_preset` references the specific presetter (`tool_gauge.id`) used for measurement. Because `tool_gauge` carries `calibration_status` and `current_calibration_id`, the traceability chain from a measured offset back to a NIST-traceable calibration record is complete:

```
tool_preset.presetter_id
  → tool_gauge.id (inspection.yaml)
  → tool_gauge.current_calibration_id
  → calibration_record.id (inspection.yaml)
  → calibration_record.lab_accreditation ("A2LA #3742.01")
```

This chain satisfies AS9100 Rev D §8.5.1 (process control evidence) and customer flowdown requirements for traceable measurement of NC program offsets.

---

## Implementation Notes (the production system Reference)

Tool room entities in the production system are currently managed across several disconnected areas:

- **Item master** — cutter types live as purchased items; holder types live as fixtures; neither has a structured assembly relationship
- **Job routing tool list** — tool numbers are documented in routing notes or attached PDF tooling lists; not linked to a catalog entity
- **Presetter software** — Zoller or similar exports offset data as CSV or proprietary format; not imported into the ERP as structured records
- **Checkout** — paper log or spreadsheet in the tool crib; no ERP record

This extension provides the data model for a future integrated tool room system. The Claude MCP integration enables queries like:

- "Which tool_pocket_requirements for open work orders have no current tool_preset within tolerance?"
- "Show me all cutter_definitions that have required a tool change in the last 30 days and their average parts_run"
- "Which fixtures are currently checked out and overdue for return?"
- "What tool_preset was loaded in pocket T05 for work order WO-25-0412, and is the presetter it was measured on currently calibrated?"
