---
id: DOC-03-00-03
title: 'Контракты — Foundation'
type: contract
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

# Контракты — Foundation

[application-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/
[domain-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Domain/
[common-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Common/
[module-contracts]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/ModuleContracts.cs
[execution-context]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/ModuleExecutionContext.cs
[use-case]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/IUseCase.cs
[results]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/UseCaseResult.cs
[bulk-results]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/BulkUseCaseResult.cs
[gateway]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/IPlatformRuntimeServicesGateway.cs
[contexts]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/ITenantContext.cs
[scope]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Domain/Abstractions/IScopeBoundEntity.cs
[event]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Domain/Abstractions/IDomainEvent.cs
[common-issue]: ../../../src/Platform/DMP.Platform.Contracts/Common/Messages/PlatformIssue.cs
[message-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Common/Messages/
[message-resolver]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/IPlatformMessageResolver.cs
[message-provider]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/IPlatformMessageTemplateProvider.cs
[common-response]: ../../../src/Platform/DMP.Platform.Contracts/Common/CreatedResponse.cs
[tests]: ../../../tests/DMP.Platform.ArchTests/

## 1. Назначение и границы

Документ описывает межмодульные и повторно используемые контракты общего слоя
Foundation. Он нужен потребителю, который реализует platform capability или
domain module и должен подключиться к единому application-контракту.

Здесь не описываются HTTP-контракты Runtime, Configuration или Tenant Security,
их DTO и event payload. Они принадлежат `03_contracts.md` соответствующих
областей. В Foundation описываются только общие типы, которые имеют более
одного потребителя или задают общую точку интеграции.

## 2. Источники истины и владельцы

| Контрактная группа | Источник кода | Документационный владелец | Потребители |
| --- | --- | --- | --- |
| Domain primitives | `DMP.BuildingBlocks.Domain` | Foundation | Platform и domain modules |
| Application contexts and use-cases | `DMP.BuildingBlocks.Application` | Foundation | Application services, Runtime и modules |
| Platform service ports | `IPlatformRuntimeServicesGateway` и вложенные gateway | Foundation задаёт порт; capability владеет реализацией | Platform areas и domain modules |
| Common transport contracts | `DMP.Platform.Contracts.Common` | Foundation только для общих типов | Несколько platform areas и frontend consumers |
| Area contracts | `DMP.Platform.Contracts.<Area>` | Соответствующая platform area | Её API, frontend и integration consumers |

Физическая сборка не меняет логического владельца. `DMP.BuildingBlocks.Application`
может ссылаться на отдельные общие типы `DMP.Platform.Contracts.Common`, но это
не делает Foundation владельцем всех контрактов библиотеки.

## 3. Карта контрактов

| Контракт | Граница | Потребитель | Стабильная часть |
| --- | --- | --- | --- |
| `IDomainModule` / `IModuleContractRegistry` | Module registration | Composition root, configuration/runtime validation | Module code, contract kind, stable code |
| `ModuleExecutionContext` | Application scenario context | Runtime, Workflow, Rules и domain modules | Tenant/site/user/roles/correlation/module/object/action/time |
| `IUseCase<TRequest, TResult>` | Use-case execution | Application services | `ExecuteAsync` и специализации сценария |
| `UseCaseResult<T>` / `BulkUseCaseResult<TItem>` | Scenario outcome | API/application adapters | Payload/issues и item-level mixed outcome |
| Request contexts | Current request context | Middleware, application services | Tenant/access/language/correlation values |
| `IPlatformRuntimeServicesGateway` | Platform service port | Application layer | Узкие nested gateway и typed result |
| Domain primitives | Domain model base contract | Domain modules and platform domain models | Identity, tenant ownership, audit/event shape |
| `Platform.Contracts.Common` | Shared transport types | API/frontend/platform contracts | Common response, issue, messages, language, list query types |

Подразделы ниже фиксируют только контракты, для которых форма важна на границе
модуля, слоя или composition root. Приватные implementation-типы и полный
каталог helper-методов сюда не включаются.

## 4. HTTP-контракты

У Foundation нет собственного HTTP route и controller. HTTP API конкретных
областей использует общие типы при необходимости, но route, request/response и
ошибки описываются владельцем API. [Общие response-типы][common-response] и
общие проблемы [PlatformIssue][common-issue] не образуют самостоятельный API.

## 5. Общие типы и DTO

| Тип | Русский смысл | Формат | Обязательность/default | Ограничение |
| --- | --- | --- | --- | --- |
| `UseCaseIssue` | Структурированная проблема сценария | `Level`, `Code`, `Message`, optional `FieldCode` | `Level`, `Code`, `Message` обязательны | Код и поле интерпретируются владельцем сценария |
| `PlatformIssue` | Общая структурированная проблема платформы для внутреннего сопоставления с внешним ответом | `Severity`, `Code`, необязательные `MessageTemplateCode`, `Args`, `FieldCode`, `Path`, `InvalidValue`, `FallbackMessage` | `Severity` и `Code` обязательны; остальные поля необязательны | Foundation задаёт форму; код, смысл поля и допустимость `InvalidValue` определяет владелец сценария |
| `PlatformMessageTemplate` | Шаблон сообщения с инвариантным текстом и вариантами по языкам | `Code`, необязательные `InvariantText`, `TextByLanguageCode` | `Code` обязателен; текстовые значения могут отсутствовать | Полное хранение шаблонов через Configuration пока не реализовано |
| `ResolvePlatformMessageRequest` | Запрос на подготовку текста по коду, аргументам и языку | `MessageTemplateCode`, `Args`, `LanguageCode`, `FallbackMessage` | Все поля, кроме запроса в целом, необязательны | Используется средством разрешения сообщений; это не HTTP DTO конкретной области |
| `ResolvedPlatformMessage` | Подготовленный текст сообщения | `Message`, необязательные `TemplateCode`, `LanguageCode`, `UsedFallback` | `Message` и `UsedFallback` обязательны | Результат разрешения текста, а не самостоятельный API-ответ |
| `PlatformAvailabilityResult` | Результат проверки доступности platform service | `IsAllowed` и optional code/message/field | `IsAllowed` обязателен | Смысл кода принадлежит вызываемому capability |
| `PlatformConfigurationValueResult` | Результат чтения значения настройки через gateway | `IsSuccess`, optional value/status | `IsSuccess` обязателен | Configuration владеет смыслом значения |
| `ReferenceDisplay` | Отображаемое представление ссылки | `Id`, optional `Code`, `Title`, optional `Subtitle` | `Id` и `Title` обязательны | Источник и язык разрешения определяет provider |
| `ModuleContractCode` | Зарегистрированный стабильный код модуля | `Kind`, `Code`, optional `Description` | `Kind` и `Code` обязательны | Код не переиспользуется для другого вида без решения |

## 6. C#-контракты и точки расширения

### 6.1. Регистрация модуля и стабильных кодов

`IDomainModule` требует `ModuleCode` и регистрацию контрактов через
`IModuleContractRegistry`. `ModuleContractCodeKind` различает `ModuleCode`,
`ObjectTypeCode`, `ActionCode`, `WorkflowCode`, `ExecutionPointCode`,
`PermissionCode`, `DatasetCode`, `ReportCode`, `OutputCode` и `EventTypeCode`.
Конкретный модуль владеет составом своих кодов; Foundation владеет только
формой регистрации и проверки. ([ModuleContracts.cs][module-contracts])

### 6.2. Контекст выполнения

`ModuleExecutionContext` передаёт tenant, optional site, user, role codes,
correlation id, module code, optional object/action codes и время UTC.
Потребитель не должен создавать локальную несовместимую структуру контекста.
([ModuleExecutionContext][execution-context])

### 6.3. Use-case и результаты

`IUseCase<TRequest, TResult>` задаёт `ExecuteAsync`; `IQueryUseCase`,
`IMutationUseCase`, `ICommandUseCase`, `IBulkUseCase` и `IResolveUseCase` дают
смысл сценария без отдельной framework-реализации. `UseCaseResult<T>` содержит
payload и проблемы, а `BulkUseCaseResult<TItem>` поддерживает результат по
элементам и смешанный успех. ([IUseCase][use-case]; [results][results];
[bulk results][bulk-results])

### 6.4. Контексты запроса

`ITenantContext`, `IRequestAccessContext`, `IRequestLanguageContext` и
`ICorrelationContext` задают чтение текущего контекста. Tenant Security и host
наполняют контекст; application-потребители его используют. Foundation не
утверждает policy authentication или authorization. ([request contexts][contexts])

### 6.5. Gateway к platform services

`IPlatformRuntimeServicesGateway` группирует порты `Authorization`, `Workflow`,
`Rules`, `Configuration`, `ReferenceData`, `Audit` и `Events`. Реализация
каждого порта принадлежит соответствующей capability или composition root.
Foundation не описывает внутренние методы этих capability и не гарантирует,
что каждый порт подключён в конкретной среде. ([gateway][gateway])

### 6.6. Domain primitives и события

`ITenantOwnedEntity`, `IScopeBoundEntity` и `IDomainEvent` являются минимальными
domain-контрактами. Потребитель описывает свои сущности и инварианты, ссылаясь
на Foundation, но не переносит их бизнес-смысл сюда. ([scope][scope];
[domain event][event])

### 6.7. Сообщения и подготовка текста

`IPlatformMessageResolver` задаёт общий application-порт, а
`IPlatformMessageTemplateProvider` поставляет доступные шаблоны. Foundation
описывает форму запроса, результата и шаблона; он не владеет перечнем кодов
сообщений конкретной области. ([порт разрешения сообщений][message-resolver];
[поставщик шаблонов][message-provider]; [общие типы сообщений][message-contracts])

Потребитель передаёт средству разрешения сообщений код шаблона, параметры и язык.
Средство разрешения сообщений возвращает
`ResolvedPlatformMessage`; если шаблон или локализованный текст не найден,
используется резервный текст по правилам `04_runtime.md`. Технические коды и
параметры должны оставаться стабильными для потребителей, но Foundation не
утверждает единый каталог кодов ошибок всех областей.

## 7. Контракты событий

Foundation не владеет event envelope или integration event payload. `IDomainEvent`
является только локальным domain-контрактом. Интеграционные события и outbox
описываются областями `Integration Events`, `Audit History` или capability-
владельцем конкретного события.

## 8. Ошибки и отказоустойчивость

`UseCaseIssue` предназначен для структурированного результата сценария.
`PlatformIssue` является общей внутренней формой проблемы, но не единым
внешним HTTP-ответом. `PlatformInvalidRequestException` предназначен для
нарушения общего application-предусловия, например неизвестного поля сортировки.
HTTP status, внешний error response, набор кодов и политика повтора определяются
владельцем API или операции. Foundation не объявляет общую гарантию retry,
idempotency или delivery.

## 9. Совместимость и изменение контрактов

- Стабильные коды модуля, объекта, действия, workflow, dataset, report, output и события нельзя переименовывать без решения о совместимости.
- Добавление optional поля не должно менять смысл существующего поля.
- Изменение обязательности, типа, семантики результата или границы tenant/site требует проверки всех потребителей.
- Вложенный gateway нельзя считать публичным контрактом конкретной capability без подтверждённого потребителя и реализации.
- Внешние HTTP DTO и event payload изменяются по правилам их владельцев, а не только по правилам Foundation.

## 10. Границы с другими владельцами

| Тема | Foundation | Другой владелец |
| --- | --- | --- |
| Авторизация | `IAuthorizationGateway` и context shape | Tenant Security |
| Конфигурация | Порт `IConfigurationGateway` | Configuration |
| Object Runtime | Общий context/result и application ports | Object Runtime |
| Workflow и Rules | Порты обращения | Workflow и Rules |
| Аудит и события | Порты обращения и domain event base | Audit History и Integration Events |
| HTTP и frontend DTO | Только общие типы при повторном использовании | Platform area или фронтенд-платформа |

## 11. Источники и тесты

| Источник | Что подтверждает |
| --- | --- |
| [BuildingBlocks Application][application-project] | Application abstractions, contexts, gateways, results и helpers |
| [BuildingBlocks Domain][domain-project] | Domain primitives и event/scope abstractions |
| [Platform Contracts Common][common-contracts] | Общие transport types |
| [Architecture tests][tests] | Направление зависимостей и запрет зависимости Foundation от Common module |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
