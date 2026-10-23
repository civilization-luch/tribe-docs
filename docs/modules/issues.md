# Модуль Issues & Workspaces

Задачи, воркспейсы, milestones, доски, комментарии.

## Модель

- **Workspace** — постоянная рабочая область: issues, discussions, позже documents.
  Принадлежит одной группе или сообществу. Repository — опциональная интеграция.
- **Issue** — задача/баг/фича/улучшение. Принадлежит ровно одному workspace и может
  входить в несколько проектов.
- **Project** — представление/план над issues, а не контейнер. См. [Projects](projects.md).
- **Milestone** — веха внутри workspace (дата + набор issues).
- **Board** — устаревшая отдельная канбан-сущность; канбан живёт как view проекта.
- **Comment** — комментарий к issue или discussion.

### Workspace

| Поле | Описание |
|---|---|
| `id` | ULID |
| `name` | Название |
| `description` | Описание |
| `slug` | URL-идентификатор (опционально, авто из id[:8]) |
| `community_id` | ID сообщества |
| `group_id` | ID группы-владельца (`null` — workspace сообщества) |
| `is_default` | Default workspace в своём scope (community или group) |
| `created_at` | Дата создания |
| `metadata` | Свободные метаданные (JSON), см. ниже |

**Владение.** У workspace один владелец: группа (`group_id`) или сообщество
(`group_id = null`). Multi-owner не поддерживается. Общий доступ нескольким группам
проектируется через access grants, а не через нескольких владельцев.

**Default workspace.** В каждом scope может быть один default workspace: отдельно для
сообщества (`group_id = null`) и отдельно для каждой группы. Group workspace создаётся
по требованию при первой задаче/обсуждении, если его ещё нет.

**Видимость.** Workspace группы наследует видимость группы (`public` / `private` /
`hidden`). Workspace сообщества доступен участникам сообщества.

**Привязка репозитория (`metadata.repository`).** Workspace можно связать с git-репозиторием, который использует community agent. Форма зафиксирована схемой `schemas/workspaces/metadata.schema.json`.

```json
{ "repository": { "url": "https://github.com/org/repo", "branch": "main" } }
```

- Поля: `url` (обязательно, https или ssh), `branch`, `provider`, `path`.
- **Credentials запрещены**: токены и SSH-ключи хранятся только в конфиге агента, не в metadata.
- Чтение: `repoUrl` / `repoBranch` на `WorkspaceType`. Запись: мутация `setWorkspaceRepository(workspaceId, url, branch)` (owner/admin), эмитит `WorkspaceUpdated`.

### Issue

| Поле | Описание |
|---|---|
| `id` | ULID |
| `workspace_id` | ULID workspace (обязателен) |
| `community_id` | ID сообщества |
| `group_id` | ID группы (совпадает с владельцем workspace или `null`) |
| `title` | Заголовок |
| `description` | Описание |
| `issue_type` | `bug` / `feature` / `task` / `improvement` |
| `status` | `open` / `in_progress` / `resolved` / `closed` |
| `assignee_id` | ID ответственного (null) |
| `priority` | 0 = normal, 1 = high, 2 = urgent |
| `parent_id` | ULID родительской issue (null) |
| `labels` | Метки (["frontend", "bug"]) |
| `checklist` | Чек-лист ([{ "text": "...", "done": false }]) |
| `links` | Связи с другими issues ([{ "target_id": "...", "type": "blocks" }]) |
| `created_at` | Дата создания |

Инварианты:

- issue всегда принадлежит одному workspace;
- если workspace принадлежит группе, `issue.group_id` совпадает с этой группой;
- issue community-workspace не имеет `group_id`;
- перенос issue между workspace меняет `workspace_id` и `group_id` вместе (`IssueMoved`).

### Issue Links (встроены в Issue)

| Link Type | Описание |
|---|---|
| `relates_to` | Связана |
| `blocks` | Блокирует |
| `blocked_by` | Заблокирована |
| `duplicate_of` | Дубликат |
| `parent_of` | Родительская |
| `child_of` | Дочерняя |

### Milestone

| Поле | Описание |
|---|---|
| `id` | ULID |
| `workspace_id` | ULID workspace |
| `name` | Название |
| `description` | Описание |
| `due_at` | Плановая дата (ISO 8601, опционально) |
| `status` | `open` / `reached` |
| `created_at` | Дата создания |

### Board

> DEPRECATED. Отдельная сущность канбан-доски больше не используется: канбан реализуется как board view внутри проекта (`Project.settings.statuses` задают колонки). Сохранена только для переноса данных — см. `scripts/migrate_boards_to_projects.py`.

| Поле | Описание |
|---|---|
| `id` | ULID |
| `name` | Название |
| `description` | Описание |
| `community_id` | ID сообщества |
| `view_type` | `kanban` (по умолчанию) |
| `columns` | JSON: [{ "id": "open", "title": "To Do" }, ...] |
| `filters` | JSON: { "workspace_ids": [...], "statuses": [...], "assignee_ids": [...], "labels": [...] } |
| `items` | Резолвер — все issues, отфильтрованные по filters |
| `created_at` | Дата создания |

### Comment

| Поле | Описание |
|---|---|
| `id` | ULID |
| `issue_id` | ULID issue |
| `body` | Текст комментария |
| `author_id` | ID автора ("system" для системных) |
| `is_system` | Автоматический комментарий (статус, назначение, линк) |
| `community_id` | ID сообщества |
| `created_at` | Дата создания |

## Commands

| Команда | Описание |
|---|---|
| `CreateWorkspace` | Создать workspace (community или group-owned) |
| `UpdateWorkspace` | Обновить workspace |
| `SetDefaultWorkspace` | Назначить default workspace в его scope |
| `CreateIssue` | Создать issue (workspace и group согласуются) |
| `MoveIssue` | Перенести issue в другой workspace |
| `UpdateIssue` | Обновить issue |
| `AssignIssue` | Назначить ответственного |
| `ChangeIssueStatus` | Сменить статус (open → in_progress → resolved → closed) |
| `LinkIssues` | Связать две issues |
| `UnlinkIssues` | Убрать связь |
| `CreateMilestone` | Создать milestone |
| `ReachMilestone` | Отметить milestone достигнутым |
| `CreateBoard` | Создать канбан-доску (DEPRECATED) |
| `UpdateBoard` | Обновить доску (DEPRECATED) |
| `CreateComment` | Добавить комментарий |
| `UpdateComment` | Редактировать комментарий |

## Events

| Событие | Описание |
|---|---|
| `WorkspaceCreated` | Workspace создан (`group_id`, `is_default`) |
| `WorkspaceUpdated` | Workspace обновлён |
| `IssueCreated` | Issue создана |
| `IssueUpdated` | Issue обновлена |
| `IssueMoved` | Issue перенесена в другой workspace/group |
| `IssueAssigned` | Назначен ответственный |
| `IssueStatusChanged` | Статус изменён |
| `IssueLinked` | Связь установлена |
| `IssueUnlinked` | Связь убрана |
| `MilestoneCreated` | Milestone создан |
| `MilestoneReached` | Milestone достигнут |
| `BoardCreated` | Доска создана (DEPRECATED) |
| `BoardUpdated` | Доска обновлена (DEPRECATED) |
| `CommentCreated` | Комментарий создан |
| `CommentUpdated` | Комментарий отредактирован |

## Системные комментарии

При изменении состояния Issue автоматически создаются системные комментарии (author_id="system", is_system=true) через SubscriptionRunner:

| Событие | Системный комментарий |
|---|---|
| `IssueAssigned` | "assigned to {assignee_id}" |
| `IssueStatusChanged` | "changed status from {old} to {new}" |
| `IssueLinked` | "linked as {type} with {target_id}" |
| `IssueUnlinked` | "unlinked {type} from {target_id}" |

## Связи

- С Groups — workspace может принадлежать группе; группа наследует видимость и роли.
- С [Projects](projects.md) — проект собирает issues из одного или нескольких workspace.
- С [Reputation](reputation.md) — бонусы за выполнение задач
- С [Content](content.md) — объявления о задачах
- С [Membership](membership.md) — проверка членства при назначении
- С [Notifications](notifications.md) — уведомления о назначениях, комментариях, смене статуса (запланировано)

## GraphQL

- `workspace(id)` — workspace по ID или slug.
- `workspaces(communityId, groupId)` — community и доступные group workspaces.
- `groupWorkspaces(groupId)` — workspace конкретной группы.
- `defaultWorkspace(communityId, groupId)` — default workspace в нужном scope.
- `createWorkspace(input: {communityId, groupId?, name, ...})` — создать workspace.
- `deleteWorkspace` — запрещён, если в workspace есть issues или discussions.
