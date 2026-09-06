---
id: DOC-04-03-15
title: 'Меню навигации - 03 Plant Structure'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 03_plant_structure
holder: '@axelprosoft'
created_at: 2026-08-18 09:32
created_by: '@axelprosoft'
updated_at: 2026-09-02 13:56
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Меню навигации - 03 Plant Structure

## 1. Назначение документа

Документ фиксирует пункты главного меню для объектов модуля `03 Plant Structure`.

Меню описывает, как пользователь открывает списки производственной структуры из общего меню приложения. Состав полей, колонок, фильтров, карточек, вкладок и списков выбора описан в `06_ui_views.md`.

## 2. Источник меню

Источник структуры меню - раздел «Меню навигации» ПР00. Ниже фиксируется
только целевая структура меню модуля 03.

## 3. Правила публикации меню

- Меню публикуется как артефакт конфигурации `Menu` в общий код меню `RuntimeWeb_MainMenu`.
- Пункты меню модуля публикуются как `NavigationItem`.
- Раздел `Производственная структура` публикуется как группа меню.
- Конечные пункты меню открывают списки Object Runtime из `06_ui_views.md`.
- Списки выбора, карточки, вкладки и зависимые объекты не получают самостоятельный пункт главного меню.
- Пункты создания объектов в меню модуля v1 не задаются отдельно. Создание выполняется из соответствующего списка стандартным действием Object Runtime.

## 4. Структура меню

| Уровень | Код пункта | Русское название | Тип | Родитель | Целевое представление | Режим открытия | Порядок | Основание |
|---:|---|---|---|---|---|---|---:|---|
| 1 | `PlantStructure` | Производственная структура | Группа | - | - | - | 300 | ПР00 |
| 2 | `PlantStructure.ProductionUnits` | Производственные единицы | Пункт | `PlantStructure` | `ProductionUnit_ListView` | Список | 10 | ПР00, `06_ui_views.md` |
| 2 | `PlantStructure.Subcontractors` | Субподрядчики | Пункт | `PlantStructure` | `Subcontractor_ListView` | Список | 20 | ПР00, `06_ui_views.md` |
| 2 | `PlantStructure.Companies` | Организации | Пункт | `PlantStructure` | `Company_ListView` | Список | 30 | ПР00, `06_ui_views.md` |
| 2 | `PlantStructure.StorageAreas` | Зоны хранения | Пункт | `PlantStructure` | `StorageArea_ListView` | Список | 40 | ПР00, `06_ui_views.md` |
| 2 | `PlantStructure.WarehouseBinTypes` | Типы складских ячеек | Пункт | `PlantStructure` | `WarehouseBinType_ListView` | Список | 50 | ПР00, `06_ui_views.md` |
| 2 | `PlantStructure.WarehouseBins` | Складские ячейки | Пункт | `PlantStructure` | `WarehouseBin_ListView` | Табличный список с древовидным режимом | 60 | ПР00, `06_ui_views.md` |

## 5. Связь с правами

Пункт меню отображается пользователю только при наличии права просмотра целевого объекта.

Предметные роли и права описаны в `11_permissions.md`. Этот документ не вводит отдельные права на пункты меню.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 13:56 +03:00 | Олег Юрьев (@axelprosoft) | Источник меню; Структура меню Plant Structure; Структура меню; Представления без пункта меню Plant Structure; Связь с правами | изменена струтура описания меню уже выпущенных модулей. содержание не изменено | [454ab8aa](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/454ab8aa3a684a27e9c7adfc61ced9407e1b77bb) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-18 09:32 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [d6817115](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/d68171152ecd4cef492d582ee7f6b08a6a5da363) |
