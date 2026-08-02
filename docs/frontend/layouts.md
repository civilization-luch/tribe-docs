# Лейауты

## Текущая структура

```
App.vue (switch по route.meta.layout)
├── GuestLayout        # центрированная карточка (Login, Register)
└── AuthLayout         # авторизованный пользователь
    ├── Sidebar        # левая панель
    │   ├── User       # email, logout, My Communities
    │   └── Community  # только когда communityId в URL
    │                  #   Home, Workspaces
    └── <router-view>  # контент страницы справа
```

## Левая панель (AuthLayout)

```
┌──────────────────────┐
│  ▓ USER              │
│  email@example.com   │
│  [Logout]            │
│  ─────────────────── │
│  My Communities →    │
│                      │
│  ▓ COMMUNITY         │  ← только когда communityId в URL
│  Название            │
│  ── Home             │
│  ── Workspaces       │
└──────────────────────┘
```

## Маршруты и лейауты

| Путь | Компонент | Лейаут | Community context |
|---|---|---|---|
| `/login` | `Login.vue` | Guest | — |
| `/register` | `Register.vue` | Guest | — |
| `/` | → redirect `/communities` | — | — |
| `/communities` | `CommunitiesList.vue` | Auth | Нет |
| `/community/:communityId` | `CommunityHome.vue` | Auth | Да |
| `/community/:communityId/workspaces` | `WorkspacesList.vue` | Auth | Да |
| `/community/:communityId/workspaces/:workspaceId` | `WorkspaceDetail.vue` | Auth | Да |
| `/community/:communityId/issues/:issueId` | `IssueDetail.vue` | Auth | Да |
| `/community/:communityId/boards/:boardId` | `BoardView.vue` | Auth | Да | (DEPRECATED) |


## План развития

### Компоненты лейаутов

```
AppLayout
├── GuestLayout            # центрированная форма (Login, Register)
└── AuthLayout             # авторизованный пользователь
    ├── AppHeader          # бренд, поиск, уведомления, профиль
    ├── Sidebar            # навигация по сообществам
    └── <router-view />    # контент страницы
        └── CommunityLayout   # внутри сообщества
            ├── CommunityHeader  # название, описание, actions
            ├── CommunityTabs    # Workspaces / Issues / Boards / Members
            └── <router-view />  # контент таба
```

### Назначение

| Компонент | Описание |
|---|---|
| `AppLayout.vue` | Корневой лейаут, проверка токена, редирект |
| `GuestLayout.vue` | Центрированная карточка для форм (логин, регистрация, восстановление пароля) |
| `AuthLayout.vue` | Хедер + сайдбар + контент |
| `CommunityLayout.vue` | Шапка сообщества + табы + контент таба |

### Конвенции

- Лейауты не содержат бизнес-логики — только разметка и стили
- Бизнес-логика в `views/` и `composables/`
- `CommunityLayout` получает `communityId` из `route.params.slug` и подгружает данные
- Переход между лейаутами через `router.beforeEach` + `meta.layout`
