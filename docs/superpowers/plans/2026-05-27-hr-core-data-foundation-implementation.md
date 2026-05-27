# HR Core Data Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the first executable version of the HR core data foundation as a MySQL 8 schema package with approval-safe table design, repeatable local validation, and versioned commits.

**Architecture:** Start SQL-first because the repository has approved business and data specs but no application code yet. Build a reproducible local MySQL runner, then add the eight core tables in versioned migrations, then add indexes and schema smoke checks so later application code can attach to a stable database contract.

**Tech Stack:** Docker Compose, MySQL 8.4, SQL migration files, Markdown verification notes, Git

---

### Task 1: Create The Local MySQL Runner And Failing Schema Smoke Test

**Files:**
- Create: `compose.yaml`
- Create: `db/tests/001_hr_core_table_count.sql`
- Create: `db/tests/README.md`

- [ ] **Step 1: Write the failing schema smoke test first**

```sql
SELECT
  CASE
    WHEN COUNT(*) = 8 THEN 'PASS'
    ELSE CONCAT('FAIL expected 8 core tables but found ', COUNT(*))
  END AS schema_check
FROM information_schema.tables
WHERE table_schema = 'hrsystem_dev'
  AND table_name IN (
    'organization',
    'person',
    'education_record',
    'certificate_record',
    'attachment',
    'change_request',
    'approval_record',
    'audit_log'
  );
```

- [ ] **Step 2: Add a reproducible local MySQL service**

```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: hrsystem-mysql
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default-time-zone=+08:00
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: hrsystem_dev
      MYSQL_USER: hr_app
      MYSQL_PASSWORD: hr_app
    ports:
      - "3307:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

- [ ] **Step 3: Document the test entrypoints**

~~~md
# DB Test Commands

Start MySQL:

```bash
docker compose up -d mysql
```

Run the schema smoke test:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/tests/001_hr_core_table_count.sql
```

Expected before migrations:

```text
FAIL expected 8 core tables but found 0
```
~~~

- [ ] **Step 4: Run the smoke test and verify it fails**

Run:

```bash
docker compose up -d mysql
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/tests/001_hr_core_table_count.sql
```

Expected:

```text
FAIL expected 8 core tables but found 0
```

- [ ] **Step 5: Commit version `v0.1.0`**

```bash
git add compose.yaml db/tests/001_hr_core_table_count.sql db/tests/README.md
git commit -m "chore: add v0.1.0 mysql runner and schema smoke test"
git push
```

### Task 2: Add The Official Organization And Person Master Tables

**Files:**
- Create: `db/migrations/0001_create_organization_and_person.sql`
- Test: `db/tests/001_hr_core_table_count.sql`

- [ ] **Step 1: Write the first migration for official master roots**

```sql
CREATE TABLE IF NOT EXISTS organization (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  parent_id BIGINT NULL,
  org_code VARCHAR(64) NOT NULL,
  org_name VARCHAR(255) NOT NULL,
  org_short_name VARCHAR(128) NULL,
  org_type VARCHAR(64) NULL,
  org_path VARCHAR(1024) NOT NULL,
  org_level INT NOT NULL DEFAULT 1,
  display_order INT NOT NULL DEFAULT 0,
  status VARCHAR(32) NOT NULL DEFAULT 'ACTIVE',
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at DATETIME NULL,
  UNIQUE KEY uk_organization_org_code (org_code),
  KEY idx_organization_parent_id (parent_id),
  KEY idx_organization_org_path (org_path)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS person (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  org_id BIGINT NOT NULL,
  employee_no VARCHAR(64) NULL,
  employee_name VARCHAR(128) NOT NULL,
  gender VARCHAR(32) NULL,
  ethnicity VARCHAR(64) NULL,
  native_place VARCHAR(255) NULL,
  political_status VARCHAR(64) NULL,
  party_join_date DATE NULL,
  birth_date DATE NULL,
  labor_company_name VARCHAR(255) NOT NULL,
  department_name VARCHAR(255) NULL,
  job_title VARCHAR(255) NULL,
  first_group_entry_date DATE NULL,
  hire_date DATE NULL,
  current_position_start_date DATE NULL,
  highest_education VARCHAR(64) NULL,
  work_start_date DATE NULL,
  national_id_number_masked VARCHAR(64) NULL,
  national_id_number_ciphertext TEXT NULL,
  position_level VARCHAR(64) NULL,
  employment_status VARCHAR(32) NOT NULL DEFAULT 'ACTIVE',
  departure_date DATE NULL,
  departure_reason VARCHAR(255) NULL,
  archive_location VARCHAR(255) NULL,
  remarks VARCHAR(500) NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at DATETIME NULL,
  CONSTRAINT fk_person_org FOREIGN KEY (org_id) REFERENCES organization(id),
  KEY idx_person_org_status_deleted (org_id, employment_status, deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

- [ ] **Step 2: Run the migration**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/migrations/0001_create_organization_and_person.sql
```

Expected:

```text
Query OK
```

- [ ] **Step 3: Re-run the smoke test and confirm it still fails for partial schema**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/tests/001_hr_core_table_count.sql
```

Expected:

```text
FAIL expected 8 core tables but found 2
```

- [ ] **Step 4: Sanity-check critical columns**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot -e "DESCRIBE hrsystem_dev.organization; DESCRIBE hrsystem_dev.person;"
```

Expected:

```text
org_code varchar(64) NO UNI
employment_status varchar(32) NO
national_id_number_ciphertext text YES
```

- [ ] **Step 5: Commit version `v0.2.0`**

```bash
git add db/migrations/0001_create_organization_and_person.sql
git commit -m "feat: add v0.2.0 organization and person schema"
git push
```

### Task 3: Add Education, Certificate, And Attachment Tables

**Files:**
- Create: `db/migrations/0002_create_person_child_records.sql`
- Test: `db/tests/001_hr_core_table_count.sql`

- [ ] **Step 1: Write the child-record migration**

```sql
CREATE TABLE IF NOT EXISTS education_record (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  person_id BIGINT NOT NULL,
  org_id BIGINT NOT NULL,
  education_type VARCHAR(64) NOT NULL,
  education_mode VARCHAR(32) NULL,
  institution_name VARCHAR(255) NOT NULL,
  major_name VARCHAR(255) NULL,
  graduation_date DATE NULL,
  degree_name VARCHAR(128) NULL,
  is_highest_education TINYINT(1) NOT NULL DEFAULT 0,
  source_sort_order INT NOT NULL DEFAULT 0,
  remarks VARCHAR(500) NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at DATETIME NULL,
  CONSTRAINT fk_education_person FOREIGN KEY (person_id) REFERENCES person(id),
  CONSTRAINT fk_education_org FOREIGN KEY (org_id) REFERENCES organization(id),
  KEY idx_education_person_deleted (person_id, deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS certificate_record (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  person_id BIGINT NOT NULL,
  org_id BIGINT NOT NULL,
  certificate_category VARCHAR(64) NOT NULL,
  certificate_level VARCHAR(64) NULL,
  certificate_name VARCHAR(255) NULL,
  certificate_major VARCHAR(255) NULL,
  certificate_series VARCHAR(255) NULL,
  obtained_date DATE NULL,
  is_title_certificate TINYINT(1) NOT NULL DEFAULT 0,
  remarks VARCHAR(500) NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at DATETIME NULL,
  CONSTRAINT fk_certificate_person FOREIGN KEY (person_id) REFERENCES person(id),
  CONSTRAINT fk_certificate_org FOREIGN KEY (org_id) REFERENCES organization(id),
  KEY idx_certificate_person_category_deleted (person_id, certificate_category, deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS attachment (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  person_id BIGINT NOT NULL,
  org_id BIGINT NOT NULL,
  business_type VARCHAR(64) NOT NULL,
  business_id BIGINT NULL,
  attachment_category VARCHAR(64) NOT NULL,
  file_name VARCHAR(255) NOT NULL,
  file_ext VARCHAR(32) NULL,
  storage_path VARCHAR(1024) NOT NULL,
  storage_provider VARCHAR(64) NOT NULL,
  file_size BIGINT NULL,
  mime_type VARCHAR(128) NULL,
  file_hash VARCHAR(128) NULL,
  is_sensitive TINYINT(1) NOT NULL DEFAULT 1,
  uploaded_by BIGINT NULL,
  uploaded_at DATETIME NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at DATETIME NULL,
  CONSTRAINT fk_attachment_person FOREIGN KEY (person_id) REFERENCES person(id),
  CONSTRAINT fk_attachment_org FOREIGN KEY (org_id) REFERENCES organization(id),
  KEY idx_attachment_person_business_deleted (person_id, business_type, business_id, deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

- [ ] **Step 2: Run the migration**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/migrations/0002_create_person_child_records.sql
```

Expected:

```text
Query OK
```

- [ ] **Step 3: Re-run the smoke test and confirm partial progress**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/tests/001_hr_core_table_count.sql
```

Expected:

```text
FAIL expected 8 core tables but found 5
```

- [ ] **Step 4: Verify the child-table foreign keys**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot -e "SELECT table_name, constraint_name FROM information_schema.table_constraints WHERE table_schema='hrsystem_dev' AND constraint_type='FOREIGN KEY' AND table_name IN ('education_record','certificate_record','attachment') ORDER BY table_name, constraint_name;"
```

Expected:

```text
attachment fk_attachment_org
attachment fk_attachment_person
certificate_record fk_certificate_org
certificate_record fk_certificate_person
education_record fk_education_org
education_record fk_education_person
```

- [ ] **Step 5: Commit version `v0.3.0`**

```bash
git add db/migrations/0002_create_person_child_records.sql
git commit -m "feat: add v0.3.0 education certificate and attachment schema"
git push
```

### Task 4: Add Change Request, Approval Record, And Audit Log Tables

**Files:**
- Create: `db/migrations/0003_create_approval_and_audit_tables.sql`
- Test: `db/tests/001_hr_core_table_count.sql`

- [ ] **Step 1: Write the workflow and audit migration**

```sql
CREATE TABLE IF NOT EXISTS change_request (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  request_no VARCHAR(64) NOT NULL,
  org_id BIGINT NOT NULL,
  person_id BIGINT NULL,
  target_type VARCHAR(32) NOT NULL,
  target_id BIGINT NULL,
  change_type VARCHAR(32) NOT NULL,
  source_type VARCHAR(32) NOT NULL,
  batch_no VARCHAR(64) NULL,
  before_snapshot JSON NULL,
  after_snapshot JSON NOT NULL,
  field_diff_summary JSON NULL,
  status VARCHAR(32) NOT NULL DEFAULT 'DRAFT',
  requested_by BIGINT NOT NULL,
  requested_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  submitted_at DATETIME NULL,
  current_approver_id BIGINT NULL,
  approved_at DATETIME NULL,
  rejected_at DATETIME NULL,
  rejection_reason VARCHAR(500) NULL,
  cancelled_at DATETIME NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at DATETIME NULL,
  UNIQUE KEY uk_change_request_request_no (request_no),
  CONSTRAINT fk_change_request_org FOREIGN KEY (org_id) REFERENCES organization(id),
  CONSTRAINT fk_change_request_person FOREIGN KEY (person_id) REFERENCES person(id),
  KEY idx_change_request_org_status_requested (org_id, status, requested_at),
  KEY idx_change_request_target (target_type, target_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS approval_record (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  change_request_id BIGINT NOT NULL,
  action VARCHAR(32) NOT NULL,
  from_status VARCHAR(32) NULL,
  to_status VARCHAR(32) NULL,
  actor_id BIGINT NOT NULL,
  actor_name VARCHAR(128) NULL,
  actor_role VARCHAR(64) NULL,
  comment VARCHAR(500) NULL,
  acted_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_approval_record_change_request FOREIGN KEY (change_request_id) REFERENCES change_request(id),
  KEY idx_approval_record_request_acted (change_request_id, acted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS audit_log (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  actor_id BIGINT NULL,
  actor_name VARCHAR(128) NULL,
  actor_org_id BIGINT NULL,
  action_type VARCHAR(64) NOT NULL,
  target_type VARCHAR(64) NULL,
  target_id BIGINT NULL,
  request_no VARCHAR(64) NULL,
  ip_address VARCHAR(64) NULL,
  user_agent VARCHAR(512) NULL,
  result VARCHAR(32) NOT NULL,
  detail_json JSON NULL,
  occurred_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  KEY idx_audit_log_actor_occurred (actor_id, occurred_at),
  KEY idx_audit_log_target_occurred (target_type, target_id, occurred_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

- [ ] **Step 2: Run the migration**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/migrations/0003_create_approval_and_audit_tables.sql
```

Expected:

```text
Query OK
```

- [ ] **Step 3: Re-run the smoke test and verify it now passes**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/tests/001_hr_core_table_count.sql
```

Expected:

```text
PASS
```

- [ ] **Step 4: Verify the key JSON and workflow columns**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot -e "DESCRIBE hrsystem_dev.change_request; DESCRIBE hrsystem_dev.approval_record; DESCRIBE hrsystem_dev.audit_log;"
```

Expected:

```text
before_snapshot json YES
after_snapshot json NO
action varchar(32) NO
detail_json json YES
```

- [ ] **Step 5: Commit version `v0.4.0`**

```bash
git add db/migrations/0003_create_approval_and_audit_tables.sql
git commit -m "feat: add v0.4.0 change request approval and audit schema"
git push
```

### Task 5: Add Index Review Notes And Final Manual Verification Guide

**Files:**
- Create: `db/migrations/0004_schema_review_notes.sql`
- Create: `docs/verification/hr-core-schema-manual-checks.md`
- Test: `db/tests/001_hr_core_table_count.sql`

- [ ] **Step 1: Write a no-op review migration that captures the exact index inspection query set**

```sql
SELECT table_name, index_name, column_name, seq_in_index
FROM information_schema.statistics
WHERE table_schema = 'hrsystem_dev'
  AND table_name IN (
    'organization',
    'person',
    'education_record',
    'certificate_record',
    'attachment',
    'change_request',
    'approval_record',
    'audit_log'
  )
ORDER BY table_name, index_name, seq_in_index;
```

- [ ] **Step 2: Write the manual verification note for this version**

~~~md
# HR Core Schema Manual Checks

## Commands

Start database:

```bash
docker compose up -d mysql
```

Apply migrations:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/migrations/0001_create_organization_and_person.sql
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/migrations/0002_create_person_child_records.sql
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/migrations/0003_create_approval_and_audit_tables.sql
```

Run smoke test:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/tests/001_hr_core_table_count.sql
```

## Expected result

- schema smoke test returns `PASS`
- `organization`, `person`, `education_record`, `certificate_record`, `attachment`, `change_request`, `approval_record`, and `audit_log` all exist
- `change_request` contains `before_snapshot`, `after_snapshot`, and `status`
- `person` contains `employment_status`, `departure_date`, and masked plus encrypted ID fields
- child tables keep both `person_id` and `org_id`
- no summary table is created as editable operational data

## Manual verification note template

`Manual verification performed: checked schema contract and core table coverage, using local MySQL container and smoke-test SQL, result pass, remaining risk application-layer approval transitions not implemented yet.`
~~~

- [ ] **Step 3: Run the review query and smoke test together**

Run:

```bash
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/migrations/0004_schema_review_notes.sql
docker compose exec -T mysql mysql -uroot -proot hrsystem_dev < db/tests/001_hr_core_table_count.sql
```

Expected:

```text
PASS
```

- [ ] **Step 4: Confirm the repository status is clean except for the new files**

Run:

```bash
git status --short
```

Expected:

```text
A  db/migrations/0004_schema_review_notes.sql
A  docs/verification/hr-core-schema-manual-checks.md
```

- [ ] **Step 5: Commit version `v0.5.0`**

```bash
git add db/migrations/0004_schema_review_notes.sql docs/verification/hr-core-schema-manual-checks.md
git commit -m "docs: add v0.5.0 schema verification guide"
git push
```

## Self-Review

- Spec coverage:
  - official master data layer is implemented by `organization`, `person`, `education_record`, `certificate_record`, and `attachment`
  - unified approval and audit layer is implemented by `change_request`, `approval_record`, and `audit_log`
  - organization scope anchoring is implemented by `org_id` on all business tables
  - approval-before-effective design is preserved by creating workflow tables before any application write path
  - Excel summary non-editability is preserved because no summary workbook table is created

- Placeholder scan:
  - no `TODO`
  - no `TBD`
  - no “implement later”
  - no unresolved file paths

  - no `implement later`
  - no unresolved file paths

- Type consistency:
  - all workflow status and target fields use the same names as the approved spec
  - all table names match the approved data foundation design
