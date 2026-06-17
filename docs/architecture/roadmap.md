# Roadmap

Горизонты реализации. Основная цель — **Phase 2**: Agent подключён к сообществу, видит задачи, формулирует новые, выполняет назначенные.

---

## Phase 0 — Foundation

| Что | Статус |
|---|---|
| Core Engine (API, Auth, Event Store) | ✅ Реализовано |
| Event Bus (pg_notify) | ✅ Реализовано |
| Монорепозиторий + CI/CD | ✅ Реализовано |

**Готовность:** можно запустить пустой инстанс с пресетом.

---

## Phase 1 — Core Modules

| Модуль | Статус |
|---|---|
| **Auth** (регистрация, JWT, боты, сессии) | ✅ Реализовано (Session entity, deterministic IDs, profile) |
| **Community** (создание, CRUD) | ✅ Реализовано (auto-join creator) |
| **Membership** (join, approve, leave) | ✅ Реализовано (deterministic member_id) |
| **Issues & Workspaces** (воркспейсы, задачи, доски, milestones, комментарии) | ✅ Реализовано (event-sourced, subscription-runner для системных комментариев, duplicate auto-close) |
| **Comments** | ✅ Реализовано (пользовательские + системные через SubscriptionRunner) |
| **Board View (Kanban)** | ✅ Реализовано (BoardView.vue, колонки, фильтры, кнопки смены статуса) |
| **Dark theme** | ✅ Реализовано (data-theme="dark", localStorage, FOUC-prevention) |
| **RBAC (Permission Registry)** | ✅ Реализовано (shared/permissions.py, require_permission, 18 permissions) |
| **E2E tests (Playwright)** | ✅ Реализовано (tribe_e2e DB, smoke test, CI workflow) |
| **Entity Store + Modifier Pattern** | ✅ Реализовано (entities table, ModifierBase, CommandProcessor) |
| **Frontend UI Kit** | ✅ Реализовано (AppButton, AppInput, AppCard, AppModal, FormField) |
| **Feedback → Issue** | ✅ Реализовано (FeedbackButton создаёт Issue, lazy Feedback workspace) |
| **Proposals** | ❌ Не реализовано |
| **Reputation** | ❌ Не реализовано |

**Готовность:** можно завести сообщество, управлять участниками, создавать воркспейсы и задачи, комментировать, отслеживать канбан-доски. Все GraphQL резолверы читают из EntityStore, пишут через CommandProcessor.

---

## Phase 1.5 — Planned Enhancements

| Что | Статус | Подход |
|---|---|---|
| **Notifications** | ❌ Запланировано | Event Subscription → Notification entity → pull по клику на 🔔 (см. [notifications.md](../modules/notifications.md)) |
| **Кастомные роли сообщества** | ❌ Запланировано | Community.custom_roles JSONB, PERMISSIONS на роль, UI для owner/admin |
| **Community MCP Server** | ✅ Реализовано | HTTP :3001, 11 MCP-инструментов (create_issue, update_issue, search_issues, get_issue, get_workspace, add_comment, get_board, get_member_reputation, notify, convert_request, reject_request), bot-token auth |
| **Issue detail: mention, markdown** | ❌ Запланировано | Рендеринг markdown в комментариях и описании, @username mentions |

---

## Phase 2 — Agent MVP ★

| Компонент | Статус |
|---|---|
| **Thin Coordinator** | 🟡 Частично (pg_notify → opencode serve) |
| **Community MCP Server** | ✅ Реализовано (см. Phase 1.5) |
| **Agent: `@classifier`** | 🟡 Частично (Request → Issue pipeline) |
| **Seed-боты как User(is_bot)** | 🟡 Заменить хардкод `agent_user_id` на реальных User'ов |
| **Agent: `@task-executor`** | ❌ Не подключён (ждёт IssueAssigned handler) |
| **Agent: `@self-healer`** | ❌ Не подключён (ждёт метрик) |
| **Skills** | 🟡 Частично (classifier.txt есть, остальные — нет) |

**Готовность:** Issue pipeline работает end-to-end. Остальные пайплайны — в следующей итерации Phase 2.

---

## Phase 3 — Bots & Management

| Что | Описание |
|---|---|
| **Seed-боты как User(is_bot)** | Заменить `agent_user_id` хардкод на User с `is_bot=true`, seed при старте Coordinator |
| **Панель управления ботами** | UI `/community/:id/bots`: создать, настроить, отключить бота |
| **Custom-боты** | Пользовательские AI-боты: свой prompt, модель, API-ключи per community |
| **RBAC роли** | ✅ Реализовано (permissions.py, require_permission, 18 permissions; кастомные роли — запланировано) |
| **Настройка Agent в сообществе** | Выбор ботов, их промптов и моделей через UI |

---

## Phase 4+ (опционально)

| Что | Когда |
|---|---|
| Внешние боты (Telegram, MAX, Discord) | После Phase 3 |
| Frontend (Landing SPA + Platform UI) | После Phase 3 |
| Auto-approve правил | После Phase 3 |
| Cross-preset расширение | После Phase 3 |
| Эволюция пресетов | После Phase 3 |
