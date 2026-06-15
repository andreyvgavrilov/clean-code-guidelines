# Clean code guidelines with agent engineering

Basic instruction set for Claude Code and other coding tools.

## Problems

- LLM coding pitfalls from [Andrej Karpathy's post](https://x.com/karpathy/status/2015883857489522876)
- Agent context limitations as indicated in [Humanlayer post](https://www.humanlayer.dev/blog/advanced-context-engineering)

## Solution

The files contain combination of several solutions:
- Research - Planning - Implementation framework from [Humanlayer post](https://www.humanlayer.dev/blog/advanced-context-engineering)
- [Karpathy-Inspired Claude Code Guidelines](https://github.com/multica-ai/andrej-karpathy-skills)
- TDD framework, so AI is instructed to write failing tests before writing any code

## When this approach is **NOT** for you

If you use "fire and forget" approach with coding tools, implementing features without human expertise, this may not work great. Both Karpathy-Inspired guidelines and Humanlayer framework rely heavily on asking human for clarifications and confirmations.

## Additional notes

- OpenAI Codex require explicit user instructions to launch subagents at the moment. Use something like this in your initial prompt for each session: *For this entire session, you are authorized to use subagents whenever AGENT.md requires them*
- By default subagents can't interact with user directly. So, this file includes relayer logic to interact with user through orchestrator.
- This file is agnostic to programming languages and frameworks (as long as they support tests), so it may need project-specific tweaks