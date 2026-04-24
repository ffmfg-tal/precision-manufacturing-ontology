# Inspection, Calibration, and Measurement Traceability Extension

## The Gap This Fills

QIF (Quality Information Framework) is the most comprehensive Layer-1 standard for dimensional inspection data. Its Resources module covers measurement instruments, its MeasurementResources module covers instrument identities and properties, and its Results module covers characteristic measurement outcomes. Despite this breadth, QIF treats measurement instruments as supporting reference objects — it does not define a calibration lifecycle. There is no QIF construct for "this instrument was calibrated on date X, is due again on date Y, and is currently in a Quarantined hold state." The calibration record is outside QIF's scope by design: QIF cares about what was measured and with what instrument type, not whether that instrument's calibration currency is maintained.

MTConnect's CuttingTool asset schema covers tool identity and asset IDs for machine-integrated tooling. It provides a bridge between the machine and the production system for in-process tool tracking. But like QIF, it carries no calibration record structure — the MTConnect asset knows nothing about when the tool was last sent to a calibration lab or what the result was.

AS9100 §7.1.5.2 requires that measurement results be traceable to international measurement standards and that this traceability be maintained and documented. The standard is explicit: when a measuring or monitoring device is found not fit for its intended purpose, the manufacturer must determine the validity of previous measurement results and take appropriate corrective action. Meeting this requirement demands more than a timestamp on a QIF file. It demands a record that links each inspection result to a specific calibrated instrument, proves that instrument was within its calibration interval when the measurement was taken, and supports a recall analysis if the instrument is later found to have been out of calibration.

No Layer-1 standard provides this structure at the manufacturer layer. The entities defined here — `inspection_event`, `calibration_record`, `tool_gauge`, and `key_characteristic` — fill that gap. Together they make AS9100 §7.1.5.2 traceability machine-queryable rather than manually assembled.

---

## Entity Definitions

### inspection_event

An `inspection_event` is a lightweight UUID envelope for a bounded inspection session: a defined period during which one inspector (or inspection crew) collects measurement results, PMI readings, and visual evidence against a specific job and unit. The entity does not store measurement data — that belongs to `pmi_event` records (dimensional values), QIF result files (CMM outputs), and photo attachments. The sole purpose of `inspection_event` is to provide a shared parent foreign key so that all evidence from the same session can be retrieved together.

The problem this solves is structural. Without a session envelope, QIF result records, `pmi_event` records, and photo attachments collected on the same afternoon for the same serial number have no common parent — they are related only by timestamp proximity and job reference, both of which are fragile query anchors. With an `inspection_event.id`, any query asking "show me everything collected during final inspection of SN-001 on 2026-04-15" has a single, stable UUID to pivot on.

`inspection_event.event_type` categorizes the session: Receiving_Inspection, InProcess_Inspection, Final_Inspection, Source_Inspection, or First_Article. The First_Article value is particularly important — `fai_package_id` links the session to the corresponding `certification_package` record (where `includes_fai = true`), allowing the FAI package to enumerate exactly which inspection sessions contributed Form 3 dimensional evidence.

The entity is not a workflow object. There are no approval gates, no reviewer assignments, no status progression beyond the session's own `result_summary`. Once the session is saved and evidence is committed, `inspection_event` is terminal. This is a deliberate design choice (see D-INSP-1). Workflow and disposition live in `nonconformance_report` and MRB processes, not in the inspection session itself.

---

### calibration_record

A `calibration_record` is the immutable audit document for a single calibration event. One calibration event produces exactly one record; that record is never modified after creation. The fields committed at creation time — `calibration_date`, `calibration_due_date`, `calibration_interval_days`, `performed_by`, `result`, `as_found_in_tolerance`, and optionally the lab name, accreditation, and certificate reference — constitute the permanent evidence of what happened during that calibration event.

The immutability is load-bearing for AS9100 compliance. An audit record that can be edited after the fact is not an audit record — it is a mutable field. The calibration history of an instrument must be a strictly append-only sequence of records, each representing what was found and what was done at a specific point in time. This is why lifecycle state does not live here (see D-INSP-2): `calibration_status` is mutable (it transitions as time passes relative to `calibration_due_date`), and mutable state belongs on `tool_gauge`, not on the immutable evidence record.

The `as_found_in_tolerance` boolean deserves explicit attention. When a technician calibrates an instrument, the "as-found" condition — whether the instrument was within tolerance before any adjustment — is required information under AS9100 §7.1.5.2. If an instrument was out of tolerance as-found, and it was adjusted back into tolerance during calibration, any measurements taken between the previous calibration and this one must be evaluated for potential impact. The `as_found_in_tolerance` field captures this information at record creation; it cannot be inferred later from the `result` field alone, since a result of `Adjusted` does not distinguish "adjusted from within tolerance" from "adjusted from outside tolerance." Recording both allows the recall workflow to correctly identify which adjusted-result records require impact evaluation.

`performed_by` distinguishes in-house calibration from external lab calibration. When performed externally, `lab_name`, `lab_accreditation`, and `certificate_number` carry the traceability chain: the lab's A2LA or NVLAP accreditation number traces the lab to NIST, and the `calibration_standard` field documents which NIST-traceable reference was used. This chain — inspection result → instrument → calibration record → accredited lab → NIST — is what AS9100 §7.1.5.2 means by traceability to international measurement standards.

---

### tool_gauge

A `tool_gauge` is the physical measurement instrument or torque tool that the manufacturer owns, labels with an asset tag, and maintains in a calibration program. Where `calibration_record` carries the time-stamped evidence of past calibration events, `tool_gauge` carries the present state: whether the instrument is currently fit for use.

The `asset_tag` is the physical label on the instrument — the "CAL-0042" sticker on the caliper, the "TW-0007" tag on the torque wrench. This is the manufacturer's internal identifier, distinct from the instrument manufacturer's `serial_number`. Both fields exist because the manufacturer assigns asset tags sequentially across all instruments regardless of make, while the OEM serial number is unique only within that manufacturer's product line. Having both enables instrument identification both by shop label (how technicians reference it) and by OEM traceability (what appears on calibration certificates).

`tool_gauge.current_calibration_id` is a denormalized pointer to the most recent `calibration_record`. When a new calibration event is logged, `current_calibration_id` is updated to point to the new record. This denormalization enables fast "is this instrument currently calibrated?" lookups without traversing the full calibration history. `calibration_due_date` is similarly denormalized from the current record for status check efficiency.

`mtconnect_tool_asset_id` bridges this entity to the MTConnect CuttingTool asset model (see D-INSP-3). For instruments that are also tracked in the MTConnect asset stream — probe assemblies, in-process gauging systems — this field provides the FK from the manufacturer ontology into the MTConnect world. QIF's instrument resources independently identify gauges by UUID; both the QIF instrument UUID and the MTConnect asset ID map to the same `tool_gauge` record, making this entity the convergence point for physical instrument identity across both standards.

The `calibration_status` field drives the five-state lifecycle machine described in the section below. The states are: Current (within interval, usable), Due_Soon (within 30 days of due date, schedule calibration), Overdue (past due date, must not be used), Quarantined (pulled from service), and Retired (permanently decommissioned). The Overdue state carries a critical operational requirement: instruments with `calibration_status = Overdue` must not be used for production or quality measurements. If an overdue instrument was used unknowingly, the calibration recall workflow must be initiated.

---

### key_characteristic

A `key_characteristic` is a design-defined attribute flagged for elevated measurement attention and traceability. Key Characteristics (KCs) are identified by design engineers and documented on drawings per ASME Y14.5. They represent features where variation directly affects safety, function, fit, or regulatory compliance — a bore diameter whose tolerance directly controls seal behavior, a true-position callout that governs mating part clearance, a thread engagement depth that determines joint strength under load.

The operational significance of the `key_characteristic` entity at the manufacturer layer is this: KCs determine which PMI callouts must be measured and recorded per serial unit rather than sampled. A standard dimensional tolerance might be verified by first-piece or sampling inspection. A KC must be verified on every piece, with a recorded result tied to the specific serial number. Without `key_characteristic` as a distinct entity, the production system has no machine-readable way to enforce this distinction — it requires human judgment at the inspection stage to identify which callouts are KCs and apply the correct measurement frequency.

`balloon_number` ties the KC record to its drawing annotation — the diamond-symbol callout or numbered balloon that appears on the engineering drawing. This string-type field accommodates whatever notation the drawing uses: "KC-01", "14", or the diamond symbol representation. `characteristic_name` provides a human-readable description that does not require the drawing to be open: "Bore diameter true position," "Flange face flatness," "Thread engagement depth."

`characteristic_type` extends beyond the strict KC designation to cover related elevated-attention categories: Safety_Critical (safety-of-flight features requiring the same per-piece traceability as KCs), GD_T (geometric dimensioning and tolerancing callouts), Functional (performance attributes not purely dimensional), and Regulatory (ITAR-controlled geometries or other compliance-driven features). This allows the manufacturer to track all elevated-importance characteristics in one entity, not just those explicitly labeled "KC" on the drawing.

The entity uses a polymorphic foreign key — `subject_id` paired with `subject_type` — per Decision 1.9 of this ontology. A KC can be defined on a `part` or on an `assembly` (integration engineering sometimes designates assembly-level KCs for mating interface features). The `subject_type` discriminator makes this unambiguous without requiring separate tables. `qif_characteristic_id` is the optional bridge to the corresponding QIF characteristic definition in an inspection plan, enabling QIF-driven inspection execution to automatically apply the correct measurement frequency when a KC characteristic is encountered (see D-INSP-4).

---

## Calibration Recall Workflow

When a gauge is discovered to be out of calibration, damaged, or producing anomalous readings, the following sequence determines the scope of impact and drives corrective action.

1. **Quarantine the instrument.** The `tool_gauge` record transitions to `calibration_status = Quarantined`. The instrument is physically removed from service — returned to the QC lab or cage, clearly labeled as out of service. No further measurements may be taken with it until it is returned to Current status or retired.

2. **Determine the last valid calibration date.** Query `calibration_record` for this `tool_gauge_id`, ordered by `calibration_date` descending. The most recent record with `result = Pass` and `as_found_in_tolerance = true` establishes the last known good calibration date. If the current `calibration_record` has `result = Adjusted` and `as_found_in_tolerance = false`, that record is the failure event — the previous record (if it exists and has `as_found_in_tolerance = true`) defines the last valid date. If the instrument has never had a clean as-found passing result, the entire history of its use must be evaluated.

3. **Identify affected inspection sessions.** Query `inspection_event` records where this instrument appears in `pmi_event` records linked via `tool_gauge_id`, filtering to sessions whose `inspection_date` falls between the last valid calibration date and the quarantine date. These are the sessions where the instrument may have been producing unreliable results.

4. **Review measurement impact.** For each affected `inspection_event`, examine the associated `pmi_event` records. Determine whether the measurements taken with this instrument were used for conformance disposition — that is, whether a pass/fail determination was made based on a measurement that the potentially out-of-tolerance instrument produced. The severity of the dimensional departure and the instrument's measurement uncertainty relative to the feature tolerance inform the engineering assessment.

5. **Open nonconformance reports as required.** For any `serial_number` or `material_lot` where affected measurements contributed to a conformance disposition, open a `nonconformance_report`. The NCR documents the condition, the affected units, the measurement uncertainty concern, and the disposition — which may be re-inspection with a calibrated instrument, use-as-is with engineering justification, or rejection.

6. **Assess customer notification obligations.** For any affected units that have already shipped, check `customer_purchase_order.source_inspection_required` and `part.itar_controlled`. Cross-reference the quality flowdown plan for the relevant jobs — specifically whether QA-006 (24-hour notification of nonconforming shipped product) is active. If affected measurements were used to release product to a customer, customer notification may be contractually required, and the NCR record should document whether notification was made and the method and date.

---

## State Machine: tool_gauge.calibration_status

The calibration status lifecycle governs whether a physical instrument is fit for production use. The initial state when a new instrument is entered into the calibration program is `Current`. The only terminal state is `Retired`. All transitions are described below.

**Current → Due_Soon.** Triggered by the system detecting that `calibration_due_date` is within 30 days of today. The instrument remains fully usable in this state; the transition is a scheduling signal. The 30-day window provides time to schedule and complete calibration before the instrument lapses.

**Current → Overdue.** Triggered by the system detecting that `calibration_due_date` has passed with no new `calibration_record` created. The instrument must not be used for production or quality measurements from this point. In practice, the Overdue state should immediately cascade to Quarantined (see below), since an overdue instrument in a production environment is an AS9100 compliance gap.

**Current → Quarantined.** Manual action: an inspector or quality engineer pulls the instrument from service due to suspected damage, an anomalous reading, or an active investigation. The transition does not require waiting for the calibration due date to approach — any time an instrument's reliability is in question, it should be quarantined immediately.

**Current → Retired.** Direct decommission: the instrument is permanently removed from service due to physical damage beyond repair, sale, disposal, or end-of-life management decision.

**Due_Soon → Current.** Triggered by a new `calibration_record` being created with `result = Pass` or `result = Adjusted` before the due date lapses. Guard condition: `calibration_record.calibration_date` must be on or before today, and `result` must be Pass or Adjusted. A Fail result does not return the instrument to Current — it transitions to Quarantined or Retired.

**Due_Soon → Overdue.** Triggered by the system detecting that `calibration_due_date` has passed with no new calibration record. Same consequence as Current → Overdue.

**Due_Soon → Quarantined.** Manual action: the instrument is pulled from service before calibration is completed, typically due to a field incident or damage discovered during the Due_Soon window.

**Overdue → Quarantined.** System auto-quarantine. When an instrument transitions to Overdue, the system should automatically advance it to Quarantined to prevent production use. An instrument cannot remain in the Overdue state in a production environment without triggering a compliance finding — the Quarantined state makes the hold explicit and auditable.

**Quarantined → Current.** Triggered by a new `calibration_record` being created with `result = Pass` or `result = Adjusted`, and the instrument being formally released from hold. Guard condition: `calibration_record.result` must be Pass or Adjusted. A Fail result during a calibration attempt while quarantined transitions the instrument to Retired, not Current.

**Quarantined → Retired.** Investigation or damage assessment determines the instrument cannot be returned to service. This may result from a calibration Fail result, physical damage assessment, or a determination that the instrument type no longer meets measurement uncertainty requirements for the features being measured.

**Any state → Retired.** Any instrument may be retired at any time by end-of-life management decision, regardless of current calibration status. An instrument that is retired is permanently removed from service; no further calibration events are expected or recorded.

---

## Decision Log

**D-INSP-1: inspection_event is terminal — no approval gates, no status progression. Value is UUID grouping only.**

The entity exists to solve one structural problem: QIF result records, `pmi_event` records, and photo attachments from the same session have no shared parent foreign key without it. Adding workflow state (approval gates, reviewer assignment, status progression) to this entity would conflate evidence collection with disposition. Disposition lives in `nonconformance_report` and MRB processes. The inspection session records what was measured; other entities handle what to do about it.

**D-INSP-2: calibration_record is immutable — append-only audit document. Lifecycle on tool_gauge.calibration_status.**

An audit document that can be edited after the fact is not an audit document. AS9100 §7.1.5.2 requires that traceability records demonstrate what was actually found and done at a specific moment in time. Mutable lifecycle state (whether the instrument is currently fit for use) belongs on `tool_gauge`, which is designed to carry mutable state. The separation means: query `calibration_record` to understand history; query `tool_gauge.calibration_status` to understand present fitness for use.

**D-INSP-3: tool_gauge bridges QIF instrument identity and MTConnect CuttingTool asset model via mtconnect_tool_asset_id.**

QIF Resources module and MTConnect CuttingTool both reference physical measurement instruments, using different identifier namespaces. Without an explicit bridge, a physical caliper exists as two disconnected records in two standards with no shared key. `tool_gauge` is the manufacturer-layer entity that unifies them: `asset_tag` is the shop's own label, QIF's Resources module identifies the instrument by its own UUID in inspection plans, and `mtconnect_tool_asset_id` identifies it in the MTConnect asset stream. All reference the same physical object through `tool_gauge` as the convergence point.

**D-INSP-4: key_characteristic uses polymorphic FK (subject_id + subject_type) per Decision 1.9 — same pattern as other cross-type entities in this ontology.**

A Key Characteristic can be defined on a part (by design) or on an assembly (by integration engineering, for mating interface features). The polymorphic FK pattern avoids introducing separate `part_key_characteristic` and `assembly_key_characteristic` tables that would be structurally identical. Decision 1.9 establishes this as the standard pattern across the ontology for entities that must relate to more than one subject type. Consistency reduces the cognitive overhead of navigating the schema.

**D-INSP-5: calibration_record.as_found_in_tolerance (boolean) captures the "as-found" condition before any adjustment. Required for AS9100 §7.1.5.2 compliance when instruments require adjustment during calibration.**

A calibration `result` of `Adjusted` is ambiguous: it covers both "the instrument was already within tolerance but we made a minor adjustment" and "the instrument was out of tolerance and we corrected it." Only the second case requires a recall analysis of measurements taken since the previous calibration. The `as_found_in_tolerance` field disambiguates these cases at record creation time, when the calibration technician has direct knowledge of the as-found condition. Attempting to reconstruct this information later — from the `result` field alone, or from the magnitude of the adjustment — is unreliable and is not an acceptable substitute for recording the as-found condition explicitly.
