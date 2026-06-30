---
name: feature-implementation-workflow
description: Use within the main/orchestrator agent when the user asks to create a new feature or modify an existing feature. Guides tool-adapted agent research, planning, approval, failing-test authoring, implementation, human-input relay, and verification before feature code changes.
---

# Feature Implementation Workflow

Use this workflow for feature implementation requests. Separate research, planning, approval, failing-test authoring, and implementation so final work is grounded in the codebase, explicitly scoped, and approved before code changes begin.

## Select The Tool Adapter

Before starting the workflow, inspect the available built-in agent tools and load exactly one adapter:

- If `define_subagent` and `invoke_subagent` are available, read [references/antigravity.md](references/antigravity.md). Do not read the Codex adapter.
- Otherwise, if Codex subagent tools and project-scoped agent definitions are available, read [references/codex.md](references/codex.md). Do not read the Antigravity adapter.
- If neither adapter is usable, perform the same phases directly, preserve the user approval gate before implementation, and tell the user that delegation is unavailable.

Use the selected adapter only for tool-specific agent definitions, invocation, and communication. Use this file as the authority for phase order, inputs, outputs, approval, and test ownership.

The main agent is the orchestrator. It owns all user communication, preserves phase outputs, reviews delegated work, and decides whether user feedback returns the task to research, planning, failing-test authoring, or implementation.

Delegated agents cannot ask the user directly. If an agent needs product judgment, scope clarification, or human approval, it must stop and return a human input request containing:

- The unclear point or decision.
- Why it matters.
- Reasonable options, if known.
- The agent's recommendation, if it has one.

The orchestrator asks the user, then returns the user's answer to the agent or routes back to the appropriate earlier phase.

## Step 1: Research

Run the research agent through the selected adapter to inspect the relevant code, tests, documentation, configuration, and existing patterns. The research agent must not implement the feature.

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

Run the planning agent through the selected adapter to create the feature plan. Provide the user's request and the research result. The planning agent must not implement the feature.

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
- Proceed to failing-test authoring, then implementation, when comments only adjust the current plan and clearly authorize the work.

Ask for clarification when approval feedback is ambiguous.

## Step 4: Failing Tests

Run the failing-test agent through the selected adapter to write the failing test or failing check from the approved plan. Provide the original request, research result, approved plan, and any user adjustments.

The failing-test agent must:

- Keep changes tightly scoped to tests or test fixtures needed for the approved goal.
- Avoid production implementation changes.
- Run the focused test or check to confirm it fails for the expected reason before implementation begins.
- Return a human input request if the approved test plan is impossible, unsafe, ambiguous, or conflicts with the success criteria.

The failing-test result should include:

- The success criteria covered by the test or check.
- Files changed.
- The exact test or check command run.
- The initial failure result and why it is expected.
- Any concerns about the test design or implementation handoff.

Preserve the failing-test result as input for implementation. The implementation agent does not own these tests.

## Step 5: Implementation

Run the implementation agent through the selected adapter. Provide the original request, research result, approved plan, any user adjustments, and the failing-test result.

The implementation agent should:

- Keep changes tightly scoped to the approved goal.
- Implement the minimum behavior needed to satisfy the success criteria.
- Treat tests written by the failing-test agent as owned input and do not modify them.
- Add or update other tests only when they still verify the approved goal and do not change the failing-test agent's tests.
- Run focused verification.
- Remove only unused code introduced by its own changes.

If the approved plan is impossible, unsafe, substantially more complex than expected, or conflicts with the success criteria, the implementation agent must stop and return a human input request.

If the implementation agent finds an error in the failing-test agent's tests, such as an invalid assertion, wrong fixture, conflicting expectation, or test infrastructure problem, it must not edit those tests. It should report the issue in its implementation result after finishing the implementation work it can safely complete. The orchestrator decides whether to recall the failing-test agent with that feedback before final verification.

The implementation result should include:

- The success criteria used.
- The failing-test result it received and whether the implementation now satisfies it.
- Summary of changes made.
- Files changed.
- Tests or checks run, including failures.
- Any reported issues with tests written by the failing-test agent.
- Remaining risks, limitations, or follow-up work.

## Orchestrator Finish

Review the implementation result and perform any necessary final verification. If the implementation agent reports an issue with tests written by the failing-test agent, route that feedback back to the failing-test agent when the test needs correction. If verification fails because the implementation is wrong, continue debugging the implementation rather than weakening tests.

Final responses should clearly summarize what changed, how it was verified, and any unresolved issues.
