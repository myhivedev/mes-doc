---
id: DOC-03-11-90
title: 'Трассировка и открытые решения — Reporting и Output'
type: traceability
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

# Трассировка и открытые решения — Reporting и Output

## 1. Назначение документа

Документ показывает, что о Reporting/Output подтверждено текущими исходниками,
что перенесено из материалов-источников, где находится владелец содержания и
какие решения не нужно выдавать за текущий контракт.

## 2. Источники и требования

Исходники из `src/` и связанные документы проверены по снимку
`origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed`. Эта ревизия относится
ко всем источникам текущего репозитория в таблице; старые материалы указаны как
источники требований, а не как активные ссылки.

| Источник | Требование или тезис | Решение | Документ-владелец | Статус | Ревизия источника |
| --- | --- | --- | --- | --- | --- |
| [ReportContractController.cs](../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/ReportContractController.cs) | Генерировать и публиковать контракт отчёта | Перенесено в contracts/runtime | Configuration | Подтверждено MVP | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| [ReportDesignOperationsController.cs](../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/ReportDesignOperationsController.cs) | Импортировать, экспортировать и проверять design, создавать preview | Перенесено в contracts/runtime | Configuration | Подтверждено MVP | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| [GenerateOutputService.cs](../../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/GenerateOutputService.cs) | Выполнять Output и Report | Перенесено в architecture/runtime | Reporting/Output | Подтверждено MVP | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| [ReportController.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/ReportController.java) | HTTP API рендеринга и preview | Перенесено в contracts/operations | Report Service | Подтверждено MVP | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| [reportDesignPanel.tsx](../../../src/Frontend/apps/studio/src/features/configuration-artifact-editor/model/reportDesignPanel.tsx), [reportDesign.ts](../../../src/Frontend/packages/contracts/src/reportDesign.ts) и [reportArtifactTreeFilter.test.ts](../../../src/Frontend/apps/studio/src/features/configuration-artifact-editor/model/reportArtifactTreeFilter.test.ts) | UI подготовки design в Studio и распознавание типов | Перенесено в UX/quality; тест подтверждает только распознавание типов | Frontend/Configuration | Подтверждено, но ограничено | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| [runtimeOutputContextBuilder.ts](../../../src/Frontend/packages/runtime-react/src/model/runtimeOutputContextBuilder.ts) и [runtimeOutputLaunchActions.ts](../../../src/Frontend/packages/runtime-react/src/model/runtimeOutputLaunchActions.ts) | Запуск runtime output | Перенесено в UX/границу контрактов | Frontend/Runtime | Подтверждено MVP | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| `docs/01 sources/06.3.6_configuration_platform_output.md` | Общая модель Output и способов выдачи | Подтверждённое MVP перенесено; расширения отделены | Configuration / Reporting Output | Подтверждено, но ограничено | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| `docs/01 sources/06.3.7_configuration_platform_report_output_birt.md` | Поток подготовки design и рендеринга BIRT | Текущий поток перенесён; целевые расширения отделены | Reporting Output / Report Service | Подтверждено, но ограничено | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |
| [07_reports_outputs.md](../../04_domain_modules/02_product_process_definition/07_reports_outputs.md) | Предметное применение reports/outputs | Не переносится в платформенную функциональность | Domain Modules | Владелец другой области | `origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed` |

## 3. Принятые решения

| Тема | Текущее подтверждённое состояние | Маршрут |
| --- | --- | --- |
| Отдельный исходный модуль | Сборки `DMP.Platform.ReportingOutput` нет | Документационная область описывает capability |
| Схемы `Report`/`Output` | Принадлежат Configuration | `02_configuration/artifact_types/` |
| Формирование runtime-результата | Реализовано в `DMP.Platform.Runtime` | `04_runtime.md` |
| Сервис рендеринга | Реализован отдельным Java `DMP.ReportService` | `03_contracts.md`, `08_operations.md` |
| Форматы MVP | Полный маршрут исполнения подтверждает `Pdf` и `Excel` | `03_contracts.md`, `04_runtime.md` |
| Способы выдачи MVP | Подтверждены `Download` и `Inline` | `04_runtime.md` |

## 4. Расхождения и открытые решения

| ID | Тема | Текущее подтверждённое состояние | Влияние на текущую документацию | Статус сведения | Владелец | Следующий шаг | Документ для обновления после решения |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `RO-DEC-01` | Владелец Dataset / Read Query | Reporting использует `IReportDatasetQuery`; общий владелец не закреплён | Не блокирует описание порта runtime | Открытый вопрос | Configuration / Object Runtime | Закрепить владельца общего механизма чтения | `02_architecture.md`, `03_contracts.md` |
| `RO-DEC-02` | Политика retry и timeout Report Service | Timeout подтверждён; единая политика retry не установлена | Не выдавать retry за гарантию | Будущая доработка | Report Service / Operations | Определить retry, backoff и idempotency | `04_runtime.md`, `08_operations.md` |
| `RO-DEC-03` | Выдача результата за пределами HTTP-ответа | `Download`/`Inline` подтверждены; email/storage/schedule нет | Не блокирует MVP | Будущая доработка | Reporting / Platform Operations | Оформить отдельный контракт выдачи | `01_scope.md`, `03_contracts.md` |
| `RO-DEC-04` | Идентичности operational XML | Поддержаны зарегистрированные identities; код не универсален | Неизвестный code останавливает runtime | Подтверждено, но ограничено | Runtime / Reporting | Определить registry и расширение | `02_architecture.md`, `04_runtime.md` |
| `RO-DEC-05` | Формат HTML | Java enum знает `Html`, сопоставление Runtime его не передаёт | Не считать HTML поддержанным сквозным маршрутом | Расхождение кода и источника | Runtime / Report Service | Согласовать сопоставление, MIME и тесты | `03_contracts.md`, `07_quality.md` |
| `RO-DEC-06` | Аудит формирования output | Отдельная запись аудита не подтверждена | Не обещать гарантию аудита | Будущая доработка | Audit History / Tenant/Security | Определить событие/record и retention | `05_security_and_audit.md` |
| `RO-DEC-07` | Полный frontend-сценарий | Модульные тесты есть; сквозной браузерный тест не подтверждён | Не заявлять сквозной браузерный тест | Подтверждено, но ограничено | Frontend Platform / Studio | Определить объём сквозной проверки при необходимости | `06_user_experience.md`, `07_quality.md` |

## 5. Маршрут в целевые документы

| Содержание источника | Куда перенесено | Что не переносится |
| --- | --- | --- |
| Свойства схемы Report/Output | Configuration `artifact_types/report.md` и `output.md` | Не дублируются в этой области |
| Генерация, импорт и preview design | Контракты Configuration и граница runtime Reporting | Внутренние C#-детали реализации |
| Выполнение → XML → файлы → рендеринг | `02_architecture.md`, `04_runtime.md` | Полная реализация BIRT |
| Design отчёта в Studio и запуск output | `06_user_experience.md` | Общая frontend-оболочка |
| Экспорт между test/staging/production | `RO-DEC-03`, backlog | Не выдаётся за текущий контракт |
| Предметные отчёты прикладного модуля | Domain Modules | Не переносится в Platform Core |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
