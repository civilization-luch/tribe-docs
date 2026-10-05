# Модуль Projects & Boards

Канбаны и проекты сообщества. Отдельный community-модуль `projects`, который **зависит от** модуля
[Issues & Workspaces](issues.md) (`workspaces`), так как использует задачи (issues).

## Назначение

- **Project** — контейнер для доски и связанных задач.
- **Board** — канбан-представление проекта: колонки по статусам, карточки-задачи.

Канбан реализован как **board view внутри проекта**: колонки задаются `Project.settings.statuses`.
Отдельная сущность Board сохранена только для переноса данных (см. `scripts/migrate_boards_to_projects.py`).

## Сущности

### Project

| Поле | Описание |
|---|---|
| `id` | ULID |
| `name` | Название |
| `description` | Описание |
| `community_id` | ID сообщества |
| `views` | Представления проекта (board/list) |
| `items` | Элементы проекта (задачи) |
| `settings` | Настройки (в т.ч. `statuses` — колонки канбана) |
| `created_at` | Дата создания |

### Board view

Колонки и фильтры определяются `Project.settings.statuses`; карточки — задачи (issues) сообщества.

## Связь с Issues & Workspaces

- Задачи берутся из модуля [Issues & Workspaces](issues.md).
- Перемещение карточки синхронизирует статус задачи (`sync_issue_status`).

## Регистрация как модуля

| Поле | Значение |
|---|---|
| `id` | `projects` |
| `scope` | `community` |
| `category` | `collaboration` |
| `modifiers` | `["projects"]` |
| `depends_on` | `["workspaces"]` |
| `route` | `/community/:id/projects` (доска — `/community/:id/boards/:boardId`) |
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
| `CreateProject` | Создать проект |
| `UpdateProject` | Обновить проект |
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
