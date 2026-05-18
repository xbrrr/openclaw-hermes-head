# Agentic Stack shared routing rules

This directory is the shared, cautious MVP workspace for Hermes / Master Mind and OpenClaw / Bud Askins coordination in Telegram.

## Stack Goal

Build Agentic Stack as a topic-scoped multi-agent system:

- Every Telegram topic can have its own lane policy.
- Topic 642 is the public Hermes/Bud coordination lane with a logged task loop.
- Topic 723 is the Hermes-only lane.
- Other existing/future topics stay on safe default until explicitly registered.
- Shared board/watchdog/probe tooling is a passive audit layer, not a global router.
- Routing decisions are scoped by topic, not applied globally to the whole chat.

## Scope

The coordination-lane rules below apply only to:

- Platform: Telegram
- Group: Agentic Stack
- chat_id: `-1003772186616`
- topic_id: `642`
- topic: `MM`

They do **not** apply to other topics or the whole group.

For other topics, use the policy registry in chats.yaml. If a topic is not explicitly registered, it uses safe default: no inherited topic-642 Hermes/Bud loop, no global routing/enforcement change, and Bud stays silent unless directly addressed.

## Tool Routing

X/Twitter/social search has a special route because direct `x.com` web/browser scraping is Cloudflare-prone.

- For open-ended X/social research, Hermes/MM must use the Perplex Sonar helper: `perplex <query> [quick|pro|reason|deep]`. This path sends `search_sources=["social","web"]` through Perplexity chat/completions.
- For exact tweet/thread/article URLs, use the `bird` skill path: SocialData API first, Twitter/X cookies fallback second, browser only as last resort.
- Do not use direct `x.com` browser automation or plain web fetch as the primary path for X/social search.
- Use OpenClaw `web_search` only after Perplex/Sonar helper failure or for provider smoke tests, and say explicitly that this is a fallback.

## Roles

- Hermes / Master Mind (`@ceo5000_bot`)
  - Role: coordinator / head
  - Answers normal messages by default in this topic.
  - Designs plans, assigns implementation tasks, verifies results, and reports back to Mikhail.

- OpenClaw / Bud Askins (`@iq5000_bot`)
  - Role: executor
  - Does not answer normal messages by default in this topic.
  - Answers when directly tagged, when a user replies to Bud's own message, or when given an internal task by Hermes once the queue is enabled.

## Routing priority

1. Reply to a known agent's message -> that same agent answers.
2. Explicit mention -> mentioned agent answers.
3. Normal message in topic 642 -> Hermes answers.
4. If uncertain -> Hermes fallback.

## Current rollout mode

Current mode: `documented_only`.

This means:

- No automatic routing enforcement yet.
- No queue processing yet.
- No auto-suppression of agents yet.
- These files are only the source of truth for humans/agents to read.

## Board sync

The local lifecycle board is append/update-only:

- DB: `agent-board.sqlite3`
- Helper: `agent_board.py`
- Source log: `chat-events.jsonl`

Auto-run sync is enabled in the logging helpers:

- `outbound_task_logger.py` runs `agent_board.py sync` after a Hermes outbound `/task@iq5000_bot` event is appended.
- `append_chat_event.py` runs `agent_board.py sync` after an inbound scoped chat event is appended, so Bud ACK/status messages update the board.

The sync is fail-open: if board sync fails, the chat event append still succeeds and the sync error is returned in the helper JSON output. This keeps logging/routing behavior unchanged.

Hermes -> Bud task handoffs should use the send-message/tool path that returns a Telegram message_id and then invokes outbound_task_logger.py. A plain Hermes final response containing /task@iq5000_bot may be visible in Telegram but not synthetic-logged as a Hermes outbound task. Use outbound_task_probe.py --correlation-id <id> to verify that a new task is visible in both chat-events.jsonl and agent-board.sqlite3.

## Reliable Bud -> Hermes channel

Telegram topic 642 is the public audit and coordination surface, but Bud must not rely on Telegram bot-to-bot mentions as the only control channel for Bud -> Hermes advisory requests. Telegram bot-to-bot wake can be ambiguous: a message may be visible to humans while the other bot does not receive it as an actionable inbound event.

For Bud -> Hermes advisory requests, use the local reliable channel:

- Helper: `bin/ask-mm`
- Artifacts: `reliable-mm/<correlation_id>/`
- Local Hermes wrapper: `/Users/xbr/.local/bin/hermes-local`

Protocol:

1. Bud writes a request through `bin/ask-mm --correlation-id <id> --prompt ...`.
2. The helper stores `request.md`, `meta.json`, and `response.md`.
3. Bud verifies the response and executes the concrete fix.
4. If public visibility is useful, Bud summarizes the result in Telegram after the local channel succeeds.

Use public `@ceo5000_bot` mentions only for human-visible coordination and audit, not as the required wake path.

## Subconscious memory v1

Local structured memory for Hermes/Bud coordination lives under:

- Helper: `subconscious_memory.py`
- Store: `subconscious/`
- Layers: `working`, `episodic`, `semantic`, `procedural`
- Reflection reports: `subconscious/reflections/YYYY-MM-DD.md`

This v1 is deliberately local and stdlib-only. It does not require Mem0,
Hindsight, API keys, or a runtime restart. It preserves existing built-in
memory, session_search, skills, board sync, and topic policies.

Useful commands:

- `python3 subconscious_memory.py init`
- `python3 subconscious_memory.py reflect`
- `python3 subconscious_memory.py doctor`
- `python3 subconscious_memory.py self-test`
- `python3 subconscious_memory.py install-cron`

The daily reflection job reads existing chat/board/topic artifacts, writes a
compact daily report, records durable semantic/procedural facts, and compacts
expired/duplicate local memory entries. It is passive: it does not send
Telegram messages and does not change routing.

## Planned rollout

1. `documented_only` — create and validate shared files.
2. `dry_run` — log routing decisions without changing behavior.
3. `soft_enforced` — suppress the wrong agent only within this exact topic.
4. `agent_queue` — Hermes assigns tasks to Bud through `agent-queue.jsonl`.
5. Bridge/API — replace JSONL with SQLite/API when stable.

## Safety rules

- Never apply these rules outside `chat_id=-1003772186616` and `topic_id=642`.
- Never inherit topic 642 rules into topic 723 or unrelated topics.
- Do not change global routing/enforcement/require_mention as part of topic 642 work.
- Do not perform destructive actions without explicit approval.
- Bud should not post public replies for internal queue tasks unless the task explicitly allows it.
- If routing is uncertain or errors, fallback to Hermes or current default behavior rather than dropping the message.
