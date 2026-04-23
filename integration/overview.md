# Integration Layer Overview

## The Role of the production system

The four standards — STEP AP242, MTConnect, QIF, and X12 EDI — define data formats and protocols. They do not define an ERP. They do not define a job router, a scheduler, a cost tracker, or a cert package assembler. They assume those systems exist and that translation layers bridge them.

At the manufacturer, **the production system** is that bridging system. It is the operational ERP and implicit MES for a precision CM shop: the system of record for jobs, operations, materials, serial numbers, vendors, inspections, and shipping. It is also already integrated with Claude via the the production system MCP server, making it the access point for the AI layer.

The integration layer documents in this directory describe how the production system maps to each standard — not abstractly, but concretely enough to implement:

| Document | What It Covers |
|----------|----------------|
| `fulcrum-mtconnect.md` | production job → MTConnect WorkOrder; UUID injection; machine state back into the production system |
| `fulcrum-qif.md` | production job → QIF inspection plan; CMM results → the production system inspection record; FAI close-out |
| `ip-boundary.md` | What data leaves the manufacturer in each direction; customer data classification; ITAR data residency |

---

## Mapping Philosophy

The integration layer makes three commitments:

**1. UUID persistence across translations.**
When a production job UUID is assigned to a job, that UUID appears in the MTConnect WorkOrder asset, in the QIF document header, and in X12 REF segments on the 856 ASN. It does not get translated, re-coded, or substituted at any boundary. The UUID is the persistent identity of the job across all systems. See `extensions/uuid-discipline.md`.

**2. the production system as system of record; standards as transport.**
Authoritative data lives in the production system Pro. MTConnect streams are ingested as telemetry — they inform the production system, they do not replace it. QIF results are imported into the production system inspection records — the QIF file is a transfer vehicle, not the persistent store. X12 transactions initiate production system records — the 850 creates a job, the 856 closes a shipment, but the production system is what makes them real.

**3. Gaps are documented, not papered over.**
Each integration document explicitly identifies what the production system captures that no standard provides, and what the standards provide that the production system must be extended to consume. The extensions in `extensions/` fill these gaps with defined entities. The integration docs explain how the data actually flows.

---

## The Three-Layer Data Flow

```
LAYER 1: STANDARDS (where data originates or is defined)

  Customer MBD package (STEP AP242)
    -> Geometry, PMI, GD&T callouts, material text

  Machine controller (MTConnect)
    -> Spindle data, axis positions, program state, execution events

  CMM software (QIF)
    -> Characteristic measurements, pass/fail, deviations

  Customer/supplier EDI (X12)
    -> Purchase orders, acknowledgments, advance ship notices, invoices

LAYER 2: FULCRUM PRO (system of record; orchestrates the thread)

  Job creation <- 850 PO data
  Item master <- STEP AP242 P/N, revision, CAGE
  Routing <- operator/engineer entry; STEP AP238 if available
  Material lot tracking <- receiving records; MTR/CoC attachments
  Machine assignment -> MTConnect WorkOrder UUID push
  Inspection plan <- QIF plan generation
  Inspection results <- QIF import
  Cert package assembly <- MTR + CoC + process certs + QIF results + FAI
  Shipment <- 856 ASN generation

LAYER 3: AI LAYER (Claude MCP, reads from the production system)

  Natural language queries over all Layer 2 data
  Anomaly detection (QIF failures -> NCR creation flag)
  Process correlation (MTConnect spindle load vs. QIF dimensional results)
  Cert package completeness checks pre-shipment
  Supplier approval validation at PO creation
```

---

## the production system Entity Map

The the production system entities that anchor the integration layer and their counterparts in the standards stack:

| the production system Entity | Standard Counterpart | Extension |
|----------------|---------------------|-----------|
| Job | MTConnect WorkOrder, QIF JobId header, X12 850 line item | — |
| Job Operation | MTConnect path/component, STEP AP238 machining operation | — |
| Serial Number | QIF SerialNumber header, X12 856 SN1 segment | uuid-discipline |
| Material Lot | (none in standards) | material-certs |
| Material Spec | STEP AP242 PMI text (unstructured) | material-certs |
| Process Spec | STEP AP238 reference, Nadcap scope | outside-processing |
| Outside Process Op | (gap — hybrid PO + routing op) | outside-processing |
| Vendor | (gap — no supplier registry in any standard) | supplier-approval |
| Inspection Plan | QIF Plans module | — |
| Inspection Record | QIF Results module | — |
| FAI Record | (gap — QIF has Form 3 only) | quality-flowdowns |
| NCR | (gap — not in QIF or any standard) | quality-flowdowns |
| Quality Flowdown Plan | X12 850 MSG text (unstructured) | quality-flowdowns |
| Cert Document | (gap — no standard for cert manifest) | material-certs |
| Shipment | X12 856 ASN | — |
| Invoice | X12 810 | — |

---

## Implementation Sequence

The integration layer is built in parallel with Phase 1–4 of the implementation plan:

**Phase 1 (Now — 3 months): MTConnect**
- Deploy MTConnect agents on priority Mazak machines
- Configure the production system → MTConnect UUID push (WorkOrder asset population)
- Verify telemetry is being captured; link machine state events to production job records
- See `fulcrum-mtconnect.md`

**Phase 2 (3–6 months): QIF**
- Configure CMM software (Renishaw/Hexagon/Zeiss) to export QIF results files
- Add JobId and SerialNumber UUID extension fields to QIF header
- Build the production system QIF ingestion: parse results, link to inspection plan, update pass/fail status
- Use QIF as basis for FAI Form 3 population
- See `fulcrum-qif.md`

**Phase 3 (6–9 months): STEP AP242**
- Build STEP PMI extraction pipeline (for customers who supply MBD packages)
- Link extracted GD&T characteristics to QIF inspection plan characteristics via UUID
- Feed material callout text into material_specification entity in the production system
- Manual entry fallback for 2D-PDF-only customers

**Phase 4 (9–12 months): X12 EDI**
- Implement inbound 850 processing (auto-create job in the production system from PO)
- Implement outbound 856 ASN (auto-generate from the production system shipment record)
- Carry production job UUID in 856 REF*ZZ segment
- Implement 810 invoice generation from the production system invoice record

**Phase 5: Publish**
- Document confirmed results from operating system
- Submit to journal (JMSE, SME Journal, or similar)
- Reference academic contacts: Dr. Moneer Helu (ARLIS/UMD), Dr. Martin Hardwick (RPI), William Sobel (MTConnect Institute)
