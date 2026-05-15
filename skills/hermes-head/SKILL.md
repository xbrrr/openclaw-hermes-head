---
name: hermes-head
description: Use when Hermes Agent should act as strategic/self-learning head while OpenClaw remains the concrete executor. Routes goals to Hermes only through safe local CLI boundaries; does not start Hermes gateway or migrate secrets.
metadata:
  version: 0.1.0
  installed: 2026-05-15
---

# hermes-head

Use this skill when the user wants Hermes Agent to act as the planning / learning / long-running strategy layer and OpenClaw to execute the concrete work.

## Role split

- Hermes = head: strategy, long goals, decomposition, self-learning, memory experiments, Kanban/checkpoints/MCP research.
- OpenClaw = executor: Telegram surface, tools, files, messages, subagents, healthchecks, verification, user-facing reports.

## Safety defaults

- Do not run `hermes setup`, `hermes claw migrate`, `hermes gateway start`, or any installer script without explicit approval.
- Do not migrate OpenClaw secrets into Hermes automatically.
- Do not share the OpenClaw Telegram token with Hermes.
- Prefer `hermes --oneshot` or CLI inspection over long-running Hermes processes.
- Keep Hermes outputs advisory until OpenClaw verifies and executes.

## Current local install

- Hermes repo: `/Users/xbr/.local/src/hermes-agent`
- CLI: `/Users/xbr/.local/bin/hermes`
- Safe wrapper: `/Users/xbr/.local/bin/hermes-local`
- Architecture note: `/Users/xbr/.openclaw/workspace/docs/openclaw-hermes-architecture-2026-05-15.md`

## Workflow

1. Clarify the goal and execution boundary.
2. Ask Hermes for strategy/decomposition only if it adds value.
3. Convert Hermes advice into OpenClaw-executable steps.
4. Execute with OpenClaw tools/subagents.
5. Verify with direct evidence.
6. Store reusable learning in OpenClaw memory or a dedicated public-safe note.

## Recommended pairing

- `get-shit-done` for phase loop.
- `shaw` for coding quality gates.
- `systematic-debugging` for incidents/bugs.
- `migration-cutover-safety` for any runtime switch.
- `healthcheck`, `guardian-angel`, `server-doctor` for host/service operations.
- `verification-before-completion` before claiming done.
