---
id: DOC-11-99-05
title: 'Термины Object Runtime DMP'
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
created_at: 2026-08-26 00:00
created_by: '@codex'
updated_at: 2026-08-27 17:07
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@2854252e23fc3c49225f6a6531853dcd4a39e28d
---

# Термины Object Runtime DMP

## 1. Назначение документа

Документ фиксирует рабочие русские термины, английские имена и технические алиасы для платформенной области Object Runtime.

Термины общего платформенного ядра ведутся в [платформенном глоссарии][platform-terms]. Термины Configuration ведутся в [глоссарии Configuration][configuration-terms]. Этот документ содержит только понятия, которые описывают исполняемый контракт объекта, runtime-интерпретацию и выполнение операций над объектами.

Локальные таблицы в документах Object Runtime повторяют ключевые определения для удобства чтения и не заменяют этот глоссарий.

## 2. Источники терминов

| Источник | Что подтверждает |
| --- | --- |
| [Object Runtime overview](../03_platform/03_object_runtime/00_platform_overview.md) | Границу области, локальные определения и состав runtime-возможностей. |
| [Граница Object Runtime](../03_platform/03_object_runtime/01_scope.md) | Владельцев терминов и границы с Configuration, фронтенд-платформой, Workflow, Rules и Domain Modules. |
| [Object Runtime architecture](../03_platform/03_object_runtime/02_architecture.md) | Модель контракта, исполняемого описания, реестра и адаптеров хранилища. |
| [Object Runtime contracts](../03_platform/03_object_runtime/03_contracts.md) | C#-контракты, DTO, события и точки расширения. |
| [Object Runtime runtime](../03_platform/03_object_runtime/04_runtime.md) | Семантику чтения, изменения, действий, иерархии и создания на основании существующего. |
| [Object Runtime code](../../src/Platform/DMP.Platform.Runtime/) | Фактические имена классов, интерфейсов, DTO и сервисов. |

## 3. Термины-кандидаты Object Runtime

| Русский термин | Preferred English | Технический алиас | Статус | Определение | Источник в коде | Нежелательные синонимы |
| --- | --- | --- | --- | --- | --- | --- |
| Исполняемый контракт объекта | Business object contract | `BusinessObjectContract<T>` | Кандидат | Кодовое декларативное описание состава и поведения объекта: идентичности, полей, наборов данных, действий, проверок, обработчиков жизненного цикла, хранилища и политик выполнения. | [контракт объекта](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Objects/BusinessObjectContract.cs) | Схема объекта, конфигурационный объект без уточнения |
| Исполняемое описание объекта | Object runtime descriptor | `ObjectRuntimeDescriptor` | Кандидат | Неизменяемое runtime-представление контракта, используемое реестром и исполнителем стандартных операций над объектом. | [описание объекта](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Objects/ObjectRuntimeDescriptor.cs) | DTO объекта, схема объекта |
| Точка входа Object Runtime | Object Runtime entry point | `IObjectRuntime`, `ObjectRuntime` | Кандидат | Публичная для runtime-служб точка, которая находит исполняемое описание и передаёт ему стандартную операцию над объектом. | [точка входа](../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ObjectRuntime.cs) | Provider как отдельная модель исполнения |
| Конвейер изменения объекта | Object mutation pipeline | `ObjectMutationPipeline` | Кандидат | Единый порядок проверок, транзакционной границы, обработчиков жизненного цикла, предметного выполнения, аудита и события завершения изменения. | [конвейер изменения](../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ObjectMutationPipeline.cs) | Отдельный маршрут сохранения, свободная последовательность обработчиков |
| Набор данных объекта | Object Dataset | `ObjectRuntimeDatasetDescriptor` | Кандидат | Набор данных, связанный с исполняемым описанием конкретного типа объекта и используемый для `List` и `Lookup`. Не является общим Standalone Dataset. | [описание набора данных](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Objects/ObjectRuntimeDescriptor.cs) | Общий Dataset, read-модель без указания владельца |
| Фасад runtime | Runtime Facade | `ApplicationRuntimeService` | Кандидат | Слой, который проверяет доступ, получает опубликованные effective-метаданные, объединяет их с runtime-данными и формирует ответы API для потребителя. | [служба runtime-приложения](../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ApplicationRuntimeService.cs) | Сам Object Runtime, рендерер фронтенда |
| Групповое действие | Bulk action | `RuntimeObjectBulkActionRequest` | Кандидат | Операция действия над явно переданным списком идентификаторов объектов с общим и поэлементным результатом. Выбор по фильтру в текущий контракт не входит. | [контракты группового действия](../../src/Platform/DMP.Platform.Contracts/Runtime/) | Массовое сохранение, действие по всему списку без выборки |
| Создание на основании существующего | Create from existing | `RuntimeObjectCreateFromExistingRequest` | Кандидат | Подготовка несохранённой записи из существующего объекта с переносом только разрешённых значений; постоянная запись, аудит и outbox на этом шаге не создаются. | [служба runtime-приложения](../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ApplicationRuntimeService.cs) | Копирование объекта без политики полей, дублирование |
| Обработчик записи | Mutation writer | `IGenericRuntimeObjectMutationWriter` | Кандидат | Точка расширения прикладного модуля, которая подготавливает и сохраняет состояние объекта внутри конвейера изменения. | [контракт обработчика записи](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Objects/IGenericRuntimeObjectMutationWriter.cs) | Репозиторий как полная замена writer |
| Подключаемый приёмник | Runtime sink | `IAuditHistoryWriter`, `IIntegrationEventPublisher` | Кандидат | Необязательная точка передачи результата изменения во внешний механизм аудита или интеграционных событий после успешной операции. | [конвейер изменения](../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ObjectMutationPipeline.cs) | Владелец аудита или доставки событий |

## 4. Термины, требующие владельца

| Формулировка | Почему требует уточнения | Предпочтительная форма |
| --- | --- | --- |
| Набор данных / запрос чтения (`Dataset / Read Query`) | Может означать набор данных конкретного объекта, общий механизм чтения, аналитический запрос или интеграционный контракт. | Набор данных объекта (`Object Dataset`) либо общий Dataset / Read Query с указанным владельцем. |
| Фасад runtime (`Runtime Facade`) | Текущий код размещает фасад и Object Runtime в одном проекте, но обязанности между ними пересекаются. | Фасад runtime для слоя `ApplicationRuntimeService`; Object Runtime для исполнения описания объекта. Граница требует решения `ORT-DEC-07`. |
| Групповое действие по фильтру (`Bulk action`) | Текущий контракт принимает только явные `ObjectIds`; выбор объектов по фильтру не подтверждён. | Групповое действие по явно выбранным объектам; действие по фильтру — отдельная будущая возможность. |
| Наблюдаемость runtime | В коде не закреплён полный набор метрик, трассировок и проверок работоспособности. | Контракт наблюдаемости Object Runtime после решения `ORT-DEC-08`. |

## 5. Границы применения

| Понятие | Не входит в этот глоссарий |
| --- | --- |
| Configuration | Каноническая модель `ArtifactDocument` / `ArtifactNode` / `ArtifactValue`, схема `ObjectType`, `View`, `Action`, effective-слияние и редактор. |
| Фронтенд-платформа | Renderer, компоновка, элементы управления, навигация и состояние интерфейса. |
| Workflow и Rules | Полная машина состояний workflow и вычисление правил. Object Runtime описывает только точки вызова и проекции, подтверждённые кодом. |
| Audit History и Integration Events | Хранение аудита, политика повторных попыток, outbox/inbox и гарантии доставки. |
| Domain Modules | Предметная модель, инварианты, репозитории, миграции и смысл конкретных бизнес-объектов. |

## 6. Нежелательные формулировки

| Формулировка | Почему не использовать | Чем заменить |
| --- | --- | --- |
| Object Runtime как схема объекта | Смешивает исполняемый механизм со схемой конфигурационного артефакта. | Исполняемый контракт объекта или исполняемое описание объекта. |
| Provider как отдельная модель исполнения | Скрывает текущую точку входа `IObjectRuntime` и роль универсального исполнителя. | Точка входа Object Runtime, исполняемое описание или обработчик записи по смыслу. |
| Dataset без владельца | Неясно, речь об Object Dataset или общем механизме чтения. | Набор данных объекта (`Object Dataset`) либо общий Dataset / Read Query с владельцем. |
| Рендерер фронтенда в документации Object Runtime | Переносит ответственность за интерфейс в серверную область. | Компонент отображения фронтенд-платформы. |
| DTO, projection или derived value как свойство артефакта | Смешивает транспортное или вычисляемое значение с канонической моделью артефакта. | DTO, проекция или вычисляемое runtime-значение по фактическому контексту. |
| Неясная формулировка для UI: `Поверхность / surface` | Неясная калька, не определяющая часть интерфейса. | Интерфейсная часть, раздел интерфейса, экран, сценарий или представление фронтенда. |

[platform-terms]: platform_terms.md
[configuration-terms]: configuration_terms.md

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 17:07 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: исправить порядок ссылок в глоссарии Object Runtime | [493a8228](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/493a82280a122b2826d638952aad874ceacaaf55) |
