---
id: DOC-10-03-01
title: 'Backlog открытых решений Platform API и контрактов'
type: requirement
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: backlog
module: contract_governance
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
---

# Backlog открытых решений Platform API и контрактов

## 1. Назначение

Документ фиксирует только открытые решения, которые остались после разбора
устаревшего обзорного source-материала
`docs/01 sources/13_platform_api_and_contracts.md`.

Source-документ не переносится как нормативный целевой контракт. Он смешивал
общие правила, уже описанные Runtime DTO, желаемые формы будущих API и открытые
governance-вопросы. Подтверждённые application-контракты Foundation описаны в
`03_platform/00_foundation`, Object Runtime API — в `03_platform/03_object_runtime`,
а runtime-поведение — в `04_runtime.md` соответствующей области. Старые
машинные материалы из `docs/05 contracts` остаются источниками сверки и не
создают отдельный раздел `docs-new`.

## 2. Что уже имеет владельца

| Тема | Текущий владелец |
| --- | --- |
| `IDomainModule`, `IModuleContractRegistry`, `ModuleExecutionContext`, use-case и result contracts | `03_platform/00_foundation` |
| Object Runtime read/details/lookup/action/bulk/workflow-facing API semantics | `03_platform/03_object_runtime` |
| Event envelope и local outbox | `03_platform/10_integration_events` |
| HTTP/OpenAPI/JSON Schema/XSD и examples | Документы владельцев; старые файлы `docs/05 contracts` остаются источниками сверки |
| Request pipeline, context, error, concurrency, cache и observability conventions | `04_runtime.md`, `07_quality.md` и `08_operations.md` соответствующих владельцев |

## 3. Что не переносится из source-документа

| Тема source-документа | Решение |
| --- | --- |
| Формы `RuntimeObject*Request/Response`, mutation, action, bulk и lookup | Уже описываются Object Runtime по фактическим DTO и controller. Старые псевдо-DTO из source не являются источником истины. |
| Реестр метаданных как единый общий сервис | Не подтверждён как отдельная capability. Foundation фиксирует форму регистрации stable codes, Configuration и Runtime владеют своими metadata/effective-моделями. |
| "Минимальный состав первой версии Platform API" | Не переносится как единый scope. Реальный состав определяется документами областей. |
| Общие правила совместимости в виде полного policy | Частично уже покрыты стандартом контрактов и документами областей; оставшиеся вопросы ниже оформлены как решения, а не как утверждённые требования. |
| Детали долгих и массовых операций | Не утверждены как единый общий контракт. Подлежат отдельному решению только при появлении владельца operation/job capability или конкретного API у владельца области. |

## 4. Оставшиеся открытые решения

| ID | Решение | Почему не закрыто текущими документами | Куда вносить результат | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-API-01` | Развести public/internal boundary для HTTP, events, module registration и published configuration. | Стандарт описания контрактов задаёт форму документа, но не полный каталог публичных границ DMP. | `00_governance/03_contract_documentation_standard.md`, traceability владельцев областей. | Open |
| `PCDOC-API-02` | Утвердить владельцев stable-code catalog по видам кодов. | Foundation подтверждает `ModuleContractCodeKind`, но не владеет всеми code catalogs и миграционной политикой каждого вида кода. | Foundation, Configuration, Object Runtime, Integration Events и владельцы кодов. | Open |
| `PCDOC-API-03` | Утвердить внешний error model и HTTP status mapping. | Foundation прямо не объявляет `PlatformIssue` единым внешним HTTP-ответом; Object Runtime имеет локальный `RuntimeErrorResponse`. | `00_governance/03_contract_documentation_standard.md`, runtime/area contracts. | Open |
| `PCDOC-API-04` | Утвердить HTTP API versioning, endpoint naming и deprecation policy. | Стандарт требует фиксировать совместимость, но не выбирает общий способ версионирования публичного HTTP API. | `00_governance`, area `03_contracts.md`. | Open |
| `PCDOC-API-05` | Утвердить namespace/versioning для integration contracts. | Integration Capability вынесена в отдельный backlog; текущие Integration Events описывают event foundation, а не полный внешний integration API. | `10_backlog/platform_capabilities/integration_capability_backlog.md`, будущий владелец Integration Capability. | Open |
| `PCDOC-API-06` | Утвердить governance для `ExecutionProfileCode`. | Object Runtime содержит execution profiles, но общая политика для импорта, миграций, ремонта данных и массового обслуживания требует владельца и ограничений. | Object Runtime / Operations. | Open |
| `PCDOC-API-07` | Закрыть source staging `03_platform/09_common_application_contracts/offers`. | Source признан устаревшим; сохраняются только открытые решения из этой таблицы. | Staging-копия удалена, активные ссылки переведены. | Completed |

## 5. Зафиксированные ограничения

Новые решения по API и контрактам не должны разрешать:

- раскрытие внутренних таблиц и EF/internal classes как public API;
- прямой доступ интеграций к данным прикладных модулей;
- флаги `SkipValidation`, `SkipBusinessLogic`, `SkipRules`, `SkipLifecycle`, `SkipAudit`;
- принятие бизнес-решений на клиенте вместо серверной платформенной проверки;
- произвольное создание публичных stable codes без владельца и миграционной политики.

## 6. Связанные документы

- [Контракты Foundation](../../03_platform/00_foundation/03_contracts.md)
- [Контракты Object Runtime](../../03_platform/03_object_runtime/03_contracts.md)
- [Стандарт описания контрактов](../../00_governance/03_contract_documentation_standard.md)
- [Backlog Integration Capability](../platform_capabilities/integration_capability_backlog.md)
- [Backlog документации Platform Core](../roadmap/preparation/platform_core_documentation_backlog.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
