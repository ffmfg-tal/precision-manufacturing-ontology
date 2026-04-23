# the production system ↔ QIF Integration

## Integration Goal

QIF (Quality Information Framework, ISO 23952) is the standard format for CMM measurement plans and results. the production system manages the job, the inspection event, and the cert package. This integration links them: the production system generates the measurement plan (as a QIF Plan), the CMM operator runs it and exports results (as a QIF Results file), and the production system ingests the results to update the job's inspection record and trigger cert package eligibility.

The integration also provides the data that populates FAI Form 3 (Characteristic Accountability per AS9102) — the dimensional measurement evidence required for First Article Inspection.

---

## Data Flow

```
FULCRUM PRO                               CMM SOFTWARE
    |                                          |
    | Job 25-0412, Op 60 (CMM Inspect)         |
    | Part: 12345-1 Rev C                      |
    | Serial: 7832-007                         |
    |                                          |
    | Generate QIF Plan                        |
    |   Plan UUID: a3f7c2d1-...               |
    |   JobId (ext): f91c0d6b-...            |
    |   SerialNumber (ext): a3f7c2d1-...     |
    |   Characteristics: 47 from drawing      |
    |                                          |
    |--- QIF Plan file output ---------------->|
    |                                          |
    |   CMM operator loads plan                |
    |   CMM runs measurement cycle             |
    |   47 characteristics measured            |
    |                                          |
    |<-- QIF Results file ------------------- |
    |    Plan UUID matched                     |
    |    JobId, SerialNumber present           |
    |    Per-characteristic: measured,         |
    |    deviation, pass/fail                  |
    |                                          |
    | Parse QIF Results                        |
    | Link to the production system inspection record        |
    | Update each characteristic: pass/fail   |
    | If all pass: cert package eligible       |
    | If any fail: flag for NCR creation       |
    | FAI Form 3: populated from results       |
```

---

## QIF Document Header (Manufacturer Extension)

The base QIF standard does not include job or serial number references. the manufacturer extends the QIF document header to carry the production system UUIDs:

```xml
<QIFDocument xmlns="http://qifstandards.org/xsd/qif3"
             xmlns:ffm="http://ffm.io/qif-ext/1.0">

  <Header>
    <!-- Standard QIF fields -->
    <QPId>a3f7c2d1-8b4e-4a2f-9e1c-0d6b5a3f8c91</QPId>  <!-- Plan UUID -->
    <Author>
      <Name>Manufacturer Inspection System</Name>
      <Organization>Precision Manufacturer</Organization>
    </Author>
    <Created>2025-04-15T10:00:00Z</Created>

    <!-- extension fields -->
    <ffm:FulcrumJobId>f91c0d6b-5a3f-8c91-a3f7-c2d18b4e4a2f</ffm:FulcrumJobId>
    <ffm:FulcrumSerialId>a3f7c2d1-8b4e-4a2f-9e1c-0d6b5a3f8c91</ffm:FulcrumSerialId>
    <ffm:FulcrumOperationId>e4a2f9e1-c0d6-b5a3-f8c9-1a3f7c2d18b4</ffm:FulcrumOperationId>
    <ffm:PartNumber>12345-1</ffm:PartNumber>
    <ffm:Revision>C</ffm:Revision>
    <ffm:IsFAIUnit>true</ffm:IsFAIUnit>  <!-- Whether this is the FAI article -->
  </Header>

  ...

</QIFDocument>
```

These extension fields are the UUID anchors that allow the QIF ingestion process to locate the correct production job, serial number, and operation without manual matching.

---

## QIF Plan Generation: the production system → CMM

### From 2D Drawing (Manual Entry)

For parts where the manufacturer receives a 2D PDF drawing (the common case), characteristics are manually entered into Fulcrum's inspection plan builder:

1. Engineer or quality technician enters each characteristic from the drawing into the production system
2. the production system assigns a UUID to each characteristic at entry time
3. the production system generates a QIF Plan file with those characteristic definitions and UUIDs
4. CMM operator loads the QIF Plan into CMM software
5. CMM software populates results against the same characteristic UUIDs

This is manual for the entry step but fully machine-tracked once entered.

### From STEP AP242 PMI (When Available)

For customers who supply STEP AP242 MBD packages (Phase 3):

1. STEP reader extracts GD&T characteristics from PMI annotations
2. Each characteristic is assigned a UUID at extraction time
3. Characteristics are imported into the production system inspection plan
4. QIF Plan is generated from the production system record (same as manual flow, but entry is automated)

The UUID assigned to a characteristic at STEP extraction time is the same UUID used in the QIF Plan and QIF Results — creating a direct link from design intent (STEP AP242) to measured result (QIF).

---

## QIF Results Ingestion: CMM → the production system

When the CMM operator completes a measurement run and exports a QIF Results file, the ingestion pipeline:

### Parse and Validate

1. Read QIF Results file
2. Extract `ffm:FulcrumJobId` and `ffm:FulcrumSerialId` from header
3. Locate matching production job and serial number records
4. Validate: plan UUID in Results matches the plan UUID used for this inspection event
5. Validate: measurement date is within the job's active inspection window

### Map Results to Characteristics

For each characteristic in the QIF Results:
- Match to the production system characteristic record by UUID
- Store: `measured_value`, `deviation`, `upper_tolerance`, `lower_tolerance`, `pass_fail`
- Store: `measuring_equipment`, `operator`, `measurement_date`
- Flag: if `pass_fail = Fail`, mark characteristic for NCR evaluation

### Update Inspection Status

- If all characteristics pass: inspection event status → `Complete/Pass`; cert package eligibility flag set
- If any characteristic fails: inspection event status → `Complete/Fail`; NCR creation flag raised
- If any characteristics are not measured: inspection event status → `Incomplete`

### FAI Form 3 Population

If `IsFAIUnit = true` in the QIF header:
- QIF Results data feeds directly into `first_article_inspection.form3.characteristics[]`
- Pass/fail status, deviation, equipment used, date — all populated from QIF
- Form 3 is considered complete when all drawing-required characteristics have results
- Any `pass_fail = Fail` result holds FAI Form 3 until NCR disposition is complete (use-as-is or rework + re-measure)

---

## CMM Software Configuration

Common CMM options and their QIF export capability:

| CMM Software | QIF Export | Version Required | Notes |
|-------------|------------|-----------------|-------|
| PC-DMIS (Hexagon) | Native QIF 3.0 | 2022 R2+ | QIF export in Reports module |
| CALYPSO (Zeiss) | Native QIF 3.0 | 2021+ | QIF export settings under Output |
| MODUS (Renishaw) | QIF 3.0 via plugin | Latest | Check Renishaw update channel |
| CAMIO (Hexagon) | QIF 3.0 | 2022+ | QIF export via reporting config |

**Configuration requirements for each software:**
- Enable QIF output format in reporting settings
- Configure output path (shared folder or folder watch directory)
- Enable header extension fields (manufacturer namespace) — requires custom header template configuration
- Test: verify exported XML contains `ffm:FulcrumJobId` and characteristic UUIDs

---

## Inspection Plan Characteristic Types

QIF handles these characteristic types, all of which appear in aerospace part inspection:

| QIF Type | Drawing Callout | Example |
|----------|----------------|---------|
| `LinearCharacteristic` | Linear dimension, diameter, radius | Ø 25.400 ±0.025 |
| `GeometricCharacteristic` | GD&T (flatness, position, runout) | ⊕ Ø0.010 M A B C |
| `AngularCharacteristic` | Angle tolerance | 45° ±0.5° |
| `ThreadCharacteristic` | Thread form, pitch, major/minor dia. | M12×1.75-6H |
| `SurfaceTextureCharacteristic` | Surface finish | Ra 0.8 μm max |
| `UserDefinedCharacteristic` | Notes, material certs, process reqs | "DFARS Required" |

`UserDefinedCharacteristic` is used for non-geometric drawing requirements (notes, material callouts, surface treatment requirements) that appear on FAI Form 3 but cannot be CMM-measured. These are recorded with manual pass/fail entry.

---

## SPC Integration (Phase 2+)

The QIF Statistics module enables statistical process control analysis across multiple part measurements. Once QIF ingestion is operational:

1. QIF Results files accumulate in the production system for a given characteristic (across multiple serials)
2. QIF Statistics are computed: Cp, Cpk, Pp, Ppk per characteristic
3. Control chart data (X-bar/R, Individuals/MR) generated from the dataset
4. Out-of-control signals flagged per Western Electric rules
5. SPC data included in cert package when QA-017 (SPC required) is active on the job

This capability is the technical foundation for responding to QA-017 quality clause requirements without manual data compilation.

---

## AI Layer Queries (Claude MCP)

With Fulcrum-QIF integration active, Claude can answer quality-specific queries:

- "Which characteristics failed on job 25-0412 serial 007 and what was the deviation?"
- "Show me all FAI Form 3 results for part 12345-1 Rev C"
- "Which open jobs have inspection events with failed characteristics that don't have NCRs yet?"
- "What is the Cpk for Bore Diameter A on part 12345-1 across the last 50 measurements?"
- "Is the cert package for serial 7832-007 complete — do all required documents have pass status?"
- "How many characteristics on job 25-0412 have not been measured yet?"

---

## Phase 2 Verification Checklist

Before declaring Phase 2 complete:

- [ ] CMM software configured to export QIF 3.0 with manufacturer extension header fields
- [ ] QIF ingestion pipeline parsing `FulcrumJobId` and `FulcrumSerialId` correctly
- [ ] Characteristic UUIDs in QIF Results matching the production system inspection plan records
- [ ] Pass/fail results appearing on production job inspection record after ingestion
- [ ] FAI Form 3 populated from QIF Results for at least one FAI unit
- [ ] Failed characteristic triggering NCR creation flag in the production system
- [ ] Cert package eligibility flag set when all characteristics pass
- [ ] Claude MCP able to query inspection results by job number and serial number
