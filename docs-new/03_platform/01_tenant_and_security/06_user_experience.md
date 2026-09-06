---
id: DOC-03-01-06
title: 'Пользовательский слой — Tenant Security'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: tenant_and_security
holder: '@axelprosoft'
created_at: 2026-08-25 12:00
created_by: '@VeronikaV2121'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Пользовательский слой — Tenant Security

[api]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/packages/tenant-security-api/src/index.ts
[contracts]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/packages/tenant-security-contracts/src/index.ts
[features]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/features
[role-localization-view]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/features/roles/runtime/admin.role-localizations.runtime.views.ts
[role-localization-provider]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/features/roles/model/roleLocalizationsRuntimeDataProvider.ts
[permission-codes]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/features/common/model/adminAccessPermissionCodes.ts
[server-contracts]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.Contracts/TenantSecurity
[server-controller]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Api/Controllers/TenantSecurityController.cs
[menu]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/ui-runtime/navigation/admin.navigation.runtime.ts
[menu-texts]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/ui-runtime/navigation/admin.navigation.runtime.texts.ts
[routes]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/router/routes.ts
[ui-config]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/app/uiConfig.ts
[app]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/App.tsx
[manifest]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs
[registration]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityCapabilityRegistrationService.cs
[service]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityService.cs
[language]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Hosts/DMP.Platform.Api/Composition/PlatformLanguageCodeResolver.cs
[frontend-platform-backlog]: ../../10_backlog/roadmap/preparation/platform_core_documentation_backlog.md

## 1. Назначение документа

Документ описывает реализованный пользовательский слой Tenant Security: frontend contracts, разделы интерфейса Admin, сценарии, навигацию и локализацию.

Документ не является владельцем общей фронтенд-платформы, приложений Admin/Runtime/Studio, shell, bootstrap/session и shared runtime packages. Общий механизм описан в [UX-документе фронтенд-платформы](../12_frontend_platform/06_user_experience.md).

## 2. Frontend-контракты

`@dmp/tenant-security-contracts` объявляет TypeScript request/response types, list operations и union types статусов/scope; `@dmp/tenant-security-api` преобразует их в HTTP calls. Нормативный server contract остаётся в `DMP.Platform.Contracts/TenantSecurity`. ([frontend contracts][contracts]; [API client][api]; [server contracts][server-contracts])

| Слой | Ответственность | Источник |
| --- | --- | --- |
| `tenant-security-contracts` | Request/response, list filters, governance responses, enums/codes | [contracts][contracts] |
| `tenant-security-api` | Routes, query serialization и чтение ответов | [API client][api] |
| Admin feature providers | Object Runtime metadata, views, actions, data providers и visibility по правам | [features][features] |

Общие runtime artifacts фронта (`@dmp/app-admin`, `@dmp/app-runtime`, `@dmp/app-studio`, `@dmp/runtime-contracts`, `@dmp/runtime-react`, `@dmp/ui`) описаны в [фронтенд-платформе](../12_frontend_platform/00_platform_overview.md). Tenant Security фиксирует только свои контракты, API client и Admin feature providers.

## 3. Разделы интерфейса и сценарии

| Раздел или сценарий | Представление или пакет | API | Право | Статус реализации |
| --- | --- | --- | --- | --- |
| Tenants | Admin list/card/create/edit | Tenant list/get/create/update/status | `TenantSecurity.Tenant.*` | Реализовано. ([features][features]; [api][api]) |
| Users | Admin list/card/create/edit | User list/get/create/update/status/password | `TenantSecurity.User.*` | Реализовано. ([features][features]; [api][api]) |
| Roles | Admin list/card/create/edit | Role list/get/create/update/status | `TenantSecurity.Role.*` | Реализовано. ([features][features]; [api][api]) |
| Permissions | Admin list/lookup | Permission list/get/filter-options | `TenantSecurity.Permission.View` | Реализовано для чтения; `filter-options` заполняет lookup по capability и resource. ([features][features]; [api][api]) |
| Role permissions | Admin actions | List/grant/remove | Permission assign и role permissions | Реализовано. ([features][features]; [api][api]) |
| User-role assignments | Admin providers/actions | List/assign/activate/deactivate/remove | Role assign и user actions | Реализовано. ([features][features]; [api][api]) |
| Authorization/effective permissions/grant policy | Server routes | Authorize, effective permissions, policy upsert | Server governance | Отдельного frontend-представления нет. ([api][api]; [server controller][server-controller]) |

## 4. Навигация

Локальное fallback-menu `Admin.MainMenu` содержит группы «Основное» и «Безопасность». ([menu][menu]; [menu texts][menu-texts])

| Пункт навигации | Родитель | Цель | Право | Порядок | Источник |
| --- | --- | --- | --- | ---: | --- |
| `Admin.Nav.Dashboard` — Панель | `Admin.Nav.Group.Main` | `Admin.Dashboard` → `/admin/dashboard` | Не задано | 10 | [menu][menu]; [routes][routes] |
| `Admin.Nav.Tenants` — Предприятия | `Admin.Nav.Group.Main` | `Admin.Tenants.List` → `/admin/tenants` | `TenantSecurity.Tenant.View` | 20 | [menu][menu]; [routes][routes] |
| `Admin.Nav.Users` — Пользователи | `Admin.Nav.Group.Security` | `Admin.Users.List` → `/admin/users` | `TenantSecurity.User.View` | 30 | [menu][menu]; [routes][routes] |
| `Admin.Nav.Roles` — Роли | `Admin.Nav.Group.Security` | `Admin.Roles.List` → `/admin/roles` | `TenantSecurity.Role.View` | 40 | [menu][menu]; [routes][routes] |

`getAdminNavigationGroups` проверяет permission в `session.permissions`, а route elements дополнительно используют `RequirePermission`. Права и назначения открываются внутри разделов ролей и пользователей; card/create/edit routes не становятся отдельными menu items. ([ui config][ui-config]; [app][app]; [routes][routes]; [permission codes][permission-codes])

При `VITE_ADMIN_REMOTE_NAVIGATION_ENABLED=true` Admin запрашивает effective navigation; при ошибке или выключенном flag использует local fallback menu. Production-источник навигации см. в [трассировке](90_traceability.md): `TS-DEC-08` — Admin navigation. ([app][app]; [menu][menu])

## 5. Локализация

| Ресурс | Ключ | Язык | Резервное значение | Владелец | Покрытие |
| --- | --- | --- | --- | --- | --- |
| Capability/resource/verb | Stable code | Invariant, `ru-RU`, `en-US` из manifest | Exact → neutral → invariant | Owner manifest / registry | Registration и startup validation. ([manifest][manifest]; [registration][registration]) |
| Permission | Stable permission code | Invariant и manifest localizations | Exact → neutral → invariant | Owner manifest / registry | Registration и presentation. ([registration][registration]; [service][service]) |
| Системная роль | Stable role code | Invariant и manifest localizations | Exact → neutral → invariant | Owner manifest / registry | Registration и coverage check. ([manifest][manifest]; [registration][registration]) |
| Пользовательская роль | `RoleLocalization` по `RoleCode` и optional language | Invariant и API-provided locales | Exact → neutral → invariant | Tenant Security caller | Create/update и presentation. ([service][service]) |

Для серверного API Platform language resolver строит fallback candidates из explicit
language, request context и активного языка Configuration; `Tenant.DefaultLanguage`
в этот порядок не входит. Для текстов каталога Tenant Security затем используются
точное совпадение, нейтральный код и invariant. ([language resolver][language]; [service][service])

Для frontend-приложений действует отдельный выбор языка интерфейса: после входа
Admin, Runtime и Studio передают `tenantDefaultLanguage` из `UserSession` в
`resolveI18nLocale`; текущий порядок — tenant default, затем язык браузера.
Это влияет на frontend `ConfigText`, но не меняет порядок разрешения локализаций
серверного каталога и не делает `Tenant.DefaultLanguage` серверным API fallback.
([shared i18n](https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/packages/shared/src/index.ts); [Admin bootstrap](https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/main.tsx); [Runtime bootstrap](https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/runtime/src/main.tsx); [Studio bootstrap](https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/studio/src/main.tsx))

Общий frontend i18n сейчас содержит только словари `ru-RU` и `en-US`.
Endpoint каталога языков и поле `Tenant.DefaultLanguage` допускают другие
активные коды, но `normalizeI18nLocale` для них возвращает frontend-локаль по
умолчанию. Поэтому сохранение, например, `de-DE` не означает, что тексты Admin,
Runtime или Studio уже переведены на немецкий; это отдельный пробел
совместимости, `TS-DEC-11`. ([shared i18n](https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/packages/shared/src/index.ts); [language catalog](https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Hosts/DMP.Platform.Api/Composition/PlatformLanguageCodeResolver.cs); [трассировка](90_traceability.md))

Invariant-представление системной или пользовательской роли хранится с `LanguageCode=null`. После регистрации manifests initializer вызывает `ValidateLocalizationCoverageAsync`: отсутствие перевода standard verb завершает проверку ошибкой, остальные пробелы catalog localization регистрируются как warning. ([service][service]; [registration][registration])

### 5.1. Что именно локализуется в Admin/API

| Объект | Откуда берётся текст | Что получает клиент | Что не следует путать |
| --- | --- | --- | --- |
| Capability, resource, verb | Manifest owner → Tenant Security registry | `CapabilityTitle`, `ResourceTitle`, `VerbTitle` и тексты с fallback | Это не тексты из frontend `ConfigText` |
| Permission | Manifest owner → `PermissionLocalizations` | `Code`, `Description` и названия связанных capability/resource/verb | У permission нет отдельного `Name` в текущем контракте |
| System role | Owner manifest → `RoleLocalizations` | Разрешённые `Name`/`Description`; editing в Admin недоступен | `Role.Code` остаётся техническим идентификатором |
| Custom role | `CreateRoleRequest`/`UpdateRoleRequest` → `RoleLocalizations` | `RoleDetailsResponse.Localizations` и разрешённые поля списка/карточки | Связь локализации задаётся `RoleCode`, не `Role.Id` |
| Tenant, Site, User | Обычные поля `Name`/`DisplayName` | Значение хранится как одно поле | Отдельной таблицы переводов для этих объектов нет |
| Тексты самого Admin UI | Frontend feature/runtime files с `ConfigText` и локальными `ru-RU`/`en-US` значениями | Разрешённый frontend текст; язык выбирается через `resolveI18nLocale` | Они не регистрируются в Tenant Security и не сохраняются в его БД |

В Admin для custom role есть вложенный раздел локализаций: список и форма
используют `RoleLocalization` runtime object, язык после создания строки
блокируется, invariant-строка обязательна. Для system role этот раздел доступен
только для чтения, потому что источник текста — owner manifest. Реализация
находится в [runtime view][role-localization-view] и
[data provider][role-localization-provider]. ([features][features]; [contracts][server-contracts]; [server-controller][server-controller])

## 6. Ограничения

- TypeScript contracts поддерживаются вручную рядом с C# contracts; автоматическая полная parity generation/test не подтверждена. ([contracts][contracts]; [server contracts][server-contracts])
- Общий frontend shell, границы приложений Admin/Runtime/Studio и shared runtime packages не являются владельческой информацией Tenant Security; см. `PCDOC-10` и `PCDOC-11` в [backlog документации Platform Core][frontend-platform-backlog].
- API client не экспортирует отдельные функции для authorize, effective permissions и grant-policy upsert; наличие server route не считается готовым экраном или сценарием в Admin. ([api][api]; [server controller][server-controller])
- Production-источник Admin navigation см. в [трассировке](90_traceability.md): `TS-DEC-08` — Admin navigation. ([app][app]; [menu][menu])
- Catalog validator проверяет platform catalog localizations, а полнота Admin UI texts остаётся в frontend tests. ([registration][registration]; [features][features])

<details>
<summary>Логика вывода</summary>

Раздел или сценарий интерфейса считается реализованным при наличии client call и потребляющего feature flow. Server-only route отмечен ограничением. Аналогично navigation считается двойной, пока code path выбирает remote source и local fallback. ([api][api]; [server controller][server-controller]; [features][features]; [app][app])

</details>

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
