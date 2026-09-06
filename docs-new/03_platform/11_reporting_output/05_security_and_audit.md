---
id: DOC-03-11-05
title: 'Безопасность и аудит — Reporting и Output'
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

# Безопасность и аудит — Reporting и Output

## 1. Назначение документа

Документ показывает реальные меры защиты и границы аудита при операциях подготовки
и runtime операциях. Каталог ролей и policy принадлежит Tenant/Security.

## 2. Ресурсы и права

| Ресурс | Действие | Область действия | Политика или роль | Источник решения |
| --- | --- | --- | --- | --- |
| Отчёт и design Configuration | Создать, изменить, импортировать, проверить, открыть preview | Версия Configuration | `Configuration.Entry.Edit` | Контроллеры Configuration |
| Runtime Output | Сформировать результат | Tenant/Site и объектный контекст | `Output.PermissionCode`, если задан | `GenerateOutputService`, runtime-проверка прав |
| Сеанс preview | Открыть HTML preview | Идентификатор session и token | Проверка token preview | `DMP.ReportService` |

## 3. Принятие решения и управление

Runtime использует контекст Tenant/Site и вызывает проверку прав, когда
у `Output` задан `PermissionCode`. Reporting/Output не создаёт роли, не ведёт
каталог permission и не определяет принадлежность пользователя к Tenant/Site.
Контроллеры операций подготовки design требуют `Configuration.Entry.Edit`.

Preview использует идентификатор session и token; Java-сервис проверяет token перед выдачей
HTML. Секрет подписи и TTL задаются конфигурацией Report Service.

## 4. Контроли и риски

| Угроза | Контроль | Подтверждение | Остаточный риск |
| --- | --- | --- | --- |
| Запуск без права | Проверка permission по `PermissionCode` | [GenerateOutputService](../../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/GenerateOutputService.cs) | Если permission code не задан, отдельная проверка output не выполняется |
| Доступ к чужому контексту | Tenant/Site передаются провайдеру определений и в контекст Tenant | [Runtime-контракты](../../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/RuntimeOutputEngineModels.cs) | Политика принадлежности находится у Tenant/Security |
| Выход за каталог через имя файла | Java DTO принимает только имя с ожидаемым расширением | [RenderReportRequest.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/dto/RenderReportRequest.java) | Безопасность каталога и учётной записи сервиса требует контроля развёртывания |
| Использование просроченного preview token | Подпись и срок действия token | [PreviewSessionTokenValidator.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/security/PreviewSessionTokenValidator.java) | Реализация ротации секрета не описана текущим контрактом |

## 5. Аудит

| Событие аудита | Инициатор | Контекст | Запись | Покрытие | Подтверждение |
| --- | --- | --- | --- | --- | --- |
| Design import/publish | Configuration | User и configuration version | Отдельная запись Reporting не подтверждена | Владелец аудита — Configuration/Audit History | Текущие сервисы |
| Формирование runtime-результата | Runtime | идентификатор запроса, Tenant/Site и контекст Output | Отдельная запись аудита не подтверждена | Не считать запись гарантированной | [GenerateOutputService](../../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/GenerateOutputService.cs) |
| Ошибка рендеринга | Runtime/Report Service | идентификатор запроса и код ошибки | Ответ с ошибкой и журналы; запись аудита не подтверждена | Операционный сигнал | [GlobalExceptionHandler.java](../../../src/Services/DMP.ReportService/src/main/java/com/dmp/reportservice/api/GlobalExceptionHandler.java) |

## 6. Покрытие и пробелы

- Подтверждены проверки permission для операций подготовки design и условная проверка
  `Output.PermissionCode` для runtime.
- Не подтверждены обязательная запись каждого output, неизменяемый журнал аудита,
  срок хранения и связь записи аудита с бизнес-историей.
- Решения по этим темам принадлежат Configuration, Audit History и Tenant/Security,
  а не создаются этим документом.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
