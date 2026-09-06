---
id: DOC-03-07-00
title: 'Обзор платформенной области - Settings'
type: design
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

# Обзор платформенной области - Settings

## 1. Назначение документа

Документ даёт карту платформенной области Settings: каталог определений
настроек, отдельные значения переопределений, быстрый runtime-доступ,
пользовательские предпочтения и экран управления настройками в Studio.

Settings не владеет схемами Configuration и не превращает настройку в поле
конфигурационного артефакта. Снимок каталога формируется при публикации
`SystemBaseline` в Configuration, а Settings использует его для разрешения
значений и формирования API-ответа. ([публикация каталога][configuration-publication];
[резолвер effective каталога][catalog-resolver])

## 2. Роль в Platform Core

Settings решает две связанные, но разные задачи:

- предоставляет модулям типизированный ключ `ModuleSettingKey<T>` и
  `IModuleSettingsAccessor` для чтения эффективных runtime-значений;
- предоставляет API и frontend-представление для просмотра и изменения
  переопределений настроек в выбранном configuration scope;
- хранит персональные `UserPreferenceValue`, которые принадлежат текущему
  пользователю и не являются runtime-настройками модуля.

Общая карта межмодульных взаимодействий находится в
[`07_integration_architecture.md`](../../02_architecture/07_integration_architecture.md).
Этот пакет описывает только локальные границы Settings.

## 3. Граница и владельцы

| Тема | Входит в Settings | Владелец подробностей |
| --- | --- | --- |
| Каталог настроек | `ISettingsCatalogManifest`, `SettingsCatalogBuilder`, `SettingsCatalogSnapshot` и его runtime-использование | Settings; структура версии Configuration хранится у Configuration |
| Значения настройки | Переопределение по scope, effective value, snapshot и cache | Settings |
| Пользовательские предпочтения | Значение для пары `TenantId + UserId + PreferenceCode + SurfaceCode` | Settings; конкретный смысл пользовательского предпочтения определяет его потребитель |
| API | `api/platform/settings` и DTO `DMP.Platform.Contracts.Settings` | Settings |
| Права | Регистрация capability и permission codes Settings; окончательное решение авторизации через Tenant/Security adapter | Settings / Tenant/Security |
| Аудит | Передача изменений runtime-значений в `IAuditHistoryWriter` | Audit History |
| Экран Studio | Локальный feature provider и адаптер полей Settings | Settings / фронтенд-платформа |
| Смысл конкретной настройки | Название, назначение и применение настройки модуля | Модуль, который зарегистрировал `ISettingsCatalogManifest` |

## 4. Ключевые решения

| Решение | Суть | Документ-владелец |
| --- | --- | --- |
| Каталог и значение разделены | `SettingsCatalogSnapshot` описывает структуру, а `RuntimeSettingValue` хранит локальное переопределение отдельно | [`02_architecture.md`](02_architecture.md) |
| Каталог привязан к baseline | Для effective scope каталог читается из снимка `SystemBaseline` version; Settings не строит каталог из текущего незакреплённого manifest | [`04_runtime.md`](04_runtime.md), Configuration |
| Effective value строится по scope chain | Значение по умолчанию переопределяется ближайшим подходящим активным значением от корня к выбранному scope | [`04_runtime.md`](04_runtime.md) |
| API и runtime-чтение разделены | UI использует `SettingsController`, а серверный код использует `IModuleSettingsAccessor` и snapshot | [`03_contracts.md`](03_contracts.md), [`04_runtime.md`](04_runtime.md) |
| Preferences отделены от runtime settings | User preference хранится для текущего пользователя и не участвует в effective runtime snapshot | [`02_architecture.md`](02_architecture.md) |

## 5. Зависимости

| Зависимость | Роль в Settings | Граница |
| --- | --- | --- |
| Configuration | Публикует snapshot каталога, предоставляет scope chain и system enum definitions | Settings не владеет configuration version и schema артефактов |
| Tenant/Security | Проверяет доступ к scope и permission codes Settings | Settings не определяет роли и assignment policy |
| Audit History | Принимает записи о создании, изменении и сбросе runtime override | Settings не владеет форматом хранения истории |
| фронтенд-платформа | Даёт shell, общие UI-компоненты и runtime feature host | Settings владеет данными и локальным поведением экрана |
| Platform Runtime и прикладные модули | Читают значения через `IModuleSettingsAccessor` или gateway host | Конкретный смысл настройки остаётся у потребителя |

## 6. Статус реализации

| Возможность | Назначение | Статус | Основание |
| --- | --- | --- | --- |
| Регистрация каталога | Собрать и проверить иерархию module/section/group/setting | Подтверждено MVP | `SettingsCatalogBuilder`, `ISettingsCatalogManifest`, контрактные тесты |
| Снимок каталога | Закрепить каталог в опубликованной `SystemBaseline` version | Подтверждено MVP | Configuration publication и integration tests |
| Runtime override | Изменить или сбросить значение на разрешённом scope | Подтверждено MVP | `SettingsValueService`, API и storage tests |
| Effective snapshot | Получить default/Corporate/Tenant/Site value и закэшировать результат | Подтверждено MVP | `SettingsRuntimeSnapshotProvider`, `SettingsRuntimeMemoryCache` |
| User preferences | Читать, сохранять и сбрасывать собственные JSON preferences | Подтверждено MVP | `UserPreferenceService` и integration tests |
| Studio Settings | Показать каталог, effective/local/inherited values и выполнить update/reset | Подтверждено MVP | frontend feature provider и API tests |
| Перезапуск или reload по `SettingApplyPolicy` | Автоматически применить специальный режим после изменения | Будущая доработка | В enum есть значения, но отдельный механизм restart/reload не реализован |

## 7. Состав документов

| Документ | Содержание |
| --- | --- |
| `01_scope.md` | Граница Settings и владельцы соседних обязанностей |
| `02_architecture.md` | Техническая модель каталога, значений, snapshot, cache и persistence |
| `03_contracts.md` | HTTP, DTO и C#-контракты Settings |
| `04_runtime.md` | Разрешение каталога и значений, update/reset, cache invalidation и preferences |
| `05_security_and_audit.md` | Scope access, permission manifest и audit hand-off |
| `06_user_experience.md` | Frontend feature provider и сценарий экрана Studio Settings |
| `07_quality.md` | Тестовые свидетельства, ограничения и наблюдаемость |
| `08_operations.md` | DI, database migrations, host adapter и диагностика |
| `90_traceability.md` | Источники, расхождения, ограничения MVP и будущие маршруты |

## Локальные термины

| Русский термин | Техническое имя | Значение | Источник |
| --- | --- | --- | --- |
| Каталог настроек | `SettingsCatalogSnapshot` | Зафиксированное описание структуры настроек модулей | [контракт снимка][catalog-contract] |
| Определение настройки | `SettingDefinition` | Код, тип, default, validation, lifecycle и правила переопределения | [контракт снимка][catalog-contract] |
| Переопределение настройки | `RuntimeSettingValue` | Локальное значение настройки для configuration scope | [entity][runtime-value] |
| Эффективный снимок настроек | `SettingsRuntimeSnapshot` | Набор значений, доступный runtime-коду для выбранного scope | [snapshot provider][snapshot-provider] |
| Пользовательское предпочтение | `UserPreferenceValue` | JSON-значение текущего пользователя для конкретного раздела интерфейса | [preference entity][preference-value] |

Технические имена в таблице не заменяют русское объяснение и приведены для
точного поиска в коде.

## Источники

- [Settings project][settings-project];
- [Settings contracts][settings-contracts];
- [Settings tests][settings-tests].
[settings-project]: ../../../src/Platform/DMP.Platform.Settings/
[settings-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Settings/
[settings-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/
[configuration-publication]: ../../../src/Platform/DMP.Platform.Configuration/Application/Services/Core/ConfigurationPublicationService.cs
[catalog-resolver]: ../../../src/Platform/DMP.Platform.Configuration/Application/Services/Catalog/EffectiveSettingsCatalogResolver.cs
[catalog-contract]: ../../../src/Platform/DMP.Platform.Contracts/Settings/Registration/SettingsCatalogSnapshot.cs
[runtime-value]: ../../../src/Platform/DMP.Platform.Settings/Domain/Entities/RuntimeSettingValue.cs
[snapshot-provider]: ../../../src/Platform/DMP.Platform.Settings/Application/Services/SettingsRuntimeSnapshotProvider.cs
[preference-value]: ../../../src/Platform/DMP.Platform.Settings/Domain/Entities/UserPreferenceValue.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
