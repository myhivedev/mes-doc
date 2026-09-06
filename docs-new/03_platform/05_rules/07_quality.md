---
id: DOC-03-05-07
title: 'Качество - Rules'
type: assurance
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

# Качество - Rules

## 1. Назначение документа

Документ фиксирует проверяемые свойства текущего Rules MVP, тестовые
свидетельства и ограничения покрытия. Он не объявляет полноту будущего
`Rule Engine`.

## 2. Проверяемые гарантии

| Гарантия | Условие | Ожидаемый результат | Доказательство |
| --- | --- | --- | --- |
| Сигнатура gateway стабильна | Создан `RuleEvaluationRequest` | Поля и коды доступны потребителю | [тест сериализации][serialization-test] |
| Результат отклонения различим | Gateway возвращает `Rejected` | `IsRejected` истинно | [interface][gateway] |
| Не выполненная оценка различима | Gateway возвращает `NotEvaluated` | `IsNotEvaluated` истинно | [workflow tests][workflow-tests] |
| Workflow учитывает результат guard | Rules gateway возвращает отказ или отсутствие оценки | Команда становится недоступной или отклоняется | [workflow tests][workflow-tests] |

## 3. Известные ограничения

- отдельного набора интеграционных тестов Rules в проекте не найдено;
- основные интеграционные проверки находятся в тестах Workflow;
- `RuntimeRuleEvaluationGateway` реализован в составе host-приложения, поэтому его
  поведение зависит от регистрации адаптеров Configuration и Object Runtime;
- в текущем MVP не подтверждены проверки выражений по множеству свойств,
  лимиты времени оценки и метрики производительности;
- политика production для fallback `NotConfiguredRuleEvaluationGateway` не
  закреплена.

## 4. Наблюдаемость

Контракт Rules возвращает reason codes и binding code, но отдельные метрики,
интервалы трассировки и проверки работоспособности Rules не определены. Общая политика
наблюдаемости должна быть согласована на уровне Foundation/Operations до
промышленной готовности.

## 5. Источники и статус проверки

Код и тесты проверены по `origin/master@3e2037ac37683eef331d70223c6babfc61aa0539`.
Тесты в рамках подготовки документации не запускались.
[gateway]: ../../../src/Platform/DMP.Platform.Rules/Abstractions/IRuleEvaluationGateway.cs
[serialization-test]: ../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowRuntimeContractSerializationTests.cs
[workflow-tests]: ../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowDurableFoundationIntegrationTests.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
