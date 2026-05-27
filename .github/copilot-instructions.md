# HR Repository Copilot Instructions

This repository is building an intranet HR ledger system for a parent group and its subsidiaries.

Read these files before proposing or making meaningful changes:

- `AGENTS.md`
- `docs/ai-constraints.md`
- `docs/superpowers/specs/2026-05-27-hr-system-design.md`
- the workbook samples in the repository root

## Project Priorities

- Preserve business correctness over UI convenience.
- Preserve Excel import/export compatibility over internal implementation shortcuts.
- Preserve organization-scope permissions over broad queries or convenience APIs.
- Preserve approval-before-effective writes over direct mutation flows.

## What This MVP Is

- Group-company personnel ledger
- Headquarters plus subsidiary HR collaboration
- Ledger-first system, closer to "Excel online" than full HCM
- Single-stage approval for all data-changing actions

## What This MVP Is Not

- Payroll system
- Full recruiting suite
- Complex attendance engine
- Free-form report builder

## Behavior Expectations For Copilot

- Do not guess workbook fields from memory. Inspect workbook samples or the data dictionary first.
- Do not flatten Excel summary sheets into editable business tables.
- Do not bypass approval logic to "simplify" writes.
- Do not expose full personal identifiers in examples, tests, or screenshots.
- Do not propose unrelated refactors when the task is focused on one business contract.

## Documentation To Use

- Use `docs/data-dictionary.md` for canonical field names and sensitivity flags.
- Use `docs/permission-matrix.md` for role and organization boundaries.
- Use `docs/network-and-approval-policy.md` for external access and approval expectations.
- Use `docs/verification-checklist.md` before claiming task completion.

## Delivery Expectations

When changing logic related to import/export, approval, permissions, or attachments:

- explain which business contract is affected
- explain how the change preserves the contract
- mention what verification was run
- call out any remaining contract risk clearly
