# Модуль Issues & Workspaces

Задачи, воркспейсы, milestones, доски, комментарии.

## Сущности

- **Workspace** — контейнер для группы issues (фича, модуль, релиз)
- **Issue** — задача/баг/фича/улучшение
- **Milestone** — веха внутри workspace (дата + набор issues)
- **Board** — канбан-доска (view/projection с колонками и фильтрами)
- **Comment** — комментарий к issue (пользовательский или системный)

### Workspace

| Поле | Описание |
|---|---|
| `id` | ULID |
| `name` | Название |
| `description` | Описание |
| `slug` | URL-идентификатор (опционально, авто из id[:8]) |
| `community_id` | ID сообщества |
| `created_at` | Дата создания |

### Issue

| Поле | Описание |
|---|---|
| `id` | ULID |
| `workspace_id` | ULID workspace |
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
| `CreateWorkspace` | Создать workspace |
| `UpdateWorkspace` | Обновить workspace |
| `CreateIssue` | Создать issue |
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
| `WorkspaceCreated` | Workspace создан |
| `WorkspaceUpdated` | Workspace обновлён |
| `IssueCreated` | Issue создана |
| `IssueUpdated` | Issue обновлена |
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

- С [Reputation](reputation.md) — бонусы за выполнение задач
- С [Content](content.md) — объявления о задачах
- С [Membership](membership.md) — проверка членства при назначении
- С [Notifications](notifications.md) — уведомления о назначениях, комментариях, смене статуса (запланировано)
