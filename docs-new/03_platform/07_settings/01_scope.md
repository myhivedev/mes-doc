---
id: DOC-03-07-01
title: 'Граница и владельцы - Settings'
type: scope
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

# Граница и владельцы - Settings

## 1. Назначение документа

Документ фиксирует, какие технические обязанности принадлежат Settings, а
какие остаются у Configuration, Tenant/Security, Audit History, Frontend
Platform и потребляющих модулей.

## 2. Что входит

- регистрация и проверка каталога настроек через `ISettingsCatalogManifest`;
- контракт `SettingsCatalogSnapshot`, который Settings читает из опубликованной
  baseline version;
- хранение `RuntimeSettingValue` для локальных переопределений;
- разрешение effective value по цепочке configuration scopes;
- runtime snapshot, typed access через `ModuleSettingKey<T>` и cache;
- HTTP операции для просмотра, изменения и сброса runtime values;
- чтение, запись и сброс собственных user preferences;
- capability/permission registration Settings и передача изменений в Audit History;
- frontend feature provider экрана Settings.

## 3. Что не входит

- схема и lifecycle конфигурационных артефактов;
- создание scope, публикация configuration version и построение effective
  configuration chain;
- смысл конкретной настройки и её использование в предметном модуле;
- роли, grants, assignment scopes и окончательное решение Tenant/Security;
- каноническая модель audit record и хранение истории;
- общий Studio shell, routing, UI-компоненты и общая локализация;
- автоматическая доставка событий, outbox и распределённая инвалидация cache;
- механизм перезапуска или reload, который должен следовать `SettingApplyPolicy`.

## 4. Граница с соседними областями и модулями

| Граница | Settings предоставляет или использует | Соседний владелец | Что остаётся у соседа | Основание |
| --- | --- | --- | --- | --- |
| Settings - Configuration | Использует `IEffectiveSettingsCatalogResolver`, `IConfigurationDbContext` и scope chain; получает baseline catalog snapshot | Configuration | Публикация, version lineage, scope persistence и system enum definitions | [resolver][resolver], [configuration contract][configuration-contract] |
| Settings - Tenant/Security | Регистрирует capability и permission codes; host adapter вызывает `ITenantSecurityService` | Tenant/Security | Роли, assignment и policy decision | [security manifest][security-manifest], [host authorizer][host-authorizer] |
| Settings - Audit History | Отправляет `AuditRecord` при изменении и сбросе runtime override | Audit History | Формат, persistence и запрос истории | [value service][value-service] |
| Settings - фронтенд-платформа | Публикует Settings feature provider и использует общие runtime/UI packages | фронтенд-платформа | Shell, route host, базовые controls и общие UI conventions | [frontend provider][frontend-provider] |
| Settings - Потребители runtime-контракта | Даёт `IModuleSettingsAccessor` и `ModuleSettingKey<T>` | Platform Runtime / Domain Modules | Решение, когда и зачем читать настройку | [accessor][accessor] |
| Settings - module manifest owner | Принимает `ISettingsCatalogManifest` и сохраняет definition в snapshot | Конкретный модуль | Семантика, описание и реальное применение setting | [demo manifest][demo-manifest] |

## 5. Соответствие требованиям

| Требование | Покрытие | Решение | Документ-владелец |
| --- | --- | --- | --- |
| Настройка должна иметь default, type и validation | Подтверждено для каталога | `SettingDefinition` и `SettingsCatalogBuilder` проверяют форму и default | [`03_contracts.md`](03_contracts.md) |
| Значение должно наследоваться по scope | Подтверждено для runtime | Effective snapshot строится по scope chain; разрешённые уровни задаёт `OverridePolicy` | [`04_runtime.md`](04_runtime.md) |
| Изменение runtime value не публикует configuration version | Подтверждено MVP | Значение сохраняется в Settings storage отдельно | [`02_architecture.md`](02_architecture.md) |
| Личный интерфейс пользователя должен иметь собственные preferences | Подтверждено ограниченно | Preferences хранятся отдельно и читаются только для текущего tenant/user context | [`03_contracts.md`](03_contracts.md) |
| XML или иной импорт должен быть источником каталога | Не подтверждено текущим кодом | Нормативный пакет описывает только `ISettingsCatalogManifest`; импорт остаётся в трассировке | [`90_traceability.md`](90_traceability.md) |

## 6. Ограничения версии

Текущий пакет описывает реализованный .NET/EF Core/SQL Server-or-InMemory MVP.
Он не фиксирует отдельную distributed cache policy, выполнение custom
validators, автоматический restart/reload или migration значений между разными
ключами настроек. Эти темы не расширяют текущую границу Settings и направлены
в `90_traceability.md`.

### Источники

- [Settings project][settings-project];
- [Settings contracts][settings-contracts];
- [Settings integration tests][settings-tests].
[settings-project]: ../../../src/Platform/DMP.Platform.Settings/
[settings-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Settings/
[settings-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/
[resolver]: ../../../src/Platform/DMP.Platform.Configuration/Application/Services/Catalog/EffectiveSettingsCatalogResolver.cs
[configuration-contract]: ../../../src/Platform/DMP.Platform.Configuration/Application/Abstractions/Catalog/IEffectiveSettingsCatalogResolver.cs
[security-manifest]: ../../../src/Platform/DMP.Platform.Settings/Infrastructure/Security/SettingsSecurityCatalogManifest.cs
[host-authorizer]: ../../../src/Hosts/DMP.Platform.Api/Composition/SettingsPermissionAuthorizer.cs
[value-service]: ../../../src/Platform/DMP.Platform.Settings/Application/Services/SettingsValueService.cs
[frontend-provider]: ../../../src/Frontend/apps/studio/src/features/settings/model/settingsFeatureProviderDefinition.tsx
[accessor]: ../../../src/Platform/DMP.Platform.Settings/Application/Abstractions/IModuleSettingsAccessor.cs
[demo-manifest]: ../../../src/Demos/DMP.DemoModules.ProductDefinition/Configuration/Settings/ProductDefinitionSettingsConfiguration.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
