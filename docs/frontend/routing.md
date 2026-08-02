# Маршруты (Screen Map)

## Текущие маршруты

| Путь | Компонент | Auth | Описание |
|---|---|---|---|
| `/login` | `Login.vue` | Нет | Форма email + password → `login` mutation → JWT в localStorage |
| `/register` | `Register.vue` | Нет | Форма email + password + name → `register` mutation |
| `/communities` | `CommunitiesList.vue` | Да | Список community, create, join |
| `/community/:communityId` | `CommunityHome.vue` | Да | Детальная страница сообщества (members, workspaces, boards) |
| `/community/:communityId/workspaces` | `WorkspacesList.vue` | Да | Список workspaces, create |
| `/community/:communityId/workspaces/:workspaceId` | `WorkspaceDetail.vue` | Да | Workspace + issues (create, assign, status change) |
| `/community/:communityId/issues/:issueId` | `IssueDetail.vue` | Да | Детали issue + комментарии |
| `/community/:communityId/boards/:boardId` | `BoardView.vue` | Да | Канбан-доска с колонками (DEPRECATED) |

## Guards

- `beforeEach` в router: проверка токена, редирект на `/login` если нет
- `beforeEach` для guest-маршрутов: редирект на `/communities` если токен есть
