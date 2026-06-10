# Архитектура платформы

```mermaid
graph TB
    subgraph Presets["Preset Layer"]
        P1["Пресет CoLive<br/>9 модулей"]
        P2["Пресет CoSpace<br/>8 модулей"]
        P3["Пресет CoSettle<br/>7 модулей"]
        P4["Пресет CoGuild<br/>8 модулей"]
        P5["Пресет CoOper<br/>7 модулей"]
    end

    subgraph Engine["Platform Core"]
        CE["Core Engine<br/>API / Auth / RBAC"]
        MR["Module Registry"]
        EE["Event Engine<br/>Notification / Webhook"]
    end

    subgraph Modules["Module Layer"]
        M1["Membership"]
        M2["Billing"]
        M3["Bookings"]
        M4["Tasks"]
        M5["Treasury"]
        M6["Content"]
        M7["Proposals"]
        M8["Reputation"]
        M9["Documents"]
        M10["LMS"]
        M11["Marketplace"]
        M12["Revenue"]
        M13["Registry"]
        M14["Schedule"]
    end

    subgraph Region["Multi-Region Layer"]
        RKZ["Region: Kazakhstan"]
        RVN["Region: Vietnam"]
        RPY["Region: Paraguay"]
    end

    subgraph Infrastructure["Infrastructure"]
        DB[("PostgreSQL")]
        S3[("File Storage")]
        QUEUE[("Message Queue")]
        CDN[("CDN / SPA")]
    end

    P1 --> CE
    P2 --> CE
    P3 --> CE
    P4 --> CE
    P5 --> CE

    CE --> MR
    CE --> EE

    MR --> M1
    MR --> M2
    MR --> M3
    MR --> M4
    MR --> M5
    MR --> M6
    MR --> M7
    MR --> M8
    MR --> M9
    MR --> M10
    MR --> M11
    MR --> M12
    MR --> M13
    MR --> M14

    M2 --> RKZ
    M2 --> RVN
    M2 --> RPY
    M5 --> RKZ
    M5 --> RVN
    M5 --> RPY
    M9 --> RKZ
    M9 --> RVN
    M9 --> RPY

    CE --> DB
    CE --> S3
    CE --> QUEUE
    CE --> CDN
```

## Уровни

### 0. Preset Layer (пресеты)

Пресет — это конфигурация по умолчанию: набор модулей, настройки и модель управления под конкретный кейс. При создании сообщества пользователь выбирает пресет, а затем может донастроить модули.

| Пресет | Модули |
|---|---|
| **CoLive** | Membership, Billing, Bookings, Tasks, Treasury, Content, Proposals, Reputation, Documents |
| **CoSpace** | Membership, Schedule, Bookings, Billing, Content, Proposals, Reputation, Documents |
| **CoSettle** | Membership, Registry, Treasury, Proposals, Tasks, Content, Documents |
| **CoGuild** | Membership, LMS, Tasks, Billing, Content, Proposals, Reputation, Documents |
| **CoOper** | Membership, Registry, Billing, Treasury, Proposals, Content, Marketplace |

### 1. Platform Core

- **Core Engine** — API, авторизация (JWT), роли и права (RBAC)
- **Module Registry** — реестр всех модулей, их версии и зависимости
- **Event Engine** — уведомления, вебхуки, внутренние события

### 2. Module Layer

Каждый модуль — независимый блок с API, хранилищем и настройками. Модули не зависят друг от друга, но могут обмениваться событиями. Все 14 модулей общие для всех пресетов.

### 3. Multi-Region Layer

Абстракция над региональными особенностями: валюта, платёжные системы, юридические шаблоны.

### 4. Infrastructure

PostgreSQL, S3-совместимое хранилище, очередь сообщений, CDN для фронтенда.

## Структура репозитория

```
tribe/
├── core/                    # общее ядро (API, auth, RBAC)
├── modules/                 # все модули
│   ├── membership/
│   ├── billing/
│   ├── treasury/
│   └── ...
├── presets/                 # конфиги пресетов
│   ├── colive.json
│   ├── cospace.json
│   ├── cosettle.json
│   ├── coguild.json
│   └── cooper.json
├── apps/                    # фронтенды / мобилки
├── docs/                    # документация
└── deployments/             # инфраструктура
```
