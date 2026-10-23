# Модуль Projects & Boards

Канбаны и проекты сообщества. Отдельный community-модуль `projects`, который **зависит от** модуля
[Issues & Workspaces](issues.md) (`workspaces`), так как использует задачи (issues).

## Назначение

- **Project** — представление/план над issues. Проект не владеет задачами и не перемещает их.
- **Board** — канбан-представление проекта: колонки по статусам, карточки-задачи.

Канбан реализован как **board view внутри проекта**: колонки задаются `view.config.columns`,
а список доступных статусов — `Project.settings.statuses`. Если `settings.statuses` пуст,
используются колонки board view (это важно для мигрированных досок). Legacy Board удалён из
GraphQL API и UI; агрегат `Board` оставлен только для чтения исторических событий и скрипта
`scripts/migrate_boards_to_projects.py`.

## Сущности

### Project

| Поле | Описание |
|---|---|
| `id` | ULID |
| `name` | Название |
| `description` | Описание |
| `community_id` | ID сообщества |
| `group_id` | ID группы-владельца (пусто для проекта сообщества) |
| `collaborator_group_ids` | Группы-партнёры, которым доступен проект |
| `source_workspace_ids` | Workspace-источники; пусто = без ограничения |
| `owner_id` | Ответственный/lead проекта (пользователь) |
| `status` | `planned` / `active` / `paused` / `done` / `archived` |
| `start_date` | Дата начала |
| `due_date` | Целевая дата |
| `views` | Представления проекта (board/table/roadmap) |
| `items` | Элементы проекта (задачи и черновики) |
| `settings` | Настройки (в т.ч. `statuses` и `fields`) |
| `created_at` | Дата создания |

`projects(communityId, includeArchived)` по умолчанию скрывает архивные проекты.
Элементы проекта читаются через `items(limit, offset)`; `itemCount` содержит общее количество.
Issue для элементов загружаются пакетно (`get_many`) и проверяются по community/group ACL.
История изменений проекта доступна через `entityHistory(aggregateType: "project", aggregateId: id)`.

### Board view

Колонки и фильтры определяются `view.config.columns`; карточки — элементы проекта. Если у
элемента статус не совпадает ни с одной колонкой, он не должен теряться: интерфейс добавляет
колонку для такого статуса. Карточки — задачи (issues) сообщества и черновики.

### Участники и роли

У проекта нет собственного membership или отдельной системы ролей. Роли наследуются:

- проект сообщества — роли сообщества; `owner_id` действует как lead;
- проект группы — роли группы-владельца; лидер/солидер управляют, участники вносят вклад;
- группы из `collaborator_group_ids` видят проект и могут вносить элементы, но не управляют
  настройками проекта;
- owner/admin сообщества имеют полный доступ к обоим типам.

`collaborator_group_ids` — это доступ, а не совладение. Владелец проекта всегда один:
группа (`group_id`) или сообщество (`group_id` пуст).

### Source workspaces

`source_workspace_ids` ограничивает набор workspace, из которых проект собирает issues.

- Пустой список — проект принимает issues из любого доступного workspace.
- Заданный список — issue можно добавить только из этих workspace.
- Workspace-источник должен принадлежать сообществу проекта и быть доступным owner/collaborator
  группе (или быть community workspace).
- Ограничение проверяется при `addProjectItem` и `convertDraft`.
- Новые issues, создаваемые из проекта, предлагают только source workspaces.

`projects` поддерживает фильтры `scope` (`all` / `community` / `group` / `shared`),
`sharedWithGroupId` и `sourceWorkspaceId`.

### Group projects

- `group_id` не задан — проект сообщества.
- `group_id` задан — проект принадлежит группе и наследует её видимость:
  `public` видят участники сообщества, `private`/`hidden` — участники группы и owner/admin.
- Создавать, настраивать и удалять групповой проект может owner/admin сообщества, лидер или
  солидер группы. Участники группы могут добавлять задачи, черновики и менять статусы элементов.
- `projects(communityId, groupId)` возвращает проекты группы и проекты, где группа является
  collaborator; без `groupId` — проекты сообщества и доступные пользователю групповые проекты.
- Группа не может быть удалена, пока у неё есть workspace или проекты-владельцы. Проекты группы
  не заменяют workspace. При удалении группы её ID убирается из collaborator-списков проектов.
- В групповой проект можно добавить issue из workspace группы-владельца или группы-партнёра.
  Community issues добавлять можно; issue из чужой группы — нельзя. Если задан
  `source_workspace_ids`, issue из workspace вне списка отклоняется.
- Видимость issue по-прежнему определяется workspace/group ACL. Если у пользователя нет доступа
  к issue группы-партнёра, проект может показать элемент без данных issue.

## Связь с Issues & Workspaces

- Проект — view над задачами, а не их контейнер.
- Задача принадлежит одному workspace и может входить в несколько проектов.
- Удаление задачи из проекта не удаляет саму задачу.
- Задачи берутся из модуля [Issues & Workspaces](issues.md).
- Проектные запросы проверяют активное членство в сообществе.
- Задачи внутри проекта проверяются по community и group ACL: чужая или скрытая задача не
  раскрывается через проект.
- Перемещение карточки меняет статус элемента проекта. Синхронизация с issue выполняется
  workflow-правилами (`sync_issue_status`, `auto_close_issue_on_item_status`) и должна учитывать
  маппинг статусов.

## Регистрация как модуля

| Поле | Значение |
|---|---|
| `id` | `projects` |
| `scope` | `community` |
| `category` | `collaboration` |
| `modifiers` | `["projects"]` |
| `depends_on` | `["workspaces"]` |
| `route` | `/community/:id/projects` (board view внутри `/projects/:projectId`) |
| `handles_entities` | `project` |

### Настройки

| Ключ | Тип | По умолчанию | Описание |
|---|---|---|---|
| `default_view` | string (`board`/`list`) | `board` | Вид по умолчанию |
| `require_approval` | bool | `false` | Требовать одобрение |
| `allow_member_boards` | bool | `true` | Разрешить участникам создавать доски |

## Commands

| Команда | Описание |
|---|---|
| `CreateProject` | Создать проект (owner group/community + collaborators) |
| `UpdateProject` | Обновить проект, включая `collaborator_group_ids` |
| `DeleteProject` | Удалить проект |
| `AddItem` / `RemoveItem` | Добавить/убрать задачу в проекте |
| `CreateDraft` / `ConvertDraft` / `UpdateDraft` / `UpdateItem` | Черновики и элементы |
| `AddView` / `UpdateView` / `RemoveView` / `ReorderViews` | Представления проекта |
| `UpdateSettings` | Настройки проекта |
| `SyncIssueStatus` / `SyncRecurringStatus` | Синхронизация статуса задачи |

## Events

| Событие | Описание |
|---|---|
| `ProjectCreated` | Проект создан |
| `ProjectUpdated` | Проект обновлён |
| `ProjectDeleted` | Проект удалён |
| `ProjectSettingsUpdated` | Настройки проекта обновлены |

## Гейтинг

- Запись: команды `projects_*` блокируются, если модуль `projects` не включён в сообществе
  (`CommandProcessor.dispatch`).
- Чтение: GraphQL-запросы `projects`, `project`, `board`, `boards` проходят проверку
  `enforce_read_module` (`graphql/module_guards.py`) и возвращают ошибку, если модуль выключен.
