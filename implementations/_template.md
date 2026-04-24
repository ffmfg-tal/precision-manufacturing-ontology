# [System Name] — Reference Implementation Mapping

Cross-reference between this ontology and [System Name]. Starting template for documenting how a conforming system implements (or falls short of) the canonical ontology.

## Architecture Overview

- **Stack:** [e.g., PostgreSQL, TypeScript, Ruby on Rails, SaaS REST API]
- **Storage pattern for ontology extensions:** [how this system stores attributes the canonical ontology requires but the system doesn't model natively — e.g., JSONB custom fields, EAV table, flat text attachment]
- **Notable architectural properties:** [multi-tenant, RLS, single-tenant, polling vs webhooks, pagination cap, etc.]

## Domain Mapping

Keep this structure in sync with `../reference/domain-map.md` — twelve domains.

### Materials

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| material | | | |
| material_specification | | | |
| material_lot | | | |
| certification_document | | | |

### Items & BOMs

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| part | | | |
| bom_line | | | |
| routing | | | |

### Manufacturing Execution

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| job | | | |
| operation | | | |
| outside_process_operation | | | |
| work_center | | | |
| process_specification | | | |

### Quality

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| first_article_inspection | | | |
| as9102_form_1 / form_2 / form_3 | | | |
| key_characteristic | | | |
| nonconformance_report | | | |
| mrb_disposition | | | |

### Supply Chain

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| supplier / vendor | | | |
| supplier_approval / approved_vendor_list_entry | | | |
| nadcap_certification | | | |
| purchase_order / sub_tier_purchase_order | | | |
| quality_clause | | | |
| quality_flowdown_plan | | | |
| receipt | | | |

### Sales & Quoting

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| customer | | | |
| customer_purchase_order | | | |
| sales_order | | | |
| export_classification | | | |

### Inventory

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| material_lot (physical) | | | |
| inventory_location | | | |
| heat_number (traceability key) | | | |
| tool_gauge | | | |

### Compliance & Governance

| Ontology Entity | [System] Entity | [System] Fields | Gaps |
|---|---|---|---|
| export_classification | | | |
| usml_category | | | |
| data_classification | | | |

## Gaps This System Does Not Cover Natively

List the ontology entities that this system does not model at all — only accessible via custom fields, prose, attachments, or external integrations.

## Patterns Worth Adopting From This System

Counterpart to the "Patterns Worth Adopting" sections elsewhere. What has this system done well that the canonical ontology might absorb? Operational lessons from running this system at scale are welcome.

## Known Deviations From the Canonical Ontology

Places where this system's model conflicts with, rather than just omits, canonical-ontology entities. These are the spots where a canonical-first rewrite would need to actively diverge from the native system's shape.

## Status

- **Mapped against ontology version:** [date + coherence-pass reference]
- **Coverage snapshot:** [rough fraction of entities with a native mapping]
- **Last refreshed:** [YYYY-MM-DD]
