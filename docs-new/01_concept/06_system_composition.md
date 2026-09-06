---
id: DOC-01-99-06
title: 'Состав системы DMP'
type: concept
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: concept
module: concept
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

# Состав системы DMP

## 1. Назначение

Документ фиксирует верхнеуровневую карту состава DMP: платформенные области, прикладные области и внешние/интеграционные контуры.

Документ не заменяет детальную карту прикладных модулей и не описывает внутренние требования к ядру платформы.

## 2. Платформенные области

| Область | Назначение на верхнем уровне | Детализация |
| --- | --- | --- |
| Foundation | Общие технические примитивы, контексты, порты и сквозные контракты. | `03_platform/00_foundation` |
| Tenant and Security | Tenant/site context, identity, роли, права и authorization decisions. | `03_platform/01_tenant_and_security` |
| Configuration | Конфигурационные артефакты, версии, публикация и effective configuration. | `03_platform/02_configuration` |
| Object Runtime | Единый механизм выполнения операций с бизнес-объектами. | `03_platform/03_object_runtime` |
| Workflow | Состояния, переходы, команды workflow и lifecycle runtime. | `03_platform/04_workflow` |
| Rules | Проверка правил и rule evaluation gateway. | `03_platform/05_rules` |
| Value Sets | Наборы допустимых значений, stable codes и значения в области действия. | `03_platform/06_value_sets` |
| Settings | Runtime-настройки модулей и пользовательские предпочтения. | `03_platform/07_settings` |
| Numbering | Правила нумерации, счётчики и выдача номеров. | `03_platform/08_numbering` |
| Audit History | Запись аудита, история действий и общая трассируемость операций. | `03_platform/09_audit_history` |
| Integration Events | Конверт события, outbox, публикация и обработка интеграционных событий. | `03_platform/10_integration_events` |
| Reporting and Output | Определения отчётов и выходных форм, а также runtime формирования результата. | `03_platform/11_reporting_output` |
| Frontend Platform | Общие frontend-приложения, shell, shared packages и runtime contracts. | `03_platform/12_frontend_platform` |
| Content Storage | Метаданные, бинарное содержимое, upload/download и жизненный цикл platform content resources. | `03_platform/13_content_storage` |

## 3. Прикладные области

| Область | Назначение |
| --- | --- |
| General Master Data | Общая НСИ и основные данные. |
| Product and Process Definition | Составы изделий, технологии, маршруты и производственные определения. |
| Plant Structure | Производственная структура как прикладная область. |
| Resource Management | Ресурсы, оборудование как производственный ресурс, инструмент и доступность. |
| Document Management | Управление производственной документацией. |
| Project Management | Управление проектами, если входит в выбранный scope очереди. |
| Order Management | Производственные заказы и их жизненный цикл. |
| Planning and Scheduling | Планирование, расписания, очереди и приоритеты. |
| Production Logistics and WIP | Производственная логистика, партии, комплектация и НЗП. |
| Shopfloor Operations Management | Выполнение операций и рабочие места. |
| Quality Management | Контроль качества, несоответствия и результаты проверок. |
| Machine Data Collection | Сбор данных оборудования, если контур входит в очередь внедрения. |
| Manufacturing Analytics | Производственная аналитика и KPI. |

Актуальный состав и глубина прикладных модулей фиксируются в `04_domain_modules`.

## 4. Интеграционные и внешние контуры

| Контур | Роль |
| --- | --- |
| ERP | Источник заказов, материалов, остатков и владелец финансово-учётных процессов. |
| CAD/PDM | Источник инженерных данных и версий изделий. |
| MDC / оборудование | Источник телеметрии и статусов оборудования. |
| Внешние отчётные или интеграционные сервисы | Подключаются через явные HTTP/API/event-контракты. |

## 5. Правило чтения

Этот документ даёт карту состава. За подробностями читатель должен переходить к документам-владельцам:

- общая архитектура: `02_architecture`;
- платформенные области: `03_platform`;
- прикладные модули: `04_domain_modules`;
- контракты: документы владельцев в `03_platform` и `04_domain_modules`;
- runtime-поведение: `04_runtime.md` в документах владельцев `03_platform`;
- открытые и принятые решения: `90_traceability.md` документов-владельцев и
  `10_backlog`; отдельные ADR создаются только по специальному решению.

## 6. Связанные документы

- [Граница системы](02_system_scope.md)
- [Обзор архитектуры](../02_architecture/01_architecture_overview.md)
- [Сервисная архитектура](../02_architecture/04_service_architecture.md)
- [Интеграционная архитектура](../02_architecture/07_integration_architecture.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Платформенные области | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 16:54 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: выровнять ID документов по структуре | [662557dd](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/662557dde131463f26ae995734181bc295e22a34) |
