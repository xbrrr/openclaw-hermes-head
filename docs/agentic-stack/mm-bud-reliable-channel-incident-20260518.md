# MM/Bud Reliable Channel Incident — 2026-05-18

## Summary

Mikhail asked Bud to ask MM/Hermes for runtime settings. Bud sent a public Telegram message in Agentic Stack topic 642:

- chat_id: `-1003772186616`
- topic_id: `642`
- Bud message_id: `1588`
- text begins: `@ceo5000_bot Mikhail спрашивает: какие у тебя сейчас runtime-настройки?`

Bud later checked local `chat-events.jsonl` and did not find a Hermes response to message `1588`. Mikhail reported that Hermes may have answered elsewhere, which means the public Telegram bot-to-bot path is ambiguous and not a reliable control channel for Bud -> Hermes asks.

## Evidence

- `chat-events.jsonl` contains Bud message `1588` in topic `642`.
- No matching Hermes response after `1588` was found in the local Agentic Stack observer log at check time.
- Existing project history already notes Telegram bot-to-bot wake can be unreliable.
- Telegram bot-to-bot delivery/visibility is not a safe dependency for operational coordination.

## Root Cause Hypothesis

Bud used a public Telegram mention to wake Hermes. That path depends on Telegram bot-to-bot behavior and each bot's update visibility. For reliable internal coordination, Bud should not rely on Telegram bot-to-bot wake.

## Required Design Direction

Use a reliable local/internal Hermes channel for Bud -> Hermes advisory requests:

1. Primary: local Hermes CLI oneshot via `/Users/xbr/.local/bin/hermes-local -z ...`.
2. Audit: write request/response artifacts under `/Users/xbr/.agentic-stack/reliable-mm/`.
3. Telegram public topic 642 remains useful for human-visible coordination, but not as the only control channel for Bud -> Hermes.
4. If public Telegram ping gets no ACK/answer, Bud should automatically fallback to local Hermes CLI and continue, without waiting for Mikhail.

## Request To Hermes/MM

Please review this incident and recommend the minimal safe fix. Focus on:
- reliable Bud -> Hermes advisory channel;
- how to keep public auditability without depending on Telegram bot-to-bot wake;
- what exact protocol and file/script should be added to `.agentic-stack`;
- what rules should change in `README.md` and `chats.yaml`.

Return compactly:
- root cause;
- recommended fix;
- protocol;
- risks;
- acceptance criteria for Bud implementation.

