---
id: DOC-03-09-03
title: 'Контракты платформенной области - Audit History'
type: contract
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: audit_history
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

# Контракты платформенной области - Audit History

## 1. Назначение и границы

Документ описывает единственный подтверждённый межкомпонентный контракт Audit
History: `IAuditHistoryWriter` и передаваемый ему `AuditRecord`. Подробности
`Details` определяет потребитель операции. Класс хранения, контекст EF Core,
HTTP API чтения и контракт интерфейсной части в этот документ не входят.

## 2. Источники истины и владельцы

| Контракт или часть | Источник истины | Владелец |
| --- | --- | --- |
| `IAuditHistoryWriter` и `AuditRecord` | `DMP.Platform.AuditHistory.Abstractions` | Audit History |
| `ActionCode` и `Details` конкретной операции | Код вызывающей области | Вызывающая область |
| Контекст tenant, user и корреляции | Foundation, Platform Runtime и вызывающая область | Соответствующий владелец контекста |
| Таблица `audit_records` | `AuditHistoryDbContext` и initializer | Audit History |

## 3. Карта контрактов

| Контракт | Канал | Потребитель | Результат |
| --- | --- | --- | --- |
| `IAuditHistoryWriter.WriteAsync` | Внутренний прикладной порт | Tenant/Security, Object Runtime, Workflow, Settings, прикладные обработчики | Асинхронная запись `AuditRecord` |
| `IAuditHistoryWriter.GetSnapshot` | Внутренний прикладной порт | Тесты и локальный код | Полная коллекция записей в порядке `OccurredAtUtc` |
| HTTP API Audit History | Не найден | Нет | Не является текущим контрактом |
| Интеграционное событие Audit History | Не найдено | Нет | Не является текущим контрактом |

## 4. HTTP-контракты

В текущем коде нет `AuditHistoryController`, маршрута для запроса записей или
публичного HTTP DTO Audit History. Поэтому данный раздел фиксирует отсутствие
HTTP-контракта, а не обещает его будущую форму.

## 5. Общие типы и DTO

### 5.1 `AuditRecord`

| Поле конструктора | Тип | Обязательность | Смысл и ограничения |
| --- | --- | --- | --- |
| `ActionCode` | `string` | Обязательно | Непустой код операции; конкретный набор кодов не задаётся Audit History |
| `TenantId` | `Guid?` | Необязательно | Tenant операции; null допустим для системного или не привязанного сценария |
| `UserId` | `Guid?` | Необязательно | Пользователь операции; null допустим |
| `CorrelationId` | `string` | Обязательно | Непустой идентификатор корреляции |
| `OccurredAtUtc` | `DateTime` | Обязательно | Время операции; модель хранения приводит `kind` к UTC |
| `Details` | `object` | Поле сигнатуры record | Произвольные детали; формат принадлежит потребителю, отдельная проверка содержимого во время выполнения не задаётся |

Поля `ObjectTypeCode`, `ObjectId`, `OperationCode`, `Result`, `ActorType` и
`DetailsReference` не являются полями текущего `AuditRecord` и не должны
добавляться в документацию как реализованный контракт.

Поясняющий пример, не являющийся общей JSON Schema:

```json
{
  "actionCode": "ObjectRuntime.Update",
  "tenantId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "userId": "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb",
  "correlationId": "request-123",
  "occurredAtUtc": "2026-08-27T10:00:00Z",
  "details": {
    "moduleCode": "ExampleModule",
    "objectTypeCode": "ExampleObject",
    "objectId": "object-1",
    "mutationKind": "Update"
  }
}
```

Структура `details` в этом примере принадлежит Object Runtime и не становится
обязательной для Tenant/Security, Workflow, Settings или Domain Modules.

## 6. C#-контракты и точки расширения

### 6.1 `IAuditHistoryWriter`

```csharp
Task WriteAsync(AuditRecord record, CancellationToken cancellationToken = default);
IReadOnlyCollection<AuditRecord> GetSnapshot();
```

| Метод | Параметры | Возвращаемое значение | Семантика |
| --- | --- | --- | --- |
| `WriteAsync` | `record`: запись аудита; `cancellationToken`: отмена операции | `Task` | Провайдер сохраняет запись или сообщает ошибку; вызывающая область решает, как обработать ошибку |
| `GetSnapshot` | Нет | `IReadOnlyCollection<AuditRecord>` | Возвращает полный доступный снимок; контракт не задаёт фильтрацию или пагинацию |

`InMemoryAuditHistoryWriter` и `PersistentAuditHistoryWriter` - реализации
этого контракта. `AuditRecordEntry` и `AuditHistoryDbContext` являются
внутренними деталями хранения и не являются контрактами расширения.

## 7. Контракты событий

Audit History не публикует собственного интеграционного события в текущем коде.
Запись аудита не равна публикации события: Integration Events владеет конвертом,
доставкой, outbox/inbox и повторной обработкой.

## 8. Ошибки и отказоустойчивость

- пустые `ActionCode` и `CorrelationId` отклоняются при создании записи хранения
  через `ArgumentException`;
- null `AuditRecord` отклоняется реализациями через `ArgumentNullException`;
- ошибка сериализации `Details` или `SaveChangesAsync` возвращается вызывающему
  коду как ошибка задачи;
- контракт не определяет retry, outbox, fallback или атомарность с операцией
  потребителя;
- отсутствие `IAuditHistoryWriter` в `ObjectRuntimeAuditSink` приводит к
  пропуску записи, что является ограничением текущей host-композиции.

## 9. Совместимость и изменение контрактов

Добавление нового потребителя не меняет `AuditRecord`, если он может передать
собственные детали. Изменение полей `AuditRecord` или семантики `WriteAsync`
является изменением общего прикладного контракта и требует проверки всех
потребителей и тестов. Стабильный реестр кодов действий и типизированные детали
для всех областей текущим контрактом не предусмотрены.

## 10. Границы с другими владельцами

| Соседний владелец | Что получает или передаёт | Что не является его частью в этом документе |
| --- | --- | --- |
| Tenant/Security | Передаёт записи аудита команд безопасности | Политика прав на чтение и смысл кодов действий безопасности |
| Object Runtime | Передаёт запись аудита mutation | Pipeline mutation и схема объекта |
| Workflow | Передаёт записи выполнения Workflow | История Workflow и модель переходов состояний |
| Settings / Configuration | Settings передаёт записи изменения значений | Схема настроек и модель изменения Configuration |
| Integration Events | Отдельный канал событий | Запись аудита не является конвертом события |
| Domain Modules | Передают записи своих системных операций | Предметная история объекта и бизнес-смысл |

## 11. Источники и тесты

- [`AuditRecord`][record];
- [`IAuditHistoryWriter`][writer];
- [persistent implementation][persistent];
- [Object Runtime audit sink][runtime-sink];
- [integration tests][tests].

[record]: ../../../src/Platform/DMP.Platform.AuditHistory/Abstractions/AuditRecord.cs
[writer]: ../../../src/Platform/DMP.Platform.AuditHistory/Abstractions/IAuditHistoryWriter.cs
[persistent]: ../../../src/Platform/DMP.Platform.AuditHistory/Infrastructure/Persistence/PersistentAuditHistoryWriter.cs
[runtime-sink]: ../../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ObjectRuntimeAuditSink.cs
[tests]: ../../../tests/DMP.Platform.IntegrationTests/

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
