---
name: product-value-analysis
description: "Prioritize product features using Cost-Benefit Analysis (user impact vs technical complexity), define KPI for each feature, and warn when BA requirements are high complexity but low value. Use for backlog prioritization, roadmap trade-offs, and scope negotiation."
argument-hint: "Feature list, target users, business goals, technical constraints, and release timeline"
user-invocable: true
---

# Product Value Analysis (Phan tich Gia tri San pham)

## What This Skill Produces
- A ranked feature list based on value versus implementation cost.
- KPI definitions for every feature with measurable targets.
- Warning flags for BA requests that are technically complex but deliver low user value.
- Action recommendations: build now, simplify, phase, defer, or reject.

## When To Use
- Backlog grooming and release planning.
- Comparing competing feature requests from BA, PM, and stakeholders.
- Scope negotiation when engineering capacity is limited.
- Identifying features that look attractive but have weak product impact.

## Required Inputs
- Feature name and brief description.
- Target user segment and pain point.
- Expected user impact and business impact.
- Engineering complexity estimate (effort, risk, dependencies).
- Constraints: deadline, team capacity, compliance or platform limitations.

## Evaluation Model
Use a 1-5 scale for both Value and Complexity.

### Value Score (1-5)
Score each feature across these dimensions, then average:
- User Impact: How strongly the feature solves a real user pain.
- Reach: How many users are affected.
- Strategic Fit: Alignment with product goals and roadmap.
- Revenue or Retention Potential: Expected contribution to business outcomes.

### Complexity Score (1-5)
Score each feature across these dimensions, then average:
- Engineering Effort: Build time and implementation size.
- Technical Risk: Unknowns, architecture impact, failure risk.
- Dependency Weight: External systems, cross-team blockers, integration depth.
- Operational Cost: Maintenance, observability, support overhead.

### Priority Index
Use this formula:

`Priority Index = Value Score / Complexity Score`

Higher is better.

## KPI Definition Rules (Mandatory)
For each feature, define at least one primary KPI and optional guardrail KPI.

Each KPI must include:
- Metric name.
- Formula or counting rule.
- Current baseline.
- Target value.
- Time window.
- Data source.

If KPI details are missing, mark feature as `Needs KPI Clarification` and do not finalize priority.

## Decision Logic And Branching
1. If value is high (>=4) and complexity is low (<=2):
   - Recommend `Build Now`.
2. If value is high (>=4) and complexity is high (>=4):
   - Recommend `Phase Delivery` or run technical spike for uncertainty reduction.
3. If value is medium (>=3 and <4) and complexity is low-to-medium (<=3):
   - Recommend `Build if Capacity Allows`.
4. If value is low (<=2) and complexity is low (<=2):
   - Recommend `Defer` unless needed for strategic dependency.
5. If value is low (<=2) and complexity is high (>=4):
   - Trigger `BA Complexity-Value Warning` and recommend `Reject or Redesign`.

## BA Complexity-Value Warning Policy
Always warn when a BA requirement is expensive but low impact.

Trigger conditions:
- Value Score <=2.5 AND Complexity Score >=3.5, or
- Priority Index <0.8 and no strategic/compliance justification.

Warning message template:
- `Warning: BA requirement <name> appears high complexity but low user value. Recommend scope reduction, hypothesis validation, or deprioritization.`

## Output Format
Return results using this structure:

1. Analysis Scope
2. Scoring Assumptions
3. Feature Scoring Table
4. KPI Definitions Per Feature
5. BA Complexity-Value Warnings
6. Prioritized Recommendations
7. Risks, Dependencies, and Open Questions

Use this row format in Feature Scoring Table:
- Feature: <name>
- Value Score: <1-5>
- Complexity Score: <1-5>
- Priority Index: <value/complexity>
- Primary KPI: <metric + target + timeline>
- Decision: Build Now | Phase Delivery | Build if Capacity Allows | Defer | Reject/Redesign
- Rationale: <short reason>

## Quality Checks Before Finalizing
- Every feature has both Value and Complexity scores with explicit rationale.
- Every feature has at least one KPI with baseline, target, and time window.
- All low-value/high-complexity BA items are explicitly flagged.
- Final recommendation list is sorted by Priority Index and strategic constraints.
- Any missing data is surfaced as open questions, not hidden assumptions.

## Example Skill Prompts
- `/product-value-analysis Evaluate these 8 backlog features for Q3 release and define KPIs.`
- `/product-value-analysis Compare BA requests for onboarding revamp, referral flow, and analytics dashboard.`
- `/product-value-analysis Flag low-value high-complexity requirements and propose simpler alternatives.`
