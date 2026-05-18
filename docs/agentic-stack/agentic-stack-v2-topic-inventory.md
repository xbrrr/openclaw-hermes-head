# Agentic Stack v2 Blueprint v0.7 — Topic Inventory + Role Mapping

Date: 2026-05-18
Status: final documented inventory / chats.yaml validated
Scope: Telegram group "Agentic Stack" and Agentic Stack v2 operating model
Source inputs: Hermes registry + Bud inventory chunks `correlation_id=topic-inventory-2026-05-17` + refreshed Hermes messaging targets/chats.yaml on 2026-05-18 + Bud refresh request `correlation_id=topic-inventory-refresh-20260518` + 2026-05-18 daily delta for Server-doctor/Subconscious topics + Mikhail clarification: Analyst title/id via Telegram link, topic 1 ignored/disabled + Bud alignment ACK `correlation_id=topic-inventory-final-align-20260518`

## Executive summary

Agentic Stack v2 should start from the real Telegram workspace structure, not from an abstract list of agents.

Current direction:

- Treat each Telegram topic as a separate lane with its own policy.
- Hermes / Master Mind is the top-level coordinator and default management layer.
- Bud / OpenClaw is executor, not default chatter, unless explicitly triggered.
- Per-topic Bud skill sets must be preserved as routing/knowledge bindings.
- If Hermes answers in a domain topic, Hermes should load/use the relevant topic skill set first.
- If the task is execution-heavy — files, code, runtime/gateway, tables, media extraction, scripts, board/watchdog lifecycle — Hermes should triage/structure and delegate to Bud.
- Do not inherit `MM / topic 642` coordination rules globally.
- Do not change runtime routing/enforcement yet; this file is a documented-only policy map.

## Core routing rule

- Direct `@iq5000_bot` wakes Bud for quick direct questions/tasks.
- `/task@iq5000_bot correlation_id=...` is for formal tasks needing lifecycle, board, watchdog, or durable audit.
- Explicit bot mention beats reply context.
- Bud should not become default responder in domain topics.

## Current topic inventory

Refresh delta 2026-05-18:
- Hermes messaging targets currently visible: topic `92`, `723`, `1`, `642`, `1347`, `1346`.
- New/changed since v0.2: `1347` is now registered in `chats.yaml` as `Subconscious`; `1346` is now mapped as `Server-doctor`.
- Previously inventoried Bud/domain topics: `259 / Perplex`, `584 / Summarize`, `21 / SKILLS (2) / Music`, `488 / HR`, plus CFO skills/history. `584/21/488` are retained as historical/Bud domain lanes and are not currently visible in Hermes `send_message(action=list)` targets.
- Mikhail provided Telegram link `https://t.me/c/3772186616/4/1547`; parsed as `chat_id=-1003772186616`, `topic_id=4`, `message_id=1547`, so `Analyst` is topic `4`.
- Mikhail clarified: `topic 1` should be ignored; Hermes and Bud should not answer there.
- Bud refresh requested publicly in topic 642: `correlation_id=topic-inventory-refresh-20260518`.

### 1. MM / topic 642

Lane type: public coordination lane.

Purpose:
- Hermes ↔ Bud public coordination.
- Handoff, board/watchdog/probe lifecycle.
- Routing smoke-tests and agent-agent operating model.

Skill bindings from Bud:
- `hermes-head`
- `get-shit-done`
- `systematic-debugging`
- `server-doctor`
- `migration-cutover-safety`
- `openclaw-update-safety`
- `telegram-bot-coding`

Rules:
- Hermes is default coordinator/responder.
- Bud is executor.
- Direct `@iq5000_bot` wakes Bud.
- Direct `@ceo5000_bot` wakes Hermes.
- Explicit mention beats reply context.
- `/task@iq5000_bot` only for lifecycle/board/watchdog-level tasks.

Hermes behavior:
- Answer directly for coordination, planning, decisions, synthesis.
- Delegate execution-heavy work to Bud.
- Keep agent-agent coordination public unless Mikhail explicitly requests private handling.

Audit/artifacts:
- `/Users/xbr/.agentic-stack/`
- `agent-board.sqlite3`, `task-board.sqlite3`
- `chat-events.jsonl`, `routing-decisions.jsonl`
- watchdog/probe/outbound logging scripts.

Refresh 2026-05-18:
- Still present in `send_message(action=list)`: `telegram:Agentic Stack / topic 642`.
- `chats.yaml` currently has this lane as the only `soft_enforced` topic.
- Direct `@iq5000_bot` must wake Bud for quick questions/tasks; `/task@iq5000_bot` remains for formal lifecycle tasks.

### 2. MM Only / topic 723

Lane type: Hermes-only lane.

Purpose:
- Direct Mikhail ↔ Hermes work.
- Strategy, decisions, planning, review, quiet management context.

Skill bindings from Bud:
- `hermes-head`
- `writing-plans`
- `reasoning-personas`
- `get-shit-done` as needed

Rules:
- Bud silent by default.
- Bud only participates if explicitly requested or correcting its own prior mistake.
- Executor work should usually be moved to `MM / topic 642` with logged handoff.

Hermes behavior:
- Answer directly.
- Do not pull Bud into topic 723 by default.
- If Bud approves a plan in the established flow, continue without asking Mikhail again unless risk changes.

### 3. Web Admin / topic 92 / SKILLS (4)

Lane type: coding / web-admin / IT automation lane.

Purpose:
- Coding, web-admin, IT management, automation.
- InSales work.
- Telegram bots.
- Runtime/gateway/config/update work.

Skill bindings from Bud:
- `hermes-head`
- `get-shit-done`
- `using-superpowers`
- `shaw`
- `migration-cutover-safety`
- `openclaw-update-safety`
- `systematic-debugging`
- `healthcheck`
- `guardian-angel`
- `server-doctor`
- `insales`
- `telegram-bot-coding`

Rules:
- Before runtime/gateway/config/update actions, read source of truth and profile skill.
- For OpenClaw updates, use `openclaw-update-safety` + runbook.
- Avoid improvising operational changes without checking the relevant skill/source files.

Hermes behavior:
- Can answer directly only after loading relevant topic skill set.
- Delegate operations, patches, diagnostics, deployments, or risky config/runtime work to Bud or follow Bud-style execution protocol.

### 4. Perplex / topic 259

Lane type: research / critique / reports lane.

Purpose:
- Search, research, critique, reports.
- External source gathering and synthesis.

Skill bindings from Bud:
- `perplex`
- `deep`
- `markdown-new/pdf`
- `bird`
- `digest`
- `reasoning-personas`
- niche skills: `crypto-price`, `weather`, `music`, `bestchange`, `telegram-chip`

Rules:
- Source quality matters.
- For important decisions, separate facts, assumptions, and opinion.
- Use citations/sources for external research.

Hermes behavior:
- Can run research directly if loaded with `perplex`/`deep`-class skills.
- Use Bud for batch research, file generation, automation, and source verification workflows.

### 5. Summarize / topic 584

Lane type: summarization / digest lane.

Purpose:
- Summaries of video, audio, YouTube, Telegram, articles, PDFs, long threads.
- Executive digests and action extraction.

Skill bindings from Bud:
- `content-summarizer`
- `perplex`
- `deep`
- `video-frames`
- `markdown-new/openclaw_pdf`
- `digest`
- `bird`

Tools/scripts likely involved:
- `yt-dlp`
- `ffmpeg`
- `whisper`

Rules:
- Do not summarize sensitive content into broader channels without permission.
- Keep summaries concise and action-oriented unless long-form requested.

Hermes behavior:
- Answer directly for short summaries and synthesis.
- Delegate extraction, batch, media processing, PDFs, long files, and transcript generation to Bud.

### 6. Music / topic 21 / SKILLS (2)

Lane type: DJ / music discovery lane.

Purpose:
- DJ/music discovery.
- Track recognition.
- Beatport/Soundeo workflows.
- CSV/Markdown for Rekordbox.

Skill bindings from Bud:
- `music`

Rules:
- Preserve track metadata quality.
- For Rekordbox outputs, format files consistently.

Hermes behavior:
- Can receive the request and hold light dialogue.
- Use Bud/skill execution for recognition, list building, files, CSV/Markdown outputs.

### 7. CFO / topic ID unknown / Skills history

Lane type: finance / analytics lane.

Current status:
- Prior Bud inventory associated CFO with `topic 4 / Skills`, but Mikhail later provided a Telegram link proving `topic_id=4` is `Analyst`.
- CFO is therefore kept as a skill/domain history entry until an exact current topic ID/title is confirmed.

Purpose:
- CFO contour.
- Excel/Sheets audit.
- Product analytics.
- Yandex.Metrica finance connector.

Skill bindings from Bud:
- `excel-financial-auditor`
- `yandex-metrica-finance-connector`
- `product-abc-xyz`
- `ai-research-analyst`

Rules:
- Hermes should not improvise finance/table conclusions without relevant skills and source data.
- Keep assumptions explicit.
- Treat writes to financial sheets/databases as approval-required unless already scoped.

Hermes behavior:
- Load CFO/finance skills before answering materially.
- Delegate files, tables, scripts, spreadsheet audits, and connector/API work to Bud.

### 8. HR / topic 488

Lane type: HRBP / people operations lane.

Purpose:
- Recruiting.
- Performance.
- Employee relations.
- People analytics.
- Psychology/coaching support without clinical diagnosis.

Skill bindings from Bud:
- `hr-coach`
- `perplex`

Scripts/artifacts from Bud:
- `hr_interview_kit.py`
- `hr_resume_match.py`
- `hr_employee_listening.py`
- `hr_redact.py`
- `hr_er_case_map.py`
- `hr_performance_review.py`
- `hr_coaching_plan.py`

Rules:
- Human-in-the-loop.
- Confidentiality.
- Anti-bias.
- No auto-reject.
- No external candidate outreach without explicit approval.
- No clinical diagnosis.

Hermes behavior:
- Can answer directly only with `hr-coach` context loaded.
- Delegate documents, tables, resume matching, PII redaction, templates, and batch workflows to Bud.

### 9. Analyst / topic 4 / title confirmed

Lane type: analytics / research lane.

Current status:
- Mikhail provided Telegram source link `https://t.me/c/3772186616/4/1547`.
- Parsed mapping: `chat_id=-1003772186616`, `topic_id=4`, `message_id=1547`.
- Therefore current confirmed mapping is `topic 4 = Analyst`.
- Prior Bud inventory associated `topic 4` with CFO/Skills; that is now treated as historical/conflicting data. CFO is preserved as skill/domain history with `topic ID unknown` until separately confirmed.

Skill bindings:
- Primary: `ai-research-analyst`, `perplex`, `deep`.
- Supporting when finance/spreadsheets/metrics are involved: `excel-financial-auditor`, `product-abc-xyz`, `yandex-metrica-finance-connector`.

Needed:
- No blocker for Analyst; topic ID is confirmed.
- Optional later: confirm the current CFO topic/title if CFO remains an active Telegram topic.

Hermes behavior:
- Answer directly for research triage, analysis framing, source review, synthesis, and lightweight analytics.
- Delegate data files, scripts, batch source checks, spreadsheet/report generation, and connector/API work to Bud.
- Do not route Analyst work to CFO solely because of the old historical `topic 4 = CFO` entry.

### 10. Subconscious / topic 1347

Lane type: Hermes subconscious / memory-system development lane.

Purpose:
- Development and operation of Hermes local subconscious/memory helper.
- Reflection, semantic/procedural/episodic/working local stores.
- Safe iteration on memory architecture without replacing built-in Hermes memory/session search/skills.

Skill bindings from current Hermes registry/chats.yaml:
- `hermes-agent` when modifying Hermes itself.
- `systematic-debugging` for defects/regressions.
- `test-driven-development` / `spike` for helper changes as appropriate.

Bud-side skill bindings used in the 2026-05-18 implementation/review:
- `hermes-head` / Hermes-context review
- `systematic-debugging`
- `test-driven-development` for helper self-tests/checks where appropriate
- `get-shit-done` for safe local implementation flow

Known artifacts:
- `/Users/xbr/.agentic-stack/subconscious_memory.py`
- `/Users/xbr/.agentic-stack/subconscious/working.jsonl`
- `/Users/xbr/.agentic-stack/subconscious/episodic.jsonl`
- `/Users/xbr/.agentic-stack/subconscious/semantic.jsonl`
- `/Users/xbr/.agentic-stack/subconscious/procedural.jsonl`
- `/Users/xbr/.agentic-stack/subconscious/reflections/`

Rules:
- Hermes is owner/default coordinator.
- Bud is implementation reviewer/executor only when directly called or formally tasked.
- No runtime restart/cutover without separate smoke plan.
- Preserve built-in Hermes memory/session_search/skills; local subconscious is additive, not replacement.

Hermes behavior:
- Answer/design directly for memory architecture and policy.
- Load `hermes-agent` before Hermes config/runtime/tool changes.
- Delegate implementation/review/testing to Bud when execution-heavy.

### 11. Server-doctor / topic 1346

Lane type: ops / runtime diagnostics / incident lane.

Current status:
- Visible today in `send_message(action=list)` as `telegram:Agentic Stack / topic 1346`.
- Confirmed in Bud daily delta as dedicated `Server-doctor` topic.
- Added to `chats.yaml` as documented-only policy on 2026-05-18 refresh.

Purpose:
- Hosts, OpenClaw, Telegram bots, Docker/launchd/systemd, health checks, incidents, restarts, postmortems.
- Split from `Web Admin`: Web Admin stays UI/admin/settings/product web tasks; Server-doctor owns runtime/host/ops work.

Skill bindings from Bud:
- `server-doctor`
- `systematic-debugging`
- `healthcheck`
- `migration-cutover-safety`
- `openclaw-update-safety`
- `telegram-bot-coding` when Telegram bot runtime is involved

Working protocol:
- baseline -> host/runtime/access map -> diagnostics -> classify `down/degraded/partial/unstable` -> safe action -> verification -> rollback/stop condition -> short report.
- Before restart/reset/cutover, verify this is a fresh symptom, inspect logs, and choose the minimum action.
- Avoid repeated blind gateway/service restarts.
- Prefer monitored/supervised service operations with smoke checks.

Hermes behavior:
- Can triage and summarize ops state directly after loading the Server-doctor skill set.
- Delegate execution-heavy diagnostics, host commands, config edits, restarts, rollback/smoke checks, and incident repair to Bud.
- Use `/task@iq5000_bot` for lifecycle/board/watchdog tracked incident work; direct `@iq5000_bot` is fine for quick checks.
- Do not inherit `MM / topic 642` coordination rules.

### 12. topic 1 / ignored

Lane type: ignored / disabled.

Current policy:
- Mikhail clarified on 2026-05-18: ignore topic 1.
- Do not inherit `MM / topic 642` rules.
- Hermes should not answer there.
- Bud should not answer there.

Needed:
- None unless Mikhail re-enables it later.

### 13. Unregistered/unknown topics

Lane type: safe default.

Default behavior:
- Hermes only if addressed/context requires.
- Bud silent unless directly requested.
- Passive audit only.
- No runtime enforcement before documented policy and smoke tests.

## Agent role map

### Hermes / Master Mind

Primary role:
- Coordinator, planner, final synthesizer, manager-level responder.

Default in:
- `MM / topic 642`
- `MM Only / topic 723`
- Domain topics after loading relevant skill set.

Must do:
- Preserve topic skill binding.
- Answer directly when the task is conversational/managerial and skill context is known.
- Triage/delegate when execution-heavy.

### Bud / OpenClaw

Primary role:
- Executor/operator.

Active when:
- Direct `@iq5000_bot` quick request.
- `/task@iq5000_bot` lifecycle task.
- Hermes delegates execution.

Should not:
- Become default responder in domain topics.
- Perform external side effects without approval/scope.

### Future domain agents

- Research / Analyst Agent: behind Hermes for Perplex/Analyst/CFO-style work.
- Summarizer Agent: behind Hermes for Summarize workflows.
- HR Agent: behind Hermes for HR workflows.
- Reviewer/Critic: quality/hallucination/decision checks behind Hermes.
- Policy/Guardrail: approval, data boundary, side-effect checks.
- Memory/Journal: compact per-topic summaries and durable decision logs.

## Minimum policy fields for every registered topic

- `topic_id`
- `topic_name`
- `lane_type`
- `purpose`
- `skill_bindings`
- `default_responder`
- `silent_agents`
- `allowed_task_types`
- `forbidden_actions`
- `approval_rules`
- `delegation_rules`
- `audit_level`
- `board_policy`
- `watchdog_policy`
- `escalation_path`
- `data_sensitivity`
- `external_side_effects_policy`
- `human_readable_description`

## New topic onboarding process

1. Confirm topic title and ID.
2. Add documented-only policy entry.
3. Define purpose, lane type, default responder, silent agents.
4. Define topic skill bindings.
5. Define allowed/forbidden actions and approval rules.
6. Run 1–2 manual smoke tasks.
7. Enable passive audit if useful.
8. Enable board tracking for delegated tasks.
9. Only after stability, consider dry-run routing or soft enforcement.

Never:
- Apply `MM / topic 642` rules globally.
- Give Bud default responder status by default.
- Enable enforcement before documented behavior is validated.
- Allow external side effects without approval policy.

## Working layer recommendation

Do not load full histories into every answer. Build and use a compact registry:

`topic → purpose → skills → rules → artifacts → delegation policy → durable history summary`

Full chat history should be used only as source material during onboarding/audit or when a specific past decision must be verified.

## Missing context / next inventory work

1. Bud alignment completed: `correlation_id=topic-inventory-final-align-20260518` ACK confirmed `4 = Analyst` and `CFO = topic ID unknown`.
2. Confirm whether historical/Bud domain lanes are still active Telegram topics: `259 / Perplex`, `584 / Summarize`, `21 / SKILLS (2) / Music`, `488 / HR`.
3. Confirm current CFO topic/title if CFO remains an active Telegram topic; do not reuse `topic 4` for CFO.
4. Walk session history per topic and add durable decisions/artifacts only; avoid stale progress/noise.
5. Convert this markdown inventory into a machine-readable registry once the mapping is stable.

## First workflow recommendation

Recommended first Agentic Stack v2 pilot: Research & Briefing in Analyst/Perplex lane.

Why:
- Low risk.
- High business value.
- Exercises Hermes + Research + Reviewer + Memory.
- Uses Bud only for execution support.
- Does not require external side effects.
