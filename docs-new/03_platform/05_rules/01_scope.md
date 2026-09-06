---
id: DOC-03-05-01
title: 'Граница и владельцы - Rules'
type: scope
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: rules
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

# Граница и владельцы - Rules

## 1. Назначение документа

Документ фиксирует, какую техническую возможность предоставляет Rules, какие
границы она пересекает и где находятся подробности соседних областей.

## 2. Что входит

- C#-порт `IRuleEvaluationGateway`;
- типы запроса и ответа оценки привязок;
- коды результата `Satisfied`, `Rejected`, `NotEvaluated`;
- fallback `NotConfiguredRuleEvaluationGateway`;
- host-реализация `RuntimeRuleEvaluationGateway` в пределах текущего MVP;
- правила маршрутизации результата для потребителя Workflow.

## 3. Что не входит

- схема и редактирование конфигурационного артефакта `Rule`;
- схемы `ObjectRuleBinding` и `WorkflowRuleBinding`;
- хранение и вычисление эффективной конфигурации;
- описание бизнес-объекта и обязательные доменные проверки;
- выбор перехода и изменение состояния Workflow;
- HTTP API Rules;
- редактор, рендерер и локальная динамика UI;
- полноценный механизм выполнения выражений (`expression engine`), decision tables,
  scripts и DSL;
- аудит конфигурации, аудит исполнения и доставка событий.

## 4. Граница с соседними областями и модулями

| Граница | Rules предоставляет | Соседний владелец | Что остаётся у соседа |
| --- | --- | --- | --- |
| Configuration | Запрашивает эффективное описание `Rule` по контексту | Configuration | Схема, публикация, слияние и проверка |
| Object Runtime | Передаёт список полей для чтения | Object Runtime | Descriptor, чтение объекта и представление значений |
| Workflow | Возвращает результат проверки guard | Workflow | Выбор применимых bindings и решение о переходе |
| Domain Modules | Не меняет объект | Domain Modules | Предметные инварианты и бизнес-правила |
| Tenant/Security | Не принимает решение о правах | Tenant/Security | Авторизация и permission policy |
| фронтенд-платформа | Не формирует UI | фронтенд-платформа | Editor, рендерер и клиентская отзывчивость |

## 5. Соответствие требованиям

Потребитель передаёт контекст объекта, workflow-команды и набор привязок.
Потребитель обязан обработать `NotEvaluated` отдельно от `Rejected`:
`NotEvaluated` означает, что проверка не выполнена, а не что правило успешно
разрешено.

## 6. Ограничения версии

Текущий MVP ограничен одним портом оценки правил, поддержанным синтаксисом
выражения и fallback-реализацией. Полноценный Rule Engine, HTTP API и отдельный
frontend-сценарий не входят в текущую версию.

### Навигация

Общая карта межмодульных связей находится в
[`07_integration_architecture.md`](../../02_architecture/07_integration_architecture.md).
Подробности схемы `Rule` находятся у Configuration:
[`rule.md`](../02_configuration/artifact_types/rule.md).

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
