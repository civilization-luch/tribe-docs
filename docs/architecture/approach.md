# Архитектурный подход

## Event Sourcing

Все изменения состояния в системе — события. Вместо хранения текущего состояния хранится последовательность событий, которые к нему привели.

### Где применяется

| Модуль | Пример события |
|---|---|
| Membership | `MemberJoined`, `MemberStatusChanged`, `MemberExited` |
| Billing | `PaymentReceived`, `SubscriptionCreated`, `InvoiceGenerated` |
| Treasury | `FundDeposited`, `FundWithdrawn`, `BudgetApproved` |
| Tasks | `TaskCreated`, `TaskAssigned`, `TaskCompleted` |
| Proposals | `ProposalCreated`, `VoteCasted`, `ProposalClosed` |
| Reputation | `ScoreChanged`, `BadgeAwarded` |

### Почему

- Полная аудит-логия — любое действие восстанавливается
- Возможность «откатить» состояние до любой точки
- События — источник для Revenue (подсчёт долей), репутации, аналитики
- Асинхронность: модули реагируют на события друг друга

### Схема

```
Command → Aggregate → Event → Event Store → Projection → Read Model
                                              ↓
                                         Event Bus → другие модули
```

## DDD (Domain-Driven Design)

### Bounded Contexts

Каждый модуль — отдельный bounded context со своим языком и хранилищем.

```
┌────────────────────┐   ┌────────────────────┐
│  Membership        │   │  Billing            │
│  (участники)       │   │  (платежи)          │
│  Язык: резидент,   │   │  Язык: подписка,    │
│  заявка, статус    │   │  счёт, сплит        │
└────────┬───────────┘   └────────┬────────────┘
         │                        │
         │     События через Bus  │
         └────────────────────────┘
```

### Агрегаты

Каждый модуль содержит 2–5 агрегатов. Пример для Membership:

```
Member (Aggregate Root)
  ├── memberId: UUID
  ├── status: MemberStatus (applicant / probation / resident / suspended / exited)
  ├── profile: Profile
  ├── applications: Application[]
  └── events: MemberEvent[]
```

### Бизнес-правила в агрегате

Агрегат сам проверяет инварианты:

- Нельзя проголосовать, если статус не `resident`
- Нельзя выйти из сообщества с долгом
- Нельзя утвердить бюджет без кворума

## CQRS

Command и Query разделены на уровне агрегатов:

- **Command** — пишет в Event Store (Membership::join, Billing::pay)
- **Query** — читает из Read Model (список участников, баланс)
- Read Model обновляется проекциями из событий

## Стек

| Слой | Технология | Обоснование |
|---|---|---|
| **Backend** | Go / Rust / TypeScript (NestJS) | Производительность + типизация |
| **Event Store** | PostgreSQL (events table) или EventStoreDB | На старте PG, с ростом — ESDB |
| **Message Bus** | RabbitMQ / NATS | Асинхронная связь модулей |
| **Read Model** | PostgreSQL | Материализованные проекции |
| **Frontend** | React / Vue / Solid | SPA, динамическая навигация |
| **API** | GraphQL | Гибкая загрузка данных под UI |

## Флоу данных (пример: оплата взноса)

```
1. UI → POST /billing/pay  (Command)
2. Billing Aggregate → валидация
3. Событие PaymentReceived → Event Store
4. Event Bus → Reputation (+баллы), Treasury (+баланс), Billing Projection
5. Read Model обновлён → UI получает новый баланс
```

## Связь с пресетами

Пресет определяет, какие агрегаты активны. Если пресет CoLive — активны Member, Room, Booking, Duty. Если CoSettle — Member, Share, Plot, Infrastructure.
