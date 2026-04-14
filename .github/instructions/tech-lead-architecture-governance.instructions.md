---
description: "Use when Tech Lead creates architecture blueprints, validates implementation compliance, governs API/DB contract changes, produces ADRs, and enforces hybrid blueprint gates across the delivery pipeline."
name: "Tech Lead Architecture Governance"
---

# Tech Lead Architecture Governance Instruction

## Table of Contents
1. Architecture Decision Record (ADR) Process
2. Blueprint Creation Workflow
3. API Contract Governance
4. Database Schema Review Protocol
5. Blueprint Enforcement Gate
6. Developer Handoff Checklist
7. Architecture Change Request Protocol
8. NFR Validation Matrix
9. Output Quality Gate

## 1. Architecture Decision Record (ADR) Process
- Every significant architecture decision MUST produce an ADR.
- Save ADRs to `/documents/architecture/ADR-[number]-[short-title].md`.

### ADR Template
```markdown
# ADR-[number]: [Title]

## Status
Proposed | Accepted | Deprecated | Superseded by ADR-[number]

## Context
- Business driver and technical trigger for this decision.

## Decision
- What was decided and the chosen approach.

## Options Considered
| Option | Pros | Cons |
|---|---|---|
| Option A | ... | ... |
| Option B | ... | ... |

## Consequences
- Positive: ...
- Negative: ...
- Risks: ...

## Traceability
- BA Requirement Refs: US-xxx, AC-xxx
- Blueprint Section: ...
- Date: YYYY-MM-DDTHH:MM:SS+07:00
```

### ADR Rules
- ADR must be created BEFORE implementation starts.
- ADR must reference BA requirement IDs for traceability.
- ADR status must be updated when decisions change.
- Deprecated ADR must link to its replacement.

## 2. Blueprint Creation Workflow
After receiving BA/PA specs, follow this sequence:

1. **Intake**: Read PRD, SRS, User Stories, and acceptance criteria from `/documents/`.
2. **Constraint Analysis**: Identify NFRs, scale targets, security rules, and tech constraints.
3. **Architecture Reasoning**: Apply `/system-design-reasoning` skill to decompose into modules/services.
4. **Blueprint Draft**: Produce architecture document with:
   - System context diagram (Mermaid `flowchart`)
   - Container/component diagram (Mermaid `flowchart` or `C4`)
   - Critical sequence diagram (Mermaid `sequenceDiagram`)
   - Module boundary definitions with ownership
   - Data flow mapping (FE → BE → DB)
5. **API Contract Design**: Define OpenAPI 3.1 specification using `/api-contract-first-development` skill.
6. **DB Schema Blueprint**: Define entity relationships, constraints, and index strategy.
7. **Review Gate**: Validate completeness against NFR matrix before handoff.
8. **Save**: Store blueprint to `/documents/architecture/ARCH-[Scope].md`.

## 3. API Contract Governance
- All API contracts MUST be defined in OpenAPI 3.1 YAML format.
- Contract MUST be created BEFORE backend implementation starts.
- Contract changes after initial approval require Architecture Change Request.

### API Design Checklist
- [ ] Resource naming follows RESTful conventions (plural nouns, kebab-case).
- [ ] All endpoints have request/response schemas with examples.
- [ ] Error responses follow standard error model (`code`, `message`, `details`).
- [ ] Authentication/authorization schemes are specified.
- [ ] Pagination, filtering, and sorting patterns are standardized.
- [ ] Versioning strategy is defined (URI path or header).
- [ ] Rate limiting and idempotency requirements are documented.
- [ ] CORS policy is specified.

### Breaking Change Rules
- Breaking changes MUST bump API major version.
- Breaking changes MUST include deprecation window (minimum 1 sprint).
- Breaking changes MUST include migration guide.
- Non-breaking changes: add optional fields, add new endpoints, add new enum values.

## 4. Database Schema Review Protocol
Before any DB implementation:

1. **Verify Entity Alignment**: Every entity must trace to BA business rules.
2. **Normalization Check**: Validate 3NF compliance (allow controlled denormalization with documented rationale).
3. **Constraint Completeness**:
   - NOT NULL for required business fields.
   - UNIQUE for business-unique identifiers.
   - FK with ON DELETE/UPDATE policies.
   - CHECK constraints for business rules where applicable.
4. **Index Strategy Review**:
   - Primary key indexes (automatic).
   - Foreign key indexes (mandatory for join performance).
   - Query-driven indexes (based on critical query patterns).
   - Composite indexes (ordered by selectivity).
5. **Migration Safety**:
   - Backward-compatible changes preferred (expand-then-contract).
   - Data type widening allowed; narrowing requires migration plan.
   - Column drops require deprecation window.

## 5. Blueprint Enforcement Gate

### Hard Gate (Blocking — must pass before merge)
| Check | Criteria | Enforcer |
|---|---|---|
| API Contract Match | Backend response schemas match OpenAPI definition | @code-reviewer |
| DB Schema Match | Entity/migration matches blueprint ERD | @sql-database-specialist |
| Security Controls | Auth/AuthZ matches blueprint security spec | @security-auditor |
| Transaction Boundaries | Critical operations follow blueprint transaction rules | @code-reviewer |

### Soft Gate (Advisory — warning with follow-up)
| Check | Criteria | Enforcer |
|---|---|---|
| Module Boundaries | Service/component placement follows architecture layers | @code-reviewer |
| Naming Conventions | Class/method/package names follow blueprint conventions | @code-reviewer |
| Error Handling | Error codes and messages follow blueprint error model | @code-reviewer |
| Performance Patterns | N+1, pagination, caching follow blueprint guidance | @code-reviewer |

### Escalation Rule
- Unresolved hard-gate mismatch blocks release candidate.
- Soft-gate violations accumulate; 5+ unresolved = escalation to Tech Lead review.

### Exception Process
- Exception requires Tech Lead + PMO approval.
- Exception must include: reason, risk assessment, expiry date, remediation plan.
- Exceptions are tracked in `/PROJECT_STATE.md`.

## 6. Developer Handoff Checklist
Before handing blueprint to developer agents, verify:

- [ ] Architecture document saved to `/documents/architecture/`.
- [ ] All ADRs written and status is `Accepted`.
- [ ] OpenAPI 3.1 contract complete with examples and error models.
- [ ] DB schema blueprint complete with ERD, constraints, and indexes.
- [ ] Module boundaries and service responsibilities documented.
- [ ] Engineering rules (guardrails) listed with clear do/don't examples.
- [ ] NFR targets documented (latency, throughput, availability).
- [ ] Security rules documented (auth, data protection, input validation).
- [ ] Blueprint enforcement policy communicated (hard vs soft gates).
- [ ] `/PROJECT_STATE.md` updated with handoff context.

## 7. Architecture Change Request Protocol
After blueprint is approved, changes require:

1. **Change Trigger**: Developer/QA/Reviewer detects need for architecture change.
2. **Impact Assessment**: Tech Lead evaluates scope, risk, and downstream effects.
3. **Decision Options**: Present alternatives with trade-offs.
4. **Approval**: PMO approves timeline impact; Tech Lead approves technical approach.
5. **Update Artifacts**:
   - Update architecture document.
   - Update affected ADRs (create new or deprecate old).
   - Update OpenAPI contract if API is affected.
   - Update DB blueprint if schema is affected.
6. **Notify**: Update `/PROJECT_STATE.md` and notify affected agents.

### Change Classification
| Type | Impact | Approval Required |
|---|---|---|
| Cosmetic | Naming, documentation | Tech Lead only |
| Minor | Internal implementation approach | Tech Lead only |
| Significant | API contract, DB schema, module boundary | Tech Lead + PMO |
| Breaking | Cross-system interface change | Tech Lead + PMO + BA confirmation |

## 8. NFR Validation Matrix
Tech Lead MUST define and validate these non-functional requirements:

| NFR Category | Metric | Target | Validation Method |
|---|---|---|---|
| Latency | P95 API response time | ≤ 200ms (simple), ≤ 500ms (complex) | Load test, APM |
| Throughput | Requests per second | Define per endpoint | Load test |
| Availability | Uptime SLA | ≥ 99.9% (production) | Monitoring, health checks |
| Scalability | Horizontal scaling | Define min/max instances | Auto-scaling test |
| Security | OWASP Top 10 coverage | Zero critical findings | Security scan |
| Data Integrity | Transaction success rate | 100% for critical ops | Integration test |
| Observability | Log/metric/trace coverage | All critical paths | Audit review |
| Recovery | RTO/RPO | Define per data tier | DR test |

## 9. Output Quality Gate (Mandatory)
Tech Lead task is NOT complete unless:

- [ ] Architecture document with diagrams saved to `/documents/architecture/`.
- [ ] At least one ADR created for each significant decision.
- [ ] OpenAPI 3.1 contract produced for all API endpoints.
- [ ] DB schema blueprint with ERD, constraints, and indexes.
- [ ] Blueprint enforcement policy defined (hard gate + soft gate).
- [ ] NFR targets documented with validation methods.
- [ ] Developer handoff checklist completed.
- [ ] `/PROJECT_STATE.md` updated with Tech Lead completion status.
- [ ] Inter-agent handoff issued to developer agents with blueprint references.
