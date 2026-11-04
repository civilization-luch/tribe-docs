<!-- generated: module-guide; verified_by: e2e; date: 2026-11-04 -->
# Наставничество

Наставничество один-на-один. Отслеживание сессий, целей и прогресса.

## Где найти

`/community/:id/mentorship`

## Настройки

- **Макс. пар на наставника** (`max_pairs_per_mentor`) — целое, по умолчанию `5`
- **Одобрять пары** (`require_approval`) — да/нет, по умолчанию `True`
- **Напоминание о сессии, дн** (`session_reminder_days`) — целое, по умолчанию `1`

## Проверено e2e

- mentorship full flow: mentor → request → accept → lessons → complete → rate

## Ограничения

- Модуль включает администратор сообщества; если он выключен, раздела нет в меню.
