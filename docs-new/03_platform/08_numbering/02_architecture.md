---
id: DOC-03-08-02
title: 'Архитектура платформенной области — Numbering'
type: architecture
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

# Архитектура платформенной области — Numbering

## 1. Назначение документа

Документ описывает компоненты Numbering, логическую модель runtime-состояния,
хранение и зависимости. Схема `NumberingRule` здесь не дублируется: её владелец
Configuration.

## 2. Граница и компоненты

```text
Object Runtime
  IObjectMutationPreValidationHook / IObjectMutationLifecycleHook
             |
             v
INumberingRuleRuntimeEvaluator -> INumberingRuleResolver
             |
             v
INumberingService -> INumberingCounterStore -> NumberingCounters
             |
             v
Object Runtime result -> NumberingIssueLogs
```

| Компонент | Ответственность |
| --- | --- |
| `INumberingRuleResolver` | Получить правила для поля, момента и контекста |
| `INumberingRuleRuntimeEvaluator` | Проверить условие и построить кандидатов |
| `INumberingService` | Выделить и отформатировать номер |
| `INumberingCounterStore` | Найти или создать счётчик и атомарно увеличить его |
| Object Runtime hooks | Вызвать Numbering до основной проверки и записать журнал после результата |

## 3. Архитектурная модель и инварианты

```text
NumberingRule (Configuration)
  -> target: ModuleCode + ObjectTypeCode + FieldCode
  -> candidate: condition + assignment moment + effective scope
  -> counter key: RuleIdentity + ScopeId + PartitionKey
  -> issued value: Template + SequenceNumber + source values
```

Каноническое состояние Numbering не включает `NumberingRule` и не заменяет
конфигурационное дерево. Владелец правила хранит конфигурацию, а Numbering
хранит изменяемое runtime-состояние.

Инварианты:

- один ключ `RuleIdentity + ScopeId + PartitionKey` соответствует одному счётчику;
- `NumberingStep` должен быть больше нуля;
- номер выделяется до форматирования результата;
- уже выделенный номер не возвращается в последовательность после ошибки сохранения;
- запись `NumberingIssueLog` создаётся только после успешного результата mutation.

## 4. Persistence-модель и хранение

| Таблица | Назначение | Ключ или индекс | Владелец |
| --- | --- | --- | --- |
| `numbering.NumberingCounters` | Состояние счётчика и последний sequence number | `Id`; unique `(RuleIdentity, ScopeId, PartitionKey)` | Numbering |
| `numbering.NumberingIssueLogs` | Факт выдачи номера и контекст операции | `Id`; индексы по rule/time, value, correlation, counter и object | Numbering |

`NumberingCounterEntry` хранит `RuleIdentity`, `ScopeId`, `PartitionKey`,
`LastNumber`, даты создания/изменения и `RowVersion`. `NumberingIssueLogEntry`
хранит правило, выданное значение, sequence number, объект, tenant/site/user,
`CorrelationId`, ссылки на configuration entry и published version.

Счётчики и журнал находятся в отдельной схеме `numbering`. Они не являются
свойствами `NumberingRule` и не хранятся в Configuration.

## 5. Зависимости и точки расширения

| Компонент Foundation | Как используется | Владелец определения | Семантика области |
| --- | --- | --- | --- |
| `ITenantContext` | Получить tenant/site/user для контекста выдачи | Foundation | Numbering сохраняет контекст в operational log |
| `ICorrelationContext` | Получить correlation id операции | Foundation | Идентифицирует операцию выдачи |
| `IClock` | Получить время выдачи и изменения счётчика | Foundation | Время не является частью правила |
| `ObjectMutationContext` | Получить effective values и контекст создания | Object Runtime | Numbering использует только разрешенные источники |

`INumberingRuleResolver` является точкой расширения host-композиции. Host API
регистрирует resolver опубликованной Configuration и baseline-данных; пакет
Numbering по умолчанию содержит fallback, возвращающий пустой набор правил.

## 6. Технические ограничения

- Numbering не предоставляет HTTP-контракт для редактирования `NumberingRule`;
- текущая атомарность подтверждена реализацией store и тестами с in-memory
  provider, а отдельное испытание конкурентности каждого production provider-а
  требуется перед соответствующей нагрузкой;
- отсутствие `NumberingRule` не является ошибкой самого сервиса выдачи, но может
  привести к ошибке обязательного поля в Object Runtime;
- полная семантика `InheritanceMode` принадлежит Configuration resolver и не
  дублируется здесь;
- persistence-модель не утверждает наличие общего outbox или integration event.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
