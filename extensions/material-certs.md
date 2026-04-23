# Material Certification Extension

## The Gap This Fills

STEP AP242 carries material callouts as PMI text: `"7075-T7351 ALUMINUM ALLOY PER AMS 4078"`. QIF measures part dimensions. Neither standard has a schema for:

- The AMS/ASTM specification as a queryable, versioned object
- The physical material lot that was actually used (heat number, lot number, form, dimensions)
- The Mill Test Report (MTR) documenting actual chemistry and mechanical test results
- The Certificate of Conformance (CoC) from the distributor
- The DFARS specialty metals compliance declaration
- The chain connecting drawing callout → specification → lot → certifications → finished part

This extension defines that chain. It is the traceability backbone that makes aerospace shipments certifiable and DFARS-auditable.

---

## Entity Definitions

### Material Specification

> **Canonical schema:** [`schemas/material.yaml`](../schemas/material.yaml) — `material_specification` entity

A published standard defining the chemistry, mechanical properties, test requirements, and acceptance criteria for a material family.

```yaml
material_specification:
  id: uuid                          # UUID v4
  spec_number: string               # e.g., "AMS 4078"
  spec_body: enum                   # AMS | ASTM | MIL-SPEC | UNS | OEM | Other
  title: string                     # e.g., "Aluminum Alloy, Plate and Sheet, 7.6Zn..."
  revision: string                  # e.g., "L" or "2021"
  spec_category: enum               # Material | Process | Test Method
  commodity_group:                  # AMS commodity group (if AMS)
    range: string                   # e.g., "4000-4999 Non-ferrous"
  alloy_common_name: string         # e.g., "7075"
  alloy_family: enum                # Aluminum | Titanium | Steel | Nickel | Copper | Refractory | Composite
  uns_number: string                # e.g., "A97075"
  forms_covered:                    # which material forms this spec applies to
    - enum: Bar | Plate | Sheet | Forging | Casting | Tube | Wire | Extrusion | Powder
  conditions_defined:               # heat treatment / temper conditions covered
    - condition: string             # e.g., "T7351"
      min_tensile_ksi: number
      min_yield_ksi: number
      min_elongation_pct: number
      hardness_range: string
  cross_references:
    - spec_number: string           # related specs (e.g., ASTM A564 for 17-4 PH)
      relationship: enum            # Equivalent | Superseded_By | Supersedes | Alternative
  dfars_applicable: boolean         # whether this material is covered by DFARS 252.225-7009
  nadcap_process_required: string   # if heat treatment or processing requires Nadcap (e.g., "HT")
  status: enum                      # Active | Superseded | Withdrawn
```

**AMS Specification System Structure:**

| Range | Commodity |
|-------|-----------|
| AMS 2000–2999 | Process specifications (heat treat, plating, testing) |
| AMS 4000–4999 | Non-ferrous metals (aluminum, titanium, copper, nickel) |
| AMS 5000–5999 | Ferrous metals (steel, stainless, superalloys) |
| AMS 6000–6999 | Steels (alloy, tool) |
| AMS 7000–7999 | Refractory metals, advanced materials |

**Common Aerospace Material Specifications:**

*Aluminum Alloys:*
| Alloy | UNS | Spec (Plate) | Common Tempers |
|-------|-----|--------------|----------------|
| 7075 | A97075 | AMS 4078 | T7351 |
| 7075 | A97075 | AMS 4045 | T651 |
| 2024 | A92024 | AMS 4035 | T351 |
| 6061 | A96061 | AMS 4025 | T651 |

*Precipitation Hardening Stainless Steels:*
| Alloy | UNS | Spec (Bar) | Conditions |
|-------|-----|-----------|-----------|
| 15-5 PH | S15500 | AMS 5659 | H900, H1025, H1100, H1150 |
| 17-4 PH | S17400 | AMS 5643 | H900, H925, H1025, H1075, H1150 |
| 13-8 Mo | S13800 | AMS 5629 | H950, H1000, H1050 |

*Nickel Superalloys:*
| Alloy | UNS | Spec (Bar) | Condition |
|-------|-----|-----------|----------|
| Inconel 625 | N06625 | AMS 5666 | Annealed |
| Inconel 718 | N07718 | AMS 5662 (bar) / AMS 5663 (bar, premium) | Solution + aged |
| Waspaloy | N07001 | AMS 5704 (bar) | Solution + aged |

*Titanium Alloys:*
| Alloy | UNS | Spec (Bar) | Condition |
|-------|-----|-----------|----------|
| Ti 6Al-4V | R56400 | AMS 4928 | Annealed |
| Ti 6Al-4V ELI | R56401 | AMS 4930 | Annealed |

*Alloy Steels:*
| Alloy | UNS | Spec (Bar) |
|-------|-----|-----------|
| 4340 | G43400 | AMS 6414 |
| 300M | K44220 | AMS 6419 |
| 4130 | G41300 | AMS 6350 |

---

### Material Lot

> **Canonical schema:** [`schemas/material.yaml`](../schemas/material.yaml) — `material_lot` entity

A specific quantity of material purchased from a specific supplier, traceable to a specific melt heat, with documented conformance.

```yaml
material_lot:
  id: uuid                          # UUID v4 — minted by the production system at receiving
  lot_number: string                # shop internal lot tracking number
  heat_number: string               # Mill's heat/melt identifier (on MTR)
  material_spec: uuid               # -> material_specification.id
  supplier: string                  # Distributor name and CAGE code
  mill_source: string               # Mill name (e.g., "Kaiser Aluminum", "Carpenter Technology")
  melt_country: string              # Country of melt (ISO 3166-1 alpha-2)
  manufacture_country: string       # Country of manufacture/conversion (may differ from melt)
  dfars_compliant: boolean          # 252.225-7009 qualified — domestic melt AND manufacture
  melt_practice: enum               # VIM | VAR | VIM-VAR | ESR | CEVM | Air_Melt
  form: enum                        # Bar | Plate | Sheet | Forging | Tube | Extrusion
  condition: string                 # e.g., "T7351", "H1025", "Annealed"
  dimensions:
    # Form-dependent; examples:
    # Plate: thickness, width, length
    # Bar (round): diameter, length
    # Bar (flat): thickness, width, length
  quantity_received: number
  quantity_on_hand: number
  uom: string                       # "in", "mm", "lb", "kg"
  po_reference: uuid                # -> purchase_order.id
  location: uuid                    # -> inventory_location.id
  receiving_date: date
  shelf_life_expiration: date       # if applicable
  pmi_verified: boolean             # XRF or OES positive material identification performed
  status: enum                      # Available | Reserved | InUse | Quarantined | Consumed | Rejected
  certifications:
    - uuid                          # -> certification_document.id (MTR, CoC, etc.)
```

---

### Certification Document

> **Canonical schema:** [`schemas/certification_document.yaml`](../schemas/certification_document.yaml) — `certification_document` entity (discriminated supertype)
> Per Decision 1.6: MTR, CoC, DFARS_Declaration, PMI_Report, ConflictMineral, ShelfLife, ExportCompliance are `cert_type` values within this entity, not standalone entities.

A document issued by a supplier, mill, or test lab certifying material conformance to a specification.

```yaml
certification_document:
  id: uuid                          # UUID v4
  cert_type: enum                   # MTR | CoC | DFARS_Declaration | PMI_Report | ConflictMineral | ShelfLife
  issuer: string                    # Company issuing the cert
  issue_date: date
  material_lot: uuid                # -> material_lot.id
  spec_reference: uuid              # -> material_specification.id
  document_number: string           # Issuer's internal cert number
  file_reference: string            # Path or URL to cert document image/PDF

  # For MTR (Mill Test Report):
  mtr_fields:
    heat_number: string
    chemistry_actual:               # Actual element percentages from spectrographic analysis
      - element: string
        percent: number
    chemistry_limits:               # Spec limits for comparison
      - element: string
        min_pct: number
        max_pct: number
    mechanical_tests:
      - test_type: enum             # Tensile | Yield | Elongation | HardnessRB | HardnessRC | Impact
        direction: enum             # L | LT | ST (longitudinal, long transverse, short transverse)
        result: number
        units: string
        spec_min: number
    melt_practice: string
    product_form: string
    heat_treat_condition: string

  # For DFARS Declaration:
  dfars_fields:
    clause: string                  # "252.225-7009" or "252.225-7014"
    melt_country: string
    manufacture_country: string
    declarant_name: string
    declarant_title: string
    declaration_date: date

  # For Conflict Minerals (3TG per 252.225-7052):
  conflict_mineral_fields:
    clause: string                  # "252.225-7052"
    minerals_covered: [string]      # ["Tantalum", "Tin", "Tungsten", "Gold"]
    conflict_free: boolean
    country_of_origin: string
```

---

### Material Traceability Chain

The complete traceability chain from drawing callout to finished part:

```
DRAWING (STEP AP242 PMI or 2D PDF)
  | callout: "7075-T7351 PER AMS 4078, DFARS REQUIRED"
  v
MATERIAL SPECIFICATION ENTITY
  | AMS 4078, rev L, 7075 plate, T7351 condition
  | DFARS applicable: yes
  v
PURCHASE ORDER (to distributor)
  | spec: AMS 4078, condition: T7351, form: plate
  | DFARS clause 252.225-7009 required
  v
MATERIAL LOT (received from distributor)
  | lot: MFG-2025-0088, heat: J7291
  | melt_country: USA, dfars_compliant: true
  | pmi_verified: true (XRF at receiving)
  v
CERTIFICATIONS (attached to lot)
  | MTR: Kaiser Aluminum, chemistry + mechanical results
  | CoC: Distributor CoC referencing heat J7291
  | DFARS Declaration: domestic melt + manufacture
  v
JOB MATERIAL ISSUE
  | job 25-0412, op 10 (saw cut), issued qty: 3.25" x 12.5" x 14.5"
  | lot J7291 UUID linked to job UUID
  v
FINISHED PART (serial 7832-007)
  | serial UUID linked to material lot UUID
  v
CERT PACKAGE (assembled at shipment)
  | includes: MTR, CoC, DFARS declaration
  | serial 7832-007 → lot J7291 → heat J7291 → Kaiser mill
```

---

### Heat Treatment Specifications

Material heat treatment is governed by separate process specifications (AMS 2759 series for steel, AMS 2770 for aluminum, AMS 2774 for titanium). These are not captured in STEP AP242 material callouts with enough structure to validate the process.

**AMS 2759 Series (Steel Heat Treatment):**

| Spec | Covers |
|------|--------|
| AMS 2759 | Base requirements — general |
| AMS 2759/1 | Carbon and low-alloy steel |
| AMS 2759/2 | Special low-alloy steel (high hardenability) |
| AMS 2759/3 | PH corrosion-resistant steel (15-5, 17-4, 13-8, Custom 455) |
| AMS 2759/4 | Austenitic corrosion-resistant steel (321, 347, A286) |
| AMS 2759/5 | Martensitic corrosion-resistant steel (410, 440C) |
| AMS 2759/7 | Carburizing and carbonitriding |
| AMS 2759/9 | Hardness/conductivity inspection for aluminum |
| AMS 2759/11 | Stress relief of steel |

**Other Heat Treatment Specs:**
- AMS 2770: Aluminum alloys (all tempers, aging treatments)
- AMS 2774: Titanium alloys
- AMS 2801: Nickel alloys
- AMS 2761: PH stainless (older, still referenced on legacy drawings)

**PH Stainless Condition Reference (17-4 PH per AMS 5643 / AMS 2759/3):**

| Condition | Age Temp (°F) | Time | Min UTS (ksi) | Min YS (ksi) | Typical HRC |
|-----------|--------------|------|--------------|-------------|-------------|
| Condition A | — | — | — | — | Solution annealed (soft) |
| H900 | 900 | 1 hr | 190 | 170 | 40–47 |
| H925 | 925 | 4 hr | 170 | 155 | 38–45 |
| H1025 | 1025 | 4 hr | 155 | 145 | 35–42 |
| H1075 | 1075 | 4 hr | 145 | 125 | 31–38 |
| H1150 | 1150 | 4 hr | 135 | 105 | 28–37 |
| H1150M | 1150/4hr + 1150/4hr | — | 115 | 75 | 24–33 |

---

## Integration with the Digital Thread

| Material Cert Data | From | Linked To |
|-------------------|------|-----------|
| Material spec UUID | the production system material spec entity | STEP AP242 PMI callout (text match) |
| Material lot UUID | the production system receiving record | Job material issue, serial number |
| MTR heat number | Mill cert document | Material lot, DFARS declaration |
| DFARS compliant flag | DFARS declaration cert | Job, shipment, X12 856 REF segments |
| Heat treat spec | Process specification entity | Outside processing PO (see `extensions/outside-processing.md`) |

---

## Implementation Notes (the production system Reference)

Material specs and lots are managed in the production system Pro using:
- **Items** of item class `Raw Material` as the inventory entity (the lot)
- **Custom fields** on receipt records for heat number, DFARS status, PMI verification
- **Cert documents** stored as attachments to the item/receipt record
- **Material spec** as a separate item type with spec number, body, and revision as structured fields

The Claude MCP integration enables queries like:
- "What material lots are DFARS non-compliant in current inventory?"
- "Show me all jobs that used heat number J7291"
- "Which open POs have DFARS specialty metals requirements?"
