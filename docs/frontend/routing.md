# Маршруты (Screen Map)

## Текущие маршруты

| Путь | Компонент | Auth | Описание |
|---|---|---|---|
| `/login` | `Login.vue` | Нет | Форма email + password → `login` mutation → JWT в localStorage |
| `/register` | `Register.vue` | Нет | Форма email + password + name → `register` mutation |
| `/communities` | `CommunitiesList.vue` | Да | Список community, create, join |
| `/community/:communityId` | `CommunityHome.vue` | Да | Детальная страница сообщества (members, projects) |
| `/community/:communityId/projects` | `ProjectsView.vue` | Да | Список проектов, create |
| `/community/:communityId/projects/:projectId` | `ProjectDetail.vue` | Да | Проект + задачи (create, assign, complete) |

## Guards

- `beforeEach` в router: проверка токена, редирект на `/login` если нет
- `beforeEach` для guest-маршрутов: редирект на `/communities` если токен есть
