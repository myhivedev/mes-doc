---
id: DOC-03-09-07
title: 'Качество и проверки платформенной области - Audit History'
type: assurance
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: audit_history
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed
---

# Качество и проверки платформенной области - Audit History

## 1. Назначение документа

Документ перечисляет фактические тестовые свидетельства и ограничения качества
Audit History. Наличие теста потребителя не означает, что все операции системы
аудируются.

## 2. Надёжность

| Отказ | Гарантия | Предел | Восстановление | Проверка |
| --- | --- | --- | --- | --- |
| `IAuditHistoryWriter` отсутствует в Object Runtime | `ObjectRuntimeAuditSink` завершает вызов без записи | Запись аудита пропускается | Автоматическое восстановление отсутствует; настройку host проверяет владелец композиции | `ObjectRuntimeAuditSink` |
| Ошибка сериализации или сохранения | Ошибка передаётся как ошибка `Task` | Повторная попытка не выполняется | Повторный вызов остаётся ответственностью потребителя | Интеграционные сценарии средства записи |
| Одновременная запись в памяти | Список защищён блокировкой | Это не гарантия провайдера SQL Server | Отдельного восстановления не требуется | `InMemoryAuditHistoryWriter`; нагрузочный тест отсутствует |
| Полное покрытие значимых операций | Не гарантируется общим средством записи | Подтверждены только проверенные сценарии потребителей | Покрытие добавляется вместе с операцией владельцем области | Матрица полного покрытия отсутствует |

## 3. Наблюдаемость

| Сигнал | Источник | Корреляция | Потребитель | Диагностический пробел |
| --- | --- | --- | --- | --- |
| `CorrelationId` записи | `AuditRecord` и хранилище | Значение `CorrelationId` операции | Область-потребитель и расследование | Сквозная связь со всеми API, событиями и трассировками не оформлена |
| Ошибка записи | Исключение `WriteAsync` | Контекст операции и `CorrelationId` | Потребитель средства записи | Отдельные метрики и журналы ошибок Audit History не подтверждены |
| Размер таблицы `audit_records` | SQL Server | `TenantId` и `OccurredAtUtc` доступны в записи | Platform Operations | Нет метрик роста, задержки и срока хранения |
| Состояние средства записи | Нет отдельной проверки состояния | — | Platform Operations | Конечная точка проверки состояния Audit History не найдена |

## 4. Производительность

| Сценарий нагрузки | Предел или SLO | Метод | Результат |
| --- | --- | --- | --- |
| Последовательная запись в SQL Server | Не задан | `SaveChangesAsync` в `PersistentAuditHistoryWriter` | Измерение отсутствует |
| Чтение полного снимка | Ограничение задаётся объёмом `audit_records`; SLO не задан | `GetSnapshot` читает и сортирует весь доступный набор | Отдельный нагрузочный замер отсутствует |
| Конкурентная запись в памяти | SLO не задан | Блокировка списка в `InMemoryAuditHistoryWriter` | Нагрузочный тест отсутствует |

## 5. Проверки и результаты

| Сценарий проверки | Уровень | Источник теста | Результат запуска | Пробел |
| --- | --- | --- | --- | --- |
| Object Runtime записывает запись изменения | Интеграционный | `ObjectMutationPipelineIntegrationTests.ExecuteAsync_WithAuditHistoryWriter_WritesObjectMutationAuditRecord` | Подтверждено тестом | Покрыт проверенный сценарий, не все mutation-команды |
| Tenant/Security записывает запись аудита | Интеграционный | `TenantSecurityIntegrationTests.CreateTenant_PublishesIntegrationEvent_AndWritesAuditRecord`, сценарий пароля | Подтверждено тестами | Полное покрытие команд не доказано |
| Workflow использует writer | Интеграционный | `WorkflowDurableFoundationIntegrationTests` | Подтверждено для проверенных сценариев | Полное покрытие Workflow-команд не доказано |
| Settings использует средство записи | Интеграционный | `SettingsStorageIntegrationTests` | Подтверждено для проверенных сценариев | Полное покрытие Settings-операций не доказано |
| Хранилище сохраняет детали и индексы | Интеграционный и инфраструктурный | SQL initializer и интеграционные тесты | Подтверждено для проверенных провайдеров | Нагрузочные и конкурентные проверки отсутствуют |
| Полный набор интеграционных тестов | Не запускался | — | Не запускался | Полная регрессионная проверка не выполнялась |

## 6. Неподтверждённые свойства

- API чтения и отдельный экран просмотра;
- политика неизменяемости на уровне базы данных;
- политика хранения и архивирование;
- повторные попытки (`retry`), outbox и атомарность с состоянием потребителя;
- общий реестр кодов действий и схема для деталей;
- обязательность audit для полного набора операций.

### 6.1 Источники

- [Object Runtime tests][runtime-tests];
- [Tenant/Security tests][tenant-tests];
- [Workflow tests][workflow-tests];
- [Settings tests][settings-tests].

[runtime-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectMutationPipelineIntegrationTests.cs
[tenant-tests]: ../../../tests/DMP.Platform.IntegrationTests/FoundationSlice/TenantSecurityIntegrationTests.cs
[workflow-tests]: ../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowDurableFoundationIntegrationTests.cs
[settings-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsStorageIntegrationTests.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
