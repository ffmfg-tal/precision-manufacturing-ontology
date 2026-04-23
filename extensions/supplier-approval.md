# Supplier Approval Extension

## The Gap This Fills

Selecting a vendor for an outside processing operation is not a simple procurement decision. In aerospace manufacturing, the vendor selection is constrained by a layered approval hierarchy:

1. The customer may specify a **mandatory source** — only one company is approved, and the manufacturer has no choice.
2. The customer may specify a **customer-approved source list (ASL)** — the manufacturer must select from the customer's approved processors for that operation.
3. The contract may require **Nadcap accreditation** — the vendor must hold current accreditation for the specific process category.
4. the manufacturer maintains its own **internal approved vendor list** — audit history, performance, and ITAR status tracked internally.

None of this qualification hierarchy is captured in any of the four standards:

- **STEP AP238** defines process plans — it has no vendor qualification entity.
- **MTConnect** is machine data — the concept of vendor approval doesn't exist in the standard.
- **QIF** covers metrology — it has no supplier registry or accreditation schema.
- **X12 EDI** carries commercial transactions — vendor selection happens before any X12 document is generated.

The result: the manufacturer must verify vendor qualification manually against customer ASLs, Nadcap scope certificates, and OEM approval databases — before generating every outside process PO. There is no machine-readable representation of the approval hierarchy that could enforce this automatically.

This extension defines the **Approved Supplier List Entry** entity and the **approval hierarchy** that governs outside process supplier selection. It is the prerequisite to the outside-processing workflow (`extensions/outside-processing.md`).

---

## The Approval Hierarchy

```
LEVEL 1: Customer-Directed Source (mandatory)
  Customer specifies a single vendor by name or CAGE code on the PO.
  the manufacturer has no discretion. If the directed vendor cannot perform, customer must approve an alternative.
  Triggers: "HEAT TREAT AT: Bodycote - Garden Grove, CA" on customer PO notes

LEVEL 2: Customer-Approved Source (ASL)
  Customer maintains a list of approved processors per process category or AMS spec.
  the manufacturer must select from this list; using a non-listed vendor requires customer approval.
  Triggers: QA-013 + customer provides ASL document or directs the manufacturer to customer portal

LEVEL 3: Nadcap-Accredited Source (mandatory for special processes on most aerospace POs)
  Vendor must hold current Nadcap accreditation for the specific process category.
  Nadcap accreditation is verifiable at https://www.p-r-i.org/nadcap (public eAuditNet database).
  Triggers: QA-013 active on the quality flowdown plan

LEVEL 4: Internally-Approved Source
  the manufacturer's own qualification — vendor survey, quality agreement, ITAR registration confirmation.
  Baseline requirement for any vendor regardless of Nadcap or customer ASL status.
  Does not override higher levels; adding criteria, not substituting.
```

**When multiple levels apply, all must be satisfied.** A vendor can be Nadcap-certified and on the customer's ASL, but if the manufacturer has not internally approved them (quality survey, insurance, ITAR check), they cannot be used.

---

## Entity Definitions

### Approved Supplier List Entry

the manufacturer's record of a supplier's approval status, capabilities, and accreditations. This is the master record for supplier selection — updated when Nadcap certs expire, customer approvals change, or performance issues arise.

```yaml
approved_supplier_list_entry:
  id: uuid                          # UUID v4
  supplier_id: uuid                 # -> supplier.id (core contact/address record)
  supplier_name: string
  cage_code: string
  duns_number: string

  # Location
  address: string
  city: string
  state: string
  country: string                   # ISO 3166-1 alpha-2

  # ITAR / Export Control
  itar_registered: boolean          # DoS DDTC registration (required to handle ITAR articles)
  itar_registration_number: string  # DoS registration number
  itar_registration_expiration: date
  foreign_national_access: boolean  # Whether foreign nationals have access (affects ITAR handling)

  # Quality System
  iso_9001_certified: boolean
  as9100_certified: boolean
  as9100_certificate_number: string
  as9100_registrar: string          # e.g., "Bureau Veritas", "DNV"
  as9100_expiration: date

  # Nadcap Accreditation (may have multiple scopes)
  nadcap_accreditations:
    - category: string              # "HT" | "CP" | "NDT" | "WLD" | "CT" | etc.
      certificate_number: string
      issue_date: date
      expiration_date: date
      audit_body: string            # "PRI / Nadcap"
      primes_subscribing: [string]  # Which primes have subscribed to this audit
                                    # (eAuditNet shows subscriber list)
      scope_detail: string          # Specific materials/specs covered (from scope doc)

  # Process Capabilities
  process_capabilities:
    - process_category: enum        # HeatTreat | ChemProcess | Plating | NDT | Welding | Coating
      process_spec: uuid            # -> process_specification.id
      conditions_approved: [string] # Specific conditions vendor is qualified to run (e.g., "H1025", "H900")
      material_families: [string]   # Materials they are qualified for (e.g., "Aluminum", "PH Stainless")

  # Customer-Specific Approvals (OEM ASL entries)
  customer_approvals:
    - customer_id: uuid             # -> customer.id
      customer_name: string
      approval_number: string       # Customer's APL entry number
      approval_scope: string        # What they are approved for (process + material)
      approval_date: date
      expiration_date: date
      verification_source: string   # How the manufacturer verified (portal, ASL document, phone confirmation)
      last_verified_date: date

  # the manufacturer Internal Qualification
  ffm_approval_date: date
  ffm_approval_type: enum           # Survey | Document_Review | Third_Party_Audit
  ffm_quality_agreement: boolean    # Quality agreement on file
  ffm_quality_agreement_date: date
  ffm_approved_by: string
  performance_rating: enum      # Preferred | Approved | Conditional | Probation | Suspended
  ffm_performance_notes: string

  # Insurance / Commercial
  insurance_verified: boolean
  insurance_expiration: date
  payment_terms: string

  # Status
  status: enum                      # Active | Inactive | Suspended | Disqualified
  suspension_reason: string
  suspension_date: date
```

---

### Customer ASL (Approved Source List)

A customer's list of approved processors for specific operations, imported or referenced in the production system to constrain vendor selection.

```yaml
customer_asl:
  id: uuid                          # UUID v4
  customer_id: uuid                 # -> customer.id
  document_reference: string        # Customer's ASL document number or portal URL
  effective_date: date
  expiration_date: date
  source_type: enum                 # Customer_Provided_Document | Customer_Portal | Verbal_Confirmation
  last_verified_date: date

  # Approved entries per process scope
  entries:
    - supplier_name: string
      supplier_cage: string
      supplier_id: uuid             # -> supplier.id (if matched in the manufacturer system)
      approved_scopes:
        - process_category: string  # e.g., "Heat Treat"
          process_specs: [string]   # e.g., ["AMS 2759/3", "AMS 2770"]
          conditions: [string]      # e.g., ["H1025", "H900"]
          material_families: [string]
      notes: string
```

---

### Supplier Qualification Event

A timestamped record of a qualification action — initial approval, re-qualification, suspension, or reinstatement. Provides audit trail.

```yaml
supplier_qualification_event:
  id: uuid                          # UUID v4
  supplier_id: uuid                 # -> supplier.id
  event_type: enum                  # Initial_Approval | Annual_Review | Nadcap_Update |
                                    # Customer_Approval_Added | Performance_Issue |
                                    # Suspension | Reinstatement | Disqualification
  event_date: date
  performed_by: string              # the manufacturer quality person
  description: string
  documents_reviewed: [string]      # What was checked (Nadcap cert, AS9100 cert, etc.)
  outcome: enum                     # Approved | Conditional | Rejected
  next_review_date: date
```

---

## Vendor Selection Decision Logic

When a job operation requires outside processing, vendor selection must satisfy all active constraints. This logic should be enforced by the production system before a PO can be issued:

```
INPUT: job_id, outside_process_operation.id

1. Get quality_flowdown_plan for the job
   - Is nadcap_required active?
   - Is there a customer-directed source specified?

2. If customer_directed_source exists:
   - Only that vendor is eligible
   - If unavailable, escalate to customer for alternate approval
   - END

3. If customer ASL applies (QA-013 + customer ASL on file):
   - Filter vendors to: customer_asl.entries where process_category matches
   - If empty: escalate to customer — no eligible vendor
   - Continue with ASL-filtered set

4. If nadcap_required:
   - Filter vendors to: nadcap_accreditations where category matches AND expiration_date > today
   - If empty: escalate — no qualified vendor available

5. Filter to manufacturer-approved vendors (status = Active, performance_rating ≠ Suspended)

6. Filter to ITAR-eligible vendors (if job is ITAR-controlled)
   - itar_registered must be true
   - If foreign_national_access = true, verify with compliance officer

7. Present ranked eligible vendors to buyer/scheduler
   - Preferred vendors first (performance_rating = Preferred)
   - Sort by: performance rating, turnaround time, distance

8. Generate outside process PO to selected vendor
   - Capture supplier_id and relevant approval references in PO
   - Record selection rationale in PO notes
```

---

## Nadcap Scope and eAuditNet

Nadcap certificates are publicly verifiable through the PRI eAuditNet database at `https://eauditnet.com`. The database shows:
- Which companies hold Nadcap accreditation
- Which categories (HT, CP, NDT, etc.) they are accredited for
- Whether specific prime contractors (subscribers) have accepted the audit
- Certificate expiration dates

**the manufacturer's verification protocol:**
- At initial vendor qualification: pull eAuditNet certificate, verify scope matches process specs used
- At each PO issuance: verify certificate has not expired (expiration_date check against today)
- At annual vendor review: re-pull eAuditNet, update production system record

**Nadcap Category Scope Detail:**

A Nadcap "HT" (Heat Treating) accreditation is not a blanket approval. The scope document specifies:
- Which materials the vendor is approved to heat treat (aluminum, steel, titanium, nickel)
- Which specifications they run (AMS 2759/3, AMS 2770, AMS 2774, etc.)
- Which specific conditions/cycles they are approved for

A vendor with Nadcap HT who is only approved for aluminum per AMS 2770 **cannot** heat treat 17-4 PH to H1025 per AMS 2759/3. The scope must be verified against the specific process requirement, not just the Nadcap category.

---

## DFARS Qualifying Countries

For specialty metals on DoD contracts (DFARS 252.225-7009), material must be melted and manufactured in the U.S. or a qualifying country. Qualifying countries as of current DFARS:

Australia, Belgium, Canada, Czech Republic, Denmark, Egypt, Finland, France, Germany, Greece, Israel, Italy, Japan, Luxembourg, Netherlands, Norway, Poland, Portugal, Spain, Sweden, Switzerland, Turkey, United Kingdom

**Vendor relevance:** If a vendor performs a process that constitutes "manufacture" of specialty metals (e.g., heat treating steel that changes its properties), their country of manufacture may be relevant to DFARS compliance. This is rare for standard outside processing (heat treat, plating of machined parts) but is relevant for raw material supply chain.

---

## Integration with the Digital Thread

| Supplier Approval Data | From | Linked To |
|------------------------|------|-----------|
| Vendor ASL entries | Customer ASL document | Vendor selection constraint (quality flowdown) |
| Nadcap scope | eAuditNet / cert document | Outside process OP vendor selection |
| Customer approval number | Customer portal / document | Outside process PO documentation |
| Vendor ITAR status | Vendor qualification event | Job ITAR handling requirements |
| the manufacturer performance rating | Qualification event history | Vendor ranking in selection |
| Expiration dates | All cert records | Automated expiration alerts |

**Expiration Tracking:**

Supplier approvals have multiple expiration horizons that must be monitored:
- AS9100 certificate: typically 3-year cycle with surveillance audits
- Nadcap: typically 12–18 month audit interval; varies by prime subscriber requirements
- Customer ASL entry: some are indefinite; some expire annually; some are tied to specific contracts
- ITAR registration: annual renewal with DoS DDTC

A daily query against vendor expiration dates, cross-referenced against open POs and upcoming outside process operations, is a core operational requirement — not a luxury.

---

## Implementation Notes (the production system Reference)

Supplier approval management in the production system Pro:

- **Vendor records** with custom fields for AS9100, Nadcap scopes (one per row in a subtable), and customer approvals
- **Nadcap expiration alerts** via production system automation — flag vendors where any Nadcap certificate expires within 60 days and the vendor has open POs or upcoming scheduled OP work
- **Customer ASL import** — periodic update from customer-provided documents, stored as attachments with structured field extraction
- **PO creation validation** — Claude MCP pre-checks vendor against quality flowdown requirements before PO is issued
- **Qualification event log** — stored as notes or audit records on the vendor entity

The Claude MCP integration enables queries like:
- "Which vendors have Nadcap HT expiring in the next 90 days and are on open POs?"
- "Is Acme Heat Treat on the Lockheed ASL for AMS 2759/3 heat treat?"
- "Show me all outside process vendors qualified for titanium heat treat per AMS 2774"
- "Which vendors for job 25-0412 heat treat operation are both Nadcap-certified and on the customer ASL?"
- "List all vendors on probation or suspension and flag any open POs with them"
