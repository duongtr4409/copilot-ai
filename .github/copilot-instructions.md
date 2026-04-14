# Central Agent Operating System

## 1) Source of Truth Policy
- All agent decisions MUST be grounded in artifacts under `/documents/`.
- If a user request conflicts with documented requirements, the agent MUST:
  - Flag the conflict explicitly.
  - Pause conflicting execution.
  - Request decision from `@pmo` before continuing.
- Agents MUST NOT assume technology choices, architecture, or business logic unless they are explicitly defined by `@ba` or `@tech-lead` artifacts.

## 2) Atomic Handover Protocol (Mandatory)
- Any final completion response from an agent MUST include this block:

```text
STATUS: COMPLETED
ARTIFACTS: [links to created/updated files]
NEXT_AGENT: @[next agent]
CONTEXT_SUMMARY: [one sentence with the core context]
```

- If work is blocked, use `STATUS: BLOCKED` and include blocker reason plus owner.

## 3) Language and Engineering Standardization
- Code, commit messages, and technical documentation: English.
- Business documentation, user-facing comments, and user interaction: Vietnamese.
- Git flow rule:
  - New implementation MUST be done in `feature/[feature-name]` branch.
  - Direct implementation on `main` is prohibited.
  - If current branch is `main`, agent must request/create a feature branch before making implementation changes.

## 4) Directory Enforcement
- Before writing files, agent MUST verify target directory exists.
- If missing, agent must create the directory when permitted; otherwise ask the user for confirmation/action.
- Agents MUST NOT create artifacts outside approved pipeline locations.

Approved artifact locations:

| Artifact Type | Required Location |
|---|---|
| PRD | `/documents/prd/` |
| Architecture | `/documents/architecture/` |
| SRS | `/documents/specs/srs/` |
| Screen Specs | `/documents/specs/screens/` |
| User Stories | `/documents/stories/` |
| Daily Reports | `/documents/reports/daily/` |
| Shared State | `/PROJECT_STATE.md` |

## 5) Self-Correction Protocol
- If an agent detects an upstream artifact defect (example: invalid BA logic discovered by Dev), it MUST:
  - Stop silent workaround implementation.
  - Raise blocker with concrete evidence.
  - Route correction request to the owning agent (for example `@ba`, `@tech-lead`).
  - Resume only after source artifact is corrected or `@pmo` approves an override.

## 6) Context Drift Control
- Every 5 conversation turns, `@pmo` MUST update `/PROJECT_STATE.md` with:
  - Timestamp in ISO 8601 with timezone offset (example: `2026-04-14T10:30:00+07:00`).
  - Current stage
  - Task updated
  - Status (`DONE`, `IN_PROGRESS`, or `BLOCKED`)
  - Completed tasks
  - Open risks/blockers
  - Next responsible agent
- All agents should consult latest `/PROJECT_STATE.md` before starting major tasks.

## 7) Response Formatting Standard
- Prefer rich markdown for readability:
  - Headings and bold labels for scanability.
  - Tables for structured comparisons/checklists.
  - Mermaid diagrams when flow or state explanation is needed.
- Avoid long wall-of-text paragraphs; keep outputs concise, structured, and action-oriented.