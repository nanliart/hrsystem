# HR Verification Checklist

## 1. How To Use This Checklist

Use the relevant sections before declaring a task complete.

Not every task needs every check, but any task that touches import/export, approvals, permissions, summaries, attachments, or employee data should explicitly state which checks were run.

## 2. Core Completion Checks

- The change stays within the approved task scope.
- The change does not introduce real personal data into tests, screenshots, fixtures, or docs.
- The change does not bypass approval-before-effective business rules.
- The change does not weaken organization-scope access.

## 3. Import Checklist

- Sample workbook still parses with the expected sheet structure.
- Required columns or header groups still map correctly.
- Import failures produce clear errors for missing or invalid columns.
- Summary sheets are not treated as writable source data.
- Batch import still routes data changes through approval when required.

## 4. Export Checklist

- Export contains the expected sheets.
- Column order remains stable.
- Multi-row header structure remains stable.
- Summary layout still matches the contract.
- Hyperlink behavior is preserved or clearly documented if not yet implemented.
- Round-trip expectations are still satisfied for representative data.

## 5. Summary Checklist

- Hierarchy summary counts still derive from approved detail records.
- Structure summary dimensions still match the intended definitions.
- Derived totals or check columns remain internally consistent.
- Summary output does not leak out-of-scope data to scoped users.

## 6. Approval Checklist

- Draft or submitted changes do not directly alter official records.
- Approved requests update official records correctly.
- Rejected requests do not change official records.
- Approval history remains visible and traceable.
- Self-approval behavior, if allowed at all, is still explicitly controlled.

## 7. Permission Checklist

- In-scope users can complete allowed actions.
- Out-of-scope access is denied for list, detail, export, and attachment routes.
- API and UI both enforce the same effective scope.
- Sensitive fields remain masked or restricted where required.

## 8. Attachment Checklist

- Upload access follows role and scope rules.
- Download access follows role and scope rules.
- Attachment metadata does not leak sensitive paths unnecessarily.
- Audit logging captures material attachment actions when implemented.

## 9. Audit Checklist

- Change requests preserve actor, timestamp, and change type.
- Approval actions preserve approver, timestamp, and decision.
- Import and export actions remain traceable when implemented.
- Security-sensitive denials are captured when feasible.

## 10. Manual Verification Note Template

When automation is not ready, include a note like this in the task summary:

`Manual verification performed: checked [contract area], using [sample/workflow], result [pass/fail], remaining risk [short note].`
