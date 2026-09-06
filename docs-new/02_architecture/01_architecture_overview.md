---
id: DOC-02-99-01
title: 'Обзор архитектуры DMP'
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

# Обзор архитектуры DMP

## 1. Назначение

Документ даёт верхнеуровневую архитектурную карту Digital Manufacturing Platform. Он показывает логические слои, основные группы компонентов, границы данных, подход к развёртыванию и связь с интеграционной архитектурой.

Документ не заменяет детальные документы платформенных областей `03_platform` и прикладных модулей `04_domain_modules`.

## 2. Архитектурная рамка

DMP проектируется как единая корпоративная производственная платформа с общими платформенными механизмами и набором прикладных модулей.

Ключевая архитектурная рамка:

- общая кодовая и контрактная основа;
- логическое разделение платформенных областей и прикладных модулей;
- конфигурируемость вместо fork-реализаций под предприятия;
- явные API, application ports и integration events между владельцами;
- гибридная multi-tenant модель данных;
- центральный контур платформы и локальные производственные контуры.

## 3. Логические слои

| Слой | Назначение | Документ-подробность |
| --- | --- | --- |
| User Layer | Web UI, Studio/Admin/Runtime-приложения, рабочие терминалы и frontend-представления. | [Логическая архитектура](03_logical_architecture.md), `03_platform/12_frontend_platform` |
| Application Layer | Прикладная производственная логика и use-cases прикладных модулей. | `04_domain_modules` |
| Platform Layer | Общие платформенные области: configuration, object runtime, workflow, rules, security, value sets, events, audit, content storage и другие. | `03_platform` |
| Data Layer | Хранение данных, владение данными, multi-tenant модель, events/history и analytical storage. | [Архитектура данных](05_data_architecture.md) |
| Интеграционный контур | Обмен между областями и внешними системами через API, application ports и events. | [Интеграционная архитектура](07_integration_architecture.md) |

## 4. Platform Core и прикладные модули

Platform Core задаёт общие правила и механизмы, на которые опираются прикладные модули. Прикладный модуль не должен описывать собственную альтернативную платформу для workflow, rules, object runtime, security, value sets, numbering или audit.

| Группа | Что описывает | Где находится детализация |
| --- | --- | --- |
| Platform Core | Общие механизмы и контракты, независимые от конкретной производственной области. | `03_platform` |
| Domain Modules | Предметные сущности, сценарии, правила и UI конкретной производственной области. | `04_domain_modules` |
| Contracts | API, schema, event и stable code contracts. | Документы-владельцы |
| Runtime behavior | Исполнение, ошибки, конкурентность, кэш, observability и recovery в границах конкретного механизма. | `04_runtime.md`, `07_quality.md` и `08_operations.md` соответствующей области `03_platform` |

## 5. MVP-состав Platform Core

Минимальный состав Platform Core для MVP включает capabilities, без которых
прикладные модули начнут создавать несовместимые локальные механизмы:

| Capability | Роль в MVP | Где уточняется |
| --- | --- | --- |
| Tenant and Security | Tenant context, users, roles, permissions и базовые authorization decisions. | `03_platform/01_tenant_and_security` |
| Configuration | Первичная модель конфигурационных артефактов, версий, публикации и effective configuration. | `03_platform/02_configuration` |
| Object Runtime | Единый путь выполнения операций с бизнес-объектами. | `03_platform/03_object_runtime` |
| Workflow | Первая версия workflow definitions, states, transitions и command execution. | `03_platform/04_workflow` |
| Rules | Первая версия rule evaluation boundary. | `03_platform/05_rules` |
| Value Sets | Наборы допустимых значений, stable codes и значения в области действия. | `03_platform/06_value_sets` |
| Integration Events | Event envelope, outbox и публикация интеграционных событий. | `03_platform/10_integration_events` |
| Audit History | Базовая запись audit/history для значимых действий. | `03_platform/09_audit_history` |
| Content Storage | Platform `ContentRef`, upload policy, descriptor projection, защищённый binary stream и lifecycle attach/detach. | `03_platform/13_content_storage` |
| Contracts and conventions | Публичные API, application ports, stable codes и developer conventions. | Документы-владельцы |

Advanced UI designer, advanced reporting designer, полный low-code цикл,
расширенная аналитика, AI services и сложные cross-tenant orchestration
scenarios относятся к будущим решениям и не должны описываться как обязательная
часть MVP без отдельного ADR или backlog-задачи.

## 6. MVP и физическое развертывание

Архитектурные документы используют термины `service`, `platform service` и `domain service` как обозначения логических границ ответственности.

Для MVP эти границы не означают автоматическое физическое выделение каждого компонента в отдельный развёртываемый сервис. Допустима общая host-композиция, если соблюдаются:

- явные владельцы данных и контрактов;
- запрет прямого доступа к чужим данным как к публичному способу интеграции;
- стабильные application/API/event boundaries;
- возможность последующего физического разделения без переписывания предметной модели.

## 7. Нефункциональная рамка Platform Core

Platform Core должен проектироваться с учётом следующих верхнеуровневых
ожиданий:

- масштабирование корпоративной модели примерно на 20 предприятий;
- изоляция tenant data и явный tenant context для operational data;
- гибридное развёртывание с центральным контуром и локальными производственными контурами;
- расширяемость без fork code base под отдельные предприятия;
- параллельное внедрение на нескольких предприятиях;
- импортонезависимый стек;
- совместимость целевой архитектуры с Linux, PostgreSQL, REST API и event-based integration.

Эта рамка задаёт направление для документов-владельцев. Конкретные SLO,
параметры окружений, отказоустойчивость, observability и recovery-процедуры
должны фиксироваться в документах соответствующих platform areas; сквозной
`07_operations` создаётся только при появлении общего operations policy или
platform-wide runbooks.

## 8. Связанные документы

- [Архитектурные принципы](02_architecture_principles.md)
- [Логическая архитектура](03_logical_architecture.md)
- [Сервисная архитектура](04_service_architecture.md)
- [Архитектура данных](05_data_architecture.md)
- [Deployment-архитектура](06_deployment_architecture.md)
- [Интеграционная архитектура](07_integration_architecture.md)
- [Архитектура безопасности](08_security_architecture.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Логические слои; MVP-состав Platform Core | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 16:54 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: выровнять ID документов по структуре | [662557dd](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/662557dde131463f26ae995734181bc295e22a34) |
