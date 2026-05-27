# HR Core Data Foundation Design

## 1. Purpose

This document defines the phase-1 core HR data foundation for the personnel-ledger MVP.

It covers only the HR core domain:

- organization
- approved person master data
- approved education records
- approved certificate records
- approved attachment metadata
- unified change requests
- approval records
- audit logs

It does not yet cover user, role, data-scope authorization tables, import/export task tables, or template configuration tables.

## 2. Design Goals

The data foundation must satisfy these non-negotiable goals:

- approved data only becomes official after approval
- database is the only source of truth for live business data
- Excel workbooks remain import and export contracts, not operational truth
- organization scope is enforceable on every record path
- summaries are generated from approved detail data only
- personnel, education, certificate, attachment, approval, and audit data remain traceable
- destructive operations default to soft delete or status transition

## 3. Design Choice

The chosen approach is:

- normalized official master data
- one unified `change_request` approval framework
- separate `approval_record` action history
- separate `audit_log` system audit history

This approach was chosen over event sourcing and per-domain draft tables because it best fits the current MVP scope:

- easiest to align with approval-before-effective behavior
- easier to support Excel import review and batch approval later
- easier to generate stable summaries and exports from approved records
- avoids table explosion from parallel draft tables per domain

## 4. Data Layering

### 4.1 Official Master Data Layer

Stores only approved business facts.

Entities:

- `organization`
- `person`
- `education_record`
- `certificate_record`
- `attachment`

Rules:

- all list, detail, summary, export, and reporting queries default to this layer
- users do not directly mutate this layer through normal editing flows
- approval success is the only standard entry point for business changes

### 4.2 Change Request Layer

Stores requested but not-yet-effective changes.

Entity:

- `change_request`

Rules:

- all create, update, and delete operations first write here
- import-generated changes also enter here
- change requests preserve before and after snapshots for review and traceability

### 4.3 Approval Trace Layer

Stores the lifecycle actions that move a request from one state to another.

Entity:

- `approval_record`

Rules:

- `change_request` holds the current state
- `approval_record` holds the action-by-action path

### 4.4 Audit Layer

Stores security and system behavior traces beyond approval itself.

Entity:

- `audit_log`

Rules:

- records sensitive views, downloads, imports, exports, and approval actions
- does not replace business history

## 5. Domain Boundaries

### 5.1 Organization

`organization` is the scope anchor for permissions and reporting.

Each personnel record must belong to one official organization. Child records also preserve organization scope so queries do not need to rely only on joins back to `person`.

### 5.2 Person

`person` stores the current approved employee master profile only.

It does not embed education rows, certificate rows, or attachment rows.

Key ideas:

- one current approved employee record
- employment status is part of the approved master profile
- departure is modeled as a status transition, not hard deletion

### 5.3 Education

`education_record` stores one-to-many approved education experiences for a person.

This is independent from the person master record except for reference and summary-driving fields such as highest education.

### 5.4 Certificate

`certificate_record` stores one-to-many approved title, skill, professional qualification, and other certificate records for a person.

### 5.5 Attachment

`attachment` stores attachment metadata only.

Attachment metadata authorization and physical file download authorization must be treated separately.

## 6. Core Relationships

- `organization` 1:n `person`
- `person` 1:n `education_record`
- `person` 1:n `certificate_record`
- `person` 1:n `attachment`
- `change_request` n:1 `organization`
- `change_request` optional n:1 `person`
- `approval_record` n:1 `change_request`

`change_request` uses:

- `target_type`
- `target_id`

to point at the business object being changed.

Supported phase-1 `target_type` values:

- `PERSON`
- `EDUCATION`
- `CERTIFICATE`
- `ATTACHMENT`

Rules:

- for create requests, `target_id` may be null until approval creates the official record
- for update or delete requests, `target_id` must reference an existing approved record

## 7. Approval Model

### 7.1 Unified Request Model

All data-changing HR operations use one `change_request` table.

Supported phase-1 change categories:

- person create
- person update
- person delete request
- education create or update or delete
- certificate create or update or delete
- attachment create or update or delete
- batch import generated changes

### 7.2 Request Lifecycle

Recommended lifecycle:

- `DRAFT`
- `PENDING`
- `APPROVED`
- `REJECTED`
- `CANCELLED`

Rules:

- only `APPROVED` requests may update official master data
- `REJECTED` and `CANCELLED` requests remain queryable for history
- normal flows do not self-approve by default when an alternative approver exists

### 7.3 Approval Actions

Recommended action types in `approval_record`:

- `CREATE_DRAFT`
- `SUBMIT`
- `APPROVE`
- `REJECT`
- `CANCEL`
- `RESUBMIT`

### 7.4 Effective-Write Rule

Approval pass behavior must run in one transaction:

1. validate request state is `PENDING`
2. apply `after_snapshot` into official target tables
3. update `change_request.status` to `APPROVED`
4. write `approval_record`
5. write `audit_log`

Rejection behavior must not modify official master data.

## 8. MySQL Entity Design

### 8.1 `organization`

Purpose:

- organization ownership
- organization tree filtering
- reporting scope anchor

Suggested columns:

- `id` bigint primary key
- `parent_id` bigint null
- `org_code` varchar(64) not null
- `org_name` varchar(255) not null
- `org_short_name` varchar(128) null
- `org_type` varchar(64) null
- `org_path` varchar(1024) not null
- `org_level` int not null default 1
- `display_order` int not null default 0
- `status` varchar(32) not null default 'ACTIVE'
- `created_at` datetime not null
- `updated_at` datetime not null
- `deleted_at` datetime null

Constraints and notes:

- unique key on `org_code`
- index on `parent_id`
- index on `org_path`
- soft delete only

### 8.2 `person`

Purpose:

- approved current employee master data

Suggested columns:

- `id` bigint primary key
- `org_id` bigint not null
- `employee_no` varchar(64) null
- `employee_name` varchar(128) not null
- `gender` varchar(32) null
- `ethnicity` varchar(64) null
- `native_place` varchar(255) null
- `political_status` varchar(64) null
- `party_join_date` date null
- `birth_date` date null
- `labor_company_name` varchar(255) not null
- `department_name` varchar(255) null
- `job_title` varchar(255) null
- `first_group_entry_date` date null
- `hire_date` date null
- `current_position_start_date` date null
- `highest_education` varchar(64) null
- `work_start_date` date null
- `national_id_number_masked` varchar(64) null
- `national_id_number_ciphertext` text null
- `position_level` varchar(64) null
- `employment_status` varchar(32) not null default 'ACTIVE'
- `departure_date` date null
- `departure_reason` varchar(255) null
- `archive_location` varchar(255) null
- `remarks` varchar(500) null
- `created_at` datetime not null
- `updated_at` datetime not null
- `deleted_at` datetime null

Constraints and notes:

- foreign key `org_id -> organization.id`
- optional unique key on `employee_no` when a stable employee code policy exists
- no plain-text-only national ID storage
- `employment_status` models active versus departed state

### 8.3 `education_record`

Purpose:

- approved one-to-many education history

Suggested columns:

- `id` bigint primary key
- `person_id` bigint not null
- `org_id` bigint not null
- `education_type` varchar(64) not null
- `education_mode` varchar(32) null
- `institution_name` varchar(255) not null
- `major_name` varchar(255) null
- `graduation_date` date null
- `degree_name` varchar(128) null
- `is_highest_education` tinyint(1) not null default 0
- `source_sort_order` int not null default 0
- `remarks` varchar(500) null
- `created_at` datetime not null
- `updated_at` datetime not null
- `deleted_at` datetime null

Constraints and notes:

- foreign key `person_id -> person.id`
- foreign key `org_id -> organization.id`
- keep `org_id` for direct scoped filtering
- `source_sort_order` preserves workbook display ordering where needed

### 8.4 `certificate_record`

Purpose:

- approved one-to-many certificate history

Suggested columns:

- `id` bigint primary key
- `person_id` bigint not null
- `org_id` bigint not null
- `certificate_category` varchar(64) not null
- `certificate_level` varchar(64) null
- `certificate_name` varchar(255) null
- `certificate_major` varchar(255) null
- `certificate_series` varchar(255) null
- `obtained_date` date null
- `is_title_certificate` tinyint(1) not null default 0
- `remarks` varchar(500) null
- `created_at` datetime not null
- `updated_at` datetime not null
- `deleted_at` datetime null

Constraints and notes:

- foreign key `person_id -> person.id`
- foreign key `org_id -> organization.id`
- one unified table is preferred in phase 1 over separate title and other certificate tables

### 8.5 `attachment`

Purpose:

- approved attachment metadata

Suggested columns:

- `id` bigint primary key
- `person_id` bigint not null
- `org_id` bigint not null
- `business_type` varchar(64) not null
- `business_id` bigint null
- `attachment_category` varchar(64) not null
- `file_name` varchar(255) not null
- `file_ext` varchar(32) null
- `storage_path` varchar(1024) not null
- `storage_provider` varchar(64) not null
- `file_size` bigint null
- `mime_type` varchar(128) null
- `file_hash` varchar(128) null
- `is_sensitive` tinyint(1) not null default 1
- `uploaded_by` bigint null
- `uploaded_at` datetime null
- `created_at` datetime not null
- `updated_at` datetime not null
- `deleted_at` datetime null

Constraints and notes:

- foreign key `person_id -> person.id`
- foreign key `org_id -> organization.id`
- `business_type + business_id` supports linking to education, certificate, or person-level objects
- `storage_path` is internal and should not be exposed directly as a public URL

### 8.6 `change_request`

Purpose:

- unified HR mutation request

Suggested columns:

- `id` bigint primary key
- `request_no` varchar(64) not null
- `org_id` bigint not null
- `person_id` bigint null
- `target_type` varchar(32) not null
- `target_id` bigint null
- `change_type` varchar(32) not null
- `source_type` varchar(32) not null
- `batch_no` varchar(64) null
- `before_snapshot` json null
- `after_snapshot` json not null
- `field_diff_summary` json null
- `status` varchar(32) not null default 'DRAFT'
- `requested_by` bigint not null
- `requested_at` datetime not null
- `submitted_at` datetime null
- `current_approver_id` bigint null
- `approved_at` datetime null
- `rejected_at` datetime null
- `rejection_reason` varchar(500) null
- `cancelled_at` datetime null
- `created_at` datetime not null
- `updated_at` datetime not null
- `deleted_at` datetime null

Constraints and notes:

- unique key on `request_no`
- foreign key `org_id -> organization.id`
- optional foreign key `person_id -> person.id`
- index on `status`
- index on `requested_at`
- composite index on `target_type, target_id`
- `before_snapshot` and `after_snapshot` use MySQL `json` in phase 1
- `field_diff_summary` supports lightweight approval list rendering without parsing full snapshots every time

### 8.7 `approval_record`

Purpose:

- action trace for request workflow

Suggested columns:

- `id` bigint primary key
- `change_request_id` bigint not null
- `action` varchar(32) not null
- `from_status` varchar(32) null
- `to_status` varchar(32) null
- `actor_id` bigint not null
- `actor_name` varchar(128) null
- `actor_role` varchar(64) null
- `comment` varchar(500) null
- `acted_at` datetime not null
- `created_at` datetime not null

Constraints and notes:

- foreign key `change_request_id -> change_request.id`
- index on `change_request_id, acted_at`

### 8.8 `audit_log`

Purpose:

- system-level audit trace

Suggested columns:

- `id` bigint primary key
- `actor_id` bigint null
- `actor_name` varchar(128) null
- `actor_org_id` bigint null
- `action_type` varchar(64) not null
- `target_type` varchar(64) null
- `target_id` bigint null
- `request_no` varchar(64) null
- `ip_address` varchar(64) null
- `user_agent` varchar(512) null
- `result` varchar(32) not null
- `detail_json` json null
- `occurred_at` datetime not null
- `created_at` datetime not null

Constraints and notes:

- index on `actor_id, occurred_at`
- index on `target_type, target_id, occurred_at`

## 9. Enumerations And Controlled Values

Recommended phase-1 hard enums:

- `employment_status`: `ACTIVE`, `DEPARTED`
- `target_type`: `PERSON`, `EDUCATION`, `CERTIFICATE`, `ATTACHMENT`
- `change_type`: `CREATE`, `UPDATE`, `DELETE`
- `source_type`: `MANUAL`, `IMPORT`
- `change_request.status`: `DRAFT`, `PENDING`, `APPROVED`, `REJECTED`, `CANCELLED`
- `approval_record.action`: `CREATE_DRAFT`, `SUBMIT`, `APPROVE`, `REJECT`, `CANCEL`, `RESUBMIT`
- `certificate_category`: `TITLE`, `SKILL`, `PROFESSIONAL_QUALIFICATION`, `OTHER`
- `organization.status`: `ACTIVE`, `DISABLED`

Recommended controlled text first, dictionary later:

- `gender`
- `ethnicity`
- `political_status`
- `position_level`
- `education_type`
- `education_mode`
- `certificate_level`

Reason:

- current workbook compatibility matters more than full dictionary governance in phase 1
- this keeps import and export mapping simpler while leaving room for later normalization

## 10. Effective Business Rules

### 10.1 Approval Before Effective

- all create, update, and delete operations first write `change_request`
- normal business edits do not directly mutate official master tables

### 10.2 Approved-Only Official Writes

- only `APPROVED` requests may write to `person`, `education_record`, `certificate_record`, or `attachment`

### 10.3 Rejected And Cancelled History Preservation

- rejected and cancelled requests remain stored and queryable
- they are not overwritten or recycled

### 10.4 Summary And Export Read Rules

- hierarchy summaries, structure summaries, and exports read official approved detail data only
- pending requests never contribute to official summary results

### 10.5 Departure Handling

- departure is modeled primarily through `employment_status = DEPARTED`
- `departure_date` and `departure_reason` capture details
- hard delete is not the default behavior

### 10.6 Attachment Approval Rule

- attachment metadata association is approval-gated
- physical upload may happen earlier into controlled storage
- official business linkage becomes effective only after approval

## 11. Excel Contract Mapping

### 11.1 Personnel Ledger Workbook

Workbook:

- `土投集团人员台账(4).xlsx`

Mapping:

- `数据` sheet maps to active approved person master baseline
- `离职人员` sheet maps to departed employee baseline
- `层级汇总` is a generated summary output
- `人员结构汇总` is a generated summary output

Implication:

- summary sheets are not editable source data

### 11.2 Education And Degree Workbook

Workbook:

- `学历学位证书台账(1).xlsx`

Mapping:

- company-specific sheets map to one unified `education_record` domain with organization ownership
- sheet-per-company is an import or export presentation structure, not a database table split rule

### 11.3 Certificate Summary Workbook

Workbook:

- `数据汇总(2).xlsx`

Mapping:

- `职称证书` maps to certificate detail rows
- `其他证书` maps to certificate detail rows
- `总` is generated summary output

Implication:

- yearly and summary outputs must derive from approved detail records

## 12. Indexing Recommendations

- `person (org_id, employment_status, deleted_at)`
- `education_record (person_id, deleted_at)`
- `certificate_record (person_id, certificate_category, deleted_at)`
- `attachment (person_id, business_type, business_id, deleted_at)`
- `change_request (org_id, status, requested_at)`
- `change_request (target_type, target_id)`
- `approval_record (change_request_id, acted_at)`
- `audit_log (actor_id, occurred_at)`
- `audit_log (target_type, target_id, occurred_at)`

## 13. Explicit Phase-1 Non-Scope

The following are intentionally out of scope for this design:

- user tables
- role tables
- user-role tables
- data-scope authorization grant tables
- import task tables
- export task tables
- export template configuration tables
- online template editing
- multi-step or parallel approval workflows
- BI aggregate storage tables

These may be added later without changing the core assumptions in this document.

## 14. Risks And Follow-Up Decisions

Open decisions that should be confirmed before implementation of import and permission modules:

- whether `employee_no` can be enforced as unique in phase 1
- whether full national ID access is allowed for any non-admin HR role
- whether `position_level` should become a controlled dictionary before summary implementation
- whether attachment file upload should create a temporary file table before approval linkage
- whether subsidiary HR may resubmit rejected requests with the same request number or must create a new request

## 15. Recommended Next Plan Slice

Implementation should start with:

1. schema and migration design for the eight core tables
2. request status transition rules
3. approved-write transaction service
4. organization-scoped query foundations
5. import and export mapping built on top of the approved master data layer
