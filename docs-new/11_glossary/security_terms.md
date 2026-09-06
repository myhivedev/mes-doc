---
id: DOC-11-99-03
title: 'Термины безопасности и Tenant Security DMP'
type: glossary
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: glossary
module: glossary
holder: '@axelprosoft'
created_at: 2026-08-25 12:00
created_by: '@VeronikaV2121'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@2854252e23fc3c49225f6a6531853dcd4a39e28d
---

# Термины безопасности и Tenant Security DMP

## 1. Назначение документа

Документ фиксирует рабочие русские термины, preferred English и технические алиасы для Tenant Security и связанных механизмов безопасности платформенного ядра.

Русские термины берутся из backend security manifest и `ru-RU` текстов Admin frontend, если они совпадают с архитектурным смыслом. UI-подпись не становится архитектурным термином автоматически: если UI использует короткую форму, в этом документе фиксируется её область применения.

## 2. Источники терминов

| Источник | Что подтверждает |
| --- | --- |
| [Tenant Security security catalog manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs) | Русские и английские имена capability, resources, verbs и описаний permissions. |
| [Tenant Security contracts](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/) | Технические имена DTO, enums, request/response types и published contract. |
| [Tenant Security entities](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/) | Технические имена domain entities и отношений. |
| [Admin Tenant texts](../../src/Frontend/apps/admin/src/features/tenants/runtime/admin.tenants.runtime.texts.ts) | `ru-RU` и `en-US` подписи UI для Tenant. |
| [Admin User texts](../../src/Frontend/apps/admin/src/features/users/runtime/admin.users.runtime.feature-texts.ts) | `ru-RU` и `en-US` подписи UI для User. |
| [Admin Role texts](../../src/Frontend/apps/admin/src/features/roles/runtime/admin.roles.runtime.feature-texts.ts) | `ru-RU` и `en-US` подписи UI для Role. |
| [Admin permission view](../../src/Frontend/apps/admin/src/features/permissions/runtime/admin.permissions.runtime.views.ts) | `ru-RU` и `en-US` подписи UI для Permission, Resource, Verb и filters. |
| [Admin assignment texts](../../src/Frontend/apps/admin/src/features/user-role-assignments/runtime/admin.user-role-assignments.runtime.texts.ts) | `ru-RU` и `en-US` подписи UI для role assignments. |

## 3. Термины-кандидаты Tenant Security

| Русский термин | Preferred English | Технический алиас | Статус | Определение | Источник в коде | Нежелательные синонимы |
| --- | --- | --- | --- | --- | --- | --- |
| Безопасность предприятий | Tenant Security | `DMP.Platform.TenantSecurity`, `TenantSecurity` | Кандидат | Платформенная область, которая владеет предприятиями, пользователями, ролями, правами, назначениями и проверкой доступа. | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs) | IAM без уточнения границы, Security module |
| Предприятие | Tenant | `Tenant`, `TenantSecurity.Tenant` | Кандидат | Платформенный tenant и административный объект Tenant Security. В UI подтверждён как «Предприятие». | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs), [frontend texts](../../src/Frontend/apps/admin/src/features/tenants/runtime/admin.tenants.runtime.texts.ts) | Организация, компания, plant |
| Площадка | Site | `Site` | Кандидат | Платформенная площадка внутри tenant, используемая как координата назначения и проверки доступа. | [entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/Site.cs), [assignment texts](../../src/Frontend/apps/admin/src/features/user-role-assignments/runtime/admin.user-role-assignments.runtime.texts.ts) | Plant для платформенного scope |
| Пользователь | User | `User`, `TenantSecurity.User` | Кандидат | Пользователь Tenant Security, принадлежащий одному tenant в текущем MVP. | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs), [entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/User.cs) | Account без связи с моделью |
| Локальные учётные данные | Local credential | `UserLocalCredential` | Кандидат | Локальная запись password hash/salt для проверки пароля пользователя. | [entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/UserLocalCredential.cs) | Session, token |
| Роль | Role | `Role`, `TenantSecurity.Role` | Кандидат | Набор прав с областью действия, категорией, статусом и правилами назначения. | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs), [entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/Role.cs) | Группа доступа |
| Системная роль | System role | `RoleCategory.System`, `Role.ManagedByPlatform`, `CapabilityRoleTemplateRequest` | Кандидат | Роль, созданная и управляемая платформой на основании capability manifest. Такая роль не является пользовательской ролью и не редактируется как custom role. `CapabilityRoleTemplateRequest` — техническое имя входного описания роли в manifest, а не основной архитектурный термин. | [role entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/Role.cs), [role registration](../../src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityCapabilityRegistrationService.cs), [contract](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Policies/CapabilityRoleTemplateRequest.cs) | Role template в архитектурном тексте, если речь о зарегистрированной роли |
| Пользовательская роль | Custom role | `RoleCategory.Custom`, `Role.ManagedByPlatform = false` | Кандидат | Роль, создаваемая и управляемая через пользовательский сценарий Tenant Security, а не владельцем capability manifest. | [role entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/Role.cs), [service](../../src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityService.cs) | Custom без пояснения, роль-шаблон |
| Право | Permission | `Permission`, `PermissionCode`, `TenantSecurity.Permission` | Кандидат | Стабильное право на действие с ресурсом, опубликованное через security catalog. | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs), [entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/Permission.cs) | Разрешение, privilege |
| Ресурс права | Permission resource | `PermissionResource`, resource code | Кандидат | Ресурс, к которому относится право: Tenant, User, Role, Permission или Audit. В UI может называться «Ресурс». | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs), [permission view](../../src/Frontend/apps/admin/src/features/permissions/runtime/admin.permissions.runtime.views.ts) | Объект без уточнения |
| Операция права | Permission verb | `PermissionVerb`, verb code | Кандидат | Операция права: View, Create, Edit, Delete, Activate, Disable, Enable, ScopeSwitch, TenantAssign, TenantRemove или Assign. В UI подтверждена как «Операция». | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs), [permission view](../../src/Frontend/apps/admin/src/features/permissions/runtime/admin.permissions.runtime.views.ts) | Action без контекста |
| Каталог безопасности | Security catalog | `ITenantSecurityCapabilityCatalogManifest`, `CapabilitySecurityCatalogManifestRequest` | Кандидат | Каталог capability, resources, verbs, permissions, roles, policies и localizations, публикуемый владельцами платформенных областей и регистрируемый Tenant Security. | [manifest contract](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Policies/ITenantSecurityCapabilityCatalogManifest.cs), [registration service](../../src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityCapabilityRegistrationService.cs) | Каталог прав, если речь не только о permissions |
| Локализация каталога безопасности | Security catalog localization | `SecurityCapabilityLocalization`, `PermissionResourceLocalization`, `PermissionVerbLocalization`, `PermissionLocalization`, `RoleLocalization` | Кандидат | Текстовая запись для capability, resource, verb, permission или role на конкретном языке; хранится в registry и используется для представления, но не участвует в authorization decision. | [entities](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities), [DbContext](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Persistence/TenantSecurityDbContext.cs) | Перевод права, если речь обо всём каталоге |
| Код языка | Language code | `LanguageCode`, BCP 47 | Кандидат | Нормализованный код языка и культуры, например `ru-RU` или `en-US`, используемый для выбора локализованного текста. | [LocalizedTextTemplate](../../src/Platform/DMP.Platform.Contracts/Common/Localization/LocalizedTextTemplate.cs), [language catalog](../../src/Hosts/DMP.Platform.Api/Composition/PlatformLanguageCodeResolver.cs) | Язык tenant без уточнения |
| Инвариантный текст | Invariant text | `LocalizedTextTemplate.Invariant`, `LanguageCode = null` | Кандидат | Обязательный базовый текст `Name` или необязательный базовый `Description` в manifest; для role и permission invariant-строка хранится с `LanguageCode = null`, а для capability/resource/verb базовый текст также находится в основной записи. | [localized template](../../src/Platform/DMP.Platform.Contracts/Common/Localization/LocalizedTextTemplate.cs), [DbContext](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Persistence/TenantSecurityDbContext.cs) | Default translation |
| Язык предприятия | Tenant default language | `Tenant.DefaultLanguage`, `UserSession.tenantDefaultLanguage` | Кандидат | Настройка языка интерфейса tenant. В текущем MVP её используют frontend-приложения после входа; серверный API resolver не использует её автоматически как fallback. | [Tenant](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/Tenant.cs), [shared i18n](../../src/Frontend/packages/shared/src/index.ts) | Язык по умолчанию для всех API без уточнения |
| Назначение роли | Role assignment | `UserRoleAssignment` | Кандидат | Связь пользователя и роли в координате Global, Corporate, Tenant или Site. В UI подтверждено как «Назначения». | [entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/UserRoleAssignment.cs), [assignment texts](../../src/Frontend/apps/admin/src/features/user-role-assignments/runtime/admin.user-role-assignments.runtime.texts.ts) | Assignment без указания роли |
| Политика выдачи права | Permission grant policy | `PermissionGrantPolicy` | Кандидат | Правило, которое определяет, кто и на каких scope может назначать конкретное permission. | [entity](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/PermissionGrantPolicy.cs), [contract](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Policies/PermissionGrantPolicyRequest.cs) | Grant policy без указания permission |
| Проверка доступа | Authorization decision | `AuthorizeAsync`, `AuthorizeRequest`, `AuthorizeResult` | Кандидат | Решение `ALLOW` или отказ с reason code по пользователю, tenant, optional site, optional role filter и permission code. | [service interface](../../src/Platform/DMP.Platform.TenantSecurity/Application/Abstractions/ITenantSecurityService.cs), [request](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Requests/AuthorizeRequest.cs), [response](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Responses/AuthorizeResult.cs) | Аутентификация, если речь о permission decision |
| Эффективные права | Effective permissions | `EffectivePermissionsResponse` | Кандидат | Список прав, вычисленный для пользователя и контекста. | [response](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Responses/EffectivePermissionsResponse.cs) | Роли пользователя, если возвращаются permissions |
| Контекст запроса | Request context | `PlatformRequestContext` | Кандидат | Координаты запроса, которые Tenant Security использует для проверки доступа и governance. | [PlatformRequestContext](../../src/Platform/DMP.Platform.Runtime/Context/PlatformRequestContext.cs) | HTTP headers как источник доверия |
| Область действия роли | Role scope | `Role.ScopeType`, `ScopeType` | Кандидат | Scope, который определяет допустимую coordinate назначения роли. | [ScopeType](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Enums/ScopeType.cs), [Role](../../src/Platform/DMP.Platform.TenantSecurity/Domain/Entities/Role.cs) | Scope без владельца |
| Управление назначением | Assignment governance | `AssignmentGovernanceMode`, `AssignmentGovernanceResponse` | Кандидат | Правила, ограничивающие назначение ролей пользователям или пользователей ролям. | [enum](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Enums/AssignmentGovernanceMode.cs), [response](../../src/Platform/DMP.Platform.Contracts/TenantSecurity/Responses/AssignmentGovernanceResponse.cs) | Governance без контекста |
| Аудит безопасности | Security audit | `TenantSecurity.Audit`, `IAuditHistoryWriter` | Кандидат | Audit records, которые Tenant Security формирует для части команд; хранение принадлежит Audit History. | [manifest](../../src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Security/TenantSecuritySecurityCatalogManifest.cs), [service](../../src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityService.cs) | Audit как самостоятельное хранилище Tenant Security |

## 4. Термины UI

| UI-подпись `ru-RU` | `en-US` | Где используется | Правило для документации |
| --- | --- | --- | --- |
| Предприятие / Предприятия | Tenant / Tenants | Admin Tenant views и navigation. | В архитектурном тексте писать «предприятие (`Tenant`)» при первом употреблении. |
| Пользователь / Пользователи | User / Users | Admin User views и navigation. | Использовать как основной русский термин. |
| Роль / Роли | Role / Roles | Admin Role views и navigation. | Использовать как основной русский термин. |
| Право / Права | Permission / Permissions | Admin Permission views. | Использовать для конкретного `Permission`; не расширять до всего security catalog. |
| Назначения | Assignments | User-role assignment views. | В архитектурном тексте уточнять «назначение роли (`UserRoleAssignment`)». |
| Область | Scope / Area | Role scope и permission filters. | Уточнять владельца: область действия роли, область права или область платформы. |
| Ресурс | Resource | Permission resource filter/column. | Использовать как «ресурс права», если речь о `PermissionResource`. |
| Операция | Verb | Permission verb filter/column. | Использовать как «операция права», если речь о `PermissionVerb`. |

## 5. Нежелательные формулировки

| Формулировка | Почему не использовать | Чем заменить |
| --- | --- | --- |
| Каталог прав | Сужает `security catalog` до permissions, хотя каталог содержит resources, verbs, roles, policies и localizations. | Каталог безопасности (`security catalog`). |
| Авторизация как synonym для login | Login проверяет credential, а authorization decision проверяет permission. | Вход по локальному паролю или проверка доступа по контексту. |
| Предприятие как `Enterprise` для Tenant Security | В коде Tenant Security технический алиас — `Tenant`; `Enterprise` относится к производственной/прикладной модели. | Предприятие (`Tenant`). |
| Термин без уточнения (`Plant`) | Исторически смешивает предприятие, площадку и производственную единицу. | Площадка (`Site`) для платформенного scope. |
| Assignment без уточнения | Непонятно, речь о роли, пользователе или другой связи. | Назначение роли (`UserRoleAssignment`). |
| Область действия без владельца (`Scope`) | Термин используется в нескольких механизмах. | Область действия роли (`Role.ScopeType`) или конкретный уровень Global/Corporate/Tenant/Site. |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
