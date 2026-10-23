# Модуль Discussions

Обсуждения — рабочие разговоры внутри workspace.

## Модель

- **Discussion** принадлежит ровно одному workspace.
- Если workspace принадлежит группе, discussion имеет `group_id` этой группы.
- Community discussion живёт в community workspace (`group_id = null`).
- Групповые обсуждения переносятся в default workspace группы; community — в General
  workspace сообщества.
- Обсуждения не заменяют governance: решения, бюджет, мандат и объявления остаются на
  уровне группы.

## Сущности

### Discussion

| Поле | Описание |
|---|---|
| `id` | ULID |
| `workspace_id` | ULID workspace (обязателен) |
| `community_id` | ID сообщества |
| `group_id` | ID группы или `null` |
| `title` | Заголовок |
| `description` | Описание |
| `author_id` | Автор |
| `status` | `open` / `closed` |
| `pinned` | Закреплено |
| `labels` | Метки |
| `created_at` | Дата создания |

## GraphQL

- `discussions(communityId, status?, groupId?, workspaceId?)` — обсуждения с фильтрами.
- `discussion(id)` — одно обсуждение.
- `createDiscussion(input: {title, communityId, groupId?, workspaceId?, ...})` —
  создать обсуждение. Если workspace не задан, используется default workspace группы
  или сообщества.
- `updateDiscussion(input: {..., workspaceId?})` — изменить обсуждение или перенести его
  в другой workspace.
- `closeDiscussion` / `reopenDiscussion` / `setDiscussionPinned`.

## Permissions

- Создание и изменение: участник сообщества с правом `discussion.create` / `discussion.update`.
- Для группового обсуждения дополнительно требуется членство в группе или owner/admin.
- Приватная/скрытая группа не раскрывает свои обсуждения через community-запросы.
