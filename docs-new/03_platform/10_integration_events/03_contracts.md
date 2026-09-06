---
id: DOC-03-10-03
title: 'Контракты платформенной области — Integration Events'
type: contract
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

# Контракты платформенной области — Integration Events

## 1. Назначение и границы

Документ описывает реальные публичные application-порты Integration Events и
наблюдаемые контракты событий. HTTP API и внутренние вспомогательные методы здесь не
описываются.

## 2. Источники истины и владельцы

| Контракт | Источник | Владелец |
| --- | --- | --- |
| `IntegrationEventEnvelope` | `DMP.Platform.IntegrationEvents.Abstractions` | Integration Events |
| Payload `TenantCreated` | `DMP.Platform.Contracts.IntegrationEvents` и модуль-источник Tenant/Security | Tenant/Security |
| Payload `ObjectRuntime.MutationCompleted` | `DMP.Platform.Contracts.IntegrationEvents` и модуль-источник Object Runtime | Object Runtime |
| Payload `Workflow.InstanceInitializationRequested` | `DMP.Platform.Contracts.IntegrationEvents` и модуль-источник Object Runtime | Граница Workflow и Object Runtime |
| Встроенная полезная нагрузка других событий | Код модуля-источника | Соответствующий модуль-источник; стабильная отдельная схема пока не подтверждена |

## 3. Карта контрактов

| Контракт | Потребитель | Назначение | Состояние |
| --- | --- | --- | --- |
| `IIntegrationEventPublisher.PublishAsync` | Модули-источники | Положить envelope в outbox | Реализован |
| `IIntegrationEventStore.GetAllAsync` | Тесты и внутренние потребители | Прочитать сохранённые outbox-события | Реализован |
| `IIntegrationEventProcessorHandler` | Host и модуль-потребитель | Обработать тип события | Реализован |
| `IOutboxOperatorService` | Операторский слой host-приложения | Получить dead letter-записи, повторно поставить их и подтвердить | Реализован как application-порт |

## 4. HTTP-контракты

В проекте Integration Events отдельный HTTP controller или маршрут не найден.
Наличие `IOutboxOperatorService` не означает, что эти операции уже доступны
через HTTP или UI.

## 5. Общие типы и DTO

### 5.1. `IntegrationEventEnvelope`

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `EventTypeCode` | `string` | Нет | Обязательно | Непустой код, задаётся модулем-источником | Код типа события. | Должен однозначно выбирать обработчик в host-приложении. | `IntegrationEventEnvelope` |
| `TenantId` | `Guid` | Да | Необязательно | `null` для событий без tenant-контекста | Идентификатор tenant. | Само поле не выполняет проверку доступа. | `IntegrationEventEnvelope` |
| `CorrelationId` | `string` | Нет | Обязательно | Непустая строка | Идентификатор корреляции. | Должен передаваться в пределах сценария. | `IntegrationEventEnvelope` |
| `OccurredAtUtc` | `DateTime` | Нет | Обязательно | UTC-время | Время возникновения события. | Значение интерпретируется как UTC. | `IntegrationEventEnvelope` |
| `Payload` | `object` | Нет | Обязательно | Объект, сериализуемый в JSON | Полезная нагрузка события. | Схему и ограничения задаёт модуль-источник. | `IntegrationEventEnvelope` |

### 5.2. `OutboxDeadLetterItem`

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `Id` | `Guid` | Нет | Обязательно | Уникальный внутренний ключ | Идентификатор outbox-записи. | Не является публичной идентичностью бизнес-события. | `OutboxDeadLetterItem` |
| `EventTypeCode` | `string` | Нет | Обязательно | Код исходного envelope | Код типа события. | Должен соответствовать зарегистрированному обработчику. | `OutboxDeadLetterItem` |
| `TenantId` | `Guid` | Да | Необязательно | Из исходного envelope или `null` | Идентификатор tenant. | Значение не заменяет проверку доступа. | `OutboxDeadLetterItem` |
| `CorrelationId` | `string` | Нет | Обязательно | Из исходного envelope | Идентификатор корреляции. | Используется для диагностики. | `OutboxDeadLetterItem` |
| `CreatedAtUtc` | `DateTime` | Нет | Обязательно | UTC-время | Время сохранения. | Интерпретируется как UTC. | `OutboxDeadLetterItem` |
| `AttemptCount` | `int` | Нет | Обязательно | Неотрицательное число | Число попыток обработки. | Увеличивается фоновым обработчиком. | `OutboxDeadLetterItem` |
| `PublishError` | `string` | Да | Необязательно | Текст последней ошибки | Причина последнего сбоя обработки. | Может содержать технические подробности ошибки. | `OutboxDeadLetterItem` |
| `DeadLetteredAtUtc` | `DateTime` | Да | Необязательно | UTC-время | Время перевода в dead letter. | Заполняется после исчерпания лимита повторных попыток. | `OutboxDeadLetterItem` |

## 6. C#-контракты и точки расширения

`IIntegrationEventProcessorHandler.EventTypeCode` связывает обработчик с кодом
события. Host регистрирует обработчики через DI. Это точка расширения локального
host-а, а не реестр всех событий платформы.

| Метод или member | Параметры | Результат | Предусловия | Исключения | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- | --- |
| `IIntegrationEventPublisher.PublishAsync` | `IntegrationEventEnvelope`, `CancellationToken` | `Task` | Envelope заполнен и payload сериализуем | Ошибка проверки или сохранения | Создаёт outbox-запись | Локальная транзакция контекста Integration Events |
| `IIntegrationEventStore.GetAllAsync` | `CancellationToken` | Коллекция envelope | Хранилище доступно | Ошибка чтения хранилища | Не изменяет записи | Только чтение |
| `IIntegrationEventProcessorHandler.HandleAsync` | `IntegrationEventEnvelope`, `CancellationToken` | `Task` | Обработчик зарегистрирован для `EventTypeCode` | Исключение запускает повторную попытку или dead letter | Выполняет бизнес-действие потребителя | Не устанавливается этим портом |
| `IOutboxOperatorService.GetDeadLettersAsync` | `take`, `CancellationToken` | Коллекция `OutboxDeadLetterItem` | `take` находится в диапазоне 1–500 | Ошибка чтения хранилища | Не изменяет записи | Только чтение |
| `IOutboxOperatorService.RequeueAsync` | `Guid`, `CancellationToken` | `bool` | Запись найдена операторским слоем | Ошибка хранилища | Сбрасывает состояние для повторной обработки | Локальная транзакция хранилища |
| `IOutboxOperatorService.AcknowledgeAsync` | `Guid`, `CancellationToken` | `bool` | Запись найдена операторским слоем | Ошибка хранилища | Помечает запись обработанной | Локальная транзакция хранилища |

## 7. Контракты событий

Текущие модули-источники публикуют, среди прочих, следующие коды:

| `EventTypeCode` | Payload | Владелец смысла | Найденный обработчик |
| --- | --- | --- | --- |
| `TenantCreated` | `TenantCreatedIntegrationEventPayload` | Tenant/Security | Configuration: создание Tenant scope |
| `ObjectRuntime.MutationCompleted` | `ObjectMutationCompletedIntegrationEventPayload` | Object Runtime | Не найден |
| `Workflow.InstanceInitializationRequested` | `WorkflowInstanceInitializationRequestedPayload` | Object Runtime; обработчик относится к Workflow | Host: `WorkflowInstanceInitializationRequestedEventHandler` |
| `Workflow.CommandExecuted` | Inline payload в Workflow | Workflow | Не найден |
| `Workflow.InstanceReassigned` | Inline payload в Workflow | Workflow | Не найден |
| `TenantUpdated`, `TenantStatusChanged`, `UserUpdated`, `UserStatusChanged`, `PermissionChanged` | Inline payload в Tenant/Security | Tenant/Security | Не найден |
| `UserRoleAssigned`, `UserRoleAssignmentActivated`, `UserRoleAssignmentDeactivated`, `UserRoleAssignmentRemoved` | Inline payload в Tenant/Security | Tenant/Security | Не найден |

Список является инвентаризацией текущих модулей-источников, а не обещанием общей
публичной схемы. Для inline payload отдельные поля и правила совместимости пока
не закреплены стабильным контрактом.

## 8. Ошибки и отказоустойчивость

Ошибку обработки фоновый обработчик сохраняет в `PublishError`, увеличивает
`AttemptCount`, назначает следующую попытку или устанавливает
`DeadLetteredAtUtc`. После 10 попыток запись переводится в dead letter. Это
локальное поведение обработчика outbox; политика внешней доставки не описана.

| Код или тип ошибки | Условие | HTTP или транспортный результат | Поле или путь | Повторить запрос | Ответственный |
| --- | --- | --- | --- | --- | --- |
| Ошибка сериализации | Payload нельзя преобразовать в JSON. | Исключение application-порта | `Payload` | После исправления payload | Модуль-источник |
| Ошибка сохранения | Outbox-хранилище недоступно или отклоняет запись. | Исключение application-порта | Outbox-запись | После восстановления хранилища | Integration Events / Operations |
| Ошибка обработчика | `HandleAsync` завершился исключением. | Повторная попытка или dead letter | `PublishError`, `AttemptCount` | Через `RequeueAsync` после устранения причины | Модуль-потребитель |

## 9. Совместимость и изменение контрактов

В текущем envelope нет отдельного `PayloadVersion` и общего реестра типов событий.
Поэтому правила совместимости, миграции версий и гарантии обратной совместимости
не считаются установленными. Изменение типизированного payload требует проверки
модулем-источником и всеми найденными потребителями.

| Изменение | Совместимо назад | Потребители | Миграция | Версия или решение |
| --- | --- | --- | --- | --- |
| Изменение обязательности или смысла поля envelope | Нет без проверки | Все модули-источники и потребители | Согласовать новый контракт и переходный период | Будущее версионирование событий в `90_traceability.md` |
| Изменение типизированного payload | Не подтверждено | Потребители конкретного события | Проверить модуль-источник и каждого потребителя | Владелец payload |
| Изменение встроенной полезной нагрузки | Не подтверждено | Потребители конкретного события | Сначала закрепить отдельную схему, если она нужна | Владелец события |

## 10. Границы с другими владельцами

Семантика payload, бизнес-ошибки и решение о публикации принадлежат модулю-источнику.
Обработка конкретного события принадлежит зарегистрированному потребителю или
host-приложению. Общая доставка принадлежит Integration Events. Аудит, интерфейс и внешняя
интеграция не становятся частью этой области автоматически.

## 11. Источники и тесты

Основные источники и тестовые свидетельства перечислены в
`90_traceability.md`. Ссылки на конкретные файлы кода нужны для проверки
контракта, но не превращают внутренние классы в публичную модель.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
