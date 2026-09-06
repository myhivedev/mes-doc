---
id: DOC-02-99-04
title: 'Сервисная архитектура DMP'
type: architecture
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: architecture
module: architecture
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-09-03 14:49
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
---

# Сервисная архитектура DMP

## 1. Назначение

Документ описывает сервисные границы DMP на логическом уровне. Он не утверждает, что каждая логическая сервисная граница обязана быть отдельным развёртываемым сервисом в MVP.

## 2. Терминология

| Термин | Значение в этом документе |
| --- | --- |
| Platform service | Логическая область платформенного ядра, владеющая общим механизмом или контрактом. |
| Domain service | Логическая прикладная область, владеющая предметными use-cases и данными. |
| External service | Внешняя система или сервис, с которым DMP взаимодействует через контракт. |
| Host composition | Фактическая сборка нескольких логических областей в одном приложении или runtime-контуре. |

## 3. Platform services

| Логическая сервисная граница | Ответственность | Документ-владелец |
| --- | --- | --- |
| Foundation | Общие типы, контексты и application-порты. | `03_platform/00_foundation` |
| Tenant and Security | Tenant, identity, roles, permissions, authorization decisions. | `03_platform/01_tenant_and_security` |
| Configuration | Configuration scopes, artifacts, versions, publish/effective state. | `03_platform/02_configuration` |
| Object Runtime | Единое исполнение операций с бизнес-объектами. | `03_platform/03_object_runtime` |
| Workflow | Workflow definitions, states, commands, transitions и runtime. | `03_platform/04_workflow` |
| Rules | Rule evaluation boundary. | `03_platform/05_rules` |
| Value Sets | Определения наборов значений, элементы, области действия и API чтения/редактирования. | `03_platform/06_value_sets` |
| Settings | Runtime settings и пользовательские предпочтения. | `03_platform/07_settings` |
| Numbering | Правила нумерации и счётчики. | `03_platform/08_numbering` |
| Audit History | Audit records и граница записи аудита. | `03_platform/09_audit_history` |
| Integration Events | Event envelope, outbox и обработка событий. | `03_platform/10_integration_events` |
| Reporting and Output | Report/output definitions и путь формирования результата. | `03_platform/11_reporting_output` |
| Frontend Platform | Frontend shell, приложения и shared packages. | `03_platform/12_frontend_platform` |
| Content Storage | Метаданные content resources, upload/finalize, attach/detach, descriptor projection и защищённое чтение binary. | `03_platform/13_content_storage` |

## 4. Domain services

Domain services соответствуют прикладным модулям. Они владеют предметной моделью, use-cases, operational data и специфическими правилами производственной области.

Подробный состав domain services находится в `04_domain_modules`. Верхняя архитектура фиксирует только правило: прикладной модуль использует platform services через публичные границы и не читает чужие данные как публичный способ интеграции.

## 5. Integration and external services

Внешние системы подключаются через интеграционные контракты:

- ERP;
- CAD/PDM;
- MDC и оборудование;
- внешние отчётные или интеграционные сервисы;
- аналитические и BI-контуры.

Синхронные и асинхронные взаимодействия описываются в [интеграционной архитектуре](07_integration_architecture.md).

## 6. MVP-ограничение

В MVP логические сервисные границы могут быть реализованы внутри modular monolith или общей host-композиции. Это допустимо, если:

- сервисная граница имеет владельца;
- данные имеют владельца;
- внешнее взаимодействие идёт через контракт;
- будущее физическое выделение сервиса не требует переопределения предметной модели;
- документация не описывает физическое разделение как уже реализованную гарантию.

## 7. Связанные документы

- [Обзор архитектуры](01_architecture_overview.md)
- [Логическая архитектура](03_logical_architecture.md)
- [Интеграционная архитектура](07_integration_architecture.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Platform services | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 16:54 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: выровнять ID документов по структуре | [662557dd](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/662557dde131463f26ae995734181bc295e22a34) |
