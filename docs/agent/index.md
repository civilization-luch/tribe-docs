# Agent

Per-community сервис. Подробно — в [platform-ecosystem.md](../architecture/platform-ecosystem.md).

## Схема работы

```
Community Event Bus
    ↓ (TaskAssigned / RequestCreated / MetricAnomaly)
Agent Coordinator  ← конфигурация сообщества (LLM API key, роль)
    │
    ├── Pipeline 1 → LLM API (classify / dedup / enrich → create Task)
    │
    ├── Pipeline 2 → opencode --mcp community-mcp (Task → analyze → code → PR)
    │
    └── Pipeline 3 → LLM API + метрики (create Task: self-healing)
```

## Компоненты

| Компонент | Роль |
|---|---|
| **Agent Coordinator** | Event listener, роутер, запуск пайплайнов |
| **Community MCP Server** | MCP-прокси: данные и действия сообщества |
| **Opencode** | Исполнитель Task: анализ кода, генерация PR |
| **LLM API** | Классификация, дедупликация, self-healing |

## Документы

- [Community MCP](community-mcp.md) — MCP-сервер для доступа к сообществу
