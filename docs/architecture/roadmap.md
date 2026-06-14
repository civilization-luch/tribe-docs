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
| **Tasks & Projects** | ✅ Реализовано (community-scoped, is_default, project_id required) |
| **Requests** | ✅ Реализовано (7 событий, 9 mutation, community-scoped) |
| **Entity Store + Modifier Pattern** | ✅ Реализовано (entities table, ModifierBase, CommandProcessor) |
| **Frontend UI Kit** | ✅ Реализовано (AppButton, AppInput, AppCard, AppModal, FormField) |
| **Proposals** | ❌ Не реализовано |
| **Reputation** | ❌ Не реализовано |

**Готовность:** можно завести сообщество, управлять участниками, создавать задачи и проекты, отправлять запросы. Все GraphQL резолверы читают из EntityStore, пишут через CommandProcessor.

---

## Phase 2 — Agent MVP ★

| Компонент | Статус |
|---|---|
| **Requests** | ✅ Реализовано |
| **Thin Coordinator** | ✅ Реализовано (pg_notify → opencode serve) |
| **Community MCP Server** | ✅ Реализовано (10 tools, JSON-RPC + SSE) |
| **Opencode serve** | ✅ Настроено (:4096, MCP, agents) |
| **Agent: `@classifier`** | ✅ Pipeline 1 работает (Request → Task, dedup) |
| **Seed-боты как User(is_bot)** | 🟡 Заменить хардкод `agent_user_id` на реальных User'ов |
| **Agent: `@task-executor`** | ❌ Не подключён (ждёт TaskAssigned handler) |
| **Agent: `@self-healer`** | ❌ Не подключён (ждёт метрик) |
| **Skills** | 🟡 Частично (classifier.txt есть, остальные — нет) |

**Готовность:** Request → Task pipeline работает end-to-end. Остальные пайплайны — в следующей итерации Phase 2.

---

## Phase 3 — Bots & Management

| Что | Описание |
|---|---|
| **Seed-боты как User(is_bot)** | Заменить `agent_user_id` хардкод на User с `is_bot=true`, seed при старте Coordinator |
| **Панель управления ботами** | UI `/community/:id/bots`: создать, настроить, отключить бота |
| **Custom-боты** | Пользовательские AI-боты: свой prompt, модель, API-ключи per community |
| **RBAC роли** | admin / moderator / member / bot — на уровне Member |
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
