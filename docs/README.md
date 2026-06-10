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

| Пресет | Модули |
|---|---|
| **CoLive** | Membership, Billing, Bookings, Tasks, Treasury, Content, Proposals, Reputation, Documents |
| **CoSpace** | Membership, Schedule, Bookings, Billing, Content, Proposals, Reputation, Documents |
| **CoSettle** | Membership, Registry, Treasury, Proposals, Tasks, Content, Documents |
| **CoGuild** | Membership, LMS, Tasks, Billing, Content, Proposals, Reputation, Documents |
| **CoOper** | Membership, Registry, Billing, Treasury, Proposals, Content, Marketplace |

## Архитектура

- [Архитектура платформы](architecture.md)
- [Модель участников коллайвинга](coliving/members.md)
- [Дома как группы](coliving/groups.md)

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
