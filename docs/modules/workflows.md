# Workflows (автоматизация по событиям)

Модуль автоматизации процессов в сообществе. Два уровня:

1. **ECA Rules** (реализовано) — «когда событие X после условия → действие Y». Агрегат `WorkflowDefinition`, реестр правил, каталог шаблонов.
2. **BPMN-движок** (план, см. `workflows-syntax.md`) — многошаговые процессы: `states` + `gateways` + `transitions`, `WorkflowInstance`.

Подробности грамматики процесса — в [workflows-syntax.md](workflows-syntax.md).

---

## Как это работает

Правило связывает **событие** из Event Store и **действие** (modifier action). Когда в событии срабатывает триггер (тип + опциональный фильтр по payload), движок выполняет шаги.

Движок работает на **подписке на все события** (`SubscriptionRunner`, catch-all `*`): на каждое сохранённое событие проверяются включённые определения сообщества.

### Пример ECA-правила (YAML)

```yaml
trigger:
  event_type: IssueStatusChanged
  filter:
    new_status: closed
steps:
  - type: service_task
    dispatch: issues_sync_item_status
    args:
      issue_id: "${event.aggregate_id}"
      new_status: "${event.payload.new_status}"
```

## Агрегат WorkflowDefinition (event-sourced)

`aggregate_type = "workflow_definition"`.

```python
class WorkflowDefinition(AggregateRoot):
    aggregate_type = "workflow_definition"
    # community_id, name, description, template_key
    # trigger_event_type, trigger_filter
    # steps[] | graph{nodes, edges}
    # enabled, deleted, version, roles, states, gateways, transitions
```

События: `WorkflowDefinitionCreated/Updated/Toggled/Deleted`.

## Агрегат WorkflowInstance (план)

`aggregate_type = "workflow_instance"` — запущенный экземпляр процесса:
`definition_id`, `definition_version`, `context`, `current`, `completed_states[]`, `open_branches[]`, `variables`, `timers[]`, `status`.

События: `WorkflowStarted`, `StateEntered`, `StateLeft`, `UserTaskCreated`, `SlaExpired`, `Completed`, `Failed`, `WorkflowUserTaskCompleted`.

## Actions (WorkflowsModifier)

| Action | Описание |
|---|---|
| `workflows_create_rule` | создать правило (ECA) |
| `workflows_update_rule` | обновить правило |
| `workflows_toggle_rule` | вкл/выкл |
| `workflows_delete_rule` | удалить (soft) |
| `workflows_activate_template` | активировать шаблон каталога |
| `workflows_start` *(план)* | создать `WorkflowInstance` по триггеру |
| `workflows_advance` *(план)* | продвинуть экземпляр по событию/сигналу |
| `workflows_complete_user_task` *(план)* | approve/rework/escalate пользователем |
| `workflows_signal` *(план)* | внешний сигнал |

## Каталог шаблонов

YAML в `src/tribe_engine/workflows/catalog/`. Активация: `workflows_activate_template`.

Текущие:
- `auto_close_issue_on_item_status`
- `sync_issue_status_to_projects`
- `recurring_item_on_iteration`

## GraphQL

- **Query**: `availableEventTypes`, `availableActions`, `workflowRules`, `workflowRule`, `workflowCatalog`
- **Mutation**: `createWorkflowRule`, `updateWorkflowRule`, `toggleWorkflowRule`, `deleteWorkflowRule`, `activateWorkflow`

## Permissions

- `workflow.manage` — owner/admin.

## Веб-интерфейс

- Вкладка Workflow в CommunitySettings: список правил, create/edit modal (триггер, фильтр, шаги), toggle, delete.
- Планируется конструктор BPMN-процессов (states/gateways/transitions) для owner/admin.