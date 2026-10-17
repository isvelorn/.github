# Межрепозиторные контракты · v0.1

Контракты описывают **намерения и критерии приёмки**. Наличие документов не означает работающий API.

| Contract | Producer → Consumer | Где определение |
| --- | --- | --- |
| `content-definition/v1` | world → builder/server/game | [world/data](https://github.com/isvelorn/world/tree/main/data) и [схемы](https://github.com/isvelorn/world/tree/main/schemas) |
| `ai-proposal/v1` | director → валидатор/деплой | [director/schemas/proposal.schema.json](https://github.com/isvelorn/director/blob/main/schemas/proposal.schema.json) 🔒 |
| `telemetry-summary/v1` | server → observatory → director | [observatory schema](https://github.com/isvelorn/observatory/blob/main/schemas/summary.schema.json) 🔒 |
| `scheduled-event/v1` | проверенный пакет → server event queue | [server/docs/event-engine.md](https://github.com/isvelorn/server/blob/main/docs/event-engine.md) 🔒 |
| `delivery-receipt/v1` | server → infrastructure/director | [infrastructure/docs/delivery.md](https://github.com/isvelorn/infrastructure/blob/main/docs/delivery.md) 🔒 |

🔒 — интерфейсы в приватных репозиториях. Публичные файлы не должны ссылаться на приватные детали как на доступные анонимно.

## Идентификаторы и ссылки

- Идентификатор определения стабилен: `namespace.slug`, например `creature.forest_wolf` или `region.north_forest`; rename — операция миграции.
- Публикуемый контент содержит **полный набор разрешённых ссылок**; никаких ссылок на отсутствующие объекты.
- Редакция контента должна указывать Git commit SHA (40 hex), версию схемы, минимальную версию движка, manifest digest.
- При несовместимой смене схемы увеличиваем major и предоставляем миграцию; старый сервер должен отклонить неподдерживаемый пакет.

## Порядок обработки входа

`untrusted snapshot → typed proposal → deterministic policy → sandbox simulation/tests → immutable artifact → authorized apply → durable receipt → reconcile`.

Проверки **не** считаются заменой авторизации. Ни GitHub PR, ни ответ ИИ, ни запись в приватном репозитории сами по себе не являются командой на изменение игрового сервера.

## Две независимые оси версий

- **Content revision** — версия опубликованных определений `world`.
- **Runtime state revision** — последовательность транзакций авторитетной БД.

План с устаревшей телеметрией может не удовлетворять предусловиям: сервер обязан проверить их на **момент атомарного применения**. Для изменения контента применяется сравнение ожидаемой версии контента/базы, а для мировых операций — семантические предикаты, не глобальная блокировка всех тиков.

## Аварийные ситуации

- GitHub/ChatGPT недоступны → мир выполняет уже принятые события.
- Неизвестно, прошла ли запись → запросить receipt по idempotency key; без подтверждения не генерировать новый ключ.
- Одновременные предложения → последовательная публикация, expected revision, rebase/revalidate.
- Снимок старый или скомпрометирован → отклонить или пересчитать; не выводить из него право на действие.
- Новый тип операции не поддерживается → безопасный отказ и Issue для реализации, а не исполнение произвольного кода.
