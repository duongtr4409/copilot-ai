---
name: api-contract-first-development
description: "Define Swagger/OpenAPI contracts before implementation, generate Mock APIs for frontend parallel work, and enforce backend-frontend contract alignment. Use for API design, parallel delivery, and integration risk reduction."
argument-hint: "Business domain, required endpoints, auth rules, data models, and timeline"
user-invocable: true
---

# API Contract-First Development (Phat trien theo Hop dong API)

## What This Skill Produces
- A validated OpenAPI contract before business logic code starts.
- Mock APIs generated from the contract so Frontend can integrate early.
- Shared request/response schemas used by both Backend and Frontend.
- Contract change decisions with clear versioning and compatibility impact.
- Quality gates that confirm Backend and Frontend stay aligned.

## When To Use
- New API feature development where FE and BE run in parallel.
- Large integrations where early contract validation reduces rework.
- Teams that need strict API consistency and predictable releases.
- Projects that require consumer-first API design and mock-driven UI delivery.

## Required Inputs
- Business use cases and user flows.
- Endpoint list with methods and ownership.
- Request/response payload examples.
- Authentication and authorization rules.
- Error model, pagination, filtering, and idempotency requirements.
- Non-functional constraints: latency, rate limits, and versioning policy.

## Standard Workflow
1. Define API Scope First
   - Capture user journeys and map them to endpoint candidates.
   - Freeze initial scope for this iteration (must-have vs later).

2. Write OpenAPI Before Logic
   - Create or update OpenAPI 3.x file with paths, schemas, and examples.
   - Include status codes, error responses, and security schemes.
   - Ensure naming consistency for fields and enums.

3. Validate Contract Quality
   - Run OpenAPI lint/validation.
   - Resolve unresolved references, ambiguous schema fields, and missing examples.
   - Reject implementation start if contract is invalid.

4. Generate Mock API
   - Generate a mock server directly from OpenAPI.
   - Provide a stable base URL and dataset/examples for FE testing.
   - Confirm all documented endpoints return mock responses.

5. Start Parallel Delivery
   - Frontend integrates against Mock API immediately.
   - Backend implements handlers against the exact contract.
   - Avoid undocumented fields or response shape drift.

6. Run Contract Conformance Checks
   - Verify Backend responses against OpenAPI schemas.
   - Run FE integration tests against mock first, then real backend.
   - Track and fix contract drift before merge.

7. Release Readiness
   - Confirm mock and real API behavior match accepted contract.
   - Publish changelog for any contract updates.
   - Mark contract version and rollout notes.

## Decision Logic And Branching
1. If contract is incomplete or invalid:
   - Stop implementation.
   - Fix contract first, then regenerate mocks.

2. If Frontend is blocked by missing behavior:
   - Add explicit examples/defaults to OpenAPI.
   - Regenerate mocks and continue FE work.

3. If Backend needs response shape changes:
   - Update OpenAPI first.
   - Re-generate mock/client artifacts.
   - Re-run FE checks before merge.

4. If change is backward-compatible:
   - Add fields as optional or additive updates.
   - Keep current major version, update changelog.

5. If change is breaking:
   - Create new API version path or major version.
   - Require a defined deprecation window for the old contract.
   - Provide migration notes and deprecation plan.

## Mandatory Quality Gates
- OpenAPI passes lint and schema validation with zero critical errors.
- Every endpoint has request/response examples and error responses.
- Mock API generated from latest contract and reachable by FE.
- Backend contract tests pass against OpenAPI schemas.
- FE integration scenarios pass against both mock and real backend.
- No undocumented fields in production responses.

## Output Format
Return results with this structure:

1. API Scope Summary
2. OpenAPI Contract Status
3. Mock API Availability
4. FE/BE Parallel Progress
5. Contract Drift Findings
6. Versioning Decisions
7. Merge/Release Recommendation

## Completion Checklist
- Contract written before business logic coding.
- Mock API available before or at FE implementation start.
- FE and BE both reference the same OpenAPI source.
- Contract conformance checks executed in CI or pre-merge workflow.
- Versioning and changelog updates completed for all contract changes.
- Breaking changes include version bump and deprecation window details.

## Example Skill Prompts
- /api-contract-first-development Define OpenAPI for checkout flow and generate mock endpoints for frontend integration.
- /api-contract-first-development Review this pull request for contract drift between backend responses and OpenAPI.
- /api-contract-first-development Plan a safe breaking change for order status API with versioning and migration notes.