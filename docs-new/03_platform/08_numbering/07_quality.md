---
id: DOC-03-08-07
title: 'Качество и проверки платформенной области — Numbering'
type: assurance
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
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Качество и проверки платформенной области — Numbering

## 1. Назначение документа

Документ перечисляет проверяемые свойства Numbering и фактические свидетельства.

## 2. Надёжность

| Свойство | Свидетельство | Состояние |
| --- | --- | --- |
| Последовательное выделение одного счётчика | `NumberingServiceIntegrationTests` | Подтверждено для проверенных сценариев |
| Разные разделы имеют независимые последовательности | Тест `IssueAsync_DifferentPartitionKeys_AllocatesIndependentSequences` | Подтверждено |
| Номер не возвращается после ошибки записи | Тест pipeline с failed write | Подтверждено |
| Нельзя выдать номер при равном максимальном приоритете | Тест ambiguous rules | Подтверждено |

## 3. Наблюдаемость

`NumberingIssueLog` содержит `CorrelationId`, идентичность правила, счётчика,
объекта и выданное значение. Отдельные метрики, distributed tracing и replay
для Numbering в проверенных источниках не подтверждены.

## 4. Производительность

Уникальный индекс и concurrency token рассчитаны на конкурентное обновление
счётчика. Отдельный тест конкурентности для каждого production database provider-а
в текущем наборе свидетельств не найден. Поэтому общая гарантия нагрузки не
утверждается.

## 5. Проверки и результаты

| Область проверки | Источник | Результат |
| --- | --- | --- |
| Выдача и форматирование | `NumberingServiceIntegrationTests` | Тесты существуют |
| Mutation pipeline | `NumberingServiceIntegrationTests` | Тесты существуют |
| Schema `NumberingRule` | `NumberingRuleSchemaIntegrationTests` | Тесты существуют |
| Публикация и ограничения правила | `NumberingPublishValidationIntegrationTests` | Тесты существуют |
| Production provider concurrency | Отдельного evidence не найдено | Требует отдельной проверки |

## 6. Неподтверждённые свойства

- выполнение на `OnSave`, `OnPublish`, `OnWorkflowTransition`, `OnAction` и
  `ManualRequest`;
- отсутствие пропусков при rollback; напротив, текущая модель допускает пропуск;
- полноценные метрики и distributed tracing;
- отдельный пользовательский интерфейс управления счётчиками.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
