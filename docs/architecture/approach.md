# Архитектурный подход

## Event Sourcing

Все изменения состояния в системе — события. Вместо хранения текущего состояния хранится последовательность событий, которые к нему привели.

### Где применяется

| Модуль | Пример события |
|---|---|
| Auth | `UserRegistered`, `UserProfileUpdated` |
| Membership | `MemberJoined`, `MemberApproved`, `MemberLeft` |
| Community | `CommunityCreated`, `CommunityUpdated` |
| Issues | `IssueCreated`, `IssueAssigned`, `IssueStatusChanged`, `WorkspaceCreated` |
| Requests | `RequestCreated`, `RequestAccepted`, `RequestConverted` |

### Схема

```
                ┌──────────────────────────────────────────┐
                │  GraphQL Query (EntityStore.get/list/    │
                │           find_one)                       │
                └────────────┬─────────────────────────────┘
                             │
                ┌────────────▼─────────────────────────────┐
                │  Entity Store (entities table)           │
                │  id | type | community_id | attributes   │
                └────────────▲─────────────────────────────┘
                             │ sync on save_events()
                             │
Command ──► Modifier.dispatch ──► Aggregate ──► Event ──► EventStore.save_events()
                                                             │
                                                    ┌────────▼────────┐
                                                    │  events table   │
                                                    │  (audit log)    │
                                                    └────────┬────────┘
                                                             │
                                                    pg_notify → subscribers
```

### Почему

- Полная аудит-логия — любое действие восстанавливается
- Возможность «откатить» состояние до любой точки
- События — источник для аналитики, репутации, revenue
- Entity Store — быстрый O(1) доступ к текущему состоянию

## Entity Store + Modifier Pattern

Текущая архитектура (мигрировано в 2025):

| Компонент | Роль |
|---|---|
| **EntityStore** | Единая таблица `entities` с jsonb attributes, GIN index. Чтение через `get(id)`, `list_by_type(type)`, `find_one(type, attr, value)` |
| **ModifierBase** | Абстрактный класс бизнес-логики. `handle(action, args, ctx)` — точка входа для команд |
| **CommandProcessor** | Реестр модификаторов. `dispatch("module_action", args, ctx)` → находит модификатор → вызывает handle |
| **ActionContext** | Контекст вызова: actor_id, community_id, pool, entity_store, command_processor |
| **EventStore** | Сохраняет события, синкает attributes в Entity Store, отправляет pg_notify |

### Флоу данных (пример: регистрация пользователя)

```
1. GraphQL Mutation → command_processor.dispatch("auth_register", {email, password, name}, ctx)
2. AuthModifier.handle("register", ...) → UserAggregate.create(email, password, name)
3. EventStore.save_events(user, ...) → INSERT INTO events + EntityStore.save("user", ...)
4. AuthModifier._auto_join_platform → cp.dispatch("membership_join", ...)
5. MembershipModifier.handle("join", ...) → MemberAggregate.create(...) → EventStore.save_events
6. GraphQL → возвращает AuthPayload (access_token, refresh_token, user_id)
```

### Cross-modifier dispatch

Модификаторы могут вызывать другие модификаторы через `ctx.command_processor.dispatch()`:

- `AuthModifier._register` → `membership_join` (авто-вступление в Platform)
- `CommunityModifier._create` → `membership_join` (создатель становится участником)

## DDD (Domain-Driven Design)

### Bounded Contexts

Каждый модуль — отдельный bounded context со своим языком и агрегатами.

```
┌────────────────────┐   ┌────────────────────┐
│  Auth              │   │  Membership         │
│  (пользователи)    │   │  (участники)        │
│  Язык: email,      │   │  Язык: заявка,      │
│  пароль, сессия    │   │  статус, роль       │
└────────┬───────────┘   └────────┬────────────┘
         │                        │
         │  Cross-modifier через  │
         │  CommandProcessor      │
         └────────────────────────┘
```

### Агрегаты

Каждый модуль содержит 1–3 агрегата:

```
User (Aggregate Root)
  ├── aggregate_id: str (deterministic_id("user", email))
  ├── name, email, avatar_url, bio
  ├── is_bot, bot_type
  └── events: UserRegistered, UserProfileUpdated

Member (Aggregate Root)
  ├── aggregate_id: str (member_id(user_id, community_id))
  ├── user_id, community_id, status
  └── events: MemberJoined, MemberApproved, MemberLeft

Community (Aggregate Root)
  ├── aggregate_id: str (ULID)
  ├── name, description, created_by
  └── events: CommunityCreated, CommunityUpdated
```

### Бизнес-правила в агрегате

Агрегат сам проверяет инварианты:
- Нельзя покинуть сообщество дважды (MemberLeft → status=EXITED)
- Нельзя аппрувить не-applicant
- Нельзя завершить не-in_progress таску

## CQRS

Command и Query разделены на уровне агрегатов:

- **Command** → Modifier → Aggregate → EventStore
- **Query** → EntityStore.get / list_by_type / find_one
- Read Model = Entity Store (единая таблица entities, обновляется при save_events)

## Стек

| Слой | Технология | Обоснование |
|---|---|---|
| **Backend** | Python 3.12 + FastAPI + Strawberry GraphQL | Async, типизация, экосистема |
| **Event Store** | PostgreSQL (events table, UNIQUE per aggregate+version) | Простота, транзакционность |
| **Entity Store** | PostgreSQL (entities table, GIN index на attributes) | Единое чтение для GraphQL |
| **Message Bus** | pg_notify | Встроено в PG, 0 зависимостей |
| **Frontend** | Vue 3 + Composition API + TypeScript + Vite | SPA, реактивность |
| **API** | GraphQL (Strawberry) | Гибкая загрузка данных под UI |

## Session

- Session хранится как строка в `entities` таблице (type="session").
- Не агрегат (без событий) — эфемерные данные, не требуют аудита.
- Поля: `user_id`, `refresh_token_hash`, `expires_at`, `revoked_at`.
- Refresh: валидация JWT → поиск сессии по hash → проверка revoked_at → обновление.

## Связь с пресетами

Пресет определяет, какие агрегаты активны. Если пресет CoLive — активны Member, Room, Booking, Duty. Если CoSettle — Member, Share, Plot, Infrastructure.
