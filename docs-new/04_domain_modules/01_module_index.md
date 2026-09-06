---
id: DOC-04-99-01
title: 'Индекс прикладных модулей'
type: appendix
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: domain_modules
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-09-02 12:21
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
---

# Индекс прикладных модулей

## 1. Назначение

Документ фиксирует рабочий состав прикладных модулей DMP и связывает номер,
код, каталог и текущий статус документации модуля.

Индекс не описывает платформенные области. Платформенное ядро ведётся в
`03_platform/`, а этот документ перечисляет только прикладные модули из
утверждённой структуры `04_domain_modules/`.

## 2. Как читать индекс

| Поле | Смысл |
| --- | --- |
| `ModuleNo` | Двузначный номер модуля в структуре `04_domain_modules`. |
| `ModuleCode` | Технический код модуля для документации и ссылок. |
| `ModuleFolder` | Каталог модуля в `docs-new/04_domain_modules/`. |
| Название | Русское и английское название модуля. |
| Владелец | Рабочий владелец документации модуля. |
| Статус документации | Текущий статус подготовки пакета документов. |
| Основные документы | Точка входа в документацию модуля, если она уже оформлена. |
| Примечания | Ограничения, объединения или уточнения границы. |

## 3. Реестр модулей

| ModuleNo | ModuleCode | ModuleFolder | Название | Владелец | Статус документации | Основные документы | Примечания |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 00 | `common` | `00_common` | Общие правила прикладных модулей / Common Domain Module Rules | `module:common` | draft | [02_domain_model.md](00_common/02_domain_model.md), [05_rules.md](00_common/05_rules.md) | Общие решения для прикладных модулей; не является предметным модулем очереди. |
| 01 | `general_master_data` | `01_general_master_data` | Общая НСИ и основные данные / General Master Data | `module:general_master_data` | draft | [00_module_overview.md](01_general_master_data/00_module_overview.md) | Ведёт общие справочные и основные данные. |
| 02 | `product_process_definition` | `02_product_process_definition` | Составы и технологии / Product & Process Definition | `module:product_process_definition` | draft | [90_traceability_pr02.md](02_product_process_definition/90_traceability_pr02.md), [backlog.md](02_product_process_definition/backlog.md) | Единый модуль по требованиям до отдельного решения о разделении Product и Process. |
| 03 | `plant_structure` | `03_plant_structure` | Производственная структура / Plant Structure | `module:plant_structure` | draft | [90_traceability_pr03.md](03_plant_structure/90_traceability_pr03.md) | Использует термин `Site` для платформенной области; предметная структура описывается отдельно. |
| 04 | `resource_management` | `04_resource_management` | Управление ресурсами / Resource Management | `module:resource_management` | draft | [90_traceability_pr04.md](04_resource_management/90_traceability_pr04.md) | Сохраняет рабочую папку `_working/` для текущей проработки. |
| 05 | `document_management` | `05_document_management` | Управление документацией / Document Management | `module:document_management` | draft | [90_traceability_pr05.md](05_document_management/90_traceability_pr05.md) | Пакет находится в начальной стадии трассировки. |
| 06 | `project_management` | `06_project_management` | Управление проектами / Project Management | `module:project_management` | draft | [00_module_overview.md](06_project_management/00_module_overview.md), [90_traceability_pr06.md](06_project_management/90_traceability_pr06.md), [backlog.md](06_project_management/backlog.md) | Комплект документов создан в draft; прикладной workflow в MVP не вводится. |
| 07 | `order_management` | `07_order_management` | Управление заказами / Order Management | `module:order_management` | draft | - | Документы модуля ещё не оформлены. |
| 08 | `planning_scheduling` | `08_planning_scheduling` | Планирование и расписания / Advanced Planning & Scheduling | `module:planning_scheduling` | draft | - | Переходные материалы пока находятся в `offers/` и должны быть разобраны отдельно. |
| 09 | `production_logistics` | `09_production_logistics` | Производственная логистика / Production Logistics | `module:production_logistics` | draft | - | Документы модуля ещё не оформлены. |
| 10 | `shopfloor_execution` | `10_shopfloor_execution` | Управление операциями / Shopfloor Operations Management | `module:shopfloor_execution` | draft | - | Документы модуля ещё не оформлены. |
| 11 | `quality_management` | `11_quality_management` | Управление качеством / Quality Management | `module:quality_management` | draft | - | Документы модуля ещё не оформлены. |
| 12 | `machine_data_collection` | `12_machine_data_collection` | Мониторинг оборудования / Machine Data Collection | `module:machine_data_collection` | draft | - | Не является платформенной Integration Capability. |
| 13 | `manufacturing_analytics` | `13_manufacturing_analytics` | Производственная аналитика / Manufacturing Analytics | `module:manufacturing_analytics` | draft | - | Документы модуля ещё не оформлены. |

## 4. Связанные документы

- [Состав системы DMP](../01_concept/06_system_composition.md)
- [Шаблон документации прикладного модуля](00_module_documentation_template.md)
- [Backlog подготовки документации ядра платформы](../10_backlog/roadmap/preparation/platform_core_documentation_backlog.md)

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-02 12:21 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #56: мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [PR #56](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/56) |
| 0.2 | 2026-09-02 12:19 +03:00 | Олег Юрьев (@axelprosoft) | Реестр модулей | docs: exclude working files from versioning | [b04af06e](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/b04af06e9d1fa719ffa8d242a9ee7d33d7d0f30e) |
| 0.1 | 2026-08-27 16:40 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: нормализовать модульные шаблоны и проверки ID | [e9d751cf](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/e9d751cf8728cd963b9d160870b884481b162219) |
