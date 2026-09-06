---
id: DOC-03-05-90
title: 'Трассировка - Rules'
type: traceability
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

# Трассировка - Rules

## 1. Назначение документа

Документ показывает, какие сведения Rules подтверждены текущим кодом, какие
материалы принадлежат другим владельцам и какие будущие темы не являются
гарантиями MVP.

## 2. Источники и маршрут содержания

| Источник | Содержание | Маршрут | Статус |
| --- | --- | --- | --- |
| `docs/03 conf_artefacts/Rules.md` | Схема `Rule`, параметры, типы и модель привязок | Схема и редактирование остаются в Configuration; подтверждённые runtime-границы отражены в `01`-`04` | Владелец другой области |
| `docs/05 contracts/rule_and_condition_model_platform_contract.md` | Целевая модель `Rule`, `LocalCondition`, эффекты и разные механизмы выполнения | Использованы только подтверждённые границы; неподтверждённые возможности не перенесены как MVP | Частично подтверждено |
| `DMP.Platform.Rules` | Gateway, типы запроса и ответа и fallback | `02_architecture.md`, `03_contracts.md`, `08_operations.md` | Подтверждено MVP |
| `RuntimeRuleEvaluationGateway` | Фактическая оценка эффективного описания `Rule` | `02_architecture.md`, `04_runtime.md`, `07_quality.md` | Подтверждено, но ограничено |
| Workflow tests | Сериализация и реакция Workflow на коды результата | `03_contracts.md`, `07_quality.md` | Подтверждено тестами |

## 3. Расхождения и открытые вопросы

| ID | Тема | Текущее подтверждённое состояние | Расхождение или неопределённость | Влияние на текущую документацию | Статус | Владелец / следующий шаг | Документ после решения |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `RULE-DEC-01` | Объём Rules runtime после MVP | Текущий host оценивает только `Expression` с `Boolean` результатом | Не реализованы `DecisionTable`, `Script`, `Dsl`, вычисления и эффекты | Не блокирует: текущий ограниченный контракт описан | Будущая доработка | Rules / Architecture: определить отдельный контракт и механизм выполнения перед расширением | `03_contracts.md`, `04_runtime.md` |
| `RULE-DEC-02` | Семантика параметров Rule | Коды параметров попадают в список требуемых полей; значений параметров в запросе нет | Полная привязка параметров и подстановка не реализованы | Не блокирует: документ не обещает оценку параметров | Расхождение кода и источника | Configuration / Rules: согласовать структуру параметров и привязку параметров при расширении | `03_contracts.md`, `04_runtime.md` |
| `RULE-DEC-03` | Production-политика fallback | Fallback возвращает `NotEvaluated` и не разрешает проверку | Не определено, допустим ли fallback в production | Не блокирует MVP; важно до промышленной готовности | Не блокирует текущую документацию | Platform Operations / Rules: определить запрет или явное разрешение fallback | `07_quality.md`, `08_operations.md` |
| `RULE-DEC-04` | Наблюдаемость Rules | Контракт содержит коды результата и диагностических записей | Нет отдельного обязательного набора метрик, интервалов трассировки и проверок работоспособности | Не блокирует текущую документацию | Не блокирует текущую документацию | Foundation / Platform Operations: определить общий контракт наблюдаемости | `07_quality.md`, `08_operations.md` |

## 4. Переданные другим владельцам темы

| Тема | Владелец | Почему не входит в Rules |
| --- | --- | --- |
| Схемы `Rule`, `RuleParameter` и узлы привязок | Configuration | Rules получает эффективное описание, но не владеет схемой редактирования |
| Guard selection и переход workflow | Workflow | Workflow определяет применимость binding и реакцию на результат |
| Descriptor и детали объекта | Object Runtime | Rules только запрашивает данные для оценки |
| Предметные инварианты | Domain Modules | Они не должны заменяться настраиваемым правилом |
| Editor и рендерер | фронтенд-платформа / Configuration | В Rules нет UI-контракта |
| Аудит и доставка событий | Audit History / Integration Events | В текущем MVP Rules не публикует события и не хранит аудит |

## 5. Источники кода

- `src/Platform/DMP.Platform.Rules/Abstractions/IRuleEvaluationGateway.cs`;
- `src/Platform/DMP.Platform.Rules/Services/NotConfiguredRuleEvaluationGateway.cs`;
- `src/Platform/DMP.Platform.Rules/DependencyInjection.cs`;
- `src/Hosts/DMP.Platform.Api/Composition/RuntimeRuleEvaluationGateway.cs`;
- `src/Platform/DMP.Platform.Workflow/Application/Services/WorkflowRuntimeService.cs`;
- `tests/DMP.Platform.IntegrationTests/Workflow/WorkflowRuntimeContractSerializationTests.cs`;
- `tests/DMP.Platform.IntegrationTests/Workflow/WorkflowDurableFoundationIntegrationTests.cs`.

Проверенная ревизия: `origin/master@3e2037ac37683eef331d70223c6babfc61aa0539`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
