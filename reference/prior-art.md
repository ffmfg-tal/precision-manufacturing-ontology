# Standards Prior Art Review

This document establishes what existing standards cover and where confirmed gaps exist at the precision manufacturer (CM) layer. It is the "we checked" foundation for the gap-filling work in the `extensions/` directory.

---

## Summary

The NIST digital thread reference architecture (NIST AMS 300-1, 2017) specifies four standards as the technical backbone of digital thread in discrete manufacturing: STEP AP242 (design), STEP AP238 (manufacturing process), MTConnect (machine telemetry), and QIF (quality/metrology). X12 EDI provides the commercial transaction layer.

These standards are mature, well-maintained, and widely implemented — at primes and large Tier 1 integrators. **None of them addresses the precision manufacturer layer directly.** The gaps documented here are not theoretical; they are operational realities at the shop confirmed through implementation experience.

---

## Standard-by-Standard Review

### STEP AP242 (ISO 10303-242) — Design / MBD

**Document:** ISO 10303-242:2022, Edition 2, "Managed model-based 3D engineering." Maintained by ISO TC184/SC4. Implemented in CAD/PLM systems (Siemens NX, PTC Creo, CATIA, SolidWorks).

**What it covers:**
- B-rep solid geometry and tessellated geometry for manufactured parts
- Product and Manufacturing Information (PMI): GD&T, surface finish, notes, reference designators
- Geometric dimensions and tolerances (GD&T) per ASME Y14.5 and ISO 1101
- Material callouts (alloy, specification reference) embedded in PMI annotations
- Bill of Materials (BOM) structure, assembly relationships
- Part identification: part number, revision, CAGE code, effectivity
- Drawing references and model-based definition (MBD) replacing 2D drawings
- Product configuration management: variants, options, effectivity dates

**What it does NOT cover (manufacturer gaps):**
- Material lot traceability — STEP defines what material *should* be used; it does not track what material *was* used, from which heat/lot, with which certifications
- Mill Test Reports (MTR) and Certificates of Conformance (CoC) — no schema for material certification documents
- DFARS and conflict mineral compliance declarations — not in scope
- AMS/ASTM material specifications as queryable entities — material callouts are PMI text strings, not structured first-class objects linked to a specification database
- The difference between "drawing says 7075-T651" and "we have this specific lot in inventory with these certs" is invisible to STEP

**What the shop extends:** `extensions/material-certs.md` defines the material certification chain that bridges the STEP material callout to the physical lot and its documentation.

---

### STEP AP238 (ISO 10303-238) — Manufacturing Process / STEP-NC

**Document:** ISO 10303-238:2007, Edition 2, "Application interpreted model for computer numeric controllers." Also known as STEP-NC. Maintained by ISO TC184/SC4.

**What it covers:**
- CNC machining process plans: operations, setups, tool paths, cutting parameters
- Milling, turning, drilling, EDM, and multi-function machining operation types
- Workingstep sequencing and operation dependencies
- Tool definitions: geometry, material, coating, expected life
- Machine capability specifications: axis configuration, spindle range, tool changer capacity
- Links to STEP AP242 geometry: operations reference features defined in the part model

**What it does NOT cover (manufacturer gaps):**
- Special processes — heat treatment, plating, anodizing, NDT, welding — are not manufacturing operations in STEP AP238's model. These are performed at outside vendors, not on CNC machines.
- Outside processing as a supply chain entity — sending a part to a Nadcap-certified vendor and receiving it back with a cert package is a logistics and quality event, not a machining operation. No representation in STEP AP238.
- Work instruction content beyond machine-readable parameters — the traveler, first article setup notes, operator sign-offs, inspection hold points
- Job-level traceability — STEP AP238 defines process plans, not production instances. It doesn't distinguish "the plan for P/N 7832" from "job 25-0412 which made 50 of P/N 7832 in March"

**What the shop extends:** `extensions/outside-processing.md` defines the outside processing entity (hybrid operation + PO + cert collection) that STEP AP238 omits.

---

### MTConnect (ANSI/MTC) — Machine Telemetry

**Document:** ANSI/MTC1.1 through ANSI/MTC1.6, current version 2.2 (2024). Maintained by AMT and the MTConnect Institute. Open-source standard; agent and adapter implementations available at github.com/mtconnect.

**What it covers:**
- Real-time machine data streaming via HTTP/XML
- Data items: execution state, controller mode, program execution, axis positions, spindle speed, feed rate, load, temperature, power, message/alarm events
- Asset documents: cutting tools (geometry, life, usage), device configuration, work orders, raw materials (at schema level)
- Work order tracking: `<WorkOrder>` element links machine data to a job/serial reference
- Tool life management: usage counts, wear metrics, replacement triggers
- Condition monitoring: alarm states, fault codes, hardware error events
- Device topology: machine components, axes, controllers, spindles, rotaries

**What it does NOT cover (manufacturer gaps):**
- Inspection data — MTConnect monitors machines; it does not measure parts. CMM data, surface roughness, dimensional results are outside its scope.
- Material tracking — MTConnect has a `<RawMaterial>` asset element but it carries only what the machine knows (stock dimensions); it does not carry heat numbers, lot numbers, or cert references.
- Quality hold decisions — MTConnect reports process data; it does not implement a pass/fail disposition on a part.
- Outside processing — special process operations do not produce MTConnect data (they happen at external vendors on different equipment).
- Job cost tracking — MTConnect can report spindle time but does not roll up labor, material, and outside process costs into a job record.

**What the shop implements:** `integration/fulcrum-mtconnect.md` defines how the manufacturer's production job UUID flows into the MTConnect `<WorkOrder>` reference and how machine data is retrieved and linked back to jobs via Claude MCP queries against the production system data.

**the manufacturer's machine fleet producing MTConnect data:** Mazak Integrex, UMC series, DVF series, NL series, Puma series; Okuma (adapter configuration required).

---

### QIF (ISO 23952 / ANSI QIF) — Quality / Metrology

**Document:** ISO 23952:2020, equivalent to ANSI QIF 3.0. Maintained by the Digital Metrology Standards Consortium (DMSC). XML-based schema with seven primary modules.

**What it covers:**
- **QIF Plans:** Measurement plans derived from design intent (STEP AP242 PMI). Defines which characteristics to measure, with what equipment, in what sequence.
- **QIF Results:** Actual measurement data: nominal value, measured value, deviation, tolerance band, pass/fail status per characteristic.
- **QIF Statistics:** SPC data — Cp, Cpk, histograms, control charts, out-of-control signals.
- **QIF MBD:** Model-based definition linkage — associating measurement plans to geometry in the part model.
- **QIF Resources:** Measurement equipment definitions — CMM type, probe configuration, calibration status, measurement uncertainty.
- **QIF Rules:** Measurement strategy rules and tolerancing interpretations.
- **UUID identity:** Every characteristic, measurement, and result in QIF carries a UUID, enabling precise cross-reference.

**What it does NOT cover (manufacturer gaps):**
- First Article Inspection per AS9102 — FAI requires three specific forms (Part Number Accountability, Product Accountability, Characteristic Accountability) that integrate material/process compliance data alongside dimensional results. QIF covers Form 3 (characteristic measurement) only; Forms 1 and 2 are outside its scope.
- Nonconformance reporting (NCR) — QIF documents what was measured; it does not define what happens when a part fails. MRB disposition (use-as-is, rework, repair, scrap) is not in QIF.
- Material certification integration — QIF results do not reference the MTR or CoC for the material the part was made from.
- Special process certification — a QIF results file for a CMM measurement does not include the Nadcap cert from the heat treat vendor or the FPI report.
- In-process quality data — inspection hold points during machining (mid-operation dimensional checks, go/no-go gage results) are not CMM events and are not captured in QIF.
- Rework tracking — when a part is reworked and re-inspected, QIF does not natively model the "original inspection → rework → re-inspection" lifecycle.

**What the shop extends:** `extensions/quality-flowdowns.md` defines the AS9102 FAI structure. NCR/MRB workflow will be addressed in a future extension.

---

### X12 EDI — Commercial Transactions

**Document:** ASC X12 transaction sets. Aerospace-specific implementation guides published by AIA (Aerospace Industries Association) as X12-004010-AIA-E4 and successors. Maintained by ASC X12.

**Relevant transaction sets:**
- **X12 850** — Purchase Order (customer sends to the manufacturer)
- **X12 855** — Purchase Order Acknowledgment (manufacturer confirms)
- **X12 856** — Advance Ship Notice / Bill of Lading (manufacturer sends on shipment)
- **X12 810** — Invoice (manufacturer sends for payment)
- **X12 830** — Planning Schedule / Forecast (customer sends for capacity planning)

**What it covers:**
- Commercial terms: prices, quantities, delivery dates, payment terms, ship-to/bill-to
- Item identification: customer part number, revision, CAGE code, unit of measure
- Shipment details: carrier, tracking, packaging, serial/lot numbers at line item level
- Invoice reconciliation: PO line item to invoice amount mapping
- Schedule forecasts: volume projections by week/period

**What it does NOT cover (manufacturer gaps):**
- Quality clause flowdowns — X12 850 has no standard mechanism for embedding AS9100 quality clauses (QA-001 through QA-017), Nadcap requirements, FAI status, certification package requirements, or source inspection requirements. These are communicated via attached PDFs, separate quality agreements, or notes fields — not machine-readable.
- Material specification requirements — the X12 850 carries a part number; it does not carry the AMS spec, DFARS declaration requirement, or conflict mineral status.
- Nadcap vendor requirements — which special processes are required, which Nadcap-certified vendors are approved by the customer, are not in the EDI transaction set.
- Outside processing purchase orders — when the manufacturer sends a part to a heat treat vendor, that PO is generated as a sub-tier transaction. The linkage from the customer's X12 850 to the manufacturer's outbound PO for outside processing is not standardized.
- Cert package content requirements — what must accompany the shipment (MTR, CoC, FAI, process certs, source inspection report) is not defined in X12.

**What the shop extends:** `extensions/quality-flowdowns.md` defines the machine-readable quality clause model that X12 omits. `extensions/outside-processing.md` defines the sub-tier PO structure for special processes.

---

### ISA-95 / IEC 62264 — Enterprise-Control Integration

**Document:** IEC 62264 series (equivalent to ISA-95). Part 1: models and terminology; Part 2: object model attributes; Part 3: activity models of manufacturing operations management. Maintained by ISA and IEC.

**What it covers:**
- Hierarchy model: L4 (ERP) through L0 (physical process), with MES at L3
- Resource models: personnel, equipment, material, process segments
- Production scheduling, dispatching, and execution data categories
- Work order lifecycle: scheduled → dispatched → executing → complete
- Material inventory: lot/batch tracking, material definitions, physical asset tracking

**What it does NOT cover (manufacturer gaps):**
- Multi-customer, multi-program isolation — ISA-95 assumes a single enterprise managing its own production. At a CM, every job has a different customer with different contractual requirements, export control classifications, and quality obligations. ISA-95 has no model for customer-specific data access controls or ITAR data segregation within the MES layer.
- Traveler-centric traceability — aerospace CM traceability is anchored to the physical traveler (a job packet that follows the part through every operation). ISA-95's work order model is digital-first; the traveler is not a first-class entity.
- DFARS and specialty metals compliance — not in scope.

**ISA-95's role in the manufacturer's architecture:** Referenced for Inventory domain (lot/batch tracking) and Manufacturing domain (work order and operation structure). the production system implements ISA-95 concepts for work orders and material tracking.

---

## Confirmed CM-Layer Gaps (Summary)

These are the gaps no existing standard addresses. Each has a corresponding extension document in this repo:

| Gap | Standards That Don't Cover It | the manufacturer Extension |
|-----|------------------------------|---------------|
| Material lot traceability to heat/cert chain | STEP AP242, QIF | `extensions/material-certs.md` |
| Outside processing (special process vendor ops) | STEP AP238, X12 | `extensions/outside-processing.md` |
| Quality clause flowdowns (customer→manufacturer→sub-tier) | X12, QIF | `extensions/quality-flowdowns.md` |
| Supplier qualification (Nadcap, ASL, OEM approval) | X12, QIF, STEP | `extensions/supplier-approval.md` |
| UUID persistence across format boundaries | All four (no shared scheme) | `extensions/uuid-discipline.md` |
| First Article Inspection per AS9102 Forms 1–2 | QIF (covers Form 3 only) | Referenced in `extensions/quality-flowdowns.md` |
| Traveler-centric traceability without PLM/MES | ISA-95, STEP AP238 | `integration/overview.md` |

---

## Key Academic References

- **NIST AMS 300-1** (2017) — Sobel, W. and Helu, M. "Reference Architecture for Smart Manufacturing Part 1." The primary reference for the four-tier digital thread stack.
- **NIST GCR.24-057** (2024) — Sobel, W. et al. "MTConnect: Enabling the Digital Thread for Manufacturing." Policy-level gap analysis confirming manufacturer-layer gaps remain unaddressed.
- **CIRP 2017** — Helu, M. et al. Empirical validation of the four-tier architecture in a machining context. University of Maryland / NIST Smart Manufacturing Systems Test Bed.
- **Hardwick, M. (RPI / DMDII)** — Federated digital thread UUID persistence research, DMDII Project 14-05-02. Presents the case that UUID discipline across STEP/MTConnect/QIF is a precondition for coherent digital thread.
