---
id: DOC-11-99-07
title: 'Термины Reporting и Output DMP'
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

# Термины Reporting и Output DMP

## 1. Назначение документа

Документ фиксирует русские термины, предпочтительные английские названия и
технические алиасы для платформенной области Reporting и Output. Английское
название в отдельной колонке не заменяет русский текст документа: оно нужно для
сопоставления с кодом и межкомпонентными контрактами. Схемы конфигурационных
артефактов остаются у Configuration; этот глоссарий описывает термины
сквозного выполнения и рендеринга.

## 2. Источники терминов

| Источник | Что подтверждает |
| --- | --- |
| [Операции подготовки отчёта](../../src/Platform/DMP.Platform.Configuration/Application/Services/Reports/ReportDesignOperationsApplicationService.cs) | Модель и операции подготовки отчёта (`design-time`) |
| [Исполнение отчёта](../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/GenerateOutputService.cs) | Исполнение отчёта и формирование результата (`output`) |
| [Порты и модели отчёта](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/IReportDatasetExecution.cs) | Порты приложения и runtime-модели |
| [HTTP-сервис отчётов](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/ReportController.java) | HTTP API, рендеринг BIRT, preview и ошибки |
| [Studio и runtime frontend](../../src/Frontend/apps/studio/src/features/configuration-artifact-editor/model/reportDesignPanel.tsx) и [runtime bindings](../../src/Frontend/packages/runtime-react/src/model/runtimeOutputContextBuilder.ts) | UI подготовки design в Studio и запуск результата из runtime-интерфейса |

## 3. Термины-кандидаты Reporting и Output

| Русский термин | Preferred English | Технический алиас | Статус | Определение | Источник в коде | Нежелательные синонимы |
| --- | --- | --- | --- | --- | --- | --- |
| Отчёт | Report | `Report` | Кандидат | Конфигурационное определение набора полей, параметров, фильтров и design, которое может выполнить runtime. Схема принадлежит Configuration. | [Схема Report](../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ReportArtifactSchemas.cs), [runtime-модели](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/RuntimeOutputEngineModels.cs) | Вывод, форма |
| Результат выдачи | Output | `Output` | Кандидат | Конфигурационное определение запуска и выдачи результата, связанное с `Report`. Схема принадлежит Configuration. | [Схема Output](../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/OutputArtifactSchemas.cs), [runtime-модели](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/RuntimeOutputEngineModels.cs) | Отчёт, файл без контекста |
| Design отчёта | Report design | `ReportDesign` | Кандидат | Содержимое и метаданные design, schema и sample XML, используемые для проверки, preview и рендеринга отчёта. | [Операции design](../../src/Platform/DMP.Platform.Configuration/Application/Services/Reports/ReportDesignOperationsApplicationService.cs), [runtime-модели](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/RuntimeOutputEngineModels.cs) | Шаблон отчёта |
| Предварительный просмотр | Preview | `preview`, `ReportDesignPreviewSessionResponse` | Кандидат | Временный просмотр design по идентификатору session и token до обычного запуска runtime. | [HTTP API](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/ReportController.java), [проверка token](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/security/PreviewSessionTokenValidator.java) | Просмотр результата, тестовый отчёт |
| Сеанс предварительного просмотра | Preview session | `ReportDesignPreviewSessionResponse` | Кандидат | Временный идентификатор и срок действия, по которым Report Service отдаёт HTML preview. | [Preview service](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/service/ReportPreviewService.java) | Сессия отчёта без уточнения |
| Рендеринг отчёта | Report rendering | `IReportServiceClient`, `ReportRenderService` | Кандидат | Преобразование файлов design, schema и data в бинарный результат заданного формата. | [Клиент runtime](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/IReportServiceClient.cs), [HTTP API](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/ReportController.java) | Генерация отчёта, отображение |
| Сервис рендеринга отчётов | Report Service | `DMP.ReportService` | Кандидат | Отдельный Java-сервис, который выполняет рендеринг BIRT и preview. | [Приложение Report Service](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/ReportServiceApplication.java), [HTTP API](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/ReportController.java) | Reporting module, Configuration service |
| Набор данных отчёта | Report dataset | `IReportDatasetQuery`, `ReportDatasetQueryRequest` | Кандидат | Данные конкретного отчёта, полученные через порт приложения для чтения. Семантика и владелец общего Dataset этим термином не определяются. | [Исполнение набора данных](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/IReportDatasetExecution.cs), [модели набора данных](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/ReportDatasetModels.cs) | Общий Dataset без владельца |
| Операционный XML | Operational XML | `ReportOperationalXml*` | Кандидат | XML-документ со строками данных и контекстом выполнения, подготовленный для Report Service. | [Модели XML](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/ReportOperationalXmlModels.cs), [материализация XML](../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/ReportOperationalXmlMaterializer.cs) | Schema XML, sample XML |
| Формат результата | Output format | `Pdf`, `Excel`, `Html` | Кандидат | Формат, в котором сервис рендеринга возвращает результат. В полном текущем маршруте подтверждены `Pdf` и `Excel`; наличие `Html` только в Java enum не расширяет этот маршрут. | [Сопоставление форматов](../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/ReportOutputFormatMapper.cs), [Java enum](../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/domain/OutputFormat.java) | Тип отчёта |
| Способ выдачи результата | Output delivery mode | `Download`, `Inline` | Кандидат | Способ возврата сформированного результата из Runtime вызывающему клиенту. | [Модели runtime](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/RuntimeOutputEngineModels.cs), [генерация результата](../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/GenerateOutputService.cs) | Канал доставки, транспорт без уточнения |
| Генерация результата | Output generation | `GenerateOutputService`, `GenerateOutputRequest` | Кандидат | Операция runtime: найти опубликованные определения, получить данные, выполнить рендеринг и вернуть результат. | [Генерация результата](../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/GenerateOutputService.cs), [запрос](../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs) | Публикация отчёта |
| Ссылка на содержимое | Content reference | `ContentRef` | Кандидат | Стабильная ссылка на сохранённое содержимое design, schema или sample, используемая Configuration и Runtime. | [Операции design](../../src/Platform/DMP.Platform.Configuration/Application/Services/Reports/ReportDesignOperationsApplicationService.cs), [runtime-модели](../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/RuntimeOutputEngineModels.cs) | Имя файла, URL без уточнения |

## 4. Границы с соседними глоссариями

| Термин или группа | Владелец определения | Как использовать здесь |
| --- | --- | --- |
| `Report`, `Output`, `ReportDesign` как schema | [Configuration terms](configuration_terms.md) | Этот глоссарий объясняет только их роль в исполнении; свойства и коллекции не дублируются. |
| Dataset / Read Query | [Object Runtime terms](object_runtime_terms.md) или владелец общего механизма чтения после его определения | Использовать только с указанным владельцем; `Report dataset` означает вход конкретного отчёта. |
| Tenant/Site и permission | [Security terms](security_terms.md) | Reporting описывает только фактическое использование context и permission decision. |
| Frontend-рендеринг и экран | [Frontend Platform terms](frontend_platform_terms.md) | Reporting описывает только сценарии интерфейса, специфичные для отчётов. |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
