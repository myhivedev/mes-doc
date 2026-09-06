---
id: DOC-02-99-08
title: 'Архитектура безопасности DMP'
type: architecture
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: architecture
module: architecture
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 16:54
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
---

# Архитектура безопасности DMP

## 1. Назначение

Документ фиксирует верхнеуровневую архитектуру безопасности DMP: tenant boundary, identity, authorization, data isolation и связи security с audit/history.

Детальная модель пользователей, ролей, прав, security catalog, permission manifests и runtime checks принадлежит `03_platform/01_tenant_and_security`.

## 2. Главные принципы

| Принцип | Значение |
| --- | --- |
| Security as platform capability | Аутентификация, роли, права и проверки доступа являются общей платформенной областью, а не локальной логикой каждого модуля. |
| Tenant-aware runtime | Операции с operational data выполняются в tenant/site context. |
| Backend authorization is mandatory | Видимость элемента в UI не заменяет backend-проверку прав. |
| Least privilege | Пользователь получает только необходимые права в нужной области действия. |
| Audit critical actions | Критичные действия, изменения прав, конфигурации и runtime-операции должны иметь audit trail там, где это поддержано владельцем. |
| No implicit trust between boundaries | Межобластные вызовы должны передавать контекст и проходить проверки, которые требуются владельцем контракта. |

## 3. Tenant and site boundary

Tenant является операционной границей данных, доступа и поведения. Site используется как более точный platform scope, если область поддерживает такой уровень.

| Scope | Использование |
| --- | --- |
| Corporate / global | Общие определения, baseline configuration и shared data. |
| Tenant | Предприятие или операционная граница. |
| Site | Производственная площадка внутри tenant. |

`Plant Structure` остаётся прикладной производственной областью и не заменяет термин `Site` в platform scope.

## 4. Identity and authorization

На верхнем уровне модель включает:

- user identity;
- role assignments;
- permission catalog;
- tenant/site scope;
- authorization decision;
- security manifests платформенных областей и прикладных модулей.

Подробные сущности, коды прав, контракты авторизации и runtime-проверки описывает `03_platform/01_tenant_and_security`.

## 5. Уровни проверки

| Уровень | Правило |
| --- | --- |
| API / host boundary | Запрос должен иметь контекст и проходить требуемую проверку доступа. |
| Application/runtime service | Use-case или runtime-операция проверяет права через платформенный контракт. |
| Workflow/action boundary | Доступная команда workflow или business action требует проверки прав и правил. |
| UI | UI может скрывать недоступные разделы и действия, но не является источником безопасности. |

## 6. Security and audit

Решения безопасности должны быть связаны с audit/history там, где действие критично для трассируемости:

- login/logout и failed authorization;
- изменение пользователей, ролей, прав и назначений;
- публикация конфигурации;
- критичные object runtime mutations;
- workflow commands;
- интеграционные операции.

Фактические гарантии записи, schema audit record, retention и query API принадлежат `03_platform/09_audit_history`.

## 7. Интеграционная безопасность

Внешние интеграции должны иметь отдельную границу безопасности (`security boundary`):

- защищённые API;
- credentials для внешних систем;
- tenant-aware processing;
- external id mapping, если он нужен для сценария;
- audit и diagnostics для критичных интеграционных операций.

Подробности принадлежат интеграционным документам и будущей области интеграций.

## 8. Связанные документы

- [Граница системы](../01_concept/02_system_scope.md)
- [Архитектура данных](05_data_architecture.md)
- [Интеграционная архитектура](07_integration_architecture.md)
- [Tenant and Security](../03_platform/01_tenant_and_security/00_platform_overview.md)
- [Audit History](../03_platform/09_audit_history/00_platform_overview.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:54 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: выровнять ID документов по структуре | [662557dd](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/662557dde131463f26ae995734181bc295e22a34) |
