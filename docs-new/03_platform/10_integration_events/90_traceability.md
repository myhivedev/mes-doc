---
id: DOC-03-10-90
title: 'Трассировка и открытые решения — Integration Events'
type: traceability
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: integration_events
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed
---

# Трассировка и открытые решения — Integration Events

## 1. Назначение документа

Документ показывает, какие сведения о Integration Events подтверждены текущим
кодом и тестами, что перенесено из старого материала, а какие вопросы не закрыты.

## 2. Источники и требования

| Источник | Что проверено | Ревизия или состояние |
| --- | --- | --- |
| `src/Platform/DMP.Platform.IntegrationEvents` | Envelope, порты, сущность outbox, хранение, фоновый обработчик, операторский сервис и DI | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| `src/Platform/DMP.Platform.Contracts/IntegrationEvents` | Типизированные payload | Та же ревизия |
| `src/Platform/DMP.Platform.TenantSecurity`, `Configuration`, `Workflow`, `Runtime` | Модули-источники и найденные обработчики | Та же ревизия |
| `src/Hosts/DMP.Platform.Api/Composition` | Регистрация Integration Events и обработчиков, инициализатор запуска | Та же ревизия |
| `tests/DMP.Platform.IntegrationTests` | Постоянный outbox, поведение модулей-источников и проверки событий | Та же ревизия |
| `docs/01 sources/11_integration_event_platform.md` | Требования к envelope, tenant/correlation, outbox, повторным попыткам и идемпотентности | Старый источник; используется для проверки покрытия |
| `docs/01 sources/11_1_integration_capability_architecture.md` | Граница будущей Integration Capability: внешние системы, соединители, потоки, сопоставления и состояние выполнения | Старый источник перенесён в [backlog будущей Integration Capability](../../10_backlog/platform_capabilities/integration_capability_backlog.md); не является текущим контрактом Integration Events |
| `docs/adr/drafts/ADR-0014-workflow-init-via-platform-outbox.md` | Целевое решение для workflow initialization через общий outbox | `Accepted`, но отдельные гарантии требуют сверки с кодом |
| `docs/adr/drafts/ADR-010-persistent-audit-workflow-history-outbox.md` | Целевая транзакционная граница и запрет in-memory в production | `Draft`; не заменяет подтверждение кодом |
| `docs/architecture/module-integration-rules.md` | Общие правила API/events, запрет module-local queues и cross-service storage | Проектное правило для проверки границы |
| `docs/10_backlog/roadmap/preparation/07_platform_gap_backlog.md` | Разрыв между имеющимся outbox и будущей Integration Capability | Старый backlog-источник; детализация перенесена в [backlog будущей Integration Capability](../../10_backlog/platform_capabilities/integration_capability_backlog.md) |
| Переходный материал `07_event_foundation/offers/11_integration_event_platform.md (01 sources).md` | Предыдущая концепция Event Foundation | Закрыт после переноса маршрута в этот документ; текущим контрактом является `10_integration_events` |
| `docs-new/02_architecture/07_integration_architecture.md` | Общая карта взаимодействий | Используется только навигационно |

## 3. Принятые решения

### Подтверждённое MVP

| Тема | Подтверждённое состояние | Где описано |
| --- | --- | --- |
| Event envelope | Четыре metadata-поля и object payload | `02_architecture.md`, `03_contracts.md` |
| Постоянный outbox | `OutboxMessage` в `integration_events.outbox_messages` | `02_architecture.md`, `04_runtime.md` |
| Локальная обработка | Обработчик выбирается по `EventTypeCode` в host | `03_contracts.md`, `04_runtime.md` |
| Повторные попытки и dead letter | 10 попыток, 30 секунд, диагностические даты и ошибка | `04_runtime.md`, `08_operations.md` |
| Операторский порт | Получение записей dead letter, повторная постановка и подтверждение | `03_contracts.md`, `08_operations.md` |
| Текущие модули-источники | Tenant/Security, Object Runtime и Workflow | `03_contracts.md` |

## 4. Расхождения и открытые решения

| ID | Тема | Ожидание или источник | Текущее подтверждённое состояние | Расхождение или неопределённость | Влияние на текущую документацию | Статус сведения | Владелец / следующий шаг | Документ для обновления после решения |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `IE-DEC-01` | Транзакционная граница outbox | Сохранение состояния модуля и события должно иметь общую транзакционную границу. | Outbox и бизнес-сохранение не подтверждены общей транзакцией. | Нельзя утверждать атомарное сохранение состояния и события. | Будущая доработка | Будущая доработка | Architecture и владельцы модулей-источников: отдельно проверить транзакционную границу и тесты. | `02_architecture.md`, `04_runtime.md`, `07_quality.md` |
| `IE-DEC-02` | Дедупликация потребителя | Потребитель должен защищать побочные эффекты от повторной доставки. | Отдельные inbox-модель и идентификатор события не найдены. | Нельзя обещать однократную обработку. | Будущая доработка | Будущая доработка | Integration Events и владельцы потребителей: определить политику дедупликации. | `03_contracts.md`, `04_runtime.md` |
| `IE-DEC-03` | Несколько экземпляров фонового обработчика | Несколько экземпляров обработчика должны безопасно распределять записи. | Механизм распределения и блокировки не подтверждён. | Нельзя обещать безопасную параллельную обработку. | Будущая доработка | Будущая доработка | Integration Events и Operations: провести проверку для используемого провайдера хранения. | `02_architecture.md`, `07_quality.md`, `08_operations.md` |
| `IE-DEC-04` | Версионирование событий | Схема payload должна изменяться совместимо. | `PayloadVersion` и общий реестр версий не найдены. | Совместимость payload не установлена. | Будущая доработка | Будущая доработка | Architecture и владельцы событий: определить каталог событий и правила совместимости. | `03_contracts.md`, `90_traceability.md` |
| `IE-DEC-05` | Внешняя доставка | События должны доставляться через внешний транспорт. | Брокер и внешняя доставка текущим кодом не подтверждены. | Локальный outbox не является внешней интеграцией. | Будущая доработка | Будущая доработка | Архитектура: определить отдельную область Integration Capability. | `01_scope.md`, `03_contracts.md` |
| `IE-DEC-06` | Операторский API и интерфейс | Оператору могут потребоваться HTTP API и UI для работы с dead letter. | Application-порт есть, HTTP-маршрут и UI не найдены. | Операторский UI нельзя объявлять готовым. | Будущая доработка | Будущая доработка | Host и фронтенд-платформа: определить API и владельца интерфейса. | `03_contracts.md`, `08_operations.md` |
| `IE-DEC-07` | Типизированные payload | Каждое публикуемое событие может иметь отдельную типизированную схему. | Только часть событий имеет DTO; часть создаётся непосредственно в модуле-источнике. | Встроенная полезная нагрузка не объявляется стабильным общим контрактом. | Не блокирует текущую документацию | Подтверждено, но ограничено | Владелец каждого события: решить, нужен ли отдельный DTO. | `03_contracts.md` |
| `IE-DEC-08` | Идемпотентность инициализации Workflow | `ADR-0014` требует идемпотентной инициализации по идентичности объекта и привязке Workflow. | Отдельная inbox-модель и дедупликация в текущем коде не найдены. | Идемпотентность нельзя объявлять гарантией Integration Events. | Будущая доработка | Будущая доработка | Integration Events, Workflow и Object Runtime: определить механизм дедупликации. | `04_workflow/04_runtime.md`, `04_runtime.md` |
| `IE-DEC-09` | Транзакционная граница из `ADR-010` | `ADR-010` требует записи outbox в общей границе изменения до фиксации. | Текущий издатель сохраняет запись через отдельный `DbContext`. | Требование ADR не подтверждено текущей реализацией. | Будущая доработка | Будущая доработка | Архитектура и владельцы модулей-источников: проверить дизайн транзакции и приёмочные тесты. | `02_architecture.md`, `07_quality.md` |
| `IE-DEC-10` | `TenantId` и `CorrelationId` у источника | Для события должны быть понятны tenant-контекст и корреляция. | Поля есть в envelope/entity; `TenantId` допускает `null`, централизованная проверка источника не найдена. | Нельзя обещать полноту служебных полей для каждого события. | Не блокирует текущую документацию | Подтверждено, но ограничено | Foundation и владельцы событий: уточнить правило для tenant-scoped событий. | `03_contracts.md`, `04_runtime.md` |
| `IE-DEC-11` | Публичная идентичность payload | Старый источник требует стабильные коды или внешние идентификаторы вместо внутренних ID. | `TenantCreatedIntegrationEventPayload` содержит `Guid Id`. | Семантика `Id` и единое правило внешней идентичности не согласованы. | Не блокирует текущую документацию | Не блокирует текущую документацию | Владельцы событий и архитектура: проверить идентичность каждого payload. | `03_contracts.md`, `04_runtime.md` |
| `IE-DEC-12` | Несколько потребителей | Одно событие может обрабатываться несколькими независимыми потребителями. | Фоновый обработчик выбирает один обработчик по `EventTypeCode`; распределение по нескольким потребителям не подтверждено. | Нельзя обещать доставку нескольким потребителям. | Будущая доработка | Будущая доработка | Integration Events и архитектура: определить распределение по нескольким потребителям или явную маршрутизацию в одном обработчике. | `03_contracts.md`, `04_runtime.md` |
| `IE-DEC-13` | Срок хранения и очистка | Обработанные и dead letter-записи должны храниться по определённой политике. | Политика хранения и очистки не найдена. | Нельзя обещать автоматическую очистку и срок хранения. | Будущая доработка | Будущая доработка | Integration Events и Operations: определить хранение, архивирование и очистку. | `02_architecture.md`, `08_operations.md` |
| `IE-DEC-14` | Провайдер в памяти в эксплуатации | Production должен использовать постоянное хранилище. | Издатель в памяти доступен в проекте; регистрация production использует постоянный издатель. | Необходимо явно проверить конфигурации host-приложений. | Не блокирует текущую документацию | Подтверждено, но ограничено | Host и Operations: закрепить правило и проверить конфигурации. | `08_operations.md`, `07_quality.md` |

## 5. Маршрут в целевые документы

### Маршрут старого материала

| Содержание старого материала | Куда направлено | Текущий статус |
| --- | --- | --- |
| Envelope и metadata | `02_architecture.md`, `03_contracts.md` | Перенесено в подтверждённой части |
| Постоянный outbox, повторные попытки и dead letter | `02_architecture.md`, `04_runtime.md`, `08_operations.md` | Перенесено по текущему коду |
| Inbox/idempotency | `04_runtime.md`, раздел 4 этого документа | Не подтверждено; будущая доработка |
| External systems, connectors, flows и mappings | [Backlog будущей Integration Capability](../../10_backlog/platform_capabilities/integration_capability_backlog.md) | Не входит в область |
| Выбор MassTransit, Wolverine, NServiceBus или Rebus | Не переносится в текущий контракт | Будущий архитектурный выбор, не решение MVP |
| Корпоративный транспорт, воспроизведение, реестр версий и срок хранения | `04_runtime.md`, `07_quality.md`, этот документ | Открытые будущие темы |

### Маршрут в целевые документы

| Тема | Документ-владелец |
| --- | --- |
| Общая карта взаимодействий | `../../02_architecture/07_integration_architecture.md` |
| Семантика `TenantCreated` и Tenant scope | `../01_tenant_and_security/04_runtime.md`, `../02_configuration/04_runtime.md` |
| Семантика mutation event | `../03_object_runtime/04_runtime.md` |
| Семантика workflow event | `../04_workflow/04_runtime.md` |
| Audit record | `../09_audit_history/02_architecture.md` |
| Общие application-контракты | `../00_foundation/03_contracts.md` |
| Внешняя интеграция | [Backlog будущей Integration Capability](../../10_backlog/platform_capabilities/integration_capability_backlog.md) |

Технический ID `IE-DEC-01` и подобные идентификаторы здесь не используются без
понятного названия темы. Открытые вопросы описаны таблицей и не считаются
решёнными только потому, что механизм outbox уже существует.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
