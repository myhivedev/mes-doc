---
id: DOC-03-01-07
title: 'Качество — Tenant Security'
type: assurance
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
updated_at: 2026-08-26 23:30
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Качество — Tenant Security

[db]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Persistence/TenantSecurityDbContext.cs
[entities]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Domain/Entities
[service]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityService.cs
[registration]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Application/Services/TenantSecurityCapabilityRegistrationService.cs
[initializer]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Infrastructure/Persistence/TenantSecurityDatabaseInitializer.cs
[di]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/DependencyInjection.cs
[middleware]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.Runtime/Context/PlatformRequestContextMiddleware.cs
[language]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Hosts/DMP.Platform.Api/Composition/PlatformLanguageCodeResolver.cs
[controller]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity/Api/Controllers/TenantSecurityController.cs
[capability]: https://github.com/axelprosoft/DMP.PlatformFoundation/tree/3e2037ac37683eef331d70223c6babfc61aa0539/src/Platform/DMP.Platform.TenantSecurity
[domain-tests]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/tests/DMP.Platform.ArchTests/Domain/TenantSecurityDomainInvariantsArchTests.cs
[contract-tests]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/tests/DMP.Platform.ArchTests/Contracts/TenantSecurityContractStabilityArchTests.cs
[api-tests]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/tests/DMP.Platform.IntegrationTests/FoundationSlice/TenantSecurityIntegrationTests.cs
[catalog-tests]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/tests/DMP.Platform.IntegrationTests/FoundationSlice/TenantSecurityCatalogRegistrationIntegrationTests.cs
[migration-tests]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/tests/DMP.Platform.IntegrationTests/FoundationSlice/TenantSecurityCatalogMigrationIntegrationTests.cs
[test-host]: https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/tests/DMP.Platform.IntegrationTests/Infrastructure/TenantSecurityIntegrationTestWebApplicationFactory.cs

## 1. Назначение документа

Документ фиксирует проверяемые свойства качества и результаты проверок.

## 2. Надёжность

| Отказ | Гарантия | Предел | Восстановление | Проверка |
| --- | --- | --- | --- | --- |
| Дублирование stable code или assignment | Unique indexes отвергают конфликт | Только coordinates из EF model | Исправить input и повторить command | [DbContext][db] |
| Параллельное изменение entity | Concurrency token обнаруживает lost update | HTTP contract не передаёт expected token/ETag | Перечитать state и повторить изменение | [entities][entities]; [DbContext][db] |
| Повторная регистрация manifest | Upsert/reconcile сохраняет ownership и не создаёт дубль | Transaction охватывает один manifest на relational store | Повторить registration после устранения причины | [registration][registration]; [catalog tests][catalog-tests] |
| Исчезновение catalog item | Owned permission/role деактивируется без удаления | Только items того же owner | Повторная публикация code реактивирует запись | [registration][registration] |
| Повторная регистрация локализации | Existing localization обновляется без дубля | Уникальность зависит от объекта и language coordinate | Повторить registration после изменения manifest | [registration][registration]; [catalog tests][catalog-tests] |
| Резервирование языка при чтении | Exact/neutral/invariant дают стабильное presentation | Tenant default language отдельно не участвует в resolver | Проверить активные языки и request `Accept-Language` | [service][service]; [language][language] |
| Frontend language parity | Admin/Runtime/Studio выбирают язык через `tenantDefaultLanguage` и browser fallback | Frontend dictionaries ограничены `ru-RU`/`en-US`, а Configuration catalog может содержать другие активные коды | Нужны отдельные language-parity tests и решение `TS-DEC-11` | [трассировка](90_traceability.md); [shared i18n](https://github.com/axelprosoft/DMP.PlatformFoundation/blob/3e2037ac37683eef331d70223c6babfc61aa0539/src/Frontend/packages/shared/src/index.ts) |
| Удаление перевода из manifest | Текущий registration выполняет upsert, но не удаляет исчезнувшую языковую запись | Поведение stale localization при изменении manifest не закреплено | Нужен отдельный сценарий и решение о retain/remove | [registration][registration]; [трассировка](90_traceability.md) |
| Переход с legacy catalog schema | Expand/reconcile/contract сохраняет проверенные legacy data | Проверен прямой сценарий; production rollback не подтверждён | Повторить initializer по штатному пути | [initializer][initializer]; [migration tests][migration-tests] |
| Сбой event/audit после state commit | Основное state остаётся сохранённым | Атомарность и replay не гарантированы | Диагностика по correlation; replay см. в [трассировке](90_traceability.md): `TS-DEC-06` — audit/outbox | [service][service]; [трассировка](90_traceability.md) |

Catalog registration повторяема. Обычный application command не имеет automatic retry, command idempotency key или recovery queue для побочных действий. ([registration][registration]; [service][service])

## 3. Наблюдаемость

| Сигнал | Источник | Корреляция | Потребитель | Диагностический пробел |
| --- | --- | --- | --- | --- |
| HTTP error и authorization reason | Middleware, filters и decision | Request correlation ID | API caller/operator | Нет capability-specific error metric. ([middleware][middleware]; [controller][controller]) |
| Audit record | Поддержанная command | `AuditRecord.CorrelationId` | Audit History | Покрыты не все mutations и decisions. ([service][service]) |
| Integration event | Поддержанная command | Envelope correlation | Event consumers | Нет локального сигнала replay. ([service][service]) |
| Catalog warning/error | Registration service | Startup log scope | Operator/manifest owner | Dashboard и alert не подтверждены. ([registration][registration]) |
| Initialization failure | Initializer/startup composition | Startup logs | Operator | Отдельный health check не подтверждён. ([initializer][initializer]; [capability][capability]) |

Для request path доступны correlation ID и reason code, для startup path — initializer/registration logs. Security metrics, distributed traces, dashboards, alerts и capability health check в исходниках области не найдены. ([middleware][middleware]; [service][service]; [registration][registration]; [capability][capability])

## 4. Производительность

| Сценарий нагрузки | Предел или SLO | Метод | Результат |
| --- | --- | --- | --- |
| Authorization decision | SLO не задан | Анализ query path; benchmark отсутствует | Несколько последовательных database reads, latency/throughput не измерены. ([service][service]) |
| Effective permissions | SLO и cache policy не заданы | Анализ service и DI | Локальный decision cache не зарегистрирован. ([service][service]; [DI][di]) |
| List endpoints | Default page 50, maximum 500 | Нормализация query в service | Functional coverage есть; load result отсутствует. ([service][service]; [DbContext][db]) |
| Catalog registration | Startup budget не задан | Integration tests без performance criteria | Идемпотентность проверена, performance не измерена. ([catalog tests][catalog-tests]) |

Allowed scopes policy сериализованы и разбираются application layer. Authorization и effective-permissions выполняют database reads без подтверждённого локального cache. ([db][db]; [service][service]; [di][di])

## 5. Проверки и результаты

| Сценарий проверки | Уровень | Источник теста | Результат запуска | Пробел |
| --- | --- | --- | --- | --- |
| Assignment scope, UTC и concurrency invariants | Architecture/domain | [source][domain-tests] | 2026-08-24: 10 Tenant Security architecture/domain/contract tests прошли | Не проверяет HTTP authentication boundary |
| Numeric enum stability | Contract architecture | [source][contract-tests] | 2026-08-24: прошёл в общей группе 10 тестов | Не проверяет полный C#/TypeScript parity |
| Login, authorization, CRUD, governance, audit и events | API integration | [source][api-tests] | Сборка выполнена; 103 integration tests остановлены недоступным SQL Server до scenarios | Нет результата поведения в текущем окружении |
| `session/probe`, login и authorization guard | API integration | [source][api-tests] | В `origin/master` добавлены сценарии проверки default session policy; локальный запуск этого снимка в рамках текущей вычитки не выполнялся | Нужен CI или локальный запуск с integration database |
| Catalog ownership, localization и reconcile | Integration | [source][catalog-tests] | Та же остановка на database connection | Требуется integration database |
| Expand/contract с legacy data | Migration integration | [source][migration-tests] | Та же остановка на database connection | Обратная migration не проверяется |

Результаты относятся к локальному запуску 2026-08-24 на ревизии, указанной в
`source_revision`, с `DOTNET_ROLL_FORWARD=Major`. Новые session-probe тесты
подтверждены наличием в источнике, но не результатом локального запуска в рамках
этой вычитки. Локальный запуск не заменяет CI evidence; блокирующая причина для
integration suite проверялась по test-host configuration и connection failure до
выполнения scenarios. ([test host][test-host])

## 6. Неподтверждённые свойства

| Свойство | Текущий вывод | Владелец решения |
| --- | --- | --- |
| Production-аутентификация и trusted ingress | Не подтверждены тестами текущего пакета; отдельно добавленная локальная session policy этого не заменяет. | [трассировка](90_traceability.md): `TS-DEC-01` — production-аутентификация |
| Route policy coverage | Нужна единая проверка sensitive routes. | [трассировка](90_traceability.md): `TS-DEC-04` — route policy coverage |
| Default credentials и brute-force controls | Не подтверждены как production policy. | [трассировка](90_traceability.md): `TS-DEC-05` — bootstrap credentials |
| Atomic audit/outbox и replay | Не подтверждены общей transaction/outbox boundary. | [трассировка](90_traceability.md): `TS-DEC-06` — audit/outbox |
| Site scope semantics | Требуется решение по `SiteId` и ownership site assignment. | [трассировка](90_traceability.md): `TS-DEC-07` — site scope semantics |
| Frontend language parity | Active Configuration languages и frontend dictionaries могут расходиться. | [трассировка](90_traceability.md): `TS-DEC-11` — frontend language parity |
| SLO, large-catalog performance, health check и DR procedure | Не найдены в исходниках области. | Quality / Operations |

Эти пункты не превращаются в обещания или задачи внутри quality-документа; маршруты решений ведутся в [трассировке](90_traceability.md). ([API tests][api-tests]; [catalog tests][catalog-tests]; [migration tests][migration-tests]; [capability][capability])

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-26 23:30 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | предварително готовые модули ядра и связанные изменения | [ca13b19b](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/ca13b19bd17dd297927c1e66a97f95c29735b971) |
