---
id: DOC-03-04-03
title: 'Контракты — Workflow'
type: contract
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

# Контракты — Workflow

## 1. Назначение и границы

Документ описывает HTTP-контракты Workflow Runtime, DTO ответов, межкомпонентные
порты, связанные integration events и коды ошибок. Он не заменяет schema
артефакта `Workflow`, внутреннее хранение или рендерер фронтенда.

Публичной командой является `CommandCode`. Клиент не передаёт `TransitionCode`
для выбора перехода и не запускает `ActionCode` вместо команды workflow.

## 2. Источники истины и владельцы

| Контрактный слой | Источник структуры | Владелец |
| --- | --- | --- |
| HTTP | `WorkflowRuntimeController` и DTO `DMP.Platform.Contracts.Workflow` | Workflow |
| Effective definition | `IWorkflowDefinitionResolver` и Configuration adapter | Configuration поставляет данные; Workflow интерпретирует их |
| Guard | `IWorkflowTransitionGuard`, `IRuleEvaluationGateway` | Workflow задаёт boundary; Rules вычисляет правило |
| Permission | `IWorkflowPermissionAuthorizer` | Tenant Security |
| Audit и event envelope | `IAuditHistoryWriter`, `IIntegrationEventPublisher` и общие contracts | Audit History / Integration Events |
| Workflow initialization | `WorkflowInstanceInitializationRequestedPayload` и host handler | Integration Events доставляет; Workflow/Object Runtime исполняют |

## 3. Карта контрактов

| Контракт | Вид | Владелец | Потребитель | Статус сведения | Подробное описание |
| --- | --- | --- | --- | --- | --- |
| `/api/workflow/instances/initialize` | HTTP | Workflow | Runtime Facade, host и приложения | Подтверждено, но ограничено | [4.1](#41-initializeinstance) |
| `/api/workflow/instances/reassign` | HTTP | Workflow | Административный или runtime caller | Подтверждено, но ограничено | [4.2](#42-reassigninstance) |
| `/api/workflow/state` | HTTP | Workflow | Runtime Facade и приложения | Подтверждено | [4.3](#43-getstate) |
| `/api/workflow/history` | HTTP | Workflow | Runtime UI и диагностика | Подтверждено, но ограничено | [4.4](#44-gethistory) |
| `/api/workflow/commands/available` | HTTP | Workflow | Runtime UI | Подтверждено, но ограничено | [4.5](#45-getavailablecommands) |
| `/api/workflow/commands/execute` | HTTP | Workflow | Runtime UI, host и приложения | Подтверждено | [4.6](#46-executecommand) |
| `WorkflowStateResponse` и связанные DTO | DTO | Workflow | HTTP и frontend-адаптеры | Подтверждено | [5.1](#51-workflowstateresponse) |
| `IWorkflowRuntimeService` | C#-контракт | Workflow | HTTP-контроллер и `RuntimeWorkflowProjectionService` | Подтверждено | [6.1](#61-iworkflowruntimeservice) |
| `IWorkflowDefinitionResolver` | C#-порт | Граница Workflow | Адаптер Configuration в host-композиции | Подтверждено | [6.2](#62-iworkflowdefinitionresolver) |
| `IWorkflowDefinitionRevisionStore` | C#-порт | Граница Workflow | Workflow Runtime и `RuntimeWorkflowProjectionService` | Подтверждено | [6.3](#63-iworkflowdefinitionrevisionstore) |
| `IWorkflowStateStore` | C#-порт | Граница Workflow | Workflow Runtime и `RuntimeWorkflowProjectionService` | Подтверждено, но ограничено | [6.7](#67-iworkflowstatestore) |
| `IWorkflowTransitionGuard` | C#-точка расширения | Workflow | Реализации guard | Подтверждено, но ограничено | [6.4](#64-iworkflowtransitionguard) |
| `Workflow.InstanceInitializationRequested` | Event payload | Workflow/Object Runtime | Integration Events processor | Подтверждено, но ограничено | [7.1](#71-workflowinstanceinitializationrequested) |
| `Workflow.CommandExecuted` | Event | Workflow | Integration consumers | Подтверждено, но доставка внешнему потребителю принадлежит Integration Events | [7.2](#72-workflowcommandexecuted) |
| `Workflow.InstanceReassigned` | Event | Workflow | Integration consumers | Подтверждено, но доставка внешнему потребителю принадлежит Integration Events | [7.3](#73-workflowinstancereassigned) |

## 4. HTTP-контракты

Все маршруты используют `POST` и принимают JSON-тело. `[ApiController]` отвечает
за привязку данных и базовую HTTP-обработку; результат Workflow возвращается внутри
соответствующего response DTO.

| Метод и маршрут | Запрос | Ответ | Права | Идемпотентность | Конкурентность |
| --- | --- | --- | --- | --- | --- |
| `POST /api/workflow/instances/initialize` | `InitializeWorkflowInstanceRequest` | `InitializeWorkflowInstanceResponse` | `Workflow.Execute` | Повтор безопасен: `AlreadyInitialized` | Проверка токена не предусмотрена |
| `POST /api/workflow/instances/reassign` | `ReassignWorkflowInstanceRequest` | `ReassignWorkflowInstanceResponse` | `Workflow.Execute` | Повтор отклоняется при существующем target | Отдельный expected token не предусмотрен |
| `POST /api/workflow/state` | `GetWorkflowStateRequest` | `WorkflowStateResponse` | В сервисе явная проверка отсутствует | Чтение | Не применяется |
| `POST /api/workflow/history` | `GetWorkflowHistoryRequest` | `GetWorkflowHistoryResponse` | В сервисе явная проверка отсутствует | Чтение | Не применяется |
| `POST /api/workflow/commands/available` | `GetAvailableWorkflowCommandsRequest` | `GetAvailableWorkflowCommandsResponse` | `Workflow.View`, затем проверка `Workflow.Execute` для availability | Чтение | Состояние фиксируется на момент вычисления |
| `POST /api/workflow/commands/execute` | `ExecuteWorkflowCommandRequest` | `ExecuteWorkflowCommandResponse` | `Workflow.Execute` | Повтор зависит от state и concurrency | `ExpectedStateCode`, `ExpectedConcurrencyToken` |

### 4.1. InitializeInstance

#### Назначение и владелец

Операция Workflow Runtime создаёт состояние экземпляра в начальном состоянии
выбранной definition.

#### Маршрут и права

**Маршрут:** `POST /api/workflow/instances/initialize`.

Повторный вызов для уже существующего состояния возвращает
`ResultCode = AlreadyInitialized`. Права выполнения проверяются до операции.

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode` | `string` | Нет | Обязательно | Непустой код после обрезки внешних пробелов | Код владельца объекта | Используется для разрешения definition и permission | `InitializeWorkflowInstanceRequest` |
| `ObjectTypeCode` | `string` | Нет | Обязательно | Непустой код после обрезки внешних пробелов | Код типа объекта | Должен соответствовать опубликованной definition | `InitializeWorkflowInstanceRequest` |
| `ObjectId` | `string` | Нет | Обязательно | Непустая строка | Идентификатор объекта | Состояние хранится отдельно от данных объекта | `InitializeWorkflowInstanceRequest` |
| `WorkflowCode` | `string` | Нет | Обязательно | Непустой код | Код workflow | Повторная инициализация использует тот же код | `InitializeWorkflowInstanceRequest` |
| `InitialStateCode` | `string` | Да | Из definition | Код состояния из выбранной definition | Явное начальное состояние | Если код не найден, возвращается `WORKFLOW_INITIAL_STATE_NOT_FOUND` | `InitializeWorkflowInstanceRequest` |
| `LanguageCode` | `string` | Да | Язык платформы | Код языка, поддержанный host | Язык заголовка состояния | Используется resolver языка | `InitializeWorkflowInstanceRequest` |
| `Context` | `IReadOnlyDictionary<string, object?>` | Да | `null` | Произвольный контекст | Контекст выбора и проверки | В текущей инициализации не передаётся в guard | `InitializeWorkflowInstanceRequest` |

#### Ответ и ошибки

Ответ — `InitializeWorkflowInstanceResponse`: `WorkflowCode`, `ResultCode`,
`State` и необязательная [ошибка](#8-ошибки-и-отказоустойчивость). При отсутствии
definition или начального state операция отклоняется.

#### Ограничения и совместимость

Операция не принимает expected state или concurrency token. Повторный вызов не
создаёт вторую запись состояния.

### 4.2. ReassignInstance

#### Назначение и владелец

Workflow создаёт состояние целевого workflow для уже инициализированного
объекта; исходное состояние сохраняется.

#### Маршрут и права

**Маршрут:** `POST /api/workflow/instances/reassign`.

Source и target workflow должны различаться,
а target не должен быть уже инициализирован.

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId` | `string` | Нет | Обязательно | Непустые значения после обрезки внешних пробелов | Идентичность объекта | Для source и target используется одна идентичность | `ReassignWorkflowInstanceRequest` |
| `SourceWorkflowCode` | `string` | Нет | Обязательно | Непустой код | Исходный workflow | Source state должен существовать | `ReassignWorkflowInstanceRequest` |
| `TargetWorkflowCode` | `string` | Нет | Обязательно | Непустой код, отличный от source | Новый workflow | Target state не должен существовать | `ReassignWorkflowInstanceRequest` |
| `InitialStateCode` | `string` | Да | Из target definition | Код состояния target definition | Начальное состояние target | Неизвестный код отклоняется | `ReassignWorkflowInstanceRequest` |
| `LanguageCode` | `string` | Да | Язык платформы | Код языка, поддержанный host | Язык definition | Передаётся resolver-у | `ReassignWorkflowInstanceRequest` |
| `Reason` | `string` | Да | `null` | Произвольная строка | Причина переназначения | Сохраняется в details history/audit/event | `ReassignWorkflowInstanceRequest` |

#### Ответ и ошибки

Ответ — `ReassignWorkflowInstanceResponse` с source/target code, result, target
state и ошибкой при отказе.

#### Ограничения и совместимость

Исходное состояние сохраняется. Операция не принимает expected state или
concurrency token для исходного экземпляра.

### 4.3. GetState

#### Назначение и владелец

Операция возвращает текущее состояние экземпляра Workflow для объекта.

#### Маршрут и права

**Маршрут:** `POST /api/workflow/state`.

Возвращает `WorkflowStateResponse` для объекта. Если состояние отсутствует,
поля текущего state, revision, concurrency token, времени и пользователя могут
быть `null`. Заголовок состояния разрешается из закреплённой definition, если
она найдена.

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId`, `WorkflowCode` | `string` | Нет | Обязательно | Непустые значения после обрезки внешних пробелов | Идентичность экземпляра | Определяет tenant-scoped state | `GetWorkflowStateRequest` |
| `LanguageCode` | `string` | Да | Язык платформы | Код языка, поддержанный host | Язык заголовка state | Влияет только на разрешение title | `GetWorkflowStateRequest` |

#### Ответ, ошибки и ограничения

Ответом является `WorkflowStateResponse`. При отсутствии состояния его поля
состояния остаются `null`; отдельный HTTP-код для отсутствия состояния текущим
контрактом не закреплён.

### 4.4. GetHistory

#### Назначение и владелец

Операция возвращает историю выполнения команд, включая зафиксированные отказы,
и успешных переназначений для
выбранного экземпляра Workflow.

#### Маршрут и права

**Маршрут:** `POST /api/workflow/history`.

Возвращает историю по object/workflow identity. `Skip` по умолчанию равен `0`;
`Take` по умолчанию равен `50` и ограничен значением `200`. Сортировка идёт по
`OccurredAtUtc` и затем по идентификатору в обратном порядке.

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId`, `WorkflowCode` | `string` | Нет | Обязательно | Непустые значения после обрезки внешних пробелов | Фильтр истории | История ограничена текущим tenant | `GetWorkflowHistoryRequest` |
| `Skip` | `int` | Да | `0` | Неотрицательное значение после нормализации | Число пропускаемых записей | Отрицательное значение заменяется на `0` | `GetWorkflowHistoryRequest` |
| `Take` | `int` | Да | `50` | От `1` до `200` после ограничения | Размер страницы | Максимум `200` | `GetWorkflowHistoryRequest` |

#### Ответ, ошибки и ограничения

Ответ — `GetWorkflowHistoryResponse` с `Items` и `TotalCount`. Текущая реализация
читает через хранилище снимка истории; отдельный оптимизированный контракт запроса
не закреплён.

### 4.5. GetAvailableCommands

#### Назначение и владелец

Операция возвращает команды, объявленные в definition, и их доступность для
текущего состояния и пользователя.

#### Маршрут и права

**Маршрут:** `POST /api/workflow/commands/available`.

Возвращает все команды definition, сопоставленные с transition для текущего
state. Для каждой команды учитываются право выполнения и результат guard.

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId`, `WorkflowCode` | `string` | Нет | Обязательно | Непустые значения после обрезки внешних пробелов | Идентичность экземпляра | Определяет state и definition | `GetAvailableWorkflowCommandsRequest` |
| `LanguageCode` | `string` | Да | Язык платформы | Код языка, поддержанный host | Локализация title | Передаётся resolver-у | `GetAvailableWorkflowCommandsRequest` |
| `ViewCode` | `string` | Да | `null` | Произвольный код представления | Контекст представления | В текущем сервисе не используется для выбора transition | `GetAvailableWorkflowCommandsRequest` |
| `Context` | `IReadOnlyDictionary<string, object?>` | Да | `null` | Произвольный контекст | Контекст guard | Передаётся в оценку guard | `GetAvailableWorkflowCommandsRequest` |

#### Ответ, ошибки и ограничения

Ответ — `GetAvailableWorkflowCommandsResponse` с текущим state, concurrency
token и коллекцией `WorkflowCommandAvailabilityResponse`. Команда может быть
возвращена как недоступная с `DisabledReasonCode`.

### 4.6. ExecuteCommand

#### Назначение и владелец

Операция запускает серверную маршрутизацию по `CommandCode` и, если все проверки
пройдены, переводит экземпляр в целевое состояние.

#### Маршрут и права

**Маршрут:** `POST /api/workflow/commands/execute`.

Запускает backend-маршрутизацию по `CommandCode`. Сервер выбирает transition,
проверяет guard и concurrency, сохраняет состояние, пишет history/audit и
публикует integration event.

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId`, `WorkflowCode` | `string` | Нет | Обязательно | Непустые значения после обрезки внешних пробелов | Идентичность экземпляра | Должна совпадать с pinned state и definition | `ExecuteWorkflowCommandRequest` |
| `CommandCode` | `string` | Нет | Обязательно | Непустой код команды из definition | Публичная команда workflow | Клиент не передаёт `TransitionCode` | `ExecuteWorkflowCommandRequest` |
| `ExpectedStateCode` | `string` | Да | `null` | Код ожидаемого состояния | Защита от устаревшего состояния | Проверяется при передаче | `ExecuteWorkflowCommandRequest` |
| `ExpectedConcurrencyToken` | `string` | Да | `null` | Текущее строковое представление token | Защита от конкурентного изменения | При несовпадении команда отклоняется | `ExecuteWorkflowCommandRequest` |
| `Parameters` | `IReadOnlyDictionary<string, object?>` | Да | `null` | Произвольный объект параметров | Зарезервированные параметры команды | В текущей реализации не валидируются и не исполняются | `ExecuteWorkflowCommandRequest` |
| `Context` | `IReadOnlyDictionary<string, object?>` | Да | `null` | Произвольный контекст | Контекст guard | Передаётся в оценку guard | `ExecuteWorkflowCommandRequest` |

#### Ответ, ошибки и ограничения

Ответ — `ExecuteWorkflowCommandResponse` с from/to state, новым concurrency
token, result code и необязательным `Result`. В текущей реализации `Result`
может содержать выбранный `transitionCode` и `workflowDefinitionRevision`.

Минимальный пример показывает связь публичной команды с экземпляром и
результатом перехода. Коды в примере иллюстративны и не задают обязательную
definition для всех модулей:

```json
{
  "ModuleCode": "TestModule",
  "ObjectTypeCode": "TestObject",
  "ObjectId": "42",
  "WorkflowCode": "Approval",
  "CommandCode": "Submit"
}
```

```json
{
  "WorkflowCode": "Approval",
  "CommandCode": "Submit",
  "ResultCode": "Succeeded",
  "FromStateCode": "Draft",
  "ToStateCode": "Submitted",
  "ConcurrencyToken": "7"
}
```

## 5. Общие типы и DTO

### 5.1. `WorkflowStateResponse`

#### Назначение

DTO текущего состояния экземпляра, которое используется ответами `GetState`,
`InitializeInstance` и `ReassignInstance`, а также host/Object Runtime
projection.

#### Структура

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId`, `WorkflowCode` | `string` | Нет | Обязательно | Непустые коды | Идентичность экземпляра | Tenant передаётся отдельно в контексте | `WorkflowStateResponse` |
| `WorkflowDefinitionRevision` | `string` | Да | `null` без состояния | Строка revision, обычно `sha256:` | Закреплённая ревизия definition | Может быть `null`, если состояние не создано | `WorkflowStateResponse` |
| `CurrentStateCode` | `string` | Да | `null` без состояния | Код состояния definition | Текущее состояние | Не является полем бизнес-объекта | `WorkflowStateResponse` |
| `CurrentStateTitle` | `string` | Да | `null`, если title не разрешён | Локализованный текст | Отображаемый заголовок состояния | Производится из definition и языка | `WorkflowStateResponse` |
| `ConcurrencyToken` | `string` | Да | `null` без состояния | Строковое представление token | Защита конкурентного изменения | Передаётся обратно в execute при необходимости | `WorkflowStateResponse` |
| `UpdatedAtUtc` | `DateTime?` | Да | `null` без состояния | UTC timestamp | Время последнего изменения | Не является временем публикации definition | `WorkflowStateResponse` |
| `UpdatedBy` | `Guid?` | Да | `null` без состояния | Идентификатор пользователя | Кто изменил состояние | Может отсутствовать для системного вызова | `WorkflowStateResponse` |

#### Ограничения и совместимость

DTO отражает runtime state и не описывает schema артефакта `Workflow` или
хранение `workflow_states`.

### 5.2. `WorkflowHistoryItemResponse`

#### Назначение и структура

DTO одной записи истории Workflow. Это контракт чтения, а не копия сущности
`workflow_history`.

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `Id` | `Guid` | Нет | Обязательно | Уникальный идентификатор | Идентификатор записи | Генерируется при записи | `WorkflowHistoryItemResponse` |
| `WorkflowCode`, `ObjectTypeCode`, `ObjectId`, `CommandCode` | `string` | Нет | Обязательно | Непустые коды | Идентичность и команда | Команда может быть `ReassignWorkflow` | `WorkflowHistoryItemResponse` |
| `FromStateCode`, `ToStateCode` | `string` | Да | `null` допустим | Коды состояний | Исходное и целевое состояние | Для отказа `ToStateCode` может быть `null` | `WorkflowHistoryItemResponse` |
| `ResultCode` | `string` | Нет | Обязательно | Например `Succeeded`, `Rejected` | Результат операции | Список кодов не вынесен в отдельный enum | `WorkflowHistoryItemResponse` |
| `OccurredAtUtc` | `DateTime` | Нет | Обязательно | UTC timestamp | Время события истории | Используется для сортировки | `WorkflowHistoryItemResponse` |
| `UserId` | `Guid?` | Да | `null` допустим | Идентификатор пользователя | Инициатор операции | Может отсутствовать для системного вызова | `WorkflowHistoryItemResponse` |
| `CorrelationId` | `string` | Нет | Обязательно | Непустой идентификатор корреляции | Сквозная связь операции | Нужен для диагностики | `WorkflowHistoryItemResponse` |

### 5.3. `WorkflowCommandAvailabilityResponse`

#### Назначение и структура

Результат вычисления доступности команды для текущего состояния. DTO помогает
клиенту построить интерфейс, но не является источником права.

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `CommandCode`, `Title` | `string` | Нет | Обязательно | Код и текст из definition | Команда и её заголовок | `Title` зависит от языка definition | `WorkflowCommandAvailabilityResponse` |
| `IsAvailable`, `IsPrimary` | `bool` | Нет | `false` | `true` или `false` | Доступность и подсказка основной команды | `IsPrimary` вычисляется только среди доступных | `WorkflowCommandAvailabilityResponse` |
| `DisabledReasonCode`, `DisabledReason` | `string` | Да | `null`, если доступна | Код и текст причины | Объяснение недоступности | Не заменяет ошибку execute | `WorkflowCommandAvailabilityResponse` |
| `Order` | `int?` | Да | `null` | Число порядка | Порядок отображения | При отсутствии используется порядок definition | `WorkflowCommandAvailabilityResponse` |
| `RequiresConfirmation`, `RequiresPayload` | `bool` | Нет | `false` | `true` или `false` | Требования клиентского сценария | Сервер не выполняет payload-схему в этом DTO | `WorkflowCommandAvailabilityResponse` |
| `PayloadSchemaCode` | `string` | Да | `null` | Код схемы | Ссылка на схему параметров | Разрешение и проверка схемы принадлежат другому контракту | `WorkflowCommandAvailabilityResponse` |
| `Metadata` | `IReadOnlyDictionary<string, object?>` | Да | `null` | Объект metadata | Дополнительные сведения runtime | Текущие ключи не являются стабильной внешней схемой | `WorkflowCommandAvailabilityResponse` |

### 5.4. `WorkflowErrorResponse` и `WorkflowErrorDetailResponse`

#### Назначение и структура

DTO ошибки операции. Envelope ошибки и соответствие HTTP-статусу определяются
транспортным слоем host-приложения.

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `Code`, `Message`, `CorrelationId` | `string` | Нет | Обязательно | Непустые строки | Код, текст и корреляция ошибки | Коды Workflow перечислены в разделе 8 | `WorkflowErrorResponse` |
| `Field` | `string` | Да | `null` | Имя поля | Поле, вызвавшее отказ | Не все ошибки связаны с полем | `WorkflowErrorResponse` |
| `Details` | `IReadOnlyCollection<WorkflowErrorDetailResponse>` | Да | `null` | Коллекция деталей | Дополнительные ошибки | В текущих сценариях может отсутствовать | `WorkflowErrorResponse` |
| `Details[].Code`, `Details[].Message` | `string` | Нет | Обязательно | Непустые строки | Код и текст детали | Структура детали стабильна в текущем DTO | `WorkflowErrorDetailResponse` |
| `Details[].Field`, `Details[].Path` | `string` | Да | `null` | Имя поля и путь | Локализация детали в запросе | Не являются обязательными | `WorkflowErrorDetailResponse` |

## 6. C#-контракты и точки расширения

### 6.1. `IWorkflowRuntimeService`

Публичная граница сервиса Workflow Runtime для HTTP-контроллера и
`RuntimeWorkflowProjectionService`. Потребитель зависит от этого интерфейса, а
не от внутреннего класса `WorkflowRuntimeService`.

| Метод | Параметры | Результат | Предусловия | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- |
| `InitializeInstanceAsync` | `InitializeWorkflowInstanceRequest` | `InitializeWorkflowInstanceResponse` | Идентичность и definition должны быть разрешимы | Создание состояния | Не задана отдельным портом |
| `ReassignInstanceAsync` | `ReassignWorkflowInstanceRequest` | `ReassignWorkflowInstanceResponse` | Source существует, target отсутствует | Создание target state, history, audit и event | Не обёрнута в `IWorkflowExecutionTransaction` |
| `GetStateAsync` | `GetWorkflowStateRequest` | `WorkflowStateResponse` | Идентичность обязательна | Возможное восстановление pinned revision | Чтение; recovery сохраняет state |
| `GetHistoryAsync` | `GetWorkflowHistoryRequest` | `GetWorkflowHistoryResponse` | Идентичность обязательна | Нет | Чтение |
| `GetAvailableCommandsAsync` | `GetAvailableWorkflowCommandsRequest` | `GetAvailableWorkflowCommandsResponse` | Должна быть разрешима definition | Оценка guard | Чтение |
| `ExecuteCommandAsync` | `ExecuteWorkflowCommandRequest` | `ExecuteWorkflowCommandResponse` | Permission, definition и текущий state | State, history, audit и event при успехе; history при отказе | `IWorkflowExecutionTransaction` |

### 6.2. `IWorkflowDefinitionResolver`

Межкомпонентный порт host-композиции. Адаптер Configuration возвращает
`WorkflowDefinition` по `moduleCode`, `objectTypeCode`, `workflowCode` и языку
или `null`, если effective definition не найдена. Resolver не читает schema
таблицы напрямую.

| Метод | Параметры | Результат | Предусловия | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- |
| `ResolveAsync` | Коды объекта, `languageCode`, cancellation token | `WorkflowDefinition?` | Коды переданы вызывающим компонентом | Не определены Workflow Runtime | Нет требования |

### 6.3. `IWorkflowDefinitionRevisionStore`

Порт хранения неизменяемых снимков runtime definition. Workflow использует его
для закрепления поведения экземпляра; `RuntimeWorkflowProjectionService` также
использует его при подготовке runtime-проекции и инициализации.

| Метод | Параметры | Результат | Предусловия | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- |
| `GetOrCreateAsync` | `WorkflowDefinition` | `WorkflowDefinition` с `sha256:` revision | Definition не `null` | Чтение или создание snapshot | Зависит от persistence provider |
| `GetAsync` | Коды объекта и `workflowRevision` | `WorkflowDefinition?` | Revision должна иметь формат `sha256:` | Нет | Чтение |

`WorkflowDefinition` включает identity, `WorkflowDefinitionRevision`,
`InitialStateCode`, states, commands, transitions, guard bindings и state
policies. Состав runtime-модели не повторяет schema артефакта `Workflow`.

### 6.4. `IWorkflowTransitionGuard`

Точка расширения для проверки конкретного перехода. В
`WorkflowTransitionGuardRequest` передаются tenant/user, object identity,
revision, transition, command, исходное и целевое состояние и `Context`.

| Метод или member | Параметры | Результат | Предусловия | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- |
| `AppliesTo` | `WorkflowTransitionGuardRequest` | `bool` | Запрос сформирован для выбранного transition | Нет | Нет требования |
| `EvaluateAsync` | `WorkflowTransitionGuardRequest` | `WorkflowTransitionGuardResult` | Guard применим | Может обратиться к Rules gateway | Вызывается внутри сценария команды |

`Allowed` означает, что проверка не отклонила переход. `Reject` содержит
`WorkflowTransitionGuardIssue`. Guard не выдаёт право на HTTP-доступ и не
заменяет `IWorkflowPermissionAuthorizer`; отсутствие оценки приводит к отказу
команды.

### 6.5. `IWorkflowPermissionAuthorizer`

Порт Tenant Security для проверки разрешений Workflow. Workflow формирует коды
`<ModuleCode>.<ObjectTypeCode>.Workflow.View` и
`<ModuleCode>.<ObjectTypeCode>.Workflow.Execute`.

| Метод | Параметры | Результат | Предусловия | Исключения или отказ | Транзакция |
| --- | --- | --- | --- | --- | --- |
| `AuthorizeAsync` | `permissionCode` | `Task` | Код разрешения не пустой | Отказ передаётся как ошибка доступа | Нет требования |
| `TryAuthorizeAsync` | `permissionCode` | `Task<bool>` | Код разрешения не пустой | Возвращает `false` при отсутствии права | Нет требования |

### 6.6. `IWorkflowArchiveStateSynchronizer`

Порт взаимодействия с Object Runtime. Он вызывается только при изменении
признака archive state и передаёт `WorkflowArchiveStateChange` с object
identity, transition и исходным/целевым archive-признаком. Workflow не меняет
поля объекта напрямую.

| Метод | Параметры | Результат | Предусловия | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- |
| `SynchronizeAsync` | `WorkflowArchiveStateChange` | `Task` | Изменение archive state уже определено | Object Runtime может изменить archive-представление объекта | Распределённая атомарность не гарантирована |

### 6.7. `IWorkflowStateStore`

Порт состояния экземпляров Workflow. Он используется самим Workflow Runtime и
host/Object Runtime для чтения состояний списков и объектов, а также для
идемпотентной инициализации после создания объекта.

| Метод | Параметры | Результат | Предусловия | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- |
| `GetAsync` | tenant, object identity, workflow | `WorkflowStateRecord?` | Идентичность обязательна | Нет | Чтение |
| `GetManyAsync` | tenant, object type, object ids, workflow | Коллекция состояний | Список идентификаторов может быть пустым | Нет | Чтение |
| `GetObjectIdsByStatesAsync` | tenant, object type, workflow, state codes | Идентификаторы объектов | Список состояний может быть пустым | Нет | Чтение |
| `UpsertAsync` | `WorkflowStateRecord` | `Task` | Состояние валидно | Создание или обновление | Persistence provider |
| `UpdateAsync` | Состояние и optional expected token | `WorkflowStateRecord?` | Token проверяется, если передан | Изменение состояния и token | Persistence provider |

### 6.8. `IAuditHistoryWriter`, `IIntegrationEventPublisher` и `IRuleEvaluationGateway`

Workflow вызывает эти порты, но не владеет их общими envelope, хранилищем,
доставкой или языком правил. Их собственная структура описывается документами
Audit History, Integration Events и Rules. В Workflow фиксируются только условия
вызова и последствия: успешные execute/reassign передают audit и event, а guard
использует `IRuleEvaluationGateway` для вычисления привязанных правил.

`IWorkflowHistoryWriter` и EF entities остаются внутренними деталями Workflow:
текущий host/Object Runtime не использует их напрямую. `IRuntimeWorkflowProjectionService`
принадлежит Object Runtime/host и вызывает Workflow, а не наоборот.

## 7. Контракты событий

Envelope событий содержит tenant/correlation/time и принадлежит Integration
Events. Ниже описываются только полезная нагрузка и условие публикации Workflow.

### 7.1. `Workflow.InstanceInitializationRequested`

После успешного создания объекта Runtime публикует событие через общий outbox и
жизненный цикл объекта. Полезная нагрузка
`WorkflowInstanceInitializationRequestedPayload` содержит `ModuleCode`,
`ObjectTypeCode`, `ObjectId` и необязательный `WorkflowCode`. Host handler
передаёт данные в runtime projection для идемпотентной инициализации.

| Свойство | Значение |
| --- | --- |
| Техническое имя | `Workflow.InstanceInitializationRequested` |
| Отправитель | Object Runtime после успешного создания объекта |
| Получатель | `WorkflowInstanceInitializationRequestedEventHandler` в host-композиции |
| Условие публикации | Создание объекта завершилось успешно и получен непустой идентификатор |
| Доставка | Envelope и outbox принадлежат Integration Events; повторная доставка возможна |
| Идемпотентность | Handler проверяет наличие состояния перед созданием |
| Версия | Отдельная версия payload не закреплена |

| Поле payload | Тип | Nullable | Обязательность/default | Смысл | Источник |
| --- | --- | --- | --- | --- | --- |
| `ModuleCode` | `string` | Нет | Обязательно | Код владельца объекта | `WorkflowInstanceInitializationRequestedPayload` |
| `ObjectTypeCode` | `string` | Нет | Обязательно | Код типа объекта | `WorkflowInstanceInitializationRequestedPayload` |
| `ObjectId` | `string` | Нет | Обязательно | Идентификатор созданного объекта | `WorkflowInstanceInitializationRequestedPayload` |
| `WorkflowCode` | `string` | Да | `null` | Явно выбранный workflow | `WorkflowInstanceInitializationRequestedPayload` |

### 7.2. `Workflow.CommandExecuted`

После успешного выполнения команды Workflow передаёт publisher-у envelope
`Workflow.CommandExecuted`. Текущий anonymous payload включает module/object
identity, workflow code, command, from/to state, transition code, definition
revision и result; tenant id и correlation id находятся в envelope, а user id в
этом payload не передаётся. Точное содержание payload не следует считать
стабильной внешней схемой до отдельного решения по контракту события.

| Свойство | Значение |
| --- | --- |
| Техническое имя | `Workflow.CommandExecuted` |
| Отправитель | Workflow Runtime после успешного `ExecuteCommand` |
| Получатели | Зарегистрированные потребители Integration Events |
| Условие публикации | Состояние обновлено и записи history/audit завершены без ошибки |
| Доставка | Publisher получает envelope; транспорт, outbox и повторы принадлежат Integration Events |
| Идемпотентность | Отдельный ключ повторной обработки не закреплён |
| Версия | Версия payload не закреплена; текущая форма является ограниченным MVP-контрактом |

| Поле payload | Тип | Nullable | Обязательность/default | Смысл | Источник |
| --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId`, `WorkflowCode` | `string` | Нет | Обязательно | Идентичность объекта и workflow | anonymous payload в `WorkflowRuntimeService` |
| `WorkflowDefinitionRevision` | `string` | Нет | Обязательно при успехе | Закреплённая ревизия definition | anonymous payload в `WorkflowRuntimeService` |
| `CommandCode`, `TransitionCode`, `FromStateCode`, `ToStateCode` | `string` | `FromStateCode` не `null` при исполнении | Обязательно при успехе | Выполненная команда и переход | anonymous payload в `WorkflowRuntimeService` |
| `ResultCode` | `string` | Нет | `Succeeded` при публикации | Результат операции | anonymous payload в `WorkflowRuntimeService` |

### 7.3. `Workflow.InstanceReassigned`

После успешного переназначения Workflow передаёт publisher-у envelope
`Workflow.InstanceReassigned`. Текущий anonymous payload включает module/object
identity, исходный и целевой workflow, исходную и целевую ревизии definition,
начальное состояние, reason и result. Внешняя доставка, повторы и политика
идемпотентности принадлежат Integration Events.

| Свойство | Значение |
| --- | --- |
| Техническое имя | `Workflow.InstanceReassigned` |
| Отправитель | Workflow Runtime после успешного `ReassignInstance` |
| Получатели | Зарегистрированные потребители Integration Events |
| Условие публикации | Target state создан, history и audit записаны без ошибки |
| Доставка | Publisher получает envelope; транспорт и повторы принадлежат Integration Events |
| Идемпотентность | Отдельный ключ повторной обработки не закреплён |
| Версия | Версия payload не закреплена; текущая форма является ограниченным MVP-контрактом |

| Поле payload | Тип | Nullable | Обязательность/default | Смысл | Источник |
| --- | --- | --- | --- | --- | --- |
| `ModuleCode`, `ObjectTypeCode`, `ObjectId` | `string` | Нет | Обязательно | Идентичность объекта | anonymous payload в `WorkflowRuntimeService` |
| `SourceWorkflowCode`, `TargetWorkflowCode` | `string` | Нет | Обязательно | Исходный и целевой workflow | anonymous payload в `WorkflowRuntimeService` |
| `SourceWorkflowDefinitionRevision`, `TargetWorkflowDefinitionRevision` | `string` | Нет | Обязательно при успехе | Ревизии исходного и целевого definition | anonymous payload в `WorkflowRuntimeService` |
| `InitialStateCode`, `Reason` | `string` | `Reason` может быть `null` | `InitialStateCode` обязателен при успехе | Начальное состояние и причина | anonymous payload в `WorkflowRuntimeService` |
| `ResultCode` | `string` | Нет | `Succeeded` при публикации | Результат операции | anonymous payload в `WorkflowRuntimeService` |

## 8. Ошибки и отказоустойчивость

| Код или тип ошибки | Условие | HTTP или transport result | Поле/path | Повторить запрос | Ответственный |
| --- | --- | --- | --- | --- | --- |
| `WORKFLOW_DEFINITION_NOT_FOUND` | Definition отсутствует | `ResultCode = Rejected`; HTTP mapping не закреплён | `TargetWorkflowCode` для reassign | После публикации или исправления definition | Configuration и Workflow |
| `WORKFLOW_DEFINITION_REVISION_NOT_FOUND` | Не найден закреплённый snapshot | `ResultCode = Rejected`; HTTP mapping не закреплён | `WorkflowDefinitionRevision` | После восстановления snapshot | Workflow Operations |
| `WORKFLOW_INITIAL_STATE_NOT_RESOLVED` | Начальное state не определено | `ResultCode = Rejected`; HTTP mapping не закреплён | `InitialStateCode` | После исправления definition или запроса | Workflow |
| `WORKFLOW_INITIAL_STATE_NOT_FOUND` | Запрошенное state отсутствует в definition | `ResultCode = Rejected`; HTTP mapping не закреплён | `InitialStateCode` | После исправления кода состояния | Вызывающий компонент и Workflow |
| `WORKFLOW_INSTANCE_NOT_INITIALIZED` | Нельзя получить валидное состояние при execute | `ResultCode = Rejected`; HTTP mapping не закреплён | `WorkflowCode` | После явной инициализации | Вызывающий компонент |
| `WORKFLOW_CURRENT_STATE_NOT_RESOLVED` | Текущее состояние неизвестно | `ResultCode = Rejected`; HTTP mapping не закреплён | Состояние экземпляра | После восстановления состояния | Workflow Operations |
| `WORKFLOW_COMMAND_NOT_AVAILABLE` | Нет transition для состояния и команды | `ResultCode = Rejected`; HTTP mapping не закреплён | `CommandCode` | После обновления состояния или команды | Вызывающий компонент |
| `WORKFLOW_STATE_MISMATCH` | `ExpectedStateCode` не совпал | `ResultCode = Rejected`; HTTP mapping не закреплён | `ExpectedStateCode` | После перечитывания состояния | Вызывающий компонент |
| `WORKFLOW_CONCURRENCY_MISMATCH` | Маркер конкурентности не совпал | `ResultCode = Rejected`; HTTP mapping не закреплён | `ExpectedConcurrencyToken` | После перечитывания и повторной проверки | Вызывающий компонент |
| `WORKFLOW_GUARDS_NOT_EVALUATED` | Guard gateway не подключён или не оценил условие | `ResultCode = Rejected`; HTTP mapping не закреплён | `CommandCode` | Только после восстановления Rules gateway | Rules и Workflow |
| `WORKFLOW_GUARD_REJECTED` | Guard вернул отказ | `ResultCode = Rejected`; HTTP mapping не закреплён | `CommandCode` или поле из guard issue | После изменения входных данных, если разрешено правилом | Rules |
| `WORKFLOW_REASSIGN_TARGET_EQUALS_SOURCE` | Source и target совпадают | `ResultCode = Rejected`; HTTP mapping не закреплён | `TargetWorkflowCode` | Нет, нужно изменить запрос | Вызывающий компонент |
| `WORKFLOW_REASSIGN_SOURCE_NOT_INITIALIZED` | Source instance отсутствует | `ResultCode = Rejected`; HTTP mapping не закреплён | `SourceWorkflowCode` | После инициализации source | Вызывающий компонент |
| `WORKFLOW_REASSIGN_TARGET_ALREADY_INITIALIZED` | Target instance уже существует | `ResultCode = Rejected`; HTTP mapping не закреплён | `TargetWorkflowCode` | Нет без отдельной политики повторного назначения | Вызывающий компонент и Workflow |

Точное соответствие кодов HTTP-статусам не закреплено отдельным транспортным
стандартом. Нельзя предполагать, что любая ошибка предметной области
автоматически получает одинаковый HTTP-статус во всех host-приложениях.

## 9. Совместимость и изменение контрактов

Стабильными идентификаторами являются `WorkflowCode`, `CommandCode`, state codes,
transition codes внутри definition revision и формат `sha256:` для revision.
Переименование command или изменение смысла state требует анализа потребителей.

Изменение `WorkflowDefinition` создаёт новую revision. Существующий экземпляр
продолжает использовать свою revision, если не выполняется явная миграция.
Добавление необязательного response-поля совместимо при сохранении прежнего
смысла; изменение обязательности, значения кода ошибки, типа поля или смысла
state является изменением контракта.

События `Workflow.CommandExecuted` и `Workflow.InstanceReassigned` пока имеют
реализованную полезную нагрузку publisher-а, но не утверждённую отдельную версию внешней
схемы. Это ограничение текущего MVP, а не обещание полной совместимости событий.

| Изменение | Совместимо назад | Потребители | Миграция | Версия или решение |
| --- | --- | --- | --- | --- |
| Добавление необязательного поля в response DTO | Да, если сохраняется прежний смысл | HTTP и frontend-адаптеры | Не требуется | Версионирование не закреплено |
| Изменение обязательности, типа или смысла поля | Нет | Зависит от DTO или операции | Согласовать переход и обновить потребителей | Решение владельца Workflow |
| Переименование `CommandCode`, state или transition code | Нет | Runtime UI, host и приложения | Миграция definition и потребителей | Требует отдельного решения |
| Изменение anonymous event payload | Не подтверждено | Integration Events и подписчики | Сначала определить event schema/version policy | `WF-DEC-02` |

## 10. Границы с другими владельцами

| Тема | Остаётся у Workflow | Не дублируется здесь |
| --- | --- | --- |
| `Workflow` schema | Ссылка на effective definition и runtime mapping | Свойства артефакта — Configuration |
| Rules | Guard request и результат для routing | Rule schema и expression engine — Rules |
| Object Runtime | Archive state port и object identity | Mutation pipeline и object fields — Object Runtime |
| Tenant Security | Требуемые permission codes | Роли, assignments и access policy — Tenant Security |
| Audit History | Событие успешного workflow действия | Хранение, retention и поиск — Audit History |
| Integration Events | Payload и условие публикации | Envelope, outbox и доставка — Integration Events |
| фронтенд-платформа | DTO, которые нужны runtime-интерфейсу | Отображение, компоновка и навигация — фронтенд-платформа |

## 11. Источники и тесты

| Источник | Что подтверждает |
| --- | --- |
| [WorkflowRuntimeController][controller] | Маршруты и методы HTTP |
| [Workflow contracts][contracts] | Request/response DTO |
| [WorkflowRuntimeService][service] | Порядок проверки, ошибки и записи результата |
| [Workflow definition ports][definition-ports] и [state port][state-port] | Runtime definition, state и точки расширения |
| [Workflow persistence][db-context] | Таблицы и поля хранения |
| [Initialization payload][init-payload] | Payload запроса инициализации |
| [Initialization publisher][init-publisher] и [host handler][init-handler] | Публикация события после успешного создания объекта и передача его в runtime projection |
| [Workflow integration tests][workflow-tests] | Execute, revision, permission, guard, history и reassign scenarios |
| [Contract serialization tests][contract-tests] | Публичные имена полей и разделение command/transition/action |
[controller]: ../../../src/Platform/DMP.Platform.Workflow/Api/Controllers/WorkflowRuntimeController.cs
[contracts]: ../../../src/Platform/DMP.Platform.Contracts/Workflow/
[service]: ../../../src/Platform/DMP.Platform.Workflow/Application/Services/WorkflowRuntimeService.cs
[definition-ports]: ../../../src/Platform/DMP.Platform.Workflow/Application/Abstractions/
[state-port]: ../../../src/Platform/DMP.Platform.Workflow/Abstractions/IWorkflowStateStore.cs
[db-context]: ../../../src/Platform/DMP.Platform.Workflow/Infrastructure/Persistence/WorkflowDbContext.cs
[init-payload]: ../../../src/Platform/DMP.Platform.Contracts/IntegrationEvents/WorkflowInstanceInitializationRequestedPayload.cs
[init-publisher]: ../../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ApplicationRuntimeService.cs
[init-handler]: ../../../src/Hosts/DMP.Platform.Api/Composition/WorkflowInstanceInitializationRequestedEventHandler.cs
[workflow-tests]: ../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowDurableFoundationIntegrationTests.cs
[contract-tests]: ../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowRuntimeContractSerializationTests.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
