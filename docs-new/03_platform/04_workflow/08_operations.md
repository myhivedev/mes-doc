---
id: DOC-03-04-08
title: 'Эксплуатация — Workflow'
type: operation
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

# Эксплуатация — Workflow

## 1. Назначение документа

Документ фиксирует эксплуатационные факты текущего Workflow Runtime: подключение
в host, создание схемы хранения, диагностику и ограничения восстановления. Он
не является runbook для SQL Server, Audit History или Integration Events.

## 2. Запуск и готовность

`AddPlatformWorkflow` регистрирует `WorkflowDbContext`, state/history/revision
stores, execution transaction и `WorkflowRuntimeService`. В host composition
дополнительно регистрируются Configuration resolver, Tenant Security authorizer,
Object Runtime archive synchronizer, runtime projection и initialization event
handler.

При старте `WorkflowDatabaseInitializer` для relational provider вызывает EF
`MigrateAsync`, создаёт схему `workflow` и проверяет наличие таблиц
`workflow_states`, `workflow_history` и `workflow_definition_revisions`.
Для нереляционного тестового provider используется `EnsureCreatedAsync`.

## 3. Миграции

Текущая SQL Server initialization содержит совместимость со старой колонкой
`WorkflowVersion`: при наличии старой колонки и отсутствии
`WorkflowDefinitionRevision` выполняется переименование. Это миграционное
поведение текущего MVP, а не универсальная стратегия schema evolution.

При переносе на другую СУБД нужно отдельно сопоставить schema, JSON storage,
unique indexes, concurrency token и migration history. Такой mapping не
встраивается в описание текущей эксплуатации.

## 4. Диагностика

| Ситуация | Технический код или объект | Что проверять |
| --- | --- | --- |
| Definition Workflow не найдена | `WORKFLOW_DEFINITION_NOT_FOUND` | Публикацию Workflow и соответствие module/object/workflow codes |
| Revision definition не найдена | `WORKFLOW_DEFINITION_REVISION_NOT_FOUND` | Наличие pinned snapshot и состояние таблицы revisions |
| Команда недоступна в текущем состоянии | `WORKFLOW_COMMAND_NOT_AVAILABLE` | `CurrentStateCode`, `CommandCode` и definition переходов |
| Guards не были вычислены | `WORKFLOW_GUARDS_NOT_EVALUATED` | Подключение Rules gateway и guard adapter |
| Обнаружено несоответствие конкурентности | `WORKFLOW_CONCURRENCY_MISMATCH` | Конкурентное обновление и переданный token |
| Нужно проверить результат команды | `Workflow.CommandExecuted` | Запись результата и состояние outbox у Integration Events |
| Нужно проследить операцию | `CorrelationId` | Сквозной поиск в history, audit и event records |

## 5. Восстановление

Definition revision хранится как snapshot и должна сохраняться вместе с
состоянием экземпляра. Если snapshot, на который ссылается state, отсутствует,
execute отклоняется. При чтении `GetState` текущий код может восстановить
definition и пере-закрепить revision, если текущая definition совместима.
Политика восстановления для production требует отдельного решения.

## 6. Откат и эскалация

При ошибке сохранения состояния команда не считается успешно выполненной.
Откат публикации событий, повторная доставка и poison-message handling
определяются Integration Events. Проблемы permission эскалируются владельцу
Tenant Security, проблемы definition — Configuration, проблемы object archive
sync — Object Runtime, проблемы guard — Rules.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
