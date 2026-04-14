---
name: automated-knowledge-synthesis
description: "Transform code changes into human-readable documentation by updating README.md, API_DOCS.md, and CHANGELOG.md after each merge, and explaining complex technical decisions in plain language. Use for continuous documentation quality and team knowledge sharing."
argument-hint: "Merged PR diff, target audience, API impact, and release context"
user-invocable: true
---

# Automated Knowledge Synthesis (Tong hop tri thuc tu dong)

## What This Skill Produces
- Human-readable documentation synchronized with merged code.
- Updated README.md, API_DOCS.md, and CHANGELOG.md after each merge.
- Plain-language explanations for complex technical decisions.
- Clear change impact communication for engineering and non-engineering readers.

## Khi nao dung (When To Use)
- Sau moi lan merge vao nhanh chinh.
- When features, APIs, configuration, or architecture behavior changes.
- Khi can giai thich quyet dinh ky thuat cho PM, QA, hoac stakeholder.
- When release notes and docs quality must stay continuously up to date.

## Required Inputs
- Merged PR or commit diff.
- Current versions of README.md, API_DOCS.md, and CHANGELOG.md.
- Audience profile (developer, QA, PM, stakeholder).
- Release type (patch, minor, major) and compatibility notes.

## Standard Workflow
1. Extract Merge Context
   - Read merged changes and identify user-visible impact.
   - Classify impact: feature, API, config, behavior, performance, or security.

2. Map Changes To Documentation Targets
   - README.md: setup, usage, architecture overview, workflows.
   - API_DOCS.md: endpoint contracts, schemas, auth, errors, examples.
   - CHANGELOG.md: added, changed, fixed, removed, deprecated, security.

3. Update README.md
   - Add or revise practical usage and integration examples.
   - Keep explanations concise and action-oriented.

4. Update API_DOCS.md
   - Reflect actual API behavior and schema changes from merged code.
   - Include request/response examples and error behavior.
   - Add migration notes for deprecated or breaking API changes.

5. Update CHANGELOG.md
   - Add entries under the correct release heading.
   - Explain what changed and why it matters in plain language.
   - Highlight breaking changes and remediation steps.

6. Explain Complex Technical Decisions
   - For each important decision, document:
     1) Decision: what was chosen.
     2) Why: constraints and trade-offs.
     3) Effect: expected impact on maintainability, performance, or users.
   - Replace heavy jargon with simple language and concrete examples.

7. Validate Documentation Consistency
   - Ensure all 3 documents match merged behavior.
   - Remove contradictory or stale statements.
   - Verify terminology consistency across files.

## Decision Logic And Branching
1. If merge is refactor-only with no behavior/API change:
   - Keep README.md and API_DOCS.md unchanged unless clarity improves.
   - Add minimal CHANGELOG.md note only if release process requires it.

2. If API changes are detected:
   - Update API_DOCS.md first.
   - Then align README.md examples and CHANGELOG.md impact summary.

3. If breaking change exists:
   - Mark clearly in CHANGELOG.md.
   - Add migration steps in README.md and API_DOCS.md.

4. If decision rationale is too technical for broad audience:
   - Add a plain-language summary first.
   - Keep deeper technical detail in a separate paragraph.

5. If multiple merges overlap in the same release window:
   - Merge notes into one coherent CHANGELOG.md section.
   - Avoid duplicate or conflicting statements.

## Mandatory Quality Gates
- README.md is aligned with current project behavior and usage.
- API_DOCS.md matches implemented endpoints and payload contracts.
- CHANGELOG.md includes user-meaningful entries for merged changes.
- Every complex decision includes Decision, Why, and Effect.
- Language is understandable for mixed audiences.
- No contradictions between README.md, API_DOCS.md, and CHANGELOG.md.

## Output Format
Return results using this structure:

1. Merge Impact Summary
2. README.md Updates
3. API_DOCS.md Updates
4. CHANGELOG.md Entry Draft
5. Complex Decision Explanations
6. Consistency Check
7. Documentation Readiness Decision

## Completion Checklist
- README.md updated or explicitly justified as unchanged.
- API_DOCS.md updated for all API-impacting merges.
- CHANGELOG.md updated for release traceability.
- Complex decisions rewritten in plain language.
- Breaking changes include migration guidance.
- Final consistency pass completed across all documentation files.

## Example Skill Prompts
- /automated-knowledge-synthesis Convert this merged PR into README.md, API_DOCS.md, and CHANGELOG.md updates.
- /automated-knowledge-synthesis Explain architecture trade-offs from this merge in plain language.
- /automated-knowledge-synthesis Generate release-ready docs with consistency checks after today merges.