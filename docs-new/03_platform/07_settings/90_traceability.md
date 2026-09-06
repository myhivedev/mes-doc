---
id: DOC-03-07-90
title: 'Трассировка - Settings'
type: traceability
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

# Трассировка - Settings

## 1. Назначение документа

Документ связывает старые материалы, текущий код и тесты с документами-владельцами
Settings. Здесь сохраняются только устойчивые расхождения, ограничения MVP и
маршруты будущей работы; открытые вопросы не являются текущими гарантиями.

## 2. Источники и маршрут содержания

| Источник | Содержание | Маршрут | Статус сведения |
| --- | --- | --- | --- |
| `src/Platform/DMP.Platform.Settings` | Services, API, persistence, cache, security manifest | `00`-`08` документов Settings | Подтверждено MVP |
| `src/Platform/DMP.Platform.Contracts/Settings` | Enums, catalog registration, requests и responses | `03_contracts.md`, `04_runtime.md` | Подтверждено MVP |
| `src/Platform/DMP.Platform.Configuration` | Baseline catalog snapshot и effective resolver | `02_architecture.md`, `04_runtime.md`; подробности остаются Configuration | Владелец другой области |
| Settings integration tests | Catalog, publication, storage, API, security и preferences | `03_contracts.md`, `07_quality.md` | Подтверждено тестами |
| `docs/01 sources/18_system_module_settings_decision.md` | Разделение catalog/value/runtime snapshot и ожидание settings storage | Подтверждённые части перенесены; варианты, не подтверждённые кодом, не перенесены | Частично подтверждено |
| `docs/04 runtime/runtime_settings_authoring_guide.md` | Правила объявления и чтения settings модулями | Подтверждённые сведения покрыты Settings contracts/runtime; неподтверждённые варианты остаются в `SET-DEC-*`. Переходная копия в `docs-new` не нужна. | Маршрут закрыт |
| `docs-new/04_domain_modules/00_common/16_settings.md` | Смысл `Common.TimeAmountFormat` | Остаётся у `00_common`; Settings описывает только механизм | Владелец другой области |
| `docs-new/04_domain_modules/05_document_management/16_settings.md` | Смысл settings Document Management | Остаётся у Document Management; platform mechanism описан здесь | Владелец другой области |

## 3. Расхождения и открытые вопросы

| ID | Тема | Текущее подтверждённое состояние | Расхождение или неопределённость | Влияние на текущую документацию | Статус сведения | Владелец / следующий шаг | Документ для обновления после решения |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `SET-DEC-01` | Применение `RequiresRestart` и `RequiresReload` | Enum и поле `SettingApplyPolicy` входят в catalog snapshot, но отдельный restart/reload executor не найден | Не определено, кто инициирует и подтверждает применение после изменения | Не блокирует текущую документацию: описана только сохранённая policy и отсутствие executor | Будущая доработка | Settings / Platform Operations: определить executor и сигнал применения | `04_runtime.md`, `08_operations.md` |
| `SET-DEC-02` | Custom validation | `CustomValidatorCodes` сохраняются в `SettingValidationDefinition`; текущая проверка Settings выполняет только встроенные правила | Нет подтверждённого registry и вызова custom validators | Не блокирует: Settings прямо описан как ограниченный MVP | Будущая доработка | Settings / владельцы модулей: определить порт validator и порядок вызова | `03_contracts.md`, `04_runtime.md` |
| `SET-DEC-03` | Оптимистическая конкурентность | `ConcurrencyToken` создаётся, обновляется и возвращается в response | Update request не принимает expected token, поэтому lost update не предотвращается контрактом | Не блокирует текущий контракт; ограничение указано явно | Будущая доработка | Settings: решить, нужен ли token в request и какой error code возвращать | `03_contracts.md`, `04_runtime.md` |
| `SET-DEC-04` | Распределённый runtime cache | Snapshot кэшируется в локальном `IMemoryCache`, локальная инвалидация проходит по descendants | Нет общего distributed cache/invalidation contract | Не блокирует single-host MVP | Будущая доработка | Foundation / Platform Operations / Settings: определить общий cache contract | `02_architecture.md`, `07_quality.md`, `08_operations.md` |
| `SET-DEC-05` | Авторизация user preferences | Preferences ограничены `TenantId + UserId` из tenant context; отдельные `[Authorize]` permissions для preference endpoints нет | Не определена отдельная permission policy для preference codes | Не блокирует текущий MVP, который описан как собственные preferences текущего пользователя | Открытый вопрос | Tenant/Security + Settings: решить, нужна ли отдельная policy для preference operations | `03_contracts.md`, `05_security_and_audit.md` |
| `SET-DEC-06` | Источник каталога кроме manifest | Старый материал допускает XML/package/иной импорт; текущий composition root регистрирует `ISettingsCatalogManifest`, а Configuration сохраняет его snapshot | XML/package importer не подтверждён текущим кодом | Не блокирует: нормативные документы описывают только MVP на основе manifest | Расхождение кода и источника | Settings / Configuration: при появлении importer определить его владельца и маршрут | `01_scope.md`, `02_architecture.md`, `04_runtime.md` |
| `SET-DEC-07` | Миграция ключа настройки | Runtime identity - `ModuleCode + SettingCode`; старые rows автоматически не переносятся на новый key | Нет общего migration mechanism | Не блокирует: ограничение совместимости уже описано | Будущая доработка | Владелец модуля и Settings: описывать migration при каждом переименовании key | `03_contracts.md`, `04_runtime.md` |

## 4. Переданные другим владельцам темы

| Тема | Владелец | Почему не входит в Settings |
| --- | --- | --- |
| Configuration scopes, versions и effective chain publication | Configuration | Settings только читает эти данные |
| Значение конкретной module setting | Регистрирующий модуль | Settings не знает предметный смысл и влияние значения |
| Roles, grants и permission assignment policy | Tenant/Security | Settings регистрирует capability и использует adapter |
| Audit schema, query и retention | Audit History | Settings только пишет audit record и возвращает descriptor |
| Общие UI shell, route host и controls | фронтенд-платформа | Settings feature использует эти компоненты |
| Поведение потребителя runtime | Platform Runtime / Domain Modules | Потребитель решает, когда читать значение и как применять его |

## 5. Что не переносится

В нормативные документы не перенесены варианты хранения snapshot из старого
решения, если они уже заменены фактическими полями `ConfigurationVersion`, а
также неподтверждённые XML/package import и обещание автоматического
restart/reload. Они сохранены здесь как контекст расхождения и будущего маршрута.

## 6. Источники кода и тестов

- [Settings project][settings-project];
- [Settings contracts][settings-contracts];
- [Settings tests][settings-tests];
- [Configuration publication][configuration-publication];
- [host composition][composition].

Проверенная ревизия исходников: `origin/master@3e2037ac37683eef331d70223c6babfc61aa0539`.
[settings-project]: ../../../src/Platform/DMP.Platform.Settings/
[settings-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Settings/
[settings-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/
[configuration-publication]: ../../../src/Platform/DMP.Platform.Configuration/Application/Services/Core/ConfigurationPublicationService.cs
[composition]: ../../../src/Hosts/DMP.Platform.Api/Composition/PlatformApiCompositionExtensions.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
