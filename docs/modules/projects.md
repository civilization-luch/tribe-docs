# Модуль Projects

Агрегатор комплексных задач: группировка, milestones, отслеживание прогресса фич и релизов.

## Сущности

- **Project** — контейнер для группы задач (фича, модуль, релиз)
- **Milestone** — веха внутри проекта (дата + набор задач)

### Project

| Поле | Описание |
|---|---|
| `id` | UUID |
| `name` | Название |
| `description` | Описание |
| `status` | `active` / `archived` |
| `owner_id` | UUID участника-владельца |
| `module` | Связанный модуль Tribe (опционально: `billing`, `frontend`, …) |

### Milestone

| Поле | Описание |
|---|---|
| `id` | UUID |
| `project_id` | UUID проекта |
| `name` | Название |
| `due_date` | Плановая дата |
| `status` | `pending` / `reached` |

## Commands

| Команда | Описание |
|---|---|
| `CreateProject` | Создать проект |
| `UpdateProject` | Обновить проект |
| `ArchiveProject` | Архивация проекта |
| `AddMilestone` | Добавить milestone |
| `CompleteMilestone` | Отметить milestone выполненным |
| `LinkTask` | Привязать задачу из модуля Tasks к проекту |
| `UnlinkTask` | Отвязать задачу от проекта |

## Events

| Событие | Описание |
|---|---|
| `ProjectCreated` | Проект создан |
| `ProjectUpdated` | Проект обновлён |
| `ProjectArchived` | Проект архивирован |
| `MilestoneAdded` | Добавлен milestone |
| `MilestoneReached` | Milestone выполнен |
| `TaskLinkedToProject` | Задача привязана к проекту |
| `TaskUnlinkedFromProject` | Задача отвязана от проекта |

## Связь с Tasks

Projects — отдельный bounded context. Tasks модуль **не знает** о Projects.

```
Projects (Event Store)
  ├── Project { id, name, description, status, owner_id, module }
  └── ProjectTaskLink { project_id, task_id, linked_at }
  └── Milestone { id, project_id, name, due_date, status }

Tasks (Event Store)
  └── Task { id, title, status, assignee, ... }
```

Привязка задачи к проекту:

```
→ LinkTask(projectId, taskId)
  → TaskLinkedToProject { projectId, taskId, linkedAt }
    → Read Model: ProjectTasks[projectId] = [taskId, …]
```

В GraphQL данные склеиваются на уровне резолвера:

```graphql
query {
  project(id: "proj-1") {
    name
    tasks {
      id
      title
      status
    }
  }
}
```

`tasks` в Project — резолвер читает `ProjectTasks` из read model Projects, затем запрашивает данные задач из read model Tasks.

## Лицензия

**Premium** — опциональный модуль. Сообщества без Projects используют Tasks как плоский список.
