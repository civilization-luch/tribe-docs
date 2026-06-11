# Roadmap

Горизонты реализации. Основная цель — **Phase 2**: Agent подключён к сообществу, видит задачи, формулирует новые, выполняет назначенные.

---

## Phase 0 — Foundation

| Что | Зависимости |
|---|---|
| Core Engine (API, Auth, RBAC) | — |
| Event Store (PostgreSQL) | Core Engine |
| Event Bus (NATS / RabbitMQ) | Core Engine |
| Module Registry | Core Engine |
| Монорепозиторий + CI/CD | — |

**Готовность:** можно запустить пустой инстанс с пресетом.

---

## Phase 1 — Core Modules

| Модуль | События | Связи |
|---|---|---|
| **Membership** | регистрация, роли, заявки | — |
| **Tasks** | CRUD, assign, complete, проекции | Reputation |
| **Proposals** | создание, голосование, закрытие | — |
| **Reputation** | баллы за Task, откат за ошибки | Tasks |
| **Platform-сообщество** | CoGuild-пресет + Projects | всё выше |

**Готовность:** можно завести сообщество, создавать задачи, голосовать. Agent ещё не подключён.

---

## Phase 2 — Agent MVP ★

| Компонент | Описание |
|---|---|
| **Requests** | форма/бот → сырой запрос |
| **Agent Coordinator** | event listener, роутер пайплайнов |
| **Community MCP Server** | MCP-прослойка для Agent'а |
| **Pipeline 1: Request → Task** | LLM: classify, dedup, enrich |
| **Pipeline 2: Task Executor** | opencode + Community MCP |
| **Pipeline 3: Self-healing** | метрики → Task |
| **Opencode** | CLI-зависимость для анализа кода и PR |

**Готовность:** Agent подключён к сообществу. Видит задачи. Формулирует новые из Request. Выполняет назначенные Task. Создаёт self-healing Task по метрикам.

---

## Phase 3+ (опционально)

| Что | Когда |
|---|---|
| Frontend (Landing SPA + Platform UI) | После Phase 2 |
| Боты (Telegram, MAX) | После Phase 2 |
| Auto-approve правил | После Phase 2 |
| Cross-preset расширение | После Phase 2 |
| Эволюция пресетов | После Phase 2 |

---

## Решения до старта

| Решение | Варианты |
|---|---|
| Язык бэкенда | Go / TypeScript (NestJS) / Rust |
| Event Bus | NATS / RabbitMQ |
| LLM для Pipeline 1 | OpenAI / Claude / local |
| Opencode | CLI-зависимость (да/нет) |
