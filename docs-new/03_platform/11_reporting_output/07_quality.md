---
id: DOC-03-11-07
title: 'Качество и проверяемость — Reporting и Output'
type: assurance
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

# Качество и проверяемость — Reporting и Output

## 1. Назначение документа

Документ отделяет фактическое тестовое свидетельство от свойств, которые ещё
не доказаны кодом или тестами.

## 2. Надёжность

| Отказ | Гарантия | Предел | Восстановление | Проверка |
| --- | --- | --- | --- | --- |
| Не найден опубликованный Output/Report | Запрос завершается различимой ошибкой | Данные должны быть опубликованы | Исправить конфигурацию и повторить | runtime-тесты |
| Java Report Service недоступен | Клиент возвращает ошибку недоступности | Синхронный вызов зависит от сервиса | Проверить готовность (`readiness`) и связь | Клиент `ReportServiceClient` |
| Некорректный design | Validate возвращает issues | Проверка относится к контракту design | Исправить или импортировать design | интеграционные тесты Configuration |
| Неизвестна identity XML | Выполнение останавливается до рендеринга | Поддержаны зарегистрированные коды отчётов | Добавить identity и тест | runtime-тесты |

## 3. Наблюдаемость

| Сигнал | Источник | Корреляция | Потребитель | Диагностический пробел |
| --- | --- | --- | --- | --- |
| `requestId` | Runtime и Report Service | `X-Report-RequestId` | API-клиент и журналы | Общая политика сквозной трассировки не подтверждена |
| Код ошибки рендеринга | Java `GlobalExceptionHandler` | идентификатор запроса | Runtime и эксплуатация | Полная метрика ошибок не подтверждена |
| Готовность сервиса (`readiness`) | `/health/ready` | Нет | Развёртывание и эксплуатация | Автоматическое включение в readiness host не подтверждено |

## 4. Производительность

| Сценарий нагрузки | Предел или SLO | Метод | Результат |
| --- | --- | --- | --- |
| Синхронный рендеринг | Timeout Runtime и Report Service — 60 секунд | Настройка `ReportServiceOptions.Timeout` и timeout Java-рендеринга | Значение подтверждено конфигурацией; нагрузочный SLO не задан |
| Размер импорта design | До 52 428 800 байт на endpoints Configuration | `RequestSizeLimit` | Ограничение подтверждено контроллером |
| Время хранения файлов данных | TTL по `DataFileTtl` | Очистка при публикации файлов | Значение по умолчанию — 1 час; отдельный эксплуатационный SLO не задан |

## 5. Проверки и результаты

| Сценарий проверки | Уровень | Источник теста | Результат запуска | Пробел |
| --- | --- | --- | --- | --- |
| HTTP-маршрут рендеринга Report Service | Интеграционный тест Java | [ReportControllerIntegrationTest.java](../../../src/Services/DMP.ReportService/src/test/java/com/dmp/reportservice/api/ReportControllerIntegrationTest.java) | Тест существует | Результат текущего запуска не фиксируется этим документом |
| Рендеринг BIRT | Интеграционный тест Java | [BirtReportRendererIntegrationTest.java](../../../src/Services/DMP.ReportService/src/test/java/com/dmp/reportservice/birt/BirtReportRendererIntegrationTest.java) | Тест существует | Полный набор форматов требует отдельной проверки |
| Token preview | Модульный тест Java | [PreviewSessionTokenValidatorTest.java](../../../src/Services/DMP.ReportService/src/test/java/com/dmp/reportservice/security/PreviewSessionTokenValidatorTest.java) | Тест существует | Не описана ротация секрета |
| Путь и доступ к файлу | Модульный тест Java | [ReportFileResolverTest.java](../../../src/Services/DMP.ReportService/src/test/java/com/dmp/reportservice/service/ReportFileResolverTest.java) | Тест существует | Права развёртывания требуют проверки окружения |
| Контекст и действия runtime output | Модульный тест frontend | [runtimeOutputContextBuilder.test.ts](../../../src/Frontend/packages/runtime-react/src/model/runtimeOutputContextBuilder.test.ts), [runtimeOutputLaunchActions.test.ts](../../../src/Frontend/packages/runtime-react/src/model/runtimeOutputLaunchActions.test.ts) | Тесты существуют | Сквозной браузерный тест не заявлен |
| Фильтр типов артефактов Report | Модульный тест frontend | [reportArtifactTreeFilter.test.ts](../../../src/Frontend/apps/studio/src/features/configuration-artifact-editor/model/reportArtifactTreeFilter.test.ts) | Подтверждено распознавание `Report` и `ReportDesign` | Отдельный тест `ReportDesignPanel` и полный пользовательский сценарий не подтверждены |

## 6. Неподтверждённые свойства

Не считаются установленными: автоматические retry/replay, фоновые задания,
нагрузочный SLO, полная сквозная трассировка, внешняя доставка, гарантированная
запись аудита каждого результата и полноценный сквозной браузерный сценарий.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
