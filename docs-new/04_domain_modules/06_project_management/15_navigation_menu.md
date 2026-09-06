---
id: DOC-04-06-15
title: 'Меню навигации - 06 Управление проектами'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers: []
scope: domain
module: 06_project_management
holder: '@axelprosoft'
created_at: 2026-09-01 00:00
created_by: '@axelprosoft'
updated_at: 2026-09-03 10:01
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Меню навигации - 06 Управление проектами

## 1. Назначение документа

Документ фиксирует пункт главного меню для объектов PR06 и связь пункта с представлениями Object Runtime.

## 2. Источник меню

Структура меню принята по решению Q11 рабочего журнала PR06 и согласована с
[06_ui_views.md](06_ui_views.md). Ниже фиксируется только целевая структура
меню модуля 06.

## 3. Правила публикации меню

- пункт `Проекты` доступен пользователю с правом просмотра `Project`;
- пункт `Коды затрат` доступен пользователю с правом просмотра `CostCode`;
- при открытии каждого пункта показывается список действующих объектов соответствующего типа;
- фильтр архивности и поиск доступны в соответствии с [06_ui_views.md](06_ui_views.md);
- переход к карточке этапа выполняется из дерева проекта.

## 4. Структура меню

| Уровень | Код пункта | Русское название | Тип | Родитель | Целевое представление | Режим открытия | Порядок | Основание |
|---:|---|---|---|---|---|---|---:|---|
| 1 | `ProjectManagement_Projects` | Проекты | `NavigationItem` | нет | `Project_ListView` | список объектов | 1 | Q11, `06_ui_views.md` |
| 1 | `ProjectManagement_CostCodes` | Коды затрат | `NavigationItem` | нет | `CostCode_ListView` | список объектов | 2 | Q19, `06_ui_views.md` |

## 5. Связь с правами

| Элемент | Требуемое право |
|---|---|---|
| Пункт `Проекты` | `view` для `ProjectManagement:Project` |
| Дерево этапов | `view` для `ProjectManagement:ProjectPhase` |
| Пункт `Коды затрат` | `view` для `ProjectManagement:CostCode` |
| Дерево кодов затрат | `view` для `ProjectManagement:CostCode` |
| Создание и изменение проекта | `create` / `edit` для `ProjectManagement:Project` |
| Создание и изменение этапа | `create` / `edit` для `ProjectManagement:ProjectPhase` |
| Создание и изменение кода затрат | `create` / `edit` для `ProjectManagement:CostCode` |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-03 10:01 +03:00 | Олег Юрьев (@axelprosoft) | Источник меню; Правила публикации меню; Структура меню; Представления без пункта меню; Связь с правами | первые версии модулей 04, 06. правки связанной документации - шаблон и предложения по изменениям ФТ (определения терминов) | [766441e6](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/766441e68e531dadabe887393fc989e5b6e268e0) |
| 1.0 | 2026-09-02 12:21 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #56: мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [PR #56](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/56) |
| 0.1 | 2026-09-02 12:19 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs: exclude working files from versioning | [b04af06e](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/b04af06e9d1fa719ffa8d242a9ee7d33d7d0f30e) |
