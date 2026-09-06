---
id: DOC-03-07-08
title: 'Эксплуатация - Settings'
type: operation
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: settings
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

# Эксплуатация - Settings

## 1. Назначение документа

Документ описывает подключение Settings, создание хранилища, диагностику
недоступного каталога и основные эксплуатационные ограничения.

## 2. Подключение и запуск

`AddPlatformSettings` регистрирует `SettingsDbContext`, application services,
runtime cache и fallback `MissingSettingsPermissionAuthorizer`. Основной API
подключает `SettingsController`, заменяет permission authorizer host-адаптером и
вызывает `SettingsDatabaseInitializer.InitializeAsync`.

| Шаг | Условие | Проверка | Результат |
| --- | --- | --- | --- |
| Зарегистрировать Settings | Host вызывает `AddPlatformSettings` | В DI доступны `ISettingsValueService`, `ISettingsPageService`, `IModuleSettingsAccessor` | Settings services разрешаются |
| Подключить catalog manifests | Module manifest зарегистрирован как `ISettingsCatalogManifest` | Configuration snapshot provider видит manifest | Catalog участвует в baseline publication |
| Подключить authorization adapter | Host регистрирует `ISettingsPermissionAuthorizer` | Вызов делегируется Tenant/Security | Edit/reset permissions вычисляются |
| Инициализировать database | Старт приложения | `SettingsDatabaseInitializer` выполняет `MigrateAsync` для relational provider | Schema `settings` создана/обновлена |

## 3. Persistence и migrations

Relational Settings DB использует schema `settings` и EF migrations:

- `InitialSettingsStorage` создаёт `RuntimeSettingValues`;
- `AddUserPreferenceValues` создаёт `UserPreferenceValues`;
- SQL Server connection берётся из connection string `Platform`;
- если connection string отсутствует, host использует InMemory database с именем
  `Settings:InMemoryDatabaseName` или `settings`.

Миграции Settings не изменяют Configuration schema. Snapshot каталога
`ConfigurationVersion` создаётся и хранится миграциями Configuration.

## 4. Диагностика

| Ситуация | Технический код или сигнал | Проверка | Действие |
| --- | --- | --- | --- |
| Нет published configuration chain | `PublishedVersionNotFound` | Проверить scope и published versions Configuration | Опубликовать требуемую version согласно Configuration procedure |
| Нет baseline catalog snapshot | `SettingsCatalogSnapshotMissing` | Проверить поля snapshot/fingerprint baseline version | Проверить baseline publication и регистрацию manifests |
| Snapshot повреждён | `SettingsCatalogSnapshotInvalid` | Проверить JSON и fingerprint | Пересоздать корректную baseline publication |
| Нет scope | `ScopeNotFound` или exception access guard | Проверить Configuration scopes | Исправить scope lifecycle в Configuration |
| Нет права | `SETTINGS_SCOPE_ACCESS_FORBIDDEN` или permission false | Проверить assignments, role и host authorizer | Передать решение Tenant/Security |
| Значение не проходит проверку | `ValueTypeMismatch`, `ValueNotAllowed`, range/length code | Сверить definition и request value | Исправить значение или definition owner |
| Runtime value устарел в cache | Snapshot key содержит старый fingerprint/scope | Проверить update/reset invalidation и TTL | Очистить instance cache или дождаться TTL; distributed policy не реализована |

## 5. Восстановление и ограничения

Settings не содержит отдельного процесса резервного копирования и восстановления
Configuration baseline. Для восстановления runtime values используется backup
Settings storage; после восстановления snapshots могут быть пересозданы через
cache invalidation или истечение TTL.

Изменение catalog manifest становится доступным во время выполнения только после публикации
нового baseline snapshot. Простое изменение кода host без публикации version не
является описанным способом обновить effective catalog.

## 6. Источники

- [DI registration][di];
- [database initializer][initializer];
- [database context][db-context];
- [host composition][composition].
[di]: ../../../src/Platform/DMP.Platform.Settings/DependencyInjection.cs
[initializer]: ../../../src/Platform/DMP.Platform.Settings/Infrastructure/Persistence/SettingsDatabaseInitializer.cs
[db-context]: ../../../src/Platform/DMP.Platform.Settings/Infrastructure/Persistence/SettingsDbContext.cs
[composition]: ../../../src/Hosts/DMP.Platform.Api/Composition/PlatformApiCompositionExtensions.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
