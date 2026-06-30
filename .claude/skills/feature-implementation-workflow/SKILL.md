---
name: feature-implementation-workflow
description: Orchestrates feature creation and modification through Claude Code project subagents, with research, planning, user approval, failing tests, implementation, and verification. Use when the user asks to add or change a feature.
---

# Feature Implementation Workflow

Use this workflow for feature implementation requests. Keep research, planning, approval, failing-test authoring, and implementation separate so changes are grounded in the codebase, explicitly scoped, and approved before code changes begin.

## Claude Code Agents

Invoke these project subagents with Claude Code's Agent tool:

- `feature-research`
- `feature-planning`
- `feature-failing-tests`
- `feature-implementation`

Run them sequentially in the foreground. Each phase depends on the previous result, and foreground execution lets the orchestrator relay permission or human-input needs. Do not substitute a generic or differently scoped agent if a named agent is unavailable. Ask the user to restart Claude Code so project agents added under `.claude/agents/` are loaded.

Each subagent starts with an isolated context. Include the complete original request and all required prior phase results in every delegation prompt; do not assume a later agent can see an earlier agent's conversation.

The main agent is the orchestrator. It owns all user communication, preserves phase outputs, reviews agent work, and decides whether feedback returns the task to research, planning, failing-test authoring, or implementation.

The phase agents cannot ask the user directly. If an agent returns a human input request, the orchestrator asks the user and then resumes the agent or routes the task back to the appropriate earlier phase. A human input request must include:

- The unclear point or decision.
- Why it matters.
- Reasonable options, if known.
- The agent's recommendation, if available.

## Step 1: Research

Invoke `feature-research` with the original request and relevant workspace context. Wait for it to inspect the relevant code, tests, documentation, configuration, and existing patterns without making changes.

The research result must include:

- Relevant files, modules, and ownership boundaries.
- Current behavior and data flow.
- Existing patterns to follow.
- Risks, unknowns, constraints, and likely test surfaces.
- Ambiguities, assumptions, and multiple reasonable interpretations.
- Simpler alternatives or reasons to push back.
- Decisions needed before implementation.

Preserve the complete result for planning.

## Step 2: Planning

Invoke `feature-planning` with the original request and complete research result. The planning agent must not edit files or implement the feature.

The plan must include:

- Success criteria.
- Proposed approach.
- Files expected to change.
- Data model, API, UI, or behavior changes, if applicable.
- Failing tests or checks to write before implementation.
- Focused verification commands.
- Rollout or migration notes, if applicable.
- Open questions and assumptions.

Review the plan for clarity, scope, and simplicity. Return it for revision if it includes speculative work, broad refactors, unnecessary abstractions, or unrelated changes.

## Step 3: User Approval

Present the implementation plan to the user before making code changes.

Do not invoke `feature-failing-tests` or `feature-implementation` until the user approves the plan or gives comments that clearly authorize proceeding with an adjusted plan.

If the user provides comments:

- Return to planning when they substantially change the approach.
- Return to research when they reveal missing or uncertain technical context.
- Proceed to failing-test authoring when they only adjust the plan and clearly authorize the work.

Ask for clarification when approval feedback is ambiguous.

## Step 4: Failing Tests

Invoke `feature-failing-tests` with the original request, research result, approved plan, and user adjustments.

The agent must:

- Keep changes limited to tests or test fixtures needed for the approved goal.
- Avoid production implementation changes.
- Run the focused test or check and confirm it fails for the expected reason.
- Return a human input request if the approved test plan is impossible, unsafe, ambiguous, or conflicts with the success criteria.

Preserve the complete failing-test result, including:

- Success criteria covered.
- Files changed.
- Exact command run.
- Initial failure and why it is expected.
- Test-design or implementation-handoff concerns.

## Step 5: Implementation

Invoke `feature-implementation` with the original request, research result, approved plan, user adjustments, and complete failing-test result.

The implementation agent must:

- Make the minimum changes needed to satisfy the approved success criteria.
- Treat tests written by `feature-failing-tests` as owned input and never modify them.
- Run focused verification.
- Report any defect in an owned test instead of changing it.
- Return a human input request if the approved plan becomes impossible, unsafe, substantially more complex than expected, or inconsistent with the success criteria.

If the implementation agent reports a defect in an owned test, route that feedback back to `feature-failing-tests` for correction before final verification.

## Orchestrator Finish

Review the implementation result and run any necessary final verification. Debug implementation failures instead of weakening checks.

The final response must summarize:

- What changed.
- How it was verified.
- Any unresolved issues, risks, or limitations.
