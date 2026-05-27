# HR Data Dictionary

## 1. Purpose

This is the initial field dictionary for the personnel-ledger MVP.

It normalizes the current workbook-driven business language into system-facing field names and flags the fields that are sensitive, approval-controlled, or summary-driving.

## 2. Dictionary Rules

- `system_name` is the recommended canonical field name for code and API contracts.
- `source_label` is the workbook-facing label or source concept.
- `required_phase1` means the field should be supported in the first ledger-first MVP.
- `sensitive` means the field must be masked, tightly permissioned, or carefully audited.
- `summary_driver` means the field participates in hierarchy or structure summaries.

## 3. Person Master Fields

| system_name | source_label | required_phase1 | sensitive | summary_driver | notes |
| --- | --- | --- | --- | --- | --- |
| employee_name | 姓名 | yes | medium | yes | Primary display name |
| gender | 性别 | yes | low | yes | Used in structure summary |
| ethnicity | 民族 | yes | medium | yes | Minority summary depends on this field |
| native_place | 籍贯 | no | medium | no | Keep as free-text or controlled dictionary later |
| political_status | 政治面貌 | yes | medium | yes | Includes party membership categories |
| party_join_date | 入党时间 | no | medium | no | Relevant when political status requires it |
| birth_date | 出生年月 | yes | high | yes | Prefer normalized date storage |
| age | 年龄 | derived | medium | yes | Compute from birth date when possible |
| labor_company_name | 公司（劳动关系） | yes | low | yes | Main organization ownership field |
| department_name | 部门 | yes | low | no | Permission scoping may reference this later |
| job_title | 职务 | yes | low | no | Current position title |
| first_group_entry_date | 首次进入集团时间 | no | medium | no | Historical tenure baseline |
| hire_date | 入职时间 | yes | medium | no | Current employment start |
| current_position_start_date | 任现职时间 | no | medium | no | Used for tenure and history |
| highest_education | 最高学历 | yes | low | yes | High-level summary field |
| work_start_date | 参加工作时间 | no | medium | no | Historical career baseline |
| national_id_number | 身份证号码 | yes | high | no | Strict masking and audit required |
| supervision_flag | 监察对象 | no | high | no | Sensitive compliance attribute |
| position_level | 层级 | yes | low | yes | Drives hierarchy summary |
| employment_status | 在职/离职状态 | yes | low | yes | System field derived from active or departed ledger |
| archive_location | 档案位置 | no | high | no | Mentioned in requirements doc, not yet shown in current workbook |

## 4. Education Fields

| system_name | source_label | required_phase1 | sensitive | summary_driver | notes |
| --- | --- | --- | --- | --- | --- |
| education_type | 学历 | yes | low | yes | Full-time and part-time records both use this concept |
| degree_name | 学位 / 取得学位 | yes | low | no | Can be empty |
| institution_name | 毕业院校 | yes | low | no | Source workbook uses school name directly |
| major_name | 专业 / 所学专业 | yes | low | no | Preserve source wording in import mapping |
| graduation_date | 毕业时间 | yes | medium | no | Normalize to date |
| education_mode | 全日制教育 / 非全日制教育 | yes | low | no | Recommended enum: full_time / part_time |
| education_certificate_file | 学历证书 | no | high | no | Attachment-controlled |
| degree_certificate_file | 学位证书 | no | high | no | Attachment-controlled |

## 5. Certificate Fields

| system_name | source_label | required_phase1 | sensitive | summary_driver | notes |
| --- | --- | --- | --- | --- | --- |
| certificate_category | 职称证书 / 技能证书 / 职业资格证书 / 其他证书 | yes | low | yes | Core certificate classification |
| certificate_level | 级别 / 职称级别 / 证书级别 | yes | low | yes | Used in yearly and title summaries |
| certificate_major | 专业 / 专业名称 | no | low | no | Meaning depends on certificate type |
| certificate_name | 职称名称 / 职业资格名称 | no | low | no | Keep flexible by subtype |
| certificate_series | 专业类别 / 专业序列 / 类别 | no | low | no | Source-specific classification |
| certificate_obtained_date | 取得时间 | yes | medium | yes | Normalize to date |
| certificate_file | 文件 / 扫描件 | no | high | no | Permission-controlled attachment |

## 6. Departure Fields

| system_name | source_label | required_phase1 | sensitive | summary_driver | notes |
| --- | --- | --- | --- | --- | --- |
| departure_date | 离职日期 | yes | medium | yes | Required for departed staff records |
| departure_reason | 离职备注 / 离职说明 | no | medium | no | Current workbook sometimes mixes reason into date text |
| departure_status | 离职状态 | yes | low | yes | Recommended normalized enum |

## 7. Approval And Audit Fields

| system_name | source_label | required_phase1 | sensitive | summary_driver | notes |
| --- | --- | --- | --- | --- | --- |
| change_request_type | 变更类型 | yes | low | no | Add, edit, delete, import batch, attachment change |
| change_request_status | 审批状态 | yes | low | no | pending / approved / rejected |
| requested_by | 申请人 | yes | medium | no | Internal user reference |
| requested_at | 申请时间 | yes | medium | no | Audit timestamp |
| approved_by | 审批人 | yes | medium | no | Internal user reference |
| approved_at | 审批时间 | yes | medium | no | Audit timestamp |
| approval_comment | 审批意见 | no | medium | no | Preserve rejection reasons |
| change_snapshot | 变更前后对比 | yes | high | no | Store before/after payload or normalized history |

## 8. Summary Dimensions

### 8.1 Hierarchy Summary

- company_group_type
- company_name
- labor_company_name
- position_level

### 8.2 Structure Summary

- gender
- ethnicity
- political_status
- age or age_band
- highest_education
- certificate_level

## 9. Open Mapping Decisions

- Confirm whether `出生年月` should always be stored as a full date or allow month-only legacy values.
- Confirm whether `离职日期` free-text reasons should be split during import or preserved as a raw note.
- Confirm whether `监察对象` is boolean-only or needs a richer compliance classification.
- Confirm the final enum list for `position_level` from the active workbook baseline.
