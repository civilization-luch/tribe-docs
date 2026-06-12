# Лейауты

## Текущая структура

```
App.vue                          # корень: навбар (email + logout) + <router-view>
├── GuestLayout (не выделен)     # Login.vue, Register.vue
└── AuthLayout (не выделен)      # Communities.vue — навбар с email/logout
```

Сейчас App.vue выполняет роль и гостевого, и авторизованного лейаута — навбар скрыт, если нет токена.

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
            ├── CommunityTabs    # Tasks / Projects / Agent / Members
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
