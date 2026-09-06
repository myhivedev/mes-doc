---
id: DOC-03-08-03
title: 'Контракты платформенной области — Numbering'
type: contract
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: numbering
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Контракты платформенной области — Numbering

## 1. Назначение и границы

Документ описывает стабильные application-порты Numbering и их DTO. HTTP API
Numbering и отдельные события выдачи в текущем коде не подтверждены.

## 2. Источники истины и владельцы

| Контракт или модель | Владелец | Где подробно |
| --- | --- | --- |
| `NumberingRule` schema | Configuration | `../02_configuration/artifact_types/numbering_rule.md` |
| `ObjectMutationContext` и mutation hooks | Object Runtime | `../03_object_runtime/03_contracts.md`, `04_runtime.md` |
| Numbering application ports и DTO | Numbering | Этот документ |
| Таблицы счётчиков и журнала | Numbering | `02_architecture.md` |

## 3. Карта контрактов

| Контракт | Потребитель | Канал | Результат |
| --- | --- | --- | --- |
| `INumberingService` | Object Runtime hook | application port | `NumberingIssue` |
| `INumberingRuleResolver` | Numbering evaluator; host composition | extension point | Набор `NumberingRuleDefinition` |
| `INumberingRuleRuntimeEvaluator` | Object Runtime hook | application port | Кандидаты правил |
| `INumberingCounterStore` | Numbering service | persistence port | Выделенный sequence number |

## 4. HTTP-контракты

Отдельный HTTP API Numbering в текущей области не подтверждён. Редактирование и
публикация `NumberingRule` выполняется контрактами Configuration.

## 5. Общие типы и DTO

### 5.1. `NumberingIssueRequest`

Запрос на выдачу номера от Object Runtime к Numbering.

| Поле | Тип | Обязательность | Назначение |
| --- | --- | --- | --- |
| `RuleIdentity` | `string` | Да | Стабильная identity правила для ключа счётчика |
| `RuleCode` | `string` | Да | Код правила |
| `ModuleCode` | `string` | Да | Владелец целевого типа |
| `ObjectTypeCode` | `string` | Да | Тип объекта |
| `FieldCode` | `string` | Да | Целевое поле |
| `ScopeId` | `string` | Да | Область счётчика |
| `PartitionKey` | `string` | Да | Раздел последовательности |
| `Template` | `string` | Да | Строка форматирования |
| `StartNumber` | `long` | Да | Начальное значение последовательности |
| `NumberingStep` | `int` | Да | Шаг, больше нуля |
| `TenantId`, `SiteId`, `UserId` | `Guid?` | Нет | Контекст операции |
| `CorrelationId` | `string` | Да | Идентификатор операции |
| `ConfigurationEntryId`, `PublishedVersionId` | `Guid?` | Нет | Ссылки на источник опубликованного правила |
| `TemplateValues` | `IReadOnlyDictionary<string, object?>?` | Нет | Значения разрешённых переменных шаблона |

### 5.2. `NumberingIssue`

| Поле | Тип | Назначение |
| --- | --- | --- |
| `CounterId` | `Guid` | Идентификатор изменённого счётчика |
| `IssuedValue` | `string` | Сформированное значение для поля объекта |
| `SequenceNumber` | `long` | Выделенное порядковое значение |

### 5.3. Запрос и результат счётчика

`NumberingCounterAllocationRequest` содержит `RuleIdentity`, `ScopeId`,
`PartitionKey`, `StartNumber` и `NumberingStep`. `NumberingCounterAllocation`
возвращает `CounterId` и `SequenceNumber`. Эти типы являются внутренним
application/persistence контрактом Numbering, а не HTTP DTO.

## 6. C#-контракты и точки расширения

| Контракт | Назначение | Точка расширения |
| --- | --- | --- |
| `INumberingService` | Выдать и отформатировать номер | Нет, основной application port |
| `INumberingRuleResolver` | Получить подходящие определения правил | Да, host должен предоставить resolver опубликованной конфигурации |
| `INumberingRuleRuntimeEvaluator` | Проверить условия и построить кандидатов | Внутренний application port |
| `INumberingCounterStore` | Абстрагировать persistence счётчиков | Да, persistence provider |

`NumberingRuleDefinition` является runtime-представлением правила, а не схемой
Configuration. `NumberingRuleRuntimeCandidate` добавляет к нему `ScopeId`,
`PartitionKey` и значения шаблона.

## 7. Контракты событий

Собственного integration event для выдачи номера или изменения счётчика в
текущем контракте не найдено. `NumberingIssueLog` является записью хранения, а
не событием доставки.

## 8. Ошибки и отказоустойчивость

| Ситуация | Технический код или объект | Результат |
| --- | --- | --- |
| Ручное значение запрещено | `NUMBERING_MANUAL_INPUT_FORBIDDEN` | Object Runtime возвращает ошибку проверки |
| Обязательное поле осталось без правила | `NUMBERING_RULE_MISSING` | Создание объекта не проходит проверку |
| Несколько правил имеют максимальный приоритет | `NUMBERING_RULE_AMBIGUOUS` | Выдача не выполняется |
| Источник шаблона пуст или недоступен | `InvalidOperationException` | Выдача прекращается до форматирования |
| Ошибка после выделения номера | `NumberingCounterEntry` | Номер не возвращается; возможен пропуск |

Гарантия повторной доставки integration event к Numbering не устанавливается,
поскольку такого события в текущем контракте нет.

## 9. Совместимость и изменение контрактов

Изменение состава `NumberingRule` выполняется владельцем Configuration. Изменение
`INumberingService` должно сохранять смысл ключа счётчика и обязательность
контекста. Новые моменты присвоения требуют согласованных изменений в
Configuration, Object Runtime, Numbering и тестах.

## 10. Границы с другими владельцами

Numbering не объявляет права редактирования правила и не предоставляет отдельный
пользовательский интерфейс.
Авторизация настройки принадлежит Configuration/Tenant/Security, авторизация
создания объекта — Object Runtime и соответствующему Domain Module.

## 11. Источники и тесты

- [Numbering service][numbering-service]
- [Numbering ports][numbering-ports]
- [тесты Numbering][numbering-tests]
- [тесты публикации NumberingRule][validation-tests]

[numbering-service]: ../../../src/Platform/DMP.Platform.Numbering/Application/Services/NumberingService.cs
[numbering-ports]: ../../../src/Platform/DMP.Platform.Numbering/Application/Abstractions/INumberingService.cs
[numbering-tests]: ../../../tests/DMP.Platform.IntegrationTests/Numbering/NumberingServiceIntegrationTests.cs
[validation-tests]: ../../../tests/DMP.Platform.IntegrationTests/Configuration/NumberingPublishValidationIntegrationTests.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
