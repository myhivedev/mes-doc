---
id: DOC-03-05-03
title: 'Контракты - Rules'
type: contract
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
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Контракты - Rules

## 1. Назначение и границы

Документ описывает единственный подтверждённый межкомпонентный контракт Rules:
`IRuleEvaluationGateway` и связанные с ним типы C# `record`. HTTP API и события
Rules в текущем MVP отсутствуют.

Схема `Rule` и схемы привязок описываются в Configuration и здесь не
дублируются.

## 2. Источники истины и владельцы

| Контракт или тип | Источник структуры | Владелец | Потребитель |
| --- | --- | --- | --- |
| `IRuleEvaluationGateway` | C# interface | Rules | Workflow Runtime и состав host-приложения |
| `RuleEvaluationRequest` | C# record | Rules | Workflow Runtime / адаптер |
| `RuleEvaluationBinding` | C# record | Rules | Workflow Runtime / адаптер |
| `RuleEvaluationResponse` | C# record | Rules | Workflow Runtime |
| `RuleEvaluationIssue` | C# record | Rules | Workflow Runtime |

## 3. Карта контрактов

| Контракт | Вид | Владелец | Потребитель | Статус сведения | Подробное описание |
| --- | --- | --- | --- | --- | --- |
| `IRuleEvaluationGateway.EvaluateAsync` | C# application port | Rules | Workflow Runtime | Подтверждено, но ограничено | [раздел 6.1](#61-iroleevaluationgateway) |
| `RuleEvaluationRequest` | C# тип запроса | Rules | Workflow Runtime | Подтверждено, но ограничено | [раздел 5.1](#51-ruleevaluationrequest) |
| `RuleEvaluationResponse` | C# тип ответа | Rules | Workflow Runtime | Подтверждено, но ограничено | [раздел 5.3](#53-ruleevaluationresponse) |

## 4. HTTP-контракты

В текущем MVP Rules не публикует собственный HTTP endpoint. Workflow может
иметь HTTP API, но его маршрут и HTTP DTO принадлежат Workflow и описываются в
его `03_contracts.md`.

## 5. Общие типы и DTO

### 5.1. `RuleEvaluationRequest`

Тип передаёт полный контекст проверки из Workflow в gateway.

| Поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ModuleCode` | `string` | нет | обязательно | код модуля | Модуль владельца объекта | Не должен быть пустым | [interface][gateway] |
| `ObjectTypeCode` | `string` | нет | обязательно | код типа объекта | Тип проверяемого объекта | Может содержать module prefix | [interface][gateway] |
| `ObjectId` | `string` | нет | обязательно | идентификатор объекта | Объект для проверки | Нужен для загрузки деталей объекта | [interface][gateway] |
| `WorkflowCode` | `string` | нет | обязательно | код workflow | Контекст перехода | Передаётся Workflow | [interface][gateway] |
| `WorkflowDefinitionRevision` | `string` | нет | обязательно | revision code | Зафиксированная revision workflow | Не изменяется Rules | [interface][gateway] |
| `CommandCode` | `string` | нет | обязательно | код команды | Команда перехода | Семантику задаёт Workflow | [interface][gateway] |
| `FromStateCode` | `string` | нет | обязательно | код состояния | Исходное состояние | Семантику задаёт Workflow | [interface][gateway] |
| `TransitionCode` | `string` | нет | обязательно | код перехода | Выбранный переход | Семантику задаёт Workflow | [interface][gateway] |
| `Bindings` | `IReadOnlyCollection<RuleEvaluationBinding>` | нет | обязательно | коллекция привязок | Правила для оценки | Пустая коллекция даёт `Satisfied` | [interface][gateway] |
| `Context` | `IReadOnlyDictionary<string, object?>?` | да | `null` | произвольный контекст | Дополнительные данные вызова | Текущий evaluator не использует его для подстановки | [interface][gateway] |

### 5.2. `RuleEvaluationBinding`

| Поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `BindingCode` | `string` | нет | обязательно | код binding | Идентифицирует причину результата | Возвращается в issue | [interface][gateway] |
| `BindingType` | `string` | нет | обязательно | тип привязки | Классификация привязки | Правила применимости задаёт потребитель | [interface][gateway] |
| `RuleCode` | `string?` | да | допускается null в типе | код `Rule` | Ссылка на эффективное описание правила | Для текущей проверки guard в Workflow фактически требуется | [interface][gateway] |
| `TargetType` | `string?` | да | допускается null | тип цели | Контекст цели | Интерпретируется Workflow | [interface][gateway] |
| `TargetCode` | `string?` | да | допускается null | код цели | Конкретная цель | Интерпретируется Workflow | [interface][gateway] |
| `ExecutionPointCode` | `string?` | да | допускается null | код точки | Контекст выполнения | Отбор применимых bindings делает Workflow | [interface][gateway] |
| `Order` | `int?` | да | допускается null | целое число | Порядок привязки | Стабильный порядок задаёт потребитель | [interface][gateway] |

### 5.3. `RuleEvaluationResponse`

| Поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ResultCode` | `string` | нет | обязательно | `Satisfied`, `Rejected`, `NotEvaluated` | Итог оценки | Сравнение выполняется по точному коду | [interface][gateway] |
| `Issues` | `IReadOnlyCollection<RuleEvaluationIssue>` | нет | обязательно | коллекция диагностических записей | Причины отказа или невозможности оценки | Для `Satisfied` текущий адаптер возвращает пустую коллекцию | [interface][gateway] |

Вспомогательные свойства `IsSatisfied`, `IsRejected` и `IsNotEvaluated` являются
вычисляемыми значениями ответа и не являются отдельными полями транспорта.

Строковые поля запроса и привязки объявлены как non-nullable в C#-сигнатурах.
Gateway не задаёт для каждого из них отдельный код ошибки пустой строки; при
некорректном значении результат определяется последующим поиском descriptor или
Rule и описан в разделе 8.

### 5.4. `RuleEvaluationIssue`

| Поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `Code` | `string` | нет | обязательно | стабильный reason code | Машинно читаемая причина | Конкретные коды перечислены в разделе 8 | [interface][gateway] |
| `Message` | `string` | нет | обязательно | текст сообщения | Диагностика причины | Не заменяет `Code` | [interface][gateway] |
| `BindingCode` | `string?` | да | default `null` | код binding | Связь issue с привязкой | Может отсутствовать для общей ошибки | [interface][gateway] |

## 6. C#-контракты и точки расширения

### 6.1. `IRuleEvaluationGateway`

```csharp
Task<RuleEvaluationResponse> EvaluateAsync(
    RuleEvaluationRequest request,
    CancellationToken cancellationToken);
```

Потребитель передаёт запрос и ожидает один `RuleEvaluationResponse`. Контракт
асинхронный, принимает `CancellationToken` и не обещает, что правило будет
оценено: `NotEvaluated` является штатным результатом для отсутствующего
descriptor, отсутствующего `Rule`, неподдерживаемого механизма или вида
результата (`LogicEngineType`/`LogicReturnKind`) или
неподдерживаемого выражения.

`IRuleEvaluationGateway` является межкомпонентным контрактом, потому что его
вызывает Workflow, а реализация выбирается composition root. Приватные методы
`RuntimeRuleEvaluationGateway` в этот документ не входят.

### 6.2. `RuleEvaluationResultCodes`

| Код | Смысл | Действие потребителя |
| --- | --- | --- |
| `Satisfied` | Все переданные привязки прошли оценку или привязок нет | Продолжить собственный сценарий |
| `Rejected` | Хотя бы одна оценённая привязка отклонила объект | Отказать в операции и использовать `Issues` |
| `NotEvaluated` | Оценка не выполнена | Не считать правило разрешённым; применить политику потребителя |

## 7. Контракты событий

Rules не публикует и не принимает собственные integration events в текущем MVP.

## 8. Ошибки и отказоустойчивость

| Код | Ситуация | Результат | Связь |
| --- | --- | --- | --- |
| `RULE_GATEWAY_NOT_CONFIGURED` | Использован fallback | `NotEvaluated` | [fallback][fallback] |
| `RUNTIME_OBJECT_DESCRIPTOR_NOT_FOUND` | Descriptor объекта не найден | `NotEvaluated` | [host gateway][host-gateway] |
| `RUNTIME_OBJECT_NOT_FOUND` | Объект не найден при загрузке деталей | `NotEvaluated` | [host gateway][host-gateway] |
| `RULE_CODE_REQUIRED` | У binding нет `RuleCode` | `NotEvaluated` | [host gateway][host-gateway] |
| `RULE_NOT_FOUND` | Эффективное описание `Rule` не найдено | `NotEvaluated` | [host gateway][host-gateway] |
| `RULE_ENGINE_NOT_SUPPORTED` | Механизм отличается от `Expression` | `NotEvaluated` | [host gateway][host-gateway] |
| `RULE_RETURN_KIND_NOT_SUPPORTED` | Вид результата отличается от `Boolean` | `NotEvaluated` | [host gateway][host-gateway] |
| `RULE_EXPRESSION_INVALID` | Выражение не соответствует поддержанному синтаксису | `NotEvaluated` | [host gateway][host-gateway] |
| `RULE_REJECTED` | Поддержанное выражение вернуло false | `Rejected` | [host gateway][host-gateway] |

Контракт не описывает retry, delivery или transaction policy. Эти аспекты
принадлежат вызывающему сценарию и владельцам инфраструктуры.

## 9. Совместимость и изменение контрактов

Изменение `ResultCode`, состава запроса/ответа или смысла `NotEvaluated`
требует согласованного изменения Rules и потребителя Workflow. Добавление новых
полей record требует проверить сериализацию и всех потребителей. Внутреннее
изменение разборщика или механизма оценки не является изменением C#-контракта, если сохраняется
наблюдаемая семантика результата.

## 10. Границы с другими владельцами

| Владелец | Что описывается там | Что описывается здесь |
| --- | --- | --- |
| Configuration | Схемы `Rule`, `RuleParameter`, `ObjectRuleBinding`, `WorkflowRuleBinding` | Как runtime получает эффективное описание `Rule` |
| Object Runtime | Descriptor и загрузка деталей объекта | Требование к чтению значений для оценки |
| Workflow | Guard selection и реакция на результат | Формат запроса и результата gateway |
| Domain Modules | Обязательные предметные проверки | Ничего о бизнес-сущностях |
| фронтенд-платформа | Editor и client behavior | Ничего о рендерер |

## 11. Источники и тесты

- [Rules gateway interface][gateway];
- [адаптер host-приложения][host-gateway];
- [реализация fallback][fallback];
- [Workflow consumer][workflow];
- [serialization test][serialization-test];
- [Workflow integration tests][workflow-tests].

Тесты в рамках подготовки документации не запускались.
[gateway]: ../../../src/Platform/DMP.Platform.Rules/Abstractions/IRuleEvaluationGateway.cs
[fallback]: ../../../src/Platform/DMP.Platform.Rules/Services/NotConfiguredRuleEvaluationGateway.cs
[host-gateway]: ../../../src/Hosts/DMP.Platform.Api/Composition/RuntimeRuleEvaluationGateway.cs
[workflow]: ../../../src/Platform/DMP.Platform.Workflow/Application/Services/WorkflowRuntimeService.cs
[serialization-test]: ../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowRuntimeContractSerializationTests.cs
[workflow-tests]: ../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowDurableFoundationIntegrationTests.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
