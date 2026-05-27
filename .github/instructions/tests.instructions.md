---
applyTo: "tests/**,**/*.test.ts,**/*.test.tsx,**/*.spec.ts,**/*.spec.tsx,**/test_*.py,**/*_test.py"
---

# HR Verification Instructions

- Prefer business-contract tests over implementation-detail tests.
- Add or update tests for the exact contract being changed: import mapping, export shape, summary generation, approval gating, permission scoping, or audit history.
- Use masked or synthetic HR data only.
- For workbook-related behavior, prefer golden samples or round-trip validation.
- For permission behavior, include at least one allowed and one denied case.
- For approval behavior, include pending, approved, and rejected paths where relevant.
- If a contract cannot yet be automatically tested, call out the gap and add a manual verification note.
