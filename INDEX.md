# Entity Index

Every entity in the precision-manufacturing-ontology. Alphabetical. One line each. From any entry you can reach the schema, prose, and matrix row in one hop.

Entity names use `snake_case` to match `reference/entity-inventory.yaml` and `schemas/material.yaml`. Archetype display names (e.g., `[Material Specification]`) are a display-layer concern handled elsewhere.

For the prose column, the link points to the primary prose definition for the entity: the extension narrative if one exists, else the schema file, else `reference/domain-map.md`, else `extensions/uuid-discipline.md`, else `reference/composition-archetypes.md`. Entities with no prose definition anywhere are marked `*(undefined — see matrix)*`.

| Entity | Schema | Prose | Matrix |
|--------|--------|-------|--------|
| approved_supplier_list_entry | [schemas/approved_supplier_list_entry.yaml](schemas/approved_supplier_list_entry.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#approved_supplier_list_entry) |
| as9102_form_1 | *(embedded in first_article_inspection — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#as9102_form_1) |
| as9102_form_2 | *(embedded in first_article_inspection — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#as9102_form_2) |
| as9102_form_3 | *(embedded in first_article_inspection — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#as9102_form_3) |
| assembly | [schemas/part.yaml](schemas/part.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#assembly) |
| assembly_operation | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#assembly_operation) |
| assembly_routing | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#assembly_routing) |
| bom_line | [schemas/part.yaml](schemas/part.yaml) | [schemas/part.yaml](schemas/part.yaml) | [matrix row](reference/entity-matrix.md#bom_line) |
| calibration_record | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#calibration_record) |
| certification_document | [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#certification_document) |
| certification_package | [schemas/certification_package.yaml](schemas/certification_package.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#certification_package) |
| characteristic_measurement | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#characteristic_measurement) |
| coc | *(alias of certification_document — cert_type: CoC)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#coc) |
| conflict_minerals_declaration | *(alias of certification_document — cert_type: ConflictMineral)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#conflict_minerals_declaration) |
| country | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#country) |
| customer | [schemas/customer.yaml](schemas/customer.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#customer) |
| customer_approval | [schemas/customer_approval.yaml](schemas/customer_approval.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#customer_approval) |
| customer_asl | [schemas/customer_asl.yaml](schemas/customer_asl.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#customer_asl) |
| customer_purchase_order | [schemas/customer_purchase_order.yaml](schemas/customer_purchase_order.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#customer_purchase_order) |
| data_classification | [schemas/data_classification.yaml](schemas/data_classification.yaml) | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#data_classification) |
| dfars_declaration | *(alias of certification_document — cert_type: DFARS_Declaration)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#dfars_declaration) |
| export_classification | [schemas/export_classification.yaml](schemas/export_classification.yaml) | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#export_classification) |
| export_compliance_declaration | *(none)* | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#export_compliance_declaration) |
| fai_form_1_header | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#fai_form_1_header) |
| fai_package | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#fai_package) |
| fastener | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#fastener) |
| first_article_inspection | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#first_article_inspection) |
| heat_number | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#heat_number) |
| inspection_event | *(none)* | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#inspection_event) |
| inventory_location | [schemas/inventory_location.yaml](schemas/inventory_location.yaml) | [schemas/inventory_location.yaml](schemas/inventory_location.yaml) | [matrix row](reference/entity-matrix.md#inventory_location) |
| item | *(removed from canon — Decision 1.9; use part \| assembly typed union)* | [domain-map §2](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#item) |
| job | [schemas/job.yaml](schemas/job.yaml) | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#job) |
| key_characteristic | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#key_characteristic) |
| kit_record | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#kit_record) |
| machine_event | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#machine_event) |
| material | [schemas/material.yaml](schemas/material.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material) |
| material_condition | [schemas/material_condition.yaml](schemas/material_condition.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material_condition) |
| material_identification | [schemas/material_identification.yaml](schemas/material_identification.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#material_identification) |
| material_lot | [schemas/material.yaml](schemas/material.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material_lot) |
| material_specification | [schemas/material.yaml](schemas/material.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material_specification) |
| mrb_disposition | *(embedded in nonconformance_report — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#mrb_disposition) |
| mtr | *(alias of certification_document — cert_type: MTR)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#mtr) |
| nadcap_category | [schemas/nadcap_category.yaml](schemas/nadcap_category.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#nadcap_category) |
| nadcap_certification | [schemas/nadcap_certification.yaml](schemas/nadcap_certification.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#nadcap_certification) |
| nonconformance_report | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#nonconformance_report) |
| operation | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#operation) |
| outside_process_operation | [schemas/outside_process_operation.yaml](schemas/outside_process_operation.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#outside_process_operation) |
| part | [schemas/part.yaml](schemas/part.yaml) | [domain-map §2](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#part) |
| part_specification | [schemas/part.yaml](schemas/part.yaml) | [domain-map §2](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#part_specification) |
| pmi_event | [schemas/pmi_event.yaml](schemas/pmi_event.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#pmi_event) |
| pmi_report | *(alias of certification_document — cert_type: PMI_Report)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#pmi_report) |
| process_certification | [schemas/outside_process_operation.yaml](schemas/outside_process_operation.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#process_certification) |
| process_identification | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#process_identification) |
| process_specification | [schemas/outside_process_operation.yaml](schemas/outside_process_operation.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#process_specification) |
| purchase_order | [schemas/purchase_order.yaml](schemas/purchase_order.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#purchase_order) |
| qif_results_document | *(none)* | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#qif_results_document) |
| quality_clause | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#quality_clause) |
| quality_engineer | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#quality_engineer) |
| quality_flowdown_plan | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#quality_flowdown_plan) |
| receipt | [schemas/receipt.yaml](schemas/receipt.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#receipt) |
| routing | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#routing) |
| sales_order | [schemas/sales_order.yaml](schemas/sales_order.yaml) | [domain-map §6](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#sales_order) |
| serial_number | [schemas/job.yaml](schemas/job.yaml) | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#serial_number) |
| shelf_life_certification | *(alias of certification_document — cert_type: ShelfLife)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#shelf_life_certification) |
| shipment | [schemas/shipment.yaml](schemas/shipment.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#shipment) |
| shipment_inbound | *(alias of shipment — direction: inbound)* [schemas/shipment.yaml](schemas/shipment.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#shipment_inbound) |
| spec_body | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#spec_body) |
| sub_tier_purchase_order | *(alias of purchase_order — po_kind: sub_tier)* [schemas/purchase_order.yaml](schemas/purchase_order.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#sub_tier_purchase_order) |
| supplier | [schemas/supplier.yaml](schemas/supplier.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#supplier) |
| supplier_approval | *(none)* | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#supplier_approval) |
| supplier_qualification_event | [schemas/supplier_qualification_event.yaml](schemas/supplier_qualification_event.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#supplier_qualification_event) |
| tool_gauge | *(none)* | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#tool_gauge) |
| torque_readings_record | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#torque_readings_record) |
| torque_sequence | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#torque_sequence) |
| torque_specification | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#torque_specification) |
| uns_number | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#uns_number) |
| user | [schemas/user.yaml](schemas/user.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#user) |
| usml_category | *(none)* | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#usml_category) |
| witness_inspection | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#witness_inspection) |
| work_center | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#work_center) |
| work_order | [schemas/job.yaml](schemas/job.yaml) | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#work_order) |
| x12_856_asn | *(none)* | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#x12_856_asn) |

## Notes

- **Entity count:** 82. (`work_order` added in Phase 2.C; `receipt` added in Phase 2.E — both were omitted from initial inventory despite being defined in extension prose.)
- **Schemas:** forty-four entities now have machine-readable schemas. Phase 2.A–2.B (28): `material`, `material_lot`, `material_specification` (in `schemas/material.yaml`); `certification_document` (`schemas/certification_document.yaml`); `material_condition` (`schemas/material_condition.yaml`); `material_identification` (`schemas/material_identification.yaml`); `pmi_event` (`schemas/pmi_event.yaml`); `part`, `assembly`, `part_specification`, `bom_line` (in `schemas/part.yaml`); `export_classification` (`schemas/export_classification.yaml`); `customer_approval` (`schemas/customer_approval.yaml`); `routing`, `operation`, `work_center`, `assembly_routing`, `assembly_operation` (in `schemas/routing.yaml`); `outside_process_operation`, `process_specification`, `process_certification` (in `schemas/outside_process_operation.yaml`); `job`, `serial_number`, `work_order` (in `schemas/job.yaml`); `quality_clause`, `quality_flowdown_plan`, `first_article_inspection`, `nonconformance_report` (in `schemas/quality.yaml`). Phase 2.E (11): `approved_supplier_list_entry`, `customer`, `customer_asl`, `customer_purchase_order`, `nadcap_category`, `nadcap_certification`, `purchase_order`, `receipt`, `shipment`, `supplier`, `supplier_qualification_event`. Phase 2.F (5): `certification_package`, `data_classification`, `inventory_location`, `sales_order`, `user`. Four entities are covered as embedded sub-objects: `as9102_form_1`, `as9102_form_2`, `as9102_form_3` (in `first_article_inspection`), `mrb_disposition` (in `nonconformance_report`). Eight entities are alias-only: `coc`, `dfars_declaration`, `conflict_minerals_declaration`, `mtr`, `pmi_report`, `shelf_life_certification` (cert_type aliases in `certification_document`); `sub_tier_purchase_order` (po_kind: sub_tier alias); `shipment_inbound` (direction: inbound alias). The other 25 entities carry `*(none)*` — a deliberate and scannable gap. (82 total − 44 standalone schemas − 4 embedded − 8 alias-only − 1 removed from canon = 25.)
- **Alphabetical ordering:** ASCII sort on the `snake_case` name; no article handling needed since no entity starts with an article.
- **Matrix anchors:** each matrix entry is rendered as `### <snake_case>`, which standard markdown renderers resolve to `#<snake_case>`. If your renderer mangles underscore anchors, the matrix file is short enough to find the section by name.
- **Open questions, aliases, and supertype decisions** for each entity live in the matrix row linked from this index.
