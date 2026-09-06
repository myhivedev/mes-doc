---
id: DOC-03-00-00
title: 'Обзор платформенной области — Foundation'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: foundation
holder: '@axelprosoft'
created_at: 2026-08-26 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed
---

# Обзор платформенной области — Foundation

[domain-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Domain/
[application-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/
[infrastructure-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Infrastructure/
[common-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Common/
[platform-template]: ../00_platform_documentation_template.md
[strategy]: ../../00_governance/00_documentation_strategy.md
[tenant-security]: ../01_tenant_and_security/00_platform_overview.md
[configuration]: ../02_configuration/00_platform_overview.md
[object-runtime]: ../03_object_runtime/00_platform_overview.md
[workflow]: ../04_workflow/00_platform_overview.md
[common-module]: ../../04_domain_modules/00_common/00_module_overview.md
[platform-terms]: ../../11_glossary/platform_terms.md

## 1. Назначение области

Foundation — общий технический слой платформенного ядра. Он предоставляет
минимальные доменные примитивы, application-контракты, контексты запроса,
общие результаты сценариев и универсальные вспомогательные правила, которыми
могут пользоваться платформенные области и прикладные модули.

Foundation не является отдельным бизнес-модулем и не владеет предметным
смыслом сущностей. Его задача — задать единые точки сопряжения и базовые
ограничения, чтобы Tenant Security, Configuration, Object Runtime и доменные
модули не создавали несовместимые локальные варианты одних и тех же механизмов.

## 2. Место в платформе

Foundation находится ниже платформенных capability и прикладных модулей.
В коде он представлен проектами [Domain][domain-project],
[Application][application-project] и [Infrastructure][infrastructure-project].
Проект `DMP.BuildingBlocks.Infrastructure` сейчас содержит только файл проекта
и не предоставляет самостоятельного runtime-поведения.

`DMP.Platform.Contracts` — отдельная библиотека контрактов. В Foundation
описывается только её общая часть [Common][common-contracts]. Контракты
`Runtime`, `Configuration`, `Workflow`, `ValueSets`, `Settings` и других
областей принадлежат документам соответствующих владельцев.

Сквозная карта взаимодействий Platform Core находится в [интеграционной
архитектуре](../../02_architecture/07_integration_architecture.md). В ней
Foundation показан как источник общих типов и портов, а не как владелец
взаимодействий capability-модулей.

## 3. Основные возможности

| Возможность | Назначение | Статус | Основание |
| --- | --- | --- | --- |
| Доменные примитивы | Единые типы идентичности, tenant-владения, аудита, агрегата и доменного события. | Реализовано | [Domain project][domain-project] |
| Контексты выполнения | Единый набор tenant, site, user, role, correlation, language и кодов текущего сценария. | Реализовано | [Application project][application-project] |
| Контракты модулей | Регистрация модуля и его стабильных кодов, а также общий контракт use-case. | Реализовано | [ModuleContracts.cs][module-contracts] |
| Результаты, проблемы и сообщения | Единые результаты сценариев, структурированные проблемы и общий механизм подготовки локализованного текста сообщения. | Реализовано с ограничениями | [UseCaseResult.cs][use-case-result], [BulkUseCaseResult.cs][bulk-result], [общие контракты сообщений][message-contracts], [порт разрешения сообщений][message-resolver] |
| Порты платформенных сервисов | Узкие application-порты для обращения к авторизации, Workflow, Rules, Configuration, справочным данным, аудиту и событиям. | Реализовано с ограничениями | [Gateway contract][gateway] |
| Общие transport-контракты | Общие сообщения, язык, сортировка, группировка и другие типы, используемые несколькими областями. | Реализовано | [Common contracts][common-contracts] |

## 4. Ключевые решения

| Решение | Суть | Документ-владелец |
| --- | --- | --- |
| Foundation не является доменным модулем | Он не владеет бизнес-сущностями, предметными инвариантами и основными данными. | [Граница](01_scope.md) |
| `BuildingBlocks.Entity` не заменяет `CommonObject` | Foundation задаёт минимальную идентичность; `Common` добавляет прикладную бизнес-семантику. | [Common module][common-module] |
| Порт и реализация принадлежат разным владельцам | Foundation задаёт форму общего application-порта, а Tenant Security, Workflow, Rules и другие области реализуют собственную семантику. | [Архитектура](02_architecture.md) |
| Область контракта важнее физической сборки | Общая часть `DMP.Platform.Contracts.Common` относится к Foundation, а типы конкретных capability описываются у их владельцев. | [Контракты](03_contracts.md) |
| Foundation не является deployable service | В текущем коде это библиотечный слой, подключаемый проектами платформы, хостом и модулями. | [Архитектура](02_architecture.md) |

## 5. Зависимости

| Потребитель | Использование Foundation | Обратная зависимость запрещена |
| --- | --- | --- |
| Tenant Security | Контексты tenant/access/language, tenant-aware сущности, unit of work и общие результаты. | Foundation не владеет политикой доступа Tenant Security. |
| Configuration | Доменные примитивы, application-контракты и общие сообщения/запросы. | Foundation не владеет схемами и effective configuration. |
| Object Runtime | `ModuleExecutionContext`, use-case/result-контракты, gateway-порты, локализация и нормализация запросов. | Foundation не владеет runtime-моделью объекта и mutation pipeline. |
| Workflow | Общие примитивы и application-порты по фактической зависимости проекта. | Foundation не подменяет реализацию Workflow. См. [область Workflow][workflow]. |
| Rules, Value Sets, Settings, Numbering, Audit History, Integration Events | Общие примитивы и application-порты по фактической зависимости проекта. | Foundation не подменяет реализацию capability. |
| Domain Modules | Базовые классы, доменные события, контексты, stable codes и use-case conventions. | Foundation не зависит от конкретного доменного модуля. |

Таблица использования конкретным модулем должна находиться в его собственном
обзоре или архитектурном документе. Здесь фиксируется только общий контракт и
граница владения. Термины сверяются с [платформенным глоссарием][platform-terms].

## 6. Статус реализации

Статусы общих возможностей приведены в таблице раздела 3. Foundation
реализован как библиотечный слой общих типов, application-портов и
transport-контрактов; отдельного deployable runtime Foundation не имеет.
Ограничения и расхождения перечислены в [трассировке](90_traceability.md).

## 7. Состав документов

| Документ | За что отвечает |
| --- | --- |
| `00_platform_overview.md` | Роль Foundation и карта общего слоя. |
| `01_scope.md` | Граница Foundation и владельцы соседних механизмов. |
| `02_architecture.md` | Слои, компоненты, зависимости и техническая модель. |
| `03_contracts.md` | Общие domain/application/transport-контракты. |
| `04_runtime.md` | Общие правила создания контекста, результатов и универсальных helper-алгоритмов. |
| `07_quality.md` | Проверяемые ограничения и test evidence. |
| `90_traceability.md` | Источники, расхождения, ограничения и решения по границам. |

Документы `05_security_and_audit.md`, `06_user_experience.md` и
`08_operations.md` для Foundation не создаются: у него нет собственной
политики безопасности, пользовательского интерфейса или deployable runtime.
[module-contracts]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/ModuleContracts.cs
[use-case-result]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/UseCaseResult.cs
[bulk-result]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/BulkUseCaseResult.cs
[gateway]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/IPlatformRuntimeServicesGateway.cs
[message-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Common/Messages/
[message-resolver]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/IPlatformMessageResolver.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
