# Frontend

## Стек

| Технология | Версия | Назначение |
|---|---|---|
| Vue 3 | ^3.5 | UI-фреймворк (Composition API, script setup) |
| Vue Router | ^4.5 | Клиентская маршрутизация |
| Vite | ^6.2 | Сборщик / dev-сервер |
| TypeScript | ^5.7 | Типизация |

## Репозиторий

Фронтенд живёт в `apps/tribe-front/` в монорепе `tribe-engine`.

```
apps/tribe-front/
├── index.html
├── package.json
├── vite.config.ts           # proxy /graphql → localhost:8000
├── tsconfig.json
└── src/
    ├── main.ts              # точка входа
    ├── App.vue              # switch лейаутов по route.meta.layout
    ├── router.ts            # маршруты + beforeEach guard
    ├── layouts/
    │   ├── GuestLayout.vue  # центрированная карточка (login/register)
    │   └── AuthLayout.vue   # левая панель (profile + community) + контент
    ├── api/
    │   ├── graphql.ts       # gql() — универсальный fetch к /graphql
    │   └── types.ts         # TypeScript-типы
    └── views/
        ├── Login.vue
        ├── Register.vue
        ├── CommunitiesList.vue
        ├── CommunityHome.vue
        ├── ProjectsView.vue
        └── ProjectDetail.vue
```

## Документация

- [UI Kit](ui-kit.md) — компоненты, конвенции, план развития
- [Маршруты](routing.md) — screen map, guards
- [Лейауты](layouts.md) — иерархия обёрток
