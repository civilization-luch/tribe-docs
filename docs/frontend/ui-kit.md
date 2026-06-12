# UI Kit

## Текущие компоненты

Минимальный набор — всё в `src/views/`.

| Компонент | Файл | Роль |
|---|---|---|
| `App.vue` | `src/App.vue` | Навбар + `<router-view>`, logout, глобальные стили |
| `Login.vue` | `src/views/Login.vue` | Форма email + password → `login` mutation |
| `Register.vue` | `src/views/Register.vue` | Форма email + password + name → `register` mutation |
| `Communities.vue` | `src/views/Communities.vue` | Список community, create, join, members |

## Конвенции

- **Composition API** + `<script setup lang="ts">` — обязательно
- **Scoped styles** — `<style scoped>`, глобальные стили только в `App.vue`
- **Нейминг** — multi-word, PascalCase для компонентов, kebab-case в шаблонах
- **Типизация** — все GraphQL-ответы типизированы через `src/api/types.ts`
- **API** — единый `gql(query, variables?)` из `src/api/graphql.ts`, JWT в `localStorage`
- **Стилизация** — plain CSS, без препроцессоров и CSS-in-JS (пока)

## План развития

### Слой shared/ui (общеиспользуемые примитивы)

```
src/
└── shared/
    └── ui/
        ├── AppButton.vue
        ├── AppInput.vue
        ├── AppCard.vue
        ├── AppModal.vue
        ├── StatusBadge.vue
        ├── EmptyState.vue
        └── index.ts
```

### Слой widgets (композитные блоки)

```
src/
└── widgets/
    ├── CommunityCard.vue
    ├── MemberRow.vue
    ├── TaskCard.vue
    ├── VoteProgress.vue
    └── AgentChat.vue
```

### Слой features (действия пользователя)

```
src/
└── features/
    ├── auth/         # LoginForm.vue, RegisterForm.vue
    ├── community/    # CreateCommunityForm.vue, JoinButton.vue
    ├── tasks/        # CreateTaskForm.vue, TaskAssigneeSelect.vue
    └── agent/        # AgentMessageInput.vue, AgentMessageBubble.vue
```

### Слой pages (страницы)

```
src/
└── pages/
    ├── LoginPage.vue
    ├── RegisterPage.vue
    ├── CommunitiesPage.vue
    ├── CommunityHomePage.vue
    └── ...
```

## Feature-Sliced Design (FSD)

Рекомендуемая архитектура на будущее — [Feature-Sliced Design](https://feature-sliced.design/):

```
src/
├── app/          # инициализация, роутер, store
├── shared/       # UI-примитивы, утилиты, API-клиент
├── entities/     # бизнес-сущности (Community, Member, Task, Proposal)
├── features/     # действия пользователя
├── widgets/      # композитные блоки
└── pages/        # страницы
```

Текущая структура плоская (`views/`, `api/`) — миграция на FSD по мере роста.
