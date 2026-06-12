# Tribe — Документация

Платформа-конструктор для самоуправляемых сообществ.

## Продукты (пресеты)

Каждый продукт — это пресет (набор модулей с настройками по умолчанию).

- [CoLive](../VISION_CoLive.md) — коливинг
- [CoSpace](../VISION_CoSpace.md) — пространства
- [CoSettle](../VISION_CoSettle.md) — поселения
- [CoGuild](../VISION_CoGuild.md) — гильдии
- [CoOper](../VISION_CoOper.md) — кооперативы
- [Tribe](../VISION.md) — общее видение

| Пресет | Community модули | Premium модули |
|---|---|---|
| **CoLive** | Membership, Tasks, Content, Proposals, Reputation, Documents | Billing, Bookings, Treasury |
| **CoSpace** | Membership, Schedule, Content, Proposals, Reputation, Documents | Bookings, Billing |
| **CoSettle** | Membership, Proposals, Tasks, Content, Documents | Registry, Treasury |
| **CoGuild** | Membership, Tasks, Content, Proposals, Reputation, Documents | LMS, Billing |
| **CoOper** | Membership, Proposals, Content | Registry, Billing, Treasury, Marketplace |

## Лицензирование модулей

| Уровень | Модули | Репозиторий |
|---|---|---|
| **Community** (OSS) | Membership, Tasks, Content, Proposals, Reputation, Documents, Schedule | `github.com/tribe/core` (public) |
| **Premium** | Billing, Bookings, Treasury, LMS, Marketplace, Revenue, Registry | `github.com/tribe/premium` (private) |

На старте — монорепозиторий (`tribe/`), после обкатки — разделение на core + premium.

## Архитектура

- [Архитектура платформы](architecture/index.md)
- [Архитектурный подход — Event Sourcing, DDD, CQRS, стек](architecture/approach.md)
- [Модель участников коллайвинга](coliving/members.md)
- [Дома как группы](coliving/groups.md)

## Приложения

- [Tribe Landing (маркетинговый SPA)](apps/landing.md)
- [Tribe Platform (основное SPA)](apps/platform.md)

## Frontend

- [Стек и структура](frontend/README.md)
- [UI Kit — компоненты, конвенции, план развития](frontend/ui-kit.md)
- [Маршруты (screen map)](frontend/routing.md)
- [Лейауты](frontend/layouts.md)

## Модули

### Ядро
- [Membership](modules/membership.md) — управление участниками, заявки, статусы
- [Billing](modules/billing.md) — платежи, подписки, сплит
- [Bookings](modules/bookings.md) — бронирование комнат и пространств
- [Tasks](modules/tasks.md) — дежурства, обязанности, напоминания
- [Treasury](modules/treasury.md) — казна, бюджеты, прозрачность
- [Content](modules/content.md) — афиша, объявления, обсуждения
- [Proposals](modules/proposals.md) — голосования, предложения
- [Reputation](modules/reputation.md) — рейтинг, уровни доверия
- [Documents](modules/documents.md) — договоры, акты, правила

### Сервисы (монетизация)
- [LMS](modules/lms.md) — курсы, обучение, сертификаты
- [Marketplace](modules/marketplace.md) — аренда комнат, услуги резидентов
- [Revenue](modules/revenue.md) — распределение доходов, выплаты участникам

### Вторая очередь
- Gamification — квесты, уровни, достижения
- Groups — комнаты, команды внутри сообщества
- Federation — объединение сообществ
