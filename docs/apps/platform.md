# Tribe Platform (SPA)

Основное приложение. Внутри него живут все сообщества.

## Структура

### Auth / Вход

- Регистрация (email, Telegram, Google)
- Вход
- Выбор/создание сообщества после входа

### Дашборд сообщества

Главный экран после входа. Отображает:

- Активные задачи (Issues)
- Ближайшие мероприятия (Content)
- Последние голосования (Proposals)
- Баланс казны (Treasury)
- Текущий рейтинг (Reputation)

Виджеты подстраиваются под пресет сообщества (у CoLive другие виджеты, чем у CoSpace).

### Разделы по модулям

Каждый активный модуль даёт свою страницу в навигации:

| Модуль | Страница |
|---|---|
| Membership | Участники, заявки, профили |
| Billing | Платежи, подписки, счета |
| Bookings | Календарь брони, комнаты |
| Issues | Доска задач, воркспейсы |
| Treasury | Казна, фонды, отчёты |
| Content | Лента, афиша, обсуждения |
| Proposals | Голосования, предложения |
| Reputation | Рейтинг, нашивки, уровни |
| Documents | Договоры, акты, шаблоны |
| LMS | Курсы, уроки, сертификаты |
| Marketplace | Объявления, заказы, услуги |
| Registry | Реестр (для CoSettle / CoOper) |
| Schedule | Расписание (для CoSpace) |

### Админка

- Настройки сообщества (пресет, модули, регион)
- Управление ролями и правами
- Настройки голосований (кворум, сроки)
- Финансовые отчёты
- Приглашения

### Профиль участника

- Имя, аватар, контакты
- История платежей
- Репутация и нашивки
- Список сообществ

## Навигация

```
┌─────────────┬──────────────────────┐
│   Sidebar   │   Content Area       │
│             │                      │
│  [Dashboard]│  [Dashboard widgets] │
│  [Members]  │                      │
│  [Workspaces]│                      │
│  [Treasury] │                      │
│  [Calendar] │                      │
│  [Voting]   │                      │
│  ...        │                      │
│             │                      │
│  [Settings] │                      │
└─────────────┴──────────────────────┘
```

## Группы, workspace и проекты

Современная структура контента сообщества:

- **Group** — команда и governance-скоуп. Владеет workspace и проектами.
- **Workspace** — постоянная рабочая область: issues, discussions, documents.
- **Project** — представление/план над issues, не контейнер.
- **Issue** — принадлежит одному workspace и может входить в несколько проектов.

У workspace один владелец: группа или сообщество. Доступ нескольким группам выдаётся через
`WorkspaceAccess` grants, а не через нескольких владельцев.

### Пример: Platform community

| Группа | Workspace | Repo |
|---|---|---|
| Platform Core | Platform Core, Docs | `tribe`, `tribe-docs` |
| Product & Community | Product & UX, Community Operations | `tribe` (`apps/tribe-front`) |
| Agents & Infrastructure | Agents, Infrastructure | `tribe-agent`, `tribe` (`deploy`) |
| community | Requests, Guarantee Engine | — |

Проекты: `Platform Roadmap`, `Release Train`, `Technical Debt`, `Docs Roadmap`,
`Product Discovery`, `Onboarding`, `Agent MVP`, `MCP & Integrations`,
`Deploy & Reliability`, `Guarantee Engine`, `Support Module`.

`source_workspace_ids` ограничивает workspace-источники проекта.
`collaborator_group_ids` даёт доступ группам-партнёрам без совладения.

## Технически

- SPA (React / Vue / Solid)
- Адаптивный (мобилка + десктоп)
- Тёмная / светлая тема
- Мультиязычность (подстраивается под регион сообщества)
- White-label (каждое сообщество может кастомизировать цвета и логотип)

## Связь с модулями

Каждая страница в платформе = один модуль. Фронтенд получает список активных модулей из пресета и строит навигацию динамически. Если модуль отключён — страница не показывается.
