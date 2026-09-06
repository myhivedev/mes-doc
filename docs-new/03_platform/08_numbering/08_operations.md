---
id: DOC-03-08-08
title: 'Эксплуатация платформенной области — Numbering'
type: operation
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

# Эксплуатация платформенной области — Numbering

## 1. Назначение документа

Документ описывает реальные операции запуска, готовности, диагностики и
восстановления Numbering.

## 2. Запуск и готовность

При запуске host должен:

1. зарегистрировать `AddPlatformNumbering`;
2. зарегистрировать resolver опубликованной Configuration, если Numbering должен
   получать реальные правила;
3. применить миграции Numbering persistence;
4. проверить наличие подключения к выбранному provider-у.

Готовность означает, что `NumberingDbContext` доступен, таблицы `NumberingCounters`
и `NumberingIssueLogs` созданы, а host resolver зарегистрирован. Пакет Numbering
сам по себе не поставляет HTTP readiness endpoint.

## 3. Миграции

Текущая persistence-модель содержит миграцию
`20260816120000_InitialNumberingStorage`, создающую схему `numbering`, таблицы,
индексы и concurrency column. Применение миграции выполняется общим способом,
принятым host-приложением.

## 4. Диагностика

| Ситуация | Технический код или объект | Что проверить | Интерпретация |
| --- | --- | --- | --- |
| Обязательное поле не получило номер | `NUMBERING_RULE_MISSING` | `NumberingEnabled`, момент `OnCreate`, опубликованное правило и resolver | Кандидат правила не найден или не передан |
| Выдача заблокирована из-за двух правил | `NUMBERING_RULE_AMBIGUOUS` | `Priority`, effective scope и условия правил | Максимальный приоритет не определяет одно правило |
| Runtime не находит ни одного правила | `MissingNumberingRuleResolver` | Регистрацию host resolver и опубликованную Configuration | Используется fallback, возвращающий пустой набор |
| Ошибка источника шаблона | `InvalidOperationException` | Значение источника и `NumberingAllowedSourcesJson` | Источник пуст или недоступен |
| Счётчик не обновляется | `numbering.NumberingCounters` | Подключение БД, миграцию и unique key | Ошибка persistence или конкурентного обновления |

## 5. Восстановление

При ошибке операции сначала сохраняются диагностический код и
`CorrelationId`. Выданный до ошибки номер не возвращается в счётчик; повторная
операция получает следующее значение. Ручное изменение `LastNumber` через
Configuration Explorer текущим контрактом не предусмотрено.

## 6. Откат и эскалация

| Ситуация | Действие | Критерий восстановления | Эскалация |
| --- | --- | --- | --- |
| Не применились миграции | Исправить подключение/provider и повторить миграцию | Таблицы и индексы доступны | Platform Operations |
| Не зарегистрирован рабочий resolver | Проверить composition root host | Resolver возвращает опубликованные правила | Configuration + host owner |
| Повторяются пропуски номеров | Проверить failed writes и бизнес-валидацию | Пропуски объясняются журналом/ошибками | Numbering + Domain Module |
| Нарушена уникальность прикладного поля | Проверить object-specific validation и DB constraint | Значения проходят проверки владельца объекта | Domain Module |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
