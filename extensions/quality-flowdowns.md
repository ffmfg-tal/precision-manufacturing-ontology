# Quality Flowdowns Extension

## The Gap This Fills

An aerospace purchase order from a prime contractor is not merely a commercial document. It is a quality contract. It specifies which quality system clauses apply, whether a First Article Inspection is required, which material certifications must accompany the shipment, and whether Nadcap-certified vendors must be used for special processes.

**None of this is defined in X12 850.** The transaction set that initiates production carries this information only as unstructured `MSG` free-text segments or as a separate PDF attachment. The consequences are significant:

- the manufacturer cannot programmatically determine what the cert package must contain
- Quality clause cascade to sub-tier POs is manual, error-prone, and auditable only through paper
- FAI status per part/revision/customer is not machine-trackable from the PO alone
- AS9102 FAI Forms 1 and 2 (material and process accountability) are outside QIF's scope entirely

This extension defines:
1. **Quality clauses** — the machine-readable representation of quality obligations that cascade from customer SO → production job → sub-tier POs
2. **First Article Inspection** — the AS9102 three-form FAI structure that links to material specs, process specs, and QIF dimensional results
3. **Nonconformance and MRB** — the quality event chain that QIF results can trigger but QIF does not track

---

## Entity Definitions

### Quality Clause

A specific quality obligation that attaches to a sales order and cascades to the job and potentially to sub-tier vendor POs.

```yaml
quality_clause:
  id: uuid                          # UUID v4
  clause_code: string               # Standard code (e.g., "QA-001", "Q-001")
  clause_source: enum               # Customer | FFM_Standard | IAQG | AS9100 | DoD_DFARS
  customer_id: uuid                 # -> customer.id (if customer-specific clause definition)
  title: string                     # Short name
  full_text: string                 # Complete clause text (customer may have version-specific wording)
  category: enum                    # QMS | FAI | Material_Cert | RightOfAccess | Nonconformance |
                                    # SubtierControl | DFARS | ITAR | CounterfeitPrevention |
                                    # RecordRetention | ChangeNotification | FOD | SPC | Nadcap
  flows_to_subtier: boolean         # Whether this clause must be included on sub-tier POs
  triggers_job_requirement:         # What this clause creates in the production system
    - enum: FAI_Required | CoC_Required | MTR_Required | DFARS_Declaration |
            Nadcap_Required | SPC_Required | SourceInspection_Required |
            CustomerApproval_Required | RecordRetention_Extended
  status: enum                      # Active | Superseded | Withdrawn
```

**Standard Quality Clause Reference:**

The following clause codes represent the most common quality obligations in aerospace CM purchasing. Customers define their own versions — clause text varies, but the functional categories are stable across the industry.

| Code | Category | Requirement |
|------|----------|-------------|
| QA-001 | QMS | AS9100 Rev D registration required; maintain certification current |
| QA-002 | FAI | First Article Inspection required per AS9102 Rev C before shipment |
| QA-003 | Material Cert | Certificate of Conformance required with each shipment |
| QA-004 | Material Cert | Mill Test Report (MTR) required — chemistry and mechanical properties |
| QA-005 | Right of Access | Buyer, customer, and regulatory authority have right of source inspection |
| QA-006 | Nonconformance | Notify buyer within 24 hours of identifying a nonconforming condition affecting shipped product |
| QA-007 | Subtier Control | No further subcontracting without written buyer approval; flow clauses to subs |
| QA-008 | DFARS | DFARS 252.225-7009 specialty metals compliance — domestic melt and manufacture |
| QA-009 | Conflict Minerals | 3TG conflict minerals declaration per DFARS 252.225-7052 |
| QA-010 | Record Retention | Retain quality records for [n] years after last shipment |
| QA-011 | ITAR | ITAR compliance required; no foreign national access without license; DoS DDTC registration |
| QA-012 | Counterfeit | Counterfeit part prevention per AS5553; use only authorized distributors |
| QA-013 | Nadcap | Nadcap accreditation required for all special processes (heat treat, plating, NDT, etc.) |
| QA-014 | Change Notification | No changes to design, material, process, or sub-tier without prior written approval |
| QA-015 | FOD | Foreign Object Debris / Foreign Object Damage prevention program required |
| QA-016 | Shelf Life | Shelf-life controls for limited-life materials; date codes required on certs |
| QA-017 | SPC | Statistical process control required; Cp/Cpk ≥ 1.33 for key characteristics |

---

### Quality Flowdown Plan

The set of quality obligations attached to a specific sales order / job. This entity is the machine-readable version of what is currently communicated as PDF quality addendum.

```yaml
quality_flowdown_plan:
  id: uuid                          # UUID v4
  sales_order_id: uuid              # -> sales_order.id
  job_id: uuid                      # -> job.id
  customer_id: uuid                 # -> customer.id
  created_date: date
  source_document: string           # Reference to the PO/quality addendum that defines these requirements

  # Quality clauses in effect for this job
  clauses:
    - clause_id: uuid               # -> quality_clause.id
      customer_clause_number: string  # Customer's own numbering (may differ from QA-00x)
      version: string               # Customer's revision of this clause
      applies_to_subtier: boolean   # Whether this specific instance flows to subs
      compliance_verified: boolean  # the manufacturer has verified compliance
      verification_notes: string

  # Derived requirements (populated from clauses)
  fai_required: boolean
  fai_status: enum                  # Not_Required | Open | Submitted | Approved | Rejected
  coc_required: boolean             # Certificate of Conformance
  mtr_required: boolean             # Mill Test Report
  dfars_required: boolean           # DFARS specialty metals declaration
  source_inspection_required: boolean  # Customer QA must witness inspection before shipment
  nadcap_required: boolean          # Special processes must use Nadcap vendors
  spc_required: boolean             # SPC data must accompany shipment
  record_retention_years: integer   # Number of years records must be retained

  # Subtier flowdown (generated on outside process POs)
  subtier_flowdowns:
    - po_id: uuid                   # -> purchase_order.id
      supplier_id: uuid             # -> supplier.id
      clauses_flowed: [uuid]        # -> quality_clause.id[] — subset appropriate for sub-tier
      flowdown_document: string     # Reference to the flowdown clause text sent to supplier
```

---

### First Article Inspection (FAI)

An AS9102 Rev C compliant FAI record. Three forms, one coherent package. The FAI is the documented proof that, for the first time, an article (part) has been produced that conforms to all drawing requirements.

```yaml
first_article_inspection:
  id: uuid                          # UUID v4
  subject_id: uuid                  # -> part.id or assembly.id per subject_type (Decision 1.9)
  subject_type: enum                # values: [part, assembly]
  part_number: string
  revision: string
  sales_order_id: uuid              # -> sales_order.id (the job that produced the FAI unit)
  job_id: uuid                      # -> job.id
  serial_number_fai_unit: uuid      # -> serial_number.id (which specific unit was the FAI article)
  fai_reason: enum                  # New_Part | Design_Change | Process_Change | Production_Lapse |
                                    # Source_Change | Sub_Change | Manufacturing_Location_Change
  production_lapse_years: number    # If reason is Production_Lapse; typical threshold is 2 years
  fai_date: date
  status: enum                      # Open | Submitted | Approved | Rejected | Conditionally_Approved
  customer_approval_number: string  # Customer's FAI approval reference (if approved)
  customer_approval_date: date

  # ---- FORM 1: Part Number Accountability ----
  form1:
    organization_name: string       # the manufacturer's name
    cage_code: string               # the manufacturer CAGE code
    drawing_number: string
    drawing_revision: string
    design_activity_name: string    # Customer name (design authority)
    authorized_signature: string
    authorized_date: date
    # If partial FAI (only affected characteristics):
    partial_fai: boolean
    partial_fai_justification: string

  # ---- FORM 2: Product Accountability (Materials and Processes) ----
  # Each line item is a standalone material_identification entity (schemas/material_identification.yaml)
  # with its own UUID, linked back to this FAI via fai_id FK. See that schema for the full field set.
  form2:
    line_item_ids: [uuid]           # -> material_identification.id[] (one per drawing callout)

  # ---- FORM 3: Characteristic Accountability ----
  form3:
    # Links to QIF results for dimensional characteristics
    qif_results_document: string    # Reference to QIF file UUID or file path
    # Each characteristic links to a QIF characteristic UUID
    characteristics:
      - characteristic_number: integer  # Balloon number from inspection drawing
        characteristic_type: enum       # Dimension | GDT | Surface_Finish | Thread | Note | Other
        drawing_requirement: string     # Nominal + tolerance as written on drawing
        qif_characteristic_id: uuid     # -> QIF characteristic UUID (from inspection plan)
        measured_value: number
        deviation: number
        upper_tolerance: number
        lower_tolerance: number
        pass_fail: enum             # Pass | Fail | Not_Measured
        measuring_equipment: string # Equipment description
        equipment_calibration_id: uuid  # -> calibration_record.id
        inspector: string
        measurement_date: date
      # Note: characteristics whose results come from QIF are linked by UUID;
      # manual entry is allowed for characteristics not measured on CMM

  # FAI completeness check
  form1_complete: boolean
  form2_complete: boolean
  form3_complete: boolean
  fai_package_complete: boolean     # All three forms complete and internally consistent
```

---

### Nonconformance Report (NCR)

A quality event record for any article that fails to meet a specified requirement — whether discovered in receiving inspection, during production, at final inspection, or after shipment.

```yaml
nonconformance_report:
  id: uuid                          # UUID v4
  ncr_number: string                # the manufacturer sequential NCR number (e.g., "NCR-2025-0077")
  discovery_date: date
  discovery_source: enum            # Receiving | InProcess | FinalInspection | Customer_Return | Audit
  job_id: uuid                      # -> job.id (if associated with a job)
  serial_ids: [uuid]                # -> serial_number.id[] (which specific units are affected)
  lot_id: uuid                      # -> material_lot.id (if a material nonconformance)
  subject_id: uuid                  # -> part.id or assembly.id per subject_type (Decision 1.9)
  subject_type: enum                # values: [part, assembly]
  quantity_nonconforming: integer
  quantity_total_in_lot: integer

  # Description
  nonconformance_description: string   # What is wrong
  characteristic_affected: string      # Which drawing requirement is not met
  deviation_from_spec: string          # Magnitude of the departure (e.g., "0.002 over tolerance")
  qif_result_reference: uuid           # -> QIF characteristic UUID if from CMM measurement

  # Discovery
  discovered_by: string
  inspector_id: uuid                   # -> user.id
  equipment_used: string

  # Customer notification
  customer_notification_required: boolean  # Per QA-006
  customer_notified: boolean
  customer_notification_date: date
  customer_notification_method: string

  # MRB Disposition
  mrb_status: enum                  # Open | Convened | Disposition_Issued | Closed
  mrb_disposition: enum             # UseAsIs | Rework | Repair | Scrap | Return_To_Vendor |
                                    # Customer_Deviation_Request
  mrb_justification: string         # Engineering justification for use-as-is or deviation
  mrb_authorized_by: string
  mrb_authorization_date: date
  customer_deviation_approved: boolean   # If customer deviation was required and obtained
  customer_deviation_number: string

  # Rework / Re-inspection
  rework_job_id: uuid               # -> job.id (rework operation)
  rework_description: string
  re_inspection_required: boolean
  re_inspection_qif_reference: string   # QIF file for re-inspection results
  re_inspection_result: enum        # Pass | Fail | Pending

  # Preventive Action
  root_cause: string
  corrective_action: string
  corrective_action_due: date
  corrective_action_verified: boolean

  status: enum                      # Open | Pending_MRB | Pending_Customer | Rework_In_Progress |
                                    # Closed | Scrapped
```

---

## Quality Flowdown Cascade

The flowdown cascade connects the customer's quality requirements to every level of the production chain:

```
CUSTOMER PURCHASE ORDER (X12 850)
  | MSG segment: "QA-001, QA-002, QA-003, QA-004, QA-008, QA-013 apply"
  | Attached: Quality addendum PDF with full clause text
  v
The manufacturer SALES ORDER / JOB
  | quality_flowdown_plan created from PO data
  | Clauses activated: QA-001 through QA-013 (as applicable)
  | Derived requirements set:
  |   fai_required: true (QA-002)
  |   coc_required: true (QA-003)
  |   mtr_required: true (QA-004)
  |   dfars_required: true (QA-008)
  |   nadcap_required: true (QA-013)
  v
The manufacturer JOB ROUTING
  | For each outside process operation:
  |   If nadcap_required: true → vendor must hold Nadcap for this category
  |   If QA-007 active: generate subtier PO with flowdown text
  v
OUTSIDE PROCESS PO (to sub-tier vendor)
  | subtier_flowdowns populated
  | Clauses included: QA-001, QA-003, QA-013, QA-014 (at minimum)
  | Vendor must return: CoC, process chart (Nadcap-certified)
  v
CERT PACKAGE (assembled at shipment)
  | FAI package (if QA-002): Forms 1, 2, 3
  | CoC (QA-003)
  | MTR (QA-004)
  | DFARS declaration (QA-008)
  | Process certs from all outside processes (QA-013 compliance evidence)
  | SPC data if required (QA-017)
```

---

## FAI Trigger Matrix

| Trigger | Re-FAI Scope | Key AS9102 Guidance |
|---------|-------------|---------------------|
| New part number | Full FAI — all three forms | First occurrence of any article |
| Drawing revision (major) | Affected characteristics only | Changes to fit/form/function |
| Drawing revision (minor) | Form 1 update minimum | Administrative changes |
| Process change | Form 2 + affected Form 3 | Material, process, or sub-tier change |
| Production lapse (≥2 years) | Full FAI typically required | Customer may accept partial |
| Manufacturing location change | Full FAI typically required | AS9102 and customer discretion |
| Sub-tier supplier change | Form 2 + sub-tier FAI | If sub affects form/fit/function |
| Source inspection finding | Affected characteristics | Customer QA may trigger re-FAI |

---

## Integration with the Digital Thread

| Quality Flowdown Data | From | Linked To |
|-----------------------|------|-----------|
| QA clause list | Sales order / PO text | production job quality plan |
| FAI status | FAI entity | Item master (per P/N + revision) |
| Form 2 spec references | FAI Form 2 | material_specification, process_specification |
| Form 3 characteristics | FAI Form 3 | QIF characteristic UUIDs |
| NCR nonconformance | NCR entity | QIF result (out-of-tolerance trigger) |
| MRB disposition | NCR → MRB | Job status, serial number record |
| Subtier flowdown | quality_flowdown_plan | Outside process PO |

**QIF → NCR Link:**

When QIF ingestion identifies a failed characteristic, the NCR workflow begins:

```
QIF result: characteristic "Bore Diameter A" — measured 0.003 over tolerance → FAIL
  v
Production system: NCR-2025-0077 created
  | linked to: serial 7832-003, job 25-0412, QIF characteristic UUID e4a2f9e1-...
  v
MRB convened:
  | Disposition: Rework — bore can be re-machined to tolerance
  v
Rework operation added to job routing
  v
Re-inspection QIF file generated
  | New characteristic result: PASS
  v
NCR closed — rework verified
  v
Cert package: NCR-2025-0077 record included as quality evidence
```

---

## Implementation Notes (the production system Reference)

Quality flowdowns and FAI in the production system Pro are managed using:

- **Quality plan custom fields** on the sales order / job: clause codes as multi-select, derived requirements as boolean flags
- **FAI record** as a separate entity type in the production system, linked to the item and job; Form 2 items stored as line items
- **QIF ingestion** populates Form 3 characteristic results via UUID linkage (see `integration/fulcrum-qif.md`)
- **NCR records** as production system NCR entities with disposition workflow; linked to QIF result and serial number
- **Subtier flowdown** text auto-generated from active clauses when an outside process PO is created

The Claude MCP integration enables queries like:
- "Which open jobs require FAI and what is the current FAI status?"
- "Show me all NCRs with use-as-is dispositions on jobs shipped in the last 90 days"
- "Which characteristics on serial 7832-003 failed QIF inspection and what was the MRB disposition?"
- "Does job 25-0412 have all required cert package items ready for shipment?"
- "Which jobs have QA-017 (SPC) active and are missing Cp/Cpk data?"
