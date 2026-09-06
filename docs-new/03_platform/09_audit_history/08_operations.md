---
id: DOC-03-09-08
title: 'Эксплуатация платформенной области - Audit History'
type: operation
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
updated_at: 2026-08-27 16:52
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed
---

# Эксплуатация платформенной области - Audit History

## 1. Назначение документа

Документ описывает фактическую регистрацию Audit History, создание хранилища и
проверки запуска. Он не вводит эксплуатационные гарантии, которых нет в коде.

## 2. Запуск и готовность

Host-приложение вызывает `AddPlatformAuditHistory(configuration)`. Метод:

1. получает connection string `Platform`;
2. выбирает SQL Server при наличии connection string;
3. иначе выбирает EF Core InMemory database;
4. регистрирует `IAuditHistoryWriter` с областью времени жизни `scoped` и реализацией
   `PersistentAuditHistoryWriter`.

После регистрации composition root вызывает
`AuditHistoryDatabaseInitializer.InitializeAsync`. Для реляционного провайдера он
создаёт схему `audit_history` и таблицу `audit_records`, если их ещё нет.

| Шаг запуска | Условие | Идемпотентность | Критерий готовности | Откат |
| --- | --- | --- | --- | --- |
| Зарегистрировать сервисы Audit History | Host вызывает `AddPlatformAuditHistory` | Повторная регистрация выполняется по правилам DI host | Разрешается `IAuditHistoryWriter` | Удалить регистрацию из композиции host |
| Выбрать провайдер хранения | Есть `ConnectionStrings:Platform` или имя InMemory-базы | Выбор повторяем для одинаковой конфигурации | Создан зарегистрированный `AuditHistoryDbContext` | Исправить конфигурацию и перезапустить host |
| Инициализировать базу | Вызван `AuditHistoryDatabaseInitializer.InitializeAsync` | Схема и таблица создаются только при отсутствии | Схема `audit_history` и таблица `audit_records` доступны | Исправить права DDL или вернуть конфигурацию host |
| Выполнить первую запись | Средство записи разрешено и передано потребителю | Повторный вызов создаёт отдельную запись | `WriteAsync` завершён без ошибки | Проверить контекст, `Details` и соединение с БД |

## 3. Миграции

| Изменение | Порядок | Совместимость | Проверка | Откат |
| --- | --- | --- | --- | --- |
| Собственные миграции EF Core Audit History | Не применяются: миграционные классы в модуле не найдены; для реляционной базы инициализатор вызывает `MigrateAsync`, затем выполняет проверку и прямой DDL | Не меняет контракт `IAuditHistoryWriter` | Проверить наличие `audit_history.audit_records` и индексов | Исправить права или вернуть внешнее изменение базы владельцу базы |

### 3.1 Настройки хранения

| Настройка | Смысл | Значение по умолчанию |
| --- | --- | --- |
| `ConnectionStrings:Platform` | Выбор провайдера SQL Server | Если отсутствует, используется InMemory |
| `AuditHistory:InMemoryDatabaseName` | Имя базы EF Core InMemory | `audit-history` |

Текущий инициализатор выполняет SQL Server DDL напрямую и не предоставляет
отдельной команды срока хранения, архивирования или очистки.

## 4. Диагностика

| Сигнал | Диагностика | Действие | Критерий восстановления | Эскалация |
| --- | --- | --- | --- | --- |
| `IAuditHistoryWriter` не разрешается | Проверить вызов `AddPlatformAuditHistory` и композицию host | Подключить модуль к корню композиции | Средство записи разрешается из DI | Владелец композиции host |
| Хранилище SQL Server не создаётся | Проверить `ConnectionStrings:Platform`, доступ к базе и права DDL | Исправить строку подключения или права | Схема и таблица доступны | Владелец базы и Platform Operations |
| Запись завершается ошибкой | Проверить `ActionCode`, `CorrelationId`, сериализацию `Details` и соединение с БД | Исправить данные или подключение; повторная попытка не задана | `WriteAsync` завершается без ошибки | Область-потребитель и владелец хранилища |
| Записи есть, но полный снимок велик | Проверить размер `audit_records` и отсутствие фильтра | Не использовать `GetSnapshot` как масштабируемый API чтения | Проблема чтения устранена отдельным будущим API | Владелец будущего API чтения |
| В промышленной среде нет защиты хранения | Проверить разрешения БД и политику резервного копирования host | Применить внешнюю политику защиты | Политика host подтверждена | Platform Operations |

Отдельные конечная точка проверки состояния, панель и оповещения Audit History в коде не
подтверждены.

## 5. Восстановление

Текущий код не реализует повторные попытки (`retry`), outbox, повторную обработку, очистку по сроку хранения или
восстановление пропущенной записи. Операционная команда должна рассматривать
запись аудита как отдельный шаг сценария потребителя и проверять `CorrelationId`
при расследовании ошибки. Политика допуска в промышленную среду и защиты хранения
остаётся открытой темой в [`90_traceability.md`](90_traceability.md).

## 6. Откат и эскалация

Audit History не содержит собственной команды отката или удаления записей.
При ошибке запуска откатываются конфигурация host и внешние изменения базы по
правилам их владельцев. Ошибки формирования `AuditRecord` передаются области-
потребителю; ошибки создания схемы и таблицы эскалируются владельцу базы и
Platform Operations.

### 6.1 Источники

- [DI][di];
- [database initializer][initializer];
- [контекст базы][db-context];
- [композиция host][composition].

[di]: ../../../src/Platform/DMP.Platform.AuditHistory/DependencyInjection.cs
[initializer]: ../../../src/Platform/DMP.Platform.AuditHistory/Infrastructure/Persistence/AuditHistoryDatabaseInitializer.cs
[db-context]: ../../../src/Platform/DMP.Platform.AuditHistory/Infrastructure/Persistence/AuditHistoryDbContext.cs
[composition]: ../../../src/Hosts/DMP.Platform.Api/Composition/PlatformApiCompositionExtensions.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
