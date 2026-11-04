# Сценарии

Один файл — три артефакта: пошаговый гайд модуля, e2e-прогон на стенде и раскадровка ролика. Сценарий описывает то, что человек делает руками; кода он не знает.

```
tribe-docs/docs/scenarios/<module>.yaml
        │
        ├── tribe-docs/docs/modules/<module>.md      блок «Сценарии» в гайде
        ├── apps/tribe-front/tests/scenarios          прогон в браузере
        └── tribe-video/storyboards/<id>.md|json     раскадровка ролика
```

## Зачем

Кнопки переименовывают, гайды устаревают молча. Здесь шаг записан один раз: если кнопка пропала, падает прогон, а не гайд. Метка берётся из i18n-ключа, поэтому гайд и тест всегда про одно и то же слово.

## Формат

```yaml
id: treasury-deposit            # kebab-case, совпадает с именем файла
module: treasury                # id из src/tribe_engine/modules/definitions
title: Пополнить казну
goal: Что показывает сценарий и зачем он нужен
route: /community/:id/treasury  # начинается с маршрута модуля

preconditions:                  # что должно быть готово на стенде
  - модуль «Казна» включён в сообществе

prepare:                        # необязательно: данные, которые создаёт сам прогон
  - query: |
      mutation ($input: CreateSupportRequestInput!) {
        createSupportRequest(input: $input) { id title }
      }
    variables:
      input:
        communityId: "{{community_id}}"
        title: "Сквозной сценарий {{now}}"

steps:
  - do: click
    target: { i18n: treasury.deposit }   # ключ из ru.json — метка и селектор сразу
    expect: { i18n: treasury.amountPlaceholder }
    guide: Нажмите «Внести в казну»       # строка для гайда, пишется руками
    caption: Казна — общий счёт сообщества # титр для видео, необязательно
```

### Действия

| `do` | Что делает |
|---|---|
| `goto` | переходит по `path` |
| `click` | кликает по кнопке, ссылке или тексту с меткой |
| `fill` | заполняет поле: `value` |
| `select` | выбирает в списке: `value` |
| `expect_text` | ждёт, что метка появилась на экране |

### Метки

| `target` | Когда |
|---|---|
| `{ i18n: ключ }` | по умолчанию: метка из `apps/tribe-front/src/i18n/locales/ru.json` |
| `{ text: "..." }` | текст без ключа, например заголовок созданной сущности |
| `{ css: ".column__add-btn" }` | когда у элемента нет ни текста, ни test id |
| `{ testid: "..." }` | когда в вьюхе есть `data-testid` |

`within: dialog` сужает поиск до открытого модального окна, `index` берёт второй-третий элемент (по умолчанию первый).

### Подстановки

`{{now}}` — метка запуска, `{{today}}` — дата, `{{community}}` — слаг сообщества, `{{community_id}}` — его id (нужен в GraphQL, где ждут id).

## Как прогнать

```bash
make check-scenarios    # схема, ключи i18n, маршруты модулей
make test-scenarios     # вход demo@local.test на localhost:5175 и проход шагов
make storyboards        # раскадровки в tribe-video/storyboards
```

Стенд по умолчанию — локальный (`http://localhost:5175`, сообщество `onboarding-demo`). Другой стенд задаётся переменными `SCENARIO_BASE_URL`, `SCENARIO_COMMUNITY`, `SCENARIO_EMAIL`, `SCENARIO_PASSWORD`.

Отчёт прогона — `apps/tribe-front/tests/scenarios/report.json` (в git не идёт). Гайд помечается «проверено на стенде» только по зелёному отчёту.

## Правила

- Сценарий повторяемый: нужные данные создаёт `prepare`, а не человек перед прогоном.
- В сценарии нет бизнес-логики и API-вызовов, кроме `prepare`.
- `guide` пишется для человека, а не пересказывает `target`.
- Если шаг нельзя проверить в UI — его нет в сценарии; про такое пишут в гайде словами.
