# Change Management Extension

## The Gap This Fills

Engineering change control and deviation/waiver authorization are routine administrative realities in aerospace contract manufacturing. Every part number has a drawing revision; every routing has a version; every nonconforming article requires a disposition decision. The Layer-1 standards stack addresses the *requirements* for change control but defines no data structures for it:

- **AS9100 §8.3.6** requires that design and development changes be identified, reviewed, and controlled. It mandates that changes be authorized before implementation and that the impact on already-delivered products be evaluated. It does not define what an "engineering change notice" record looks like, what fields it carries, or how it links to the part number and routing it modifies.
- **AS9100 §8.7.1** requires documented control of nonconforming output — including obtaining concessions (deviations and waivers) from customers or authorized bodies. The standard names the mechanism but defines no data entity for it.
- **STEP AP238** defines process plans in a neutral format but has no concept of "this process plan is being superseded by a change notice" or "this lot is operating under a customer deviation."
- **X12 EDI** transactions handle the commercial layer (purchase orders, invoices) and have no mechanism for communicating engineering change authorization or deviation approval between the manufacturer and customer.
- **MTConnect** captures machine state during in-house operations. It has no linkage to which drawing revision governed the operation or whether any in-flight job was operating under a deviation.

The result: the digital thread has no native record of *authorization*. A part is machined to a drawing revision, but nothing in the Layer-1 standards connects the specific job run to the ECN that released that revision, or to the deviation that allowed a nonconforming feature to be accepted by the customer. Change history and disposition authority exist only in scattered emails, ERP attachments, and manual logbooks.

This extension defines two authorization entities:

1. **Engineering Change Notice (ECN)** — the formal record that authorizes a change to a part number's drawing revision, routing, or governing specification. It links the old revision to the new one and records who authorized the change and when.
2. **Deviation / Waiver** — the customer or DER authorization document that permits nonconforming material to be used or accepted. A Deviation is pre-production authorization for a specific lot or quantity. A Waiver is post-production acceptance of already-produced nonconforming material.

---

## Entity Definitions

### Engineering Change Notice

An ECN is the authorization record for a change to a part's controlling documents. It records what changed, why, who approved it, and which specific revisions (part specification, routing) are being superseded and introduced. The ECN does **not** implement the cascade — it is not responsible for creating new routing records or updating the part master. Those are production system or PLM operations. The ECN is the authorization and the audit trail for those operations.

Approved ECNs drive a revision cascade: a new `part_specification` revision is minted, a new `routing` version is created (if the change affects the process plan), and any in-flight jobs against the old revision must be evaluated for impact. The ECN record holds references to both the old and new revisions so the full change history is traceable from either direction.

Customer notification and approval are separate concepts. `customer_notification_required` records whether the customer's quality flowdown requires the manufacturer to notify the customer of this type of change. `customer_approval_required` records whether the customer must affirmatively approve before the change is implemented. If approval is required, the `customer_approval_id` links to the `customer_approval` record documenting that authorization.

```yaml
engineering_change_notice:
  id: uuid                              # UUID v4 (required)
  ecn_number: string                    # e.g. "ECN-2026-0042" (required)
  title: string                         # Short description (required)
  description: string                   # Full change description: what changed and why (required)
  change_type: enum                     # Drawing | Routing | Material | Process | Specification (required)
  initiator_id: uuid                    # FK → user.id (required)
  initiated_date: date                  # (required)
  status: enum                          # Draft | UnderReview | Approved | Rejected | Cancelled | Released (required)
  approved_by_id: uuid                  # FK → user.id (optional)
  approved_date: date                   # (optional)
  effective_date: date                  # Date the change goes into production (optional)
  affected_part_id: uuid                # FK → part.id (required)
  old_part_specification_id: uuid       # FK → part_specification.id — revision being superseded (optional)
  new_part_specification_id: uuid       # FK → part_specification.id — new revision (optional)
  old_routing_id: uuid                  # FK → routing.id — routing being obsoleted (optional)
  new_routing_id: uuid                  # FK → routing.id — new routing version (optional)
  customer_notification_required: boolean  # (required)
  customer_approval_required: boolean      # (required)
  customer_approval_id: uuid            # FK → customer_approval.id — if customer approval required (optional)
  notes: string                         # (optional)
```

---

### Deviation / Waiver

A deviation/waiver is a customer or DER authorization document. It covers two distinct cases that share a common data structure:

- **Deviation**: pre-production authorization to produce or use material that deviates from the governing specification for a specific lot or quantity. Issued *before* production when the manufacturer knows in advance that a requirement cannot be met. Requires customer approval or a Designated Engineering Representative (DER) authorization before production begins.
- **Waiver**: post-production acceptance of nonconforming material that has already been produced. Issued *after* the fact when an NCR has documented the nonconformance and the customer is being asked to accept the articles as-is or with specified rework. A waiver always references an NCR.

**Distinction from `nonconformance_report`**: The NCR is the internal quality record. It captures the nonconformance finding, disposition decision, root cause analysis, and corrective action. The deviation/waiver is the *external* authorization document — the customer's or DER's formal acceptance of the nonconformance. An NCR triggers a waiver request; a waiver references the NCR that substantiates it.

The `approved_by` field is a free-text string, not a foreign key, because the approving authority is external — a customer quality representative by name and title, or a DER by certificate number. There is no internal `user` record for an external approver.

Expiry is controlled by two optional limits, either of which may apply: `expiry_date` limits the time window during which the deviation is valid; `expiry_quantity` limits the number of articles that may be produced or accepted under the authorization. Both may be set simultaneously — the deviation expires when either limit is reached.

```yaml
deviation_waiver:
  id: uuid                              # UUID v4 (required)
  dw_number: string                     # e.g. "DEV-2026-0007" or "WAI-2026-0003" (required)
  document_type: enum                   # Deviation | Waiver (required)
  description: string                   # What condition is being deviated/waived (required)
  part_id: uuid                         # FK → part.id (required)
  part_specification_id: uuid           # FK → part_specification.id — revision in scope (optional)
  serial_id: uuid                       # FK → serial_number.id — serialized scope (optional)
                                        #   serial_number defined in schemas/job.yaml
  lot_id: uuid                          # FK → material_lot.id — lot scope (optional)
                                        #   material_lot defined in schemas/material.yaml
  quantity_affected: integer            # Number of articles covered (optional)
  ncr_id: uuid                          # FK → nonconformance_report.id — for waivers (optional)
                                        #   nonconformance_report defined in schemas/quality.yaml
  nonconformance_description: string    # Description of the nonconforming condition (required)
  proposed_disposition: string          # What will be done with the nonconforming article (required)
  initiator_id: uuid                    # FK → user.id (required)
  initiated_date: date                  # (required)
  status: enum                          # Draft | Submitted | Approved | Rejected | Expired (required)
  approved_by: string                   # Customer rep name or DER number — external, not FK (optional)
  approved_date: date                   # (optional)
  expiry_date: date                     # Time limit on the deviation (optional)
  expiry_quantity: integer              # Quantity limit on the deviation (optional)
  customer_id: uuid                     # FK → customer.id (optional)
  notes: string                         # (optional)
```

---

## Workflow

### ECN Lifecycle

```
DRAFT
  | Engineer identifies need for change (drawing error, obsolete material,
  | customer-driven requirement change, process improvement)
  | ECN created: change_type, description, affected_part_id
  v
UNDER REVIEW
  | Engineering review: impact analysis, affected jobs evaluation
  | Customer notification issued if customer_notification_required = true
  | Customer approval sought if customer_approval_required = true
  |   → customer_approval record created; linked via customer_approval_id
  v
APPROVED
  | approved_by_id, approved_date recorded
  | new_part_specification_id linked (new revision minted in PLM/production system)
  | new_routing_id linked (new routing version created if routing affected)
  | effective_date set
  v
RELEASED
  | Change is in production; new routing drives job creation
  | Old routing status → Obsolete; old part_specification revision → Superseded
  | In-flight jobs against old revision evaluated for impact
  v
[REJECTED | CANCELLED] — terminal states
```

### Deviation/Waiver Lifecycle

```
DRAFT
  | Quality engineer identifies nonconformance or anticipated deviation
  | Deviation: before production — spec cannot be met for this lot
  | Waiver: after production — NCR documented; customer acceptance needed
  | ncr_id linked (waiver only)
  v
SUBMITTED
  | Customer or DER contacted; formal request transmitted
  | proposed_disposition documented
  v
APPROVED
  | approved_by recorded (customer rep name / DER number)
  | approved_date recorded
  | expiry_date and/or expiry_quantity set if time- or quantity-limited
  v
EXPIRED — quantity consumed or expiry_date reached
  [REJECTED] — customer does not accept; article disposition per NCR (scrap or rework)
```

---

## Integration with the Digital Thread

| Change Management Record | From | Linked To |
|--------------------------|------|-----------|
| ECN UUID | Engineering change authorization | part_specification (old/new), routing (old/new) |
| ECN → customer_approval | Customer approval record | customer.id, customer_purchase_order.id |
| Deviation/Waiver UUID | Quality disposition | part.id, material_lot.id, serial_number.id |
| Waiver → NCR | Internal quality record | nonconformance_report.id (schemas/quality.yaml) |
| Deviation scope | Lot or serial coverage | material_lot.id (schemas/material.yaml), serial_number.id (schemas/job.yaml) |

**Drawing Revision Traceability:**

Every job is produced against a specific `part_specification` revision. When an ECN releases a new revision, jobs created after the `effective_date` reference the new `part_specification_id`. Jobs in-flight at the ECN release date may continue under the old revision (with a documented justification) or be updated — the ECN record provides the audit trail for that decision.

**Cert Package Impact:**

Approved deviations and waivers must be included in the certification package shipped with the affected articles. The cert package assembly process (see `schemas/certification_package.yaml`) must enumerate any `deviation_waiver` records linked to the serial numbers or lot in the shipment.

---

## Decision Log

**Why `approved_by` on deviation_waiver is a string, not a FK**

External approvers — customer quality representatives and DERs — do not have records in the internal `user` table. A DER is identified by FAA certificate number, not by an internal user ID. A customer quality rep is identified by name, title, and company. Forcing a FK would either require maintaining stub user records for every external approver (maintenance burden, incorrect model) or leave the field consistently null (useless). Free-text string with the customer rep's name or DER certificate number is the correct model for an external authorization.

**Why ECN does not model the cascade**

The ECN records *authorization*. The production system (ERP, PLM, or MES) executes the cascade: it creates the new part_specification revision, versions the routing, and evaluates in-flight jobs. The ontology models what the ECN *is* — an authorization and audit trail — not the imperative logic of what the production system must do when one is approved. Embedding cascade logic in the schema would conflate data modeling with business process automation.

**Why deviation and waiver share one entity**

The data structure for a pre-production deviation and a post-production waiver are near-identical: both describe a nonconforming condition, both require external approval, both carry scope (lot, serial, quantity), and both expire. The `document_type` discriminator is sufficient to distinguish them. Splitting them into two entities would duplicate ~90% of the fields with no modeling benefit. The one meaningful difference — waivers reference an NCR, deviations typically do not — is handled by making `ncr_id` optional.

**Why ECN does not reference jobs directly**

In-flight job impact is an operational concern, not a structural one. The ECN records which part and which revisions are affected. Determining which specific open jobs are affected against the old revision is a query — not a modeled relationship. Adding a many-to-many `affected_jobs` collection to the ECN schema would create a maintenance burden (jobs close, jobs are cancelled) with no benefit that a query cannot provide.
