# Workflow (Рабочие процессы)

Пошаговые процессы с этапами и автоматическим продвижением. Стандартизация повторяющихся операций.

Sequence machine — линейный конечный автомат без ветвлений (продвижение только вперёд/назад по массиву этапов).

## Примеры процессов

| Процесс | Этапы |
|---|---|
| Ремонт общего имущества | Заявка → Оценка → Утверждение → Выполнение → Приёмка |
| Сезонные работы | Планирование → Распределение → Выполнение → Отчёт |
| Закупка | Заявка → Сбор → Закупка → Распределение → Расчёт |

## Агрегат Workflow (event-sourced)

```python
class Workflow(AggregateRoot):
    aggregate_type = "workflow"

    @classmethod
    def create(cls, template: str, stages: list[dict], title: str):
        # template — пресет этапов ("repair", "procurement", "custom")
        # stages — кастомный массив [{name, description}, ...]
        r.raise_event(WorkflowCreated(
            template=template,
            stages=stages,
            current_index=0,
            status="active",
        ))

    def advance(self):
        self.raise_event(StageAdvanced(new_index=self._current_index + 1))

    def revert(self):
        self.raise_event(StageReverted(new_index=self._current_index - 1))

    def complete(self):
        self.raise_event(WorkflowCompleted())

    def cancel(self):
        self.raise_event(WorkflowCancelled())

    def _when(self, event):
        match event.event_type:
            case "WorkflowCreated":
                self._stages = event.payload["stages"]
                self._current_index = 0
                self._status = "active"
            case "StageAdvanced":
                self._current_index = event.payload["new_index"]
            case "StageReverted":
                self._current_index = event.payload["new_index"]
            case "WorkflowCompleted":
                self._status = "completed"
            case "WorkflowCancelled":
                self._status = "cancelled"

    def to_attributes(self):
        return {
            "title": self._title,
            "stages": self._stages,
            "current_stage": self._stages[self._current_index],
            "current_index": self._current_index,
            "status": self._status,
        }
```

## Конструктор Workflow

Визуальный редактор последовательности этапов:

```
Pipeline:

  [Request]  ➜  [Estimate]  ➜  [Approval]  ➜  [Execution]  ➜  [Acceptance]
                           [+ Add Stage]
```

- StageCard: name, description, кнопки удаления и перетаскивания
- Drag-and-drop реордеринг этапов
- Шаблоны (preset) заполняют список этапов
- Кастомный процесс — пользователь собирает этапы вручную

## Actions (6)

| Action | Описание |
|---|---|
| `createWorkflow` | Создать процесс (из шаблона или кастомный) |
| `addStage` | Добавить этап (только до первого advance) |
| `advanceStage` | Продвинуть на следующий этап |
| `revertStage` | Вернуть на предыдущий этап |
| `completeWorkflow` | Завершить процесс (на последнем этапе) |
| `cancelWorkflow` | Отменить процесс |

## Интерфейс

- Список процессов с названием и количеством этапов
- Визуализация этапов (прогресс-бар)
- Текущий этап и следующие шаги
- Кнопки: Создать процесс, Добавить этап, Продвинуть, Вернуть, Завершить, Отменить
