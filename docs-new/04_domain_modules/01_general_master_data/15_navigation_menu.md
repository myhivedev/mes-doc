---
id: DOC-04-01-15
title: 'Меню навигации - 01 General Master Data'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 01_general_master_data
holder: '@axelprosoft'
created_at: 2026-08-17 10:31
created_by: '@axelprosoft'
updated_at: 2026-09-02 13:56
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Меню навигации - 01 General Master Data

## 1. Назначение документа

Документ фиксирует пункты главного меню для объектов модуля `01 General Master Data`.

Меню описывает, как пользователь открывает списки GMD из общего меню приложения. Состав полей, колонок, фильтров, карточек, вкладок и списков выбора описан в `06_ui_views.md`.

## 2. Источник меню

Источник структуры меню - раздел «Меню навигации» ПР00. Ниже фиксируется
только целевая структура меню модуля 01.

## 3. Правила публикации меню

- Меню публикуется как артефакт конфигурации `Menu` в общий код меню `RuntimeWeb_MainMenu`.
- Пункты меню GMD публикуются как `NavigationItem`.
- Раздел `Общая НСИ` публикуется как группа меню.
- Конечные пункты меню открывают списки Object Runtime из `06_ui_views.md`.
- Списки выбора, карточки, формы строк и вкладки коллекций не получают самостоятельный пункт главного меню.
- Пункты создания объектов в меню GMD v1 не задаются отдельно. Создание выполняется из соответствующего списка стандартным действием Object Runtime.

## 4. Структура меню

| Уровень | Код пункта | Русское название | Тип | Родитель | Целевое представление | Режим открытия | Порядок | Основание |
|---:|---|---|---|---|---|---|---:|---|
| 1 | `GMD` | Общая НСИ | Группа | - | - | - | 100 | ПР00 |
| 2 | `GMD.UnitGroups` | Группы ЕИ | Пункт | `GMD` | `UnitGroup_ListView` | Список | 10 | ПР00, `06_ui_views.md` |
| 2 | `GMD.Units` | Единицы измерения | Пункт | `GMD` | `Unit_ListView` | Список | 20 | ПР00, `06_ui_views.md` |
| 2 | `GMD.Nomenclatures` | Номенклатура | Пункт | `GMD` | `Nomenclature_ListView` | Список | 30 | ПР00, `06_ui_views.md` |
| 2 | `GMD.NomenclatureGroups` | Группы номенклатуры | Пункт | `GMD` | `NomenclatureGroup_ListView` | Список | 40 | ПР00, `06_ui_views.md` |
| 2 | `GMD.NomenclatureKinds` | Виды номенклатуры | Пункт | `GMD` | `NomenclatureKind_ListView` | Список | 50 | ПР00, `06_ui_views.md` |
| 2 | `GMD.NomenclatureParameters` | Реквизиты номенклатуры | Пункт | `GMD` | `NomenclatureParameter_ListView` | Список | 60 | ПР00, `06_ui_views.md` |
| 2 | `GMD.SerialNumbers` | Серийные номера | Пункт | `GMD` | `SerialNumber_ListView` | Список | 70 | ПР00, `06_ui_views.md` |
| 2 | `GMD.ContractorGroups` | Группы контрагентов | Пункт | `GMD` | `ContractorGroup_ListView` | Список | 80 | ПР00, `06_ui_views.md` |
| 2 | `GMD.Contractors` | Контрагенты | Пункт | `GMD` | `Contractor_ListView` | Список | 90 | ПР00, `06_ui_views.md` |

## 5. Связь с правами

Пункт меню отображается пользователю только при наличии права просмотра целевого объекта.

Предметные роли и права описаны в `11_permissions.md`. Этот документ не вводит отдельные права на пункты меню.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 13:56 +03:00 | Олег Юрьев (@axelprosoft) | Источник меню; Структура меню GMD; Структура меню; Представления без пункта меню GMD; Связь с правами | изменена струтура описания меню уже выпущенных модулей. содержание не изменено | [454ab8aa](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/454ab8aa3a684a27e9c7adfc61ced9407e1b77bb) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-17 10:31 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [66c56b5d](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/66c56b5dc65d9e04759613e71e56bc8b9ebefb4a) |
