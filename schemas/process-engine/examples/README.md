# Пример: закупка (Purchase)

Эталонное определение для первого запуска движка. Соответствует агрегату `ge_purchase` в `tribe-engine`.

Файл: [`purchase.process.json`](purchase.process.json)

## Процесс

| Состояние | Тип | Смысл |
|---|---|---|
| `__start__` | start_event | закупка создана |
| `collecting` | wait | сбор заявок до `join_deadline` |
| `ready` | wait | минимум собран, ждём запуска |
| `launched` | end_event | запущено; дальнейшее исполнение — процесс сделок |
| `expired` | end_event | дедлайн прошёл, минимум не собран |
| `cancelled` | end_event | отменено организатором |
| `failed` | end_event | запуск провалился, сделки не созданы |

## Принятые решения

| Вопрос | Решение | Как выражено |
|---|---|---|
| Частичный запуск сделок | **всё-или-ничего** | при ошибке адаптер присылает `PurchaseLaunchFailed`, процесс уходит в `failed` |
| Отмена после запуска | **вне этого процесса** | `launched` финальный; отмена относится к процессу сделок |
| Заявки (`PurchaseApplication`) | **отдельные агрегаты**, не моделируются здесь | их дедлайн и SLA — будущий процесс заявки |
| Отмена после `expired` | **невозможна** | запреты `deny_expired_*` |
| Идемпотентность команд | ключ формирует адаптер | `(definition_id, instance_id, transition_id)` |

## Переменные

Адаптер передаёт в инстанс на старте и поддерживает при событиях:

| Переменная | Источник |
|---|---|
| `collected` | сумма поданных заявок |
| `min_total` | `purchase.min_total` |
| `max_total` | `purchase.max_total` |
| `max_per_member` | `purchase.max_per_member` |
| `join_deadline` | `purchase.join_deadline` |
| `delivery_covered` | покрытие поставки при `advance_percent > 0` |

## Модель процесса

| Блок | Значение |
|---|---|
| `passport.owner` | `organizer` |
| `roles` | `organizer` (human), `member` (human), `system` (system) |
| `rules` | `br_min_collection` — детерминированное, версия 2026-10-01 |
| `data` | `collected`, `min_total`, `max_total`, `max_per_member`, `join_deadline`, `delivery_covered` |
| `agent.deny` | `guarantee_launch_purchase`, `guarantee_cancel_purchase` |
| `tests` | 4 декларативных сценария |

## Расхождения с текущей реализацией

| Что в коде | Что в определении |
|---|---|
| `ready`/`expired` ставит ежедневный скан | таймер `timer_join_deadline` |
| запуск допускает частичные отказы | запуск атомарен, ошибка → `failed` |
| `cancel()` разрешён из `launched` | отмена после запуска вне процесса |
| нет SLA на рассмотрение заявки | отложено до процесса заявки |
| `collected` считается в модификаторе | переменная инстанса |

## Таймер

`timer_join_deadline` объявляет только дедлайн. Развилка — переходами:

```
tr_ready   collecting → ready    on timer, collected >= min_total
tr_expire  collecting → expired  on timer, collected <  min_total
```
