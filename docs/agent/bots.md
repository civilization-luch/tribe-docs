# Боты (AI-участники сообществ)

> **Статус: черновик концепции.** Детали уточняются на этапе реализации.

## Концепция

Бот — это полноценный участник сообщества, за которым стоит AI, а не человек.
Технически бот — это **User с флагом `is_bot=true`**.
Member, JWT, роли, репутация — всё идентично человеку.

Внешние каналы (Telegram, Discord, MAX) — см. [Боты в platform-ecosystem.md](../architecture/platform-ecosystem.md#Боты).

## Модель

```
User ──┬── is_bot: bool
       ├── bot_type: string (classifier / executor / custom / ...)
       └── ...стандартные поля User

Member ──┬── metadata: JSONB {
             "bot": {
               "enabled": true,
               "type": "classifier",
               "prompt": "...",
               "model": { "provider": "...", "id": "..." },
               "api_keys": { "openai": "sk-...", "github": "ghp_..." },
                "tools": ["search_issues", "create_issue", ...]
             },
             "nickname": "Helper Bot",
             "custom": {}
           }
```

Конфигурация в `Member.metadata` — один бот может иметь разную конфигурацию
в разных сообществах (промпт, ключи, модель).

## Типы ботов

| bot_type | Роль | Создаётся |
|---|---|---|
| `classifier` | Pipeline 1: Request → Issue | Системный (seed) |
| `executor` | Pipeline 2: Issue → PR/review | Системный (seed) |
| `self-healer` | Pipeline 3: метрики → Issue | Системный (seed) |
| `custom` | Пользовательский AI-бот | Через UI сообщества |

`bot_type` — свободная строка, без строгой валидации на уровне модели.

## Жизненный цикл

### Поток А: создать бота через панель сообщества

```
1. Админ → "Создать бота" (имя, тип, опционально промпт)
2. Система:
   a. Регистрирует User(id, name, is_bot=true, bot_type, ...)
   b. Генерирует долгоживущий JWT (bot_token)
   c. Создаёт Member(user_id, community_id, status=ACTIVE, metadata)
   d. Возвращает bot_token (показывается один раз)
3. Бот готов: состоит в сообществе, может вызывать API
```

### Поток Б: подключить существующего бота

```
1. Админ → "Подключить бота" → вводит ID бота
2. Система:
   a. Проверяет, что User с таким ID существует и is_bot=true
   b. Создаёт Member + metadata с дефолтными настройками
3. Админ настраивает промпт, ключи, модель
```

### Seed-боты при первом запуске

При первом запуске Coordinator проверяет, существует ли в БД
бот `classifier` (по `bot_type`). Если нет — регистрирует через
GraphQL mutation и сохраняет токен в конфигурацию.

## GraphQL API

```graphql
# UserType расширяется:
type UserType {
  id: String!
  email: String!
  name: String!
  isBot: Boolean!
  botType: String
  createdAt: DateTime!
}

# Новые мутации:
type Mutation {
  createBot(input: CreateBotInput!): CreateBotPayload!
  updateBotConfig(memberId: String!, input: BotConfigInput!): MemberType!
  regenerateBotToken(userId: String!): String!
}

input CreateBotInput {
  communityId: String!
  name: String!
  botType: String!
  prompt: String
}

type CreateBotPayload {
  user: UserType!
  member: MemberType!
  botToken: String!
}

input BotConfigInput {
  enabled: Boolean
  prompt: String
  modelProvider: String
  modelId: String
  apiKeys: JSON
  tools: [String!]
}

# Новые запросы:
type Query {
  communityBots(communityId: String!): [MemberType!]!
  botUser(userId: String!): UserType
}
```

## Аутентификация

Долгоживущий JWT (access token с expiry ~365 дней) как Bearer-токен
в `Authorization` header — ровно как и человек.

Отличие: бот не логинится через `login(email, password)` — у него
может не быть пароля. Только через готовый токен.

## UI

Вкладка `/community/:communityId/bots`:
- Список ботов сообщества (имя, тип, статус)
- Кнопка "Создать бота" → модалка (имя, тип, промпт)
- Для каждого бота:
  - Переключатель enabled
  - Раскрываемая панель: промпт (textarea), модель, API keys (key-value)
  - Кнопки: "Скопировать токен", "Отключить", "Удалить из сообщества"

## Боты (AI) vs Внешние боты

| | AI-бот (User.is_bot) | Внешний бот (Telegram/Discord) |
|---|---|---|
| Сущность | User в системе | Отдельный сервис, webhook |
| В сообществе | Member с правами | Нет (внешний канал) |
| Роль | AI-обработка внутри | Коммуникация с пользователем |
| API | GraphQL (как участник) | MCP или REST |
| Пример | classifier, executor | CoLive-Telegram-bot |
