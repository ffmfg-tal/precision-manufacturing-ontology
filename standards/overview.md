# Digital Thread Standards Foundation

This document describes the four standards the manufacturer adopts as the foundation of its digital thread implementation. These standards form the reference architecture for digital thread in discrete manufacturing, as described in the NIST Advanced Manufacturing Series (NIST AMS 300-1, 2017) and the 2024 MTConnect roadmap (NIST GCR.24-057).

The manufacturer does not reinvent what these standards define. the manufacturer implements them, connects them, and extends them specifically for the precision manufacturer layer — which is the gap the existing standards do not address.

---

## The Four-Tier Stack

| Standard | Maintained By | Role | What It Defines |
|----------|--------------|------|-----------------|
| **STEP AP242** (ISO 10303-242) | ISO TC184/SC4 | Design / MBD | "What we want to make" — part geometry, PMI, GD&T, tolerances, material callouts, assembly structure |
| **STEP AP238** (ISO 10303-238) | ISO TC184/SC4 | Manufacturing process | CNC machining process plans, STEP-NC, tool paths, setup instructions |
| **MTConnect** (ANSI/MTC) | AMT / MTConnect Institute | Machine telemetry | "How we made it" — real-time and historical machine data streams: spindle speed, feed rate, axis position, program execution, alarms |
| **QIF** (ISO 23952 / ANSI QIF) | DMSC | Quality / metrology | "How close the real part is to ideal" — inspection plans, CMM measurement results, statistical process data |

**X12 EDI** (ASC X12) handles the commercial layer: purchase orders, advance shipping notices, invoices, and forecast signals. It is the transactional envelope that moves the artifact through the supply chain commercially, while the four standards above define the artifact's technical identity.

---

## How the Standards Interoperate

A single manufactured part crosses all four standards in sequence:

```
PRIME / CUSTOMER
  |
  | sends STEP AP242 package:
  |   - part geometry (B-rep solid model)
  |   - PMI annotations (GD&T, surface finish, notes)
  |   - material callouts (spec, alloy, form, temper)
  |   - drawing references, revision, effectivity
  v
The manufacturer PLANNING
  |
  | produces machining process plan:
  |   - STEP AP238 / internal routing
  |   - operation sequence, setups, fixturing
  |   - CNC programs referencing STEP geometry
  v
The manufacturer SHOP FLOOR
  |
  | machines the part:
  |   - MTConnect streams machine data
  |   - spindle hours, feeds/speeds, program execution, alarms
  |   - linked to job UUID and serial number
  v
The manufacturer INSPECTION
  |
  | measures conformance:
  |   - QIF inspection plan (derived from STEP AP242 PMI)
  |   - CMM results (actual vs. nominal, pass/fail per characteristic)
  |   - linked to same job UUID and serial number
  v
The manufacturer SHIPPING
  |
  | transacts commercially:
  |   - X12 856 Advance Shipping Notice
  |   - X12 810 Invoice
  |   - cert package assembled alongside
  v
PRIME / CUSTOMER
```

The artifact — a physical part — maintains a single persistent identity across all five stages. The mechanism is **UUID discipline** (see `extensions/uuid-discipline.md`).

---

## UUID as Connective Tissue

The four standards do not share a common identity scheme. STEP uses instance IDs within a file. MTConnect uses device/component identifiers. QIF assigns UUIDs per its own schema. X12 uses PO numbers and line item references. Without an external discipline, the same physical part appears as four unrelated objects in four separate systems.

the manufacturer's approach: assign persistent UUIDs at artifact creation in the production system Pro (the Layer 2 integration anchor) and carry those UUIDs into every downstream system as the primary cross-reference key.

This mirrors the approach validated in federated digital thread research (Hardwick et al., RPI / DMDII) and is a precondition for the AI-native query layer to reason across domains.

---

## The CM-Layer Gap

The four-tier stack was designed for primes and large Tier 1 integrators with:
- Full PLM systems (Siemens NX, PTC Creo, CATIA) generating and consuming STEP AP242
- Dedicated MES systems managing shop floor execution
- Internal quality labs with QIF-native CMM software
- Large IT organizations maintaining EDI gateways

**Precision manufacturers operate without any of these.** A typical aerospace CM receives a 2D drawing or PDF (not a STEP package), tracks jobs in an ERP without a PLM, manages inspection with paper travelers and CMM reports, and handles EDI through an intermediary or not at all.

The gap the manufacturer is filling — documented in the `extensions/` directory — is not in the standards themselves but in the layer between the standards and real CM operations:

- Material lot traceability and certifications are not defined in STEP or QIF
- Quality clause flowdowns (customer→manufacturer→sub-tier) have no standard representation
- Outside processing (special processes via Nadcap-certified vendors) is a hybrid operation+PO not addressed in STEP AP238 or X12
- Supplier qualification (Nadcap scope, ASLs, OEM approvals) is invisible to all four standards
- First Article Inspection per AS9102 requires data from both STEP (design intent) and QIF (measurement) but is defined in neither

The `extensions/` directory defines the manufacturer's solutions to each of these gaps, building on the standards foundation rather than replacing it.

---

## Canonical References

- **NIST AMS 300-1** (2017) — "Reference Architecture for Smart Manufacturing Part 1: Functional Models." NIST Advanced Manufacturing Series. The primary reference for how STEP/MTConnect/QIF/ISA-95 fit together in the digital thread.
- **NIST GCR.24-057** (2024) — "MTConnect: Enabling the Digital Thread for Manufacturing." Sobel, W. et al. Policy-level gap analysis; affirms that manufacturer-layer implementation gaps remain unaddressed.
- **ISO 10303-242:2022** — STEP AP242, Edition 2. "Managed model-based 3D engineering."
- **ISO 10303-238:2007** — STEP AP238, Edition 2. "Application interpreted model for computer numeric controllers."
- **ANSI/MTC1.1** onward — MTConnect standard, current version 2.2 (2024). Published by AMT / MTConnect Institute.
- **ISO 23952:2020** — Quality Information Framework (QIF), equivalent to ANSI QIF 3.0. Published by DMSC.
- **ISA-95 / IEC 62264** — Enterprise-control integration; Part 2 defines object model attributes for resource management. Referenced for Inventory and Manufacturing domains.
