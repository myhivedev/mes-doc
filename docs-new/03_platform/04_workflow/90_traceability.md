---
id: DOC-03-04-90
title: 'Трассировка — Workflow'
type: traceability
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
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Трассировка — Workflow

## 1. Назначение документа

Документ показывает, как исходные материалы и факты реализации распределены по
пакету Workflow. Он не является вторым описанием архитектуры и не закрывает
открытые решения без решения владельца.

## 2. Источники и требования

| Источник | Что проверено | Маршрут |
| --- | --- | --- |
| `DMP.Platform.Workflow` | Controller, service, ports, persistence и DI | `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `08_operations.md` |
| Host composition `RuntimeWorkflowProjectionService` и initialization handler | Потребление Workflow service/state/revision ports и обработка события после создания объекта | `02_architecture.md`, `03_contracts.md`, `04_runtime.md` |
| `DMP.Platform.Contracts.Workflow` | Request/response DTO и error contract | `03_contracts.md` |
| `WorkflowDurableFoundationIntegrationTests` | State, history, revision, permissions, guards, reassign | `07_quality.md` |
| `WorkflowRuntimeContractSerializationTests` | Public JSON names и CommandCode boundary | `03_contracts.md`, `07_quality.md` |
| Configuration Workflow schema | Definition source и граница schema/runtime | `01_scope.md`, `02_architecture.md` |
| Старые Workflow artifact и UI materials | Ожидания и варианты, не подтверждённые автоматически | Рассмотрены; source-копии не входят в новый пакет |
| ADR-004, ADR-005, ADR-010, ADR-011 | Целевые правила и ограничения | Использованы только как решения или открытые темы после сверки с кодом |

## 3. Принятые решения

| Тема | Решение | Документ-владелец |
| --- | --- | --- |
| Разделение `CommandCode` и `ActionCode` | Команда запускает workflow routing; action не является workflow command | `03_contracts.md`, `04_runtime.md` |
| Закрепление definition | Экземпляр хранит `WorkflowDefinitionRevision`; latest publication не переключает его молча | `02_architecture.md`, `04_runtime.md` |
| Runtime boundary | Workflow владеет routing/state/history API, но не schema и object mutation | `01_scope.md`, `02_architecture.md` |
| Workflow initialization | После create используется Integration Events payload и runtime projection | `03_contracts.md`, `04_runtime.md` |

## 4. Расхождения и открытые решения

| ID / тема | Ожидание или источник | Текущее подтверждённое состояние | Расхождение или неопределённость | Влияние на текущую документацию | Статус сведения | Владелец / следующий шаг | Документ для обновления после решения |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `WF-DEC-01` — доступ к state и history | Workflow UI должен получать состояние и историю объекта | Service явно авторизует available/execute/init/reassign, но не вызывает authorizer в `GetState` и `GetHistory` | Не закреплена единая политика permission для маршрутов чтения | Не блокирует текущую документацию: отсутствие проверки и ограничение гарантии описаны; решение нужно для production-политики | Открытый вопрос | Tenant Security + Workflow определить политику чтения | `03_contracts.md`, `05_security_and_audit.md` |
| `WF-DEC-02` — внешняя версия event payload | События должны быть совместимы между потребителями | Publisher формирует `Workflow.CommandExecuted` и `Workflow.InstanceReassigned`; отдельная schema/version policy не задана | Не определены версия, обязательные поля и backward compatibility | Нет, текущие события описываются как ограниченный MVP publisher contract | Будущая доработка | Integration Events подготовить event standard | `03_contracts.md`, `07_quality.md` |
| `WF-DEC-03` — recovery pinned revision | Экземпляр не должен потерять поведение при публикации нового definition | `GetState` умеет recovery/re-pin, `ExecuteCommand` отклоняет отсутствующую revision | Различается поведение чтения и выполнения | Не блокирует текущую документацию: различие поведения и ограничение recovery описаны; решение нужно для production-политики восстановления | Открытый вопрос | Workflow + Operations определить production recovery policy | `04_runtime.md`, `08_operations.md` |
| `WF-DEC-04` — transaction boundary audit/outbox | State, history, audit и event должны быть согласованы | Execute обёрнут в локальную transaction, но распределённая atomicity не доказана | Не определена политика действий после фиксации транзакции и повторов между владельцами | Нет для текущего описания; гарантии ограничены явно | Будущая доработка | Audit History + Integration Events оформить policy | `04_runtime.md`, `05_security_and_audit.md`, `08_operations.md` |
| `WF-DEC-05` — Workflow steps and assignments | Старые материалы описывают steps, entry/exit hooks и assignment | Текущая `WorkflowDefinition` содержит states, commands, transitions, guard bindings и state policies; steps/assignments не исполняются | Ожидание исходных материалов шире MVP | Нет, будущие элементы не включены в нормативное описание | Будущая доработка | Workflow подготовить отдельный проект расширения | `01_scope.md`, `04_runtime.md` |
| `WF-DEC-06` — multiple workflow selector | Один object type может иметь несколько workflows | Host selector выбирает один workflow, отклоняет unknown/ambiguous code; explicit selector поддержан ограниченно | Нет общего публичного selector contract для всех callers | Не блокирует текущую документацию: текущий явный выбор и ограничение описаны; общий selector contract нужен только для будущих callers | Будущая доработка | Object Runtime + Workflow согласовать selector contract | `03_contracts.md`, `04_runtime.md` |
| `WF-DEC-07` — Java/PostgreSQL mapping | Целевой архитектурный этап использует Java/PostgreSQL | MVP реализован на .NET/EF Core/SQL Server | Mapping не разработан | Нет, не блокирует описание текущего MVP | Будущая доработка | Выполнить отдельный migration mapping на целевом этапе | `02_architecture.md`, `08_operations.md` |

## 5. Маршрут в целевые документы

| Source block | Результат миграции | Целевой документ |
| --- | --- | --- |
| Workflow artifact schema | Оставлена у владельца Configuration; в Workflow описано только runtime mapping | `../02_configuration/artifact_types/workflow.md`, `01_scope.md` |
| Workflow runtime controller/service | Перенесены маршруты и алгоритм выполнения | `03_contracts.md`, `04_runtime.md` |
| Workflow persistence and migrations | Перенесены архитектурно значимые таблицы и эксплуатационные ограничения | `02_architecture.md`, `08_operations.md` |
| Workflow Editor requirements | Не перенесены как backend guarantee; направлены Configuration/фронтенд-платформа | backlog и документы владельцев |
| WF Runtime UI requirements | Server DTO оставлены в Workflow, рендерер и layout принадлежат фронтенд-платформа | `03_contracts.md`, `01_scope.md` |
| ADR expectations not present in code | Не выданы за MVP; сохранены как открытые темы или будущие доработки | этот документ и backlog |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
