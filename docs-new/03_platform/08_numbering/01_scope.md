---
id: DOC-03-08-01
title: 'Граница платформенной области — Numbering'
type: scope
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: numbering
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 16:52
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Граница платформенной области — Numbering

## 1. Назначение документа

Документ определяет, какие данные и операции принадлежат Numbering, а какие
остаются у Configuration, Object Runtime и прикладных модулей.

## 2. Что входит

- выделение следующего значения по `NumberingRule`;
- вычисление `PartitionKey` и ведение независимых счётчиков;
- форматирование значения по `Template`;
- runtime-модели `NumberingCounterEntry` и `NumberingIssueLogEntry`;
- интеграционные application-порты Numbering;
- заполнение пустого нумеруемого поля при создании объекта.

## 3. Что не входит

| Исключение | Владелец и маршрут |
| --- | --- |
| Схема и публикация `NumberingRule` | Configuration, `../02_configuration/artifact_types/numbering_rule.md` |
| Свойства `ObjectMember`, разрешающие нумерацию | Configuration, спецификация `ObjectType` |
| Эффективная конфигурация объекта и mutation pipeline | Object Runtime, `../03_object_runtime/` |
| Поле `Code`, `Number` или `SerialNumber` прикладного объекта | Соответствующий Domain Module |
| Общая оболочка и маршрутизация интерфейса | фронтенд-платформа. В текущем коде отдельный экран Numbering для настройки правил или управления счётчиками не подтверждён; правила редактируются через Configuration |
| Общие права, tenant/site context и авторизация исходной операции | Tenant/Security |
| Общий журнал аудита платформы | Audit History; Numbering хранит только operational log выдачи |

## 4. Граница с соседними областями и модулями

| Соседняя область | Граница взаимодействия | Кто владеет деталями |
| --- | --- | --- |
| Configuration | Передаёт опубликованное правило и ограничения целевого `ObjectMember` | Configuration — schema и публикация; Numbering — runtime-потребление |
| Object Runtime | Вызывает Numbering в create pipeline и принимает результат | Object Runtime — pipeline; Numbering — выдача |
| Tenant/Security | Даёт контекст tenant/site/user и разрешение исходной операции | Tenant/Security |
| Domain Modules | Объявляют нумеруемые поля и проверяют предметную уникальность результата | Каждый Domain Module |
| фронтенд-платформа | Предоставляет общие средства интерфейса | фронтенд-платформа; Configuration владеет editor contract |
| Audit History | Может использовать сведения о выдаче через установленный контракт | Audit History; отдельной integration event гарантии нет |

## 5. Соответствие требованиям

| Требование | Покрытие Numbering | Состояние |
| --- | --- | --- |
| Автоматическое формирование номера прикладного объекта | `OnCreate` для пустого поля через Object Runtime | Подтверждено MVP |
| Настройка правил формирования номера | `NumberingRule` и свойства `ObjectMember` в Configuration | Подтверждено через зависимость |
| Разные последовательности для разных разделов | `PartitionDefinitionJson` и `PartitionKey` | Подтверждено MVP |

## 6. Ограничения версии

- в текущем runtime hook поддержан только момент `OnCreate`;
- значение выдаётся только для пустого целевого поля;
- `OnSave`, `OnPublish`, `OnWorkflowTransition`, `OnAction` и `ManualRequest`
  не являются текущими гарантиями;
- предметная уникальность значения проверяется владельцем объекта, а не
  только Numbering;
- отдельные API просмотра и корректировки счётчиков не подтверждены.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
