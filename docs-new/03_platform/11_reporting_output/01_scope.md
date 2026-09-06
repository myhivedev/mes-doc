---
id: DOC-03-11-01
title: 'Граница платформенной области — Reporting и Output'
type: scope
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
updated_at: 2026-08-27 16:52
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed
---

# Граница платформенной области — Reporting и Output

## 1. Назначение документа

Документ фиксирует границы функциональности формирования отчётного результата и
назначает владельцев соседних частей. Он не переносит в эту область схемы
Configuration и не объявляет будущие способы доставки реализованными.

## 2. Что входит

- выполнение опубликованного `Output`, ссылающегося на `Report`;
- получение данных через runtime application-порт;
- подготовка operational XML и файлов design/schema/data;
- вызов `DMP.ReportService` для рендеринга;
- возврат бинарного результата с `contentType`, именем файла и `requestId`;
- граница операций подготовки и редактирования (design-time): генерация,
  импорт/экспорт, проверка и preview в связке с Configuration;
- frontend-сценарии, специфичные для отчётов, в Studio и runtime-интерфейсе.

## 3. Что не входит

| Тема | Почему не входит | Владелец или маршрут |
| --- | --- | --- |
| Схема и свойства `Report`/`Output` | Это конфигурационные артефакты и их контракт публикации | [Configuration](../02_configuration/00_platform_overview.md), [`artifact_types/`](../02_configuration/artifact_types/) |
| Общая модель Dataset и read capability | Reporting только использует порт приложения для чтения данных | Configuration / Object Runtime; решение `CFG-DEC-03` в трассировке Configuration |
| Объектная runtime-модель | Reporting получает контекст, но не владеет объектами | [Object Runtime](../03_object_runtime/00_platform_overview.md) |
| Общая оболочка, маршрутизация и базовые пакеты рендеринга | Это общие frontend-механизмы | [Frontend Platform](../12_frontend_platform/00_platform_overview.md) |
| Роли, permissions и принадлежность Tenant/Site | Reporting вызывает проверку, но не ведёт каталог прав | [Tenant/Security](../01_tenant_and_security/00_platform_overview.md) |
| Запись истории и политика аудита | Отдельный владелец записей аудита | [Audit History](../09_audit_history/00_platform_overview.md) |
| Broker, outbox и доставка integration events | Общая доставка событий не принадлежит Reporting | [Integration Events](../10_integration_events/00_platform_overview.md) |
| Расписание, email, внешнее хранилище и пакетный экспорт | В текущем контракте исполнения не подтверждены | Будущее решение, см. [`90_traceability.md`](90_traceability.md) |

## 4. Граница с соседними областями и модулями

| Сосед | Reporting/Output предоставляет или использует | Сосед владеет |
| --- | --- | --- |
| Configuration | Использует опубликованные определения и ссылки на содержимое design; операции подготовки вызываются через Configuration API | Схемы, редактирование, проверка и публикация `Report`/`Output` |
| Object Runtime | Использует `GenerateOutput` и runtime-контекст; может получать launch binding из представления | Объекты, views и общий runtime-фасад |
| Tenant/Security | Использует Tenant/Site context и permission decision | Каталог прав, принадлежность и policy |
| Frontend Platform | Использует общую оболочку приложения и инфраструктуру рендеринга | Общие приложения, маршрутизация и общие пакеты |
| Audit History | Может передавать request/context для будущей записи | Формат и хранение записи аудита |
| Integration Events | Не требует собственного контракта событий для текущего пути | Envelope, outbox и доставка событий |
| Domain Modules | Использует их механизм чтения данных, если он предоставлен | Бизнес-смысл данных и предметные отчёты |

## 5. Соответствие требованиям

| Требование или ожидание | Текущее покрытие | Документ-владелец |
| --- | --- | --- |
| Сформировать отчёт из опубликованной конфигурации | Подтверждено для `SourceType = Report` | [`04_runtime.md`](04_runtime.md) |
| Получить PDF или Excel | Подтверждено текущим runtime mapping | [`03_contracts.md`](03_contracts.md) |
| Просмотреть design до публикации | Подтверждены validate и preview session | Configuration и [`04_runtime.md`](04_runtime.md) |
| Доставить результат во внешнюю систему или по расписанию | Не подтверждено | [`90_traceability.md`](90_traceability.md) |

## 6. Ограничения версии

- Область описывает текущую распределённую реализацию, а не отдельную сборку.
- Полный runtime-путь поддерживает `Pdf` и `Excel`; наличие `Html` в Java enum не
  расширяет автоматически контракт `GenerateOutput`.
- Текущий способ выдачи ограничен `Download` и `Inline`.
- Известны identity операционного XML только для зарегистрированных кодов отчётов;
  неизвестный код приводит к ошибке runtime.
- Права, аудит, Dataset и frontend shell описываются только в пределах фактического
  использования и не переносятся под ответственность Reporting.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
