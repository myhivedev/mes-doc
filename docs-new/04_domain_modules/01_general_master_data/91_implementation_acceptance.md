---
id: DOC-04-01-91
title: 'GMD v1 — техническая приемка'
type: testing
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 01_general_master_data
holder: '@axelprosoft'
created_at: 2026-08-17 12:35
created_by: '@A-Zhigalin'
updated_at: 2026-08-20 23:38
last_modified_by: '@A-Zhigalin'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# GMD v1 — техническая приемка

## Статус

Автоматизированная техническая приемка выполняется адресными тестами. Ручная проверка CRUD на целевом frontend runtime остается открытой и фиксируется в GMD-17/GMD-22.

Platform Numbering, машиночитаемая идентификация и Integration Foundation отложены до появления соответствующих платформенных модулей. Они не входят в текущую техническую приемку.

## Матрица реализации

| Область | Реализация | Основные проверки |
| --- | --- | --- |
| Persistence и tenant ownership | `GeneralMasterDataDbContext`, единая начальная миграция | `GeneralMasterDataModuleFoundationArchTests`, runtime validation tests |
| Доменные инварианты справочников | классы `Objects/*/Domain`, validators | `GeneralMasterData*DomainInvariantsArchTests`, `GeneralMasterDataRuntimeValidationArchTests` |
| Матрица и алгоритмы `UnitOfOperation` | `UnitOfOperation`, `UnitOfOperationService` | `GeneralMasterDataUnitOfOperationDomainInvariantsArchTests` |
| Effective parameter schema и typed values | `NomenclatureParameterSchemaService`, value validators | `GeneralMasterDataRuntimeValidationArchTests`, `GeneralMasterDataValueSetsBootstrapIntegrationTests` |
| Object Runtime и коллекции | object contracts и generated mutation writer | `GeneralMasterDataModuleFoundationArchTests`, целевые тесты `ApplicationRuntimeIntegrationTests` |
| Workflow, archive/restore | GMD lifecycle configuration и Platform Workflow | `GeneralMasterDataLifecycleGuardTests`, lifecycle integration test |
| Permissions и ownership | GMD capability registration и security configuration | security/permission integration tests |
| Audit и диагностика | Object Runtime Audit History, workflow audit, GMD system-handler audit, structured warnings | audit sink test и адресные GMD integration tests |
| UI, navigation, localization | GMD baseline packages | `GeneralMasterDataModuleFoundationArchTests`, bootstrap integration test |
| Внешние ссылки | `INomenclatureExternalReferenceAdapter` и fallback validator | `GeneralMasterDataRuntimeValidationArchTests` |

## Ограничения текущей приемки

- Ручная проверка пользовательского CRUD на реальном frontend runtime не выполнена.
- Конкретные adapters внешних ссылок появятся вместе с модулями-владельцами; без адаптера mutation отклоняет новую ссылку.
- Полный integration suite не запускался: во время разработки используются только относящиеся к изменениям тесты; полный прогон выполняется отдельно в CI или по явному запросу.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-08-20 23:38 +04:00 | A-Zhigalin (@A-Zhigalin) | YAML-шапка: review_status, status | Утверждение после merge PR #28: Feature/gmd modules | [PR #28](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/28) |
| 0.1 | 2026-08-20 23:33 +04:00 | A-Zhigalin (@A-Zhigalin) | Создание документа | Приведение документации к стандарту оформления и правка кодировки скриптов | [7916a132](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7916a1320070fe74d3ee3a72b0756912c094e766) |
