# Agent

Per-community сервис на базе [opencode](https://opencode.ai). Подробно — в [platform-ecosystem.md](../architecture/platform-ecosystem.md).

## Архитектура

```
Community Event Bus
    ↓ (TaskAssigned / RequestCreated / MetricAnomaly)
Thin Coordinator (~100 строк)
    ↓ POST /session/:id/message
opencode serve (per-community инстанс)
  ├── Agents (кастомные роли)
  ├── Skills (инструкции для пайплайнов)
  └── MCP → Community MCP Server (данные сообщества)
    ↓
Результат → update_task_status() / create_task()
```

Opencode serve — headless HTTP-сервер. Всю работу делает он: LLM-вызовы, MCP-инструменты, контекст, агенты. Нам остаётся только сконфигурировать agents, skills, MCP и написать коннектор событий.

## Компоненты

| Компонент | Роль |
|---|---|
| **Thin Coordinator** | Event listener → HTTP-запрос к opencode serve |
| **Opencode serve** | Headless HTTP-сервер: агенты, LLM, MCP |
| **Agents** | Кастомные роли (classifier, executor, self-healer) |
| **Skills** | SKILL.md — инструкции для каждого пайплайна |
| **Community MCP Server** | Remote MCP: данные и действия сообщества |

## Пайплайны как Agents + Skills

| Пайплайн | Agent | Skill |
|---|---|---|
| **Pipeline 1: Request → Task** | `@classifier` | `classify-request` |
| **Pipeline 2: Task Executor** | `@task-executor` | `analyze-code` |
| **Pipeline 3: Self-healing** | `@self-healer` | `self-heal` |

Каждый agent — своя роль, свой prompt, своя модель.

## Opencode serve настройка

Запуск per-community:

```bash
OPENCODE_SERVER_PASSWORD=community-token opencode serve --port 4096
```

Opencode.jsonc конфигурация Community MCP:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "community": {
      "type": "remote",
      "url": "https://tribe.example.com/mcp",
      "headers": {
        "Authorization": "Bearer {env:BOT_TOKEN}"
      },
      "enabled": true
    }
  },
  "agent": {
    "classifier": {
      "description": "Классифицирует Request, ищет дубликаты, создаёт Task",
      "mode": "subagent",
      "temperature": 0.1,
      "prompt": "{file:./prompts/classifier.txt}"
    },
    "task-executor": {
      "description": "Анализирует код, генерирует PR по Task",
      "mode": "subagent",
      "prompt": "{file:./prompts/task-executor.txt}"
    },
    "self-healer": {
      "description": "Анализирует метрики и создаёт Task при аномалиях",
      "mode": "subagent",
      "temperature": 0.2,
      "prompt": "{file:./prompts/self-healer.txt}"
    }
  }
}
```

## Thin Coordinator

Самый простой компонент — слушает Event Bus и шлёт HTTP-запросы к `opencode serve`.

```python
# Псевдокод: обработчик события TaskAssigned
def on_task_assigned(event):
    session = requests.post("http://localhost:4096/session", json={
        "title": f"Execute: {event.task_id}"
    }).json()

    requests.post(f"http://localhost:4096/session/{session['id']}/message", json={
        "parts": [{"type": "text", "text": f"Выполни задачу {event.task_id}: {event.description}"}],
        "agent": "task-executor"
    })
```

## Skills (SKILL.md)

`.opencode/skills/classify-request/SKILL.md`:
```markdown
---
name: classify-request
description: Классифицирует сырые запросы: баг / фича / вопрос / идея
---
Правила классификации:
- Если есть stack trace или скриншот ошибки → bug
- Если предложение новой возможности → feature
- Если вопрос по использованию → question
- Иначе → idea
После классификации ищи дубликаты через community MCP `search_tasks`.
```

`.opencode/skills/self-heal/SKILL.md`:
```markdown
---
name: self-heal
description: Анализирует метрики сообщества и создаёт Task при аномалиях
---
Правила:
- 0 сделок за 24ч → create_task("Проверить модуль продаж")
- Рост 5xx > 5% → create_task("Исследовать рост ошибок")
- latency > 2s → create_task("Оптимизировать производительность")
```

## Документы

- [Community MCP](community-mcp.md) — MCP-сервер для доступа к сообществу
