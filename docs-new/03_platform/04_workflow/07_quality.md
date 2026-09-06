---
id: DOC-03-04-07
title: 'Качество — Workflow'
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

# Качество — Workflow

## 1. Назначение документа

Документ показывает, какие свойства Workflow подтверждены тестами и где остаются
ограничения. Он не превращает наличие интеграционного теста в гарантию
промышленной производительности, доставки событий или полного покрытия прав.

## 2. Надёжность

| Свойство | Проверка | Результат |
| --- | --- | --- |
| Публичные имена `CommandCode`, revision и error fields | `WorkflowRuntimeContractSerializationTests` | Подтверждено |
| Инициализация, повторная инициализация и начальное состояние | `WorkflowDurableFoundationIntegrationTests` | Подтверждено, но ограничено текущим host |
| Выбор transition и обновление state | `WorkflowDurableFoundationIntegrationTests` | Подтверждено |
| Optimistic concurrency | `WorkflowDurableFoundationIntegrationTests` | Подтверждено |
| Закрепление definition revision | `WorkflowDurableFoundationIntegrationTests` | Подтверждено |
| Reassign workflow instance | `WorkflowDurableFoundationIntegrationTests` | Подтверждено, но ограничено |
| History и correlation id | `WorkflowDurableFoundationIntegrationTests` | Подтверждено |
| Permission allow/deny | `WorkflowDurableFoundationIntegrationTests` | Подтверждено для execute/available |
| Guard allowed/rejected/not evaluated | `WorkflowDurableFoundationIntegrationTests` | Подтверждено, но поддержка Rules ограничена |

## 3. Наблюдаемость

Workflow передаёт `CorrelationId` в history, audit и integration event flow.
Полный набор обязательных metrics, traces, logs и health checks не определён.
До утверждения общего platform operations contract это ограничение фиксируется
как пробел, а не как гарантия наблюдаемости.

## 4. Производительность

Текущие тесты подтверждают корректность сценариев, но не задают пределы
latency, throughput или размера definition. `GetHistory` ограничивает страницу
200 записями, однако внутренний snapshot может читать больше данных. Сценарии
массового исполнения команд и массовой проверки availability не закреплены.

## 5. Проверки и результаты

При подготовке пакета выполнена сверка публичных маршрутов, DTO, ссылок на
исходники,
таблиц Markdown и Mermaid-блоков. Тесты
`WorkflowDurableFoundationIntegrationTests` и
`WorkflowRuntimeContractSerializationTests` использованы как источники покрытия,
но в рамках документальной проверки не запускались. Полный набор
интеграционных тестов для подготовки документации не требуется.

## 6. Неподтверждённые свойства

- нет доказательства production SLO;
- не закреплена внешняя версия event payload;
- не доказана атомарность между Workflow, Audit History и Integration Events на
  распределённой границе;
- не завершена политика доступа к чтению state/history;
- не реализованы полноценные steps, assignments и универсальные bulk commands.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-26 23:30 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | предварително готовые модули ядра и связанные изменения | [ca13b19b](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/ca13b19bd17dd297927c1e66a97f95c29735b971) |
