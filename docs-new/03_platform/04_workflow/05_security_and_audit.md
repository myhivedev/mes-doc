---
id: DOC-03-04-05
title: 'Безопасность и аудит — Workflow'
type: assurance
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
updated_at: 2026-08-26 23:30
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Безопасность и аудит — Workflow

## 1. Назначение документа

Документ фиксирует фактические проверки доступа и записи аудита в Workflow
Runtime. Каталог ролей и permission assignments принадлежит Tenant Security,
а долговременное хранение и поиск audit records — Audit History.

## 2. Ресурсы и права

| Операция | Permission code | Текущая проверка |
| --- | --- | --- |
| Просмотр доступных команд | `<ModuleCode>.<ObjectTypeCode>.Workflow.View` | Обязательная проверка |
| Выполнение команды | `<ModuleCode>.<ObjectTypeCode>.Workflow.Execute` | Обязательная проверка |
| Инициализация экземпляра | `<ModuleCode>.<ObjectTypeCode>.Workflow.Execute` | Обязательная проверка |
| Переназначение workflow | `<ModuleCode>.<ObjectTypeCode>.Workflow.Execute` | Обязательная проверка |
| Получение state | Явный permission вызов в `WorkflowRuntimeService` не выполняется | Требует общей политики host |
| Получение истории | Явный вызов permission в `WorkflowRuntimeService` не выполняется | Требует общей политики host |

Host adapter `WorkflowPermissionAuthorizer` передаёт в Tenant Security user,
tenant, site, roles и permission code. Fallback authorizer при неполной
композиции выбрасывает ошибку конфигурации и не является разрешением доступа.

## 3. Принятие решения и управление

Frontend может скрыть недоступную команду, но это только отображение результата. Перед
`ExecuteCommand` право проверяется повторно на сервере. Guard и permission —
разные решения: permission разрешает использовать capability, guard проверяет
условие конкретного transition.

Маршруты чтения состояния и истории сейчас не вызывают
`IWorkflowPermissionAuthorizer`
внутри сервиса. До промышленного утверждения нужно определить общую политику
доступа к просмотру состояния и истории; это открытый вопрос, а не гарантия
публичной анонимной доступности.

## 4. Контроли и риски

| Контроль | Что предотвращает | Статус |
| --- | --- | --- |
| Нормализация identity | Пустые или неоднозначные коды | Подтверждено MVP |
| Проверка `Workflow.Execute` | Запуск команды без права | Подтверждено MVP |
| Проверка expected state/token | Потерю конкурентного изменения | Подтверждено MVP |
| Server-side guard | Обход условия перехода через UI | Подтверждено, но ограничено подключением Rules |
| Tenant filter в stores | Чтение состояния другого tenant | Подтверждено persistence-кодом |
| Повторная проверка при execute | Доверие клиентскому availability | Подтверждено MVP |
| Доступ к state/history | Защита диагностических данных | Политика не закреплена в Workflow service |

## 5. Аудит

Успешные `ExecuteCommand` и `ReassignInstance` передают в Audit History записи
с операцией, tenant/user, correlation id, object identity, workflow и state
codes, transition code, definition revision и result. Workflow не владеет
таблицей аудита и не определяет retention.

Инициализация экземпляра через обычный `InitializeInstance` не создаёт отдельную
audit запись в текущем сервисе. Обработка `Workflow.InstanceInitializationRequested`
также не создаёт отдельную запись в Workflow Runtime; host projection выполняет
идемпотентное создание состояния. Это ограничение нужно учитывать при
проектировании полного аудит-покрытия.

## 6. Покрытие и пробелы

- Подтверждено: permission adapter, server-side execute check, tenant context,
  correlation id и audit calls для execute/reassign.
- Ограничено: guard зависит от подключённого Rules gateway; `NotEvaluated`
  приводит к отказу.
- Открыто: явная политика для чтения состояния и истории, аудит и идемпотентность
  инициализации, а также общая политика действий после фиксации транзакции для
  audit/outbox.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-26 23:30 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | предварително готовые модули ядра и связанные изменения | [ca13b19b](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/ca13b19bd17dd297927c1e66a97f95c29735b971) |
