---
id: DOC-03-05-08
title: 'Эксплуатация - Rules'
type: operation
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

# Эксплуатация - Rules

## 1. Назначение документа

Документ описывает регистрацию Rules в DI и проверяемые эксплуатационные
ограничения. Отдельного процесса миграции или хранилища у Rules нет.

## 2. Регистрация компонентов

Библиотека Rules предоставляет `AddPlatformRules(IServiceCollection)`. Она
регистрирует scoped резервную реализацию (`fallback`)
`NotConfiguredRuleEvaluationGateway` через
`TryAddScoped`.

Host может заменить fallback собственной реализацией. В текущем API host
регистрирует `RuntimeRuleEvaluationGateway` как scoped реализацию после
подключения адаптеров Configuration и Object Runtime.

| Шаг | Условие | Проверка | Результат |
| --- | --- | --- | --- |
| Подключить библиотеку Rules | Проект host использует Rules | В DI доступен `AddPlatformRules` | Зарегистрирована резервная реализация |
| Подключить рабочий адаптер | Host использует Configuration и Object Runtime | В DI зарегистрирован `RuntimeRuleEvaluationGateway` | Workflow получает рабочую оценку |
| Проверить обязательные зависимости | Выполняется guard evaluation | Resolver и Object Runtime разрешаются из DI | Нет ошибки регистрации |

## 3. Поведение при неполной конфигурации

Если рабочий gateway не заменил fallback, вызов возвращает
`NotEvaluated` и `RULE_GATEWAY_NOT_CONFIGURED`. Это не является разрешением
операции. Production-допустимость такого fallback остаётся открытым вопросом.

## 4. Диагностика

| Сигнал | Диагностика | Действие | Критерий восстановления |
| --- | --- | --- | --- |
| `RULE_GATEWAY_NOT_CONFIGURED` | Host использует fallback | Проверить регистрацию рабочей реализации | `IRuleEvaluationGateway` разрешается в `RuntimeRuleEvaluationGateway` |
| `RUNTIME_OBJECT_DESCRIPTOR_NOT_FOUND` | Нет descriptor объекта | Проверить регистрацию descriptor | Descriptor доступен для типа объекта модуля |
| `RULE_NOT_FOUND` | `Rule` отсутствует в эффективной конфигурации | Проверить публикацию и scope | Эффективное описание `Rule` найдено |
| `RULE_ENGINE_NOT_SUPPORTED` | Используется неподдержанный механизм | Проверить схему и ограничение MVP | `Rule` использует `Expression` |
| `RULE_EXPRESSION_INVALID` | Выражение не разбирается текущим разборщиком | Исправить конфигурацию или вынести возможность в backlog | Выражение соответствует поддержанному синтаксису |

## 5. Ограничения эксплуатации

Rules не владеет миграциями, retention, outbox delivery или восстановлением
данных. Эти операции принадлежат Configuration, Object Runtime, Integration
Events и другим владельцам. Для промышленной эксплуатации отдельно требуется
решение о допустимости fallback и обязательном наборе метрик и проверок
работоспособности.

## 6. Источники

- [DI registration][di];
- [fallback][fallback];
- [регистрация в составе host-приложения][composition].
[di]: ../../../src/Platform/DMP.Platform.Rules/DependencyInjection.cs
[fallback]: ../../../src/Platform/DMP.Platform.Rules/Services/NotConfiguredRuleEvaluationGateway.cs
[composition]: ../../../src/Hosts/DMP.Platform.Api/Composition/PlatformApiCompositionExtensions.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
