---
id: DOC-04-03-91
title: 'Plant Structure v1 — техническая приемка'
type: testing
status: draft
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 03_plant_structure
holder: '@axelprosoft'
created_at: 2026-09-02 13:50
created_by: 'Codex'
updated_at: 2026-09-02 13:50
last_modified_by: 'Codex'
last_reviewed: null
review_status: draft
supersedes: []
source: implementation
---

# Plant Structure v1 — техническая приемка

## 1. Статус

Автоматизированная техническая приемка выполняется адресными архитектурными и SQL Server integration tests. Ручная проверка шести пунктов меню, карточек и lookup выполняется владельцем продукта на целевом frontend runtime и до подтверждения остается открытой.

Полный integration suite не является частью адресной проверки этапа 5 и запускается отдельно в CI или по явному запросу.

## 2. Матрица реализации и проверок

| Область | Реализация | Основные проверки |
| --- | --- | --- |
| Persistence, TPH и tenant ownership | `PlantStructureDbContext`, `InitialPlantStructure`, доменные классы | `PlantStructurePersistenceModelTests`, `PlantStructureMigrationIntegrationTests` |
| Object Runtime и baseline UI | восемь object contracts, datasets и baseline packages | `PlantStructureModuleFoundationArchTests`, `PlantStructureStageTwoIntegrationTests`, `PlantStructureStageThreeIntegrationTests`, `PlantStructureStageFourIntegrationTests` |
| `Company`, `WarehouseBinType` | contracts, validators, list/lookup/details/cards | `PlantStructureStageTwoIntegrationTests` |
| `OrganizationalUnit`, `Subcontractor`, `ProductionUnit` | TPH, abstract lookup, lifecycle и hierarchy | `PlantStructureStageThreeIntegrationTests` |
| `StorageArea`, `WarehouseBin` | contextual lookup, hierarchy и dependency guards | `PlantStructureStageFourIntegrationTests` |
| Параметры места хранения | системная синхронизация и зависимая inline-карточка | `PlantStructureStageFourIntegrationTests` |
| IAM и навигация | `PlantStructureSecurityConfiguration`, permission-protected navigation items | `PlantStructureStageFiveIntegrationTests` |
| Audit History | Object Runtime audit sink, Workflow audit, аудит системной синхронизации | `PlantStructureStageFiveIntegrationTests`, platform `ObjectMutationPipelineIntegrationTests` |
| Диагностика | структурированные события rejected reference, hierarchy и dependency guard | `PlantStructureDiagnostics`, `ObjectHierarchyMutationValidator`, validation cases этапов 2–5 |

## 3. Матрица предметных правил

| Правила | Автоматическая проверка |
| --- | --- |
| `PS-OU-001` — `PS-OU-004` | polymorphism/defaults/synchronization cases в `PlantStructureStageThreeIntegrationTests`; required/discriminator — стандартные validators Object Runtime |
| `PS-PU-001` — `PS-PU-011` | lifecycle, root/child, tenant, cycle, move subtree и read-only hierarchy cases в `PlantStructureStageThreeIntegrationTests` |
| `PS-SC-001` — `PS-SC-003` | optional/external reference/storage synchronization cases в `PlantStructureStageThreeIntegrationTests` |
| `PS-CO-001` — `PS-CO-003` | CRUD, optional/external reference cases в `PlantStructureStageTwoIntegrationTests` |
| `PS-ST-001` — `PS-ST-003` | storage-location lookup, uniqueness and disable guard cases в `PlantStructureStageFourIntegrationTests` |
| `PS-SA-001` — `PS-SA-005` | ownership, uniqueness, archive и warehouse-change guards в `PlantStructureStageFourIntegrationTests` |
| `PS-WBT-001` — `PS-WBT-003` | required, uniqueness и archive guard в `PlantStructureStageTwoIntegrationTests` |
| `PS-WB-001` — `PS-WB-010` | ownership, contextual references, uniqueness, cycle, calculated hierarchy и guards в `PlantStructureStageFourIntegrationTests` |
| `PS-IP-001` — `PS-IP-012` | system lifecycle, defaults, four lookup pairs, flags, segment-only и owner lifecycle cases в `PlantStructureStageFourIntegrationTests` |

`PS-ST-004` зарезервировано документацией и не выполняется в v1.

## 4. Стабильные коды ошибок

- Предметные нарушения возвращают коды правил из `PlantStructureErrors`; для `PS-SA-004` и `PS-SA-005` используются разные коды архивирования и смены места хранения.
- Обязательность полей (`PS-OU-003`, `PS-PU-001`, `PS-SA-002`, `PS-WBT-001`, `PS-WB-002`) обеспечивается единым кодом `RUNTIME_MUTATION_REQUIRED_FIELD` с `FieldCode`.
- Прямое изменение abstract/reference-only объекта (`PS-OU-002`) возвращает стабильный `RUNTIME_OBJECT_OPERATION_NOT_SUPPORTED`.
- Запрет изменения служебных hierarchy-полей (`PS-PU-006`, `PS-PU-010`, `PS-WB-009`) возвращает `RUNTIME_HIERARCHY_SYSTEM_FIELD_READ_ONLY`.
- Self-parent, отсутствующий родитель и перенос в собственное поддерево используют соответственно `RUNTIME_HIERARCHY_SELF_PARENT`, `RUNTIME_HIERARCHY_PARENT_NOT_FOUND`, `RUNTIME_HIERARCHY_PARENT_INSIDE_OWN_SUBTREE`.
- Отказ авторизации отделен HTTP-статусом `403` и reason code Tenant Security; ошибки инфраструктуры не преобразуются в предметные validation-коды.

## 5. Аудит и безопасная диагностика

- CRUD и archive/restore записываются стандартным `ObjectRuntime.*` audit action с tenant, user, object identity и correlation id.
- Workflow-переходы записываются как `Workflow.ExecuteCommand`.
- Автоматическое создание и физическое удаление параметров записываются как `PlantStructure.OrganizationalUnitInventoryParameters.SystemCreated` и `.SystemDeleted` в контексте владельца.
- Перенос hierarchy-узла создает одно событие `ObjectRuntime.Update`; `Parent` и пересчитанные `Main`/`Path` входят в его change set, отдельные шумовые события не создаются.
- Структурированные warnings содержат только категорию, код ошибки, tenant, тип и идентификатор объекта, операцию и код поля. В них не передаются mutation payload, отображаемые значения или данные ссылочного объекта.

## 6. Границы и ограничения v1

- Собственные ValueSet и ValueSetData не регистрируются; используются SystemEnum и переиспользуемый `InventoryLocationControlType` GMD.
- Ручные CRUD endpoints и специальные прикладные операции отсутствуют; используется Object Runtime и Workflow Runtime.
- Пять status-полей физически зарезервированы, но не опубликованы в runtime payload и UI.
- Входящие dependency guards других модулей, `ProductionUnitOperationParameters` и расширенные логистические правила остаются за границей v1 согласно `90_traceability_pr03.md`.

## 7. Ручная функциональная приемка

После пересборки System Baseline проверить под соответствующими ролями:

- порядок и открытие шести пунктов: производственные единицы, субподрядчики, организации, зоны хранения, типы складских ячеек, складские ячейки;
- отсутствие пункта при снятом праве просмотра целевого объекта;
- списки, lookup, карточки `View/Create/Edit`, а для складских ячеек — табличный и древовидный режимы;
- вкладку параметров только у мест хранения и запрет их изменения у опубликованного владельца;
- отсутствие create/delete/archive для параметров хранения и отсутствие `ReturnToDraft` у `PlantStructureStorageResponsible`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-09-02 13:50 +04:00 | Codex | Создание документа | Добавлена матрица автоматизированной и ручной приемки Plant Structure v1 | — |
