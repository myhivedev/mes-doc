---
id: DOC-04-04-10
title: 'Использование ValueSetData - 04 Управление ресурсами'
type: design
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers: []
scope: domain
module: 04_resource_management
holder: '@axelprosoft'
created_at: 2026-09-02 14:00
created_by: '@axelprosoft'
updated_at: 2026-09-03 10:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Использование ValueSetData - 04 Управление ресурсами

## 1. Назначение документа

Документ фиксирует использование настраиваемых наборов значений
(`ValueSetData`) в модуле `04_resource_management`.

Полный состав фиксированных перечислений модуля приведен в
[`02_domain_model.md`](02_domain_model.md). Механизм хранения, публикации и
переопределения наборов значений относится к платформе и здесь не повторяется.

## 2. Решение

Модуль 04 не вводит и не поставляет собственные `ValueSetData` в текущей
версии.

Фиксированные перечисления ПР04, включая типы ресурсов, типы графиков,
периоды, дни недели, типы интервалов и типы назначений графиков, описываются
как `SystemEnum` модуля в `02_domain_model.md`.

## 3. Справочники модуля

`Profession`, `PaymentGroup`, `PersonnelAllowanceType`, `AbsenceReason`,
`DayType`, `WorkSchedule` и `ShiftRotationModel` являются самостоятельными
объектами модуля, а не наборами `ValueSetData`.

Ссылка на `ToolBase` и ссылки на внешние номенклатурные объекты не являются
использованием `ValueSetData`.

## 4. Поля со ссылкой на ValueSet

В целевой доменной модели модуля 04 нет полей типа `ValueSetCode` и нет полей,
которые должны получать значения из произвольного `ValueSetData`.

## 5. Baseline

Baseline модуля 04 не регистрирует и не поставляет `ValueSetData`.

## 6. Правило для будущих расширений

Если в модуле появится настраиваемая классификация, ее нельзя добавлять как
`ValueSetData` без отдельного проектного решения. Такое решение должно
определить владельца набора, код `ValueSet`, область данных, способ выбора и
связь с полем доменной модели. После принятия решения сведения добавляются в
этот документ, `02_domain_model.md` и `03_object_runtime_model.md` в нужной
части.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 10:04 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #59: первые версии прикладных модулей 04 и 06 | [PR #59](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/59) |
| 0.1 | 2026-09-03 10:01 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | первые версии модулей 04, 06. правки связанной документации - шаблон и предложения по изменениям ФТ (определения терминов) | [766441e6](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/766441e68e531dadabe887393fc989e5b6e268e0) |
