---
name: clean-code-patterns-enforcement
description: "Enforce clean, maintainable code by applying SOLID and DRY, selecting context-appropriate design patterns (Factory, Singleton, Observer), and using /refactor with #style-guide. Use for code implementation, refactoring, and maintainability reviews."
argument-hint: "Language, code context, business intent, current code smells, and constraints"
user-invocable: true
---

# Clean Code and Patterns Enforcement (Ap dung Ma sach va Mau thiet ke)

## What This Skill Produces
- Refactoring recommendations that improve maintainability without changing behavior.
- Explicit SOLID and DRY checks mapped to concrete code smells.
- Context-based design pattern selection with clear rationale.
- Structured refactor output using /refactor and #style-guide.

## When To Use
- Writing new features where long-term maintainability matters.
- Refactoring code that works but is hard to read, test, or extend.
- Reviewing pull requests for architecture and clean-code quality.
- Deciding whether to apply Factory, Singleton, Observer, or no pattern.

## Required Inputs
- Target files, symbols, or modules.
- Business behavior that must remain unchanged.
- Current pain points: duplication, tight coupling, large classes, unstable abstractions.
- Constraints: timeline, backward compatibility, performance, team conventions.

## Tools
- #style-guide:
  - Load first to anchor refactor decisions to project conventions.
  - Use again during quality gate to verify naming, structure, and test style alignment.
- /refactor:
  - Use to execute maintainability-focused changes in small reversible steps.
  - Require explicit rationale per change mapped to SOLID, DRY, or pattern decision.

## Workflow
1. Load coding conventions first.
- Use #style-guide to identify naming, formatting, architecture, and test conventions.
- If style guide is unavailable, state assumptions explicitly before refactoring.

2. Identify code smells and maintainability risks.
- Detect duplication, long methods, god classes, high coupling, low cohesion, hidden dependencies.
- Classify each issue under SOLID or DRY.
- Prioritize by risk and impact.

3. Run SOLID and DRY enforcement checks.
- SRP: Split responsibilities when a module changes for multiple reasons.
- OCP: Prefer extension points over repeated modification of stable code.
- LSP: Ensure subtype behavior remains substitutable.
- ISP: Avoid forcing clients to depend on unused interfaces.
- DIP: Depend on abstractions, not concrete implementations.
- DRY: Extract repeated logic only when duplication is semantic, not accidental.

4. Select pattern only if it simplifies maintenance.
- Factory:
  - Use when object creation logic is complex, branching, or environment-dependent.
  - Avoid when direct construction is simple and stable.
- Singleton:
  - Use for truly single-instance cross-cutting services (for example config provider).
  - Avoid mutable global state, hidden dependencies, and hard-to-test access paths.
- Observer:
  - Use for event-driven updates where publishers should not know subscribers.
  - Avoid when event ordering, fan-out, or side effects become hard to reason about.

5. Decide refactor strategy and execute with /refactor.
- Prefer small, reversible steps.
- Preserve public contracts unless change is explicitly approved.
- Add or update focused tests when behavior risk exists.

6. Validate against quality gate.
- Behavior parity maintained (tests pass or equivalent evidence provided).
- Readability and cohesion improved.
- Coupling and duplication reduced.
- Pattern usage has clear benefit and no over-engineering signs.
- Result follows #style-guide conventions.

## Decision Branching
- If duplication appears in 3 or more locations with shared intent:
  - Extract reusable abstraction.
- If abstraction adds indirection without reducing change frequency:
  - Reject abstraction and keep implementation simple.
- If new variants are expected to grow:
  - Prefer Factory or strategy-style extension points.
- If one shared instance is required for correctness:
  - Consider Singleton with explicit lifecycle and test seam.
- If modules communicate via cascading callbacks:
  - Consider Observer with bounded subscribers and event contracts.

## Output Contract
When invoked, return these sections in order:
1. Scope and Behavior Constraints
2. Detected Smells (SOLID/DRY Mapping)
3. Pattern Decision Matrix
4. Refactor Plan (/refactor)
5. Quality Gate Result
6. Risks and Follow-up Actions

## Completion Criteria
- Every proposed change maps to a specific maintainability issue.
- Pattern choice includes why and why-not alternatives.
- No unnecessary complexity is introduced.
- Public behavior remains stable and verifiable.
- Final recommendations are actionable in small implementation steps.

## Example Prompts
- /clean-code-patterns-enforcement Refactor this service to reduce duplication and enforce SOLID.
- /clean-code-patterns-enforcement Choose between Factory and Observer for this notification flow.
- /clean-code-patterns-enforcement Review this module for DRY violations and propose safe refactor steps.
