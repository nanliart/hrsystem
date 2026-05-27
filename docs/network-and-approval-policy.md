# Network And Approval Policy

## 1. Purpose

This repository handles sensitive HR data and workbook contracts.

AI agents working on this project should default to the safest posture:

- minimal internet access
- minimal write access
- explicit approval for risky operations

## 2. Default Network Policy

- Default to no internet access while analyzing or editing repository content.
- Prefer local documents, workbook samples, and repository rules first.
- Use the internet only when current external information is required, such as official documentation, security guidance, or package behavior that may have changed.

## 3. Allowed External Sources

Prefer these categories:

- official product documentation
- official language or framework documentation
- official vendor security advisories
- GitHub repository pages for directly relevant dependencies or standards

Avoid casual third-party tutorials when a primary source exists.

## 4. HTTP Method Expectations

- Prefer `GET`, `HEAD`, and `OPTIONS`.
- Do not send repository data, HR data, or workbook contents to external services unless the task explicitly requires it and approval is clear.
- Do not upload employee data, attachment paths, or workbook samples to external tools by default.

## 5. Approval Expectations For Agents

Ask for explicit approval before:

- installing new dependencies
- accessing external private systems
- uploading files outside the repository environment
- running destructive commands
- changing workbook contracts
- weakening permission or approval boundaries

## 6. HR-Specific Safety Rules

- Never share real ID numbers, archive locations, or certificate file paths externally.
- Treat workbook examples as sensitive even if they are already present locally.
- If a task involves logs, screenshots, or demos, mask personal identifiers first.

## 7. When To Escalate Internally

Pause and ask for confirmation when a task would:

- alter the approval-before-effective rule
- broaden organization-scope data access
- change summary calculation rules
- change export fidelity expectations
- expose attachments more broadly than before

## 8. Recommended Review Order

Before using external resources, review:

1. `AGENTS.md`
2. `docs/ai-constraints.md`
3. `docs/data-dictionary.md`
4. `docs/permission-matrix.md`
5. local workbook samples
