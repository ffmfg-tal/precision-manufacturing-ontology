# Aerospace Precision Manufacturing Ontology
## Domain Knowledge Reference for ERP Database Specification

---

## 1. Material Specifications in Aerospace

### 1.1 How Materials Are Specified

Aerospace materials are identified through a layered system of specifications, conditions, and forms. A typical material callout on an engineering drawing looks like:

```
15-5 PH STAINLESS STEEL PER AMS 5659 TYPE 1, CONDITION A
H/T TO H1150M PER AMS 2759/3
```

This encodes:
- **Common name**: 15-5 PH Stainless Steel
- **Specification**: AMS 5659 (defines chemistry, mechanical properties, form)
- **Type/Class**: Subtypes within a spec (e.g., Type 1 = bar, Type 2 = wire)
- **Condition**: The delivery condition (e.g., Condition A = solution treated)
- **Heat treatment callout**: Post-machining H/T requirement with its own AMS spec

**Data model implication**: A material entity must support multiple overlapping identifiers (common name, UNS number, AMS spec, ASTM spec, customer spec), plus condition codes, type/class designators, and linked process specifications.

### 1.2 Specification Systems

Materials are cross-referenced across several overlapping specification bodies:

| System | Governing Body | Purpose | Example |
|--------|---------------|---------|---------|
| **AMS** (Aerospace Material Specifications) | SAE International | Primary aerospace material specs | AMS 5659 (15-5PH bar) |
| **ASTM** | ASTM International | General material/testing standards | ASTM A564 (PH stainless) |
| **UNS** (Unified Numbering System) | SAE/ASTM joint | Cross-reference alloy identity | S15500 (15-5PH) |
| **MIL-SPEC** | U.S. DoD | Military specifications (many superseded by AMS) | MIL-S-83165 |
| **AMS-H** / **AMS-C** / **AMS-QQ** | SAE | Heat treat / chemical process / QQ-series specs | AMS 2759 (H/T of steel) |
| **ASME** | ASME | Pressure vessel, piping materials | ASME SA-479 |
| **Customer/OEM specs** | Boeing, Pratt & Whitney, etc. | OEM-specific material and process specs | BAC 5602, PWA 36604 |

**AMS Specification Structure**:
- AMS covers 24 commodity groups including corrosion/heat-resistant steels, wrought low-alloy steels, aluminum, magnesium, copper, titanium, and composites
- Each AMS spec defines: chemical composition, mechanical property requirements, form/product (bar, sheet, plate, forging), condition/temper, testing requirements, quality assurance provisions
- AMS process specs (AMS 2759 series for heat treat, AMS 2700 series for passivation, etc.) are separate from material specs

### 1.3 Common Aerospace Alloys and Their Specifications

#### Precipitation Hardening Stainless Steels
| Alloy | Primary AMS | ASTM | UNS | Common Conditions |
|-------|-----------|------|-----|-------------------|
| 15-5 PH | AMS 5659 (bar), AMS 5862 (plate) | A564 XM-12 | S15500 | H900, H1025, H1100, H1150 |
| 17-4 PH | AMS 5643 (bar), AMS 5604 (sheet) | A564 Type 630 | S17400 | H900, H1025, H1075, H1150 |
| 13-8 Mo | AMS 5629 | A564 XM-13 | S13800 | H950, H1000, H1050 |
| Custom 455 | AMS 5617 | A564 XM-16 | S45500 | H900, H950, H1000 |

#### Nickel-Based Superalloys
| Alloy | Primary AMS | UNS | Notes |
|-------|-----------|-----|-------|
| Inconel 625 | AMS 5666 (bar), AMS 5599 (sheet) | N06625 | Corrosion/heat resistant |
| Inconel 718 | AMS 5662/5663 (bar), AMS 5596 (sheet) | N07718 | Most common aerospace superalloy |
| Waspaloy | AMS 5704/5706 | N07001 | High-temp turbine applications |
| Hastelloy X | AMS 5754 (sheet), AMS 5536 (bar) | N06002 | Combustion components |

#### Titanium Alloys
| Alloy | Primary AMS | UNS | Notes |
|-------|-----------|-----|-------|
| Ti 6Al-4V | AMS 4928 (bar), AMS 4911 (sheet) | R56400 | Most common titanium alloy |
| Ti 6Al-4V ELI | AMS 4930 | R56401 | Extra-low interstitial, surgical/cryo |
| CP Ti Grade 2 | AMS 4902 | R50400 | Commercially pure |

#### Aluminum Alloys
| Alloy | Primary AMS | UNS | Tempers |
|-------|-----------|-----|---------|
| 7075 | AMS 4045 (sheet), AMS 4122 (bar) | A97075 | T6, T651, T7351 |
| 2024 | AMS 4037 (sheet), AMS 4120 (bar) | A92024 | T3, T351, T4 |
| 6061 | AMS 4027 (sheet), AMS 4150 (bar) | A96061 | T6, T651 |

#### Alloy Steels
| Alloy | Primary AMS | UNS | Notes |
|-------|-----------|-----|-------|
| 4340 | AMS 6414 (bar), AMS 6359 (sheet) | G43400 | High-strength structural |
| 300M | AMS 6419 | K44220 | Ultra-high strength |
| 4130 | AMS 6370 (tube), AMS 6350 (bar) | G41300 | Weldable structural |

### 1.4 Heat Treatment Callouts

Heat treatment in aerospace is called out by referencing a process specification plus a condition code:

```
HEAT TREAT TO H1025 PER AMS 2759/3
SOLUTION TREAT AND AGE PER AMS 5662 (718)
HEAT TREAT PER BAC 5617 CLASS 3 (Boeing-specific)
```

**Heat Treatment Specification Hierarchy**:
- **AMS 2759** - Heat Treatment of Steel Parts (general)
  - AMS 2759/1 - Carbon and low-alloy steel
  - AMS 2759/3 - Precipitation hardening stainless (17-4, 15-5, 13-8)
  - AMS 2759/4 - Austenitic corrosion-resistant steel
  - AMS 2759/5 - Martensitic corrosion-resistant steel
  - AMS 2759/9 - Hardness and conductivity of aluminum
- **AMS 2774** - Heat Treatment of Titanium
- **AMS 2770** - Heat Treatment of Aluminum
- **AMS 2801** - Heat Treatment of Nickel Alloys

**Condition codes** (e.g., H900, H1025, H1150M):
- The number represents temperature in Fahrenheit (H1025 = aged at 1025F)
- Suffix letters indicate variant (M = modified, DH = double-aged/hardened)
- Each condition maps to specific mechanical property targets (tensile, yield, hardness)

**Data model implication**: Heat treatment is both a material attribute (the spec defines allowed conditions) and a manufacturing operation (with its own routing step, certifications, and quality records). The relationship is many-to-many: a material can have multiple valid H/T conditions, and a H/T process spec applies to multiple materials.

### 1.5 Material Forms

Materials are procured in specific product forms, each with distinct specifications:

| Form | Description | Typical Specs |
|------|-------------|---------------|
| **Bar** (round, flat, hex) | Solid wrought stock, hot/cold finished | AMS 5659 (15-5 bar) |
| **Plate** | Flat rolled, >0.187" thick typically | AMS 5862 (15-5 plate) |
| **Sheet** | Flat rolled, <0.187" thick typically | AMS 5604 (17-4 sheet) |
| **Forging** | Die or hand forged shapes | AMS 5673 (718 forging) |
| **Casting** | Investment or sand cast | AMS 5362 (17-4 casting) |
| **Tube/Pipe** | Hollow cylindrical | AMS 5556 (321 tube) |
| **Wire** | Small-diameter drawn product | AMS 5678 (718 wire) |
| **Extrusion** | Pushed through a die | AMS 4150 (6061 extrusion) |
| **Powder** | For additive/PM processes | AMS 5663 (718 powder) |

**Data model implication**: The same base alloy has different AMS specs per form. The form is essential for purchasing (drives supplier selection), estimating (affects material cost and machinability), and traceability.

### 1.6 Material Certifications and Traceability

#### Certificate of Conformance (C of C / CoC)
A document from the material supplier stating the material conforms to the purchase order and referenced specifications. Contains:
- Heat/lot number
- Chemistry (actual vs. spec limits)
- Mechanical test results (tensile, yield, elongation, hardness)
- Dimensional verification
- Specification compliance statement
- Supplier signature/authorization

#### Mill Test Report (MTR) / Certified Material Test Report (CMTR)
The original test report from the mill/foundry, documenting:
- Melt source (country of melt, country of manufacture)
- Heat number, lot number
- Chemical analysis (ladle and product)
- Mechanical test results
- Conformance to the applicable spec

#### DFARS Compliance (252.225-7009/7014)
For DoD contracts, specialty metals (steel, titanium, zirconium, tantalum, certain alloys) must be melted in the U.S. or a qualifying country. Requirements:
- MTR must show melt country of origin
- CoC must affirm DFARS compliance
- Documentation flows down through entire supply chain
- Qualifying countries include: Australia, Belgium, Canada, Czech Republic, Denmark, Egypt, Finland, France, Germany, Greece, Israel, Italy, Japan, Luxembourg, Netherlands, Norway, Poland, Portugal, Spain, Sweden, Switzerland, Turkey, UK

#### Conflict Minerals (DFARS 252.225-7052)
- Tin, tantalum, tungsten, gold (3TG) must not originate from conflict zones
- Since December 2019, tungsten from China, North Korea, Russia, Iran is restricted
- Suppliers must provide conflict mineral declarations

#### Traceability Chain
```
Raw Material Mill -> Distributor -> Machine Shop -> Sub-tier -> Prime -> OEM
     MTR              CoC           Internal       CoC+certs   Full package
                                    lot tracking
```

Each link in the chain must maintain:
- Lot/heat traceability to original melt source
- Positive material identification (PMI) where required
- Shelf-life tracking for age-sensitive materials
- Certificate retention (typically 7-20 years depending on program)

**Data model implication**: The material certification system requires a document management layer linked to material lots, purchase orders, and jobs. A single material lot may have: MTR, CoC, DFARS declaration, conflict minerals declaration, PMI report, and shelf-life records. These documents must be retrievable by lot, part, job, customer, or contract.

---

## 2. Part/Item Hierarchy

### 2.1 Part Definition

An aerospace part is defined by the intersection of:

**Core Identity**:
- **Part Number**: Unique identifier. May be customer-assigned or internal
- **Revision Level**: Letter or number indicating design iteration (-, A, B, C... or 01, 02, 03...)
- **Dash Number/Configuration**: Variant within a part number (e.g., 12345-1, 12345-2)
- **Drawing Number**: May or may not equal the part number
- **CAGE Code**: Commercial and Government Entity code identifying the design authority

**Part Number Schemes**:
- **Significant (smart) numbering**: Encodes information in the number (e.g., prefix = product line, suffix = size)
- **Non-significant (dumb) numbering**: Sequential numbers with no encoded meaning
- **Hybrid**: Some segments meaningful, others sequential
- Typical format: alphanumeric, 6-15 characters, may include dashes

**Revision Control Rules**:
- A new revision is issued for changes that do NOT affect fit, form, or function (editorial, tolerance refinement)
- A new part number is issued for changes that DO affect fit, form, or function (interchangeability broken)
- "One-up where-used" rule: if a component revises, the parent assembly BOM must also revise
- Effectivity dating: tracking which revision is valid for which serial number range or date range

### 2.2 Item Classifications

| Classification | Description | Examples |
|---------------|-------------|---------|
| **Make** | Manufactured in-house from raw material | Machined housings, brackets |
| **Buy** | Purchased as finished goods | Bearings, fasteners, O-rings |
| **Outside Process** | Sent out for special processing then returned | Heat treat, plating, NDT |
| **Phantom/Reference** | Logical grouping, not physically stocked | Planning BOM levels |
| **Raw Material** | Unprocessed stock material | Bar stock, plate, sheet |
| **Tooling** | Fixtures, gages, cutters | Soft jaws, inspection fixtures |
| **Consumable** | Used in production but not part of product | Coolant, deburring media |

### 2.3 BOM (Bill of Materials) Structure

#### Single-Level BOM
Lists only the immediate children of an assembly:
```
Assembly 12345 Rev C
  |-- Part 12345-1 (Housing)      Qty: 1   Make
  |-- Part 12345-2 (Cover)        Qty: 1   Make
  |-- Part AN3-5A (Bolt)          Qty: 4   Buy
  |-- Part MS29513-012 (O-Ring)   Qty: 2   Buy
```

#### Indented (Multi-Level) BOM
Shows full hierarchy:
```
Level 0: Assembly 12345 Rev C
  Level 1: Part 12345-1 (Housing)        Qty: 1   Make
    Level 2: Raw Material - 15-5 Bar      Qty: 1   Buy (raw)
    Level 2: Outside Process - H/T        Qty: 1   Outside
    Level 2: Outside Process - Passivate  Qty: 1   Outside
  Level 1: Part 12345-2 (Cover)           Qty: 1   Make
    Level 2: Raw Material - 15-5 Plate    Qty: 1   Buy (raw)
  Level 1: AN3-5A (Bolt)                  Qty: 4   Buy
  Level 1: MS29513-012 (O-Ring)           Qty: 2   Buy
```

#### BOM Fields for Aerospace
- Part number + revision
- Quantity per assembly
- Unit of measure
- Find number / reference designator (matches balloon on drawing)
- Make/Buy designation
- Effectivity range (serial numbers or dates)
- Notes/callouts (e.g., "APPLY LOCTITE 242 AT ASSEMBLY")
- Alternate parts (approved substitutions)
- Next higher assembly (where-used)

### 2.4 Drawing and Model References

**Traditional Drawing-Based Definition**:
- 2D drawing (PDF or TIFF) with dimensions, tolerances, notes, and material callout
- Drawing number typically equals part number
- Drawing revision letter tracks with part revision

**Model-Based Definition (MBD)**:
- 3D CAD model contains all GD&T, notes, and specifications directly
- Governed by ASME Y14.41 (Digital Product Definition Data Practices)
- The model IS the authority; no separate 2D drawing needed
- PMI (Product and Manufacturing Information) embedded in model

**Hybrid approach** (most common in job shops):
- Customer provides a 2D drawing (PDF) as contractual authority
- 3D model (STEP, IGES, or native CAD) provided for programming reference
- Both must be revision-controlled and linked to the part record

**Data model implication**: Parts need links to multiple document types (drawing, model, specification sheets), each with its own revision tracking. The system must enforce that the manufacturing planning references the correct revision of all documents.

---

## 3. Manufacturing Operations

### 3.1 Common Operations in Precision Aerospace Machining

#### Primary Material Removal
| Operation | Description | Typical Tolerances |
|-----------|-------------|-------------------|
| **CNC Milling (3-axis)** | Flat/prismatic features | +/- 0.001" to 0.005" |
| **CNC Milling (4/5-axis)** | Complex contours, compound angles | +/- 0.0005" to 0.002" |
| **CNC Turning** | Rotational parts on lathe | +/- 0.0005" to 0.002" |
| **Mill-Turn** | Combined milling and turning in one setup | +/- 0.0005" |
| **Swiss-type Turning** | Small precision parts with guide bushing | +/- 0.0002" to 0.0005" |
| **EDM (Wire)** | Complex profiles cut by wire electrode | +/- 0.0001" to 0.0005" |
| **EDM (Sinker/Ram)** | Cavities/blind features by shaped electrode | +/- 0.0002" to 0.001" |
| **Grinding (Surface)** | Flat surface finishing | +/- 0.0001" to 0.0005" |
| **Grinding (Cylindrical)** | OD/ID round surfaces | +/- 0.0001" to 0.0002" |
| **Grinding (Centerless)** | High-volume OD grinding | +/- 0.0002" |
| **Honing** | Bore finishing for size/finish | +/- 0.0001" |
| **Lapping** | Ultra-precision flat finishing | +/- 0.00005" |
| **Broaching** | Internal keyways, splines | +/- 0.0005" |

#### Secondary/Finishing Operations
| Operation | Description |
|-----------|-------------|
| **Deburring** | Manual or mechanical edge break |
| **Vibratory finishing** | Mass finishing for edge break/surface blend |
| **Bead blasting/glass bead peening** | Surface texturing/stress relief |
| **Shot peening** | Controlled surface compression per AMS 2430/2432 |
| **Polishing/Buffing** | Achieve specified surface finish (Ra/Rz) |
| **Marking** | Part identification per MIL-STD-130 or drawing callout |
| **Assembly** | Mechanical assembly of multi-component parts |

#### Special Processes (typically outside-processed, Nadcap required)
| Process Category | Examples | Governing Specs |
|-----------------|----------|-----------------|
| **Heat Treatment** | Solution treat, age, stress relieve, anneal | AMS 2759 series, AMS 2770/2774 |
| **Chemical Processing** | Passivation, anodizing, chromate conversion, chem film | AMS 2700 (passivation), MIL-A-8625 (anodize) |
| **Plating** | Cadmium, nickel, chrome, silver, gold | AMS 2400 series, QQ-N-290 |
| **Painting/Coating** | Primer, topcoat, dry film lube, HVOF, plasma spray | MIL-PRF-23377 (primer), MIL-PRF-85285 |
| **NDT** | FPI (fluorescent penetrant), MPI, UT, radiography, eddy current | ASTM E1417 (FPI), ASTM E1444 (MPI) |
| **Welding** | TIG, EB, laser, friction stir | AMS 2680/2681, AWS D17.1 |

### 3.2 Process Specifications and Nadcap

**Nadcap** (National Aerospace and Defense Contractors Accreditation Program) is the industry-standard accreditation for special processes. Administered by the Performance Review Institute (PRI), it covers 24 process categories including:

- Heat Treating (HT)
- Chemical Processing (CP)
- Non-Destructive Testing (NDT)
- Coatings (CT)
- Welding (WLD)
- Non-Conventional Machining and Surface Enhancement (NMSE)
- Conventional Machining as a Special Process (CMSP)
- Materials Testing Laboratories (MTL)
- Measurement and Inspection (M&I)
- Composites (COMP)
- Elastomer Seals (SEAL)

**Why "special process"**: These are processes whose output cannot be fully verified by subsequent inspection or testing. You cannot tell by looking at a part whether the heat treatment was done correctly -- hence the need for rigorous process control and certification.

**Data model implication**: Operations in the routing must be classified as standard vs. special process. Special processes require: Nadcap accreditation status of the processor, process specification reference (e.g., "AMS 2759/3"), customer/OEM approval status (some OEMs maintain their own approved processor lists beyond Nadcap), and certification documentation linked to the job.

### 3.3 Setup and Runtime Concepts

**Job Routing / Work Order Operations**:
Each job has a routing (sequence of operations), where each operation includes:

| Field | Description |
|-------|-------------|
| **Op Number** | Sequence position (10, 20, 30... allows insertion) |
| **Op Code** | Standard operation identifier (e.g., CNC_MILL, OD_GRIND, HT_OUT) |
| **Work Center** | Where the operation is performed (machine or department) |
| **Setup Time** | Time to set up the machine/fixture (hours) |
| **Cycle Time / Runtime** | Time per piece (or per lot) |
| **Overlap/Concurrent** | Whether op can start before prior op finishes |
| **Instructions** | Setup sheet, program number, tooling list |
| **Inspection Points** | In-process inspection requirements |
| **Outside Vendor** | If outsourced, which vendor |

**Time Estimation Categories**:
- **Setup time**: Fixed time per lot (mount fixture, load program, first-piece verify)
- **Cycle time**: Time per piece (machine time + load/unload)
- **Queue time**: Wait time before work begins at a work center
- **Move time**: Transit time between work centers
- **Inspection time**: Time for in-process or final inspection

### 3.4 Tooling and Fixtures

| Type | Description | Tracking Needs |
|------|-------------|---------------|
| **Cutting tools** | End mills, drills, inserts, reamers | Tool life tracking, regrind history |
| **Workholding** | Vises, chucks, collets, soft jaws | Maintenance schedule, part association |
| **Fixtures** | Custom-built part-specific holding devices | Part number link, revision, storage location |
| **Inspection gages** | Go/no-go, plug, ring, thread gages | Calibration due dates, calibration certs |
| **CMM fixtures** | Coordinate measuring machine holding fixtures | Part association, program reference |
| **Cutting programs** | CNC G-code programs | Version control, post-processor reference |

**Data model implication**: Tooling is an entity class that intersects with operations (which tools are used at which operation), quality (gage calibration), and cost accounting (tooling cost allocation per job).

---

## 4. Quality and Compliance

### 4.1 AS9100 Requirements Affecting Data Modeling

AS9100 (current revision D, transitioning to IA9100 in 2026-2029) adds 105 requirements beyond ISO 9001:2015. Key data-impacting requirements:

| Requirement Area | Data Model Impact |
|-----------------|-------------------|
| **Traceability** | Every part must be traceable to raw material lot, supplier, manufacturing records |
| **Configuration Management** | Must track as-designed vs. as-built configuration |
| **Risk Management** | Risk registers linked to parts, processes, suppliers |
| **Key Characteristics** | Special identification and statistical control of critical dimensions |
| **Counterfeit Parts Prevention** | Verification of authenticity, approved source tracking |
| **Operator Competency** | Training records linked to operations they can perform |
| **Equipment Calibration** | Calibration status of all measurement equipment |
| **Document Control** | Revision-controlled access to all procedures and specifications |
| **Nonconformance Management** | NCR tracking, disposition, root cause, corrective action |
| **Customer Property** | Tracking of customer-furnished material, tooling, or data |
| **Records Retention** | Defined retention periods per contract/regulation |

**IA9100 (upcoming)** adds emphasis on:
- Data integrity and digital manufacturing controls
- AI-assisted inspection validation
- Digital twin and MES/ERP data traceability
- Cybersecurity as part of quality management
- Statistical Process Control (SPC), Measurement System Analysis (MSA), Design of Experiments (DOE)

### 4.2 First Article Inspection (FAI) per AS9102

AS9102 Rev C defines the FAI process using three forms:

**Form 1 - Part Number Accountability**:
- Part number, revision, drawing number
- Serial number of FAI unit
- Organization name, CAGE code
- Reason for FAI (new part, design change, process change, production lapse)
- Signature and date

**Form 2 - Product Accountability (Materials and Processes)**:
- Line items for every material and process specification called out on the drawing
- Specification number, class, type
- Whether customer technical approval is required
- Reference to CoC/CoA document number
- Whether a sub-tier FAI exists

**Form 3 - Characteristic Accountability**:
- Every dimension, tolerance, note, and characteristic from the drawing
- Characteristic number (matches balloon number on inspection drawing)
- Drawing requirement (nominal, tolerance)
- Actual measured result
- Pass/fail/deviation status
- Measuring equipment used

**FAI Triggers**:
- First production run of a new part
- Design change affecting fit, form, or function
- Change in manufacturing process or source
- Natural or man-made event affecting the process
- Lapse in production (typically >2 years)
- Change in inspection method
- Sub-contractor or supplier change

**Data model implication**: FAI is a document package that aggregates data from the BOM (materials), routing (processes), and inspection results (characteristics). The ERP must generate or link to all three form types, track FAI status per part/revision, and flag when re-FAI is needed.

### 4.3 ITAR/EAR Considerations

**ITAR (International Traffic in Arms Regulations)**:
- Administered by U.S. State Department / DDTC
- Applies to items on the U.S. Munitions List (USML)
- Restricts access to technical data and defense articles to U.S. persons only
- Requires registration, licensing for exports
- Flows down through entire supply chain

**EAR (Export Administration Regulations)**:
- Administered by U.S. Commerce Department / BIS
- Applies to dual-use items on the Commerce Control List (CCL)
- Less restrictive than ITAR but still requires licensing for certain destinations

**ERP Data Model Requirements for ITAR/EAR**:

| Requirement | Implementation |
|------------|----------------|
| **Data segregation** | ITAR-controlled data must be isolated from non-ITAR data |
| **Access control** | Role-based access restricted to U.S. persons for ITAR data |
| **Audit trail** | Complete log of who accessed ITAR-controlled information and when |
| **Marking/labeling** | Items and data must be marked with export control classification |
| **Physical security** | Controlled areas for ITAR parts/materials in shop |
| **Electronic transmission** | ITAR data cannot traverse foreign servers or be accessed from abroad |
| **Cloud hosting** | Must use ITAR-compliant cloud (e.g., AWS GovCloud) |
| **Personnel tracking** | Citizenship/authorization status of all employees with access |
| **License tracking** | Export license numbers, conditions, expiration dates |

**Data model implication**: Every item, document, and data record needs an export control classification field. Access control must be granular enough to restrict ITAR items while allowing normal operations on commercial work. Adds 15-30% complexity to ERP implementation.

### 4.4 Nonconformance and MRB (Material Review Board) Process

**Nonconformance Report (NCR) Data Elements**:
- NCR number (sequential, unique)
- Date discovered
- Part number, revision, serial/lot number
- Job/work order number
- Operation where discovered
- Description of nonconformance
- Drawing requirement vs. actual condition
- Quantity affected
- Discovered by whom
- Severity classification (major, minor, critical)

**MRB Composition**:
- Quality representative (facilitates and documents)
- Engineering representative (technical authority)
- Customer representative (when required for safety-critical or contract items)
- Additional SMEs as needed (stress, manufacturing, procurement)

**Disposition Options**:
| Disposition | Description | Authority Level |
|------------|-------------|-----------------|
| **Use As Is** | No rework needed; does not affect fit/form/function | MRB (may need customer approval) |
| **Rework** | Can be brought to drawing requirements | Quality + Engineering |
| **Repair** | Can be made functional but not to original spec | MRB + customer approval usually |
| **Scrap** | Cannot be made conforming; destroy/recycle | Quality authority |
| **Return to Vendor** | Supplier-caused nonconformance | Quality + Purchasing |

**Process Flow**:
```
Discovery -> Documentation (NCR) -> Segregation/Quarantine
    -> Investigation (root cause analysis)
    -> MRB Review -> Disposition Decision -> Sign-off
    -> Corrective Action (if systemic)
    -> Closure and Records Retention
```

**Data model implication**: NCRs link to jobs, parts, operations, and suppliers. Dispositions may generate new work orders (rework routing), change order status, or trigger supplier corrective actions (SCAR). Cost of quality metrics (scrap cost, rework cost) must be calculated and tracked.

---

## 5. Supply Chain

### 5.1 Approved Supplier Lists (ASL)

**ASL Data Elements**:
- Supplier name, address, CAGE code
- Commodities/capabilities approved for
- Quality certifications (ISO 9001, AS9100, Nadcap)
- Specific process approvals (which AMS/MIL specs they are approved for)
- OEM/customer approval status (Boeing-approved, Pratt-approved, etc.)
- ITAR registration status
- Performance metrics (on-time delivery, quality rejection rate)
- Audit/survey dates and results
- Approved/conditional/probationary/disqualified status
- DUNS number, tax ID

**Approval Hierarchy**:
```
Customer-Directed Source (mandatory)
  -> Customer-Approved Source (from customer's ASL)
    -> Nadcap-Accredited Source (for special processes)
      -> Internally-Approved Source (shop's own qualification)
```

**Special Process Supplier Qualification**:
For special processes (heat treat, plating, NDT, welding), suppliers must typically:
1. Hold Nadcap accreditation for the specific process
2. Be on the end-customer's approved processor list (if customer-directed)
3. Have been audited/surveyed by the machine shop
4. Maintain current certifications and insurance

### 5.2 Purchase Order Structures with Aerospace Callouts

A precision aerospace PO contains far more information than a commercial PO:

**Header-Level Fields**:
- PO number, date, revision
- Supplier information
- Ship-to/bill-to
- Contract/program reference
- Export control classification (ITAR/EAR marking)
- Required delivery date
- Payment terms

**Line-Item Fields**:
- Part number, revision, drawing reference
- Quantity, unit of measure, unit price
- Material specification (for raw material POs)
- Condition/temper (for raw material)
- DFARS compliance requirement
- Certification requirements (CoC, MTR, etc.)
- Inspection requirements
- Specification callouts

**Quality Clause Flowdowns** (typically referenced by code on PO):
Common clause categories:
- QA-001: Quality system requirements (AS9100 registration)
- QA-002: First Article Inspection required per AS9102
- QA-003: Certificates of Conformance required with each shipment
- QA-004: Material certifications (MTR) required
- QA-005: Right of access for buyer/customer/regulatory audit
- QA-006: Nonconforming material notification requirements
- QA-007: Sub-tier supplier control (no further subcontracting without approval)
- QA-008: DFARS specialty metals compliance
- QA-009: Conflict minerals compliance
- QA-010: Record retention requirements
- QA-011: ITAR compliance and handling requirements
- QA-012: Counterfeit parts prevention
- QA-013: Nadcap accreditation required for special processes
- QA-014: Change notification requirements (cannot change process without buyer approval)
- QA-015: FOD (Foreign Object Debris) prevention
- QA-016: Shelf-life material controls
- QA-017: Statistical data required (Cp/Cpk for key characteristics)

### 5.3 Outside Processing (Subcontracted Operations)

Outside processing is where partially-completed parts are sent to a vendor for a specific operation (heat treat, plating, NDT, etc.) and then returned.

**Outside Processing Data Model**:

```
Job/Work Order
  |-- Op 10: CNC Mill (internal)
  |-- Op 20: CNC Mill (internal)
  |-- Op 30: SHIP TO H/T VENDOR (outside process)
  |       |-- Outside PO created
  |       |-- Ship parts out (packing list, traveler)
  |       |-- Vendor performs H/T per AMS 2759/3 to H1025
  |       |-- Parts returned with CoC
  |       |-- Receiving inspection
  |-- Op 40: Final machine (internal)
  |-- Op 50: SHIP TO FPI VENDOR (outside process)
  |       |-- Similar flow as above
  |-- Op 60: Final inspection (internal)
```

**Outside Process PO Requirements**:
- Reference to job/work order
- Process specification (e.g., "HEAT TREAT PER AMS 2759/3 TO H1025")
- Drawing number and revision
- Material type (so vendor knows what they are processing)
- Quantity and part identification
- Flowdown quality clauses
- Required certifications (CoC, test reports)
- Nadcap accreditation verification
- Special handling instructions (ITAR, ESD, etc.)
- Required turnaround time

**Data model implication**: Outside processing creates a hybrid entity that is both an operation in the routing AND a purchase order. The ERP must track: parts shipped out (which serial/lot numbers), vendor receipt confirmation, vendor processing, parts returned, receiving inspection, and documentation (CoCs, test reports). Cost is tracked as both a job cost and a PO cost.

---

## 6. Existing Standards and Ontologies

### 6.1 STEP (ISO 10303) - Standard for the Exchange of Product Model Data

**Relevant Application Protocols**:
- **AP 203**: Configuration Controlled 3D Design
- **AP 214**: Automotive design (widely used in manufacturing)
- **AP 238 (STEP-NC)**: CNC machining data -- model-based machining instructions
- **AP 242**: Managed Model-Based 3D Engineering -- the current flagship AP for aerospace, combining capabilities of AP 203 and AP 214 with MBD support

**AP 242 Capabilities**:
- 3D geometry with PMI (GD&T, annotations)
- Product structure (BOM)
- Configuration management
- Tessellated geometry for visualization
- Composite materials definition
- Kinematics for mechanisms
- Classification and identification schemes
- Long-term data retention (critical for aerospace programs spanning decades)

**Data model relevance**: STEP provides the product definition data model. Parts, assemblies, geometry, tolerances, and BOMs can be represented in STEP format. For an ERP, STEP is the format for exchanging product definition data with PLM/CAD systems.

### 6.2 QIF (Quality Information Framework) - ISO 23952:2020

QIF is an XML-based standard from DMSC (Dimensional Metrology Standards Consortium) covering the quality measurement lifecycle:

**QIF Modules**:
| Module | Purpose |
|--------|---------|
| **QIF MBD** | Model-Based Definition -- product geometry with GD&T |
| **QIF Plans** | Measurement plans -- what to inspect, how, with what equipment |
| **QIF Resources** | Measurement device capabilities and characteristics |
| **QIF Rules** | Measurement rules and strategies |
| **QIF Results** | Actual measurement data from inspection |
| **QIF Statistics** | Statistical analysis of measurement results (SPC, Cpk) |

**Key Design Principles**:
- UUID (Universally Unique Identifier) for every characteristic and feature
- Common library approach -- all modules share base data models
- XML Schema (XSD) architecture
- Platform-independent
- Integrates with STEP for product definition

**Data model relevance**: QIF provides the schema for quality/inspection data. An ERP system should be able to import/export QIF data for inspection plans (tied to FAI forms) and inspection results (tied to job quality records). Particularly important for: first article inspection data, in-process inspection records, SPC data for key characteristics, and CMM measurement results.

### 6.3 MTConnect

MTConnect is an open, royalty-free standard for manufacturing equipment data:
- Real-time machine monitoring (spindle speed, feed rate, axis position)
- Machine status (running, idle, alarm, setup)
- Production counts
- Alarm/error reporting

**Data model relevance**: MTConnect provides shop-floor machine data that feeds into the ERP for: actual cycle times vs. estimated, machine utilization metrics, OEE (Overall Equipment Effectiveness) calculations, and predictive maintenance scheduling.

### 6.4 ISA-95 / IEC 62264 - Enterprise-Control System Integration

ISA-95 defines the standard terminology and data models for integrating business systems (ERP, Level 4) with manufacturing operations systems (MES, Level 3) and control systems (Levels 1-2).

**ISA-95 Data Model Categories**:

| Category | Contents |
|----------|----------|
| **Product Definition** | What to make: BOM, routing, specifications |
| **Production Capability** | What can be made: equipment, personnel, material capabilities |
| **Production Schedule** | What is planned: work orders, sequencing, timing |
| **Production Performance** | What was made: actuals vs. plan, quality, efficiency |

**Resource Models** (each category above references these):
- Personnel (operators, qualifications, availability)
- Equipment (machines, tools, maintenance status)
- Material (raw material, WIP, finished goods)
- Process Segments (reusable operation definitions)

**ISA-95 Hierarchy Levels**:
```
Level 4: Business Planning & Logistics (ERP)
   - Order processing, production scheduling, inventory management
Level 3: Manufacturing Operations Management (MES)
   - Dispatching, detailed scheduling, quality management
Level 2: Monitoring, Supervision, and Control
   - Operator interface, supervisory control
Level 1: Sensing and Manipulation
   - Direct machine control, sensors, actuators
Level 0: Physical Process
   - Actual manufacturing operations
```

**Data model relevance**: ISA-95 provides the canonical data exchange model between ERP and shop floor. The four information categories (product definition, capability, schedule, performance) map directly to ERP modules. B2MML (Business to Manufacturing Markup Language) is the XML implementation of ISA-95.

### 6.5 OAGIS (Open Applications Group Integration Specification)

OAGIS provides standardized XML-based message formats (called BODs - Business Object Documents) for application-to-application integration:

**Relevant BODs for Manufacturing**:
- SyncBOM -- Bill of Materials synchronization
- SyncProductionOrder -- Work order exchange
- SyncShipment -- Shipment data
- SyncPurchaseOrder -- Purchase order exchange
- SyncInventory -- Inventory transactions
- SyncInspection -- Quality inspection data

**Data model relevance**: OAGIS defines the message format for integrating the ERP with external systems (customer portals, supplier systems, EDI). Historically used more in discrete manufacturing, now aligned with ISA-95/B2MML for manufacturing operations data.

### 6.6 Other Relevant Standards and Ontologies

| Standard/Framework | Scope | Relevance |
|-------------------|-------|-----------|
| **NIST AMS 300-12** | Smart manufacturing use of STEP, QIF, MTConnect together | Reference architecture for digital thread |
| **SAMULET Ontology** | High-level manufacturing ontology for aerospace | Defines taxonomy for machine tools, cutting tools, fixtures, workpieces |
| **Digital Twin Consortium Manufacturing Ontologies** | ISA-95-based factory ontology | Reference solution for digital twin data models |
| **DTDL (Digital Twin Definition Language)** | Microsoft's ontology language for IoT/digital twins | Machine and process modeling |
| **OPC UA** | Secure industrial communication protocol | Carries ISA-95 metadata for machine-to-ERP data flow |
| **CMSD (Core Manufacturing Simulation Data)** | SISO standard for manufacturing simulation | Defines entities for machines, jobs, parts, resources |
| **SCOR (Supply Chain Operations Reference)** | APICS supply chain model | Defines plan, source, make, deliver, return processes |
| **APQP / PPAP** | Advanced Product Quality Planning | Production Part Approval Process (more automotive, but adopted by some aero primes) |

---

## 7. Proposed Ontology Entity Summary

Based on the domain analysis above, the following are the core entity classes for an aerospace precision manufacturing ERP ontology:

### Core Entities

```
ITEM (Part/Product)
  - PartNumber, Revision, DashNumber, Description
  - DrawingNumber, ModelReference
  - CAGECode, DesignAuthority
  - ExportControlClassification (ITAR/EAR/none)
  - ItemType (Make/Buy/RawMaterial/OutsideProcess/Phantom/Tooling/Consumable)
  - UnitOfMeasure
  - Status (Active/Obsolete/Prototype)

MATERIAL_SPECIFICATION
  - SpecNumber (e.g., "AMS 5659")
  - SpecBody (SAE, ASTM, MIL, etc.)
  - AlloyName, UNSNumber
  - MaterialForm (bar, plate, sheet, forging, casting)
  - Type, Class, Grade (within the spec)
  - Conditions (allowed heat treat conditions)
  - ChemistryLimits, MechanicalPropertyLimits

MATERIAL_LOT
  - LotNumber, HeatNumber
  - MaterialSpec reference
  - Condition/Temper
  - MeltCountry, ManufactureCountry
  - DFARSCompliant (boolean)
  - ConflictMineralStatus
  - Quantity, UOM
  - CertificationDocuments (MTR, CoC, PMI report)
  - ShelfLifeExpiration
  - SupplierReference

BOM (Bill of Materials)
  - ParentItem, ChildItem
  - Quantity, UOM, FindNumber
  - EffectivityRange (serial or date)
  - AlternateItems
  - Notes

ROUTING (Manufacturing Process Plan)
  - Item reference
  - Revision

OPERATION (within a Routing)
  - OperationNumber, OperationCode
  - WorkCenter reference
  - SetupTime, CycleTime, QueueTime
  - IsSpecialProcess (boolean)
  - ProcessSpecification reference
  - NadcapRequired (boolean)
  - OutsideVendor reference (if external)
  - Instructions, ProgramNumber
  - InspectionRequirements
  - ToolingRequirements

WORK_CENTER
  - Name, Code
  - Type (CNC Mill, CNC Lathe, Grinder, Inspection, Assembly, Outside)
  - MachineList
  - Capabilities (max travel, tolerance capability, materials handled)
  - CostRate (setup rate, run rate, overhead rate)

JOB / WORK_ORDER
  - JobNumber, Status
  - Item reference (what is being made)
  - Quantity ordered, Quantity completed, Quantity scrapped
  - SalesOrder reference
  - CustomerReference
  - Priority
  - DueDate, StartDate
  - MaterialLot references (raw material consumed)
  - OperationInstances (actual routing with actuals)

CUSTOMER
  - Name, Address, CAGE code
  - QualityRequirements (AS9100, Nadcap, OEM-specific)
  - ExportControlStatus
  - ApprovedProcessorLists
  - FlowdownClauseDefaults
  - PricingAgreements, Contracts

SUPPLIER
  - Name, Address, CAGE code, DUNS
  - Capabilities/Commodities
  - Certifications (ISO, AS9100, Nadcap scopes)
  - CustomerApprovals (which OEMs approve them)
  - ITARRegistered (boolean)
  - PerformanceMetrics
  - ApprovalStatus (Approved/Conditional/Probationary/Disqualified)
  - AuditHistory

PURCHASE_ORDER
  - PONumber, Date, Revision
  - Supplier reference
  - LineItems
  - QualityClauseCodes
  - ExportControlMarking
  - ContractFlowdowns

PO_LINE_ITEM
  - PartNumber or MaterialSpec or ProcessSpec
  - Quantity, UOM, Price
  - DeliveryDate
  - CertificationRequirements
  - DFARSRequired
  - SpecificationCallouts

SALES_ORDER
  - OrderNumber, Date
  - Customer reference
  - Contract/Program reference
  - LineItems (Part, Qty, Price, DueDate)
  - ExportControlClassification
  - FlowdownRequirements

NONCONFORMANCE_REPORT (NCR)
  - NCRNumber, Date
  - Part, Job, Operation references
  - Description, DrawingRequirement, ActualCondition
  - QuantityAffected
  - Severity (Critical/Major/Minor)
  - Disposition (UseAsIs/Rework/Repair/Scrap/RTV)
  - MRBSignatures
  - RootCauseAnalysis
  - CorrectiveAction reference
  - CustomerNotification (if required)

FIRST_ARTICLE_INSPECTION (FAI)
  - PartNumber, Revision
  - FAIStatus (Open/Submitted/Approved/Rejected)
  - Reason (NewPart/DesignChange/ProcessChange/Lapse)
  - Form1Data (part accountability)
  - Form2Data (material and process accountability)
  - Form3Data (characteristic measurements)
  - ApprovalSignatures

INSPECTION_RECORD
  - Job, Operation references
  - InspectionType (FirstPiece, InProcess, Final, Receiving)
  - Characteristics measured
  - ActualValues, PassFail
  - EquipmentUsed
  - InspectorReference
  - Date/Time

CALIBRATION_RECORD
  - Equipment reference
  - CalibrationDate, DueDate
  - Standard/Reference used
  - Results (Pass/Fail/Limited)
  - CertificateNumber
  - CalibrationVendor

CERTIFICATION_DOCUMENT
  - DocumentType (MTR, CoC, CoA, Nadcap cert, test report)
  - DocumentNumber
  - SourceSupplier
  - LinkedEntities (MaterialLot, PO, Job, Part)
  - ExpirationDate (if applicable)
  - FileReference (PDF/image storage)

PROCESS_SPECIFICATION
  - SpecNumber (e.g., "AMS 2759/3")
  - SpecBody
  - ProcessType (HeatTreat, ChemProcess, NDT, Plating, etc.)
  - ApplicableMaterials
  - NadcapCategory
  - Revision, Status
```

### Key Relationships

```
ITEM --has--> BOM (parent-child hierarchy)
ITEM --has--> ROUTING (one or more manufacturing routings)
ROUTING --contains--> OPERATION (ordered sequence)
OPERATION --performed-at--> WORK_CENTER
OPERATION --references--> PROCESS_SPECIFICATION
OPERATION --may-reference--> SUPPLIER (outside processing)

JOB --manufactures--> ITEM
JOB --consumes--> MATERIAL_LOT
JOB --follows--> ROUTING (creates operation instances)
JOB --linked-to--> SALES_ORDER
JOB --generates--> INSPECTION_RECORD
JOB --may-generate--> NCR
JOB --may-generate--> PURCHASE_ORDER (outside processing)

MATERIAL_LOT --conforms-to--> MATERIAL_SPECIFICATION
MATERIAL_LOT --has--> CERTIFICATION_DOCUMENT
MATERIAL_LOT --purchased-via--> PURCHASE_ORDER
MATERIAL_LOT --supplied-by--> SUPPLIER

ITEM --may-require--> FIRST_ARTICLE_INSPECTION
FAI --references--> MATERIAL_SPECIFICATION (Form 2)
FAI --references--> PROCESS_SPECIFICATION (Form 2)
FAI --contains--> INSPECTION_RECORD (Form 3 data)

NCR --references--> JOB, ITEM, OPERATION
NCR --may-generate--> CORRECTIVE_ACTION
NCR --disposition-by--> MRB_REVIEW

SUPPLIER --on--> APPROVED_SUPPLIER_LIST
SUPPLIER --holds--> CERTIFICATION_DOCUMENT (Nadcap, ISO, etc.)
SUPPLIER --approved-by--> CUSTOMER (OEM-specific approvals)
```

---

## 8. Data Governance Considerations

### Retention Requirements
- FAI records: Life of program + 3-7 years (varies by customer)
- Quality records: Typically 7-20 years
- ITAR records: 5 years minimum per DDTC
- Material certifications: Life of program
- Calibration records: Typically 2-3 calibration cycles

### Classification and Access Control
- ITAR data: U.S. persons only, physically and electronically controlled
- EAR data: Controlled based on classification level and destination
- Proprietary customer data: NDA-controlled, restricted access
- CUI (Controlled Unclassified Information): Per NIST SP 800-171 / CMMC

### Audit Trail Requirements
- All changes to quality records must be traceable (who, what, when, why)
- Deletion of quality records must be prevented or logged
- Electronic signatures must comply with 21 CFR Part 11 concepts (where applicable)

---

## Sources

- [SAE Aerospace Material Specifications](https://www.sae.org/standards/aerospace-material-specifications)
- [Visure Solutions - AMS Standards Explained](https://visuresolutions.com/aerospace-and-defense/ams-material-standards/)
- [AS9100 Certification Guide - Deltek](https://www.deltek.com/en/manufacturing/as9100-certification)
- [IA9100 Transition - CVG Strategy](https://cvgstrategy.com/ia9100-series-standards/)
- [OmegaCube - AS9100 and ERP](https://www.omegacube.com/blog/aerospace-defense-erp/as9100-certification-the-invisible-backbone-for-aerospace-defense-manufacturing-and-how-erp-makes-it-achievable/)
- [Quality Magazine - ISO 9001 and AS9100 Changes](https://www.qualitymag.com/articles/99324-iso-9001-in-2026-whats-changingand-how-as9100-ia9100-iatf-16949-nist-and-cmmc-fit-together)
- [Nadcap / Performance Review Institute](https://www.p-r-i.org/nadcap)
- [SAE AS9102 First Article Inspection](https://www.sae.org/standards/content/as9102/)
- [IAQG - 9102 Standard](https://iaqg.org/standard/9102-first-article-inspection-requirement/)
- [DFARS 252.225-7009 - Acquisition.gov](https://www.acquisition.gov/dfars/252.225-7009-restriction-acquisition-certain-articles-containing-specialty-metals.)
- [Visibility ERP - ITAR Compliance](https://www.visibility.com/blog/erp-helping-defense-manufacturers-meet-itar-compliance)
- [QIF Standards](https://qifstandards.org/about-qif/)
- [ISO 23952:2020 - QIF](https://www.iso.org/standard/77461.html)
- [NIST AMS 300-12 - Digital Thread](https://nvlpubs.nist.gov/nistpubs/ams/NIST.AMS.300-12.pdf)
- [ISA-95 Standard](https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard)
- [Rhize - ISA-95 as Manufacturing Ontology](https://rhize.com/blog/what-is-isa95/)
- [MESA International - B2MML](https://mesa.org/topics-resources/b2mml/)
- [iBASEt - Smart Manufacturing Integration Standards](https://www.ibaset.com/smart-manufacturing-integration-standards-opc-ua-step-oagis-isa95/)
- [Digital Twin Consortium - Manufacturing Ontologies](https://github.com/digitaltwinconsortium/ManufacturingOntologies)
- [Silvex - Understanding NADCAP Special Processes](https://www.silvexinc.com/about-us/news/understanding-nadcap-special-processes-in-aerospace-plating-and-finishing/)
- [Tulip - Material Review Board](https://tulip.co/blog/material-review-board/)
- [Precision Aerospace - Vendor Flowdowns](https://www.precisionaerospace.com/wp-content/uploads/2023/09/Vendor-Flow-Downs-20230305.pdf)
