# Модуль Bookings

Бронирование комнат, рабочих мест, общих пространств.

## Сущности

- **Room** — комната/место (с типом: private, shared, workspace)
- **Booking** — бронь на даты
- **Calendar** — календарь занятости
- **Rate** — цена за ночь/месяц

## Actions

| Action | Описание |
|---|---|
| `list_rooms` | Список доступных комнат |
| `book` | Забронировать |
| `cancel_booking` | Отменить бронь |
| `check_in` | Заселение |
| `check_out` | Выезд |
| `get_calendar` | Календарь занятости |
| `set_rate` | Установить цену |

## Типы пространств

- Private room — отдельная комната
- Shared room — место в общей комнате
- Workspace — рабочее место в коворкинге
- Common area — мероприятие/ивент

## Связи

- С [Billing](billing.md) — оплата брони
- С [Content](content.md) — мероприятия в пространствах
- С [Groups](../coliving/groups.md) — бронирование в рамках дома
