# OpenClaw ↔ Hermes Agent — архитектура сосуществования

Дата: 2026-05-15
Контекст: Mac mini Mike, Telegram topic `Web Admin`, OpenClaw уже работает как основной Telegram-исполнитель.

## Целевая модель ролей

- **Hermes Agent = голова**
  - самообучающийся агент;
  - долгие цели, память, профили, state.db;
  - Kanban/cron/checkpoints/MCP;
  - генерация новых навыков и улучшение процедур;
  - планирование и стратегическое разложение задач.

- **OpenClaw = исполнитель**
  - стабильная Telegram-поверхность;
  - доступ к уже настроенным чатам, скиллам, памяти, инструментам;
  - конкретные действия: сообщения, проверки, файлы, субагенты, OpenClaw Gateway;
  - короткий операционный контур: получил задачу → сделал → проверил → отчитался.

## Вывод из поста human20/220

Пост фиксирует практическую разницу:

- OpenClaw — «обжитый кабинет»: SOUL.md, USER.md, AGENTS.md, CHATS.md, рабочие привычки, плотный личный контекст.
- Hermes — более модульный конструктор: config.yaml, providers, toolsets, gateway, state.db, skills, cron, profiles, sessions, MCP.
- OpenClaw быстрее даёт ощущение живого личного агента, но исторические слои могут усложнять сопровождение.
- Hermes проще переносить, чинить, объяснять и расширять.
- Для уже существующего OpenClaw-сетапа оптимальный путь — не резкий переезд, а постепенный перенос функций на Hermes-рельсы.

## Рекомендуемая схема взаимодействия

### Этап 1 — безопасное параллельное сосуществование

Hermes установлен локально, но не получает Telegram gateway и не трогает OpenClaw secrets.

Поток:

1. Пользователь пишет в Telegram → OpenClaw принимает.
2. OpenClaw классифицирует задачу.
3. Для сложной/долгой задачи OpenClaw может вызвать Hermes через CLI/oneshot или отдельный backend-процесс.
4. Hermes возвращает план/решение/стратегию.
5. OpenClaw исполняет конкретные действия и отчитывается в Telegram.

Плюс: нет конфликта ботов, токенов, портов и gateway.

### Этап 2 — Hermes как стратегический backend

Добавить OpenClaw-скилл/адаптер `hermes-head`:

- принимает цель;
- отправляет её в Hermes;
- получает план/декомпозицию;
- сохраняет артефакты в workspace;
- исполняет через OpenClaw tools/subagents;
- возвращает результат пользователю.

Hermes не пишет наружу напрямую, пока мы не проверили безопасность.

### Этап 3 — отдельный Hermes Telegram bot, если нужен

Только после отдельного решения:

- отдельный Telegram bot token;
- отдельные allowlists;
- отдельный topic/channel;
- без shared token с OpenClaw;
- без автоматической миграции OpenClaw secrets;
- healthcheck двух gateway отдельно.

## Запреты по умолчанию

- Не запускать `scripts/install.sh` и `setup-hermes.sh`, потому что scan нашёл remote install patterns.
- Не запускать `hermes claw migrate` без отдельного подтверждения: там может быть перенос API keys и messaging settings.
- Не запускать Hermes gateway поверх текущего OpenClaw Telegram token.
- Не отдавать Hermes полный доступ к OpenClaw workspace без sandbox/approval.

## Установка на Mac mini — факт

Установлено безопасным путём:

- repo: `/Users/xbr/.local/src/hermes-agent`
- source: `https://github.com/NousResearch/hermes-agent.git`
- commit: `77276070f5a1302908456734f2a5bdfe790260de`
- command: `/Users/xbr/.local/bin/hermes`
- wrapper: `/Users/xbr/.local/bin/hermes-local`
- Python: Homebrew `python3.13`
- uv: Homebrew `uv`
- venv: `/Users/xbr/.local/src/hermes-agent/.venv`
- extras: `cron`, `cli`, `mcp`, `honcho`, `messaging`

Не выполнено намеренно:

- upstream `install.sh` не запускался;
- `setup-hermes.sh` не запускался;
- `hermes setup` не запускался;
- `hermes claw migrate` не запускался;
- Hermes gateway не стартовал;
- secrets из OpenClaw не переносились.

## Проверка

- `hermes version` → Hermes Agent v0.13.0 (2026.5.7), Python 3.13.13.
- `hermes doctor`:
  - core packages OK;
  - python-telegram-bot OK;
  - discord.py OK;
  - `~/.hermes/.env` создан без секретов;
  - `~/.local/bin/hermes` указывает на правильный venv entrypoint;
  - нет API key/provider auth — ожидаемо;
  - Hermes gateway не слушает порты.
- `lsof` показал только OpenClaw gateway на `127.0.0.1:18789` / `[::1]:18789`.

## ClawGuard decision

Primary module: `cg-antivirus-surface`.

Decision: **sanitize / staged allow**.

Reason:

- репозиторий полезен, но автоматический installer unsafe для прямого запуска;
- scan нашёл critical/high паттерны: `rm -rf` в тестах/guardrail-коде, `curl | bash` в install scripts, `subprocess shell=True` в отдельных инструментах.

Evidence:

- `scripts/install.sh`, `setup-hermes.sh`, `hermes_cli/*` содержат remote install command references;
- реальные опасные команды в основном находятся в approval/guardrail/test contexts, но требуют ручной осторожности;
- установка выполнена без этих скриптов: clone → uv venv → uv pip install editable extras.

Next action:

1. Настроить provider auth для Hermes отдельным способом (`hermes auth` или ручной `.env`).
2. Сделать OpenClaw skill-adapter `hermes-head` для безопасного вызова Hermes как головы.
3. Только потом решать, нужен ли отдельный Hermes gateway в Telegram.

## Детализация варианта B — Claude Code/OAuth внутри Hermes, безопасный режим

Контекст: пользователь хочет, чтобы Hermes был стратегическим/self-learning слоем и мог ставить задачи OpenClaw. Вариант B означает: Hermes может использовать Claude Code/OAuth как мыслительный/кодовый усилитель, но не как бесконтрольный автономный воркер.

### Цель

Дать Hermes доступ к Claude для сложных задач:

- анализ архитектуры;
- генерация планов;
- ревью кода/скиллов;
- улучшение процедур;
- исследование ошибок;
- подготовка задач для OpenClaw.

При этом снизить риск блокировки Claude-аккаунта и не нарушать границы использования.

### Главный принцип

Claude Code/OAuth не должен быть постоянным фоновым двигателем самообучения.

Правильная роль:

- Hermes решает, что нужна глубокая модель;
- создаёт конкретный запрос с лимитом;
- вызывает Claude через контролируемый wrapper;
- сохраняет результат;
- отдаёт OpenClaw задачу/вывод;
- не запускает бесконечные циклы и массовую параллельность.

### Контур задач Hermes → OpenClaw

1. Hermes создаёт `task_request`:
   - цель;
   - причина;
   - ожидаемый результат;
   - риск;
   - требуемые инструменты;
   - лимит времени/стоимости;
   - нужен ли Claude.
2. OpenClaw принимает задачу как gatekeeper.
3. OpenClaw классифицирует:
   - `low`: чтение, анализ, черновик, локальные безопасные файлы;
   - `medium`: изменение файлов, настройка cron, запуск долгих команд;
   - `high`: секреты, токены, внешние сообщения, миграции, gateway/restart, платежи.
4. OpenClaw исполняет low сам; medium/high — только с approval пользователя.
5. Результат возвращается Hermes для обучения/памяти.

### Claude Code/OAuth guardrails

Обязательные ограничения:

- один активный Claude-запрос от Hermes за раз;
- дневной лимит запросов;
- недельный лимит запросов;
- cooldown между запросами;
- запрет бесконечных циклов;
- запрет auto-retry без backoff;
- запрет обхода rate limits;
- запрет использования нескольких аккаунтов для расширения лимитов;
- запрет массового фонового скрейпинга/генерации;
- запрет передачи Claude секретов без явного allowlist;
- полный лог: кто запросил, зачем, какой prompt, какой результат, сколько длилось.

### Рекомендуемые лимиты для старта

Консервативный режим:

- максимум 10 Hermes→Claude задач в день;
- максимум 1 задача одновременно;
- минимум 2–5 минут паузы между задачами;
- максимум 20–30 минут на одну задачу;
- no auto-loop: не больше 1 self-improvement шага без подтверждения;
- ночной quiet window без тяжёлых запусков.

После 1–2 недель наблюдения лимиты можно поднять.

### Безопасная техническая схема

```
Telegram topic
  ├─ Hermes Bot: стратег / обучение / постановка задач
  └─ OpenClaw Bot: gatekeeper / execution / verification

Hermes
  └─ hermes-claude-wrapper
       ├─ проверка лимитов
       ├─ фильтр секретов
       ├─ классификация риска
       ├─ запуск Claude Code/OAuth
       ├─ таймаут
       └─ запись audit log

OpenClaw
  ├─ принимает task_request
  ├─ проверяет риск
  ├─ выполняет или просит approval
  └─ возвращает result_report
```

### Почему это безопаснее прямого подключения

Опасная схема:

- Hermes сам бесконечно вызывает Claude;
- сам меняет файлы;
- сам запускает команды;
- сам отправляет сообщения;
- сам обучается без лимитов.

Безопасная схема:

- Hermes может думать и предлагать;
- Claude вызывается редко и осмысленно;
- OpenClaw исполняет и проверяет;
- пользователь подтверждает рискованные действия;
- лимиты защищают аккаунт и бюджет.

### Что НЕ делать

- Не использовать Claude Pro/Max как серверный API для постоянного автономного агента.
- Не пытаться обходить лимиты подписки.
- Не распараллеливать Claude Code через несколько процессов.
- Не ставить Claude на бесконечный cron самообучения.
- Не давать Hermes прямой доступ к OpenClaw secrets.
- Не смешивать Telegram token Hermes и OpenClaw.

### Лучший путь внедрения

Фаза 1: dry-run
- Hermes генерирует задачи только в файл/чат.
- OpenClaw вручную/полуавтоматически исполняет.
- Claude не подключён напрямую.

Фаза 2: Claude wrapper
- Добавить wrapper с лимитами и логами.
- Разрешить только advisory-запросы: планы, ревью, анализ.
- Запретить изменение файлов Claude напрямую.

Фаза 3: controlled execution
- Hermes может ставить low-risk задачи OpenClaw.
- OpenClaw исполняет только после классификации.
- Medium/high требуют approval.

Фаза 4: learning loop
- Hermes анализирует результаты.
- Обновляет свою память/правила.
- Self-improvement только через pull-request/patch/review, не напрямую в runtime.

### Рекомендация

Вариант B допустим, если Claude Code/OAuth используется как ограниченный усилитель мышления, а не как бесконечный backend. Для production-режима лучше позже перейти на Anthropic API с отдельным workspace/key/budget, но B можно безопасно протестировать через wrapper и строгие лимиты.
