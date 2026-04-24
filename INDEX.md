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
| calibration_record | [schemas/inspection.yaml](schemas/inspection.yaml) | [inspection-calibration](extensions/inspection-calibration.md) | [matrix row](reference/entity-matrix.md#calibration_record) |
| cam_file | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#cam_file) |
| certification_document | [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#certification_document) |
| certification_package | [schemas/certification_package.yaml](schemas/certification_package.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#certification_package) |
| characteristic | [schemas/characteristic.yaml](schemas/characteristic.yaml) | [schemas/characteristic.yaml](schemas/characteristic.yaml) | [matrix row](reference/entity-matrix.md#characteristic) |
| characteristic_measurement | *(none)* | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#characteristic_measurement) |
| cmm_program | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#cmm_program) |
| coc | *(alias of certification_document — cert_type: CoC)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#coc) |
| conflict_minerals_declaration | *(alias of certification_document — cert_type: ConflictMineral)* [schemas/certification_document.yaml](schemas/certification_document.yaml) | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#conflict_minerals_declaration) |
| control_plan_item | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [schemas/quality_tools.yaml](schemas/quality_tools.yaml) | [matrix row](reference/entity-matrix.md#control_plan_item) |
| country | *(none)* | [material-certs](extensions/material-certs.md) | [matrix row](reference/entity-matrix.md#country) |
| cutter_definition | [schemas/tool_room.yaml](schemas/tool_room.yaml) | [tool-room](extensions/tool-room.md) | [matrix row](reference/entity-matrix.md#cutter_definition) |
| customer | [schemas/customer.yaml](schemas/customer.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#customer) |
| customer_approval | [schemas/customer_approval.yaml](schemas/customer_approval.yaml) | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#customer_approval) |
| customer_asl | [schemas/customer_asl.yaml](schemas/customer_asl.yaml) | [supplier-approval](extensions/supplier-approval.md) | [matrix row](reference/entity-matrix.md#customer_asl) |
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
| prove_out_record | [schemas/process_engineering.yaml](schemas/process_engineering.yaml) | [process-engineering-nre](extensions/process-engineering-nre.md) | [matrix row](reference/entity-matrix.md#prove_out_record) |
| purchase_order | [schemas/purchase_order.yaml](schemas/purchase_order.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#purchase_order) |
| qif_results_document | *(none)* | [uuid-discipline](extensions/uuid-discipline.md) | [matrix row](reference/entity-matrix.md#qif_results_document) |
| quality_clause | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#quality_clause) |
| quality_engineer | *(none)* | [composition-archetypes](reference/composition-archetypes.md) | [matrix row](reference/entity-matrix.md#quality_engineer) |
| quality_flowdown_plan | [schemas/quality.yaml](schemas/quality.yaml) | [quality-flowdowns](extensions/quality-flowdowns.md) | [matrix row](reference/entity-matrix.md#quality_flowdown_plan) |
| receipt | [schemas/receipt.yaml](schemas/receipt.yaml) | [outside-processing](extensions/outside-processing.md) | [matrix row](reference/entity-matrix.md#receipt) |
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

- **Entity count:** 123 (82 original + 35 Phase 2 additions + 6 InspectAI-derived additions: `characteristic`, `measurement_result` (in `schemas/characteristic.yaml`, `schemas/measurement_result.yaml`); `control_plan_item`, `fmea_item`, `ppap_submission`, `spc_result` (in `schemas/quality_tools.yaml`)).
- **Schemas:** 94 entities now have machine-readable schemas. Phase 2.A–2.F (44 original): unchanged. Phase 3.4 fixes (9): `calibration_record`, `inspection_event`, `key_characteristic`, `tool_gauge` (in `schemas/inspection.yaml`); `kit_record`, `torque_readings_record`, `torque_sequence`, `torque_specification`, `witness_inspection` (in `schemas/assembly.yaml`). Phase 2 additions (35): `cam_file`, `cmm_program`, `fixture_use`, `inspection_plan`, `nc_program`, `nre_package`, `print_file`, `prove_out_record`, `setup_sheet`, `tooling_list`, `tooling_list_line` (in `schemas/process_engineering.yaml`); `cutter_definition`, `gauge_definition`, `holder_definition`, `tool_assembly_definition`, `tool_life_record`, `tool_maintenance_event`, `tool_pocket_assignment`, `tool_pocket_requirement`, `tool_preset`, `tool_room_allocation` (in `schemas/tool_room.yaml`); `packaging_item`, `packaging_specification`, `packing_record` (in `schemas/packaging.yaml`); `deviation_waiver`, `engineering_change_notice` (in `schemas/change_management.yaml`); `insert_installation_record`, `press_fit_record` (in `schemas/assembly_hardware.yaml`); `operator_qualification` (in `schemas/operator_qualification.yaml`). InspectAI-derived (6): `characteristic`, `measurement_result`, `fmea_item`, `control_plan_item`, `spc_result`, `ppap_submission`.
- **Alphabetical ordering:** ASCII sort on the `snake_case` name; no article handling needed since no entity starts with an article.
- **Matrix anchors:** each matrix entry is rendered as `### <snake_case>`, which standard markdown renderers resolve to `#<snake_case>`. If your renderer mangles underscore anchors, the matrix file is short enough to find the section by name.
- **insert_installation_record / press_fit_record prose:** linked to `extensions/change-management.md` which covers all assembly hardware extensions alongside the ECN/waiver content.
