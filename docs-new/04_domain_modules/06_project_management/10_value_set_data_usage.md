---
id: DOC-04-06-10
title: 'Использование наборов значений - 06 Управление проектами'
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

# Использование наборов значений - 06 Управление проектами

## 1. Итог

PR06 не использует и не поставляет настраиваемые данные `ValueSetData` / `ValueSetCode` в MVP.

В модели нет модульных списков значений, которые должны настраиваться пользователем или администратором предприятия.

## 2. Системные перечисления

Собственные `SystemEnum` PR06 отсутствуют.

Поле `Status`, найденное в DMP_DATA у `ProjectBase`, не переносится в целевую модель. Для `Project`, `ProjectPhase` и `CostCode` не принят прикладной workflow, поэтому отдельный enum состояния не создается.

Признак архивности (`IsArchived`) является общим полем Common/Object Runtime и не является `ValueSet` или модульным `SystemEnum`.

## 3. Внешние типы значений

| Тип / источник | Использование PR06 |
|---|---|
| `ProductionUnit` | Внешний объект PR03 для ответственного подразделения. |
| `OrganizationalUnit` | Внешний тип PR03 для исполнителя; конкретно допускаются `ProductionUnit` и `Subcontractor`. |
| `Personnel` | Внешний объект PR04 для ответственного сотрудника. |

PR06 не копирует значения внешних перечислений и не создает локальные дубликаты lookup-источников.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-03 10:01 +03:00 | Олег Юрьев (@axelprosoft) | Системные перечисления | первые версии модулей 04, 06. правки связанной документации - шаблон и предложения по изменениям ФТ (определения терминов) | [766441e6](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/766441e68e531dadabe887393fc989e5b6e268e0) |
| 1.0 | 2026-09-02 12:21 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #56: мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [PR #56](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/56) |
| 0.1 | 2026-09-02 12:19 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs: exclude working files from versioning | [b04af06e](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/b04af06e9d1fa719ffa8d242a9ee7d33d7d0f30e) |
