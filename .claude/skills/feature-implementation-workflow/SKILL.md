---
name: feature-implementation-workflow
description: Use within the main/orchestrator agent and the user asks to create a new feature or modify an existing feature. Guides subagent research, planning, approval, implementation, human-input relay, and verification before feature code changes.
---

# Feature Implementation Workflow

Use this workflow for feature implementation requests. The goal is to separate research, planning, approval, and implementation so final work is grounded in the codebase, explicitly scoped, and approved before code changes begin.

The main agent is the orchestrator. It owns all user communication, preserves phase outputs, reviews subagent work, and decides whether user feedback returns the task to research, planning, or implementation.

Subagents cannot ask the user directly. If a subagent needs product judgment, scope clarification, or human approval, it must stop and return a human input request containing:

- The unclear point or decision.
- Why it matters.
- Reasonable options, if known.
- The subagent's recommendation, if it has one.

The orchestrator asks the user, then returns the user's answer to the subagent or routes back to the appropriate earlier phase.

## Step 1: Research

Call a subagent to inspect the relevant code, tests, documentation, configuration, and existing patterns. The research subagent must not implement the feature.

The research result should include:

- Relevant files, modules, and ownership boundaries.
- Current behavior and data flow.
- Existing patterns to follow.
- Risks, unknowns, constraints, and likely test surfaces.
- Ambiguities, assumptions, and multiple possible interpretations.
- Simpler alternatives or reasons to push back.
- Decisions needed before implementation.

Preserve the research result as input for planning.

## Step 2: Planning

Call a clean subagent to create the feature plan. Provide the user's request and the research result. The planning subagent must not implement the feature.

The plan should include:

- Success criteria.
- Proposed approach.
- Files expected to change.
- Data model, API, UI, or behavior changes, if applicable.
- Failing test or tests to write before implementation.
- Focused verification commands.
- Rollout or migration notes, if applicable.
- Open questions or assumptions.

Review the plan for clarity, scope, and simplicity before showing it to the user. Send it back for revision if it includes speculative work, broad refactors, unnecessary abstractions, or changes outside the user's request.

## Step 3: User Approval

Present the implementation plan to the user before code changes.

Do not begin implementation until the user approves the plan or gives comments that clearly authorize proceeding with an adjusted plan.

If the user provides comments:

- Return to planning when comments substantially change the approach.
- Return to research when comments reveal missing or uncertain technical context.
- Proceed to implementation when comments only adjust the current plan and clearly authorize the work.

Ask for clarification when approval feedback is ambiguous.

## Step 4: Implementation

Call another subagent to implement the feature. Provide the original request, research result, approved plan, and any user adjustments.

The implementation subagent should:

- Keep changes tightly scoped to the approved goal.
- Write a failing test or failing check before feature code when the codebase supports it.
- Implement the minimum behavior needed to satisfy the success criteria.
- Add or update tests only when they still verify the approved goal.
- Run focused verification.
- Remove only unused code introduced by its own changes.

If the approved plan is impossible, unsafe, substantially more complex than expected, or conflicts with the success criteria, the implementation subagent must stop and return a human input request.

The implementation result should include:

- The success criteria used.
- The failing test or check written before implementation and its initial failure result.
- Summary of changes made.
- Files changed.
- Tests or checks run, including failures.
- Remaining risks, limitations, or follow-up work.

## Orchestrator Finish

Review the implementation result and perform any necessary final verification. If verification fails, continue debugging the implementation rather than weakening tests.

Final responses should clearly summarize what changed, how it was verified, and any unresolved issues.

If subagent tools are unavailable, perform the same phases yourself, preserve the user approval gate before implementation, and mention that limitation to the user.
