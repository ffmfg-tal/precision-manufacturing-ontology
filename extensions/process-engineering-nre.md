# Process Engineering NRE Extension

## The Gap This Fills

Process engineering for a CNC machining operation produces a specific set of documents before a job can run: NC programs, CAM files, setup sheets, CMM programs, inspection plans, prove-out records, tooling lists, and a controlled copy of the drawing. These documents are interdependent — they all describe the same part revision at the same routing revision — but the standards stack treats them as unrelated artifacts or ignores them entirely.

**STEP AP238** (ISO 10303-238) covers process plan structure for CNC operations. It has no concept of the NC program file itself, the prove-out event, or the relationship between a CAM project and the machine-specific programs it posts. The standard defines *what* operations to perform; it has no model for the engineering artifacts that enable a machine to actually perform them.

**STEP AP242** covers Product and Manufacturing Information (PMI) and key characteristics. It does not model the inspection plan as an ordered sequence of measurement steps, and it has no concept of which gauge type or CMM program executes each measurement.

**QIF** (Quality Information Framework) covers inspection plans and measurement results in detail — but only the inspection side. QIF has no concept of CAM files, NC programs, or the CAM-to-NC posting relationship.

**None of these standards** have an NRE package as an organizing entity: a single envelope that locks together the drawing revision, routing revision, and all associated engineering documents such that a change to either revision forces a new package.

The result is a documentation gap at the center of the manufacturing digital thread. NC programs float unlinked to the drawing revision that drove them. Prove-out events are informal records — or no record at all. CAM files exist in shared drives with no traceable connection to which machine programs they produced or which routing operation they serve. When a drawing changes, it is unclear which engineering documents are now stale.

This extension defines the **NRE Package** as the revision-locking envelope and the ten supporting entities that constitute its contents: NC programs, CAM files, setup sheets, CMM programs, inspection plans, prove-out records, print files, tooling lists, tooling list lines, and fixture use records.

---

## Entity Definitions

### NRE Package

The parent envelope that locks together all process engineering documents for a specific drawing revision at a specific routing revision. When either the drawing revision (`part_specification_id`) or routing revision (`routing_id`) changes, a new `nre_package` is required — the existing package does not carry forward. This is the revision-lock invariant: the package is a point-in-time snapshot of "how we make this drawing at this routing."

```yaml
nre_package:
  id: uuid                          # UUID v4 — minted when the NRE package record is created
  part_id: uuid                     # FK → part.id
  part_specification_id: uuid       # FK → part_specification.id (drawing revision locked)
  routing_id: uuid                  # FK → routing.id (routing revision locked)
  status: enum                      # Draft | InReview | Approved | Superseded
  approved_by_id: uuid              # FK → user.id (engineer or quality approver)
  approved_date: date
  notes: string
```

**Revision-lock invariant:** `part_specification_id` and `routing_id` must remain stable for the life of a package. A change to either — new drawing revision, new routing revision — requires creating a new `nre_package` with status `Draft` and superseding the prior package. The prior package transitions to `Superseded` and is retained for traceability.

---

### NC Program

The machine-specific CNC program file for one operation at one work center. The natural key is `(operation_id, work_center_id)` — the same operation may have multiple NC programs if it can run on different machines (e.g., a 5-axis mill operation with a Mazak variant and a DMG variant). Each NC program is a distinct controlled artifact with its own revision and status lifecycle.

```yaml
nc_program:
  id: uuid                          # UUID v4
  operation_id: uuid                # FK → operation.id
  work_center_id: uuid              # FK → work_center.id (the specific machine this program runs on)
  nre_package_id: uuid              # FK → nre_package.id
  program_number: string            # e.g. "7832-001-OP10-MAZAK1"
  revision: string                  # e.g. "A", "B", "1"
  cam_file_id: uuid                 # FK → cam_file.id (source CAM file; optional if program was hand-coded)
  file_reference: string            # Path or URL to the .nc file
  status: enum                      # Draft | Proven | Released | Obsolete
  notes: string
```

**Status lifecycle:** `Draft` → `Proven` (first-piece accepted; see `prove_out_record`) → `Released` (authorized for production). `Obsolete` programs are superseded by a new revision. Only `Released` programs may be issued to the floor for production jobs.

---

### CAM File

The CAM project file (Mastercam, Fusion, Esprit, etc.) that serves as the engineering source for one or more NC programs. A single CAM file posts to multiple `nc_program` records when the same machining strategy is adapted for different machines — the CAM geometry and toolpaths are identical, but the post-processor output differs per work center. This is the CAM-to-NC one-to-many relationship: one CAM file, many machine programs.

```yaml
cam_file:
  id: uuid                          # UUID v4
  nre_package_id: uuid              # FK → nre_package.id
  operation_id: uuid                # FK → operation.id
  file_reference: string            # Path or URL to CAM project file (e.g., .mcam, .f3d, .mfg)
  cad_model_reference: string       # Path or URL to 3D model used as input (STEP/Parasolid/etc.)
  cam_system: string                # e.g. "Mastercam 2025", "Fusion 360", "Esprit"
  revision: string
  created_by_id: uuid               # FK → user.id
  created_date: date
  notes: string
```

---

### Setup Sheet

The operator setup instruction for one operation at one work center — typically a PDF or structured document showing fixturing orientation, work offsets, tool list, and any critical setup notes. The setup sheet is work-center-specific because fixturing, machine coordinate systems, and pallet configurations differ between machines even for the same operation.

```yaml
setup_sheet:
  id: uuid                          # UUID v4
  operation_id: uuid                # FK → operation.id
  work_center_id: uuid              # FK → work_center.id
  nre_package_id: uuid              # FK → nre_package.id
  revision: string
  file_reference: string            # Path or URL to PDF setup sheet
  fixture_part_id: uuid             # FK → part.id (primary fixture shown; part_kind: fixture)
  notes: string
```

---

### CMM Program

The coordinate measuring machine program for an inspection operation. CMM programs are machine-specific (PC-DMIS for a Zeiss, Calypso for another CMM) and are tied to a specific CMM work center. A CMM program is not an inspection plan — the plan specifies *what* to measure and *in what order*; the CMM program is the executable code that measures it on a specific machine.

```yaml
cmm_program:
  id: uuid                          # UUID v4
  operation_id: uuid                # FK → operation.id (the inspection op, e.g., Op 50 CMM)
  work_center_id: uuid              # FK → work_center.id (the CMM machine)
  nre_package_id: uuid              # FK → nre_package.id
  program_name: string              # e.g. "7832-001-FAI-CMM.mpx"
  revision: string
  cmm_system: string                # e.g. "PC-DMIS 2023 R2", "Calypso 2022"
  file_reference: string            # Path or URL to CMM program file
  qif_template_reference: string    # Optional path to QIF plan file used to generate the program
  notes: string
```

---

### Inspection Plan

The pre-defined plan specifying which key characteristics to check, in what order, and with what gauge type or method. Unlike a CMM program (which is machine-executable code), the inspection plan is method-agnostic: it defines the measurement sequence that could be executed by a CMM, a manual gauge, or a vision system. The `plan_type` enum distinguishes FAI (first article, per AS9102) from routine in-process, receiving, and final inspection plans.

```yaml
inspection_plan:
  id: uuid                          # UUID v4
  part_id: uuid                     # FK → part.id
  nre_package_id: uuid              # FK → nre_package.id
  revision: string
  plan_type: enum                   # FAI | InProcess | Receiving | Final
  file_reference: string            # Path or URL to inspection plan document
  qif_plan_reference: string        # Optional path or URL to QIF plan file
  notes: string
```

**plan_type values:**
- `FAI` — First Article Inspection per AS9102 Rev C; required for new part numbers and engineering changes
- `InProcess` — Checks performed at intermediate operations before the part is complete
- `Receiving` — Incoming inspection of raw material or purchased parts
- `Final` — Dimensional and visual inspection at shipment

---

### Prove-Out Record

Immutable evidence that an NC program was physically proven on a specific machine. The prove-out event captures who ran it, when, on which machine, and the result of the first-piece inspection. Once created, prove-out records are not edited — a subsequent prove-out creates a new record. Engineering or quality must sign off before the associated NC program can advance from `Proven` to `Released`.

```yaml
prove_out_record:
  id: uuid                          # UUID v4
  nc_program_id: uuid               # FK → nc_program.id
  work_center_id: uuid              # FK → work_center.id (machine actually used)
  operator_id: uuid                 # FK → user.id
  prove_out_date: date
  first_piece_result: enum          # Pass | Fail | Conditional
  approved_by_id: uuid              # FK → user.id (engineer or quality sign-off)
  approved_date: date
  file_reference: string            # Path or URL to first-piece inspection sheet
  notes: string
```

**first_piece_result values:**
- `Pass` — All dimensions within tolerance; program released as-is
- `Fail` — One or more dimensions out of tolerance; program returns to Draft for rework
- `Conditional` — Dimensions acceptable with documented deviation or offset adjustment; requires engineering disposition

---

### Print File

The controlled copy of the drawing PDF packaged with the NRE set. This is distinct from `part_specification.pdf_file_ref`, which is the master drawing record. The print file is the specific revision of the drawing that was current when the NRE package was approved — it travels with the NRE set and makes the drawing revision visible without navigating to the master record. The `distribution_statement` must match `part_specification.distribution_statement` for the same revision.

```yaml
print_file:
  id: uuid                          # UUID v4
  nre_package_id: uuid              # FK → nre_package.id
  part_specification_id: uuid       # FK → part_specification.id
  file_reference: string            # Controlled copy of the drawing PDF
  revision: string                  # Matches part_specification.revision
  distribution_statement: enum      # Unlimited | Export_Controlled_EAR | ITAR_Controlled |
                                    # CUI | FOUO | Proprietary
  notes: string
```

---

### Tooling List

The complete list of tools required to run a specific NC program at a specific work center. A tooling list is tied to an NC program (not just an operation) because tool selections are machine-specific — different machines may accept different holder styles, collet sizes, or tool lengths. The tooling list is the authoritative reference for pre-job tool staging and crib pull.

```yaml
tooling_list:
  id: uuid                          # UUID v4
  operation_id: uuid                # FK → operation.id
  nc_program_id: uuid               # FK → nc_program.id (optional; list may predate program assignment)
  nre_package_id: uuid              # FK → nre_package.id
  revision: string
  notes: string
```

---

### Tooling List Line

One line in a tooling list. Each line specifies the tool pocket number as used in the NC program (T#1, T#12, etc.) and references the tool assembly definition from the tool room schema. The `description` field provides a human-readable summary for the operator.

```yaml
tooling_list_line:
  id: uuid                          # UUID v4
  tooling_list_id: uuid             # FK → tooling_list.id
  pocket_number: integer            # Tool pocket number in NC program (e.g., 1, 12 for T#1, T#12)
  tool_assembly_definition_id: uuid # FK → tool_assembly_definition.id
                                    # (defined in schemas/tool_room.yaml — forward reference;
                                    #  tool_room.yaml not yet authored)
  description: string               # e.g. "T#3 — 0.500 chamfer mill, ER20 holder, 3.125 stickout"
  sequence_position: integer        # Order in tool list (optional; for display/staging order)
  notes: string
```

---

### Fixture Use

A join entity linking a routing operation to a fixture part. Fixtures are modeled as `part` entities with `part_kind: fixture` — they are not a separate entity type. One operation may use multiple fixtures (primary workholding, secondary clamp, locating pin plate, etc.). The `required` flag distinguishes mandatory fixtures from alternatives that exist when the primary is unavailable.

```yaml
fixture_use:
  id: uuid                          # UUID v4
  operation_id: uuid                # FK → operation.id
  fixture_part_id: uuid             # FK → part.id (where part_kind = fixture)
  description: string               # e.g. "primary workholding", "secondary clamp", "locating pin plate"
  required: boolean                 # True = mandatory; False = alternative exists
  notes: string
```

---

## Decision Log

**Decision 1 — nre_package as revision-lock envelope.**
The NRE package enforces a hard invariant: `part_specification_id` and `routing_id` are immutable after creation. When either changes, a new package is required and the prior is superseded. This was preferred over a softer "version bump" model because it makes drawing-routing currency checkable by query: any nc_program or setup_sheet whose nre_package references a `Superseded` package is by definition stale. The envelope also provides the organizing anchor for the entire NRE document set, which would otherwise be a flat list of files with no shared parent.

**Decision 2 — nc_program natural key is (operation_id, work_center_id).**
An NC program is per-operation and per-machine. The same operation may run on two machines (machine flex), producing two distinct NC programs from the same CAM file. Modeling the natural key this way makes machine-substitution visible as a data structure rather than a naming convention. It also means adding a flex machine for an existing operation is additive — a new nc_program record — not a mutation of the existing one.

**Decision 3 — cam_file → nc_program is one-to-many.**
One CAM project posts to multiple NC programs for different work centers (e.g., Mazak 5-axis vs. DMG 5-axis). This captures the real workflow: the programmer authors one CAM file and runs different post-processors to generate machine-specific .nc files. Without this edge, the CAM file and the NC programs are unlinked and the re-use relationship is invisible. The FK on `nc_program.cam_file_id` is optional to accommodate hand-coded programs that have no CAM source.

**Decision 4 — fixtures are parts, not a separate entity class.**
Fixtures are tracked in the part master with `part_kind: fixture`. They have drawings, revision histories, material specs, and are manufactured (often in-house or sourced). Creating a separate `fixture` entity type would duplicate the identity, revision, and lifecycle infrastructure already present in `part`. The `fixture_use` join entity provides the operation-to-fixture linkage without requiring a new top-level entity.

**Decision 5 — print_file is distinct from part_specification.pdf_file_ref.**
`part_specification.pdf_file_ref` is the master drawing record — the authoritative source that may be updated as part of a revision workflow. The `print_file` in the NRE set is a controlled snapshot: the specific PDF that was current at NRE package approval. This distinction matters for ITAR and export-controlled drawings, where the distribution statement must be checked at the time of use. Keeping them as separate entities makes the snapshot relationship explicit and auditable.

**Decision 6 — inspection_plan and cmm_program are separate entities.**
The inspection plan is method-agnostic: it specifies key characteristics, sequence, and gauge type, and is the governing document regardless of how measurements are taken. The CMM program is machine-executable code tied to a specific CMM. Merging them would conflate a quality governance document with a machine program, making the plan unusable for manual inspection operations and hiding the fact that the same inspection plan might have multiple CMM program implementations (different CMMs, different software versions). QIF models this same separation: a QIF plan is not a CMM program.

**Decision 7 — prove_out_record is immutable evidence.**
Prove-out records are not edited after creation. A failed prove-out followed by a rework cycle produces two records: the `Fail` record (permanent) and a subsequent `Pass` record. This creates an auditable prove-out history rather than an overwritten status field. The `first_piece_result: Conditional` value accommodates the common case where dimensions are acceptable but an offset adjustment or engineering disposition is required — the record captures the disposition without falsely declaring a clean `Pass`.

**Decision 8 — tooling_list ties to nc_program, not just operation.**
Tool selection is machine-specific. A Mazak may require BT50 holders while a DMG uses HSK-A63. The same operation run on two machines needs two tooling lists. Tying the tooling list to the nc_program (which already carries the work_center_id) makes this specificity explicit without adding redundant fields. The `nc_program_id` is optional because tooling lists may be authored before the NC program is complete.

**Decision 9 — tool_assembly_definition is a forward reference to schemas/tool_room.yaml.**
The tool room schema (tool assemblies, holder types, insert grades, preset records) is a coherent domain of its own. `tooling_list_line.tool_assembly_definition_id` references it by FK rather than inlining tool definitions here. The cross-reference is annotated in the YAML; `schemas/tool_room.yaml` has not yet been authored.
