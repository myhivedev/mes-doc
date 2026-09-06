---
id: DOC-02-99-05
title: 'Архитектура данных DMP'
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

# Архитектура данных DMP

## 1. Назначение

Документ фиксирует верхнеуровневую модель данных DMP: типы данных, владельцев, multi-tenant подход и общие правила взаимодействия через контракты.

Документ не является моделью БД и не описывает таблицы конкретных платформенных областей или модулей.

## 2. Типы данных

| Тип данных | Владелец | Примеры |
| --- | --- | --- |
| Platform configuration data | `03_platform/02_configuration` | Configuration scopes, artifact schemas, versions, published/effective configuration. |
| Platform value sets | `03_platform/06_value_sets` | Наборы допустимых значений, stable codes, display names, значения в области действия. |
| Domain master data | `04_domain_modules` | Номенклатура, маршруты, ресурсы, производственная структура. |
| Operational data | Прикладные модули | Заказы, операции, партии, WIP, факты выполнения, результаты контроля. |
| Audit/history data | `03_platform/09_audit_history` и владельцы производителей событий | Audit records, workflow history и object history там, где они поддержаны владельцем. |
| Event data | `03_platform/10_integration_events` и владельцы payload | Outbox records, event envelope, payload, processing state. |
| Platform content data | `03_platform/13_content_storage` | Метаданные ресурсов, непрозрачные `ContentRef`, upload state, ownership и binary bytes; не копируется в object snapshots. |
| Analytical data | Отдельный аналитический контур или владелец BI | KPI, агрегаты, витрины и отчётные наборы. |

## 3. Hybrid multi-tenant model

DMP должна поддерживать общие корпоративные данные и operational data в границе tenant.

| Scope | Значение |
| --- | --- |
| Corporate / global | Общие определения, baseline configuration и данные, которые применяются ко всем предприятиям. |
| Tenant | Предприятие или операционная организационная граница. |
| Site | Производственная площадка внутри tenant, если область поддерживает более точный scope. |

Активные документы должны использовать `Site` как platform scope. `Plant Structure` остаётся названием прикладной производственной области и не заменяет platform scope.

## 4. Правила владения данными

- Каждая область владеет своими данными и отвечает за их целостность.
- Доступ к данным другой области осуществляется через публичный контракт, application port или событие.
- Прямой доступ к таблицам другой области не является публичным контрактом.
- Stable codes используются для устойчивых ссылок между конфигурацией, runtime, frontend и модулями.
- Tenant context должен присутствовать в runtime-операциях с operational data.

## 5. Value Sets и domain master data

Value Sets отвечает за наборы допустимых значений и их элементы. Эта область не является универсальным хранилищем всех справочников.

Domain master data принадлежат прикладным модулям. Например, номенклатура, маршруты, ресурсы и производственная структура не переносятся в Value Sets только потому, что используются как варианты выбора в UI или правилах.

## 6. Event Store и audit/history

Event Store в верхней архитектуре означает хранение событий системы и интеграций. Это не означает обязательный full event sourcing для всех данных.

Audit/history отвечает за трассируемость действий и изменений. Подробные гарантии записи, retention, atomicity и query API описываются владельцами `03_platform/09_audit_history` и `03_platform/10_integration_events`.

## 7. Связанные документы

- [Граница системы](../01_concept/02_system_scope.md)
- [Логическая архитектура](03_logical_architecture.md)
- [Интеграционная архитектура](07_integration_architecture.md)
- [Архитектура безопасности](08_security_architecture.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Типы данных | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 16:54 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: выровнять ID документов по структуре | [662557dd](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/662557dde131463f26ae995734181bc295e22a34) |
