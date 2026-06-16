# Модуль Notifications

> **Статус: запланировано. Phase 1.5.**

Уведомления о событиях в сообществе: назначенные задачи, комментарии, смена статуса, приглашения.

## Концепция

**Pull-модель**: события создают Notification entity через SubscriptionRunner, фронтенд забирает при клике на колокольчик.

```
IssueAssigned ──→ SubscriptionRunner ──→ Notification.created
CommentCreated ──→ SubscriptionRunner ──→ Notification.created
MemberJoined   ──→ SubscriptionRunner ──→ Notification.created
IssueStatusChanged → SubscriptionRunner → Notification.created
```

Позже (Phase 3) — переход на WebSocket для real-time доставки без изменения агрегата.

## Сущности

### Notification

| Поле | Описание |
|---|---|
| `id` | ULID |
| `user_id` | Кому показывать |
| `type` | Тип: `assigned`, `comment`, `status_changed`, `invited`, `linked` |
| `title` | "You were assigned to Fix login" |
| `link_id` | ID связанной сущности (issue_id, member_id, community_id) |
| `read` | Прочитано (false по умолчанию) |
| `community_id` | ID сообщества |
| `created_at` | Дата создания |

## Events

| Событие | Описание |
|---|---|
| `NotificationCreated` | Уведомление создано |
| `NotificationRead` | Помечено как прочитанное |

## Commands

| Команда | Описание |
|---|---|
| `MarkNotificationRead` | Пометить одно уведомление прочитанным |
| `MarkAllRead` | Пометить все уведомления пользователя прочитанными |

## Типы уведомлений

| Тип | Триггер (событие) | Кому | Пример текста |
|---|---|---|---|
| `assigned` | `IssueAssigned` | `assignee_id` | "You were assigned to Fix login bug" |
| `comment` | `CommentCreated` | автор issue + `assignee_id` | "New comment on Fix login bug" |
| `status_changed` | `IssueStatusChanged` | `assignee_id` | "Fix login bug → in_progress" |
| `invited` | `MemberJoined` (approve) | `user_id` вступившего | "You joined E2E Community" |
| `linked` | `IssueLinked` | `assignee_id` | "Write tests was linked as blocks" |

## GraphQL API

```graphql
type NotificationType {
  id: String!
  type: String!
  title: String!
  linkId: String
  read: Boolean!
  createdAt: String!
}

type Query {
  notifications(userId: String!, unreadOnly: Boolean): [NotificationType!]!
}

type Mutation {
  markNotificationRead(notificationId: String!): NotificationType!
  markAllNotificationsRead: Boolean!
}
```

## Реализация

| Компонент | Что |
|---|---|
| `issues/notification_models.py` | NEW — Notification агрегат + события |
| `issues/modifier.py` | ADD — `create_notification`, `mark_read` |
| `issues/modifier.py` subscriptions | ADD — handler'ы на IssueAssigned, CommentCreated, IssueStatusChanged, IssueLinked, MemberJoined |
| `issues/graphql.py` | ADD — `NotificationType`, query `notifications`, mutation `markNotificationRead` |
| Frontend | `NotificationBell.vue` — колокольчик + дропдаун + бейдж |

## Roadmap

- **Phase 1.5**: Pull-модель (Event → Notification entity → query по клику)
- **Phase 3**: WebSocket push (Strawberry subscription), кеширование на клиенте, offline-поддержка
