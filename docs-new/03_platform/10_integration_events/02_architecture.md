---
id: DOC-03-10-02
title: 'Архитектура платформенной области — Integration Events'
type: architecture
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

# Архитектура платформенной области — Integration Events

## 1. Назначение документа

Документ описывает архитектурно значимые компоненты Integration Events,
границы владения, envelope и persistence. Внутренний каталог методов и классов
не является целью документа.

## 2. Граница и компоненты

```text
Модуль-источник
  -> IIntegrationEventPublisher
  -> PersistentIntegrationEventPublisher
  -> IntegrationEventsDbContext
  -> OutboxMessage

OutboxProcessingHostedService
  -> зарегистрированный IIntegrationEventProcessorHandler
  -> обработка модулем или host-приложением
```

`AddPlatformIntegrationEvents` регистрирует постоянный издатель, хранилище,
операторский сервис и фоновый обработчик. В тестовом режиме доступны издатель и
хранилище в памяти.

| Компонент | Ответственность | Зависимости | Подтверждение |
| --- | --- | --- | --- |
| Издатель | Сериализует envelope и сохраняет outbox-запись. | Контекст базы Integration Events, сериализатор JSON | `PersistentIntegrationEventPublisher` |
| Фоновый обработчик | Выбирает ожидающие записи и вызывает обработчик. | Outbox-хранилище, реестр обработчиков, фоновая служба | `OutboxProcessingHostedService` |
| Реестр обработчиков | Связывает `EventTypeCode` с обработчиком. | DI host-приложения | `IIntegrationEventProcessorHandler` |
| Операторский сервис | Читает записи dead letter и меняет их состояние. | Outbox-хранилище | `IOutboxOperatorService` |

## 3. Архитектурная модель и инварианты

### 3.1. Envelope и границы представлений

```text
IntegrationEventEnvelope
├── EventTypeCode
├── TenantId?
├── CorrelationId
├── OccurredAtUtc
└── Payload

OutboxMessage
├── Id
├── служебные поля envelope
├── PayloadJson
└── состояние доставки
```

`IntegrationEventEnvelope` является application-контейнером передачи события.
`OutboxMessage` является persistence-сущностью. `PayloadJson` и поля состояния
доставки не являются частью бизнес-схемы payload.

Подтверждённые ограничения: `EventTypeCode` и `CorrelationId` обязательны;
`TenantId` может быть `null`; время хранится как UTC; payload сериализуется в
JSON. Отдельного поля `PayloadVersion`, `EventId` в envelope или реестра версий
в текущем контракте нет.

| Понятие или архитектурный тип | Техническое имя | Владелец | Связи | Инвариант | Подтверждение |
| --- | --- | --- | --- | --- | --- |
| Envelope события | `IntegrationEventEnvelope` | Integration Events | Содержит служебные поля и `Payload`. | `EventTypeCode`, `CorrelationId`, время и payload имеют определённый формат. | `03_contracts.md` |
| Запись outbox | `OutboxMessage` | Integration Events | Хранит envelope и состояние обработки. | Payload сохраняется как JSON; состояние выводится из временных полей. | Модель и migration Integration Events |
| Обработчик события | `IIntegrationEventProcessorHandler` | Host или модуль-потребитель | Выбирается по `EventTypeCode`. | Один ключ не должен неоднозначно выбирать обработчик. | `03_contracts.md` |

### 3.2. Фактическое использование Foundation

| Компонент Foundation | Как используется | Владелец определения | Семантика Integration Events |
| --- | --- | --- | --- |
| Общая application-зависимость `DMP.BuildingBlocks.Application` | Проект имеет ссылку на сборку | Foundation и Building Blocks | Не добавляет полей в envelope без отдельного контракта |
| `DMP.Platform.Contracts` | Содержит типизированные payload событий | Область-источник конкретного события | Используется для сериализации payload |

Общая карта взаимодействий не копируется здесь; навигация находится в
`docs-new/02_architecture/07_integration_architecture.md`.

## 4. Persistence-модель и хранение

Текущая таблица находится в схеме `integration_events` и называется
`outbox_messages`. Она содержит `Id`, `EventTypeCode`, nullable `TenantId`,
`CorrelationId`, `OccurredAtUtc`, `PayloadJson`, `CreatedAtUtc`,
`PublishedAtUtc`, `PublishError`, `AttemptCount`, `LastAttemptAtUtc`,
`NextAttemptAtUtc` и `DeadLetteredAtUtc`.

Индексы подтверждены для tenant и времени, корреляции, типа события, состояния
публикации, следующей попытки и dead letter. `IIntegrationEventStore` возвращает все
сохранённые outbox-записи как envelope; фильтр только по успешно обработанным
событиям этим портом не заявлен.

Атомарная транзакция между базой бизнес-модуля и этой таблицей текущей областью
не подтверждена: издатель сохраняет outbox через свой `IntegrationEventsDbContext`.

| Данные | Техническое представление | Владелец | Хранение | Ключ или ограничение | Подтверждение |
| --- | --- | --- | --- | --- | --- |
| Envelope события | `IntegrationEventEnvelope` | Integration Events | В памяти до сохранения | Payload сериализуется в JSON | `03_contracts.md` |
| Outbox-запись | `OutboxMessage` | Integration Events | `integration_events.outbox_messages` | `Id`, даты состояния, счётчик попыток и `EventTypeCode` | Модель и migration Integration Events |
| Payload | `PayloadJson` | Модуль-источник по смыслу; Integration Events по хранению | В `outbox_messages` | Отдельная схема payload не задаётся | `03_contracts.md` |

## 5. Зависимости и точки расширения

| Точка | Как подключается | Ограничение |
| --- | --- | --- |
| Издатель | Модуль получает `IIntegrationEventPublisher` | Издатель сохраняет envelope; транспортный брокер не предусмотрен |
| Обработчик | Host регистрирует `IIntegrationEventProcessorHandler` для `EventTypeCode` | Один ключ должен однозначно выбирать обработчик |
| Операторский сервис | Host может использовать `IOutboxOperatorService` | Публичный HTTP-маршрут в области не найден |
| Хранилище | Потребитель получает `IIntegrationEventStore` | Это чтение outbox, а не отдельное хранилище входящих событий потребителя |

## 6. Технические ограничения

- обработчик вызывается только для типа события, зарегистрированного в текущем host;
- несколько экземпляров фонового обработчика не имеют явно подтверждённого механизма распределения и блокировки;
- повторная доставка и дедупликация потребителя не реализованы отдельной моделью;
- версия payload и совместимость событий не имеют общего реестра;
- внешний транспорт и повторное воспроизведение старых событий не подтверждены;
- технический `OutboxMessage.Id` используется для операторских операций и не является
  публичной идентичностью бизнес-события.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
