# HR Repository Agent Guide

This repository is for a group-level HR ledger system that replaces Excel-led personnel management with an intranet web application.

Read these files before making changes:

- `docs/superpowers/specs/2026-05-27-hr-system-design.md`
- `docs/ai-constraints.md`
- the HR requirement draft `.docx` in the repository root
- the personnel ledger workbook in the repository root
- the education and degree workbook in the repository root
- the summary workbook in the repository root

## Product Scope

- Product type: group-company personnel ledger system
- Org model: headquarters plus subsidiary HR collaboration
- Current phase: ledger-first MVP, closer to "Excel online" than full HCM
- Core business rule: subsidiary changes do not become effective until group HR approves

## Source of Truth

- The database is the only source of truth for live business data.
- Existing Excel files are import sources and export contracts, not the operational system of record.
- Summary sheets must be generated from detailed records. Never make summary sheets manually editable.

## Non-Negotiable Business Rules

- All data-changing operations require an approval flow unless the feature is explicitly documented as read-only.
- Organization scope must be enforced on every query, mutation, import, export, and attachment access path.
- Personnel records, education records, certificate records, attachments, approvals, and audit trails must be traceable.
- Destructive operations default to soft delete or status transition, not hard delete.

## Excel Contract Rules

- Treat the existing Excel workbooks as external contracts.
- Do not change sheet names, header depth, column order, merge structure, or hyperlink behavior without an explicit approved task.
- Export compatibility is a first-class requirement, not a nice-to-have.
- Before changing any import/export logic, inspect the actual workbook samples in this repository.

## AI Working Rules

- Start by reading the relevant spec and sample workbook. Do not guess field meaning from memory.
- Stay within the requested scope. Do not perform unrelated refactors.
- Do not invent fields, approval states, or statistics definitions unless the spec requires them.
- Do not use real employee data in tests, fixtures, screenshots, or generated examples.
- Mask or omit sensitive personal data such as ID numbers, birth dates, archive locations, and certificate files in examples.

## Expected Verification

Before claiming completion on any data or workflow change, verify at least the relevant subset of:

- import validation
- export round-trip compatibility
- summary generation correctness
- approval gating behavior
- organization-scope permission checks
- audit trail persistence

## Project Documentation

- Repository-wide AI rules live in `AGENTS.md`.
- Detailed constraints live in `docs/ai-constraints.md`.
- Product and architecture intent lives in `docs/superpowers/specs/`.

If repository rules and task instructions conflict, follow the direct task instruction and document the tradeoff in your summary.
