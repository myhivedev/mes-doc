---
id: DOC-03-04-01
title: 'Граница платформенной области — Workflow'
type: scope
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: workflow
holder: '@axelprosoft'
created_at: 2026-08-26 18:00
created_by: '@codex'
updated_at: 2026-08-27 16:52
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Граница платформенной области — Workflow

## 1. Назначение документа

Документ фиксирует, какие части исполнения workflow принадлежат платформенной
области Workflow и где проходят границы с Configuration, Rules, Object Runtime,
Tenant Security, Audit History, Integration Events, фронтенд-платформа и
прикладными модулями.

## 2. Что входит

| Часть области | Что делает Workflow |
| --- | --- |
| Определение runtime-маршрута | Получает `WorkflowDefinition` из host resolver и использует states, commands, transitions, guard bindings и state policies |
| Экземпляр workflow | Хранит выбранные `WorkflowCode`, `WorkflowDefinitionRevision` и `CurrentStateCode` для объекта |
| Маршрутизация | Выбирает transition по текущему state, `CommandCode`, порядку и результату guard |
| Публичное выполнение | Предоставляет initialize, state, history, available commands, execute и reassign operations |
| Concurrency | Проверяет `ExpectedStateCode` и `ExpectedConcurrencyToken` и возвращает детерминированный отказ при расхождении |
| История выполнения | Записывает переходы, результат, пользователя, tenant, correlation id и время |
| Интеграционные границы | Вызывает permission, rule, audit, event и archive-state ports |

## 3. Что не входит

- schema и редактор конфигурационного артефакта `Workflow`;
- публикация, effective merge и хранение канонической конфигурации;
- бизнес-смысл объекта, его поля, repository и доменные операции;
- реализация Rule Engine и семантика выражений правил;
- общий mutation pipeline Object Runtime;
- хранение и политика удержания Audit History;
- outbox, доставка, повторная обработка и poison-message policy;
- frontend-редактор, отображение, навигация и локализация интерфейса;
- универсальный BPMN engine, планировщик задач и сложные assignment-сценарии.

## 4. Граница с соседними областями и модулями

| Владелец | Workflow предоставляет или использует | Workflow не присваивает себе |
| --- | --- | --- |
| Configuration | Использует опубликованное effective definition через `IWorkflowDefinitionResolver` | Свойства schema, создание, публикация и effective merge |
| Object Runtime | Передаёт `WorkflowArchiveStateChange`; Object Runtime запрашивает проекцию состояния | Изменение произвольных полей объекта и object mutation pipeline |
| Rules | Передаёт `WorkflowTransitionGuardRequest` для проверки guard | Rule definitions, expression language и итоговую семантику вычислителя |
| Tenant Security | Запрашивает `Workflow.View` и `Workflow.Execute` через `IWorkflowPermissionAuthorizer` | Роли, назначения и каталог разрешений |
| Audit History | Передаёт записи `Workflow.ExecuteCommand` и `Workflow.ReassignInstance` | Формат хранилища, поиск и сроки хранения аудита |
| Integration Events | Публикует `Workflow.CommandExecuted`, `Workflow.InstanceReassigned` и принимает запрос инициализации | Envelope, outbox и транспорт доставки |
| фронтенд-платформа | Предоставляет state, commands, errors и history DTO | Видимость, компоновка и визуальный рендерер |
| Domain Modules | Использует `ModuleCode`, `ObjectTypeCode` и object identity | Предметные правила и бизнес-жизненный цикл |

## 5. Соответствие требованиям

| Тема | Состояние | Целевой документ |
| --- | --- | --- |
| Команда запускает routing, а не `ActionCode` | Подтверждено кодом и тестом сериализации | `03_contracts.md`, `04_runtime.md` |
| Состояние экземпляра закреплено за ревизией definition | Подтверждено persistence и интеграционными тестами | `02_architecture.md`, `04_runtime.md` |
| Guard evaluation | Реализовано через gateway, но часть результатов может быть `NotEvaluated` | `04_runtime.md`, `90_traceability.md` |
| Workflow initialization after object create | Есть event payload и host handler; политика доставки принадлежит Integration Events | `03_contracts.md`, `04_runtime.md` |
| Workflow Editor | Описан в source requirements, но не является текущим backend-контрактом Workflow | Configuration / фронтенд-платформа backlog |

## 6. Ограничения версии

В текущей версии описываются только подтверждённые серверные сценарии. В модели
`WorkflowDefinition` нет исполняемых `Transition.Steps`, `State.EntrySteps` или
`State.ExitSteps`; старые описания таких шагов являются целевой или
неподтверждённой моделью и не превращаются в гарантию MVP.

Отдельный workflow может быть выбран явно. Если для object type опубликовано
несколько кандидатов и selector не получил явный код, host возвращает отказ
неоднозначного выбора. Изменение уже сохранённого экземпляра на latest
definition автоматически не выполняется.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
