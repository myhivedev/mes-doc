---
id: DOC-03-08-04
title: 'Выполнение платформенной области — Numbering'
type: runtime
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

# Выполнение платформенной области — Numbering

## 1. Назначение документа

Документ описывает фактическое выполнение Numbering при создании объекта.
Подробная схема `NumberingRule` находится у Configuration.

## 2. Основные сценарии

### 2.1. Создание объекта с пустым нумеруемым полем

```text
Create request
 -> Object Runtime применяет входные и effective values
 -> Numbering hook находит пустой ObjectMember с NumberingEnabled
 -> resolver получает опубликованные правила
 -> evaluator проверяет условия и раздел счётчика
 -> service атомарно выделяет sequence number
 -> service форматирует IssuedValue
 -> Object Runtime записывает значение и выполняет обычную validation
 -> после успешного результата пишется NumberingIssueLog
```

### 2.2. Поле уже заполнено

Numbering не вызывается. Значение проходит обычную проверку объекта и политику
ручного ввода, если оно поступило от обычной пользовательской операции.

## 3. Операции и алгоритмы

1. Hook проверяет, что операция имеет вид `Create`.
2. Для каждого `ObjectMember` читаются `NumberingEnabled`, допустимые моменты и
   политики ручного значения/отсутствующего правила.
3. Для пустого поля вызывается `INumberingRuleRuntimeEvaluator` с `OnCreate`.
4. Resolver возвращает правила для поля и контекста; evaluator исключает правила,
   условие которых не выполняется.
5. Из оставшихся правил выбирается максимальный `Priority`.
6. Если на максимальном приоритете осталось более одного правила, возвращается
   `NUMBERING_RULE_AMBIGUOUS`.
7. Формируется `PartitionKey`; затем store находит или создаёт счётчик и
   увеличивает `LastNumber` на `NumberingStep`.
8. `Template` получает sequence number и разрешённые значения источников.
9. Выданное значение записывается в `ObjectMutationContext`.
10. После успешного результата создаётся `NumberingIssueLog`.

## 4. Правила

| Правило | Поведение |
| --- | --- |
| `AssignmentMoment = OnCreate` | Рассматривать правило только при создании объекта |
| Пустое поле | Разрешает автоматическую выдачу при включённой нумерации |
| Заполненное поле | Не вызывает Numbering |
| `NumberingManualInputPolicy = Forbidden` | Запрещает ручное значение в обычной операции |
| `NumberingManualInputPolicy = Allowed` | Разрешает ручное значение |
| `NumberingManualInputPolicy = AdminOnly` | Код значения зарегистрирован; отдельный административный runtime-сценарий не подтверждён |
| `NumberingMissingRulePolicy = Block` | Для необязательного поля отсутствие правила блокирует операцию; обязательное поле и без того не может остаться пустым |
| `NumberingMissingRulePolicy = AllowManual` | Разрешает оставить поле ручным, если это допускают остальные проверки |

Условия, шаблон и допустимые источники определяются в `NumberingRule` и
`ObjectMember`. Numbering не объявляет общий формат условий для других областей.

## 5. Жизненные циклы

```text
Rule: draft -> published -> resolver candidate
Counter: absent -> created -> LastNumber increased
Issue: allocated -> object save succeeds -> issue log written
                         \-> validation/save fails -> gap, no issue log
```

Публикацией правила управляет Configuration. Счётчик создаётся при первой
выдаче для его ключа. Уже выделенное значение не переиспользуется после ошибки.

## 6. Согласованность

Уникальный индекс счётчика защищает от двух строк с одним ключом. `RowVersion`
используется как concurrency token persistence-модели. Выдача номера и запись
прикладного объекта не заявлены как одна общая транзакция: при ошибке после
выделения допускается пропуск номера.

## 7. Сбои и восстановление

| Ситуация | Технический код или объект | Результат |
| --- | --- | --- |
| Правило не найдено для обязательного поля | `NUMBERING_RULE_MISSING` | Операция создания отклоняется |
| Правила неоднозначны | `NUMBERING_RULE_AMBIGUOUS` | Операция создания отклоняется до выдачи |
| Источник шаблона пуст | `InvalidOperationException` | Выдача прекращается |
| Сбой сохранения объекта после выдачи | `NumberingCounters` | Счётчик не откатывается; диагностируется пропуск |
| Пустой resolver | `MissingNumberingRuleResolver` | Кандидаты отсутствуют; host должен зарегистрировать рабочий resolver |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
