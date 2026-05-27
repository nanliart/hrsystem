# HR AI Constraints

## 1. Purpose

This document defines the working constraints for AI-assisted design and implementation in this repository.

The goal is to keep the project aligned with the actual business contract:

- multi-company group HR collaboration
- personnel ledger as the first MVP
- approval-before-effective mutations
- high-fidelity Excel import and export

## 2. Business Context

The current source materials show three business anchors:

- the personnel ledger workbook: active employees, departed employees, hierarchy summary, structure summary
- the education and degree workbook: education and degree certificates by company sheet
- the summary workbook: certificate and yearly summary data

The Word requirement draft adds the operational rules:

- subsidiary administrators maintain their own data
- group administrators approve changes before they become effective
- summaries are auto-generated from detail data
- exported Excel should remain as close to 1:1 as possible, including hyperlinks

## 3. Hard Product Constraints

### 3.1 Scope Constraints

- Phase 1 is a personnel-ledger MVP, not a full HR suite.
- Prioritize employee master data, organization ownership, history, attachments, summaries, and exports.
- Defer payroll, performance, recruiting, and complex attendance unless a task explicitly expands scope.

### 3.2 Organization Constraints

- The system must support headquarters plus subsidiaries.
- Data permissions must combine role and organization scope.
- Subsidiary HR can maintain subsidiary data only.
- Group HR can review and approve across the full group.

### 3.3 Approval Constraints

- Mutations cannot write directly into the official record by default.
- Changes must first create a request or staging record.
- Only approved changes may update official data.
- Rejected changes must remain visible in history.

### 3.4 Excel Constraints

- Existing workbook structure is a compatibility boundary.
- Import logic must tolerate the current multi-row headers and workbook-specific layout.
- Export logic must preserve:
  - sheet names
  - column order
  - merged headers where required
  - summary layout
  - hyperlink behavior from summary to detail, when implemented
- Summary sheets are generated outputs and must be recalculated from detail data.

### 3.5 Data Constraints

- Database records are normalized domain data.
- Excel files are interoperability formats.
- Personnel, education, certificate, attachment, and approval data should be modeled separately.
- Avoid copying the workbook into a single giant table unless the task is strictly a raw import staging layer.

## 4. AI Coding Constraints

### 4.1 Change Discipline

- Read the relevant workbook and spec before editing import/export code.
- Keep changes narrowly scoped to the requested task.
- Do not rename business concepts casually. Use the terms already established in the workbook or spec.
- Do not introduce new workflow states without documenting them.

### 4.2 Safety and Privacy

- Never place real personal information into tests, seeds, screenshots, or generated docs.
- Use masked examples for names, ID numbers, phone numbers, and archive details.
- Treat attachments and certificate files as permission-controlled resources.
- Avoid unnecessary internet access while handling repository data.

### 4.3 Architecture Boundaries

- Keep import, export, summary generation, and approval logic out of UI-only layers.
- Put workbook compatibility logic into dedicated services or modules.
- Keep summary computation deterministic and reproducible.
- Keep permission checks centralized enough to audit.

## 5. Required Validation

Any meaningful implementation touching ledger data, approvals, or Excel behavior should verify the relevant parts of this list.

### 5.1 Import Validation

- Valid sample workbook imports successfully
- Multi-row headers map correctly
- Invalid or missing required columns fail clearly
- Import does not silently mutate summary outputs as source data

### 5.2 Export Validation

- Exported workbook contains the expected sheets
- Column order and header structure match the contract
- Summary layout remains stable
- Round-trip check passes for representative sample data

### 5.3 Summary Validation

- Hierarchy summary counts are reproducible from detail data
- Personnel structure summary matches defined dimensions
- Derived totals and check columns remain internally consistent

### 5.4 Approval Validation

- Submitted changes remain pending until approved
- Rejected changes do not affect official records
- Approved changes update official records and preserve history

### 5.5 Permission Validation

- Subsidiary HR cannot view or edit out-of-scope records
- Group HR can review across organizations
- Attachment access follows the same organization and role rules

### 5.6 Audit Validation

- Change submitter, approver, timestamps, and change type are recorded
- Export, import, approval, and download actions can be traced when implemented

## 6. Recommended Working Flow For Agents

1. Read task-specific requirements plus the relevant workbook sample.
2. Identify whether the task affects detail data, summaries, approvals, permissions, or export contracts.
3. Change the smallest possible vertical slice.
4. Run focused verification for the impacted contract.
5. Summarize what changed, what was verified, and any remaining contract risk.

## 7. Definition Of "Done"

A task is not done just because the UI looks correct or the code compiles.

For this project, done means:

- business rules still hold
- Excel compatibility is not accidentally broken
- permissions still enforce organization boundaries
- approval gating still protects official records
- sensitive data is not exposed in artifacts

## 8. Future Repository Additions

When the codebase grows, add:

- `.github/copilot-instructions.md` for tool-specific repository guidance
- sample test fixtures with masked data
- golden workbook comparison tests
- permission matrix documentation by role and organization
