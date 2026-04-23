# QIF — Quality Information Framework

**Standard:** ISO 23952:2020 (equivalent to ANSI QIF 3.0)
**Maintained by:** Digital Metrology Standards Consortium (DMSC) — https://qifstandards.org
**Format:** XML, with published schemas (XSD) for each module
**Status at the shop:** Phase 2 implementation — CMM output integration following MTConnect Phase 1

---

## What the Standard Defines

QIF (Quality Information Framework) is the ISO/ANSI standard for exchanging quality and metrology data across software systems. It defines an XML schema covering the complete lifecycle of inspection activity: from measurement planning (what to measure) through results (what was found) to statistical analysis (what it means for the process).

QIF is designed to be consumed and produced by:
- CMM software (PC-DMIS, CALYPSO, Renishaw MODUS, Zeiss Calypso, Hexagon)
- Quality management systems (ETQ, Arena QMS, Discus)
- SPC software (InfinityQS, SPC for Excel)
- MES/ERP (via QIF import adapters)

### The Seven QIF Modules

**QIF Plans** — measurement plans generated from design intent:
- List of characteristics to measure (derived from GD&T in STEP AP242 or 2D drawing)
- Measurement strategy per characteristic (probe path, sampling strategy, number of points)
- Equipment requirements (CMM type, probe configuration, minimum accuracy)
- Sequence dependencies (datum establishment before feature measurement)

**QIF Results** — actual measurement data from a CMM run:
- Per-characteristic: nominal value, measured value, deviation, upper/lower tolerance, pass/fail status
- Measurement timestamp and operator/equipment reference
- Multiple results per characteristic when re-measured or sampled
- Association to the inspection plan via UUID references

**QIF Statistics** — process capability analysis:
- Cp, Cpk, Pp, Ppk per characteristic over multiple part measurements
- Histograms, run charts, control charts (Shewhart X-bar/R, individuals/MR)
- Out-of-control signals (Western Electric rules)
- Linked to the result sets from which statistics are derived

**QIF MBD** — model-based definition linkage:
- Associates QIF characteristics to geometric entities in a STEP AP242 model
- Enables programmatic generation of measurement plans from PMI
- Links measurement results back to specific model annotations for disposition

**QIF Resources** — measurement equipment definitions:
- CMM model, serial number, calibration date, uncertainty specification
- Probe configuration (stylus, tip ball diameter, length, material)
- Measurement uncertainty per operation type

**QIF Rules** — measurement strategy and tolerancing rules:
- Default sampling strategies for surface types
- Tolerancing interpretation rules (least squares, minimum zone, max inscribed)
- Standard measurement conditions (temperature, contact force)

**QIF Characteristics** — characteristic type library:
- Linear distance, diameter, radius, angle, flatness, straightness, roundness, cylindricity
- Perpendicularity, parallelism, angularity, position, concentricity, symmetry, runout
- Surface texture (Ra, Rz, Rq)
- Thread characteristics (pitch, minor/major diameter, form)

---

## UUID Structure in QIF

Every object in a QIF document carries a UUID. This is the native identity mechanism that makes QIF linkable to the rest of the manufacturer's digital thread:

```xml
<QIFDocument>
  <Header>
    <QPId>a3f7c2d1-8b4e-4a2f-9e1c-0d6b5a3f8c91</QPId>  <!-- Plan UUID -->
    <JobId>f91c0d6b-5a3f-8c91-a3f7-c2d18b4e4a2f</JobId>  <!-- the production system Job UUID -->
    <SerialNumber>2d18b4e4-a2f9-e1c0-d6b5-a3f8c91a3f7</SerialNumber>  <!-- Serial UUID -->
  </Header>
  ...
  <Characteristics>
    <LinearCharacteristic>
      <Id>e4a2f9e1-c0d6-b5a3-f8c9-1a3f7c2d18b4</Id>  <!-- Characteristic UUID -->
      <Name>Bore Diameter A</Name>
      <Nominal>25.400</Nominal>
      <ToleranceUpper>0.025</ToleranceUpper>
      <ToleranceLower>-0.025</ToleranceLower>
    </LinearCharacteristic>
  </Characteristics>
</QIFDocument>
```

The `JobId` and `SerialNumber` UUIDs are the manufacturer extensions to the QIF header — not in the base standard — that link the QIF document to the production system records. See `extensions/uuid-discipline.md`.

---

## What the shop Does with QIF

### Phase 2 Implementation Plan

1. **CMM software configuration:** Configure the manufacturer's CMM (Renishaw, Hexagon, or Zeiss) to export QIF results files at job completion.
2. **QIF header extension:** Add `JobId` and `SerialNumber` UUID fields to the QIF document header so every results file is self-identifying in the the production system context.
3. **Production system ingestion:** On QIF file receipt (manual import or automated folder watch), parse results and:
   - Link to production job and serial number via UUID
   - Store pass/fail status per characteristic
   - Trigger cert package eligibility check (if all characteristics pass → cert package can be assembled)
4. **MCP query layer:** Enable Claude to answer questions like "which characteristics failed on job 25-0412 serial 003?" by querying QIF result data linked to the production system.

### Inspection Plan Generation

For parts with STEP AP242 PMI available:
- Extract characteristics from PMI using STEP reader
- Generate QIF Plans programmatically (each GD&T callout → one QIF characteristic)
- Assign UUIDs to characteristics before CMM run begins
- CMM operator loads QIF Plan into CMM software; results populate against the same UUIDs

For parts from 2D drawings:
- Manually enter characteristics into inspection plan in the production system
- Generate QIF Plan from production system data
- Same UUID chain applies

---

## CM-Layer Gaps

### Gap 1: First Article Inspection (AS9102) — Forms 1 and 2

AS9102 Rev C defines FAI as a three-form process:

- **Form 1 — Part Number Accountability:** Part number, revision, serial number, drawing references, CAGE code, FAI reason (new part, design change, process change, production lapse, etc.), authorized signature.
- **Form 2 — Product Accountability:** Every material and process specification called out on the drawing, with: conforming spec revision, customer approval status if required, CoC reference, sub-tier FAI status.
- **Form 3 — Characteristic Accountability:** Every dimension, tolerance, and note on the drawing, with measured results, pass/fail, equipment used, date.

**QIF covers Form 3 (Characteristic Accountability) only.** Forms 1 and 2 require material/process compliance data that is entirely outside QIF's scope. The `extensions/quality-flowdowns.md` document defines the FAI structure that integrates all three forms.

### Gap 2: Nonconformance and MRB Disposition

When a QIF result shows a characteristic out of tolerance, the next step is:
1. Generate a Nonconformance Report (NCR)
2. Convene Material Review Board (MRB) disposition: use-as-is, rework, repair, scrap, return-to-vendor
3. If rework: re-inspect, generate new QIF results, reference original NCR
4. If use-as-is: engineer's justification, customer approval if required
5. Document all dispositions in the cert package

QIF has no schema for NCR creation, MRB decisions, rework tracking, or customer deviation/waiver requests. These are critical quality events in aerospace CM that currently live in separate systems or paper records.

### Gap 3: Special Process Certifications Are Outside QIF

A CMM measures part geometry. It does not verify:
- Whether the heat treat cycle achieved the specified temperature/time/quench
- Whether the FPI or MPI inspection was performed and parts were accepted
- Whether the plating thickness is within spec

These special process results are certified by the outside vendor in separate documents (process certifications, cert packages). QIF has no module for importing or referencing these documents alongside dimensional results.

### Gap 4: In-Process Quality Data

Dimensional checks performed during machining — not on a CMM but with hand tools, micrometers, go/no-go gages at the machine — are not captured in QIF. They are recorded on paper travelers. These in-process check results are quality evidence but not formal CMM data.

---

## Integration Points

| QIF Data | Links To | Via |
|----------|----------|-----|
| Plan UUID | the production system inspection event UUID | UUID discipline |
| Job UUID (extension field) | production job UUID | UUID discipline |
| Serial UUID (extension field) | the production system serial number UUID | UUID discipline |
| Characteristic pass/fail results | the production system inspection record | QIF ingestion |
| Equipment ID | the production system calibration record | Equipment UUID |

---

## Resources

- **QIF schemas (XSD):** https://qifstandards.org/schemas/
- **DMSC QIF community:** https://qifstandards.org
- **PC-DMIS QIF export:** Hexagon's PC-DMIS supports QIF 3.0 export natively
- **NIST QIF test artifacts:** Publicly available QIF files for testing parsers and validators
- **IOF / QIF alignment:** The Industrial Ontology Foundry (IOF) is developing OWL representations of QIF concepts; the manufacturer's inspection extension should align with IOF's quality ontology where applicable
