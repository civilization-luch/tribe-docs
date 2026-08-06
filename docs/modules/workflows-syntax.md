# Workflow Definition Syntax (BPMN-style)

> Статус: **draft v0.1** — формат уточняется по ходу реализации.
> Источник контракта для: backend-движка (`src/tribe_engine/workflows/`), frontend-конструктора, авторов определений, автогенерации тестов.

Единый авторский формат описания процессов в стиле BPMN. Модель: `states` (узлы-деятельности и события) + `gateways` (шлюзы) + `transitions` (sequence flows) + `roles` (участники) + `sla_hours` (таймауты).

Построен по требованиям «Процессы для человека и ИИ» (Солдатов): состояние ≠ действие, запрещённые переходы, проверяемые сценарии, эскалация, человек в контуре, версии правил.

---

## 1. Общая схема

```yaml
name: "Приём человека в кооператив"
entity: "membership_application"   # context_type: главный объект процесса
version: "1.2.0"                    # версия определения (инкремент при изменении)
trigger:
  event_type: MembershipApplicationSubmitted
  filter: {}                       # опц. equality-фильтр по payload события

roles:
  - applicant
  - secretary
  - system

states:
  DRAFT:
    type: "user_task"
    assigned_role: "applicant"
  ACCEPTED:
    type: "end_event"
    is_final: true

gateways:
  GW_REVIEW_OUTCOME:
    type: "exclusive"
    conditions:
      - if: "payload.status == 'ok'"
        target: "ACCEPTED"

transitions:
  - name: "submit"
    from: "DRAFT"
    to: "ACCEPTED"
    trigger_by: "applicant"
    action: "membership_approve"
    args: {}
```

Поля верхнего уровня (все опциональные, кроме требований ниже):

| Поле | Тип | Обязательное | Смысл |
|---|---|---|---|
| `name` | string | да | название процесса |
| `entity` / `context_type` | string | нет | тип главного объекта экземпляра |
| `version` | string | нет | версия определения (semver-подобная строка) |
| `roles` | string[] | нет | участники процесса |
| `states` | map | да | состояния (узлы) |
| `gateways` | map | нет | шлюзы |
| `transitions` | list | да | переходы (sequence flows) |
| `forbidden_transitions` | list | нет | явно запрещённые переходы (для негативных тестов) |

---

## 2. Состояния (`states`)

Карта `stateId → {type, ...}`. Типы:

### `start_event`
Точка входа. `trigger.event_type` — событие Event Store, создающее экземпляр процесса.

```yaml
START:
  type: "start_event"
```

### `end_event`
Конечный узел. `is_final: true` маркирует завершённые состояния.

```yaml
ACCEPTED:
  type: "end_event"
  is_final: true
```

### `user_task`
Ожидание действия человека. Поля:

| Поле | Тип | Смысл |
|---|---|---|
| `assigned_role` | string | роль, которая может выполнить переход (проверяется `require_role`) |
| `sla_hours` | number | лимит времени; по истечении — эскалация |
| `on_timeout` | string | переход/действие при истечении SLA (эскалация) |

```yaml
SECRETARY_REVIEW:
  type: "user_task"
  assigned_role: "secretary"
  sla_hours: 24
  on_timeout: "ESCALATED"
```

### `automated_task`
Автоматическое действие: вызов modifier action через `CommandProcessor`.

```yaml
SYNC_STATUS:
  type: "automated_task"
  dispatch: "projects_sync_issue_status"
  args:
    project_id: "${event.aggregate_id}"
    new_status: "${event.payload.new_status}"
  # контракт шага (см. §6)
  input: []               # какие переменные нужны
  output: ["status"]      # что обновляется в variables
  completion_criteria:    # условие завершения
    status: "synced"
```

Выход `automated_task` = результат `dispatch`. Валидация `output_schema` перед переходом дальше (Фаза 4).

---

## 3. Шлюзы (`gateways`)

### `exclusive` (XOR)
Исполняется ровно один переход, где `condition == true` (или первый подходящий).

```yaml
GW_REVIEW_OUTCOME:
  type: "exclusive"
  description: "Развилка по результатам проверки"
  conditions:
    - if: "payload.valid == false"
      target: "NEEDS_CLARIFICATION"
    - if: "payload.valid == true"
      target: "GW_FORK_CHECKS"
```

### `parallel_fork` (AND Fork)
Запускает несколько ветвей одновременно.

```yaml
GW_FORK_CHECKS:
  type: "parallel_fork"
  targets:
    - "SECURITY_CHECK"
    - "AWAITING_PAYMENT"
```

### `parallel_join` (AND Join)
Ждёт завершения **всех** входящих ветвей. `required_incoming` — какие ветки ждать; при завершении всех — `on_success`, при любой ошибке — `on_any_failure`.

```yaml
GW_JOIN_CHECKS:
  type: "parallel_join"
  required_incoming:
    - "SECURITY_CHECK"
    - "AWAITING_PAYMENT"
  on_success: "CHAIRMAN_APPROVAL"
  on_any_failure: "REJECTED"
```

---

## 4. Переходы (`transitions`)

Sequence flows: из какого состояния, в какое, каким инициатором и при каком условии.

| Поле | Тип | Обязательный | Смысл |
|---|---|---|---|
| `name` | string | ✅ | идентификатор перехода |
| `from` | string | ✅ | исходное состояние |
| `to` | string | ✅ | целевой узел (состояние или шлюз) |
| `trigger_by` | string | нет | роль исполнителя; `"system"` для автопереходов |
| `when` / `condition` | map | нет | условие выполнения |
| `action` | string | нет | dispatch-действие на переходе |
| `args` | map | нет | аргументы action и подстановки |

```yaml
transitions:
  - name: "secretary_complete"
    from: "SECRETARY_REVIEW"
    to: "GW_REVIEW_OUTCOME"
    trigger_by: "secretary"
  - name: "chairman_approve"
    from: "CHAIRMAN_APPROVAL"
    to: "ACCEPTED"
    trigger_by: "chairman"
    action: "register_new_member"
    args:
      member_id: "${variables.user_id}"
```

---

## 5. Запрещённые переходы

Явный список переходов, которые движок **не должен** выполнять. Служит для негативных тестов (книга, Гл. 8).

```yaml
forbidden_transitions:
  - from: "CLASSIFIED"
    to: "CLOSED"
    reason: "Нельзя закрыть без утверждённого ответа и записи отправки"
```

---

## 6. Переменные и подстановка

Контекст исполнения собирается в `variables` экземпляра:

| Переменная | Источник |
|---|---|
| `${event.aggregate_id}` | id объекта события |
| `${event.community_id}` | сообщество |
| `${event.payload.<key>}` | поле payload события |
| `${variables.<key>}` | процессная переменная (выход шага) |
| `${variables.user_id}` | идентификатор инициатора |

Условия на шлюзах и переходах используют ту же dot-нотацию с equality/вхождениями:

```yaml
if: "payload.recurring == true"
if: "payload.status in ['closed', 'done']"
```

---

## 7. Совместимость с legacy (`graph`/`steps`)

`states` ⊇ `nodes`, `transitions` ⊇ `edges`, `steps[]` — частный случай. Легаси-определения **компилируются** движком в новый формат без изменения данных:

| Легаси | Новый формат |
|---|---|
| `graph.nodes[].type: start` | `state_id: start_event` |
| `graph.nodes[].type: service_task` | `state_id: automated_task (+dispatch/args)` |
| `graph.nodes[].type: end` | `state_id: end_event (is_final: true)` |
| `graph.edges[] {from,to,condition}` | `transition {from,to,when}` |
| `steps[]` | цепочка `transition` по порядку |
| inline `condition` на ребре | условный exclusive-переход |

Движок исполняет **только** новый формат; входные `graph`/`steps` проходят `compile_legacy()` при создании/чтении.

---

## 8. Порядок исполнения движком

1. Событие Event Store (catch-all подписка) → поиск определений с `trigger.event_type`.
2. Фильтр `trigger.filter` → создание `WorkflowInstance` (`WorkflowStarted`).
3. `advance(instance, event|signal)`:
   - из текущего состояния собрать допустимые `transition` (по `from`, `trigger_by`, `condition`);
   - проверить `forbidden_transitions`;
   - `automated_task` → `dispatch()` + валидация контракта выхода;
   - `user_task` → остановить, ждать `WorkflowUserTaskCompleted` (approve/rework/escalate);
   - `gateway` → выбор/ветвление;
   - `end_event (is_final)` → `WorkflowCompleted`.
4. SLA: при событии/сигнале пересчитать `sla_hours` (эскалация через `on_timeout`).

---

## 9. Соответствие требованиям книги

| Требование | Как обеспечивается |
|---|---|
| Состояние ≠ действие; статус без правил — ярлык (Гл.8) | `states` как реальные позиции; переходы с инициатором/условием |
| Формат «дано/когда/тогда», шаги проверяемы (Гл.7) | `input`/`output`/`completion_criteria` → автоген acceptance-тестов |
| Основной / альтернативный / аварийный (Гл.7) | `exclusive`-«исключения», `parallel_join.on_any_failure`, эскалация |
| Правила отдельно от процесса (Гл.9) | шлюз — только точка решения; логика вынесена |
| Человек в контуре, не декоративный (Гл.21) | `user_task` + approval для необратимых шагов |
| Версии и регрессия (Гл.22) | `version`; экземпляр фиксирует использованную версию |

---

## 10. Changelog

- **v0.1 (draft)** — базовая скелет: states/gateways/transitions, переменные, legacy-компактибилити, forbidden transitions.