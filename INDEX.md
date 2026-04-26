# Entity Index

Every entity in the precision-manufacturing-ontology. Alphabetical. One line each. From any entry you can reach the schema, prose, and matrix row in one hop.

Entity names use `snake_case` to match `reference/entity-inventory.yaml` and `schemas/material.yaml`. Archetype display names (e.g., `[Material Specification]`) are a display-layer concern handled elsewhere.

For the prose column, the link points to the primary prose definition for the entity: the extension narrative if one exists, else the schema file, else `reference/domain-map.md`, else `extensions/uuid-discipline.md`, else `reference/composition-archetypes.md`. Entities with no prose definition anywhere are marked `*(undefined — see matrix)*`.

| Entity | Schema | Prose | Matrix |
|--------|--------|-------|--------|
| approved_supplier_list_entry | [schemas/approved_supplier_list_entry.yaml](schemas/approved_supplier_list_entry.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#approved_supplier_list_entry) |
| audit_finding | [schemas/corrective_action.yaml](schemas/corrective_action.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#audit_finding) |
| audit_plan | [schemas/corrective_action.yaml](schemas/corrective_action.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#audit_plan) |
| as9102_form_1 | *(embedded in first_article_inspection — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#as9102_form_1) |
| as9102_form_2 | *(embedded in first_article_inspection — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#as9102_form_2) |
| as9102_form_3 | *(embedded in first_article_inspection — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#as9102_form_3) |
| assembly | [schemas/part.yaml](schemas/part.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#assembly) |
| assembly_operation | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#assembly_operation) |
| assembly_routing | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#assembly_routing) |
| bom_line | [schemas/part.yaml](schemas/part.yaml) | [schemas/part.yaml](schemas/part.yaml) | [matrix row](reference/entity-matrix.md#bom_line) |
| calibration_record | [schemas/inspection.yaml](schemas/inspection.yaml) | [inspection-calibration](extensions/inspection-calibration.md) | [matrix row](reference/entity-matrix.md#calibration_record) |
| cam_file | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#cam_file) |
| certification_document | [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#certification_document) |
| certification_package | [schemas/certification_package.yaml](schemas/certification_package.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#certification_package) |
| characteristic | [schemas/characteristic.yaml](schemas/characteristic.yaml) | [schemas/characteristic.yaml](schemas/characteristic.yaml) | [matrix row](reference/entity-matrix.md#characteristic) |
| characteristic_measurement | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#characteristic_measurement) |
| cmm_program | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#cmm_program) |
| coc | *(alias of certification_document — cert_type: CoC)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#coc) |
| conflict_minerals_declaration | *(alias of certification_document — cert_type: ConflictMineral)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#conflict_minerals_declaration) |
| contract_review_event | [schemas/orders.yaml](schemas/orders.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#contract_review_event) |
| control_plan_item | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [matrix row](reference/entity-matrix.md#control_plan_item) |
| corrective_action | [schemas/corrective_action.yaml](schemas/corrective_action.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#corrective_action) |
| country | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#country) |
| cutter_definition | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#cutter_definition) |
| customer | [schemas/customer.yaml](schemas/customer.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#customer) |
| customer_approval | [schemas/customer_approval.yaml](schemas/customer_approval.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#customer_approval) |
| customer_asl | [schemas/customer_asl.yaml](schemas/customer_asl.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#customer_asl) |
| customer_complaint | [schemas/orders.yaml](schemas/orders.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#customer_complaint) |
| customer_purchase_order | [schemas/customer_purchase_order.yaml](schemas/customer_purchase_order.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#customer_purchase_order) |
| data_classification | [schemas/data_classification.yaml](schemas/data_classification.yaml) | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#data_classification) |
| deviation_waiver | [schemas/change_management.yaml](schemas/change_management.yaml) | [change-management](extensions/change-management.md) | [matrix row](reference/entity-matrix.md#deviation_waiver) |
| dfars_declaration | *(alias of certification_document — cert_type: DFARS_Declaration)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#dfars_declaration) |
| engineering_change_notice | [schemas/change_management.yaml](schemas/change_management.yaml) | [change-management](extensions/change-management.md) | [matrix row](reference/entity-matrix.md#engineering_change_notice) |
| export_classification | [schemas/export_classification.yaml](schemas/export_classification.yaml) | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#export_classification) |
| export_compliance_declaration | *(none)* | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#export_compliance_declaration) |
| fai_form_1_header | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#fai_form_1_header) |
| fai_package | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#fai_package) |
| fastener | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#fastener) |
| first_article_inspection | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#first_article_inspection) |
| fixture_use | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#fixture_use) |
| fmea_item | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [matrix row](reference/entity-matrix.md#fmea_item) |
| gauge_definition | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#gauge_definition) |
| heat_number | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#heat_number) |
| holder_definition | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#holder_definition) |
| insert_installation_record | [schemas/assembly_hardware.yaml](schemas/assembly_hardware.yaml) | [change-management](extensions/change-management.md) | [matrix row](reference/entity-matrix.md#insert_installation_record) |
| inspection_event | [schemas/inspection.yaml](schemas/inspection.yaml) | [inspection-calibration](extensions/inspection-calibration.md) | [matrix row](reference/entity-matrix.md#inspection_event) |
| inspection_plan | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#inspection_plan) |
| inventory_location | [schemas/inventory_location.yaml](schemas/inventory_location.yaml) | [schemas/inventory_location.yaml](schemas/inventory_location.yaml) | [matrix row](reference/entity-matrix.md#inventory_location) |
| item | *(removed from canon — Decision 1.9; use part \| assembly typed union)* | [domain-map §2](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#item) |
| job | [schemas/job.yaml](schemas/job.yaml) | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#job) |
| key_characteristic | [schemas/inspection.yaml](schemas/inspection.yaml) | [inspection-calibration](extensions/inspection-calibration.md) | [matrix row](reference/entity-matrix.md#key_characteristic) |
| kit_record | [schemas/assembly.yaml](schemas/assembly.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#kit_record) |
| lessons_learned | [schemas/corrective_action.yaml](schemas/corrective_action.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#lessons_learned) |
| machine_event | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#machine_event) |
| material | [schemas/material.yaml](schemas/material.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material) |
| material_condition | [schemas/material_condition.yaml](schemas/material_condition.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material_condition) |
| material_identification | [schemas/material_identification.yaml](schemas/material_identification.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#material_identification) |
| material_lot | [schemas/material.yaml](schemas/material.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material_lot) |
| material_specification | [schemas/material.yaml](schemas/material.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#material_specification) |
| measurement_result | [schemas/measurement_result.yaml](schemas/measurement_result.yaml) | [schemas/measurement_result.yaml](schemas/measurement_result.yaml) | [matrix row](reference/entity-matrix.md#measurement_result) |
| mrb_disposition | *(embedded in nonconformance_report — see [schemas/quality.yaml](schemas/quality.yaml))* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#mrb_disposition) |
| mtr | *(alias of certification_document — cert_type: MTR)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#mtr) |
| nadcap_category | [schemas/nadcap_category.yaml](schemas/nadcap_category.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#nadcap_category) |
| nadcap_certification | [schemas/nadcap_certification.yaml](schemas/nadcap_certification.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#nadcap_certification) |
| nc_program | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#nc_program) |
| nonconformance_report | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#nonconformance_report) |
| nre_package | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#nre_package) |
| operation | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#operation) |
| operational_risk_assessment | [schemas/corrective_action.yaml](schemas/corrective_action.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#operational_risk_assessment) |
| operator_qualification | [schemas/operator_qualification.yaml](schemas/operator_qualification.yaml) | [schemas/operator_qualification.yaml](schemas/operator_qualification.yaml) | [matrix row](reference/entity-matrix.md#operator_qualification) |
| outside_process_operation | [schemas/outside_process_operation.yaml](schemas/outside_process_operation.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#outside_process_operation) |
| packaging_item | [schemas/packaging.yaml](schemas/packaging.yaml) | [packaging](extensions/packaging.md) | [matrix row](reference/entity-matrix.md#packaging_item) |
| packaging_specification | [schemas/packaging.yaml](schemas/packaging.yaml) | [packaging](extensions/packaging.md) | [matrix row](reference/entity-matrix.md#packaging_specification) |
| packing_record | [schemas/packaging.yaml](schemas/packaging.yaml) | [packaging](extensions/packaging.md) | [matrix row](reference/entity-matrix.md#packing_record) |
| part | [schemas/part.yaml](schemas/part.yaml) | [domain-map §2](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#part) |
| part_specification | [schemas/part.yaml](schemas/part.yaml) | [domain-map §2](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#part_specification) |
| pmi_event | [schemas/pmi_event.yaml](schemas/pmi_event.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#pmi_event) |
| pmi_report | *(alias of certification_document — cert_type: PMI_Report)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#pmi_report) |
| ppap_submission | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [matrix row](reference/entity-matrix.md#ppap_submission) |
| press_fit_record | [schemas/assembly_hardware.yaml](schemas/assembly_hardware.yaml) | [change-management](extensions/change-management.md) | [matrix row](reference/entity-matrix.md#press_fit_record) |
| print_file | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#print_file) |
| process_certification | [schemas/outside_process_operation.yaml](schemas/outside_process_operation.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#process_certification) |
| process_identification | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#process_identification) |
| process_specification | [schemas/outside_process_operation.yaml](schemas/outside_process_operation.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#process_specification) |
| product_release_authorization | [schemas/inspection.yaml](schemas/inspection.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#product_release_authorization) |
| prove_out_record | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#prove_out_record) |
| purchase_order | [schemas/purchase_order.yaml](schemas/purchase_order.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#purchase_order) |
| qif_results_document | *(none)* | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#qif_results_document) |
| quality_clause | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#quality_clause) |
| quality_engineer | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#quality_engineer) |
| quality_flowdown_plan | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#quality_flowdown_plan) |
| receipt | [schemas/receipt.yaml](schemas/receipt.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#receipt) |
| requirement_change_notification | [schemas/orders.yaml](schemas/orders.yaml) | [as9100-qms-layer](extensions/as9100-qms-layer.md) | [matrix row](reference/entity-matrix.md#requirement_change_notification) |
| routing | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#routing) |
| sales_order | [schemas/sales_order.yaml](schemas/sales_order.yaml) | [domain-map §6](reference/domain-map.md) | [matrix row](reference/entity-matrix.md#sales_order) |
| serial_number | [schemas/job.yaml](schemas/job.yaml) | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#serial_number) |
| setup_sheet | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#setup_sheet) |
| shelf_life_certification | *(alias of certification_document — cert_type: ShelfLife)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#shelf_life_certification) |
| shipment | [schemas/shipment.yaml](schemas/shipment.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#shipment) |
| shipment_inbound | *(alias of shipment — direction: inbound)* [schemas/shipment.yaml](schemas/shipment.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#shipment_inbound) |
| spc_result | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [matrix row](reference/entity-matrix.md#spc_result) |
| spec_body | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#spec_body) |
| sub_tier_purchase_order | *(alias of purchase_order — po_kind: sub_tier)* [schemas/purchase_order.yaml](schemas/purchase_order.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#sub_tier_purchase_order) |
| supplier | [schemas/supplier.yaml](schemas/supplier.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#supplier) |
| supplier_approval | *(none)* | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#supplier_approval) |
| supplier_qualification_event | [schemas/supplier_qualification_event.yaml](schemas/supplier_qualification_event.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#supplier_qualification_event) |
| tool_assembly_definition | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#tool_assembly_definition) |
| tool_gauge | [schemas/inspection.yaml](schemas/inspection.yaml) | [inspection-calibration](extensions/inspection-calibration.md) | [matrix row](reference/entity-matrix.md#tool_gauge) |
| tool_life_record | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#tool_life_record) |
| tool_maintenance_event | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#tool_maintenance_event) |
| tool_pocket_assignment | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#tool_pocket_assignment) |
| tool_pocket_requirement | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#tool_pocket_requirement) |
| tool_preset | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#tool_preset) |
| tool_room_allocation | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#tool_room_allocation) |
| tooling_list | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#tooling_list) |
| tooling_list_line | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#tooling_list_line) |
| torque_readings_record | [schemas/assembly.yaml](schemas/assembly.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#torque_readings_record) |
| torque_sequence | [schemas/assembly.yaml](schemas/assembly.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#torque_sequence) |
| torque_specification | [schemas/assembly.yaml](schemas/assembly.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#torque_specification) |
| uns_number | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#uns_number) |
| user | [schemas/user.yaml](schemas/user.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#user) |
| usml_category | *(none)* | [ip-boundary](extensions/ip-boundary.md) | [matrix row](reference/entity-matrix.md#usml_category) |
| witness_inspection | [schemas/assembly.yaml](schemas/assembly.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#witness_inspection) |
| work_center | [schemas/routing.yaml](schemas/routing.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#work_center) |
| work_order | [schemas/job.yaml](schemas/job.yaml) | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#work_order) |
| x12_856_asn | *(none)* | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#x12_856_asn) |

## Notes

- **Entity count:** 126 (117 prior + 9 AS9100D QMS layer additions 2026-04-26: `audit_finding`, `audit_plan`, `contract_review_event`, `corrective_action`, `customer_complaint`, `lessons_learned`, `operational_risk_assessment`, `product_release_authorization`, `requirement_change_notification`).
- **Schemas:** 115 entities now have machine-readable schemas. Prior count: 106 (see Phase 2 + InspectAI notes). AS9100D QMS additions (9): `audit_finding`, `audit_plan`, `corrective_action`, `lessons_learned`, `operational_risk_assessment` (in `schemas/corrective_action.yaml`); `contract_review_event`, `customer_complaint`, `requirement_change_notification` (in `schemas/orders.yaml`); `product_release_authorization` (added to `schemas/inspection.yaml`). Field additions to existing schemas: `schemas/customer_purchase_order.yaml` (`special_requirements[]`, `operational_risk_flags[]`, `contract_review_id`); `schemas/quality.yaml` (`nonconformance_report`: `counterfeit_flag`, `scar_supplier_id`, `corrective_action_id`; `first_article_inspection`: `repeat_reason_ecn_id`); `schemas/inspection.yaml` (`inspection_event`: `authorized_by`, `release_type`); `schemas/approved_supplier_list_entry.yaml` (`formal_approval_status`, `approval_scope_description`, `last_performance_review_date`, `performance_review_cadence_months`, `counterfeit_risk_tier`).
- **Alphabetical ordering:** ASCII sort on the `snake_case` name; no article handling needed since no entity starts with an article.
- **Matrix anchors:** each matrix entry is rendered as `### <snake_case>`, which standard markdown renderers resolve to `#<snake_case>`. If your renderer mangles underscore anchors, the matrix file is short enough to find the section by name.
- **insert_installation_record / press_fit_record prose:** linked to `extensions/change-management.md` which covers all assembly hardware extensions alongside the ECN/waiver content.
