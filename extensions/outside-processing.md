# Outside Processing Extension

## The Gap This Fills

Outside processing (OP) operations — heat treat, plating, anodize, FPI, MPI, passivation, welding — exist in every aerospace CM job routing. The standards stack does not handle them:

- **STEP AP238** defines in-house CNC process plans. It has no schema for "ship parts to a vendor, wait, receive back, collect certs."
- **MTConnect** produces data only for in-house CNC machines. Outside processing operations leave a structural gap in the machine data timeline.
- **X12 EDI** transactions (850, 856) track the customer→manufacturer commercial relationship. They have no linkage mechanism for the manufacturer→sub-tier vendor transactions.
- **the production system** can create a PO to a vendor and an operation on a job routing — but they are separate entities with no native linkage in the system.

The result: the digital thread has a gap wherever a part leaves and returns. The cert package is assembled from disconnected records. Quality clause flowdowns to sub-tier vendors are tracked manually.

This extension defines the **Outside Process Operation** as a hybrid entity: simultaneously a routing operation (with scheduling and status) and a purchase order (with vendor, shipping, and cost) and a certification collection event (with documented conformance).

---

## Entity Definitions

### Outside Process Operation

A manufacturing operation performed by a vendor off-site. The part leaves the manufacturer's physical control and returns with a certification package.

```yaml
outside_process_operation:
  id: uuid                          # UUID v4 — minted by the production system at job creation
  job_id: uuid                      # -> job.id
  routing_sequence: integer         # Operation number in job routing (e.g., 30)
  operation_name: string            # e.g., "Heat Treat H1025", "Type II Anodize", "FPI"
  process_category: enum            # HeatTreat | ChemProcess | Plating | Coating | NDT | Welding | Other
  process_specification: uuid       # -> process_specification.id (the AMS/MIL spec)
  condition_required: string        # e.g., "H1025", "Type II Class 1"
  nadcap_required: boolean          # Whether Nadcap accreditation is contractually required
  nadcap_category: string           # Nadcap audit category code (e.g., "HT", "CP", "NDT")
  supplier_id: uuid                 # -> supplier.id (selected from approved list)
  supplier_po: uuid                 # -> purchase_order.id (created for this OP)
  purchase_order_number: string     # shop internal PO number issued to vendor

  # Scheduling
  scheduled_ship_date: date         # When parts leave the shop
  promised_return_date: date        # Vendor's committed return date
  actual_ship_date: date
  actual_return_date: date

  # Quantities (a job may send partial quantities)
  quantity_sent: integer
  quantity_returned: integer

  # Traceability (which serials/lots are at the supplier)
  items_at_supplier:
    - entity_type: enum             # Serial | Lot | Batch
      entity_id: uuid               # -> serial_number.id or material_lot.id

  # Status
  status: enum                      # Scheduled | ShippedOut | AtVendor | Returned | Rejected | Cancelled

  # Certifications collected
  process_certifications:
    - uuid                          # -> process_certification.id (CoC, test reports, etc.)

  # Cost
  po_unit_price: number
  po_currency: string               # ISO 4217
  actual_cost: number               # Final invoiced cost

  # Flowdown
  quality_clauses_flowed_down:
    - clause_code: string           # e.g., "QA-013" (Nadcap required), "QA-003" (AS9100)
      flowed_to_supplier_po: boolean
```

---

### Process Specification

A published standard defining the requirements for a special process — chemistry, time/temperature cycles, acceptance criteria, documentation requirements.

```yaml
process_specification:
  id: uuid                          # UUID v4
  spec_number: string               # e.g., "AMS 2759/3"
  spec_body: enum                   # AMS | MIL-SPEC | ASTM | Boeing | Pratt | Honeywell | Other_OEM
  title: string                     # e.g., "Heat Treatment of Corrosion and Heat-Resistant Steel Parts"
  revision: string
  process_category: enum            # HeatTreat | ChemProcess | Plating | NDT | Welding | Coating
  nadcap_category: string           # Corresponding Nadcap audit scope (e.g., "HT", "CP", "NDT")
  applicable_materials:             # Material families this spec applies to
    - string                        # e.g., "PH Stainless", "Aluminum", "Titanium"
  conditions_defined:               # Named process conditions covered
    - condition: string             # e.g., "H1025", "Type II Class 1", "Class A Grade C"
      key_parameter: string         # e.g., "1025°F ± 10°F, 4 hours"
  documentation_required:           # What the vendor must provide as output
    - enum: CoC | TestReport | ProcessChart | HardnessSurvey | ThicknessReport | NDT_Report
  status: enum                      # Active | Superseded | Withdrawn
```

**AMS Process Specification Reference:**

*Heat Treatment (AMS 2759 series — applicable to steel):*
| Spec | Scope |
|------|-------|
| AMS 2759 | Base — general requirements for all steel H/T |
| AMS 2759/1 | Carbon and low-alloy steel |
| AMS 2759/2 | Special low-alloy steel (high hardenability) |
| AMS 2759/3 | PH corrosion-resistant (15-5, 17-4, 13-8, Custom 455) |
| AMS 2759/4 | Austenitic CRES (321, 347, A286) |
| AMS 2759/5 | Martensitic CRES (410, 440C) |
| AMS 2759/9 | Hardness and conductivity verification (aluminum) |
| AMS 2759/11 | Stress relief |
| AMS 2770 | Aluminum alloys — all tempers |
| AMS 2774 | Titanium alloys |
| AMS 2801 | Nickel alloys |

*Chemical Processing:*
| Spec | Process |
|------|---------|
| AMS 2700 | Passivation of corrosion-resistant steel |
| AMS 2470 | Anodic treatment — chromic acid (Type I) |
| AMS 2471 | Anodic treatment — sulfuric acid (Type II, undyed) |
| AMS 2472 | Anodic treatment — sulfuric acid (Type II, dyed) |
| AMS 2473 | Chemical film treatment (chromate conversion, Alodine) |
| AMS 2474 | Chemical film — low electrical resistance |
| AMS 2480 | Anodic treatment — hard coat (Type III) |
| MIL-A-8625 | Anodic coatings for aluminum (types I, IB, II, IIB, III, IIIB) |

*NDT Specifications:*
| Spec | Method |
|------|--------|
| ASTM E1417 / AMS 2647 | Liquid penetrant inspection (FPI) |
| ASTM E1444 / AMS 2641 | Magnetic particle inspection (MPI) |
| ASTM E2375 | Ultrasonic testing — wrought metals |
| AMS 2630 | Ultrasonic inspection — bar, billet, plate |
| MIL-STD-1907 | Liquid penetrant and magnetic particle — DoD |

*Plating:*
| Spec | Process |
|------|---------|
| AMS 2400 | Plating — general requirements |
| AMS 2402 | Electroless nickel |
| AMS 2404 | Electroless nickel — low phosphorus |
| AMS 2460 | Hard chrome plating |
| QQ-N-290 | Electroplated nickel |
| QQ-P-416 | Cadmium plating |

---

### Process Certification

A document issued by an outside processing vendor certifying that the work was performed in conformance with the specified process standard.

```yaml
process_certification:
  id: uuid                          # UUID v4
  cert_type: enum                   # CoC | TestReport | ProcessChart | HardnessSurvey |
                                    # ThicknessReport | NDT_Report | NadcapCertCopy
  issuer: string                    # Vendor company name
  issuer_cage: string               # Vendor CAGE code
  issue_date: date
  outside_process_op: uuid          # -> outside_process_operation.id
  spec_reference: uuid              # -> process_specification.id
  document_number: string           # Vendor's internal cert number
  file_reference: string            # Path or URL to cert document PDF

  # For Certificate of Conformance (CoC):
  coc_fields:
    job_reference: string           # Vendor's internal job or work order number
    parts_processed: string         # Description of parts processed
    quantity: integer
    serial_or_lot_numbers: [string] # Which serials/lots are covered
    condition_achieved: string      # e.g., "H1025 per AMS 2759/3"
    conformance_statement: string   # Verbatim conformance declaration
    authorized_signature: string

  # For Process Chart (heat treat time-temperature record):
  process_chart_fields:
    furnace_id: string
    thermocouple_calibration_date: date
    cycle_type: string              # e.g., "Aging", "Solution + Aging", "Stress Relief"
    set_temp_f: number
    actual_temp_range_f: string     # e.g., "1022–1028°F"
    hold_time_minutes: integer
    quench_method: string           # e.g., "Air cool", "Water quench"
    load_id: string                 # Furnace load identifier

  # For NDT Report (FPI / MPI):
  ndt_fields:
    method: enum                    # FPI | MPI | UT | RT | ET
    procedure_number: string
    penetrant_type: string          # e.g., "Type I (fluorescent), Level 3"
    equipment_used: string
    inspection_result: enum         # Accept | Reject | Conditional
    indications_found: [string]     # Description of any indications

  # For Thickness/Hardness Survey:
  survey_fields:
    measurement_type: enum          # Coating_Thickness | Hardness | Conductivity
    measurement_locations: integer  # Number of measurement points
    results_summary: string         # e.g., "0.0008–0.0012 in. hard chrome per AMS 2460"
    all_within_spec: boolean
```

---

### Supplier (Outside Processor)

```yaml
supplier:
  id: uuid                          # UUID v4
  name: string
  cage_code: string
  address: string
  phone: string
  primary_contact: string

  # Capabilities
  process_categories:               # What types of outside processing they perform
    - enum: HeatTreat | ChemProcess | Plating | NDT | Welding | Coating | Machining
  process_specs_approved:           # Specific specs they are qualified to run
    - spec_id: uuid                 # -> process_specification.id
      supplier_procedure: string    # Their internal procedure number for this spec

  # Accreditations
  nadcap_accredited: boolean
  nadcap_scopes:                    # Which Nadcap categories they hold
    - category: string              # e.g., "HT", "CP", "NDT"
      certificate_number: string
      expiration_date: date
      audit_body: string            # e.g., "PRI / Nadcap"
  iso_as9100_certified: boolean
  iso_as9100_expiration: date

  # Customer Approvals (OEM-specific approved processor lists)
  customer_approvals:
    - customer_name: string         # e.g., "Lockheed Martin", "Pratt & Whitney"
      approval_scope: string        # What they are approved for
      approval_number: string
      expiration_date: date

  # the manufacturer Status
  ffm_approved: boolean
  ffm_approval_date: date
  ffm_approval_notes: string
  itar_registered: boolean
  preferred: boolean
  status: enum                      # Active | Inactive | Suspended | Disqualified
```

---

## Outside Processing Workflow

The complete sequence from job creation through cert collection:

```
JOB ROUTING CREATION
  | Op 10: CNC Mill (in-house)
  | Op 20: CNC Mill (in-house)
  | Op 30: Heat Treat H1025 per AMS 2759/3   <-- outside_process_operation minted
  |         vendor: Acme Heat Treat (CAGE: 0ABC1)
  |         nadcap_required: true (from quality flowdown QA-013)
  | Op 40: Final CNC (in-house)
  | Op 50: FPI per ASTM E1417               <-- outside_process_operation minted
  |         vendor: Precision NDT (CAGE: 5XYZ2)
  | Op 60: CMM Inspection (in-house)
  v
VENDOR PURCHASE ORDER CREATION
  | outside_process_operation.supplier_po created
  | PO line: "Heat treat to H1025 per AMS 2759/3, qty 5 pcs"
  | PO notes: quality clauses QA-003, QA-013 (Nadcap), QA-014 flowed down
  | Required certs: CoC + process chart
  v
SHIPPING OUT
  | Packing list created: 5 pcs, S/N 001-005, job 25-0412
  | items_at_supplier[] updated: serial UUIDs linked to this OP
  | status → ShippedOut
  | actual_ship_date recorded
  v
AT VENDOR
  | status → AtVendor
  | Vendor processes parts; runs furnace load; generates process chart
  v
RETURN AND RECEIVING
  | Parts physically returned
  | Receiving inspection: qty confirmed, visual inspection
  | status → Returned
  | actual_return_date recorded
  v
CERTIFICATION COLLECTION
  | CoC received: process_certification minted, linked to OP
  | Process chart received: second process_certification minted
  | Documents scanned and stored; file_reference populated
  v
JOB CONTINUATION
  | Op 30 status → Complete
  | Op 40 begins (in-house CNC)
  v
CERT PACKAGE ASSEMBLY (at shipment)
  | process_certifications[] for all outside ops are included
  | Material lot MTR + CoC included (from material-certs.md)
  | CMM results from QIF included
  | Full traceability: S/N → Job → OP → Vendor → CoC
```

---

## Nadcap Scope Reference

Nadcap accreditation is scoped by category. The category on the outside_process_operation must match the vendor's accredited scope. Mismatches are a common nonconformance finding.

| Nadcap Category Code | Process Scope |
|---------------------|---------------|
| HT | Heat Treating (all materials) |
| CP | Chemical Processing (anodize, chromate, passivation, etch) |
| CT | Coatings (thermal spray, HVOF, plasma spray, CVD/PVD) |
| NDT | Non-Destructive Testing (FPI, MPI, UT, RT, ET) |
| WLD | Welding |
| SEAL | Elastomeric sealing compounds |
| FSS | Fluid Systems (hydraulic/pneumatic tubing) |
| E | Electronics (circuit board assembly) |

**Nadcap + OEM Approval Matrix:**

Some prime contractors maintain their own approved processor lists (APLs) that are *additive* to Nadcap — a vendor must hold both. Common combinations:

| Requirement | Description |
|-------------|-------------|
| Nadcap only | Sufficient for many commercial aerospace programs |
| Nadcap + Boeing | Boeing-specific approval via BAC/BMS compliance |
| Nadcap + Pratt | Pratt & Whitney GE/GTF program approval |
| Nadcap + Honeywell | Honeywell Aerospace approved processor list |
| Nadcap + DCMA | Defense Contract Management Agency surveillance |

The manufacturer must verify that the selected vendor holds the correct Nadcap category AND any OEM-specific approval required by the customer's quality flowdown before issuing the outside process PO.

---

## Integration with the Digital Thread

| Outside Process Data | From | Linked To |
|---------------------|------|-----------|
| OP UUID | production job routing | MTConnect timeline (gap annotation) |
| Supplier PO UUID | the production system PO record | Job cost, X12 transactions |
| Serial UUID → OP | Parts at vendor | Job traceability, cert package |
| Process cert UUID | Certification document | Cert package assembly |
| Nadcap scope | Vendor record | Quality flowdown verification (QA-013) |
| Cost actual | PO invoice | Job cost breakdown |

**MTConnect Timeline Gap:**

Outside processing creates a structural gap in the MTConnect data stream. There is no machine data for the time a part is at a vendor. The outside_process_operation entity fills this gap in the manufacturer data model:

```
MTConnect data:   [Op 20 machine run] ... [gap] ... [Op 40 machine run]
production system record:   [Op 20]  [Op 30: shipped 2025-03-01, returned 2025-03-04] [Op 40]
```

The production system record provides the narrative that explains the gap — when the part left, who processed it, what was done, when it returned, and what was certified.

---

## Implementation Notes (the production system Reference)

Outside process operations in the production system Pro are managed using:

- **Job operations** with a special operation type `Outside_Process` linking to the vendor and process spec
- **Purchase orders** created from the outside process operation template; includes quality clause flowdown notes in the PO body
- **Receiving records** linked to the PO when parts return; triggers cert document collection task
- **Vendor records** with Nadcap scope, expiration dates, and customer approval status as custom fields
- **Cert documents** stored as attachments on the job record; linked to the specific OP UUID

The Claude MCP integration enables queries like:
- "Which outside process operations on open jobs are past their promised return date?"
- "Show me all jobs that used Acme Heat Treat and check if their Nadcap HT cert is current"
- "Which vendors are Nadcap NDT certified and also on the Pratt approved list?"
- "What process certifications are missing from the cert package for job 25-0412?"
