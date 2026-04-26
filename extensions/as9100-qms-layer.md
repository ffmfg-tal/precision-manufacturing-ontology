# AS9100D QMS Layer — Manufacturer Extension

## Overview

The AS9100D quality management standard requires a set of documented process records that sit above the product manufacturing layer: contract review evidence, corrective action tracking, internal audit records, risk management, and continual improvement captures. These are pure manufacturer-layer entities — no Layer-1 standard (STEP AP242, MTConnect, QIF, X12 EDI) defines schemas for them.

This extension defines nine entities that close the alignment gap between the ontology's product-manufacturing model and a conforming AS9100D QMS implementation. They were derived from the April 2026 coherence pass of warp-core's OMS phase 1–3 entities against AS9100D clause requirements.

### Composition-Test Framing

These entities are **QMS compliance-trail entities**. They are not composition entities — a part decomposition walk (starting from a part specification, walking through its routing and inspection to the cert package) does not pass through them. Their role in the ontology is to close the AS9100D alignment gap. They reference composition entities (job, NCR, FAI, customer_purchase_order) but are not themselves nodes in any archetype walk.

The ontology's composition test (can a real aerospace part be decomposed end-to-end?) remains the measuring stick for schema completeness. These entities answer a different question: can the QMS evidence record that supports AS9100D certification be structured and traced? Both questions matter; they measure different dimensions of the spec.

---

## Entities

### contract_review_event (AS9100D §8.2.3.1)

Schema: `schemas/orders.yaml`

The §8.2.3.1 contract review is the manufacturer's pre-acceptance gate on every inbound customer PO. Before acknowledging a PO, the manufacturer must confirm three things: (1) it has the capability, capacity, and lead time to fulfil the order; (2) all drawing callouts, special requirements, and delivery terms are unambiguous; (3) quality clause flowdowns from the PO or quality addendum are mapped to actionable shop requirements.

`contract_review_event` is an immutable record created when this review is performed. The `customer_purchase_order.contract_review_id` FK back-references it; the Acknowledged state guard on `customer_purchase_order` requires this FK to be non-null and the review outcome to be Accepted before the PO can be acknowledged.

**Design note:** If a PO is modified before acknowledgment, a new `contract_review_event` is created for the revised PO — the record is immutable and the new review supersedes the old one by virtue of the PO's current `contract_review_id` pointing to the latest record.

**AS9100D mapping:** §8.2.3.1(a) capability, §8.2.3.1(b) requirements defined, §8.2.3.1(c) previously expressed requirements confirmed; §8.2.3 retained documented information.

---

### requirement_change_notification (AS9100D §8.2.4)

Schema: `schemas/orders.yaml`

When a customer changes requirements after the PO has been acknowledged, AS9100D §8.2.4 requires the change to be documented, the relevant documented information to be amended, and the relevant persons to be made aware of the changed requirements. `requirement_change_notification` is the record of that event.

One record per change notification received. Accepted changes that affect drawings or routings trigger an `engineering_change_notice` (captured in `triggered_ecn_id`). Changes that affect quality clauses trigger a `quality_flowdown_plan` re-evaluation.

**Design note:** This entity is distinct from `engineering_change_notice` (which is the manufacturer's internal change authorization) and from `deviation_waiver` (which is a one-time authorization to deviate from a specification). A `requirement_change_notification` is the record of the customer communicating a change to the order; the `engineering_change_notice` is the internal response to that communication.

**AS9100D mapping:** §8.2.4.

---

### customer_complaint (AS9100D §8.2.1)

Schema: `schemas/orders.yaml`

AS9100D §8.2.1 requires the organization to determine and implement effective arrangements for communicating with customers, including their complaints. `customer_complaint` is the structured record of a complaint received from a customer post-shipment.

This entity is distinct from `nonconformance_report` (which is internally initiated at discovery time). A customer complaint may reference an NCR (if the complaint confirms a product nonconformance that opens or updates an internal NCR) and may initiate a CAPA (if the complaint reveals a systemic issue requiring corrective action).

**AS9100D mapping:** §8.2.1(c) customer complaints and feedback; §8.5.6 control of changes (if the complaint triggers a process change).

---

### product_release_authorization (AS9100D §8.6)

Schema: `schemas/inspection.yaml`

AS9100D §8.6 requires that documented information identifying the person(s) authorizing the release of products be retained. `product_release_authorization` provides that record. It is created when a qualified quality engineer or manager signs off on inspection evidence and authorizes units for shipment.

The inspection session context (measurements, QIF results, photo attachments) lives in `inspection_event`. The `product_release_authorization` entity provides the explicit authorization sign-off that §8.6 requires — who authorized, when, and under what conditions. It is immutable after creation.

The `inspection_event.authorized_by` and `release_type` fields added in this update provide quick-access denormalized copies of the authorization data for session-level queries. The authoritative authorization record is `product_release_authorization`.

**AS9100D mapping:** §8.6 — retained documented information identifying the person authorizing release.

---

### corrective_action (AS9100D §10.2)

Schema: `schemas/corrective_action.yaml`

The ontology previously embedded corrective action tracking in `nonconformance_report` as boolean flags (`corrective_action_verified`) and free-text fields (`corrective_action`, `root_cause`). AS9100D §10.2 requires a richer model: documented actions that are appropriate to the effects of the nonconformity, effectiveness review, and retained documented information. Conforming QMS implementations typically maintain a standalone CAPA table.

`corrective_action` is that standalone entity. It carries root cause analysis method, documented actions, effectiveness criteria, and effectiveness verification. The initiating source is captured via polymorphic FK (`initiating_source_type` + `initiating_source_id`) — a CAPA can be initiated from an NCR, a customer complaint, an audit finding, a supplier issue, or an internal observation.

The embedded fields in `nonconformance_report` (`corrective_action` text, `corrective_action_due`, `corrective_action_verified`) are retained for informal quick-notes but are not the authoritative CAPA record. The `nonconformance_report.corrective_action_id` FK to this entity is the structured linkage.

**State machine:** Open → Root_Cause_Analysis → Action_In_Progress → Effectiveness_Review → Closed (or Cancelled at any pre-terminal state).

**AS9100D mapping:** §10.2.1(a–g); §10.2.2 retained documented information.

---

### audit_plan (AS9100D §9.2)

Schema: `schemas/corrective_action.yaml`

AS9100D §9.2 requires the organization to conduct internal audits at planned intervals, retain an audit program, and retain documented information of the audit results. `audit_plan` captures the schedule, scope, criteria, and team for a single audit cycle.

One audit plan per audit event. Multiple `audit_finding` records reference a single audit plan. The audit_plan.status lifecycle tracks the audit from Planned through Complete (or Cancelled). Audit reports are stored as referenced documents (`report_reference`).

**AS9100D mapping:** §9.2.1 (audit criteria, scope, methods, frequency); §9.2.2 (results reported, corrective action without undue delay, retained documented information).

---

### audit_finding (AS9100D §9.2)

Schema: `schemas/corrective_action.yaml`

An individual finding from an internal audit. Nonconformity findings require a `corrective_action` (linked via `corrective_action_id`); observations and opportunities for improvement may trigger CAPAs at discretion. One finding record per identified issue; a single audit may produce zero or many findings.

**Finding types:** Nonconformity (an applicable requirement is not met), Observation (a condition that may warrant attention but is not a nonconformity), Opportunity_for_Improvement (a positive suggestion, not a deficiency).

**AS9100D mapping:** §9.2.2(d) action without undue delay on findings; §9.2.2(f) results reported to relevant management.

---

### lessons_learned (AS9100D §10.3)

Schema: `schemas/corrective_action.yaml`

AS9100D §10.3 requires the organization to continually improve the suitability, adequacy, and effectiveness of the QMS. `lessons_learned` is the mechanism by which insights from quality events propagate to future jobs, routings, and quality planning.

A lesson is captured from a specific source (audit finding, customer complaint, NCR, FAI rejection, corrective action outcome, or project retrospective) and disseminated to relevant persons and processes. `applicable_processes` and `applicable_part_families` fields allow filtering lessons to relevant future work.

**AS9100D mapping:** §10.3 continual improvement; §7.1.6 organizational knowledge (retained documented information of experience gained).

---

### operational_risk_assessment (AS9100D §8.1.1)

Schema: `schemas/corrective_action.yaml`

AS9100D §8.1.1 requires the organization to address risks and opportunities when planning for the quality management system's operational processes. `operational_risk_assessment` provides the structured record for contract-specific or job-specific risk identification and mitigation.

Uses the polymorphic FK pattern (subject_type + subject_id) established at Decision 1.9 — an assessment can apply to a `customer_purchase_order` (risk identified at contract review time) or a `job` (risk identified at production planning time). The `customer_purchase_order.operational_risk_flags[]` field provides the initial structured risk signals identified during contract review; this entity is the formal risk assessment built from those signals.

**AS9100D mapping:** §8.1.1(a–d) — determine, plan, implement, and evaluate actions to address risks and opportunities.

---

## Field Additions to Existing Entities

### customer_purchase_order (schemas/customer_purchase_order.yaml)

**`special_requirements[]`** (§8.2.2c) — Typed enumeration of special requirements identified on the PO or quality addendum. Structured complement to `flowdown_clauses[]`. Drives job-level requirement flags in the `quality_flowdown_plan`.

**`operational_risk_flags[]`** (§8.1.1 / §8.2.2d) — Risk indicators identified at contract review time. Input signals for the formal `operational_risk_assessment`; not the assessment itself.

**`contract_review_id`** — FK → `contract_review_event.id`. Populated when §8.2.3.1 review is completed. The Acknowledged state guard now requires this FK to be non-null.

### nonconformance_report (schemas/quality.yaml)

**`counterfeit_flag`** (§8.1.4) — Boolean flag for suspected or confirmed counterfeit / fraudulently misrepresented parts. Triggers isolation, customer notification, and government/industry reporting requirements.

**`scar_supplier_id`** — FK → `supplier.id`. Populated for receiving inspection NCRs to track the supplier against whom a Supplier Corrective Action Request (SCAR) is issued.

**`corrective_action_id`** (§10.2) — FK → `corrective_action.id`. Links the NCR to a standalone CAPA when the nonconformance requires systemic investigation.

### first_article_inspection (schemas/quality.yaml)

**`repeat_reason_ecn_id`** — FK → `engineering_change_notice.id`. Required when `fai_reason = Design_Change` or `Process_Change`. Links the FAI to the change authorization that mandated it, providing §8.3.6 engineering change control traceability.

### inspection_event (schemas/inspection.yaml)

**`authorized_by`** — FK → `user.id`. Quality signatory for §8.6 product release. Denormalized from `product_release_authorization` for session-level queries.

**`release_type`** — Enum [Standard_Release, Conditional_Release, Hold_For_Customer_Source]. §8.6 release condition at the session level.

### approved_supplier_list_entry (schemas/approved_supplier_list_entry.yaml)

**`formal_approval_status`** (§8.4.1.1b) — AS9100D formal approval tier: AS9100_Approved, Conditionally_Approved, Pending_Audit, Not_Approved.

**`approval_scope_description`** — Narrative scope of what the supplier is approved to perform.

**`last_performance_review_date`** / **`performance_review_cadence_months`** (§8.4.1) — Supplier performance monitoring currency fields.

**`counterfeit_risk_tier`** (§8.1.4) — Standard / Elevated / High risk tier for counterfeit parts prevention.
