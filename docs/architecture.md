# Архитектура платформы

```mermaid
graph TB
    subgraph Сообщества["Community Layer"]
        CA["Коливинг Алматы"]
        CB["Коливинг Дананг"]
        CC["Коливинг Парагвай"]
        CN["Кооператив / Клуб / ..."]
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
    end

    subgraph Region["Multi-Region Layer"]
        RKZ["Region: Kazakhstan<br/>Тенге / Kaspi / KZ law"]
        RVN["Region: Vietnam<br/>VND / Local banking / VN law"]
        RPY["Region: Paraguay<br/>PYG / Transfers / PY law"]
    end

    subgraph Infrastructure["Infrastructure"]
        DB[("PostgreSQL")]
        S3[("File Storage<br/>Docs / Images")]
        QUEUE[("Message Queue")]
        CDN[("CDN / SPA")]
    end

    CA --> CE
    CB --> CE
    CC --> CE
    CN --> CE

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

### 1. Community Layer

Сообщества подключаются к платформе. Каждое сообщество выбирает:
- регион (юрисдикцию)
- набор модулей
- модель управления

### 2. Platform Core

- Core Engine — API, авторизация (JWT), роли и права (RBAC)
- Module Registry — реестр активных модулей, их настройки и версии
- Event Engine — уведомления, вебхуки, внутренние события

### 3. Module Layer

Каждый модуль — независимый блок с API, хранилищем и настройками. Модули не зависят друг от друга, но могут обмениваться событиями.

### 4. Multi-Region Layer

Абстракция над региональными особенностями:
- **KZ**: тенге, Kaspi Pay, юридические шаблоны по КЗ РБ
- **VN**: VND, местные платёжные системы, право Вьетнама
- **PY**: парагвайский гуарани, банковские переводы, право Парагвая

Каждый модуль, которому нужна региональная логика (Billing, Treasury, Documents), обращается к этому слою.

### 5. Infrastructure

PostgreSQL, S3-совместимое хранилище, очередь сообщений, CDN для фронтенда.
