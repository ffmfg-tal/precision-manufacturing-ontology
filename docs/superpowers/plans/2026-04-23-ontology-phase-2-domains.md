# Ontology Phase 2 — Domains 9–12 & Cross-Cutting Additions Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend the contract-manufacturer ontology from 8 domains (82 entities) to 12 domains (~120 entities) by authoring the Process Engineering/NRE, Tool Room, Packaging, and Change Management domains, plus cross-cutting additions to existing schemas and 4 new composition-archetype validation walks.

**Architecture:** Phase 1 tasks are fully independent (no shared file conflicts) and can run in parallel subagents. Phase 2 modifies shared reference files and must run sequentially after Phase 1. Phase 3 performs validation and cross-reference cleanup. Existing `schemas/inspection.yaml` (inspection_event, calibration_record, tool_gauge, key_characteristic) is authoritative and must not be re-created — only referenced and extended.

**Tech Stack:** Markdown prose + YAML schema; same patterns as `extensions/outside-processing.md` for prose files and `schemas/routing.yaml` for YAML files.

---

## Pre-Flight: Conventions

All new prose files follow `extensions/outside-processing.md` structure:
- `## The Gap This Fills` → why Layer-1 standards don't cover this
- `## Entity Definitions` → YAML blocks + prose explanation for each entity
- `## Decision Log` → numbered architectural decisions

All new YAML files follow `schemas/routing.yaml` structure:
- Header comment: Layer-1 coverage notes + architectural decisions + entity list
- `entities:` root key with one child per entity
- Each entity: `description:`, `attributes:`, `relationships:`
- Attributes: `type:`, `required:`, `description:`, optional `examples:`
- Relationships: `target:`, `cardinality:`, optional `description:`

Cardinality values: `one-to-one`, `one-to-many`, `many-to-one`, `many-to-many`

---

## Phase 1 — New Domain Authoring (PARALLEL)

Tasks 1–5 have no shared file conflicts and can run in parallel subagents.

---

### Task 1: Process Engineering / NRE Domain

**Files:**
- Create: `extensions/process-engineering-nre.md`
- Create: `schemas/process_engineering.yaml`

Entities: `nc_program`, `cam_file`, `setup_sheet`, `cmm_program`, `inspection_plan`, `nre_package`, `prove_out_record`, `print_file`, `tooling_list`, `tooling_list_line`, `fixture_use`

**Key architectural decisions to encode:**

- `cam_file` → `nc_program` is one-to-many: one CAM file posts to multiple machines (Mazak vs. DMG); machine flex without routing change
- `nc_program` is per-operation × per-machine: `(operation_id, work_center_id)` pair is the natural key; same operation may have programs for multiple machines
- `nre_package` is the parent envelope that locks together: `part_specification_id` (drawing revision) + `routing_id` (routing revision) + all child documents (NC programs, setup sheets, CMM programs, inspection plans); revision-locking is the invariant
- `fixture_use` links an operation to the fixture(s) it requires (part with part_kind: fixture); one operation may use multiple fixtures; one fixture may appear in multiple operations across multiple routings
- `prove_out_record` is the immutable evidence that an NC program was proven on a physical machine: captures machine, operator, first-piece result, approval state
- `print_file` is the controlled copy of the drawing revision packaged with the NRE set (separate from `part_specification.pdf_file_ref` which lives at the drawing level)
- `inspection_plan` is the pre-defined measurement plan: which key characteristics to check, in what order, with what gauge type; QIF linkage optional

**Layer-1 gap to describe in prose:** STEP AP238 covers process plan structure for CNC but has no concept of the NC program file itself, the prove-out event, or the CAM-to-NC relationship. STEP AP242 covers PMI and key characteristics but not the inspection plan ordering. Neither standard has an NRE package as an organizing entity.

- [ ] **Step 1: Write `extensions/process-engineering-nre.md`**

Prose structure:
```markdown
# Process Engineering / NRE Extension

## The Gap This Fills
[paragraph on STEP AP238/AP242 gap — no NC file entity, no prove-out record, no NRE package organizing the set. Manufacturer-layer requirement to lock part rev + routing rev + all supporting documents together so that a routing change or drawing revision triggers a re-approval cycle.]

## Entity Definitions

### NRE Package
[prose + yaml block]

### NC Program
[prose + yaml block]

### CAM File
[prose + yaml block]

### Setup Sheet
[prose + yaml block]

### CMM Program
[prose + yaml block]

### Inspection Plan
[prose + yaml block]

### Prove-Out Record
[prose + yaml block]

### Print File
[prose + yaml block]

### Tooling List
[prose + yaml block]

### Tooling List Line
[prose + yaml block]

### Fixture Use
[prose + yaml block]

## Decision Log
[decisions: cam-to-nc one-to-many, nc_program per operation×machine, nre_package revision-lock invariant, fixture_use as join table, prove_out_record as immutable evidence]
```

Full YAML blocks for each entity (write inline in the prose file):

```yaml
nre_package:
  id: uuid                        # UUID v4
  part_id: uuid                   # FK → part.id
  part_specification_id: uuid     # FK → part_specification.id (drawing revision locked)
  routing_id: uuid                # FK → routing.id (routing revision locked)
  status: enum                    # Draft | InReview | Approved | Superseded
  approved_by_id: uuid            # FK → user.id — quality/engineering sign-off
  approved_date: date
  notes: string
```

```yaml
nc_program:
  id: uuid
  operation_id: uuid              # FK → operation.id (the routing template step)
  work_center_id: uuid            # FK → work_center.id (the specific machine)
  nre_package_id: uuid            # FK → nre_package.id
  program_number: string          # Shop program number (e.g., "7832-001-OP10-MAZAK1")
  revision: string                # Program revision letter
  cam_file_id: uuid               # FK → cam_file.id — source CAM file
  file_reference: string          # Path/URL to the NC file (e.g., "nc-programs/7832-001-OP10-v3.nc")
  status: enum                    # Draft | Proven | Released | Obsolete
  notes: string
```

```yaml
cam_file:
  id: uuid
  nre_package_id: uuid            # FK → nre_package.id
  operation_id: uuid              # FK → operation.id
  file_reference: string          # Path/URL to the CAM project file
  cad_model_reference: string     # Path/URL to the 3D model used as CAM input
  cam_system: string              # CAM software name/version (e.g., "Mastercam 2025")
  revision: string
  created_by_id: uuid             # FK → user.id (CAM programmer)
  created_date: date
  notes: string
```

```yaml
setup_sheet:
  id: uuid
  operation_id: uuid              # FK → operation.id
  work_center_id: uuid            # FK → work_center.id
  nre_package_id: uuid            # FK → nre_package.id
  revision: string
  file_reference: string          # Path/URL to PDF or structured setup sheet file
  fixture_id: uuid                # FK → part.id (where part_kind = fixture) — primary fixture shown
  notes: string
```

```yaml
cmm_program:
  id: uuid
  operation_id: uuid              # FK → operation.id (the inspection operation, e.g., op 50 CMM)
  work_center_id: uuid            # FK → work_center.id (the CMM machine)
  nre_package_id: uuid            # FK → nre_package.id
  program_name: string            # e.g., "7832-001-FAI-CMM.mpx"
  revision: string
  cmm_system: string              # CMM software (e.g., "PC-DMIS 2023", "Calypso 2024")
  file_reference: string          # Path/URL to the CMM program file
  qif_template_reference: string  # Optional path to QIF plan file this program implements
  notes: string
```

```yaml
inspection_plan:
  id: uuid
  part_id: uuid                   # FK → part.id
  nre_package_id: uuid            # FK → nre_package.id
  revision: string
  plan_type: enum                 # FAI | InProcess | Receiving | Final
  file_reference: string          # Path/URL to inspection plan document
  qif_plan_reference: string      # Optional QIF Resources plan file
  notes: string
```

```yaml
prove_out_record:
  id: uuid
  nc_program_id: uuid             # FK → nc_program.id
  work_center_id: uuid            # FK → work_center.id (machine actually used)
  operator_id: uuid               # FK → user.id
  prove_out_date: date
  first_piece_result: enum        # Pass | Fail | Conditional
  approved_by_id: uuid            # FK → user.id — engineer or quality signoff
  approved_date: date
  notes: string
  file_reference: string          # Path/URL to prove-out record / first-piece inspection sheet
```

```yaml
print_file:
  id: uuid
  nre_package_id: uuid            # FK → nre_package.id
  part_specification_id: uuid     # FK → part_specification.id (the revision this print represents)
  file_reference: string          # Controlled copy of the drawing PDF
  revision: string                # Drawing revision (matches part_specification.revision)
  distribution_statement: enum    # Unlimited | Export_Controlled_EAR | ITAR_Controlled | CUI | Proprietary
  notes: string
```

```yaml
tooling_list:
  id: uuid
  operation_id: uuid              # FK → operation.id (the machining operation)
  nc_program_id: uuid             # FK → nc_program.id (the program this list supports)
  nre_package_id: uuid            # FK → nre_package.id
  revision: string
  notes: string
```

```yaml
tooling_list_line:
  id: uuid
  tooling_list_id: uuid           # FK → tooling_list.id
  pocket_number: integer          # Tool pocket number in the NC program (T#1, T#12, etc.)
  tool_assembly_definition_id: uuid # FK → tool_assembly_definition.id (the required tool assembly spec)
  description: string             # Human-readable: "T#3 — 0.500 chamfer mill, ER20 holder, 3.125 stickout"
  sequence_position: integer      # Order in the tool list (conventionally = pocket_number)
  notes: string
```

```yaml
fixture_use:
  id: uuid
  operation_id: uuid              # FK → operation.id (the operation requiring the fixture)
  fixture_part_id: uuid           # FK → part.id (where part_kind = fixture)
  description: string             # Role description: "primary workholding", "secondary clamp"
  required: boolean               # True = this fixture is mandatory; False = alternative exists
  notes: string
```

- [ ] **Step 2: Write `schemas/process_engineering.yaml`**

Translate all YAML blocks from the prose file into the canonical schema format with full `attributes:` and `relationships:` sections. Header comment must reference `extensions/process-engineering-nre.md` as prose source. Entity list in header: `nre_package`, `nc_program`, `cam_file`, `setup_sheet`, `cmm_program`, `inspection_plan`, `prove_out_record`, `print_file`, `tooling_list`, `tooling_list_line`, `fixture_use`.

Relationships to include:
- `nre_package`: `for_part` (→ part, many-to-one), `for_specification` (→ part_specification, many-to-one), `for_routing` (→ routing, many-to-one), `approved_by` (→ user, many-to-one), `contains_nc_programs` (→ nc_program, one-to-many), `contains_cam_files` (→ cam_file, one-to-many), `contains_setup_sheets` (→ setup_sheet, one-to-many), `contains_cmm_programs` (→ cmm_program, one-to-many), `contains_inspection_plans` (→ inspection_plan, one-to-many), `contains_tooling_lists` (→ tooling_list, one-to-many)
- `nc_program`: `for_operation` (→ operation, many-to-one), `at_work_center` (→ work_center, many-to-one), `in_nre_package` (→ nre_package, many-to-one), `from_cam_file` (→ cam_file, many-to-one), `has_prove_out_records` (→ prove_out_record, one-to-many)
- `cam_file`: `for_operation` (→ operation, many-to-one), `in_nre_package` (→ nre_package, many-to-one), `created_by` (→ user, many-to-one), `posts_to_nc_programs` (→ nc_program, one-to-many)
- `setup_sheet`: `for_operation` (→ operation, many-to-one), `at_work_center` (→ work_center, many-to-one), `in_nre_package` (→ nre_package, many-to-one), `shows_fixture` (→ part, many-to-one)
- `cmm_program`: `for_operation` (→ operation, many-to-one), `at_work_center` (→ work_center, many-to-one), `in_nre_package` (→ nre_package, many-to-one)
- `inspection_plan`: `for_part` (→ part, many-to-one), `in_nre_package` (→ nre_package, many-to-one)
- `prove_out_record`: `for_nc_program` (→ nc_program, many-to-one), `at_work_center` (→ work_center, many-to-one), `by_operator` (→ user, many-to-one), `approved_by` (→ user, many-to-one)
- `print_file`: `in_nre_package` (→ nre_package, many-to-one), `for_specification` (→ part_specification, many-to-one)
- `tooling_list`: `for_operation` (→ operation, many-to-one), `for_nc_program` (→ nc_program, many-to-one), `in_nre_package` (→ nre_package, many-to-one), `has_lines` (→ tooling_list_line, one-to-many)
- `tooling_list_line`: `in_tooling_list` (→ tooling_list, many-to-one), `requires_tool_assembly` (→ tool_assembly_definition, many-to-one)
- `fixture_use`: `for_operation` (→ operation, many-to-one), `uses_fixture` (→ part, many-to-one)

- [ ] **Step 3: Verify cross-references are consistent**

Check that every FK referenced in the YAML files points to an entity that exists in another schema or is defined in this file. Specifically confirm:
- `operation.id` exists in `schemas/routing.yaml` ✅
- `work_center.id` exists in `schemas/routing.yaml` ✅
- `part_specification.id` exists in `schemas/part.yaml` ✅
- `part.id` (fixture) exists in `schemas/part.yaml` ✅
- `tool_assembly_definition.id` will exist in Task 2 (schemas/tool_room.yaml) — annotate with `# defined in schemas/tool_room.yaml`
- `routing.id` exists in `schemas/routing.yaml` ✅
- `user.id` exists in `schemas/user.yaml` ✅

---

### Task 2: Tool Room Domain

**Files:**
- Create: `extensions/tool-room.md`
- Create: `schemas/tool_room.yaml`

Entities: `cutter_definition`, `holder_definition`, `tool_assembly_definition`, `gauge_definition`, `tool_preset`, `tool_pocket_requirement`, `tool_pocket_assignment`, `tool_room_allocation`, `tool_maintenance_event`, `tool_life_record`

**Key architectural decisions:**

- Physical serialized holders and fixtures ARE parts (part_kind: fixture). `tool_room_allocation` references `part.id` for the physical asset being checked out — no separate `holder_instance` or `fixture_instance` entity.
- `cutter_definition` and `holder_definition` are CATALOG entities (specs, not physical items). Cutters are consumable; individual cutters are not assigned UUIDs unless the shop chooses to track them as serialized parts.
- `tool_assembly_definition` = specific cutter_definition + holder_definition + stickout spec. It is the abstract specification that `tooling_list_line.tool_assembly_definition_id` references and that `tool_pocket_requirement` enforces.
- `gauge_definition` is the CATALOG entry for a gauge type. `tool_gauge` (in `schemas/inspection.yaml`) is the INSTANCE of a specific calibrated instrument. This file defines `gauge_definition`; do not re-define `tool_gauge`.
- `tool_preset` = measured offsets (length, diameter) for a specific physical tool assembly (holder + cutter) measured on the presetter before being loaded into a machine. One preset per assembly-per-measurement event.
- `tool_pocket_requirement` links `nc_program` → `tool_assembly_definition` with pocket number and offset expectations. It is the encoded requirement in the NC program.
- `tool_pocket_assignment` links `work_center` machine table pocket → `tool_preset` for a specific job. It is the actual assignment — which physical preset went into which pocket.
- `tool_room_allocation` is the checkout record: who took which physical asset (part with part_kind: fixture), when, for which job, and when it was returned.
- `tool_maintenance_event` records sharpening, cleaning, repair, inspection of a physical tool or fixture.
- `tool_life_record` tracks cumulative usage: cutting edges used, total parts run, tool change triggers.

**Layer-1 gap:** MTConnect CuttingTool asset schema covers tool identity and asset ID but not the presetter measurement workflow, the checkout/allocation system, or the holder+cutter catalog hierarchy.

- [ ] **Step 1: Write `extensions/tool-room.md`**

```markdown
# Tool Room Extension

## The Gap This Fills
[paragraph: MTConnect CuttingTool asset model has identity (asset ID, tool life) but no catalog hierarchy (cutter_definition × holder_definition = assembly_definition), no presetter measurement record (tool_preset), no checkout system (tool_room_allocation), and no gauge_definition catalog separate from the calibrated instrument (tool_gauge in inspection.yaml). The tool room manages a physical inventory of serialized holders and fixtures (tracked as parts), consumable cutters (tracked by definition + life record), and gauges (tool_gauge instances). This extension defines the catalog definitions and operational records that bridge the tool room to the NC program requirements and the job execution tracking.]

## Entity Definitions

### Cutter Definition
### Holder Definition
### Tool Assembly Definition
### Gauge Definition
### Tool Preset
### Tool Pocket Requirement
### Tool Pocket Assignment
### Tool Room Allocation
### Tool Maintenance Event
### Tool Life Record

## Decision Log
```

Full YAML blocks for each entity:

```yaml
cutter_definition:
  id: uuid
  manufacturer: string            # Tool manufacturer (e.g., "Kennametal", "Sandvik")
  manufacturer_part_number: string # Manufacturer catalog number
  description: string             # e.g., "0.500 4-flute carbide bull endmill, 0.030 corner radius"
  tool_type: enum                 # EndMill | FaceMill | Drill | Reamer | Tap | BoreTool |
                                  # ChamferMill | ThreadMill | TurningInsert | GrooveInsert | Other
  diameter_in: number             # Cutting diameter in inches
  corner_radius_in: number        # Corner radius in inches (0 for sharp)
  flute_count: integer
  overall_length_in: number       # Overall length in inches
  material: enum                  # Carbide | HSS | Ceramic | CBN | Diamond | Other
  coating: string                 # Coating description (e.g., "TiAlN", "AlCrN", "uncoated")
  notes: string
```

```yaml
holder_definition:
  id: uuid
  manufacturer: string
  manufacturer_part_number: string
  description: string             # e.g., "ER20 collet chuck, CAT40 taper, 4.00 gauge length"
  holder_style: enum              # Collet | HydroChuck | ShrinkFit | MechanicalChuck |
                                  # BoreTool | TurningBlock | FaceGroove | Other
  taper_standard: enum            # CAT40 | CAT50 | BT30 | BT40 | BT50 | HSK63A |
                                  # HSK100A | CAPTO_C6 | Other
  collet_series: string           # e.g., "ER20", "ER32", "TG100" (null if not collet holder)
  bore_diameter_in: number        # Cutter bore / collet range max in inches
  gauge_length_in: number         # Holder gauge line to nose (inches)
  notes: string
```

```yaml
tool_assembly_definition:
  id: uuid
  cutter_definition_id: uuid      # FK → cutter_definition.id
  holder_definition_id: uuid      # FK → holder_definition.id
  description: string             # e.g., "T#3 — 0.500 chamfer mill, ER20 holder, 3.125 stickout"
  nominal_stickout_in: number     # Nominal cutter stickout from holder gauge line (inches)
  stickout_tolerance_in: number   # Allowed stickout tolerance ±
  notes: string
```

```yaml
gauge_definition:
  id: uuid
  name: string                    # e.g., "Zoller Venturion 450 Presetter", "Mitutoyo 500 Series Caliper"
  gauge_type: enum                # Presetter | Caliper | Micrometer | CMM_Probe | Go_NoGo |
                                  # Ring_Gauge | Pin_Gauge | Torque_Wrench | Height_Gauge |
                                  # Surface_Plate | Optical_Comparator | XRF_Gun | Other
  manufacturer: string
  manufacturer_model: string
  measurement_range: string       # e.g., "0–6 in, 0.0001 in resolution"
  calibration_interval_days: integer # Default calibration interval for instances of this definition
  notes: string
```

```yaml
tool_preset:
  id: uuid
  tool_assembly_definition_id: uuid  # FK → tool_assembly_definition.id
  holder_part_id: uuid               # FK → part.id (where part_kind = fixture; the serialized holder)
  presetter_id: uuid                 # FK → tool_gauge.id (the presetter used; tool_gauge in inspection.yaml)
  measured_by_id: uuid               # FK → user.id
  measured_date: date
  measured_length_in: number         # Measured tool length offset (inches)
  measured_diameter_in: number       # Measured tool diameter (inches)
  stickout_actual_in: number         # Actual stickout achieved
  within_tolerance: boolean          # True if all measurements within tool_assembly_definition tolerances
  notes: string
```

```yaml
tool_pocket_requirement:
  id: uuid
  nc_program_id: uuid             # FK → nc_program.id (defined in schemas/process_engineering.yaml)
  pocket_number: integer          # Tool pocket / turret station number in the NC program
  tool_assembly_definition_id: uuid  # FK → tool_assembly_definition.id (required tool type)
  description: string             # e.g., "T03 — chamfer mill"
  notes: string
```

```yaml
tool_pocket_assignment:
  id: uuid
  work_order_id: uuid             # FK → work_order.id (the job operation execution instance)
  pocket_number: integer          # Machine table pocket number
  tool_preset_id: uuid            # FK → tool_preset.id (the actual measured preset loaded)
  loaded_by_id: uuid              # FK → user.id (operator who loaded the tool)
  loaded_date: datetime
  removed_date: datetime
  notes: string
```

```yaml
tool_room_allocation:
  id: uuid
  asset_part_id: uuid             # FK → part.id (the physical fixture or holder being checked out)
  job_id: uuid                    # FK → job.id (the job this asset is allocated to)
  operation_id: uuid              # FK → operation.id (the specific operation requiring it)
  checked_out_by_id: uuid         # FK → user.id
  checked_out_date: datetime
  expected_return_date: date
  returned_date: datetime
  status: enum                    # CheckedOut | Returned | Lost | Damaged
  notes: string
```

```yaml
tool_maintenance_event:
  id: uuid
  asset_part_id: uuid             # FK → part.id (fixture/holder) OR tool_gauge_id for gauges
  asset_type: enum                # Fixture | Holder | Gauge | Cutter
  tool_gauge_id: uuid             # FK → tool_gauge.id (when asset_type = Gauge)
  event_type: enum                # Sharpen | Clean | Repair | Inspect | Recertify | Retire
  performed_by_id: uuid           # FK → user.id
  event_date: date
  description: string
  result: enum                    # Pass | Fail | RetiredAfterService
  notes: string
```

```yaml
tool_life_record:
  id: uuid
  cutter_definition_id: uuid      # FK → cutter_definition.id (the cutter type being tracked)
  tool_assembly_definition_id: uuid # FK → tool_assembly_definition.id (the assembly it was used in)
  job_id: uuid                    # FK → job.id
  work_order_id: uuid             # FK → work_order.id
  parts_run: integer              # Parts produced in this run
  cumulative_parts_run: integer   # Total parts run on this cutter instance (if tracked)
  tool_change_required: boolean   # True if cutter was changed/replaced after this run
  notes: string
```

- [ ] **Step 2: Write `schemas/tool_room.yaml`**

Translate all YAML blocks from prose into canonical schema format. Header comment must:
- Note `tool_gauge` and `calibration_record` are NOT defined here — they are in `schemas/inspection.yaml`
- Note `nc_program` is defined in `schemas/process_engineering.yaml`
- List entities: `cutter_definition`, `holder_definition`, `tool_assembly_definition`, `gauge_definition`, `tool_preset`, `tool_pocket_requirement`, `tool_pocket_assignment`, `tool_room_allocation`, `tool_maintenance_event`, `tool_life_record`

Relationships to include:
- `cutter_definition`: `used_in_assemblies` (→ tool_assembly_definition, one-to-many)
- `holder_definition`: `used_in_assemblies` (→ tool_assembly_definition, one-to-many)
- `tool_assembly_definition`: `uses_cutter` (→ cutter_definition, many-to-one), `uses_holder` (→ holder_definition, many-to-one), `required_in_tooling_lines` (→ tooling_list_line, one-to-many), `has_presets` (→ tool_preset, one-to-many), `pocket_requirements` (→ tool_pocket_requirement, one-to-many)
- `gauge_definition`: `instances` (→ tool_gauge [inspection.yaml], one-to-many) — note this is a conceptual relationship; add FK `gauge_definition_id` annotation on tool_gauge
- `tool_preset`: `for_assembly` (→ tool_assembly_definition, many-to-one), `for_holder` (→ part, many-to-one), `measured_with` (→ tool_gauge [inspection.yaml], many-to-one), `measured_by` (→ user, many-to-one), `used_in_assignments` (→ tool_pocket_assignment, one-to-many)
- `tool_pocket_requirement`: `for_nc_program` (→ nc_program [process_engineering.yaml], many-to-one), `requires_assembly` (→ tool_assembly_definition, many-to-one)
- `tool_pocket_assignment`: `for_work_order` (→ work_order [job.yaml], many-to-one), `uses_preset` (→ tool_preset, many-to-one), `loaded_by` (→ user, many-to-one)
- `tool_room_allocation`: `for_asset` (→ part, many-to-one), `for_job` (→ job [job.yaml], many-to-one), `for_operation` (→ operation [routing.yaml], many-to-one), `checked_out_by` (→ user, many-to-one)
- `tool_maintenance_event`: `for_fixture` (→ part, many-to-one), `for_gauge` (→ tool_gauge [inspection.yaml], many-to-one), `performed_by` (→ user, many-to-one)
- `tool_life_record`: `for_cutter` (→ cutter_definition, many-to-one), `for_assembly` (→ tool_assembly_definition, many-to-one), `for_job` (→ job [job.yaml], many-to-one), `for_work_order` (→ work_order [job.yaml], many-to-one)

- [ ] **Step 3: Verify no duplication with inspection.yaml**

Confirm `gauge_definition` is defined here (catalog) and `tool_gauge` is NOT re-defined here (it's in inspection.yaml). The relationship `gauge_definition → tool_gauge` is documented in prose as a conceptual one-to-many, noting that a future migration may add `gauge_definition_id` FK to `tool_gauge`.

---

### Task 3: Packaging Domain

**Files:**
- Create: `extensions/packaging.md`
- Create: `schemas/packaging.yaml`

Entities: `packaging_specification`, `packing_record`, `packaging_item`

**Key architectural decisions:**

- 3D-printed packaging inserts and foam cutouts ARE parts (part_kind: packaging_insert). They have their own part numbers, drawings, and routings. They flow through the full part lifecycle (including inspection and FAI if required by the customer).
- `packaging_specification` is the governing document for how a part or assembly must be packaged: preservation level (MIL-STD-2073), moisture barrier requirements, ESD controls, labeling requirements (MIL-STD-129, MIL-STD-130).
- `packing_record` is the execution instance: evidence that a specific serial or lot was packaged according to its spec. Immutable after sign-off.
- `packaging_item` is a line in the packing_record listing each physical packaging component used (bag, desiccant, insert, box, label).
- Mil-spec preservation levels: A (long-term storage/overseas shipment), B (moderate storage), C (spot-pack/domestic). Level A requires heat-sealed poly bags, desiccant, humidity indicator, and MIL-D-3464 requirements.

**Layer-1 gap:** No Layer-1 manufacturing standard defines packaging as a traceable manufacturing operation. MIL-STD-2073 defines preservation requirements but has no data schema. MIL-STD-129 defines marking/labeling requirements. Shipment records in the ontology (schemas/shipment.yaml) track logistics but not the packaging execution evidence required by AS9100 §8.5.4.

- [ ] **Step 1: Write `extensions/packaging.md`**

```yaml
packaging_specification:
  id: uuid
  part_id: uuid                   # FK → part.id OR assembly_id: uuid
  assembly_id: uuid               # FK → assembly.id (one of part_id or assembly_id required)
  subject_type: enum              # part | assembly (discriminator)
  revision: string
  preservation_level: enum        # A | B | C | None
                                  # A = MIL-STD-2073 Level A (heat-sealed, desiccant, HIC)
                                  # B = Level B (moderate storage)
                                  # C = spot-pack (domestic, short-duration)
                                  # None = commercial packaging per customer instruction
  moisture_barrier_required: boolean
  esd_protection_required: boolean
  desiccant_required: boolean
  humidity_indicator_required: boolean
  max_pieces_per_bag: integer
  max_pieces_per_box: integer
  labeling_standard: string       # e.g., "MIL-STD-129", "MIL-STD-130", "customer spec"
  special_handling_notes: string
  customer_packaging_spec_ref: string  # Customer-supplied spec document reference
  packaging_insert_part_id: uuid  # FK → part.id (where part_kind = packaging_insert), if applicable
  approved_by_id: uuid            # FK → user.id
  approved_date: date
  notes: string
```

```yaml
packing_record:
  id: uuid
  shipment_id: uuid               # FK → shipment.id (the outbound shipment this packing supports)
  packaging_specification_id: uuid # FK → packaging_specification.id
  part_id: uuid                   # FK → part.id (what was packed)
  assembly_id: uuid               # FK → assembly.id (if packing an assembly)
  subject_type: enum              # part | assembly
  serial_id: uuid                 # FK → serial_number.id (if serialized)
  lot_id: uuid                    # FK → material_lot.id (if lot-controlled)
  quantity_packed: integer
  packed_by_id: uuid              # FK → user.id
  packed_date: date
  inspected_by_id: uuid           # FK → user.id (quality inspector who verified packaging)
  inspection_date: date
  result: enum                    # Pass | Fail | Rework
  notes: string
```

```yaml
packaging_item:
  id: uuid
  packing_record_id: uuid         # FK → packing_record.id
  item_type: enum                 # Bag | Box | DesiccantUnit | HumidityIndicator |
                                  # VCIBag | ESDShielding | Label | Foam | CustomInsert | Other
  part_id: uuid                   # FK → part.id (when item_type = CustomInsert or other manufactured item)
  description: string             # e.g., "4×8 heat-sealed poly bag", "2-unit desiccant per MIL-D-3464"
  quantity: integer
  lot_or_batch: string            # Lot/batch number of the packaging material (for traceability)
  notes: string
```

- [ ] **Step 2: Write `schemas/packaging.yaml`**

Full canonical YAML with attributes and relationships:
- `packaging_specification`: relationships: `for_part` (→ part, many-to-one), `for_assembly` (→ assembly, many-to-one), `uses_insert` (→ part, many-to-one, description: packaging insert part), `approved_by` (→ user, many-to-one)
- `packing_record`: relationships: `for_shipment` (→ shipment, many-to-one), `governed_by` (→ packaging_specification, many-to-one), `for_part` (→ part, many-to-one), `for_assembly` (→ assembly, many-to-one), `for_serial` (→ serial_number, many-to-one), `for_lot` (→ material_lot, many-to-one), `packed_by` (→ user, many-to-one), `inspected_by` (→ user, many-to-one), `has_items` (→ packaging_item, one-to-many)
- `packaging_item`: relationships: `in_record` (→ packing_record, many-to-one), `for_part` (→ part, many-to-one, description: manufactured packaging insert)

---

### Task 4: Change Management + Assembly Hardware + Operator Qualification

**Files:**
- Create: `extensions/change-management.md`
- Create: `schemas/change_management.yaml` (engineering_change_notice, deviation_waiver)
- Create: `schemas/assembly_hardware.yaml` (insert_installation_record, press_fit_record)
- Create: `schemas/operator_qualification.yaml` (operator_qualification)

**Key architectural decisions:**

- `engineering_change_notice` is the document-level entity that authorizes a change to a part number, revision, or routing. It triggers a cascade: new `part_specification` revision, new `routing` revision, potential new `nre_package`, and invalidation of any in-flight jobs against the old revision. The ECN does NOT describe the entire cascade — it records the authorization and the affected drawing/routing revisions.
- `deviation_waiver` covers two cases: **deviation** (use-as-is for a specific lot/serial before production; pre-approval) and **waiver** (accept nonconforming material after the fact; post-production). Distinct from `nonconformance_report` (internal quality record). Deviation/waiver requires customer or DER approval; NCR is internal disposition.
- `insert_installation_record` covers helicoil, keensert, and other threaded insert installation. It is an execution record (like `torque_readings_record`) for an `assembly_operation` with `operation_code: INSERT`.
- `press_fit_record` covers interference-fit pin installation, bearing races, bushings. Captures press force (lbf), press displacement (in), and pass/fail. Like `insert_installation_record`, it is tied to an `assembly_operation`.
- `operator_qualification` records that a specific operator is qualified to perform a specific operation type or process specification. Similar in purpose to `nadcap_certification` (which tracks the supplier's cert); this tracks the individual operator's training and certification currency. Required for special processes (welding, NDT, torque-critical assembly).

- [ ] **Step 1: Write `extensions/change-management.md`**

```yaml
engineering_change_notice:
  id: uuid
  ecn_number: string              # Shop-assigned ECN number (e.g., "ECN-2026-0042")
  title: string                   # Short description of the change
  description: string             # Full change description — what changed and why
  change_type: enum               # Drawing | Routing | Material | Process | Specification
  initiator_id: uuid              # FK → user.id — who originated the change
  initiated_date: date
  status: enum                    # Draft | UnderReview | Approved | Rejected | Cancelled | Released
  approved_by_id: uuid            # FK → user.id — engineering/quality sign-off
  approved_date: date
  effective_date: date            # Date the change goes into production effect
  affected_part_id: uuid          # FK → part.id (the part whose drawing/routing changes)
  old_part_specification_id: uuid # FK → part_specification.id (the revision being superseded)
  new_part_specification_id: uuid # FK → part_specification.id (the new revision)
  old_routing_id: uuid            # FK → routing.id (routing being obsoleted)
  new_routing_id: uuid            # FK → routing.id (new routing version)
  customer_notification_required: boolean
  customer_approval_required: boolean
  customer_approval_id: uuid      # FK → customer_approval.id (if required)
  notes: string
```

```yaml
deviation_waiver:
  id: uuid
  dw_number: string               # e.g., "DEV-2026-0007" or "WAI-2026-0003"
  document_type: enum             # Deviation | Waiver
  description: string             # What condition is being deviated/waived from
  part_id: uuid                   # FK → part.id
  part_specification_id: uuid     # FK → part_specification.id (the revision in scope)
  serial_id: uuid                 # FK → serial_number.id (serialized scope)
  lot_id: uuid                    # FK → material_lot.id (lot scope)
  quantity_affected: integer
  ncr_id: uuid                    # FK → nonconformance_report.id (for waivers; the NCR being waived)
  nonconformance_description: string
  proposed_disposition: string    # What will be done with the nonconforming article
  initiator_id: uuid              # FK → user.id
  initiated_date: date
  status: enum                    # Draft | Submitted | Approved | Rejected | Expired
  approved_by: string             # Customer rep name or DER number
  approved_date: date
  expiry_date: date               # Waivers/deviations are time- or quantity-limited
  expiry_quantity: integer        # Quantity limit on the deviation
  customer_id: uuid               # FK → customer.id
  notes: string
```

```yaml
insert_installation_record:
  id: uuid
  assembly_operation_id: uuid     # FK → assembly_operation.id (op with operation_code: INSERT)
  job_id: uuid                    # FK → job.id
  part_id: uuid                   # FK → part.id (the parent part receiving the insert)
  serial_id: uuid                 # FK → serial_number.id (if serialized)
  insert_part_id: uuid            # FK → part.id (the insert itself, part_kind: insert)
  insert_quantity: integer        # Number of inserts installed in this record
  hole_designation: string        # Which holes (e.g., "H1-H4", or drawing balloon reference)
  installation_torque_inlb: number # Torque applied during installation (in-lb)
  tang_breakoff_required: boolean # Whether the driving tang must be broken off after installation
  tang_broken_off: boolean        # Confirmation tang was removed (when required)
  installed_by_id: uuid           # FK → user.id
  installed_date: date
  inspected_by_id: uuid           # FK → user.id (quality inspection of installed inserts)
  result: enum                    # Pass | Fail | Rework
  notes: string
```

```yaml
press_fit_record:
  id: uuid
  assembly_operation_id: uuid     # FK → assembly_operation.id (op with operation_code: PRESS_FIT)
  job_id: uuid                    # FK → job.id
  part_id: uuid                   # FK → part.id (parent part receiving the pressed component)
  serial_id: uuid                 # FK → serial_number.id (if serialized)
  inserted_part_id: uuid          # FK → part.id (the pin, bushing, or bearing being pressed)
  insert_quantity: integer
  hole_designation: string
  press_force_max_lbf: number     # Maximum press force recorded (lbf)
  press_displacement_in: number   # Final displacement of pressed part (inches)
  force_within_spec: boolean      # True if press force within engineering-specified range
  installed_by_id: uuid           # FK → user.id
  installed_date: date
  inspected_by_id: uuid           # FK → user.id
  result: enum                    # Pass | Fail | Rework
  notes: string
```

```yaml
operator_qualification:
  id: uuid
  operator_id: uuid               # FK → user.id — the qualified operator
  qualification_type: enum        # Process | OperationCode | Equipment | Inspection
  process_specification_id: uuid  # FK → process_specification.id (for process qualifications)
  operation_code: string          # Operation code qualified for (e.g., "TORQUE", "NDT_FPI", "WELD")
  work_center_id: uuid            # FK → work_center.id (specific machine qualified for, if applicable)
  training_record_reference: string # Reference to external training record (e.g., LMS course ID)
  qualified_date: date
  expiry_date: date               # Qualification expiry (null = indefinite if not time-limited)
  qualified_by_id: uuid           # FK → user.id — the certifying authority
  status: enum                    # Active | Expired | Suspended | Revoked
  notes: string
```

- [ ] **Step 2: Write `schemas/change_management.yaml`**

Entities: `engineering_change_notice`, `deviation_waiver`. Include full attributes and relationships.

Relationships:
- `engineering_change_notice`: `initiated_by` (→ user, many-to-one), `approved_by` (→ user, many-to-one), `affects_part` (→ part, many-to-one), `supersedes_spec` (→ part_specification, many-to-one), `introduces_spec` (→ part_specification, many-to-one), `obsoletes_routing` (→ routing, many-to-one), `introduces_routing` (→ routing, many-to-one), `requires_customer_approval` (→ customer_approval, many-to-one)
- `deviation_waiver`: `for_part` (→ part, many-to-one), `for_spec` (→ part_specification, many-to-one), `for_serial` (→ serial_number, many-to-one), `for_lot` (→ material_lot, many-to-one), `references_ncr` (→ nonconformance_report, many-to-one), `initiated_by` (→ user, many-to-one), `for_customer` (→ customer, many-to-one)

- [ ] **Step 3: Write `schemas/assembly_hardware.yaml`**

Entities: `insert_installation_record`, `press_fit_record`. Include full attributes and relationships.

Relationships:
- `insert_installation_record`: `for_operation` (→ assembly_operation [routing.yaml], many-to-one), `for_job` (→ job [job.yaml], many-to-one), `for_part` (→ part, many-to-one), `for_serial` (→ serial_number [job.yaml], many-to-one), `uses_insert` (→ part, many-to-one), `installed_by` (→ user, many-to-one), `inspected_by` (→ user, many-to-one)
- `press_fit_record`: `for_operation` (→ assembly_operation [routing.yaml], many-to-one), `for_job` (→ job [job.yaml], many-to-one), `for_part` (→ part, many-to-one), `for_serial` (→ serial_number [job.yaml], many-to-one), `inserts_part` (→ part, many-to-one), `installed_by` (→ user, many-to-one), `inspected_by` (→ user, many-to-one)

- [ ] **Step 4: Write `schemas/operator_qualification.yaml`**

Entity: `operator_qualification`. Include full attributes and relationships.

Relationships: `for_operator` (→ user, many-to-one), `for_process` (→ process_specification, many-to-one), `at_work_center` (→ work_center, many-to-one), `certified_by` (→ user, many-to-one)

---

### Task 5: Missing Inspection-Calibration Prose

**Files:**
- Create: `extensions/inspection-calibration.md`

This file is already referenced by `schemas/inspection.yaml` (line 11: `# Prose source: extensions/inspection-calibration.md`) but does not exist. This task creates it with the appropriate prose backing.

**Content to write:**

```markdown
# Inspection, Calibration, and Measurement Traceability Extension

## The Gap This Fills
[paragraph on QIF gap: QIF defines measurement results and plans but not the calibration lifecycle of the instruments used. MTConnect CuttingTool asset schema covers tool identity. Neither provides a calibration audit record or an instrument lifecycle state machine. AS9100 §7.1.5.2 requires measurement traceability to international standards; the manufacturer must demonstrate that each inspection result was produced by a calibrated instrument at the time of measurement. This extension defines the `tool_gauge` (the calibrated instrument) and `calibration_record` (the immutable audit evidence) as distinct entities capturing that traceability chain.]

## Entity Definitions

### Inspection Event
[prose description of why a lightweight envelope is needed: without a shared parent FK, QIF result files, pmi_event records, and photo attachments from the same session have no way to be retrieved together. inspection_event is the session envelope, not the measurement data.]

### Calibration Record
[prose: immutable. One record per calibration event. Never modified. Lifecycle on tool_gauge, not here. AS9100 §7.1.5.2 requires traceable calibration certificate; this entity captures the certificate reference and as-found condition.]

### Tool Gauge
[prose: the physical instrument with mutable lifecycle state. The 5-state machine (Current → Due_Soon → Overdue → Quarantined → Retired) is documented here. Overdue instruments must not be used for production measurements — if used unknowingly, all inspection results since the last valid calibration must be evaluated for impact. The quarantine state triggers a calibration recall workflow.]

### Key Characteristic
[prose: design-defined attributes (ASME Y14.5, FAA AC 21-101). KC traceability to balloon number on drawing. QIF characteristic ID linkage. Subject may be part or assembly (polymorphic FK per Decision 1.9).]

## Calibration Recall Workflow
[prose walk: tool_gauge transitions Overdue. Query all inspection_events where the gauge was used since last_calibration_date. Review each pmi_event for impact. Open NCRs for any affected serial numbers. This is the recall scenario; full archetype walk in reference/composition-archetypes.md.]

## State Machine: tool_gauge.calibration_status
[document the 10 transitions with guards and triggers:
Current → Due_Soon (trigger: within 30 days of calibration_due_date)
Due_Soon → Overdue (trigger: past calibration_due_date)
Overdue → Quarantined (action: pulled from service)
Quarantined → Current (trigger: new calibration_record with result Pass or Adjusted)
Current → Quarantined (action: manual quarantine due to suspected damage)
Quarantined → Retired (action: calibration fails or permanent damage)
Any → Retired (action: end-of-life decision)]

## Decision Log
D-INSP-1: inspection_event is terminal — no approval gates, no status progression. Value is UUID grouping only.
D-INSP-2: calibration_record is immutable — append-only. Lifecycle on tool_gauge.calibration_status.
D-INSP-3: tool_gauge is both QIF instrument and MTConnect CuttingTool asset — bridged by mtconnect_tool_asset_id.
D-INSP-4: key_characteristic uses polymorphic FK (subject_id + subject_type) per Decision 1.9.
```

- [ ] **Step 1: Write full `extensions/inspection-calibration.md`** following the structure above, with complete prose for each section. Do not copy-paste from inspection.yaml comments — write prose explanations that a reader without the schema can understand.

---

## Phase 2 — Shared File Updates (SEQUENTIAL — run after Phase 1 complete)

All Phase 2 tasks modify files that multiple Phase 1 tasks reference. Run sequentially in the order listed.

---

### Task 6: Schema Modifications

**Files:**
- Modify: `schemas/part.yaml` — part_kind enum
- Modify: `schemas/material.yaml` — material_lot customer_furnished
- Modify: `schemas/assembly.yaml` — kit_record item_type enum

- [ ] **Step 1: Add `fixture`, `insert`, `pin`, `packaging_insert` to `part.yaml` `part_kind` enum**

Current values: `[machined, fastener, raw_stock, purchased_assembly, consumable]`

New values: `[machined, fastener, raw_stock, purchased_assembly, consumable, fixture, insert, pin, packaging_insert]`

Update the enum description to cover new values:
```yaml
part_kind:
  type: enum
  required: true
  values: [machined, fastener, raw_stock, purchased_assembly, consumable, fixture, insert, pin, packaging_insert]
  description: >
    Classification of the part type.
    machined = CNC/EDM/grind produced;
    fastener = bolt/nut/pin/rivet (purchased standard hardware);
    raw_stock = material in its as-received form;
    purchased_assembly = OTS assembly (e.g., bearing, gearbox);
    consumable = O-ring, shim, label, sealant;
    fixture = workholding fixture, jig, or soft jaw (serialized, tracked);
    insert = threaded insert (helicoil, keensert, nutplate) installed in a parent part;
    pin = interference-fit dowel or roll pin installed in a parent part;
    packaging_insert = 3D-printed or machined packaging form (tracked as a manufactured part)
```

- [ ] **Step 2: Add `customer_furnished` flag to `material_lot` in `schemas/material.yaml`**

Read `schemas/material.yaml` first. Find the `material_lot` entity attributes block. Add after the existing `lot_number` or similar field:

```yaml
      customer_furnished:
        type: boolean
        required: true
        description: >
          True if this material lot was supplied by the customer (GFM/CFM — Government/Customer
          Furnished Material). Customer-furnished lots are traceable to the customer's material
          certification; the manufacturer does not perform incoming material inspection beyond
          verifying identity and quantity. False for manufacturer-purchased material.
```

- [ ] **Step 3: Add `insert`, `pin` to `kit_record.kit_contents` `item_type` enum in `schemas/assembly.yaml`**

Read `schemas/assembly.yaml` first. Find `kit_record` entity and the `kit_contents` attribute (an array with `item_type` discriminator). Current values include `[part, fastener, consumable, material]`. Add `insert` and `pin`:

```yaml
      item_type: enum
      values: [part, fastener, consumable, material, insert, pin]
```

---

### Task 7: INDEX.md and Entity Inventory Updates

**Files:**
- Modify: `INDEX.md`
- Modify: `reference/entity-inventory.yaml`
- Modify: `reference/domain-map.md`

- [ ] **Step 1: Fix INDEX.md — 4 existing inspection entities missing schema reference**

Read `INDEX.md` to find entries for `calibration_record`, `tool_gauge`, `inspection_event`, `key_characteristic`. These currently show `*(none)*` for schema. Update each to reference `schemas/inspection.yaml`.

- [ ] **Step 2: Add all Phase 1 new entities to INDEX.md**

For each new entity, add an alphabetically-sorted entry with:
- Entity name (h3 or table row matching existing format)
- Domain: Process Engineering (9), Tool Room (10), Packaging (11), Change Management (12), Assembly Hardware, Operator Qualification
- Schema: the appropriate `schemas/*.yaml` file
- Prose: the appropriate `extensions/*.md` file

New entities to add (35 total):

Domain 9 — Process Engineering:
`cam_file`, `cmm_program`, `fixture_use`, `inspection_plan`, `nc_program`, `nre_package`, `print_file`, `prove_out_record`, `setup_sheet`, `tooling_list`, `tooling_list_line`

Domain 10 — Tool Room:
`cutter_definition`, `gauge_definition`, `holder_definition`, `tool_assembly_definition`, `tool_life_record`, `tool_maintenance_event`, `tool_pocket_assignment`, `tool_pocket_requirement`, `tool_preset`, `tool_room_allocation`

Domain 11 — Packaging:
`packaging_item`, `packaging_specification`, `packing_record`

Domain 12 — Change Management:
`deviation_waiver`, `engineering_change_notice`

Assembly Hardware:
`insert_installation_record`, `press_fit_record`

Operator Qualification:
`operator_qualification`

- [ ] **Step 3: Update `reference/entity-inventory.yaml`**

Read the current `entity-inventory.yaml`. For each new entity, add a record following the existing format (id, name, domain, schema_file, description). Update the entity count in any header comment.

- [ ] **Step 4: Update `reference/domain-map.md` — add Domains 9–12**

Read `reference/domain-map.md`. Add four new domain sections after the existing 8 domains:

```markdown
## Domain 9: Process Engineering / NRE

Covers the engineering and prove-out work required to manufacture a part for the first time or after a significant revision: NC programming, CAM work, fixture design, CMM programming, inspection planning, and the prove-out record that certifies the process is ready for production.

Entities: nre_package, nc_program, cam_file, setup_sheet, cmm_program, inspection_plan, prove_out_record, print_file, tooling_list, tooling_list_line, fixture_use

Schema: schemas/process_engineering.yaml
Extension prose: extensions/process-engineering-nre.md

## Domain 10: Tool Room

Covers the definition, procurement, preparation, allocation, and lifecycle management of cutting tools, workholding fixtures, and measurement gauges. Bridges NC program tool requirements to the physical tool room inventory.

Entities: cutter_definition, holder_definition, tool_assembly_definition, gauge_definition, tool_preset, tool_pocket_requirement, tool_pocket_assignment, tool_room_allocation, tool_maintenance_event, tool_life_record
Note: tool_gauge (serialized calibrated instrument) is defined in Domain 7 (Quality / Inspection) at schemas/inspection.yaml; Tool Room entities reference it.

Schema: schemas/tool_room.yaml
Extension prose: extensions/tool-room.md

## Domain 11: Packaging

Covers packaging specification authoring and packaging execution evidence. Treats packaging as a traceable manufacturing operation per AS9100 §8.5.4. Includes support for MIL-STD-2073 preservation levels and 3D-printed packaging inserts as manufactured parts.

Entities: packaging_specification, packing_record, packaging_item
Note: packaging_insert parts (physical packaging forms) are part entities with part_kind: packaging_insert (Domain 2).

Schema: schemas/packaging.yaml
Extension prose: extensions/packaging.md

## Domain 12: Change Management

Covers the documentation and authorization of engineering changes (ECNs) and deviation/waiver requests. ECNs authorize revision changes; deviations/waivers authorize use of nonconforming material with customer or DER approval.

Entities: engineering_change_notice, deviation_waiver
Note: nonconformance_report (internal quality record) is in Domain 7 (Quality). deviation_waiver references nonconformance_report for post-production waivers.

Schema: schemas/change_management.yaml
Extension prose: extensions/change-management.md

## Assembly Hardware (Domain 7 extension)

insert_installation_record and press_fit_record are execution records for assembly operations involving threaded inserts and press-fit components. Defined in schemas/assembly_hardware.yaml; conceptually part of Domain 7 (Quality/Assembly).

## Operator Qualification (Domain 7 extension)

operator_qualification records training and certification currency for operators performing special processes. Defined in schemas/operator_qualification.yaml; conceptually part of Domain 7.
```

---

## Phase 3 — Validation (SEQUENTIAL — run after Phase 2 complete)

### Task 8: Four Scenario Archetype Walks

**Files:**
- Modify: `reference/composition-archetypes.md`

Add four new archetype walks as new sections in the existing file.

- [ ] **Step 1: Read `reference/composition-archetypes.md`** to understand existing walk format and append point.

- [ ] **Step 2: Write Archetype Walk 4 — Calibration Recall**

Scenario: An inspector's ring gauge (tool_gauge, calibration_status: Current) is discovered to be physically damaged mid-shift. Quality quarantines it. Process to identify impact on all inspection results taken with this gauge since its last calibration.

Walk must trace: tool_gauge (quarantine) → calibration_record (query: last pass date) → inspection_event (query: gauge used between last_cal and quarantine date) → pmi_event (each measurement result) → serial_number or material_lot (which units are affected) → nonconformance_report (open NCRs for each) → customer notification if source-inspection-required parts affected.

Entity touchpoints: tool_gauge, calibration_record, inspection_event, pmi_event, serial_number, material_lot, nonconformance_report, customer_purchase_order (for notification requirement).

- [ ] **Step 3: Write Archetype Walk 5 — ECN Mid-Production**

Scenario: ECN-2026-0042 is approved for part 7832-001: drawing revision changes from Rev B to Rev C, routing routing_revision from B to C. At the time of approval, 3 active jobs are running against Rev B (quantities partially complete).

Walk must trace: engineering_change_notice (approved) → part_specification (new Rev C created) → routing (new routing_rev C created, old Rev B set to Obsolete) → nre_package (new NRE package required for Rev C) → active jobs (status: in-progress against Rev B) → disposition decision (complete under Rev B? stop and re-route? deviation_waiver for partially-complete pieces?) → if waiver chosen: deviation_waiver created linking Rev B jobs → new job creation for remaining quantity against Rev C routing.

Entity touchpoints: engineering_change_notice, part_specification, routing, nre_package, job, work_order, deviation_waiver, customer_purchase_order (if customer approval required).

- [ ] **Step 4: Write Archetype Walk 6 — Fixture Full Lifecycle**

Scenario: Engineering designs a new fixture for part 7832-001 Op 10. The fixture goes from design (part record) through manufacturing (job with routing) through qualification (prove_out_record) through active production use (tool_room_allocation + fixture_use) to a maintenance event to eventual retirement.

Walk must trace: part (part_kind: fixture, new P/N) → part_specification (fixture drawing) → routing (fixture manufacturing routing) → job (build the fixture) → work_order execution → inspection_event (fixture qualification inspection) → prove_out_record (first part machined with fixture: pass) → fixture_use (fixture_part_id ← operation_id for all future ops) → tool_room_allocation (checkout for each job) → tool_maintenance_event (wear inspection) → eventual part retirement (active = false).

Entity touchpoints: part (fixture), part_specification, routing, job, work_order, inspection_event, prove_out_record, fixture_use, tool_room_allocation, tool_maintenance_event.

- [ ] **Step 5: Write Archetype Walk 7 — Concurrent Revision Production**

Scenario: Customer has open orders for both 7832-001 Rev B (legacy order from 6 months ago, partially shipped) and 7832-001 Rev C (new order). Both revisions are in production simultaneously. The shop must maintain separate routings, NRE packages, and serialization for each revision.

Walk must trace: part (7832-001 Rev B, fai_status: Approved) + part (7832-001 Rev C, fai_status: In_Progress) → part_specification (two records, same drawing_number different revision) → routing (two Released routings — one per revision) → nre_package (two packages) → job Rev B (against routing Rev B) + job Rev C (against routing Rev C) → serial_number tracking (Rev B serials SN-001–SN-020, Rev C starts SN-021) → inspection_event per serial → FAI for Rev C: certification_package.includes_fai = true → separate shipments, separate certification packages.

Entity touchpoints: part (two records), part_specification (two records), routing (two records), nre_package (two records), job (two jobs), serial_number, inspection_event, certification_package, shipment, customer_purchase_order (two orders, one per rev).

---

### Task 9: Consistency Check and Entity Matrix Update

**Files:**
- Modify: `reference/entity-matrix.md`

- [ ] **Step 1: Add entity-matrix rows for all 35 new entities**

For each new entity, add a matrix section with all dimensions:
- Schema defined: ✅ (if Task 1–4 complete) or 🔴
- Attributes complete: ✅ (assess after writing)
- Relationships typed: ✅ or 🟡
- State machine: ✅ or ⬜ (N/A for non-lifecycle entities)
- Cross-ref integrity: 🟡 until added to UUID table (do in this task)
- Standards provenance: document which Layer-1 standard (or ⬜ for pure Layer-2)
- Composition rule: document primitive/envelope/lifecycle/hybrid
- Canonical framing: ✅ if name follows conventions

Lifecycle-bearing entities (need state machine documentation):
- `nre_package`: Draft | InReview | Approved | Superseded
- `nc_program`: Draft | Proven | Released | Obsolete
- `operator_qualification`: Active | Expired | Suspended | Revoked
- `engineering_change_notice`: Draft | UnderReview | Approved | Rejected | Cancelled | Released
- `deviation_waiver`: Draft | Submitted | Approved | Rejected | Expired
- `tool_room_allocation`: CheckedOut | Returned | Lost | Damaged
- `prove_out_record`: immutable (⬜)
- `insert_installation_record`: immutable (⬜)
- `press_fit_record`: immutable (⬜)

- [ ] **Step 2: Update `extensions/uuid-discipline.md` — add new entities to UUID entity table**

Read uuid-discipline.md. Add all 35 new entities to the entity table with: entity name, schema file, UUID discipline notes (when minted, what it represents across system boundaries).

- [ ] **Step 3: Cross-reference scan**

For each new schema file, verify:
1. Every FK target entity exists in some schema file (or is documented as external reference like `machine_event`)
2. Every `one-to-many` relationship has a corresponding `many-to-one` on the other side
3. No entity name defined in two places
4. `tool_gauge` is NOT re-defined in tool_room.yaml (should only reference it)
5. `nc_program` FK in tool_room.yaml is annotated `# defined in schemas/process_engineering.yaml`

Report any violations and fix inline.

- [ ] **Step 4: Update entity-matrix.md header — update Phase note and entity count**

Add a Phase 2 update note at the top of entity-matrix.md:
```
Phase 2 update (2026-04-23): Added Domains 9–12 (Process Engineering, Tool Room, Packaging, Change Management) plus Assembly Hardware and Operator Qualification cross-domain extensions. Entity count increased from 82 to ~117. Four new composition archetype walks added (calibration recall, ECN mid-production, fixture lifecycle, concurrent revision production).
```

---

## Self-Review

**Spec coverage check:**
- ✅ nc_program, cam_file, setup_sheet, cmm_program, inspection_plan, nre_package, prove_out_record, print_file, tooling_list, tooling_list_line, fixture_use → Task 1
- ✅ cutter_definition, holder_definition, tool_assembly_definition, gauge_definition, tool_preset, tool_pocket_requirement, tool_pocket_assignment, tool_room_allocation, tool_maintenance_event, tool_life_record → Task 2
- ✅ packaging_specification, packing_record, packaging_item → Task 3
- ✅ engineering_change_notice, deviation_waiver, insert_installation_record, press_fit_record, operator_qualification → Task 4
- ✅ extensions/inspection-calibration.md (missing prose) → Task 5
- ✅ part_kind enum additions (fixture, insert, pin, packaging_insert) → Task 6
- ✅ material_lot.customer_furnished → Task 6
- ✅ kit_record item_type additions (insert, pin) → Task 6
- ✅ INDEX.md fix for 4 inspection entities → Task 7
- ✅ INDEX.md additions for all new entities → Task 7
- ✅ 4 scenario walks (calibration recall, ECN mid-production, fixture lifecycle, concurrent revision) → Task 8
- ✅ entity-matrix + UUID table updates → Task 9

**Fixture entity model:** Fixtures are part with part_kind: fixture — confirmed consistent across Tasks 1, 2, 6. No separate fixture_instance entity.

**tool_gauge:** Defined only in schemas/inspection.yaml. Tasks 2 references it via FK annotation. gauge_definition (new catalog entity) defined in Tool Room. No duplication.

**holder_instance / cutter_instance:** Not separate entities. Holders are tracked as parts (part_kind: fixture or a future tooling-specific kind if warranted). Cutters are tracked via tool_life_record (consumable, not individually serialized by default). Consistent with part_kind: fixture pattern already established.

---

## Execution Handoff

Plan complete and saved to `docs/superpowers/plans/2026-04-23-ontology-phase-2-domains.md`.

**Two execution options:**

**1. Subagent-Driven (recommended)** — Dispatch one subagent per Phase 1 task (Tasks 1–5) in parallel, review output, then run Phase 2–3 tasks sequentially in this session or as additional subagents.

**2. Inline Execution** — Execute tasks sequentially in this session using executing-plans skill with checkpoints after each Phase.
