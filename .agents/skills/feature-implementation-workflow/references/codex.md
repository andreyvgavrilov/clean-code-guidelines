# Codex Adapter

Read this file only when running in Codex.

Codex subagent definitions are external to this skill and live in `.codex/agents/`. Do not reproduce or redefine their system prompts here.

## Phase Agents

- `feature-research`: Read-heavy codebase exploration based on Codex's `explorer` role. It must not edit files or implement the feature.
- `feature-planning`: Produces a scoped implementation plan from the request and research result. It must not edit files or implement the feature.
- `feature-failing-tests`: Writes and runs the approved failing tests or checks without implementing production behavior.
- `feature-implementation`: Executes the approved implementation based on Codex's `worker` role while preserving the failing-test agent's tests.

## Invocation

Invoke the matching project-scoped subagent for each workflow phase:

1. Invoke `feature-research` with the original request and relevant workspace context.
2. Invoke `feature-planning` with the original request and complete research result.
3. After user approval, invoke `feature-failing-tests` with the original request, research result, approved plan, and user adjustments.
4. Invoke `feature-implementation` with all prior inputs plus the failing-test result.

Use Codex's subagent messaging tools to relay follow-up context or the user's answer to a human input request. Preserve complete agent results between phases rather than asking later agents to rediscover them.

If a named project-scoped agent is unavailable, do not substitute a differently scoped agent silently. Perform that phase directly and tell the user which delegation was unavailable.
