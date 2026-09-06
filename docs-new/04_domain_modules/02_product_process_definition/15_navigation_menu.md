---
id: DOC-04-02-15
title: 'Меню навигации - 02 Product & Process Definition'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 02_product_process_definition
holder: '@axelprosoft'
created_at: 2026-08-19 12:02
created_by: '@axelprosoft'
updated_at: 2026-09-02 13:56
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Меню навигации - 02 Product & Process Definition

## 1. Назначение документа

Документ фиксирует пункты главного меню для объектов модуля `02 Product & Process Definition`.

Меню описывает, как пользователь открывает списки составов, технологий, классификаторов, настроек и анализа состава из общего меню приложения. Состав полей, колонок, фильтров, карточек, вкладок и списков выбора описан в `06_ui_views.md`.

## 2. Источник меню

Источник структуры меню - раздел «Меню навигации» ПР00. Ниже фиксируется
только целевая структура меню модуля 02.

## 3. Правила публикации меню

- Меню публикуется как артефакт конфигурации `Menu` в общий код меню `RuntimeWeb_MainMenu`.
- Пункты меню модуля публикуются как `NavigationItem`.
- Раздел `Составы и технологии` публикуется как группа меню.
- Конечные пункты меню открывают списки Object Runtime из `06_ui_views.md`.
- Списки выбора, карточки, вкладки, формы строк и зависимые объекты не получают самостоятельный пункт главного меню.
- Пункты создания объектов в меню модуля v1 не задаются отдельно. Создание выполняется из соответствующего списка стандартным действием Object Runtime.

## 4. Структура меню

| Уровень | Код пункта | Русское название | Тип | Родитель | Целевое представление | Режим открытия | Порядок | Основание |
|---:|---|---|---|---|---|---|---:|---|
| 1 | `ProductProcessDefinition` | Составы и технологии | Группа | - | - | - | 200 | ПР00 |
| 2 | `ProductProcessDefinition.Bills` | Составы | Группа | `ProductProcessDefinition` | - | - | 10 | ПР00 |
| 3 | `ProductProcessDefinition.Bills.ManufacturingBills` | Спецификации | Пункт | `ProductProcessDefinition.Bills` | `ManufacturingBill_ListView` | Список | 10 | ПР00, `06_ui_views.md` |
| 3 | `ProductProcessDefinition.Bills.Substitutions` | Замены номенклатуры | Пункт | `ProductProcessDefinition.Bills` | `NomenclatureSubstitution_ListView` | Список | 20 | ПР00, `06_ui_views.md` |
| 3 | `ProductProcessDefinition.Bills.Compositions` | Анализ составов | Пункт | `ProductProcessDefinition.Bills` | `ProductComposition_ListView` | Список | 30 | ПР00, `06_ui_views.md` |
| 2 | `ProductProcessDefinition.Processes` | Технологии | Группа | `ProductProcessDefinition` | - | - | 20 | ПР00 |
| 3 | `ProductProcessDefinition.Processes.ProcessDefinitions` | Технологические описания | Пункт | `ProductProcessDefinition.Processes` | `ProcessDefinition_ListView` | Список | 10 | ПР00, `06_ui_views.md` |
| 3 | `ProductProcessDefinition.Processes.CooperationSchemes` | Схемы кооперации | Пункт | `ProductProcessDefinition.Processes` | `CooperationScheme_ListView` | Список | 20 | ПР00, `06_ui_views.md` |
| 2 | `ProductProcessDefinition.Dictionaries` | Справочники | Группа | `ProductProcessDefinition` | - | - | 30 | ПР00 |
| 3 | `ProductProcessDefinition.Dictionaries.ProcessingOperations` | Классификатор операций | Пункт | `ProductProcessDefinition.Dictionaries` | `ProcessingOperation_ListView` | Список / дерево | 10 | ПР00, `06_ui_views.md` |
| 3 | `ProductProcessDefinition.Dictionaries.LabourTypes` | Виды работ | Пункт | `ProductProcessDefinition.Dictionaries` | `LabourType_ListView` | Список | 20 | ПР00, `06_ui_views.md` |
| 3 | `ProductProcessDefinition.Dictionaries.ManufacturingOperationTypes` | Виды операций | Пункт | `ProductProcessDefinition.Dictionaries` | `ManufacturingOperationType_ListView` | Список | 30 | Целевое решение 02, `06_ui_views.md` |
| 3 | `ProductProcessDefinition.Dictionaries.WorkEnvironments` | Условия труда | Пункт | `ProductProcessDefinition.Dictionaries` | `WorkEnvironment_ListView` | Список | 40 | ПР00, `06_ui_views.md` |
| 2 | `ProductProcessDefinition.Settings` | Настройки | Группа | `ProductProcessDefinition` | - | - | 40 | ПР00 |
| 3 | `ProductProcessDefinition.Settings.ManufacturingBillTemplates` | Шаблоны спецификаций | Пункт | `ProductProcessDefinition.Settings` | `ManufacturingBillTemplate_ListView` | Список | 10 | ПР00, `06_ui_views.md` |
| 3 | `ProductProcessDefinition.Settings.ProcessTemplates` | Шаблоны ТО | Пункт | `ProductProcessDefinition.Settings` | `ProcessTemplate_ListView` | Список | 20 | ПР00, `06_ui_views.md` |

## 5. Связь с правами

Пункт меню отображается пользователю только при наличии права просмотра целевого объекта.

Предметные роли и права описаны в `11_permissions.md`. Этот документ не вводит отдельные права на пункты меню.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 13:56 +03:00 | Олег Юрьев (@axelprosoft) | Источник меню; Структура меню Product & Process Definition; Структура меню; Представления без пункта меню Product & Process Definition; Связь с правами | изменена струтура описания меню уже выпущенных модулей. содержание не изменено | [454ab8aa](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/454ab8aa3a684a27e9c7adfc61ced9407e1b77bb) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-19 12:02 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [87c0b0a5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/87c0b0a51fdabf232ce704082d41545ae007bedc) |
