---
id: DOC-03-00-90
title: 'Трассировка — Foundation'
type: traceability
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
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Трассировка — Foundation

[domain-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Domain/
[application-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/
[infrastructure-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Infrastructure/
[common-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Common/
[common-module]: ../../04_domain_modules/00_common/00_module_overview.md
[architecture-tests]: ../../../tests/DMP.Platform.ArchTests/Architecture/
[localization-tests]: ../../../tests/DMP.Platform.ArchTests/Localization/LocalizedTextResolverTests.cs

## 1. Назначение документа

Документ связывает текущие исходники `BuildingBlocks` и общей части
`Platform.Contracts.Common` с нормативным пакетом Foundation. Он не является
каталогом исходников и не заменяет документы platform areas или domain modules.

## 2. Источники и требования

| Источник | Ожидание или тезис | Что подтверждено | Документ-владелец | Состояние |
| --- | --- | --- | --- | --- |
| `DMP.BuildingBlocks.Domain` | Общие доменные примитивы без бизнес-смысла конкретного модуля | `Entity`, tenant/audit/aggregate primitives, scope и domain event abstractions реализованы | `02_architecture.md`, `03_contracts.md` | Подтверждено MVP |
| `DMP.BuildingBlocks.Application` | Единые контексты, use-case, результаты и application ports | Реализованы context, registry, use-case markers, results, gateway и helper-контракты | `03_contracts.md`, `04_runtime.md` | Подтверждено MVP |
| `DMP.Platform.Contracts.Common` | Общие transport-типы для нескольких capability | В `Common` находятся общие сообщения, язык, list query и response types; area-specific namespaces остаются у владельцев областей | `03_contracts.md` | Подтверждено, но граница требует вычитки |
| `DMP.BuildingBlocks.Infrastructure` | Общая инфраструктурная точка подключения | Отдельного поведения и persistence-классов в текущем проекте нет | `02_architecture.md` | Подтверждено, но ограничено |
| `00_common` | Прикладная общая бизнес-модель использует Foundation | `CommonObject` использует `BuildingBlocks.Entity`, но `Common` не является частью Foundation | [Common module][common-module] | Подтверждено MVP |
| `docs/01 sources/03_platform_core.md` | Старый общий обзор Platform Core: границы, состав capability, MVP scope, данные, интеграции и ограничения для прикладных модулей | Общие тезисы разнесены в `01_concept`, `02_architecture` и документы platform areas; Foundation оставляет только собственную границу `BuildingBlocks` и общих application-контрактов | `00_platform_overview.md`, `01_scope.md`, `02_architecture.md`, `03_contracts.md`; `02_architecture/01_architecture_overview.md` | Маршрут закрыт, staging-копия не является источником истины |
| `docs/01 sources/13_platform_api_and_contracts.md` | Старый общий обзор Platform API, stable codes, runtime context, Object Runtime API, bulk/long operations и compatibility rules | Foundation забирает только общие application/result/context/stable-code формы; Object Runtime и backlog владеют своими частями | `03_contracts.md`; `03_platform/03_object_runtime/03_contracts.md`; [backlog открытых решений Platform API и контрактов](../../10_backlog/contract_governance/platform_api_contracts_backlog.md) | Маршрут закрыт, staging-копия не является источником истины |

## 3. Принятые решения

| Решение | Суть | Документ-владелец |
| --- | --- | --- |
| Foundation — общий технический слой | `BuildingBlocks` описывается в `03_platform/00_foundation/`, а не в `04_domain_modules/00_common/`. | `01_scope.md`, `02_architecture.md` |
| Порты и capability разделены | Foundation владеет формой application-порта, соседняя область — реализацией и предметным смыслом. | `03_contracts.md` |
| Общая часть `Platform.Contracts.Common` отделена от area contracts | В Foundation описываются только общие transport-типы; Runtime/Configuration/Workflow и другие контракты остаются у владельцев. | `03_contracts.md` |

## 4. Расхождения и открытые решения

| ID | Тема | Ожидание или источник | Текущее подтверждённое состояние | Расхождение или неопределённость | Влияние на текущую документацию | Статус сведения | Владелец / следующий шаг | Документ для обновления после решения |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `FND-DEC-01` | Граница общей части `Platform.Contracts.Common` | Все типы из `Common` могут считаться общим Foundation-контрактом. | В `Common` есть общие типы, но фактический круг потребителей нужно подтвердить по namespace и contracts. | Не определено, какие типы являются cross-cutting, а какие следует документировать у конкретной области. | Не блокирует текущую документацию | Открытый вопрос | Foundation / владельцы platform areas; составить матрицу потребителей перед структурным разделением контрактов. | `03_contracts.md`, документы областей |
| `FND-DEC-02` | Production-использование no-op gateway | Модули без подключённой capability могут использовать allow-all fallback. | `AllowAllPlatformRuntimeServicesGateway` реализован; его production-допустимость не подтверждена. | Не определена политика запрета или допуска fallback в production composition. | Не блокирует текущую документацию | Открытый вопрос | Architecture / Operations / владельцы capability; определить правило до промышленной готовности. | `02_architecture.md`, `07_quality.md`, `08_operations.md` после создания |
| `FND-DEC-03` | Tenant-ограничение `AggregateRoot` | Корень агрегата может быть универсальной базой для всех моделей. | `AggregateRoot` наследует `AuditableTenantEntity` и требует `TenantId`. | Global-модель не может использовать этот тип без дополнительного решения или другого базового типа. | Не блокирует текущую документацию | Подтверждено, но ограничено | Foundation / владельцы доменных моделей; явно выбирать базовый тип в архитектуре каждого модуля. | `02_architecture.md` потребителя |
| `FND-DEC-04` | Состав `DMP.BuildingBlocks.Infrastructure` | Infrastructure может содержать общие adapters и persistence. | Текущий проект не содержит самостоятельного поведения. | Неясно, останется ли проект пустой точкой подключения или получит общие реализации. | Будущая доработка | Будущая доработка | Foundation / Architecture; не создавать документацию или код до появления подтверждённой ответственности. | `02_architecture.md` после появления реализации |
| `FND-DEC-05` | Зависимость Application от Platform.Contracts | Foundation application layer должен быть независим от area transport contracts. | `BuildingBlocks.Application` прямо использует несколько типов `DMP.Platform.Contracts.Common`. | Не принято целевое решение о сохранении этой зависимости или выделении независимого общего contract layer. | Не блокирует текущую документацию | Открытый вопрос | Architecture; оценить границу при отдельном migration/layering этапе, не менять текущий MVP автоматически. | `02_architecture.md`, `03_contracts.md` |

## 5. Маршрут в целевые документы

| Тема | Foundation | Документы потребителей |
| --- | --- | --- |
| Базовый класс сущности и tenant scope | Определение и общий инвариант | Доменный модуль показывает наследование и собственный смысл |
| `ModuleExecutionContext` | Состав и правила создания | Область показывает, где контекст используется |
| `IUseCase` и результаты | Общая форма | Модуль показывает свои request/result contracts |
| Gateway к platform services | Порт и граница | Capability описывает implementation, policy и ошибки |
| HTTP/DTO/events | Только общая часть | Владелец route/event описывает полный внешний контракт |
| Старый обзор Platform Core | Foundation забирает только общие примитивы, контексты и application-порты | Верхняя архитектура описывает общий состав Platform Core; отдельные capability описывают собственные runtime, contracts, security и quality документы |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
