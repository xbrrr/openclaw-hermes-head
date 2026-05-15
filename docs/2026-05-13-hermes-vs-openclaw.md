# Hermes vs OpenClaw — ресёрч и безопасная схема теста

Дата: 2026-05-13

## Короткий вывод

Hermes стоит тестировать, но не мигрировать сразу. Лучший вариант: оставить OpenClaw как production-канал в Telegram и автопилот, а Hermes поднять отдельным лабораторным контуром на том же Mac mini без доступа к production Telegram-сессии и без копирования секретов целиком.

Главная причина интереса к Hermes: встроенная идея самоулучшения — память, поиск по прошлым сессиям, agent-managed skills, cron и периодическая рефлексия. Главная причина не спешить: у OpenClaw уже стабильно работает gateway, Telegram, memory stack, skills, cron и модель `openai-codex/gpt-5.5`.

## Проверенные источники

- `https://github.com/NousResearch/hermes-agent`
- `https://raw.githubusercontent.com/NousResearch/hermes-agent/main/README.md`
- `https://raw.githubusercontent.com/NousResearch/hermes-agent/main/website/docs/getting-started/installation.md`
- `https://raw.githubusercontent.com/NousResearch/hermes-agent/main/website/docs/user-guide/configuration.md`
- `https://raw.githubusercontent.com/NousResearch/hermes-agent/main/website/docs/user-guide/features/cron.md`
- `https://raw.githubusercontent.com/NousResearch/hermes-agent/main/website/docs/user-guide/features/memory.md`
- `https://raw.githubusercontent.com/NousResearch/hermes-agent/main/website/docs/user-guide/features/skills.md`
- `https://docs.openclaw.ai`
- локальный OpenClaw status на Mac mini

Важно: первичный Perplexity-ответ содержал слишком уверенные сравнительные claims из вторичных источников. В итог ниже включены только пункты, подтверждённые README/docs/source-level страницами или локальной проверкой.

## Что такое Hermes

Hermes Agent от Nous Research — self-improving AI agent. По README заявляет:

- работу через CLI и messaging gateway;
- Telegram, Discord, Slack, WhatsApp, Signal, Email;
- voice memo transcription;
- persistent memory;
- session search через SQLite/FTS5;
- agent-created / agent-managed skills;
- cron scheduler;
- subagents / delegation;
- разные terminal backends: local, Docker, SSH, Modal, Daytona, Vercel Sandbox, Singularity;
- миграцию из OpenClaw командой `hermes claw migrate`.

Установка по docs:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Layout per-user:

- code: `~/.hermes/hermes-agent/`
- binary: `~/.local/bin/hermes`
- data: `~/.hermes/`

Config layout:

```text
~/.hermes/
├── config.yaml
├── .env
├── auth.json
├── SOUL.md
├── memories/
├── skills/
├── cron/
├── sessions/
└── logs/
```

## Что сейчас у OpenClaw на Mac mini

Локальная проверка:

- OpenClaw gateway жив.
- Один listener:
  - `127.0.0.1:18789`
  - `[::1]:18789`
- LaunchAgent: `~/Library/LaunchAgents/ai.openclaw.gateway.plist`
- команда сервиса: `/opt/homebrew/opt/node@22/bin/node /opt/homebrew/lib/node_modules/openclaw/dist/index.js gateway --port 18789`
- working dir: `~/.openclaw`
- probe: ok.

То есть OpenClaw сейчас production-ready и не должен трогаться в ходе Hermes-теста.

## Сравнение OpenClaw vs Hermes

| Критерий | OpenClaw | Hermes |
|---|---|---|
| Основная роль | Gateway/control plane для каналов, агентов, сессий, tools, skills | Самоулучшающийся агент + gateway + cron + skills |
| Текущий статус у нас | Уже production в Telegram | Не установлен |
| Gateway | Работает на `127.0.0.1:18789` | Нужно ставить отдельно; данные в `~/.hermes` |
| Telegram | Уже настроен и проверен | Есть messaging gateway, но подключать production-сессию нельзя на первом этапе |
| Память | `SOUL.md`, `USER.md`, `MEMORY.md`, daily journals, skills | `~/.hermes/memories/MEMORY.md`, `USER.md`, `SOUL.md`; лимиты компактнее |
| Skills | Уже много локальных skills, включая кастомные | Совместим с AgentSkills-стандартом; умеет agent-managed skills |
| Самоулучшение | В основном через ручную правку skills/memory и cron-задачи | Заявлено как встроенный loop: memory + session search + skill creation/update |
| Cron | Уже используется в OpenClaw | Встроенный cron с доставкой в платформы, no-agent jobs, chaining через `context_from` |
| Изоляция команд | OpenClaw tools + approvals + host/node | Hermes terminal backends: local/docker/ssh/modal/daytona/vercel/singularity |
| Миграция | Не нужна сейчас | Есть `hermes claw migrate`, но для нас только `--dry-run` / user-data, не full |
| Риск | Низкий, так как уже настроен | Риск в installer, gateway, Telegram/session/secrets, конфликте сервисов |

## Где Hermes сильнее потенциально

1. **Самоулучшение как продуктовая идея**
   - agent-managed skills;
   - навыки могут создаваться/правиться после сложных задач;
   - память и session search встроены как core features.

2. **Cron-модель**
   - natural-language cron;
   - доставка в origin/local/Telegram/Discord/etc.;
   - no-agent режим для дешёвых watchdog-задач;
   - chaining через `context_from`.

3. **Сандбоксы**
   - Docker/SSH/Modal/Daytona/Vercel/Singularity backends;
   - можно разделять локальный агент и место исполнения.

4. **Миграционный мост из OpenClaw**
   - есть команда `hermes claw migrate`;
   - импортирует SOUL, memories, skills, settings, часть secrets по allowlist.
   - Но это использовать только dry-run/частично.

## Где Hermes слабее/опаснее для нас сейчас

1. **Не проверен в нашей production-среде.**
   OpenClaw уже живёт в Telegram и пережил настройки/апдейты. Hermes пока ноль.

2. **Риск Telegram/session conflicts.**
   Нельзя подключать оба runtime к одному и тому же production-каналу/сессии без отдельного теста. Особенно если будет Telethon/user-session или общий bot token.

3. **Миграция может скопировать лишнее.**
   `hermes claw migrate` умеет тащить API keys и messaging settings. Для первого теста это опасно. Только dry-run или `--preset user-data`.

4. **Self-improvement требует предохранителей.**
   Автоматическое создание/правка skills полезны, но без review может закреплять ошибки, раздувать память или менять поведение не туда.

## Можно ли запускать Hermes вместе с OpenClaw на том же Mac mini

Да, вероятно можно, если соблюдать изоляцию:

- OpenClaw остаётся на `127.0.0.1:18789`.
- Hermes ставится в `~/.hermes`, не в `~/.openclaw`.
- Не использовать тот же Telegram bot token / session на первом этапе.
- Не включать Hermes gateway как production LaunchAgent до проверки портов.
- Не запускать `hermes claw migrate` в full mode.
- До/после запуска Hermes проверять:

```bash
openclaw gateway status
lsof -nP -iTCP -sTCP:LISTEN | grep -E '18789|hermes|python|node'
```

Возможные конфликты:

- порт gateway/dashboard, если Hermes выберет занятый порт;
- Telegram bot token/session;
- общие env/API keys;
- shared working directory;
- фоновые cron-задачи, если обе системы начнут делать одно и то же;
- browser/Node/Playwright/ffmpeg зависимости — скорее не критично, но installer может менять PATH.

## Рекомендованный тестовый контур

### Фаза 0 — без установки

- держать этот отчёт как baseline;
- не трогать OpenClaw;
- проверить installer script через static review перед запуском.

### Фаза 1 — установка без gateway

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh -o /tmp/hermes-install.sh
# review script first
bash /tmp/hermes-install.sh
```

После:

```bash
command -v hermes
hermes doctor
hermes config
```

### Фаза 2 — локальный CLI только

- настроить не production-модель/ключ минимального риска;
- запустить `hermes` в CLI;
- дать read-only копии `SOUL.md`, `USER.md`, `MEMORY.md` в sandbox;
- попросить Hermes сделать краткую сводку предпочтений пользователя;
- проверить стиль, приватность, дисциплину памяти.

### Фаза 3 — skills portability

Не импортировать всё. Проверить 2–3 низкорисковых skills:

- weather;
- markdown/web extraction;
- perplex-like search;
- простой memory write в sandbox.

### Фаза 4 — cron/self-improvement lab

Создать отдельный sandbox cron, который не пишет в production files напрямую.

## Как реализовать «самоанализ в простое» безопасно

Не давать агенту бесконечно сам себя менять. Делать через очередь предложений.

### Правильная архитектура

1. **Наблюдение**
   - cron собирает последние события: ошибки, незавершённые задачи, частые правки, user corrections.

2. **Рефлексия**
   - агент пишет короткий отчёт:
     - что повторялось;
     - где была ошибка;
     - какой skill/memory/rule стоит поправить;
     - риск изменения.

3. **Предложение патча**
   - не применять сразу;
   - сохранять в `docs/self-improve/proposals/YYYY-MM-DD.md`.

4. **Авто-применение только low-risk**
   Разрешить автоматически:
   - добавить заметку в daily memory;
   - создать черновик skill note;
   - поправить typo/устаревший путь в skill после проверки.

   Запретить без подтверждения:
   - менять gateway/runtime config;
   - менять Telegram/session/secrets;
   - менять global model;
   - ставить внешние зависимости;
   - удалять файлы;
   - включать новые cron/gateway integrations.

5. **Верификация**
   Любой applied patch должен иметь проверку:
   - syntax check;
   - grep/inspection;
   - small smoke test;
   - запись в daily journal.

### OpenClaw-версия без Hermes

У нас это можно сделать уже сейчас в OpenClaw:

- daily/weekly cron task;
- читает последние `memory/YYYY-MM-DD.md`, ошибки и изменённые skills;
- пишет proposals;
- только low-risk правки применяет сам;
- всё остальное просит подтвердить.

Пример задачи:

```text
Каждый день в 10:30 MSK проверь последние 48 часов memory/logs/skills. Найди 1–3 повторяющиеся ошибки или устаревшие инструкции. Если фикс безопасный и локальный — предложи patch и проверь. Если риск выше low — только напиши proposal. Не меняй gateway, secrets, Telegram и model config без подтверждения.
```

### Hermes-версия

Hermes потенциально лучше подходит для этого, потому что в нём уже есть:

- cron jobs;
- agent-managed skills;
- memory tool;
- session search;
- skills as procedural memory.

Но включать это надо в sandbox, а не на production памяти.

## Рекомендация

1. Не мигрировать с OpenClaw.
2. Поднять Hermes как lab-контур.
3. Не подключать production Telegram.
4. Не запускать full migration.
5. Сначала сравнить 5 задач:
   - веб-ресёрч;
   - маленький coding patch;
   - memory write;
   - cron summary;
   - skill creation/update.
6. Если Hermes реально лучше в 3/5 — думать о частичной интеграции.

Мой текущий вывод: Hermes интересен именно как «лаборатория самоулучшения» и потенциальный execution worker. OpenClaw пока должен остаться главным production gateway.
