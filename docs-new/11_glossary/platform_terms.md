---
id: DOC-11-99-02
title: 'Платформенные термины DMP'
type: glossary
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: glossary
module: glossary
holder: '@axelprosoft'
created_at: 2026-08-25 12:00
created_by: '@VeronikaV2121'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
---

# Платформенные термины DMP

## 1. Назначение документа

Документ фиксирует рабочие русские термины, английские имена и технические алиасы для платформенного ядра DMP.

Основной язык нормативной документации — русский. Английское имя используется как технический алиас, если оно нужно для связи с кодом, API, контрактом, именем каталога или общепринятым инженерным термином.

## 2. Правила ведения

- Русский термин используется в заголовках, таблицах и обычном тексте.
- Preferred English указывает устойчивый английский термин для архитектурных обсуждений и связи с кодом.
- Технический алиас пишется точно как в коде, контракте, route, configuration key или каталоге.
- Термин без `TERM-*` имеет статус `Кандидат`.
- Локальные таблицы терминов в документах платформенных областей не заменяют этот глоссарий; они только повторяют ключевые термины для удобства чтения.
- Если термин отличается от русских подписей UI или security manifest, расхождение фиксируется в тематическом глоссарии.

## 3. Термины-кандидаты платформенного ядра

| Русский термин | Preferred English | Технический алиас | Статус | Определение | Источник в коде | Нежелательные синонимы |
| --- | --- | --- | --- | --- | --- | --- |
| Платформенная область | Platform capability | `<CC_platform_area>/`, `CapabilityCode` | Кандидат | Самостоятельная ответственность платформенного ядра с устойчивым техническим владельцем: проектом, контрактом, хранилищем, runtime service, общим frontend-механизмом или приложением. | [шаблон платформенной области](../03_platform/00_platform_documentation_template.md) | - |
| Технический владелец | Technical owner | `owner`, `holder`, project owner | Кандидат | Ответственный владелец реализации, контракта и нормативного документа. | [шаблон документа](../00_governance/01_document_template.md) | - |
| Технический алиас | Technical alias | class, interface, route, code, configuration key | Кандидат | Точное имя из кода или контракта, которое указывается рядом с русским термином. | [documentation strategy](../00_governance/00_documentation_strategy.md#21-язык-и-терминология) | - |
| Источник истины | Source of truth | source of truth | Кандидат | Место, в котором факт считается нормативно определённым: код, контракт, утверждённый документ, ADR или исходное требование в зависимости от приоритета источников. | [documentation strategy](../00_governance/00_documentation_strategy.md#3-источник-истины) | - |
| Стабильный код | Stable code | `Code`, `PermissionCode`, `EventTypeCode` | Кандидат | Идентификатор, который используется в контрактах, конфигурации, правах, событиях или UI и не переименовывается без правила совместимости. | [шаблон платформенной области](../03_platform/00_platform_documentation_template.md) | - |
| Область действия | Scope | `ScopeType`, scope | Кандидат | Уровень применения настройки, права, роли, данных или действия. Конкретные значения и ограничения определяются владельцем области. | [Tenant Security enums](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Enums/ScopeType.cs) | - |
| Контекст запроса | Request context | `PlatformRequestContext` | Кандидат | Набор координат запроса: tenant, user, site, role, correlation и язык, переданный в platform runtime. | [PlatformRequestContext](../../src/Platform/DMP.Platform.Runtime/Context/PlatformRequestContext.cs) | - |
| Проверенная ревизия MVP | Verified MVP revision | commit SHA | Кандидат | Ревизия кода, по которой сопоставлены документация, контракты и тесты. | [шаблон платформенной области](../03_platform/00_platform_documentation_template.md#3-источники-и-приоритет) | - |
| Открытый вопрос | Open question | open decision | Кандидат | Решение, которое требуется принять до переноса утверждения в нормативный документ-владелец. | [шаблон платформенной области](../03_platform/00_platform_documentation_template.md#710-90_traceabilitymd) | - |
| Будущая доработка вне MVP | Future capability gap | backlog, future work | Кандидат | Возможность, отсутствующая в MVP или реализованная неполно и требующая отдельного проектного результата. | [шаблон платформенной области](../03_platform/00_platform_documentation_template.md#710-90_traceabilitymd) | - |
| Команда workflow | Workflow command | `CommandCode` | Кандидат | Публичный код команды, по которому Workflow Runtime выбирает доступный переход для экземпляра. | [контракты Workflow](../03_platform/04_workflow/03_contracts.md#46-executecommand) | - |
| Переход workflow | Workflow transition | `TransitionCode` | Кандидат | Внутренне выбранный маршрут между состояниями workflow; клиент передаёт `CommandCode`, а не `TransitionCode`. | [архитектура Workflow](../03_platform/04_workflow/02_architecture.md#3-архитектурная-модель-и-инварианты) | - |
| Состояние workflow | Workflow state | `CurrentStateCode`, `WorkflowState` | Кандидат | Текущее состояние экземпляра workflow, сохранённое для объекта и выбранной ревизии определения. | [контракты Workflow](../03_platform/04_workflow/03_contracts.md#51-workflowstateresponse) | - |
| Ревизия определения workflow | Workflow definition revision | `WorkflowDefinitionRevision` | Кандидат | Неизменяемый снимок определения, с которым связан экземпляр workflow. | [исполнение Workflow](../03_platform/04_workflow/04_runtime.md#31-разрешение-definition) | - |
| Условие перехода | Transition guard | `Guard`, `IWorkflowTransitionGuard` | Кандидат | Проверка, которая участвует в определении доступности перехода после проверки права выполнения. | [исполнение Workflow](../03_platform/04_workflow/04_runtime.md#32-вычисление-доступных-команд) | - |
| Политика состояния | State policy | `StatePolicy` | Кандидат | Набор runtime-правил, связанных с состоянием workflow, включая ограничения перехода и синхронизацию archive state. | [архитектура Workflow](../03_platform/04_workflow/02_architecture.md#3-архитектурная-модель-и-инварианты) | - |
| Наборы значений | Value Sets | `DMP.Platform.ValueSets`, `ValueSets` | Кандидат | Платформенная область, которая хранит фактические элементы наборов значений, строит effective-набор и предоставляет API чтения и редактирования данных. | [обзор Value Sets](../03_platform/06_value_sets/00_platform_overview.md), [проект Value Sets](../../src/Platform/DMP.Platform.ValueSets/) | - |
| Набор значений | Value set | `ValueSet` | Кандидат | Определение источника допустимых значений, принадлежащее Configuration; не является хранимым списком фактических элементов Value Sets. | [schema ValueSet](../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ValueSetArtifactSchemas.cs), [спецификация ValueSet](../03_platform/02_configuration/artifact_types/value_set.md) | - |
| Данные набора значений | Value set data | `ValueSetData`, `ValueSetDataSet` | Кандидат | Фактические данные набора, хранимые областью Value Sets отдельно от schema Configuration. | [ValueSetDataSet](../../src/Platform/DMP.Platform.ValueSets/Domain/Entities/ValueSetDataSet.cs) | - |
| Элемент набора | Value set item | `ValueSetItem`, `ValueSetDataItem` | Кандидат | Одна фактическая строка набора со стабильным `Code`, отображаемым `Title`, active state, порядком, scope и возможной parent-связью. | [ValueSetItem](../../src/Platform/DMP.Platform.ValueSets/Domain/Entities/ValueSetItem.cs), [Value Sets contracts](../../src/Platform/DMP.Platform.Contracts/ValueSets/) | - |
| Область данных набора | Value set data scope | `ValueSetDataSourceScope`, `Global`, `Tenant`, `Site` | Кандидат | Scope фактической строки набора; Site требует Tenant. | [scope enum](../../src/Platform/DMP.Platform.Contracts/ValueSets/Enums/ValueSetDataSourceScope.cs), [ValueSetItem](../../src/Platform/DMP.Platform.ValueSets/Domain/Entities/ValueSetItem.cs) | - |
| Эффективный набор | Effective value set | `EffectiveCurrent`, `EffectiveSelectedScope`, `ValueSetItemsResponse` | Кандидат | Результат выбора одной наиболее специфичной строки для каждого code в заданном scope. | [query service](../../src/Platform/DMP.Platform.ValueSets/Application/Services/ValueSetsQueryService.cs), [editor service](../../src/Platform/DMP.Platform.ValueSets/Application/Services/ValueSetDataEditorService.cs) | - |
| Переопределение элемента | Scoped item override | `IsOverride`, `TenantId`, `SiteId` | Кандидат | Строка с тем же code в более специфичном scope, которая заменяет менее специфичную строку в effective-результате. | [editor service](../../src/Platform/DMP.Platform.ValueSets/Application/Services/ValueSetDataEditorService.cs) | - |
| Запись аудита | Audit record | `AuditRecord` | Кандидат | Общая запись о значимой операции, которую модуль-источник (producer) передаёт в Audit History через `IAuditHistoryWriter`; предметный смысл операции принадлежит модулю-источнику. | [AuditRecord](../../src/Platform/DMP.Platform.AuditHistory/Abstractions/AuditRecord.cs), [контракт Audit History](../03_platform/09_audit_history/03_contracts.md) | История объекта, история workflow |
| Средство записи аудита | Audit history writer | `IAuditHistoryWriter` | Кандидат | Прикладной контракт, принимающий `AuditRecord` и предоставляющий ограниченное чтение снимка записей; реализация хранения принадлежит Audit History. | [IAuditHistoryWriter](../../src/Platform/DMP.Platform.AuditHistory/Abstractions/IAuditHistoryWriter.cs), [контракт Audit History](../03_platform/09_audit_history/03_contracts.md) | Репозиторий, журнал producer-а |
| Код операции аудита | Audit action code | `AuditRecord.ActionCode` | Кандидат | Строковый код операции внутри записи аудита. Его смысл и набор значений определяет модуль-источник (producer); это не код конфигурационного артефакта `Action`. | [AuditRecord](../../src/Platform/DMP.Platform.AuditHistory/Abstractions/AuditRecord.cs), [исполнение Audit History](../03_platform/09_audit_history/04_runtime.md) | Код конфигурационного действия, Action code без контекста |
| История аудита | Audit history | `DMP.Platform.AuditHistory`, `audit_history.audit_records` | Кандидат | Платформенная область, которая принимает записи о значимых операциях через `IAuditHistoryWriter` и сохраняет их в собственной модели хранения. | [обзор Audit History](../03_platform/09_audit_history/00_platform_overview.md), [проект Audit History](../../src/Platform/DMP.Platform.AuditHistory/) | История бизнес-объекта, история workflow |
| Маскирование чувствительных значений | Sensitive value masking | `IsSensitive`, `[REDACTED]` | Кандидат | Замена чувствительного старого или нового значения на `[REDACTED]` перед записью деталей операции; в текущем MVP подтверждено для деталей изменения Object Runtime. | [ObjectRuntimeAuditSink](../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ObjectRuntimeAuditSink.cs), [безопасность Audit History](../03_platform/09_audit_history/05_security_and_audit.md) | Удаление значения без политики, скрытие без указания правила |
| Интеграционное событие | Integration event | `IntegrationEventEnvelope`, `EventTypeCode` | Кандидат | Факт, сохранённый для последующей передачи другому модулю или обработчику; бизнес-смысл и payload принадлежат модулю-источнику. | [обзор Integration Events](../03_platform/10_integration_events/00_platform_overview.md) | - |
| Конверт события | Event envelope | `IntegrationEventEnvelope` | Кандидат | Общий контейнер metadata события: код типа, tenant, корреляция, время возникновения и payload. | [контракты Integration Events](../03_platform/10_integration_events/03_contracts.md) | - |
| Исходящая очередь | Outbox | `OutboxMessage`, `outbox_messages` | Кандидат | Надёжно сохранённые записи событий, которые фоновой обработчик пытается обработать после публикации producer-ом. | [архитектура Integration Events](../03_platform/10_integration_events/02_architecture.md) | - |
| Очередь недоставленных сообщений | Dead letter | `DeadLetteredAtUtc` | Кандидат | Состояние outbox-записи после исчерпания заданного числа неудачных попыток обработки. | [выполнение Integration Events](../03_platform/10_integration_events/04_runtime.md) | - |
| Идентификатор корреляции | Correlation ID | `CorrelationId` | Кандидат | Значение, связывающее запись аудита, событие или исходный запрос со сквозным сценарием для диагностики. | [AuditRecord](../../src/Platform/DMP.Platform.AuditHistory/Abstractions/AuditRecord.cs), [контракты Integration Events](../03_platform/10_integration_events/03_contracts.md) | - |
| Модуль-источник события | Event producer | `IIntegrationEventPublisher` | Кандидат | Модуль или компонент, который создаёт событие и передаёт его в механизм исходящей очереди. Он владеет бизнес-смыслом события и его payload. | [граница Integration Events](../03_platform/10_integration_events/01_scope.md) | Producer без пояснения роли |
| Потребитель события | Event consumer | `consumer`, registered handler | Кандидат | Компонент, который получает событие из механизма обработки и выполняет принадлежащую ему реакцию. В текущем MVP потребитель представлен зарегистрированным обработчиком в host-приложении. | [контракты Integration Events](../03_platform/10_integration_events/03_contracts.md) | Consumer без пояснения роли |
| Обработчик события | Event handler | `IIntegrationEventProcessorHandler` | Кандидат | Зарегистрированный компонент, который выбирается по `EventTypeCode` и вызывается процессором для обработки события. | [выполнение Integration Events](../03_platform/10_integration_events/04_runtime.md) | Handler без пояснения роли |
| Интеграционная возможность | Integration Capability | `ExternalSystem`, `IntegrationFlow` | Кандидат | Планируемая отдельная платформенная область для обмена с внешними системами, `connectors`, `flows` и `mappings`. Она не является текущим механизмом `Integration Events` и не считается реализованной. | [трассировка Integration Events](../03_platform/10_integration_events/90_traceability.md), [backlog](../10_backlog/roadmap/preparation/platform_core_documentation_backlog.md) | Integration Events как обмен с внешними системами |

## 4. Нежелательные формулировки

| Формулировка | Почему не использовать | Чем заменить |
| --- | --- | --- |
| Платформенная возможность без русского пояснения (`Capability`) | В нормативной документации основной язык русский. | Платформенная область (`platform capability`). |
| Источник без указания смысла (`Source`) | Неясно, это исходник, источник требования или источник истины. | Источник, исходный материал или источник истины по контексту. |
| Целевое решение без статуса (`Target`) | Может скрывать будущую возможность как утверждённое поведение. | Целевое решение, открытый вопрос или будущая доработка вне MVP. |
| Область действия без русского пояснения (`Scope`) | Термин перегружен между правами, данными и конфигурацией. | Область действия (`ScopeType`) или конкретный уровень: Global, Corporate, Tenant, Site. |
| Неясная формулировка для UI: `Surface / пользовательская поверхность` | Калька не говорит архитектору и разработчику, что именно реализовано. | Интерфейсная часть, раздел интерфейса, экран, сценарий или представление фронтенда. |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
