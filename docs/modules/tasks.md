# Модуль Tasks & Projects

Задачи, проекты, milestones, дежурства.

## Сущности

- **Project** — контейнер для группы задач (фича, модуль, релиз)
- **Milestone** — веха внутри проекта (дата + набор задач)
- **Task** — задача (что, кто, когда)
- **Duty** — регулярное дежурство (ротация)
- **DutyRoster** — график дежурств

### Project

| Поле | Описание |
|---|---|
| `id` | ULID |
| `name` | Название |
| `description` | Описание |
| `status` | `active` / `archived` |
| `owner_id` | ULID участника-владельца |

### Milestone

| Поле | Описание |
|---|---|
| `id` | ULID |
| `project_id` | ULID проекта |
| `name` | Название |
| `due_date` | Плановая дата |
| `status` | `pending` / `reached` |

### Task

| Поле | Описание |
|---|---|
| `id` | ULID |
| `project_id` | ULID проекта (null для standalone) |
| `milestone_id` | ULID milestone (null) |
| `title` | Заголовок |
| `description` | Описание |
| `status` | `open` / `in_progress` / `completed` |
| `assignee_id` | ULID ответственного (null) |

## Commands

| Команда | Описание |
|---|---|
| `CreateProject` | Создать проект |
| `UpdateProject` | Обновить проект |
| `ArchiveProject` | Архивация проекта |
| `AddMilestone` | Добавить milestone |
| `CompleteMilestone` | Отметить milestone выполненным |
| `CreateTask` | Создать задачу |
| `UpdateTask` | Обновить задачу |
| `AssignTask` | Назначить ответственного |
| `CompleteTask` | Отметить выполнение |

## Events

| Событие | Описание |
|---|---|
| `ProjectCreated` | Проект создан |
| `ProjectUpdated` | Проект обновлён |
| `ProjectArchived` | Проект архивирован |
| `MilestoneAdded` | Добавлен milestone |
| `MilestoneReached` | Milestone выполнен |
| `TaskCreated` | Задача создана |
| `TaskUpdated` | Задача обновлена |
| `TaskAssigned` | Назначен ответственный |
| `TaskCompleted` | Задача выполнена |

## Типы задач

- Уборка (кухня, коридор, санузел)
- Вынос мусора
- Полив растений
- Проверка запасов
- Встреча новых жильцов
- Организация мероприятий
- Код-ревью, домашки, хакатоны

## Связи

- С [Reputation](reputation.md) — бонусы за выполнение задач
- С [Content](content.md) — объявления о дежурствах
- С [Membership](membership.md) — проверка членства при назначении

## Лицензия

**Community** (OSS). Projects — часть модуля, не требующий Premium.
