---
id: DOC-04-04-15
title: 'Меню навигации - 04 Управление ресурсами'
type: design
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers: []
scope: domain
module: 04_resource_management
holder: '@axelprosoft'
created_at: 2026-09-02 13:30
created_by: '@axelprosoft'
updated_at: 2026-09-03 10:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Меню навигации - 04 Управление ресурсами

## 1. Назначение документа

Документ фиксирует целевую структуру главного меню модуля
`04_resource_management` и соответствие конечных пунктов спискам Object
Runtime.

Состав полей, колонок, фильтров, карточек, вкладок и списков выбора описан в
[`06_ui_views.md`](06_ui_views.md). Этот документ не описывает доменную модель,
бизнес-правила или операции.

## 2. Источник меню

Исходный источник структуры меню - раздел «Меню навигации» ПР00:

`C:\@WORK\@СИГМА\01 РАЗРАБОТКА\02 Требования\01 Проектные решения\DMP ПР1\DMP ПР00 Общие требования.docx`

Ниже зафиксирована только целевая структура меню модуля 04.

## 3. Правила публикации меню

- Корневой пункт модуля и вложенные группы являются группами навигации и не
  открывают представление.
- Конечный пункт меню открывает соответствующий список Object Runtime.
- Карточки объектов открываются из списка, lookup, вкладки или связанного
  списка и отдельным пунктом главного меню не являются.
- Вложенные коллекции, описания графиков, назначения графиков, параметры
  операций и другие контекстные представления отдельными пунктами главного
  меню не являются.
- Списки выбора (`lookup`) отдельными пунктами главного меню не являются.
- Отдельные пункты для создания объектов в меню не задаются.
- Видимость конечного пункта определяется правом просмотра соответствующего
  объекта; отдельное право на пункт меню не вводится.

## 4. Структура меню

```text
Управление ресурсами
  Рабочие места
    Группы рабочих мест
    Рабочие места
  Персонал
    Группы сотрудников
    Персонал
    Профессии
    Группы оплаты
    Виды допуска персонала
    Отсутствия сотрудников
  Инструмент и оснастка
    Технологическая оснастка
    Контрольно-измерительный инструмент
  Графики работы
    Праздничные дни
    Типы рабочих дней
    Графики работы
    Модели чередования смен
    Причины отсутствия
```

Таблица целевой структуры меню:

| Уровень | Код пункта | Русское название | Тип | Родитель | Целевое представление | Режим открытия | Порядок |
|---:|---|---|---|---|---|---|---:|
| 1 | `ResourceManagement` | Управление ресурсами | Группа | - | - | - | 400 |
| 2 | `ResourceManagement.WorkPlaces` | Рабочие места | Группа | `ResourceManagement` | - | - | 10 |
| 3 | `ResourceManagement.WorkPlaces.Groups` | Группы рабочих мест | Пункт | `ResourceManagement.WorkPlaces` | `WorkPlaceGroup_ListView` | список / дерево | 10 |
| 3 | `ResourceManagement.WorkPlaces.Items` | Рабочие места | Пункт | `ResourceManagement.WorkPlaces` | `WorkPlace_ListView` | список с деревом производственных единиц | 20 |
| 2 | `ResourceManagement.Personnel` | Персонал | Группа | `ResourceManagement` | - | - | 20 |
| 3 | `ResourceManagement.Personnel.Groups` | Группы сотрудников | Пункт | `ResourceManagement.Personnel` | `PersonnelGroup_ListView` | список / дерево | 10 |
| 3 | `ResourceManagement.Personnel.Items` | Персонал | Пункт | `ResourceManagement.Personnel` | `Personnel_ListView` | список с деревом производственных единиц | 20 |
| 3 | `ResourceManagement.Personnel.Professions` | Профессии | Пункт | `ResourceManagement.Personnel` | `Profession_ListView` | список | 30 |
| 3 | `ResourceManagement.Personnel.PaymentGroups` | Группы оплаты | Пункт | `ResourceManagement.Personnel` | `PaymentGroup_ListView` | список | 40 |
| 3 | `ResourceManagement.Personnel.AllowanceTypes` | Виды допуска персонала | Пункт | `ResourceManagement.Personnel` | `PersonnelAllowanceType_ListView` | список | 50 |
| 3 | `ResourceManagement.Personnel.Absences` | Отсутствия сотрудников | Пункт | `ResourceManagement.Personnel` | `PersonnelAbsence_ListView` | список | 60 |
| 2 | `ResourceManagement.Tools` | Инструмент и оснастка | Группа | `ResourceManagement` | - | - | 30 |
| 3 | `ResourceManagement.Tools.Tooling` | Технологическая оснастка | Пункт | `ResourceManagement.Tools` | `Tooling_ListView` | список | 10 |
| 3 | `ResourceManagement.Tools.Gages` | Контрольно-измерительный инструмент | Пункт | `ResourceManagement.Tools` | `Gage_ListView` | список | 20 |
| 2 | `ResourceManagement.WorkSchedules` | Графики работы | Группа | `ResourceManagement` | - | - | 40 |
| 3 | `ResourceManagement.WorkSchedules.Holidays` | Праздничные дни | Пункт | `ResourceManagement.WorkSchedules` | `Holiday_ListView` | список | 10 |
| 3 | `ResourceManagement.WorkSchedules.DayTypes` | Типы рабочих дней | Пункт | `ResourceManagement.WorkSchedules` | `DayType_ListView` | список | 20 |
| 3 | `ResourceManagement.WorkSchedules.Schedules` | Графики работы | Пункт | `ResourceManagement.WorkSchedules` | `WorkSchedule_ListView` | список | 30 |
| 3 | `ResourceManagement.WorkSchedules.ShiftRotationModels` | Модели чередования смен | Пункт | `ResourceManagement.WorkSchedules` | `ShiftRotationModel_ListView` | список | 40 |
| 3 | `ResourceManagement.WorkSchedules.AbsenceReasons` | Причины отсутствия | Пункт | `ResourceManagement.WorkSchedules` | `AbsenceReason_ListView` | список | 50 |

## 5. Публикация и права

Меню публикуется в общем меню приложения `RuntimeWeb_MainMenu` как
конфигурация `Menu`. Пункты меню публикуются как `NavigationItem`.

Предметные права на объекты и операции описываются в
[`11_permissions.md`](11_permissions.md). Настоящий документ не вводит
собственную матрицу прав.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 10:04 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #59: первые версии прикладных модулей 04 и 06 | [PR #59](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/59) |
| 0.1 | 2026-09-03 10:01 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | первые версии модулей 04, 06. правки связанной документации - шаблон и предложения по изменениям ФТ (определения терминов) | [766441e6](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/766441e68e531dadabe887393fc989e5b6e268e0) |
