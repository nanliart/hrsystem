---
applyTo: "**/*auth*.py,**/*auth*.ts,**/*auth*.tsx,**/*permission*.py,**/*permission*.ts,**/*permission*.tsx,**/*approval*.py,**/*approval*.ts,**/*approval*.tsx,**/*attachment*.py,**/*attachment*.ts,**/*attachment*.tsx,**/*archive*.py,**/*archive*.ts,**/*archive*.tsx"
---

# Security And Privacy Instructions

- This repository handles sensitive HR data. Default to least privilege.
- Enforce permissions as role plus organization scope, not role alone.
- Never assume attachment access is safe just because the user can view basic employee data.
- Approval logic is a control boundary. Do not bypass it for convenience.
- Mask personal identifiers in logs, tests, fixtures, screenshots, and examples.
- Prefer soft delete or status transitions over hard delete for traceable HR records.
- Any endpoint or service touching employee profile, certificates, archive location, or attachments must consider audit logging requirements.
- If a change weakens organization scoping, direct downloads, or approval gating, treat it as a regression.
