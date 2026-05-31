# STEP AP242 — Design and Model-Based Definition

**Standard:** ISO 10303-242:2022 (Edition 2), "Managed model-based 3D engineering"
**Maintained by:** ISO TC184/SC4 (Industrial Automation Systems and Integration)
**Status at the shop:** Receiving standard — the manufacturer ingests customer-supplied STEP AP242 packages; the manufacturer does not currently generate STEP output

---

## What the Standard Defines

STEP AP242 is the aerospace and defense industry standard for transmitting product definition data as a neutral format file (`.stp` or `.step`). It is the primary format for Model-Based Definition (MBD) — where the 3D model carries all the information previously split across a 3D model and a 2D drawing.

A STEP AP242 file from a prime customer may contain:

**Geometry:**
- B-rep (Boundary Representation) solid model: surfaces, edges, vertices
- Tessellated geometry: mesh representation for visualization
- 2D drawing views derived from the 3D model

**Product and Manufacturing Information (PMI):**
- Geometric Dimensions and Tolerances (GD&T) per ASME Y14.5 or ISO 1101
- Surface finish callouts (Ra, Rz values, measurement direction)
- Notes and general tolerances
- Datum reference frames
- Thread callouts
- Reference designators

**Product Identification:**
- Part number and revision
- CAGE code
- Drawing number and zone references
- Effectivity dates and configuration baseline

**Material Information (embedded in PMI text):**
- Material callout: alloy, specification (e.g., "7075-T7351 per AMS 4078")
- Heat treatment requirements
- Surface treatment requirements referenced on the model

**Assembly Structure:**
- Bill of Materials (BOM) embedded in multi-component STEP files
- Assembly relationships, mates, and constraints
- Instance-level configuration information

---

## What the shop Does with STEP AP242

Most the manufacturer customers still supply 2D PDF drawings rather than STEP AP242 packages. However, prime contractor customers and government programs increasingly supply MBD packages. the manufacturer's STEP AP242 pipeline:

### Ingestion
- Receive STEP file from customer (via customer portal, email, or EDI attachment)
- Open in CAM software (Mastercam, Fusion 360, or similar) for toolpath programming
- Extract PMI annotations for inspection planning
- Extract part identification (P/N, revision, CAGE) for production system item master

### PMI Extraction
The critical operation for digital thread linkage: extracting PMI from the STEP file in structured form. Key fields needed in the production system:
- All GD&T characteristics (dimension, tolerance, datum, characteristic type)
- Surface finish requirements by zone
- Material callout text (feeds `extensions/material-certs.md` workflow)
- Notes that become quality plan requirements

### Characteristic UUID Assignment
Each GD&T characteristic extracted from the STEP PMI gets a UUID that links:
- The design characteristic (from STEP AP242) → to the QIF measurement plan characteristic → to the QIF measurement result → to the the production system inspection record

This UUID chain is what makes it possible to answer: "For serial 7832-007, which dimensions were out of tolerance and what was the deviation?"

---

## CM-Layer Gaps

### Gap 1: the manufacturer Rarely Receives Full STEP Packages

Most customers still transmit 2D PDFs (drawings) rather than STEP AP242 files. The standards infrastructure exists but adoption at the manufacturer layer lags prime contractor adoption by years. the manufacturer's implementation must handle both:
- **STEP AP242:** extract characteristics programmatically
- **2D PDF:** manual data entry of characteristics into QIF-structured inspection plan

The gap is real: the "STEP → QIF" pipeline that works for Lockheed Martin's internal CMM lab requires PMI extraction tooling and STEP reader software that most CMs don't have.

### Gap 2: Material Callout Is Unstructured Text

STEP AP242 carries material information as PMI annotation text, e.g.:
```
MATERIAL: 7075-T7351 ALUMINUM ALLOY PER AMS 4078
HEAT TREAT: PER AMS 2770
TEST: UT PER AMS 2630 CLASS A GRADE C PRIOR TO MACHINING
```

This text is not a structured entity. STEP AP242 has no schema for:
- AMS specification as a queryable object (spec number, revision, commodity group)
- UNS number cross-reference
- Required test type and class
- Whether the material must be DFARS-compliant (domestic melt)

The `extensions/material-certs.md` document defines the structured material entity that bridges this gap — taking the unstructured STEP callout and creating a queryable, traceable record.

### Gap 3: No As-Built Tracking

STEP AP242 defines design intent — "the part *should* be made from 7075-T7351." It has no mechanism for recording:
- Which specific material lot was used
- The heat number, mill source, and melt country of that lot
- The MTR and CoC that document the lot's conformance

The traceability chain from design callout to physical material lot to certification documents is entirely outside STEP's scope.

### Gap 4: No Customer-Supplier Approval State

STEP AP242 carries a product definition, not its approval state. There is no field for:
- Whether the manufacturer is an approved source for this part (from the prime's ASL)
- Whether a First Article has been completed and approved for this P/N/revision
- Whether source inspection is required for this shipment

### Gap 5: PMI Is Evidence, Not Authority — and Geometry Must Anchor Inference

The MBD promise is that GD&T travels as machine-readable **semantic PMI** inside the model, so a system can parse tolerances straight from the file. In practice this is unreliable enough that the manufacturer layer must not architect around it:

- **Semantic vs graphical PMI.** Many AP242 files carry PMI as *graphical* annotations (text positioned in 3D space) rather than *semantic* PMI (structured, queryable, associated to faces). Graphical PMI is not extractable as data. The `cad_model.semantic_pmi_present` flag (`schemas/cad_model.yaml`) records which was delivered.
- **The drawing governs.** In most aerospace contracts the controlling document is the 2D drawing; the model is reference. When the two disagree, the drawing wins — recorded explicitly via the staged `part_specification.controlling_authority` field. Parsed PMI is therefore modeled as `assertion_method: Standard_Parsed` — *one corroborating source, usually low-confidence*, not the authoritative reading.
- **Geometry anchors inference; it does not author it.** The high-value, reliable extraction from a STEP file is *exact geometry*: a kernel reconstructs the B-rep and computes a feature's dimensions and recognizes its topology. An AI is excellent at *interpreting* (this face is a precision bore; it maps to drawing balloon 12) and unreliable at *measuring* (reading Ø12.70 off a render). So the architecture splits the work: the geometry kernel measures a feature's geometry (`assertion_method: Computed`), the model interprets and reads the drawing's *requirements* (`assertion_method: AI_Inferred` — because tolerances and GD&T are not in the geometry at all), and a human confirms (`verification_event`). The kernel is the **hallucination anchor**. This is why `cad_model` is a first-class entity (the parsed geometric-truth artifact), why a *feature's* geometric dimensions may never be `AI_Inferred`, and why a *characteristic's* requirements — which the geometry cannot supply — are `AI_Inferred` from the drawing and gated by human verification.

See `extensions/provenance-and-verification.md` and `reference/composition-archetypes.md` Walk 8 (Design Ingestion / CAM Programming) for the full treatment, and Decision 1.10 in `reference/taxonomic-decisions.md`.

---

## Integration Points

| STEP AP242 Data | Maps To | the production system Entity |
|-----------------|---------|----------------|
| Part number + revision | Item master | `item.part_number`, `item.revision` |
| CAGE code | Item master | `item.cage_code` |
| GD&T characteristics | Inspection plan | Job operation inspection records |
| Material callout text | Material spec entity | `material_specification` (see `extensions/material-certs.md`) |
| PMI characteristic UUID | QIF plan characteristic ID | Inspection event UUID chain |

---

## Tools and Resources

- **STEP file readers / geometry kernels:** Siemens JT2Go (free viewer), CADExchanger, **OpenCASCADE (OCCT, open source)** — the open geometry kernel that reconstructs a STEP file into a queryable B-rep. In the manufacturer pipeline this is what produces the `cad_model` artifact: exact surfaces and topology from which dimensions are *computed* (not inferred) and features are recognized. It is the hallucination anchor described in Gap 5 — the kernel measures, the model interprets, the human verifies.
- **PMI extraction:** NIST's MBD (Model-Based Definition) test artifacts — publicly available STEP AP242 files with known PMI for testing extraction pipelines: https://www.nist.gov/el/systems-integration-division/mbd-pmi-test-cases
- **MTConnect SysML model:** https://github.com/mtconnect/mtconnect-sysml-model (reference for device/component modeling that complements STEP AP242)
- **IOF collaboration on STEP ontology:** Industrial Ontology Foundry (IOF) is developing OWL ontologies that formalize STEP concepts; the manufacturer's material-certs extension should align with IOF's manufacturing reference ontology where possible
