---
applyTo: "**/*import*.py,**/*export*.py,**/*excel*.py,**/*import*.ts,**/*export*.ts,**/*excel*.ts,**/*import*.tsx,**/*export*.tsx,**/*excel*.tsx,**/*report*.py,**/*report*.ts,**/*report*.tsx"
---

# Excel Contract Instructions

- Treat workbook structure as an external compatibility contract.
- Before editing these files, read `docs/ai-constraints.md`, `docs/data-dictionary.md`, and the workbook samples in the repository root.
- Do not change sheet names, header depth, column order, merge behavior, or summary layout without an explicit approved task.
- Summary sheets are generated outputs. Do not treat them as manually editable source tables.
- Import code should read detail data and validate headers clearly. It should not trust workbook summaries as the system of record.
- Export code should preserve the workbook contract as closely as possible, including hyperlink behavior when implemented.
- When a field is ambiguous, prefer the existing workbook label and document the mapping.
- Every change in this area should verify import parsing, export shape, and summary correctness.
