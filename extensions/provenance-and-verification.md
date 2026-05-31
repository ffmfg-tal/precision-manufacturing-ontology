# Provenance & Verification — The Epistemic Substrate

## Why This Matters

The rest of this ontology models **what is true** about a part: its geometry, its material, its routing, its characteristics, its certifications. This extension models something the others assume: **how we know it, who asserted it, by what method, with what confidence, and whether a human has verified it.**

That distinction is dormant in a world where every fact is hand-entered by a person. It becomes load-bearing the moment an AI is in the loop. When a model extracts two hundred characteristics from a drawing, recognizes the features on a solid model, and proposes a routing, the system is no longer storing facts — it is storing **assertions**, each with a different basis and a different reliability. In a regulated shop (AS9100, ITAR) an unverified AI-inferred tolerance is not a fact; it is a candidate. Treating it as a fact is how a wrong number reaches the shop floor as a real consequence.

The epistemic substrate is the cross-cutting layer that records the difference. It is to *trust* what `uuid-discipline.md` is to *identity*: not a domain, but a discipline that every domain inherits. It has three parts:

1. **`provenance`** — a value-object embedded on any asserted fact: method, confidence, source, principal, verification state.
2. **`verification_event`** — the record of a human (or agent) act of confirming, correcting, or rejecting an assertion. The auditable seam where accountability attaches.
3. **The design-ingestion artifacts** — `cad_model`, `derived_view`, `manufacturing_feature` — the front of the digital thread, where structured manufacturing definition is extracted from a customer's model and drawing. This is where assertions are *born*, and therefore where provenance first attaches.

---

## The Shape of an Assertion

Every entity whose content is **extracted, inferred, computed, or measured** — rather than being a primitive of record — carries a `provenance` value-object (`schemas/provenance.yaml`). It is a mixin, not a top-level entity: it has no identity of its own and is addressed through its owner (e.g., `manufacturing_feature.provenance`).

| Field | Role |
|---|---|
| `assertion_method` | How the fact was established (see taxonomy below) |
| `confidence` | 0.0–1.0; **required for `AI_Inferred`**, implicitly 1.0 otherwise |
| `source_artifact_refs[]` | Coded refs (`<type>:<uuid>`) to the artifacts that ground the assertion |
| `asserted_by_principal_type` / `_id` | Who asserted it — `user` / `agent` / `system` (mirrors warp-core `events.principal_type`) |
| `model_identifier` | For AI assertions: the model + version stamp, so a decision is reproducible/attributable |
| `asserted_at` | When |
| `verification_state` | `Proposed → Under_Review → Confirmed / Rejected / Superseded` (projection of verification_event) |

This generalizes the ad-hoc provenance fields that already exist in the canon — `characteristic.source` (`AI_Extracted | Manual | CMM_Imported`), `characteristic.extraction_confidence`, and `measurement_result.source_type` — which are retained as the **legacy inline form** the value-object supersedes.

### The assertion-method taxonomy — and the hallucination anchor

| Method | Meaning | Trust |
|---|---|---|
| **Measured** | Physical metrology — CMM, XRF/PMI, hand gauge | Exact within instrument tolerance |
| **Computed** | Deterministic derivation from exact geometry — a dimension read off the reconstructed B-rep by a geometry kernel | Exact |
| **AI_Inferred** | Model judgment — feature recognition, GD&T read from a drawing image, process intent | Probabilistic — `confidence` required |
| **Human_Authored** | A person entered or authored it directly | Trusted by attribution |
| **Standard_Parsed** | Decoded from a structured standard — e.g., AP242 semantic PMI | One source among several, often unreliable |
| **Legacy_Imported** | Migrated from a prior system without finer provenance | Unknown — treat as Proposed |

The single most important rule the taxonomy enforces — and it binds *where geometry is the truth*: **a feature's as-modeled geometric dimensions are `Computed` or `Measured`, never `AI_Inferred`.** A model is excellent at *recognizing* that a face is a precision bore and *associating* it with drawing balloon 12; it is unreliable at *reading* that the bore is Ø12.70. So the kernel measures the size the geometry already knows, and the model reasons. The geometry kernel (OpenCASCADE or equivalent) is the **hallucination anchor**: it hands the model the exact number so the model never has to estimate it.

**Requirements are a different kind of fact, and the rule does not — cannot — bind them.** Tolerances, GD&T control frames, and datum schemes are not present in the geometry; a perfect B-rep has no tolerance band. So a *characteristic's* requirement values can only be `AI_Inferred` (read from the controlling drawing), `Standard_Parsed` (semantic PMI), or `Human_Authored` — never `Computed` — and they must carry a `verification_event`. The two halves meet at the nominal: where a `cad_model` exists, a characteristic's nominal is cross-checked against the grounded feature's `Computed` dimension — agreement raises confidence, disagreement is exactly the model-vs-drawing conflict that `controlling_authority` resolves (the drawing governs). The discipline, then, is not "AI never touches dimensions"; it is **the AI never *measures* (the kernel does that) and never *authors a requirement without a human signing it*.** The `cad_model` entity exists to make the measurement anchor a first-class, queryable thing; the `verification_event` makes the requirement sign-off a first-class, auditable one.

---

## The Design-Ingestion Artifacts — The Missing Link

Walk any archetype tree in `reference/composition-archetypes.md`. Every one starts at `[Part Specification]` — the drawing — and assumes the routing, operations, and characteristics already exist. None model **how they came to be** from the customer's deliverable. That front segment — model + drawing → structured features and requirements → process plan — is the manufacturing-engineering / CAM-programming step, and it has historically lived entirely in a programmer's head. It is the link the digital thread has been missing, and it is exactly where AI changes the economics.

Three entities model it (and **Walk 8** in `composition-archetypes.md` walks the whole segment):

### `cad_model` — geometric ground truth (`schemas/cad_model.yaml`)
The 3D solid model for a `part_specification` revision, reconstructed into a boundary representation (B-rep) by a geometry kernel. Distinct from `part_specification`: the specification is the governing *document* (drawing number, revision, file refs, distribution statement); the `cad_model` is the parsed, kernel-queryable *artifact* with the properties that decide trust — `brep_available`, `kernel_reconstructed`, `semantic_pmi_present`, `content_hash`, `units`, `bounding_box`. It is the grounding source for every `Computed` assertion.

### `derived_view` — multimodal grounding (`schemas/derived_view.yaml`)
A rendered or projected view (isometric, orthographic, **section**, detail, exploded, annotated) generated *from* a `cad_model`. AI feature recognition is multimodal — the model sees the part the way a machinist does. A `derived_view` is the visual artifact an `AI_Inferred` assertion cites in `provenance.source_artifact_refs`, making the inference auditable ("the AI called this a pocket from section view SEC-A"). Section/detail views matter disproportionately: internal and occluded features are invisible in isometrics, so the views must be driven off the topology, not generic.

### `manufacturing_feature` — the bridge (`schemas/manufacturing_feature.yaml`)
A recognized feature (hole, pocket, slot, boss, thread, chamfer, face, …) grounded in specific B-rep faces (`brep_face_refs`) of a `cad_model`. It is the object a CAM **operation** produces and an inspection **characteristic** constrains. It carries a mandatory `provenance` (typically `AI_Inferred` + confidence) and is confirmed or corrected by a `verification_event` before it drives routing authoring.

`manufacturing_feature` is the grounded counterpart of the loose `characteristic.feature_type` string. **One feature concept, two altitudes:** `feature_type` stays an informal classifier; `manufacturing_feature` is the geometry-backed entity a characteristic FKs to. They do not compete.

---

## Verification — Where Accountability Attaches

`verification_event` (`schemas/verification_event.yaml`) records one act of confirming, correcting, rejecting, escalating, or superseding an asserted fact. It is a **domain entity** with its own lifecycle and AS9100-grade sign-off — explicitly **not** the warp-core infra event-log row. (It *generates* infra events when committed; it does not equal them. The reason it is not just an event-log query for `principal_type = user` is that it carries domain semantics the generic envelope cannot: the decision, the verification method, the corrected value, signatory eligibility, and assertion-supersession lineage.)

It is immutable after sign-off, mirroring `calibration_record` and `witness_inspection`. For regulated facts the verifier is a person — **every signature is a person, every time.** Agents may pre-screen and propose (`verifier_principal_type = agent`, `verification_method = Agent_Cross_Model`), but the accountable confirmation on regulated work is human, with `user.roles` eligibility enforced by guard.

This entity generalizes a loop the canon already runs narrowly: in `characteristic.yaml`, an AI extraction pipeline populates characteristics and "a quality engineer reviewing the extraction may flag individual records as `is_critical` and elevate them to `key_characteristic`." That QE review *is* a verification act. `verification_event` makes it first-class and applies it to every asserted fact, not just characteristics.

---

## PMI Is Not the Authority

Model-based definition promises that GD&T travels as machine-readable semantic PMI inside the STEP AP242 model, so a system could parse tolerances straight from the file. In practice this is unreliable enough that the ontology must not architect around it:

- Authored PMI quality is a coin flip. Many AP242 files carry PMI as **graphical annotations** (text glued to the model) rather than **semantic** PMI (structured, queryable). `cad_model.semantic_pmi_present` records which.
- In most aerospace contracts the **2D drawing is the controlling document.** The model is reference. When the two disagree, the drawing governs — unless the contract says otherwise.

So the substrate treats parsed PMI as **`Standard_Parsed`: one asserter among several, usually low-confidence.** It is not rejected — when good semantic PMI exists it is a free corroborating source that can agree-or-flag against the AI's drawing read, and the provenance model carries it for free. But the authoritative reading of a tolerance is the AI's `Cross_Check_Source` read of the controlling drawing, confirmed by a human `verification_event`. The authority asymmetry is recorded explicitly (see staged `part_specification.controlling_authority` below): **the drawing is the controlling document; the model and its PMI are corroborating evidence.**

---

## Staged Field Additions (applied in lockstep with implementation codegen)

The new entities above are net-new schema files and carry no implementation risk. The following **additions to existing entities** are specified here as canon and land in their YAML files *in lockstep with warp-core codegen regeneration* (warp-core's drift check regenerates listed entities and compares byte-for-byte; editing an implemented entity's schema ahead of regeneration would break that check). They are listed so the canon is complete and the implementer applies them as one step.

| Entity | Staged field | Purpose |
|---|---|---|
| `characteristic` | `manufacturing_feature_id: uuid` (FK, optional) | Links a drawing callout to the grounded feature it constrains |
| `characteristic` | `provenance: object` (optional) | The value-object generalizing the existing `source`/`extraction_confidence` |
| `part_specification` | `controlling_authority: enum {Drawing, Model, MBD_Dataset}` | Which artifact governs when model and drawing disagree |
| `part_specification` | `cad_model_id: uuid` (FK, optional) | The geometric-truth model for this drawing revision |
| `measurement_result` | `manufacturing_feature_id: uuid` (FK, optional) | The feature a measurement was taken on |
| `measurement_result` | `provenance: object` (optional) | Generalizes the existing `source_type` |
| `operation` | `produces_feature_ids: uuid[]` (optional) | The feature(s) a routing operation realizes |

---

## The End State: Human-Supported AI

Today FFMFG's platform is described as **AI-assisted humans** — "AI multiplies attention, doesn't replace judgment." The mechanical work goes to agents; everything cognitive and every signature stays with people. That is the correct starting posture, and the provenance/verification substrate is what lets it move.

The trajectory is **confidence-gated autonomy**:

- **Now** — AI proposes; every asserted fact is `Proposed` and routes to a human `verification_event`. The human does the cognitive work and verifies.
- **Next** — as AI assertion confidence and verification track records accumulate, the verification gate narrows from *everything* to *exceptions*: low confidence, key characteristics, novel geometry, regulated features. The AI does more of the cognitive work — recognizing features, reading GD&T, drafting routings — and the human reviews the residual.
- **End state — human-supported AI.** The AI runs the manufacturing-engineering loop end to end: model + drawing + views → features → requirements → process plan, each step a provenance-stamped assertion. The human role concentrates into **verification, judgment, and accountability** over AI-produced work — moving from *producer* to *verifier and owner*.

The crucial property: **the verification layer is permanent, not scaffolding.** It is tempting to read human review as a crutch to be removed once the AI is good enough. In a regulated shop it is the opposite — it is the auditable seam where human accountability attaches to the work, and it is exactly what makes the inversion *safe and certifiable*. The human never leaves the loop; the human's position in the loop changes. `verification_event` is the technical embodiment of that seam, which is why it is first-class, immutable, and signed.

This substrate is therefore the load-bearing element of the whole platform's direction: it is simultaneously the thing that makes AI-extracted manufacturing definition trustworthy *today* and the mechanism by which the human role *inverts* over time. The ontology models trust because trust is what lets the work move from people's heads into a system that can carry it.

---

## Validation Rules

1. An entity created by extraction or inference with no `provenance` value-object is non-conformant — its trust basis is unknowable. Flag for backfill.
2. An `AI_Inferred` assertion with no `confidence` is invalid — the field is required.
3. A **feature's** as-modeled geometric dimension asserted as `AI_Inferred` is a defect — feature geometry must be `Computed` or `Measured` (the kernel measures it). Flag. (This does NOT apply to a **characteristic's** requirement values — tolerances, GD&T, datums — which are not in the geometry and are correctly `AI_Inferred` from the drawing, `Standard_Parsed`, or `Human_Authored`, each gated by a `verification_event`.)
4. A regulated fact (a key characteristic, an FAI characteristic, an ITAR-controlled feature) with `verification_state = Confirmed` but no `verification_event` whose `verifier_principal_type = user` is non-conformant — regulated confirmation requires a human signature.
5. A `manufacturing_feature` with `recognized_by = AI_Inferred` and no `source_artifact_refs` citing at least one `cad_model` cannot be audited — block its use in routing authoring.
6. A `cad_model` with `brep_available = false` cannot be the source of a `Computed` assertion — any `Computed` provenance citing it is invalid.
7. A `Confirmed` characteristic whose value was `Corrected` by a `verification_event` must carry the `corrected_value`, not the original AI assertion, downstream.

---

## Reference

- ISO 10303-242 (STEP AP242) — managed model-based 3D engineering; semantic vs graphical PMI.
- ISO 10303-238 (STEP-NC / AP238) — machining features and operations; prior art for feature-to-operation linkage.
- ASME Y14.5 / Y14.41 — GD&T and digital product definition practices; the controlling-drawing convention.
- W3C PROV-O — provenance data model (who/what/when of an assertion); conceptual prior art for the value-object.
- `extensions/uuid-discipline.md` — the identity discipline this trust discipline parallels.
- `reference/composition-archetypes.md` — Walk 8 (Design Ingestion / CAM Programming) demonstrates these entities end-to-end.
- `reference/taxonomic-decisions.md` — Decision 1.10 records the value-object-vs-entity and feature-reconciliation rationale.
