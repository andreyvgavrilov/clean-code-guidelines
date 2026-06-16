# Agent Feature Implementation Workflow

Use this workflow for feature implementation requests. The goal is to separate research, planning, approval, and implementation so the final work is grounded in the codebase, explicitly scoped, and approved by the user before code changes begin.

## Principles

### Think Before Coding

Do not assume silently. Surface uncertainty before writing code.

- State assumptions explicitly.
- If the request has multiple reasonable interpretations, present them instead of choosing silently.
- Push back when a simpler approach would satisfy the goal.
- Stop when confused. Name what is unclear and ask for clarification.

Subagents cannot ask the user directly. When a subagent needs human input, it must stop and return a human input request to the orchestrator.

### Simplicity First

Use the minimum code that solves the approved goal.

- Do not add features beyond what was requested.
- Do not create abstractions for single-use code.
- Do not add flexibility, configurability, or extension points unless requested.
- Do not add error handling for impossible scenarios.
- If the implementation can be reduced substantially without losing behavior, simplify it before finishing.

### Surgical Changes

Touch only what is necessary.

- Do not improve adjacent code, comments, names, formatting, or structure while passing through.
- Do not refactor unrelated code.
- Match the existing style, even if a different style would be preferred.
- Remove imports, variables, functions, tests, and files that became unused because of this change.
- If unrelated dead code or cleanup is noticed, mention it in the final result instead of deleting it.

### Goal-Driven Execution

Turn every implementation request into a verifiable goal.

- Define success criteria before implementation.
- Never implement a feature before writing a failing test that verifies the goal.
- After writing code, if the test still fails, assume the problem is in the code by default.
- Change tests only when necessary, and only if the revised tests still verify the original goal.
- Continue until the success criteria are verified or a real blocker is surfaced.

## Orchestrator and Subagent Communication

The main agent is the orchestrator. Subagents do focused work, but the orchestrator owns all communication with the user.

If a subagent encounters ambiguity, confusion, a meaningful tradeoff, a simpler alternative that changes scope, or a decision that requires product judgment, it must pause and return a human input request instead of guessing.

A human input request should include:

- The unclear point or decision.
- Why it matters.
- The reasonable options, if known.
- The subagent's recommended option, if it has one.

When the orchestrator receives a human input request, it must ask the user for clarification or approval. After the user responds, the orchestrator must return the user's feedback to the subagent as additional input and let the subagent continue from that point.

If the user's answer changes the feature substantially, the orchestrator should route back to research or planning as described in Step 3.

## Step 1: Research

Call a subagent to research the information needed for implementation.

The research subagent should inspect the relevant code, tests, documentation, configuration, and existing patterns. It should not implement the feature. It should return a concise research result that includes:

- Relevant files, modules, and ownership boundaries.
- Current behavior and data flow.
- Existing patterns that should be followed.
- Risks, unknowns, constraints, and likely test surfaces.
- Ambiguities, assumptions, and multiple possible interpretations.
- Simpler alternatives or reasons to push back, if any.
- Decisions that need to be made before implementation.

If the research subagent cannot produce a useful research result without user input, it must return a human input request to the orchestrator. The orchestrator must ask the user and return the user's answer to the research subagent before research continues.

The main agent should preserve the research result and use it as input for the planning step.

## Step 2: Planning

Call a clean subagent to create the feature plan.

Input to the planning subagent:

- The user's original request.
- The research result from Step 1.

The planning subagent should not implement the feature. It should return an implementation plan that includes:

- Success criteria.
- Proposed approach.
- Files expected to change.
- Data model, API, UI, or behavior changes, if applicable.
- Failing test or tests to write before implementation.
- Focused verification commands to run after implementation.
- Rollout or migration notes, if applicable.
- Open questions or assumptions.

If the planning subagent needs product judgment, scope clarification, or approval of a tradeoff before it can create a responsible plan, it must return a human input request to the orchestrator. The orchestrator must ask the user and return the user's answer to the planning subagent before planning continues.

The main agent should review the plan for clarity, scope, and simplicity before presenting it to the user. If the plan includes speculative work, broad refactors, unnecessary abstractions, or changes outside the user's request, send it back for revision before user approval.

## Step 3: User Approval

Present the implementation plan to the user for approval before making code changes.

If the user approves the plan, proceed to Step 4.

If the user provides comments, evaluate the impact:

- If the comments dramatically change the approach, return to Step 2.
- If the comments reveal missing or uncertain technical context, return to Step 1.
- If the comments can be accepted by adjusting the current plan, proceed to Step 4 using the research result, the implementation plan, and the user comments as input.

Do not begin implementation until the user has approved the plan or given comments that clearly authorize proceeding with the adjusted plan. If the user's comments are ambiguous, ask for clarification instead of guessing. If a subagent needs clarification during re-planning, use the orchestrator human input request flow.

## Step 4: Implementation

Call another subagent to implement the feature.

Input to the implementation subagent:

- The user's original request.
- The research result from Step 1.
- The implementation plan from Step 2.
- Any user comments or requested adjustments from Step 3.

The implementation subagent should make the code changes, add or update tests where appropriate, and run focused verification. It should return regular implementation results that include:

- The success criteria used.
- The failing test written before implementation and its initial failure result.
- Summary of changes made.
- Files changed.
- Tests or checks run, including failures.
- Any remaining risks, limitations, or follow-up work.

The implementation subagent must keep changes surgical and simple. It should not refactor unrelated code, add speculative capability, or clean up pre-existing dead code. It should remove only unused code created by its own changes.

If the implementation subagent discovers that the approved plan is impossible, unsafe, substantially more complex than expected, or conflicts with the success criteria, it must stop and return a human input request to the orchestrator. The orchestrator must ask the user and return the user's answer to the implementation subagent before implementation continues, unless the answer requires returning to research or planning.

The main agent should review the implementation result, perform any necessary final verification, and present the final results to the user. If verification fails, continue debugging the implementation. Do not weaken tests unless the tests no longer verify the approved goal.

## General Rules

- Keep scope tightly aligned with the user's request.
- Do not let implementation begin during research or planning.
- Make assumptions, tradeoffs, and ambiguity visible.
- Route all subagent requests for human input through the orchestrator.
- Prefer existing project patterns over new abstractions.
- Preserve user changes and avoid unrelated refactors.
- Write failing tests before feature code.
- Verify against the approved success criteria.
- If approval feedback changes the feature significantly, re-plan before implementation.
- Final responses should clearly summarize what changed, how it was verified, and any unresolved issues.
