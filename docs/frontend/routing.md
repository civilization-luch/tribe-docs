# Маршруты (Screen Map)

## Текущие маршруты

| Путь | Компонент | Auth | Описание |
|---|---|---|---|
| `/login` | `Login.vue` | Нет | Форма email + password → `login` mutation → JWT в localStorage |
| `/register` | `Register.vue` | Нет | Форма email + password + name → `register` mutation |
| `/` | `Communities.vue` | Да | Список community, create, join, просмотр members |

## План развития (Phase 2+)

| Путь | Компонент | Auth | Описание |
|---|---|---|---|
| `/c/:slug` | `CommunityHome.vue` | Да | Профиль сообщества: описание, участники, активность |
| `/c/:slug/tasks` | `TaskList.vue` | Да | Список задач |
| `/c/:slug/projects` | `ProjectList.vue` | Да | Список проектов |
| `/c/:slug/agent` | `AgentChat.vue` | Да | Чат с AI Agent сообщества |
| `/c/:slug/settings` | `CommunitySettings.vue` | Да | Настройки (только admin) |
| `/profile` | `Profile.vue` | Да | Профиль пользователя |
| `/admin` | `AdminPanel.vue` | Да | Глобальная админка (только superadmin) |

## Guards

Сейчас проверка auth идёт на уровне компонента (`localStorage.getItem("token")`). В будущем:

- `beforeEach` в router: проверка токена, редирект на `/login` если нет
- `beforeEach` для `/c/:slug/*`: проверка членства в community через `members(communityId)`
- `beforeEnter` для `/c/:slug/settings`: проверка роли admin через `member(id).role`
