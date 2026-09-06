---
id: DOC-10-02-01
title: 'Подробный backlog будущей Integration Capability'
type: requirement
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: backlog
module: integration_capability
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

# Подробный backlog будущей Integration Capability

## 1. Назначение

Документ сохраняет будущие работы по полной Integration Capability DMP после
разбора переходного source-материала
`docs/01 sources/11_1_integration_capability_architecture.md`.

Это не нормативная архитектура реализованной платформенной области. До
появления подтверждённого владельца реализации, номера каталога в `03_platform`,
кода, контрактов и тестов Integration Capability остаётся backlog-направлением.

## 2. Граница будущей capability

Integration Capability должна отвечать за управляемый обмен с внешними
системами: ERP, CAD/PDM, MDC, DMS, BI/Data Lake, файловыми источниками и другими
интеграционными контурами.

| Компонент | Будущая ответственность |
| --- | --- |
| Integration Service | Исполнение integration flows, retry, sync state, external mappings, integration messages, attempts, manual reprocess и integration audit. |
| Integration Configuration | Описание external systems, flows, bindings, mappings, triggers, retry/error policies и tenant/site enablement без исполнения бизнес-логики. |
| Connectors / adapters | Протоколы, внешняя авторизация, форматы payload, вызовы внешних API, чтение файлов, очередей или gateway. |
| Domain modules | Бизнес-смысл, данные, validation и application use-cases, вызываемые интеграционным runtime. |
| Integration Events | Event envelope, outbox, локальная обработка, повторы и dead letter для событий в текущей подтверждённой области. |

## 3. Не-цели

Integration Capability не должна становиться:

- универсальным ESB-конструктором;
- no-code ETL-платформой;
- визуальным integration designer в MVP;
- механизмом прямого доступа к таблицам прикладных модулей;
- заменой Workflow, Rules, IAM, Tenant isolation или domain validation;
- местом хранения секретов в обычной configuration storage;
- способом обхода application use-cases прикладного модуля.

## 4. Будущие рабочие пакеты

| ID | Рабочий пакет | Содержание | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-IC-01` | Граница Integration Capability | Разделить Integration Events, Integration Service, connectors, external systems, flows, mappings и domain/application contracts. | Есть scope-документ будущей области или решение не создавать область; текущие Integration Events не выданы за внешнюю интеграцию. | Open |
| `PCDOC-IC-02` | Configuration model | `ExternalSystem`, `ConnectorDefinition`, `IntegrationFlow`, `IntegrationContractBinding`, `MappingDefinition`, `RetryPolicy`, `ErrorPolicy`, `TriggerDefinition`, `CredentialRef`. | Для каждого типа определены владелец, lifecycle, scope, versioning, publish/effective rules, секреты и граница с Configuration. | Open |
| `PCDOC-IC-03` | Operational runtime model | `IntegrationMessage`, `IntegrationAttempt`, `ExternalObjectMapping`, `SyncState`, `Inbox`, idempotency и manual reprocess. | Разделены inbound/outbound flow, сообщение, попытка, состояние синхронизации, дубликаты, внешний идентификатор и история. | Open |
| `PCDOC-IC-04` | Первые vertical slices | `ERP.ImportProductionOrders`, `DMP.ExportProductionOrderStatusToERP`, `CADPDM.ImportBomVersions`; отдельно Equipment/MDC telemetry. | Для каждого slice описаны trigger, mapping, вызов domain/application contract, ошибки, retry, audit, диагностика и критерий повторной обработки. | Open |
| `PCDOC-IC-05` | Каноническая platform area | Новый свободный номер и пакет `00`–`90` в `03_platform` после решения о владельце. | Пакет создаётся только после подтверждения кода, контрактов, persistence и тестов. | Open |
| `PCDOC-IC-06` | Messaging and orchestration choice | MassTransit, Wolverine, Rebus, NServiceBus, Dapr, Temporal/Elsa/Workflow Core, external integration tools. | Принято ADR: что является transport/processing layer, что не заменяет Integration Capability, какие библиотеки допустимы для MVP. | Open |
| `PCDOC-IC-07` | Integration contracts | External payload schemas, internal contract binding, event schemas, OpenAPI/AsyncAPI, stable codes. | External payload не равен internal table model; contracts имеют владельца, версию и compatibility policy; отдельный общий раздел `05_contracts` не создаётся. | Open |
| `PCDOC-IC-08` | Security and credentials | Tenant-aware external systems, credentials, endpoint profiles, permissions for manual reprocess. | Секреты не хранятся в ConfigurationProperty; доступ к секрету и reprocess контролируется Integration Service и IAM. | Open |
| `PCDOC-IC-09` | Validation before publish | Проверки flow, connector, target module/object type, internal contract, mapping, retry/error policy, credential ref, tenant scope и payload version. | Publish не допускает draft/incompatible/unsupported integration configuration. | Open |

## 5. Минимальный состав будущего MVP

Минимальный MVP будущей Integration Capability должен быть сформирован только
после решения о владельце. Candidate scope:

- Integration Service skeleton;
- Integration Configuration kind;
- `ExternalSystem`, `IntegrationFlow`, `IntegrationContractBinding`;
- базовый `MappingDefinition`;
- `ExternalObjectMapping`;
- `IntegrationMessage` и `IntegrationAttempt`;
- outbox/inbox через Integration Events или выбранный transport layer;
- tenant-aware processing и `CorrelationId` propagation;
- retry policy v1 и manual reprocess v1;
- audit integration operations;
- выбранный messaging foundation или ограниченный spike.

## 6. Открытые решения

| ID | Вопрос | Почему нужен выбор | Предполагаемый владелец |
| --- | --- | --- | --- |
| `IC-DEC-01` | Первый transport: broker, PostgreSQL-based transport/storage или комбинация. | От выбора зависит delivery model, operations и тесты. | Architecture / Integration Capability |
| `IC-DEC-02` | Где хранить `ExternalObjectMapping`. | Нужна граница между schema Integration Service, domain modules и shared platform storage. | Integration Capability / Data Architecture |
| `IC-DEC-03` | Формат Integration Configuration storage. | Нужно выбрать typed tables, generic configuration entries или typed projections. | Configuration / Integration Capability |
| `IC-DEC-04` | Формат mapping definitions. | Нужно ограничить mapping без произвольного script в MVP. | Integration Capability / Contracts |
| `IC-DEC-05` | Нужен ли connector SDK. | Влияет на расширяемость и governance external adapters. | Integration Capability |
| `IC-DEC-06` | Где граница MDC telemetry ingestion, Equipment Gateway и Time Series Storage. | Высокочастотная telemetry может требовать отдельный runtime profile. | Integration Capability / Time Series / MDC |
| `IC-DEC-07` | Нужна ли orchestration library для long-running jobs после MVP. | Нельзя заранее подменить Integration Capability внешней orchestration-платформой. | Architecture |

## 7. Связанные документы

- [Интеграционная архитектура DMP](../../02_architecture/07_integration_architecture.md)
- [Integration Events — трассировка](../../03_platform/10_integration_events/90_traceability.md)
- [Backlog Time Series / Telemetry Storage](time_series_storage_backlog.md)
- [Backlog документации Platform Core](../roadmap/preparation/platform_core_documentation_backlog.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
