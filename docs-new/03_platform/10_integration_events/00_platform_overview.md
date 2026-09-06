---
id: DOC-03-10-00
title: 'Обзор платформенной области — Integration Events'
type: design
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

# Обзор платформенной области — Integration Events

## 1. Назначение области

`Integration Events` предоставляет общий механизм сохранения и обработки
межмодульных событий. Область принимает событие через
`IIntegrationEventPublisher`, сохраняет его в outbox и передаёт зарегистрированным
обработчикам после запуска host-приложения.

Область не владеет смыслом события. `Tenant/Security`, `Workflow`, `Object Runtime`
и другие области владеют своими событиями и их payload; `Integration Events`
владеет только envelope, доставкой, состоянием outbox и повторной обработкой.

## 2. Место в платформе

```text
Модуль-владелец события
    -> IIntegrationEventPublisher
    -> integration_events.outbox_messages
    -> OutboxProcessingHostedService
    -> зарегистрированный IIntegrationEventProcessorHandler
```

Общая карта межмодульных связей находится в [интеграционной архитектуре](../../02_architecture/07_integration_architecture.md).
Здесь приведена только локальная схема ответственности.

## 3. Основные возможности

| Возможность | Назначение | Статус | Основание |
| --- | --- | --- | --- |
| Сохранение события в постоянном outbox-хранилище | Сохранить envelope до его фоновой обработки. | Подтверждено MVP | `OutboxMessage`, `PersistentIntegrationEventPublisher` |
| Обработка зарегистрированным обработчиком | Передать сохранённое событие обработчику в host-приложении. | Подтверждено MVP | `OutboxProcessingHostedService`, `IIntegrationEventProcessorHandler` |
| Ограниченное число повторных попыток | Повторить обработку после ошибки обработчика. | Подтверждено MVP | `AttemptCount`, `NextAttemptAtUtc` |
| Состояние dead letter | Сохранить запись после исчерпания повторных попыток. | Подтверждено MVP | `DeadLetteredAtUtc` |
| Повторная постановка и подтверждение для оператора | Управлять записью dead letter через application-порт. | Подтверждено, но ограничено | `IOutboxOperatorService`; HTTP и UI не найдены |
| Внешний транспорт или брокер сообщений | Доставлять события за пределы host-приложения. | Не подтверждено | В текущем коде не найдено |
| Inbox с отдельной дедупликацией потребителя | Защитить потребителя от повторного выполнения. | Не подтверждено | Отдельная модель inbox не найдена |

## 4. Ключевые решения

| Решение | Суть | Документ-владелец |
| --- | --- | --- |
| Поля envelope | `EventTypeCode`, `TenantId`, `CorrelationId`, `OccurredAtUtc` и payload входят в envelope. | `03_contracts.md` |
| Формат хранения payload | Payload сохраняется как JSON внутри outbox. | `02_architecture.md` |
| Условие успешной обработки | Событие считается обработанным после успешного вызова зарегистрированного обработчика. | `04_runtime.md` |
| Владение payload | Семантика и схема payload принадлежат модулю-источнику. | `01_scope.md`, `03_contracts.md` |
| Транзакционная граница | Атомарная фиксация бизнес-состояния и outbox не подтверждена текущей реализацией. | `90_traceability.md` |

## 5. Зависимости

| Зависимость | Назначение |
| --- | --- |
| `DMP.Platform.Contracts` | Типизированные payload отдельных событий |
| `DMP.BuildingBlocks.Application` | Общая application-зависимость проекта; отдельный контракт Integration Events в ней не определён |
| Состав host-приложения | Регистрация издателя, хранилища, фонового обработчика и обработчиков событий |
| `Tenant/Security`, `Configuration`, `Workflow`, `Object Runtime` | Источники и потребители конкретных событий |
| `Audit History` | Отдельный владелец аудита; outbox не заменяет audit record |

## 6. Статус реализации

Механизм постоянного outbox-хранилища и локальной обработки событий реализован в текущем
коде. Внешняя доставка, отдельная модель inbox, реестр версий событий и гарантии
конкурентной обработки для production не объявляются реализованными.

## 7. Состав документов

| Документ | Назначение |
| --- | --- |
| `01_scope.md` | Граница области и владельцы соседних частей |
| `02_architecture.md` | Компоненты, envelope и persistence-модель |
| `03_contracts.md` | Порты, операторский контракт и каталог наблюдаемых событий |
| `04_runtime.md` | Сохранение, обработка, повторные попытки, dead letter и ручное восстановление |
| `05_security_and_audit.md` | Граница авторизации и аудита |
| `07_quality.md` | Тестовые свидетельства и пробелы покрытия |
| `08_operations.md` | Запуск, диагностика и эксплуатационные действия |
| `90_traceability.md` | Источники, маршруты старого материала и открытые вопросы |

Отдельный `06_user_experience.md` не создаётся: собственный UI Integration
Events в текущем коде не подтверждён. Общая оболочка и будущий интерфейс оператора
относятся к фронтенд-платформа.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
