---
id: DOC-03-11-03
title: 'Контракты платформенной области — Reporting и Output'
type: contract
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: reporting_output
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

# Контракты платформенной области — Reporting и Output

## 1. Назначение и границы

Документ фиксирует внешние и межкомпонентные контракты Reporting/Output. Схемы
`Report` и `Output` принадлежат [Configuration](../02_configuration/03_contracts.md).
Внутренние методы и private implementation types сюда не входят.

## 2. Источники истины и владельцы

| Контракт | Владелец | Потребитель |
| --- | --- | --- |
| Схемы `Report`/`Output` и операции подготовки design | Configuration | Studio, Runtime и Reporting/Output |
| `GenerateOutputRequest`/`Response` | Runtime-контракты | Интерфейс runtime и другие потребители runtime |
| `IReportDatasetQuery` | Runtime / владелец Dataset | Исполнение отчёта |
| `IReportServiceClient` и HTTP-вызов рендеринга | Граница Reporting/Output | Runtime |
| Контракт preview session | Configuration и Report Service | Studio и браузерный preview |

## 3. Карта контрактов

| Контракт | Канал | Назначение | Владелец |
| --- | --- | --- | --- |
| Подготовка контракта отчёта | HTTP API Configuration | Схема (`schema`), пример XML (`sample XML`) и design по умолчанию (`default design`) | Configuration |
| Операции с design отчёта | HTTP API Configuration | Импорт/экспорт, проверка и сеанс preview | Configuration |
| Формирование runtime-результата | HTTP Runtime API | Получить итоговый файл | Runtime / Reporting Output |
| Рендеринг отчёта | Внутренний HTTP между Runtime и Java-сервисом | Обработать файлы `design`, `schema` и `data` | `DMP.ReportService` |

## 4. HTTP-контракты

### 4.1. API операций подготовки design в Configuration

Базовый маршрут:
`/api/platform/configuration/configuration-versions/{configurationVersionId}/reports/{reportCode}/...`.
Все перечисленные операции требуют policy `Configuration.Entry.Edit`.

| Операция | Маршрутный суффикс | Результат |
| --- | --- | --- |
| Сгенерировать schema | `contracts/schema` | `ReportContractArtifactResponse` |
| Сгенерировать sample XML | `contracts/sample-xml` | `ReportContractArtifactResponse` |
| Сгенерировать default design | `contracts/default-design` | `ReportContractArtifactResponse` |
| Опубликовать schema и sample | `contracts/publish` | Ответы schema/sample и ссылки на содержимое |
| Опубликовать default design | `contracts/default-design/publish` | Ответ design и ссылка на содержимое |
| Экспортировать пакет design | `design/bundle` | ZIP с design/schema/sample |
| Импортировать design | `design/import` | Ссылка на содержимое, hash и `generatedFromBaseline` |
| Проверить design | `design/validate` | `isValid` и список issues |
| Создать сеанс preview | `design/preview-session` | Идентификатор session, URL просмотра и expiry |

Источники маршрутов: [ReportContractController](../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/ReportContractController.cs)
и [ReportDesignOperationsController](../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/ReportDesignOperationsController.cs)
в Configuration.

### 4.2. Runtime output API

`POST /api/runtime/outputs/generate` принимает `GenerateOutputRequest` и
возвращает `GenerateOutputResponse`. `content` во frontend-контракте представлен
строкой, а на HTTP-уровне формируется из бинарного содержимого ответа.

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `outputCode` | String | Нет | Обязательно | Код опубликованного `Output` | Выбирает результат выдачи | Не пустой | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.moduleCode` | String | Нет | Обязательно | Код модуля | Контекст определения | Не пустой | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.objectTypeCode` | String | Нет | Обязательно | Код типа объекта | Контекст отчёта | Не пустой | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.viewCode` | String | Да | Нет | Код View | Контекст запуска из View | Не обязателен | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.listFilters` | Map<String,String?> | Да | Нет | Фильтры списка | Источник сопоставления параметров | Формат определяется провайдером запроса | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.currentObject` | Map<String,String?> | Да | Нет | Поля текущего объекта | Источник сопоставления параметров | Значения не валидируются schema Reporting/Output | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.viewState` | Map<String,String?> | Да | Нет | Состояние View | Контекст запуска | Семантика определяется потребителем | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.languageCode` | String | Да | Нет | Поддержанный код языка | Локаль результата | Разрешается серверным resolver языка | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `context.requestId` | UUID | Да | Генерируется | UUID | Корреляция запроса | Передаётся сервису рендеринга | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |
| `manualParameters` | Map<String,String?> | Да | Нет | Значения параметров отчёта | Явно переданные значения | Объединение выполняет runtime | [GenerateOutputRequest.cs](../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) |

### 4.3. API рендеринга Report Service

`POST /api/reports/render` принимает JSON `RenderReportRequest` и возвращает
бинарное тело. Заголовок `X-Report-RequestId` используется для корреляции.

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `designFileName` | String | Нет | Обязательно | Имя с `.rptdesign` | Design-файл | Не допускает path separator, до 256 символов | [RenderReportRequest.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/dto/RenderReportRequest.java) |
| `schemaFileName` | String | Нет | Обязательно | Имя с `.xsd` | Schema-файл | Не допускает path separator, до 256 символов | [RenderReportRequest.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/dto/RenderReportRequest.java) |
| `dataFileName` | String | Нет | Обязательно | Имя с `.xml` | Операционный XML | Не допускает path separator, до 256 символов | [RenderReportRequest.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/dto/RenderReportRequest.java) |
| `outputFormat` | Enum | Нет | Обязательно | `Pdf`, `Excel`, `Html` в Java enum | Формат рендеринга | Полный runtime mapping сейчас передаёт только `Pdf`/`Excel` | [OutputFormat.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/domain/OutputFormat.java), [ReportOutputFormatMapper.cs](../../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/ReportOutputFormatMapper.cs) |
| `requestId` | UUID | Нет | Обязательно | UUID | Корреляция | Возвращается в заголовке | [RenderReportRequest.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/dto/RenderReportRequest.java) |

## 5. Общие типы и DTO

| Тип | Назначение | Граница |
| --- | --- | --- |
| `GenerateOutputRequest` | Запрос формирования runtime-результата | Runtime-контракты |
| `GenerateOutputResponse` | Бинарный результат и метаданные выдачи | Runtime-контракты |
| `ReportRenderRequest` | Вход порта приложения для сервиса рендеринга | Runtime |
| `ReportRenderResult` | Содержимое, тип, имя файла и идентификатор запроса | Runtime |
| `ReportDatasetQueryRequest` | Запрос строк набора данных | Runtime / владелец Dataset |
| `ReportDatasetQueryResult` | Строки набора данных | Runtime / владелец Dataset |
| `ErrorResponse` | Код, сообщение, идентификатор запроса и детали Java-сервиса | Report Service |

## 6. C#-контракты и точки расширения

| Порт или member | Параметры | Результат | Предусловия | Исключения | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- | --- |
| `IRuntimeReportDefinitionProvider.TryGetPublishedOutputAsync` | Коды модуля, объекта и Output, язык, tenant/site | Опубликованный Output или `null` | Контекст разрешён | Ошибка provider | Чтение | Не задаётся портом |
| `IRuntimeReportDefinitionProvider.TryGetPublishedReportAsync` | Коды модуля, объекта и Report, язык, tenant/site | Опубликованный Report или `null` | Output уже разрешён | Ошибка provider | Чтение | Не задаётся портом |
| `IReportDatasetQuery.QueryAsync` | Запрос набора данных | Строки | Доступен механизм запроса данных | Ошибка провайдера запроса | Чтение источника данных | Владелец Dataset |
| `IReportServiceClient.RenderAsync` | Имена файлов `design`/`schema`/`data`, формат, идентификатор запроса | Результат рендеринга | Файлы опубликованы | Ошибка запроса runtime | HTTP-вызов Java-сервиса | Не распределяет транзакцию |
| `IRuntimeOutputLaunchBindingMaterializer.ProjectAsync` | Контекст View | Launch bindings | Доступен effective-результат View | Ошибка materializer | Чтение effective-конфигурации | Не задаётся портом |

## 7. Контракты событий

В текущем пути Reporting/Output собственный контракт событий не подтверждён.
Публикация или обработка событий не добавляется только потому, что генерация
имеет `requestId`. Если появится событие результата или аудита, его envelope и
доставка будут описаны владельцем [Integration Events](../10_integration_events/03_contracts.md),
а смысл payload — этой областью или владельцем соответствующей операции.

## 8. Ошибки и отказоустойчивость

| Код или тип ошибки | Условие | HTTP или транспортный результат | Поле или путь | Повторить запрос | Ответственный |
| --- | --- | --- | --- | --- | --- |
| `OutputCodeRequired` | Не задан `outputCode` | Ошибка исполнения | `outputCode` | После исправления запроса | Потребитель Runtime |
| `ModuleCodeRequired` | Не задан `context.moduleCode` | Ошибка исполнения | `context.moduleCode` | После исправления запроса | Потребитель Runtime |
| `ObjectTypeCodeRequired` | Не задан `context.objectTypeCode` | Ошибка исполнения | `context.objectTypeCode` | После исправления запроса | Потребитель Runtime |
| `OutputNotFound` | Опубликованный Output не найден | Ошибка исполнения | `outputCode` | После исправления конфигурации | Configuration / Runtime |
| `ReportNotFound` | Опубликованный Report не найден | Ошибка исполнения | `sourceCode` | После исправления конфигурации | Configuration / Runtime |
| `OutputSourceNotSupported` | `SourceType` не равен `Report` | Ошибка исполнения | `sourceType` | Нет до изменения определения | Reporting/Output |
| `OutputFormatUnsupported` | Формат не `Pdf` или `Excel` в маршруте исполнения | Ошибка исполнения | `outputFormat` | Нет до изменения входа | Reporting/Output |
| `OutputDeliveryModeUnsupported` | Способ выдачи не `Download`/`Inline` | Ошибка исполнения | `deliveryMode` | Нет до изменения определения | Reporting/Output |
| `ReportOperationalXmlIdentityUnknown` | Для report code нет XML identity | Ошибка исполнения | `reportCode` | Нет до регистрации identity | Runtime |
| `ReportCodeRequired` | Не задан `ReportCode` при исполнении запроса набора данных | Ошибка исполнения | `reportCode` | После исправления запроса | Runtime |
| `ReportDatasetCodeRequired` | Не задан `DatasetCode` | Ошибка исполнения | `datasetCode` | После исправления конфигурации | Dataset / Runtime |
| `ReportModuleCodeRequired` | Не задан `ModuleCode` при исполнении запроса набора данных | Ошибка исполнения | `moduleCode` | После исправления запроса | Runtime |
| `ReportObjectTypeCodeRequired` | Не задан `ObjectTypeCode` при исполнении запроса набора данных | Ошибка исполнения | `objectTypeCode` | После исправления запроса | Runtime |
| `INVALID_REQUEST` | Запрос рендеринга не прошёл проверку | HTTP 400 от Java-сервиса | Тело запроса | После исправления запроса | Потребитель Report Service |
| `REPORT_FILE_NOT_FOUND` | Java-сервис не нашёл файл | Ошибка HTTP | Имя файла | После восстановления файла | Report Service |
| `REPORT_FILE_ACCESS_DENIED` | Нет доступа к файлу | Ошибка HTTP | Имя файла | После исправления доступа | Report Service |
| `OUTPUT_FORMAT_UNSUPPORTED` | Сервис рендеринга не поддерживает формат | Ошибка HTTP | `outputFormat` | Нет до смены формата | Report Service |
| `RENDER_FAILED` | Ошибка BIRT или пустое содержимое ответа | Ошибка HTTP/исполнения | Ответ рендеринга | По политике вызывающей стороны; retry не гарантирован | Report Service |
| `REPORT_SERVICE_UNAVAILABLE` | Timeout или ошибка соединения | Ошибка исполнения | `BaseUrl`/timeout | Политика retry не установлена | Runtime / Operations |

## 9. Совместимость и изменение контрактов

| Изменение | Совместимо назад | Потребители | Миграция | Версия или решение |
| --- | --- | --- | --- | --- |
| Добавление необязательного поля запроса | Обычно да | Runtime UI, API clients | Не требуется | Проверить общую политику контрактов |
| Изменение `Output.Format` | Нет для неподдержанного значения | Runtime и Report Service | Обновить сопоставление и сервис рендеринга | Решение в трассировке |
| Изменение контракта поля XML | Не всегда | Dataset, materializer, design | Обновить schema/design и тесты | Версия `Report.SchemaVersion` |
| Изменение кода ошибки рендеринга | Требует согласования | Runtime client, operations | Обновить сопоставление и диагностику | Договориться с Report Service |

## 10. Границы с другими владельцами

Поле или свойство конфигурационного артефакта описывается только в Configuration.
В этот документ попадают только его роль в межкомпонентном вызове и подтверждённые
runtime-ограничения. Общие контракты Foundation не копируются; фактические точки
использования описаны в [архитектуре](02_architecture.md).

## 11. Источники и тесты

Источники перечислены в [трассировке](90_traceability.md). Внешними границами
текущего контракта служат [ReportContractController](../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/ReportContractController.cs),
[ReportDesignOperationsController](../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/ReportDesignOperationsController.cs),
[ApplicationRuntimeController](../../../src/Platform/DMP.Platform.Runtime/Api/Controllers/ApplicationRuntimeController.cs),
[IReportServiceClient](../../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/IReportServiceClient.cs)
и Java [ReportController](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/ReportController.java).

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
