# Модуль Invitations

Приглашения пользователей в сообщества. Поддержка двух режимов: по `userId`
(конкретному пользователю) и по ссылке (code-based, для незарегистрированных).

## Сущности

- **Invitation** — агрегат приглашения. ID = ULID. Хранится в Event Store и Entity Store (type=`"invitation"`, `community_id` скоупирован).
- **События**: `InvitationCreated`, `InvitationAccepted`, `InvitationDeclined`.

### Invitation aggregate

| Поле | Тип | Описание |
|---|---|---|
| `community_id` | `str` | Сообщество, в которое приглашают |
| `invited_user_id` | `str\|None` | Кому (null — для приглашения по ссылке) |
| `invited_by` | `str` | Кто создал приглашение (actor_id) |
| `status` | `InvitationStatus` | `pending` / `accepted` / `declined` |
| `role` | `str` | Роль при вступлении: `member`, `observer`, `admin` |
| `code` | `str` | Уникальный ULID-код (только для link-приглашений) |
| `expires_at` | `str\|None` | ISO-дата истечения (по умолчанию +7 дней) |
| `created_at` | `datetime` | Время создания |

## Actions (MembershipModifier)

| Action | События | Permission | Описание |
|---|---|---|---|
| `create_invitation` | `InvitationCreated` | `invitation.create` | Пригласить по userId. Проверки: user существует, не уже member/applicant, нет pending приглашения |
| `accept_invitation` | `InvitationAccepted` | — (принимает invited_user) | Принять по invitation_id. Проверка: `invited_user_id == actor_id`. Создаёт активного Member через `_join` |
| `decline_invitation` | `InvitationDeclined` | — (отклоняет invited_user) | Отклонить по invitation_id. Проверка: `invited_user_id == actor_id` |
| `create_invitation_link` | `InvitationCreated` | `invitation.create` | Создать приглашение по ссылке. Генерирует ULID-код. `invited_user_id = None`. `expires_at` по умолчанию +7 дней |
| `accept_invitation_by_code` | `InvitationAccepted` | — (любой авторизованный) | Принять по коду ссылки. Поиск через `find_one("invitation", "code", code)`. Проверки: status=pending, не expired. Если `invited_user_id` задан — сверяет с actor. Создаёт активного Member |

### Permission

```
invitation.create: ["owner", "admin"]
```

Создавать приглашения могут только owner и admin сообщества.

## Жизненный цикл

### Поток А: приглашение по userId

```
1. Owner/Admin → createInvitation(communityId, invitedUserId, role)
2. Система создаёт Invitation (status=pending)
3. Приглашённый видит его в myInvitations
4. Приглашённый → acceptInvitation(invitationId)
   → Invitation.status = accepted
   → Member создаётся с ролью из приглашения (auto_approve=True)
```

### Поток Б: приглашение по ссылке (зарегистрированный пользователь)

```
1. Owner/Admin → createInvitationLink(communityId, role) → возвращает code
2. Владелец копирует URL: {origin}/invite/{code}
3. Пользователь переходит по ссылке → PublicInvite.vue
   → invitationByCode(code) показывает community, role, expiry
4. Пользователь нажимает Accept → acceptInvitationByCode(code)
   → Invitation.status = accepted
   → Member создаётся
   → Редирект в сообщество
```

### Поток В: приглашение по ссылке (незарегистрированный)

```
1–2. Те же
3. Незарегистрированный открывает /invite/{code}
   → invitationByCode(code) показывает информацию
   → Кнопка "Register to join" → /register?invite={code}
4. Пользователь регистрируется
5. После регистрации фронтенд вызывает acceptInvitationByCode(code)
   → Приглашение принято
   → Редирект в сообщество
```

## GraphQL API

### Типы

```graphql
type InvitationType {
  id: String!
  communityId: String!
  invitedUserId: String       # null для link-приглашений
  invitedBy: String!
  status: String!             # "pending" | "accepted" | "declined"
  role: String!               # "member" | "observer" | "admin"
  code: String                # только для link-приглашений
  expiresAt: String           # ISO datetime
  createdAt: String
}
```

### Inputs

```graphql
input CreateInvitationInput {
  communityId: String!
  invitedUserId: String!
  role: String! = "member"
}

input CreateInvitationLinkInput {
  communityId: String!
  role: String! = "member"
  expiresAt: String           # опционально, default +7 дней
}
```

### Queries

```graphql
type Query {
  # Приглашения текущего пользователя (все статусы)
  myInvitations: [InvitationType!]!

  # Публичный запрос — возвращает приглашение по коду ссылки
  # Без аутентификации (PUBLIC_FIELDS)
  invitationByCode(code: String!): InvitationType
}
```

`invitationByCode` добавлен в `PUBLIC_FIELDS` — может вызываться
неавторизованными пользователями для показа информации о приглашении.

### Mutations

```graphql
type Mutation {
  # Пригласить конкретного пользователя
  createInvitation(input: CreateInvitationInput!): InvitationType!

  # Принять приглашение (по ID)
  acceptInvitation(invitationId: String!): MemberType!

  # Отклонить приглашение
  declineInvitation(invitationId: String!): InvitationType!

  # Создать приглашение по ссылке (возвращает code для URL)
  createInvitationLink(input: CreateInvitationLinkInput!): InvitationType!

  # Принять приглашение по коду ссылки
  acceptInvitationByCode(code: String!): MemberType!
}
```

## Дубликаты и идемпотентность

- При `create_invitation` (userId) — проверяется, нет ли уже pending приглашения
  для того же пользователя в том же сообществе.
- При `create_invitation_link` — код всегда уникальный (ULID), без проверки
  дубликатов.
- `accept` и `decline` — только из статуса `pending`.
- `accept` проверяет `expires_at`: если срок истёк — `ValueError("Invitation expired")`.
- `_on_user_registered` не создаёт приглашений — только Member.

## Связи

- С [Membership](membership.md) — при принятии создаётся Member
- С [Auth](../guide/authentication.md) — `invitationByCode` не требует JWT
- С [Notifications](notifications.md) — событие `InvitationCreated` может породить `invited` notification (запланировано)

## Frontend

| Страница | Маршрут | Layout | Описание |
|---|---|---|---|
| MyInvitations | `/invitations` | auth | Список pending + past, Accept/Decline |
| PublicInvite | `/invite/:code` | guest | Публичная страница: community, role, expiry, Accept / Register |
| CommunityHome | `/community/:id` | auth | Модалка Invite: вкладки "By User ID" / "By Link" + кнопка Copy |
| Register | `/register?invite={code}` | guest | После регистрации авто-принимает invite code |

### Примечание по роутингу

`/invite/:code` использует `meta.layout = "guest"`, но роутер-гард пропускает
авторизованных пользователей (проверка `to.path.startsWith("/invite/")`).
Это нужно, чтобы залогиненный пользователь тоже мог открыть страницу приглашения
и нажать Accept.
