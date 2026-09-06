---
id: DOC-03-07-03
title: 'Контракты - Settings'
type: contract
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
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Контракты - Settings

## 1. Назначение и границы

Документ описывает публичные HTTP-, DTO- и C#-контракты Settings. Алгоритмы
разрешения значений, схема хранения и frontend-рендерер описаны в
[`04_runtime.md`](04_runtime.md), [`02_architecture.md`](02_architecture.md) и
[`06_user_experience.md`](06_user_experience.md). Схема Configuration version
остаётся у Configuration.

## 2. Источники истины и владельцы

| Контрактный слой | Источник структуры | Владелец | Потребитель |
| --- | --- | --- | --- |
| HTTP route и DTO | `SettingsController` и `DMP.Platform.Contracts.Settings` | Settings | Studio и HTTP clients |
| Catalog registration | `ISettingsCatalogManifest`, `SettingsCatalogBuilder` и registration records | Settings contracts; meaning belongs to registering module | Configuration publication и Settings runtime |
| Runtime read port | `IModuleSettingsAccessor`, `ISettingsRuntimeSnapshotProvider` | Settings | Platform Runtime и модули |
| Configuration catalog resolver | `IEffectiveSettingsCatalogResolver` | Configuration | Settings |
| Permission adapter | `ISettingsPermissionAuthorizer` | Settings contract; decision belongs to Tenant/Security | Settings page |
| Audit writer | `IAuditHistoryWriter` | Audit History | Settings value service |

## 3. Карта контрактов

| Контракт | Вид | Владелец | Потребитель | Статус сведения | Подробное описание |
| --- | --- | --- | --- | --- | --- |
| `SettingsController` page | HTTP | Settings | Studio | Подтверждено MVP | [4.1](#41-получение-страницы-настроек) |
| `SettingsController` runtime values | HTTP | Settings | Studio или административный клиент | Подтверждено MVP | [4.2](#42-изменение-runtime-значения) и [4.3](#43-сброс-runtime-значения) |
| `SettingsController` user preferences | HTTP | Settings | Studio или frontend-клиент | Подтверждено MVP | [4.4](#44-пользовательские-предпочтения) |
| `SettingsCatalogSnapshot` | DTO/model | Settings contracts | Configuration и Settings runtime | Подтверждено MVP | [5.1](#51-каталог-настроек) |
| `SettingsPageResponse` | DTO | Settings contracts | Studio | Подтверждено MVP | [5.2](#52-ответ-страницы-настроек) |
| `ISettingsCatalogManifest` | C# extension contract | Settings contracts | Modules | Подтверждено MVP | [6.1](#61-регистрация-каталога) |
| `IModuleSettingsAccessor` | C# runtime port | Settings | Потребители runtime-контракта | Подтверждено MVP | [6.2](#62-чтение-настройки-модулем) |
| `ISettingsPermissionAuthorizer` | C# host adapter | Settings | Host composition | Подтверждено MVP | [6.3](#63-адаптер-авторизации) |
| `IAuditHistoryWriter` | C# external port | Audit History | Settings | Подтверждено MVP | [6.4](#64-передача-аудита) |

## 4. HTTP-контракты

### 4.1. Получение страницы настроек

#### Назначение и владелец

Операция возвращает для выбранного scope дерево навигации, definitions,
локальное и унаследованное значения, effective value, права редактирования и
audit query descriptor. Владелец операции - Settings.

#### Маршрут и права

| Метод и маршрут | Запрос | Ответ | Права | Идемпотентность | Конкурентность |
| --- | --- | --- | --- | --- | --- |
| `POST /api/platform/settings/page` | `GetSettingsPageRequest` | `SettingsPageResponse` | `Permission:Settings.Value.View`; scope access guard | Да, чтение | Не применимо |

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `scopeId` | `Guid` | Нет | Обязательно | Не пустой GUID | Выбранная область действия Configuration | Доступ проверяется отдельно | [request][page-request] |
| `languageCode` | `string` | Да | `null` | Код языка, поддержанный resolver | Предпочтительный язык текстов каталога | При отсутствии используется fallback | [request][page-request] |

#### Ответ

`SettingsPageResponse` описан в [5.2](#52-ответ-страницы-настроек). В ответе
`effectiveSettingsCatalogFingerprint` показывает версию каталога, а
`effectiveValue` показывает значение, применяемое для выбранного scope.

#### Ошибки

| Код или тип ошибки | Условие | HTTP или transport result | Повторить запрос | Ответственный |
| --- | --- | --- | --- | --- |
| `SETTINGS_SCOPE_ACCESS_FORBIDDEN` | Scope недоступен вызывающему пользователю | Ошибка фильтра исключений | Нет без изменения доступа | Tenant/Security / host |
| `ScopeNotFound` | Scope отсутствует в Configuration | Ошибка операции | Нет до исправления scope | Configuration |
| `PublishedVersionNotFound` | Для scope chain нет published version | Ошибка разрешения каталога | После публикации version | Configuration |
| `SettingsCatalogSnapshotMissing` | В baseline version нет snapshot | Ошибка разрешения каталога | После публикации baseline с каталогом | Configuration / Settings |

### 4.2. Изменение runtime-значения

#### Назначение и владелец

Операция создаёт или обновляет локальное переопределение для пары
`ModuleCode + SettingCode`, если definition и выбранный scope разрешают такое
изменение.

#### Маршрут и права

| Метод и маршрут | Запрос | Ответ | Права | Идемпотентность | Конкурентность |
| --- | --- | --- | --- | --- | --- |
| `PUT /api/platform/settings/runtime-values` | `UpdateRuntimeSettingValueRequest` | `RuntimeSettingValueResponse` | `Permission:Settings.Value.Edit` и scope access guard | Повторный PUT заменяет значение | Ожидаемый token во входном запросе отсутствует |

#### Запрос

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `scopeId` | `Guid` | Нет | Обязательно | Не пустой GUID | Область действия локального переопределения | Тип scope проверяется по definition | [request][update-request] |
| `moduleCode` | `string` | Нет | Обязательно | Непустая строка | Код модуля каталога | Нормализуется через trim; сравнение ordinal | [request][update-request] |
| `settingCode` | `string` | Нет | Обязательно | Непустая строка | Код настройки | Нормализуется через trim; вместе с `ModuleCode` образует identity | [request][update-request] |
| `valueJson` | `string` | Нет | Обязательно | JSON нужного `SettingValueType` | Новое локальное значение | Проверяются тип и поддержанные правила проверки | [request][update-request] |

#### Ответ

Ответ содержит `ScopeId`, identity настройки, `EffectiveValue`, необязательный
`ValueId`, `ConcurrencyToken` и `ValueTypeSnapshot`. После сохранения ответ
пересчитывается из runtime snapshot.

### 4.3. Сброс runtime-значения

| Метод и маршрут | Запрос | Ответ | Права | Идемпотентность | Конкурентность |
| --- | --- | --- | --- | --- | --- |
| `POST /api/platform/settings/runtime-values/reset` | `ResetRuntimeSettingValueRequest` | `RuntimeSettingValueResponse` | `Permission:Settings.Value.Reset` и scope access guard | Да: отсутствие local row допустимо | Expected token в request отсутствует |

Сброс удаляет локальное значение. Последующий effective value получается из
ближайшего активного ancestor или default.

### 4.4. Пользовательские предпочтения

| Метод и маршрут | Запрос | Ответ | Права | Идемпотентность | Конкурентность |
| --- | --- | --- | --- | --- | --- |
| `GET /api/platform/settings/user-preferences` | query `surfaceCode` | `UserPreferencesResponse` | Текущий tenant/user context | Да | Не применимо |
| `PUT /api/platform/settings/user-preferences` | `UpsertUserPreferenceRequest` | `UserPreferenceResponse` | Текущий tenant/user context | Повторный PUT заменяет значение | Expected token в request отсутствует |
| `POST /api/platform/settings/user-preferences/reset` | `ResetUserPreferenceRequest` | `204 No Content` | Текущий tenant/user context | Да | Не применимо |

Preferences требуют корректного `ITenantContext`; отдельный permission policy в
controller для этих трёх операций не задан. Это ограничение текущего MVP, а не
обещание общей модели доступа к любым пользовательским данным.

## 5. Общие типы и DTO

### 5.1. Каталог настроек

`SettingsCatalogSnapshot` содержит `SchemaVersion`, `Modules` и
`CatalogFingerprint`. Идентичность одной настройки определяется парой
`ModuleCode + SettingCode`; section и group задают расположение в каталоге.

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `schemaVersion` | `string` | Нет | Обязательно | Текущий builder создаёт `"1"` | Версия сериализованной формы snapshot | Изменение требует совместимости | [snapshot][snapshot] |
| `modules[]` | `SettingModuleDefinition[]` | Нет | Обязательно | Отсортированный каталог | Модули, публикующие settings | Коды уникальны в пределах snapshot builder | [snapshot][snapshot] |
| `catalogFingerprint` | `string` | Нет | Обязательно | SHA-256 hex lowercase | Идентифицирует форму каталога для cache | Вычисляется builder, не вводится пользователем | [builder][builder] |
| `SettingDefinition.valueType` | `SettingValueType` | Нет | Обязательно | `Boolean`, `String`, `Int32`, `Decimal`, `Enum`, `Json` | Форма JSON value | Runtime проверяет соответствие | [enum][value-type] |
| `SettingDefinition.overridePolicy` | `SettingOverridePolicy` | Нет | `CorporateTenantSite` по умолчанию | 5 enum values | Разрешённые scope для local override | Реальное решение проверяет scope type | [enum][override-policy] |
| `SettingDefinition.applyPolicy` | `SettingApplyPolicy` | Нет | `Immediate` по умолчанию | `Immediate`, `RequiresRestart`, `RequiresReload` | Заявленный режим применения | Restart/reload механизм не реализован | [enum][apply-policy] |
| `SettingDefinition.lifecycleStatus` | `SettingLifecycleStatus` | Нет | `Active` по умолчанию | `Active`, `Deprecated`, `Disabled`, `Removed` | Жизненный цикл definition | Disabled/Removed исключаются из обычного runtime read | [enum][lifecycle] |

Вложенные `SettingModuleDefinition`, `SettingSectionDefinition`,
`SettingGroupDefinition`, `SettingDefinition` и `SettingValidationDefinition`
определены в [snapshot contract][snapshot]. Их полный C#-каталог не нужен для
потребителя: существенные поля и ограничения приведены выше.

### 5.2. Ответ страницы настроек

`SettingsPageResponse` - серверная проекция для Studio, а не persistence-модель.
Она включает `Navigation`, `Settings` и `AuditQueryDescriptor`.

| Тип | Существенное содержимое | Источник |
| --- | --- | --- |
| `SettingsNavigationNodeResponse` | `NodeType`, `Code`, `ParentCode`, localized title/description, order и lifecycle | [page response][page-response] |
| `SettingsDefinitionResponse` | Identity, localized metadata, default, local/inherited/effective values, policies, validation, options, editability и audit descriptor | [page response][page-response] |
| `EffectiveRuntimeSettingValueResponse` | JSON value, type, source, source scope и `IsLocalOverride` | [page response][page-response] |
| `SettingsCommandErrorResponse` | `ErrorCode` и `ErrorMessage` | [page response][page-response] |
| `SettingsAuditQueryDescriptorResponse` | `EntityType`, scope и filters для Audit History query | [page response][page-response] |

## 6. C#-контракты и точки расширения

### 6.1. Регистрация каталога

| Метод или member | Параметры | Результат | Предусловия | Исключения | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- | --- |
| `ISettingsCatalogManifest.Build` | `SettingsCatalogBuilder builder` | `void` | Manifest зарегистрирован в DI | Ошибка builder validation | Добавляет definitions в snapshot builder | Нет |
| `SettingsCatalogBuilder.BuildSnapshot` | Нет | `SettingsCatalogSnapshot` | Definitions имеют metadata и valid defaults | `InvalidOperationException` при duplicate key/invalid default/rules | Вычисляет fingerprint | Нет |

Manifest задаёт metadata, default, validation, override policy, apply policy и
lifecycle. Конкретный модуль владеет смыслом своих settings и должен ссылаться на
`ModuleSettingKey<T>`, а не повторять identity строками.

### 6.2. Чтение настройки модулем

| Метод или member | Параметры | Результат | Предусловия | Исключения | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- | --- |
| `IModuleSettingsAccessor.GetSnapshotAsync` | `scopeId`, `CancellationToken` | `SettingsRuntimeSnapshot` | Scope и effective catalog доступны | `SettingsRuntimeSnapshotUnavailableException` или invalid scope | Может прочитать DB и заполнить memory cache | Нет |
| `IModuleSettingsAccessor.GetAsync<T>` | `scopeId`, `ModuleSettingKey<T>`, `CancellationToken` | `Task<T>` | Snapshot содержит key и совпадающий type | `InvalidOperationException` при отсутствии/type mismatch/deserialize failure | Может вызвать snapshot provider | Нет |
| `IModuleSettingsAccessor.Get<T>` | `SettingsRuntimeSnapshot`, `ModuleSettingKey<T>` | `T` | Snapshot прогрет и type совпадает | `InvalidOperationException` | Только чтение словаря и JSON conversion | Нет |

`Get` не должен читать БД или manifest. Для цикла потребитель получает snapshot
один раз и использует синхронный overload.

### 6.3. Адаптер авторизации

| Метод или member | Параметры | Результат | Предусловия | Исключения | Побочные эффекты | Транзакция |
| --- | --- | --- | --- | --- | --- | --- |
| `ISettingsPermissionAuthorizer.TryAuthorizeAsync` | `permissionCode`, `CancellationToken` | `Task<bool>` | Host зарегистрировал реализацию или используется fallback | Реализация host может вернуть отказ | Host adapter может зарегистрировать granted permission в request context | Нет |

Settings не реализует policy decision. В основном API host подключает
`SettingsPermissionAuthorizer`, который делегирует решение Tenant/Security.

### 6.4. Передача аудита

Settings использует `IAuditHistoryWriter.WriteAsync` после изменения или сброса
runtime override. Canonical audit storage и формат запроса истории принадлежат
Audit History; Settings передаёт действие, actor, correlation и details своей
операции.

## 7. Контракты событий

В текущем коде Settings не публикует собственные integration events и не
принимает event handlers. Изменения runtime values передаются в Audit History
синхронным application port. Новые события не выводятся из наличия audit
записи.

## 8. Ошибки и отказоустойчивость

| Код или тип ошибки | Условие | HTTP или transport result | Повторить запрос | Ответственный |
| --- | --- | --- | --- | --- |
| `ScopeRequired` | Пустой `scopeId` | Ошибка команды | Нет до исправления запроса | Вызывающий клиент |
| `SettingNotFound` | Нет definition в effective catalog | Ошибка команды | Нет до публикации каталога | Configuration / module manifest |
| `OverrideScopeNotAllowed` | Policy не разрешает выбранный scope | Ошибка команды | Нет без другого scope | Settings / manifest owner |
| `SettingReadOnlyByLifecycle` | Disabled, Removed или deprecated без разрешения редактирования | Ошибка команды | Нет без изменения lifecycle | Manifest owner |
| `InvalidJson` | `valueJson` не является JSON | Ошибка команды | Нет до исправления формы | Вызывающий клиент |
| `ValueTypeMismatch` | JSON не соответствует `SettingValueType` | Ошибка команды | Нет до исправления значения | Вызывающий клиент |
| `ValueNotAllowed` | Значение отсутствует в allowed values | Ошибка команды | Нет до исправления значения | Manifest owner / client |
| `SettingsRuntimeSnapshotUnavailableException` | Effective catalog не разрешён или snapshot недоступен | Ошибка потребителя runtime-контракта | После восстановления Configuration chain | Configuration / Settings |

Полный runtime-порядок и cache invalidation описаны в
[`04_runtime.md`](04_runtime.md), а эксплуатационные действия - в
[`08_operations.md`](08_operations.md).

## 9. Совместимость и изменение контрактов

| Изменение | Совместимо назад | Потребители | Миграция | Версия или решение |
| --- | --- | --- | --- | --- |
| Добавление необязательного локализованного текста или метаданных ответа | Обычно да | Клиенты Studio | Клиент игнорирует неизвестное поле | Проверить DTO serialization tests |
| Изменение `ModuleCode + SettingCode` | Нет для потребителей runtime-контракта | Manifest, consumers, stored values | Явная миграция значения; автоматической миграции нет | Будущая работа в [`90_traceability.md`](90_traceability.md) |
| Изменение `SettingValueType` | Нет для typed consumers | `ModuleSettingKey<T>`, API и stored JSON | Новый key/explicit migration | Не выполняется автоматически |
| Удаление или переименование enum option | Может сломать stored value и UI | Studio и module runtime | Согласовать migration catalog/value | Не закрыто MVP |
| Изменение `SchemaVersion` snapshot | Требует проверки resolver | Configuration и Settings | Поддержать старую/новую форму или мигрировать snapshot | Configuration/Settings |

## 10. Границы с другими владельцами

| Тема | В Settings | Владелец подробностей |
| --- | --- | --- |
| Configuration version и scope | Чтение effective catalog и scopes | Configuration |
| Роли и permission policy | Использование результата authorizer | Tenant/Security |
| Audit query и retention | Передача записи и descriptor | Audit History |
| Общий UI shell и controls | Использование frontend packages | фронтенд-платформа |
| Смысл module setting | Registration contract и effective value | Module owner |

## 11. Источники и тесты

- [HTTP controller][controller];
- [Settings contracts][settings-contracts];
- [catalog contract tests][catalog-tests];
- [catalog publication tests][publication-tests];
- [API tests][api-tests];
- [storage tests][storage-tests];
- [preference tests][preference-tests].

Проверенная ревизия исходников: `origin/master@3e2037ac37683eef331d70223c6babfc61aa0539`.
[controller]: ../../../src/Platform/DMP.Platform.Settings/Api/Controllers/SettingsController.cs
[settings-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Settings/
[page-request]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Requests/GetSettingsPageRequest.cs
[update-request]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Requests/UpdateRuntimeSettingValueRequest.cs
[snapshot]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Registration/SettingsCatalogSnapshot.cs
[builder]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Registration/SettingsCatalogBuilder.cs
[value-type]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Enums/SettingValueType.cs
[override-policy]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Enums/SettingOverridePolicy.cs
[apply-policy]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Enums/SettingApplyPolicy.cs
[lifecycle]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Enums/SettingLifecycleStatus.cs
[page-response]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Responses/SettingsPageResponse.cs
[catalog-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsCatalogContractTests.cs
[publication-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsCatalogPublicationIntegrationTests.cs
[api-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsApiIntegrationTests.cs
[storage-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsStorageIntegrationTests.cs
[preference-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsUserPreferenceIntegrationTests.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
