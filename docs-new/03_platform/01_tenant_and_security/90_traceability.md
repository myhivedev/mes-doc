---
id: DOC-03-01-90
title: 'Трассировка — Tenant Security'
type: traceability
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

# Трассировка — Tenant Security

[project]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity
[contracts]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.Contracts/TenantSecurity
[context]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.Runtime/Context/PlatformRequestContextMiddleware.cs
[frontend]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/features
[frontend-i18n]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/packages/shared/src/index.ts
[navigation]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/apps/admin/src/App.tsx
[tests]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/tests/DMP.Platform.IntegrationTests/FoundationSlice

## 1. Назначение документа

Документ показывает, какие группы исходных ожиданий учтены в Tenant Security, что подтверждено проверенной ревизией MVP, какие расхождения требуют решения и в какой документ переносится принятое решение.

Трассировка не является каталогом source-файлов и не хранит ссылки на старые `docs` или временные копии. Она нужна, чтобы при ревью было видно, почему утверждение попало в целевой документ, а спорный тезис остался открытым решением.

## 2. Источники и требования

| Источник | Требование или тезис | Решение | Документ-владелец | Покрытие |
| --- | --- | --- | --- | --- |
| Platform offer | Tenant, Site, context, identity/access и central security | Перенесены реализованные границы и порядок проверки доступа; целевые утверждения без владельца в коде не объявлены текущей гарантией | [Граница](01_scope.md), [архитектура](02_architecture.md), [безопасность](05_security_and_audit.md) | Частично |
| Role model requirements | Scope принадлежит role/assignment, а не permission | Permission не имеет `ScopeType`; assignment проверяется по `Role.ScopeType` | [Архитектура](02_architecture.md), [исполнение](04_runtime.md) | Частично |
| Roles requirements | Stable permission codes, системные и пользовательские роли, назначения и политика выдачи права | Базовая модель и policy реализованы; неподтверждённый состав ролей не перенесён как решение | [Безопасность](05_security_and_audit.md), [контракты](03_contracts.md) | Частично |
| Role localization requirements | Localizations capabilities, resources, verbs, permissions и roles | Manifest, registry, fallback и Admin presentation подтверждены | [Архитектура](02_architecture.md), [контракты](03_contracts.md), [исполнение](04_runtime.md), [интерфейсная часть](06_user_experience.md) | Реализовано с ограничениями |
| Role UI rules | Fields/actions зависят от permissions и governance | Закреплены фактические Admin providers/actions и server governance | [Интерфейсная часть](06_user_experience.md), [безопасность](05_security_and_audit.md) | Частично |
| Tenant UI rules | Tenant list/card, status actions и permission visibility | Реализованные разделы и сценарии list/card/create/edit/lifecycle подтверждены frontend code | [Интерфейсная часть](06_user_experience.md) | Частично |
| User UI rules | Multi-tenant membership и role assignments | Сценарии назначения ролей реализованы; текущий User принадлежит одному tenant | [Граница](01_scope.md), [архитектура](02_architecture.md), [интерфейсная часть](06_user_experience.md) | Расхождение |

Материалы из commit `36b12c9` по `ObjectType`, `SystemEnum`, Object Runtime hierarchy, enum flags и Boolean defaults не являются источниками Tenant Security. Они относятся к владельцам `02_configuration` и `03_object_runtime` и не переносятся в предметные выводы этой области, чтобы не смешивать границы областей.

## 3. Принятые решения

| Решение | Подтверждённый вариант | Основание | Документ-владелец |
| --- | --- | --- | --- |
| User ownership | Один `User.TenantId` | [project][project] | [Архитектура](02_architecture.md) |
| Assignment scope | Coordinate определяется `Role.ScopeType` | [project][project]; [contracts][contracts] | [Исполнение](04_runtime.md) |
| Catalog ownership | Definition принадлежит owner manifest, registry — Tenant Security | [project][project]; [contracts][contracts] | [Контракты](03_contracts.md) |
| Permission decision | Active identity → assignment → role → permission | [project][project] | [Безопасность](05_security_and_audit.md) |
| Порядок побочных действий | State save предшествует вызовам events/audit | [project][project] | [Исполнение](04_runtime.md) |
| Готовность frontend | Раздел или сценарий интерфейса учитывается только при client call и потребляющем feature | [frontend][frontend] | [Интерфейсная часть](06_user_experience.md) |

## 4. Расхождения и открытые решения

Строки ниже разделяют состояние сведения и влияние темы на текущую
документацию. Расхождение ожиданий с MVP не означает, что решение нужно
принять сейчас: подтверждённое ограничение можно описать в текущем пакете.
Статус `Открытый вопрос` оставлен только там, где будущая политика или
гарантия ещё не определены; это не означает автоматическую блокировку текущего
MVP. Если тема относится только к будущей возможности, используется статус
`Будущая доработка`. Темы, владельцем которых является другая область,
помечены статусом `Владелец другой области` и имеют маршрут к этому владельцу.

| ID | Тема | Ожидание или источник | Текущее подтверждённое состояние | Расхождение или неопределённость | Влияние на текущую документацию | Статус сведения | Владелец / следующий шаг | Документ для обновления после решения |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `TS-DEC-01` | Аутентификация в промышленной среде | Граница доверия, token/provider и trusted ingress для промышленной аутентификации. | Локальная проверка учётных данных и request session для обычного `[Authorize]` есть; контекст runtime формируется из headers, login не выдаёт token/cookie. ([context][context]; [project][project]) | Не определены production authentication/token и trusted ingress. | Будущая доработка | Будущая доработка | Architecture / Security; определить production-контракт до промышленной готовности. | [Контракты](03_contracts.md), [безопасность](05_security_and_audit.md), [эксплуатация](08_operations.md) |
| `TS-DEC-02` | Членство пользователя в tenant | Модель membership для нескольких tenant. | User принадлежит одному tenant через `User.TenantId`. ([project][project]) | Не определено, сохраняется ли один tenant на пользователя или появится membership-модель. | Будущая доработка | Будущая доработка | Tenant Security / domain owners; определить модель при расширении multi-tenant сценариев. | [Граница](01_scope.md), [архитектура](02_architecture.md) |
| `TS-DEC-03` | Иерархия Corporate и Site | Иерархия Corporate/Site. | Есть scope `Corporate`, но нет отдельной Corporate entity. ([contracts][contracts]; [project][project]) | Ожидаемая иерархия не совпадает с моделью MVP; связь Corporate и Site не определена. | Расхождение кода и источника | Открытый вопрос | Architecture / Tenant Security; решить необходимость отдельной Corporate entity до расширения иерархии. | [Граница](01_scope.md), [архитектура](02_architecture.md) |
| `TS-DEC-04` | Покрытие маршрутов политикой доступа | Единая policy на каждом защищённом endpoint. | Default session policy и dynamic permission policy реализованы, но часть routes опирается на service governance или не имеет route policy. ([project][project]) | Текущая политика не покрывает все ожидаемые endpoint одинаково. | Расхождение кода и источника | Подтверждено, но ограничено | API / Security; описать текущую область покрытия и определить единый production-подход до расширения API. | [Контракты](03_contracts.md), [безопасность](05_security_and_audit.md) |
| `TS-DEC-05` | Начальные учётные данные | Политика промышленной среды для defaults и demo data. | Defaults существуют в configuration/service path. ([project][project]) | Не определена промышленная политика bootstrap credentials, demo data и brute-force controls. | Будущая доработка | Будущая доработка | Security / Operations; определить политику до промышленного запуска. | [Контракты](03_contracts.md), [безопасность](05_security_and_audit.md), [эксплуатация](08_operations.md) |
| `TS-DEC-06` | Транзакционность аудита и outbox | Атомарность audit/outbox и гарантия доставки событий. | Вызовы аудита и событий выполняются после сохранения состояния без общей transaction. ([project][project]) | Схема события, покрытие аудита, outbox и replay не имеют единой межобластной гарантии. | Передано другому владельцу | Владелец другой области | Integration Events / Audit History; определить общий контракт доставки и затем синхронизировать Tenant Security. | [Исполнение](04_runtime.md), [безопасность](05_security_and_audit.md), [качество](07_quality.md), [эксплуатация](08_operations.md) |
| `TS-DEC-07` | Семантика области сайта | Единая семантика caller/site scope и ссылочная целостность назначения site. | `UserRoleAssignment` хранит optional `TenantId`/`SiteId`; domain проверяет форму coordinate, service проверяет tenant и user ownership, но проверка существования `SiteId` и его принадлежности tenant в assignment command не подтверждена. ([project][project]) | Гарантия принадлежности `SiteId` tenant и единая семантика caller/site scope не подтверждены. | Расхождение кода и источника | Открытый вопрос | Tenant Security; зафиксировать текущую ограниченную гарантию и решить ссылочную целостность до заявления полной гарантии. | [Архитектура](02_architecture.md), [Исполнение](04_runtime.md), [безопасность](05_security_and_audit.md) |
| `TS-DEC-08` | Навигация Admin | Однозначный источник навигации в промышленной среде. | Admin выбирает remote source по flag и имеет local fallback. ([navigation][navigation]) | Источник навигации Admin является общей политикой фронтенд-платформа и Configuration. | Передано другому владельцу | Владелец другой области | Frontend / Configuration; определить production source и сообщить результат Tenant Security. | [Интерфейсная часть](06_user_experience.md) |
| `TS-DEC-09` | Резервный выбор языка для tenant | `Tenant.DefaultLanguage` может задавать язык представления tenant. | Поле tenant сохраняется и возвращается в responses; frontend Admin/Runtime/Studio использует его после входа, но серверный API resolver использует explicit language, request `Accept-Language` и активный язык Configuration. ([project][project]; [contracts][contracts]; [frontend-i18n][frontend-i18n]) | Frontend и API используют разные fallback-последовательности; общий приоритет языков не определён. | Передано другому владельцу | Владелец другой области | Platform Runtime / Configuration / фронтенд-платформа / Tenant Security; согласовать общий fallback между владельцами. | [Контракты](03_contracts.md), [исполнение](04_runtime.md), [интерфейсная часть](06_user_experience.md) |
| `TS-DEC-10` | Удаление переводов manifest | При удалении перевода из manifest ожидается согласованное состояние registry. | Registration upsert-ит присутствующие переводы, но не удаляет исчезнувшие; для custom role API удаление языковых записей реализовано отдельно. ([project][project]) | Политика reconcile для исчезнувших переводов не определена. | Будущая доработка | Будущая доработка | Tenant Security / Platform areas; определить reconcile, очистку или версионирование при развитии manifest. | [Исполнение](04_runtime.md), [качество](07_quality.md), [эксплуатация](08_operations.md) |
| `TS-DEC-11` | Соответствие языков интерфейса | Активный каталог языков должен соответствовать языкам, которые поддерживают три frontend-приложения. | `Tenant.DefaultLanguage` может ссылаться на активный язык Configuration, но shared frontend i18n содержит только `ru-RU` и `en-US`; другой код сводится к frontend-локали по умолчанию. ([frontend-i18n][frontend-i18n]; [project][project]) | Не определено, ограничивать ли каталог языками frontend, расширять dictionaries или разделять язык API-каталога и интерфейса. | Передано другому владельцу | Владелец другой области | фронтенд-платформа / Configuration / Tenant Security; определить parity и затем обновить зависимые контракты. | [Контракты](03_contracts.md), [интерфейсная часть](06_user_experience.md), [качество](07_quality.md) |

## 5. Маршрут в целевые документы

| Группа исходников | Что подтверждает | Документ-владелец | Состояние переноса |
| --- | --- | --- | --- |
| [Tenant Security project][project] | Components, model, data и dependencies | `02_architecture.md` | Перенесено |
| [C# contracts][contracts] | API, enums и manifest extension | `03_contracts.md` | Перенесено |
| [Service и context][project] | Scenarios, operations, rules, lifecycle и consistency | `04_runtime.md` | Перенесено |
| [Security implementation][project] | Permission decision, governance и audit calls | `05_security_and_audit.md` | Перенесено |
| [Admin frontend][frontend] | Разделы интерфейса, navigation и localization | `06_user_experience.md` | Перенесено |
| [Foundation tests][tests] | Reliability, tests и подтверждённые gaps | `07_quality.md` | Перенесено |
| [Initializer и migrations][project] | Startup, migration и recovery boundary | `08_operations.md` | Перенесено |
| Requirements и offer | Coverage, conflicts и open decisions | `90_traceability.md` | Перенесено |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
