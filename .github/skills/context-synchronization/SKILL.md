---
name: context-synchronization
description: "Synchronize shared project context across multiple agents by updating PROJECT_STATE.md after each task and adapting actions based on other agents' changes. Use when running parallel agents, handoffs, or multi-step workflows that require consistent state awareness."
argument-hint: "Task scope, changed files, handoff notes, current PROJECT_STATE.md, and decision constraints"
user-invocable: true
---

# Context Synchronization (Dong bo hoa ngu canh)

## What This Skill Produces
- A continuously updated PROJECT_STATE.md after each completed task.
- A clear delta summary of what changed, why, and what is pending.
- Conflict-aware action adjustments based on updates from other agents.
- A reliable handoff trail that reduces duplicated or contradictory work.

## Why This Skill Matters
This is the core collaboration skill for multi-agent execution.
Without synchronized context, agents can overwrite each other, miss dependencies, or repeat work.

## When To Use
- Multiple agents work on the same feature stream.
- Tasks are handed off across roles (BA, Dev, QA, Reviewer).
- Implementation spans several files/modules with dependency ordering.
- Decisions or assumptions can change during execution.

## Required Inputs
- Current PROJECT_STATE.md content (or permission to create it if missing).
- Current task goal and scope.
- Changed files and key decisions from this task.
- Recent updates from other agents (messages, diffs, summaries, or notes).

If PROJECT_STATE.md is missing, create it using the template in this skill.
If recent updates are incomplete, state assumptions and continue with best-effort sync.

## Standard Workflow

### Step 1: Pre-Task Context Pull
1. Read PROJECT_STATE.md and extract:
- active goals
- in-progress tasks
- blockers
- owner assignments
- open decisions
2. Read latest artifacts from other agents (diffs/summaries/handoffs).
3. Build a "Current Context Snapshot" before starting changes.

Quality check:
- You can explain current priorities in 3-5 lines.
- You can identify at least one dependency or risk from shared context.

### Step 2: Execute Task With Live Awareness
1. Perform the assigned task.
2. Compare new findings with existing PROJECT_STATE.md assumptions.
3. If mismatch appears, mark a context divergence note immediately.
4. Track decisions that affect downstream tasks.

Decision branch:
- If divergence is minor and local: continue, then document under Updates.
- If divergence changes API/schema/business behavior: pause and escalate as blocker.

### Step 3: Post-Task State Update (Mandatory)
After each completed task, update PROJECT_STATE.md with:
- Task completed
- Files touched
- Decisions made
- Risks introduced/removed
- New blockers
- Next recommended actions

This step is mandatory for every task, including small tasks.

### Step 4: Cross-Agent Reconciliation
1. Re-read latest changes from other agents.
2. Detect overlaps/conflicts with your own work:
- same file edited differently
- contradictory assumptions
- duplicated tasks
- sequencing mismatch
3. Adjust your next action plan to align with newest shared state.

Decision branch:
- If conflict is resolvable by order/ownership: update plan and continue.
- If conflict affects correctness or business intent: create explicit conflict note and stop for alignment.

### Step 5: Handoff Readiness Check
Before ending your turn:
1. Verify PROJECT_STATE.md reflects latest truth.
2. Ensure a next-owner can continue without re-discovery.
3. Add concise handoff note with immediate next step.

## PROJECT_STATE.md Template
Use this structure:

1. Current Objective
2. Active Tasks
3. Completed Tasks
4. Recent Decisions
5. Risks and Blockers
6. Agent Handoffs
7. Next Actions
8. Last Updated

Each update should append or modify only the relevant sections.
Do not rewrite unrelated sections.

## Output Contract
When this skill is invoked, return:

1. Context Snapshot
2. Detected Divergences or Conflicts
3. PROJECT_STATE.md Update Summary
4. Action Adjustments From Other-Agent Changes
5. Next-Step Recommendation

## Completion Criteria
- PROJECT_STATE.md updated after task completion.
- Cross-agent updates were read and incorporated.
- Conflicts are either resolved or clearly escalated.
- Next actions and owner handoff are explicit.
- No silent assumption changes remain undocumented.

## Guardrails
- Never claim synchronization without reading latest shared state.
- Never overwrite another agent's decision without recording rationale.
- Prefer minimal, traceable updates over full-file rewrites.
- If evidence is missing, mark uncertainty explicitly.

## Example Invocations
- /context-synchronization Update PROJECT_STATE.md after finishing auth module refactor and align with QA notes.
- /context-synchronization Reconcile my checkout changes with another agent's schema update and adjust next actions.
- /context-synchronization Prepare clean handoff summary for next agent after completing BA-to-code analysis.
