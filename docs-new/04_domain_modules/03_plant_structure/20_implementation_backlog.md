---
id: DOC-04-03-20
title: 'Бэклог реализации — 03 Производственная структура v1'
type: appendix
status: draft
version: '0.8'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 03_plant_structure
holder: '@axelprosoft'
created_at: 2026-08-31 22:16
created_by: '@A-Zhigalin'
updated_at: 2026-09-02 14:05
last_modified_by: 'Codex'
last_reviewed: null
review_status: not_started
supersedes: []
source: upstream
---

# Бэклог реализации — 03 Производственная структура v1

## 1. Назначение документа

Документ задает бэклог реализации модуля `03 Plant Structure` на основе:

- `00_module_overview.md`;
- `01_scope.md`;
- `02_domain_model.md`;
- `03_object_runtime_model.md`;
- `04_workflows.md`;
- `05_rules.md`;
- `06_ui_views.md`;
- `10_value_set_data_usage.md`;
- `11_permissions.md`;
- `13_operations.md`;
- `15_navigation_menu.md`;
- `90_traceability_pr03.md`;
- анализа текущей структуры `src/Modules` и реализации production-модуля `DMP.Modules.GeneralMasterData`.

Документ является планом реализации. Нормативная семантика модуля остается в перечисленных дизайн-документах.

Основной принцип последовательности: сначала единым изменением реализуются все классы сущностей, их persistence-конфигурации и начальная миграция БД. После этого каждый справочник доводится до проверяемого вертикального среза в порядке его зависимостей. Реализация более зависимого справочника не должна быть условием проверки CRUD, runtime-контрактов и UI уже реализованного менее зависимого справочника.

## 2. Текущее состояние реализации

На момент подготовки бэклога:

- production-модуль `DMP.Modules.PlantStructure` отсутствует в `src/Modules`;
- доменные классы, DbContext, persistence-конфигурации и миграции ПР03 отсутствуют;
- object contracts, baseline, workflow, права, меню и пользовательские представления ПР03 не реализованы;
- `DMP.Modules.Common` предоставляет базовые классы `CommonObject` и `CommonCatalogObject` и общую политику archive/restore/delete;
- `DMP.Modules.GeneralMasterData` существует и является владельцем внешнего объекта `Contractor` и SystemEnum `InventoryLocationControlType`;
- платформенные доработки abstract/reference-only типов и hierarchy capability ведутся соответственно в `plans/020_object_runtime_remediation_backlog.md` и `plans/026_object_runtime_hierarchy_support_backlog.md`;
- миграция платформенного термина `Plant` в `Site` ведется отдельно в `plans/025_plant_to_site_platform_migration_backlog.md`.

Следствие: ПР03 создается как новый production-модуль без зависимости от demo-модулей. Готовность платформенных возможностей проверяется до начала блокируемого ими вертикального среза; отсутствующая возможность не заменяется локальной копией платформенной логики внутри ПР03.

## 3. Зафиксированные решения и границы

- Все объекты модуля принадлежат предприятию и имеют `TenantId`; зависимые объекты наследуют tenant владельца.
- `OrganizationalUnit`, `ProductionUnit` и `Subcontractor` хранятся в одной таблице `OrganizationalUnit` с дискриминатором `UnitKind`.
- `OrganizationalUnit` является абстрактным reference-only object type; прямое создание его экземпляров запрещено.
- `ProductionUnit` и `Subcontractor` используют общий `OrganizationalUnitLifecycle`; остальные объекты не подключают Workflow.
- `Company`, `StorageArea` и `WarehouseBinType` наследуются от `CommonCatalogObject`; `WarehouseBin` и `OrganizationalUnitInventoryParameters` — от `CommonObject`.
- `OrganizationalUnitInventoryParameters` — зависимый объект 1:0..1. Он создается и физически удаляется системой вместе с изменением `IsInventoryStorageLocation`, не имеет самостоятельных create/delete/archive.
- `ProductionUnit` и `WarehouseBin` используют hierarchy capability Object Runtime; их служебные поля иерархии не редактируются пользователем.
- `Contractor` является необязательной внешней ссылкой GMD для `Company` и `Subcontractor`.
- ПР03 переиспользует `InventoryLocationControlType` из GMD и не регистрирует его повторно.
- Собственные ValueSet в ПР03 v1 не вводятся.
- Пять nullable GUID-колонок будущих ссылок на `InventoryStatus` создаются миграцией, но не публикуются в Object Runtime, baseline и UI v1.
- `PlantParameters`, `ProductionUnitOperationParameters`, ресурсы, рабочие места, движения и остатки запасов не входят в модуль.
- Входящие зависимости других модулей при архивировании и `ReturnToDraft` в v1 не проверяются; ПР03 защищает только собственные объекты и ссылки.
- Собственные прикладные операции и специальные permission-коды модуль не вводит.

## 4. Зависимости реализации

### 4.1. Внешние зависимости

| Зависимость | Что требуется ПР03 | Блокирует |
|---|---|---|
| `00 Common` | Базовые классы, tenant-поля, archive/restore/delete policy, правила уникальности и ссылок | Все сущности и справочники |
| `01 General Master Data` | `Contractor`, SystemEnum `InventoryLocationControlType`, tenant-scoped lookup и reference validation | Полные карточки `Company`, `Subcontractor`, параметры хранения |
| Object Runtime | object contracts, list/lookup/details, collections, handlers, вычисляемое presentation | Все пользовательские вертикали |
| Abstract/reference-only support | Абстрактный `OrganizationalUnit`, единый lookup по двум конкретным типам | Организационные единицы и все ссылки на места хранения |
| Hierarchy capability | `Parent`, root/main, level, path, `HasChildren`, запрет циклов, `TreeList` | `ProductionUnit`, `WarehouseBin` |
| Workflow Runtime | Состояния, переходы, guards, синхронизация с Common archive | `ProductionUnit`, `Subcontractor`, редактирование параметров хранения |
| IAM / Tenant Security | Стандартные object rights, workflow rights и tenant-роли | Приемка UI и runtime API |
| Audit History | Аудит mutations, переходов и системной синхронизации параметров | Финальная приемка |
| Platform `Site` migration | Единый термин `Site / Площадка` | Платформенная интеграция типа `ProductionUnitType.Site`, но не локальный CRUD ПР03 |
| Будущий `InventoryStatus` | Стабильный внешний object contract | Не блокирует v1: физические GUID-колонки остаются скрытыми |

### 4.2. Зависимости между сущностями

| Сущность / вертикаль | Прямые зависимости | Когда можно проверять |
|---|---|---|
| `Company` | Common; опционально внешний `Contractor` | Сразу после общего persistence-этапа и runtime-инфраструктуры |
| `WarehouseBinType` | Common | Сразу после общего persistence-этапа и runtime-инфраструктуры |
| `Subcontractor` | Абстрактный `OrganizationalUnit`; опционально внешний `Contractor`; Workflow | После базового контракта организационной единицы |
| `ProductionUnit` | `OrganizationalUnit`, `Company`, hierarchy capability, Workflow | После `Company` и базового контракта организационной единицы |
| `StorageArea` | `OrganizationalUnit` с признаком места хранения | После хотя бы одного рабочего конкретного типа `OrganizationalUnit` |
| `WarehouseBin` | `OrganizationalUnit`, `WarehouseBinType`, опционально `StorageArea`, hierarchy capability | После типов ячеек и зон хранения |
| `OrganizationalUnitInventoryParameters` | `OrganizationalUnit`, `WarehouseBin`, SystemEnum GMD, Workflow-состояние владельца | Полный пользовательский контур — после складских ячеек; системное создание записи — вместе с организационными единицами |

### 4.3. Обязательное разделение persistence и прикладных вертикалей

До начала реализации первого справочника должны быть завершены:

1. все восемь классов модели и все шесть SystemEnum ПР03;
2. TPH-модель `OrganizationalUnit` / `ProductionUnit` / `Subcontractor`;
3. EF-конфигурации таблиц, FK, индексов, discriminator и зарезервированных status-колонок;
4. единая начальная миграция модуля;
5. проверка создания схемы на пустой БД и соответствия модели миграции.

После этого справочники реализуются вертикальными срезами. Каждый срез включает runtime contract, baseline, list/lookup/details, формы, права, правила и целевые тесты соответствующего объекта. Общая задача на UI в конце не должна откладывать работоспособность ранних справочников.

### 4.4. Результат этапа 0: принятые технические соглашения

Соглашения модуля:

| Элемент | Зафиксированное значение |
|---|---|
| Проект | `src/Modules/DMP.Modules.PlantStructure/DMP.Modules.PlantStructure.csproj` |
| Root namespace | `DMP.Modules.PlantStructure` |
| Module code | `PlantStructure` |
| Module name | `Plant Structure` |
| Module / contracts version первого baseline | `1.0.0` / `1.0.0` |
| SQL schema | `plant_structure` |
| Таблица истории миграций | `plant_structure.__EFMigrationsHistory` |
| Таблицы | Имена физического хранения из `02_domain_model.md`: `OrganizationalUnit`, `Company`, `OrganizationalUnitInventoryParameters`, `StorageArea`, `WarehouseBinType`, `WarehouseBin` |
| Object type codes | Имена классов: `OrganizationalUnit`, `ProductionUnit`, `Subcontractor`, `Company`, `OrganizationalUnitInventoryParameters`, `StorageArea`, `WarehouseBinType`, `WarehouseBin` |
| Dataset codes | `<ObjectType>_ListDataset` и `<ObjectType>_LookupDataset`; details читается стандартным details-path без отдельного dataset code, tree UI использует list/lookup dataset с hierarchy scope |
| View codes | Коды из `06_ui_views.md`: `<ObjectType>_ListView`, `<ObjectType>_LookupView`, `<ObjectType>_DetailView`; отдельный tree-view не вводится |
| Permission codes | `PlantStructure.<ObjectType>.<Verb>`; workflow — `PlantStructure.<ObjectType>.Workflow.<Verb>` |
| Workflow code | `OrganizationalUnitLifecycle` |
| Navigation codes | Коды `PlantStructure.*` из `15_navigation_menu.md` |
| Error-code prefix | `PS-`; предметные validation codes совпадают с кодами правил `PS-OU-*`, `PS-PU-*`, `PS-SC-*`, `PS-CO-*`, `PS-ST-*`, `PS-SA-*`, `PS-WBT-*`, `PS-WB-*`, `PS-IP-*` |

Принятые решения, влияющие на реализацию:

- `Code` обязателен у `OrganizationalUnit`, `Company`, `StorageArea` и `WarehouseBinType`, поскольку эти типы наследуются от `CommonCatalogObject`. Таблицы полей и правила ПР03 приведены в соответствие с нормативным контрактом Common.
- ПР03 получает project reference на Common. Для использования stable codes `Contractor` и CLR-типа `InventoryLocationControlType` допускается направленная compile-time зависимость ПР03 от GMD; обратная зависимость GMD от ПР03 не вводится.
- `Contractor` хранится как `Guid?` без межконтекстного EF FK и navigation property. Object contract ссылается на `GeneralMasterData/Contractor`, lookup использует `Contractor_LookupDataset`, а серверная проверка существования, tenant scope и права чтения выполняется через Object Runtime/IAM, без доступа ПР03 к `GeneralMasterDataDbContext`.
- `InventoryLocationControlType` использует CLR enum и SystemEnum code GMD. ПР03 не создает собственный SystemEnum с тем же кодом.
- `OrganizationalUnit` использует `AbstractReferenceOnly()` и discriminator `UnitKind`; конкретные типы используют `StoredInBaseTable<...>()`. Общий lookup полагается на существующую фильтрацию Object Runtime по праву `View` каждого concrete type.
- `ProductionUnit.Main` связывается с generic hierarchy role `Root`, несмотря на предметное имя поля. Для `WarehouseBin` root-member называется `Root`.
- Для `ProductionUnit` и `WarehouseBin` используются существующие generic hierarchy handlers. Модуль добавляет только предметные validators: допустимый тип корня, принадлежность одному месту хранения и запрет выбора недоступного/архивного родителя.
- `Site` не является полем ПР03 v1. Незавершенная платформенная миграция `Plant → Site` не блокирует локальный CRUD; будущая связь подключается отдельным интеграционным срезом.
- Пять `InventoryStatus`-колонок создаются как nullable `Guid` без FK, runtime member, lookup и validator.

Результат проверки платформенной готовности:

| Capability | Статус для ПР03 | Основание / дальнейшее действие | Блокирует |
|---|---|---|---|
| Common base objects | Готово | `CommonObject` и `CommonCatalogObject`, soft delete и archive targets реализованы | Нет |
| Object Runtime CRUD, datasets, views | Готово | Production GMD использует единый contract/mutation pipeline | Нет |
| Abstract/reference-only TPH | Готово | Реализованы `AbstractReferenceOnly`, `StoredInBaseTable`, discriminator validation, composite lookup и concrete permission filtering | Нет |
| Hierarchy v1 | Готово для требований ПР03 | Реализованы root/parent/level/path, `HasChildren`, roots/children/ancestors/descendants, create/move recalculation, cycle и active-child guards | Нет |
| Расширенный search-within-subtree | Platform gap, не входит в DoD ПР03 v1 | Владелец: `plans/026_object_runtime_hierarchy_support_backlog.md` | Нет; потребуется только будущему расширенному поиску |
| Workflow Runtime | Готово для lifecycle ПР03 | Состояния, команды, guards, permissions, history, audit и archive synchronizer используются GMD и подтверждены целевыми тестами | Нет |
| IAM / Tenant Security | Готово | Стандартные object verbs, module roles и workflow permissions реализованы на production-паттерне GMD | Нет |
| Audit History | Готово для mutations/workflow | Object Runtime audit sink и persistent audit writer присутствуют; специальные события параметров уточняются в `PS-13` | Нет |
| Flags enum | Готово | `plans/031_object_runtime_enum_flags_backlog.md` реализован; требуется для `WIPSegmentStage` | Нет |
| Boolean create defaults | Готово | `plans/030_object_runtime_boolean_create_defaults_backlog.md` реализован; explicit `true` остается для `IsCalculateInventoryStock` | Нет |
| GMD `Contractor` | Готово | Object type, list/lookup/details, права и tenant-scoped runtime contract реализованы | Нет |
| GMD `InventoryLocationControlType` | Готово | SystemEnum `None = 0`, `Enable = 1`, `Mandatory = 2` опубликован GMD | Нет |
| Platform `Site` | Отложено | Владелец: `plans/025_plant_to_site_platform_migration_backlog.md`; поле/связь не входят в v1 | Нет |
| `InventoryStatus` | Отложено | Владелец: будущий модуль производственной логистики; сохраняются только физические GUID-колонки | Нет |

Новых блокирующих platform gaps по результатам этапа 0 не выявлено. Открытые платформенные работы имеют владельцев и находятся за границей ПР03 v1.

### 4.5. Проверки этапа 0

На 2026-09-01 выполнены целевые проверки существующих платформенных контрактов:

| Проверка | Результат |
|---|---|
| Архитектурные тесты foundation и lifecycle GMD | 9/9 пройдено |
| In-memory integration tests generic hierarchy и abstract TPH contracts | 39/39 пройдено |
| In-memory integration tests Workflow Runtime, audit/history и workflow permission | 3/3 пройдено |
| SQL Server integration tests bootstrap GMD и runtime CRUD `Contractor` | 2/2 пройдено под Windows-учетной записью через SSPI |

Полная интеграционная матрица не запускалась: этап 0 проверяет только capabilities, от которых зависит начало ПР03.

## 5. Приоритеты и оценки

Приоритеты:

```text
P0 — блокирует persistence-модель или начало вертикальных срезов
P1 — обязательная предметная функциональность ПР03 v1
P2 — сквозная приемка, аудит и эксплуатационное завершение
```

Относительные оценки:

```text
S  — локальная доработка
M  — несколько связанных компонентов
L  — полный вертикальный срез одного объекта
XL — общий persistence-этап или сквозная платформенная интеграция
```

Оценки относительные и не являются календарным планом.

## 6. Бэклог

### PS-01. Проверить входные решения и готовность платформы

Приоритет: `P0`  
Оценка: `M`

Состав работ:

- [x] подтвердить stable module code, namespace, имена проекта, схемы БД, таблиц и конфигурационных кодов;
- [x] проверить готовность Common, Object Runtime, Workflow Runtime, IAM и Audit History;
- [x] проверить поддержку abstract/reference-only object type поверх TPH;
- [x] проверить hierarchy capability для двух независимых иерархий;
- [x] подтвердить runtime-контракт внешней ссылки `Contractor` и повторное использование `InventoryLocationControlType` из GMD;
- [x] оформить найденные platform gaps отдельными задачами владельцев платформы с указанием блокируемого среза;
- [x] подтвердить, что `InventoryStatus` остается скрытым контрактом v1, а его GUID-колонки создаются без FK.

Критерии готовности:

- нет открытого решения, меняющего классы или физическую схему первого этапа;
- для каждого platform gap указан владелец и блокируемая задача PS;
- отсутствие внешнего модуля не приводит к созданию фиктивной сущности внутри ПР03.

Статус: выполнено 2026-09-01. Результаты и соглашения зафиксированы в разделе 4.4; блокирующих platform gaps нет.

### PS-02. Создать каркас production-модуля PlantStructure

Приоритет: `P0`  
Оценка: `M`

Состав работ:

- [x] создать проект `DMP.Modules.PlantStructure` и зарегистрировать его в solution и production-host;
- [x] добавить stable codes, module registration и baseline provider;
- [x] создать DbContext и design-time factory;
- [x] подготовить проекты модульных и интеграционных тестов;
- [x] зафиксировать соглашения по таблицам, индексам, constraint names и локализации;
- [x] добавить smoke-тест загрузки модуля без demo-зависимостей.

Критерии готовности:

- модуль загружается production-host;
- baseline provider и DbContext обнаруживаются инфраструктурой;
- stable codes проверяются тестом на уникальность;
- каркас не содержит временных доменных сущностей или ручных CRUD endpoints.

Зависимость: `PS-01`.

Статус: выполнено 2026-09-01. Каркас подключен к production-host; 4/4 целевых архитектурных smoke-теста пройдены.

### PS-03. Реализовать все классы сущностей и начальную миграцию БД

Приоритет: `P0`  
Оценка: `XL`

Состав работ:

- [x] реализовать `OrganizationalUnit`, `ProductionUnit`, `Subcontractor`, `Company`, `OrganizationalUnitInventoryParameters`, `StorageArea`, `WarehouseBinType`, `WarehouseBin`;
- [x] реализовать `ProductionUnitType`, `OrganizationalUnitKind`, `WIPSegmentStage`, `WIPActionTransferYieldType`, `WIPActionTransferRejectType`, `WIPActionTransferScrapType`;
- [x] настроить TPH с таблицей `OrganizationalUnit` и discriminator `UnitKind`;
- [x] настроить наследование от Common, tenant ownership, длины строк, required/nullability, значения по умолчанию и delete behavior;
- [x] настроить все внутренние FK, self-reference и отношение `OrganizationalUnit` 1:0..1 `OrganizationalUnitInventoryParameters`;
- [x] хранить `SegmentStageUsage` как числовую битовую маску;
- [x] создать физические hierarchy-поля `Main`/`Root`, `Parent`, `Level`, `Path`;
- [x] создать пять nullable GUID-колонок `InventoryStatusId` и `Transfer*DefaultStatusId` без FK;
- [x] добавить индексы и ограничения уникальности, включая коды в утвержденной области, `StorageArea` по месту хранения и `WarehouseBin.Identification` по месту хранения;
- [x] сформировать одну начальную миграцию со всей схемой ПР03;
- [x] добавить migration/model snapshot и schema-тесты для пустой БД.

Критерии готовности:

- все сущности из `02_domain_model.md` представлены классами и EF-конфигурациями;
- миграция создает полную физическую схему ПР03 на пустой БД;
- повторное применение миграций идемпотентно, pending model changes отсутствуют;
- TPH, FK, индексы, defaults и зарезервированные колонки проверены автоматическими тестами;
- после этой задачи изменение схемы для реализации плановых справочников не требуется.

Зависимость: `PS-02`.

Статус: выполнено 2026-09-01. 3/3 model/schema-теста и 1/1 целевой SQL Server migration-тест пройдены; повторное применение миграции и отсутствие pending model changes подтверждены.

### PS-04. Реализовать общую runtime-инфраструктуру и baseline SystemEnum

Приоритет: `P0`  
Оценка: `L`

Состав работ:

- [x] централизованно заполнять `TenantId` корневого объекта из runtime context и запрещать его изменение payload-запросом;
- [x] наследовать `TenantId` зависимым объектом от владельца;
- [x] реализовать same-tenant и reference validation для внутренних ссылок;
- [x] подключить внешний contract/adapter проверки `Contractor` без прямого доступа к persistence GMD;
- [x] зарегистрировать шесть SystemEnum ПР03 с локализацией, порядком и `IsFlags = true` для `WIPSegmentStage`;
- [x] переиспользовать регистрацию `InventoryLocationControlType` из GMD без дублирования;
- [x] определить общие stable error codes для tenant, reference, hierarchy и dependency violations;
- [x] подготовить шаблон вертикального runtime-среза: object contract, datasets, views, permissions, navigation и тесты.

Критерии готовности:

- cross-tenant mutations и ссылки отклоняются до записи;
- lookup и mutation используют одинаковый tenant scope;
- SystemEnum доступны после bootstrap, а `InventoryLocationControlType` зарегистрирован ровно одним модулем;
- последующие задачи не дублируют общую tenant/reference инфраструктуру.

Зависимость: `PS-03`.

Статус: выполнено 2026-09-01. 4/4 целевых runtime/baseline-теста пройдены; SQL Server bootstrap подтвердил публикацию 6 SystemEnum ПР03 и единственную регистрацию `InventoryLocationControlType` из GMD.

### PS-05. Реализовать справочник организаций

Приоритет: `P1`  
Оценка: `L`

Состав работ:

- [x] реализовать object contract `Company` и правила `PS-CO-*`;
- [x] подключить необязательную внешнюю ссылку `Contractor` с tenant-scoped lookup;
- [x] реализовать presentation, `List`, `Lookup`, `Details`;
- [x] реализовать список и карточку во всех режимах из `06_ui_views.md`;
- [x] опубликовать пункт меню `PlantStructure.Companies`;
- [x] подключить стандартные CRUD-права;
- [x] запретить архивирование организации, используемой активной `ProductionUnit`;
- [x] покрыть CRUD, archive/restore, уникальность и ссылку на контрагента целевыми тестами.

Критерии готовности:

- `Company` полностью работоспособна и проверяема независимо от runtime-реализации производственных единиц;
- пустая ссылка `Contractor` допустима, недоступная или cross-tenant ссылка отклоняется;
- список, lookup и карточка соответствуют проектной документации.

Зависимость: `PS-04`.

Статус: выполнено 2026-09-01. Object Runtime, baseline UI, навигация, IAM и правила `PS-CO-*` реализованы; целевой SQL Server сценарий CRUD/list/lookup/details/archive/restore, уникальности, внешней ссылки и блокирующей зависимости пройден.

### PS-06. Реализовать справочник типов складских ячеек

Приоритет: `P1`  
Оценка: `M`

Состав работ:

- [x] реализовать object contract `WarehouseBinType` и правила `PS-WBT-*`;
- [x] реализовать presentation, `List`, `Lookup`, `Details`;
- [x] реализовать список и карточку во всех режимах из `06_ui_views.md`;
- [x] опубликовать пункт меню `PlantStructure.WarehouseBinTypes`;
- [x] подключить стандартные CRUD-права;
- [x] запретить архивирование типа, используемого активными `WarehouseBin`;
- [x] покрыть CRUD, уникальность кода и archive/restore целевыми тестами.

Критерии готовности:

- справочник полностью работоспособен без реализации складских ячеек в Object Runtime;
- код уникален в пределах tenant согласно Common policy;
- тип доступен как стабильная ссылка для последующего среза `WarehouseBin`.

Зависимость: `PS-04`.

Статус: выполнено 2026-09-01. Object Runtime, baseline UI, навигация, IAM и правила `PS-WBT-*` реализованы; целевой SQL Server сценарий CRUD/list/lookup/details/archive/restore, уникальности и блокирующей зависимости пройден. После расширения стабильного каталога модульных контрактов `ContractsVersion` повышена до `1.1.0`.

### PS-07. Реализовать abstract OrganizationalUnit и справочник субподрядчиков

Приоритет: `P1`  
Оценка: `XL`

Состав работ:

- [x] опубликовать `OrganizationalUnit` как abstract/reference-only тип без create/update/delete;
- [x] реализовать общий lookup организационных единиц с фильтрацией прав по конкретному типу;
- [x] реализовать concrete contract `Subcontractor` и правила `PS-OU-*`, `PS-SC-*`;
- [x] подключить необязательную ссылку `Contractor`;
- [x] реализовать `OrganizationalUnitLifecycle` для `Subcontractor` и синхронизацию Workflow archive с Common;
- [x] запретить изменение рабочих полей вне `Draft`;
- [x] при включении `IsInventoryStorageLocation` системно создавать одну запись `OrganizationalUnitInventoryParameters` с defaults, при допустимом отключении — физически удалять ее;
- [x] на этом этапе публиковать параметры хранения только для чтения либо скрыть вкладку до `PS-11`;
- [x] реализовать presentation, `List`, `Lookup`, `Details`, список, карточку и пункт меню `PlantStructure.Subcontractors`;
- [x] подключить стандартные CRUD- и workflow-права;
- [x] покрыть TPH-discriminator, запрет abstract create, CRUD, lifecycle, contractor reference и синхронизацию параметров тестами.

Критерии готовности:

- `Subcontractor` проходит полный пользовательский сценарий независимо от `ProductionUnit`, зон и ячеек;
- общий `OrganizationalUnit.Lookup` возвращает доступные конкретные объекты и не позволяет создать абстрактный объект;
- место хранения получает ровно одну зависимую запись с корректными начальными значениями;
- published/archived субподрядчик не допускает изменение рабочих полей.

Зависимость: `PS-04`.

Статус: выполнено 2026-09-01. Abstract/reference-only контракт, concrete `Subcontractor`, lifecycle, IAM, baseline UI и навигация реализованы; параметры места хранения создаются и удаляются системно. Целевой SQL Server сценарий CRUD/list/lookup/details, реальных layout-узлов всех режимов карточки, Workflow, внешней ссылки и permission-aware общего lookup пройден.

### PS-08. Реализовать справочник и иерархию производственных единиц

Приоритет: `P1`  
Оценка: `XL`

Состав работ:

- [x] реализовать concrete contract `ProductionUnit` и правила `PS-PU-*`;
- [x] подключить необязательную ссылку `Company`;
- [x] подключить hierarchy capability для `Parent`, `Main`, `Level`, `Path`, виртуального `HasChildren` и `TreeList`;
- [x] реализовать правила корневого `Enterprise`, обязательного родителя для остальных типов, same-tenant и запрета циклов;
- [x] реализовать `OrganizationalUnitLifecycle` для `ProductionUnit`, guards публикации, архивирования и восстановления;
- [x] переиспользовать синхронизацию параметров места хранения из `PS-07`;
- [x] реализовать presentation, `List`, `Lookup`, `Details`, `TreeList`, список, карточку и пункт меню `PlantStructure.ProductionUnits`;
- [x] подключить стандартные CRUD- и workflow-права;
- [x] покрыть несколько корней в tenant, пересчет hierarchy-полей, перемещение ветки, циклы, lifecycle и ссылку `Company` тестами.

Критерии готовности:

- дерево производственных единиц работоспособно на произвольной глубине;
- root/non-root правила и служебные поля обеспечиваются сервером, а не только UI;
- `ProductionUnit` и `Subcontractor` единообразно доступны через `OrganizationalUnit.Lookup`;
- справочник можно функционально принять до реализации зон и складских ячеек.

Зависимости: `PS-05`, `PS-07`, hierarchy capability Object Runtime.

Статус: выполнено 2026-09-01. Concrete contract, server-side hierarchy, lifecycle guards, IAM, baseline `TreeList`, карточка и навигация реализованы. Целевой SQL Server сценарий подтвердил несколько корней, произвольную глубину, перенос поддерева, пересчет `Main/Level/Path`, запрет циклов, read-only вне `Draft`, зависимость при архивировании, ссылку `Company` и переиспользование параметров места хранения. После расширения стабильного каталога контрактов `ContractsVersion` повышена до `1.2.0`.

### PS-09. Реализовать справочник зон хранения

Приоритет: `P1`  
Оценка: `L`

Состав работ:

- [x] реализовать object contract `StorageArea` и правила `PS-SA-*`;
- [x] ограничить `Warehouse` активными доступными `OrganizationalUnit` с `IsInventoryStorageLocation = true`, независимо от состояния Workflow;
- [x] реализовать уникальность `Code` в пределах места хранения;
- [x] запретить смену места хранения и архивирование зоны при наличии активных ячеек;
- [x] реализовать presentation, `List`, `Lookup`, `Details`, список и карточку;
- [x] опубликовать пункт меню `PlantStructure.StorageAreas` и стандартные CRUD-права;
- [x] покрыть CRUD, tenant scope, storage-location filter, уникальность и dependency guards тестами.

Критерии готовности:

- зона создается для места хранения любого конкретного вида организационной единицы;
- обычная организационная единица отсутствует в lookup и отклоняется серверной проверкой;
- справочник полностью проверяем без runtime-реализации `WarehouseBin`.

Зависимости: `PS-07`; `PS-08` требуется для проверки сценария с `ProductionUnit`, но не блокирует сценарий с `Subcontractor`.

Статус: выполнено 2026-09-01, уточнено 2026-09-02. Object Runtime, permission-aware lookup активных мест хранения обоих concrete-типов независимо от состояния Workflow, baseline UI, навигация, IAM, уникальность по месту хранения и dependency guards реализованы. Целевой SQL Server сценарий CRUD/list/lookup/details, layout `View/Create/Edit`, включения Draft-места хранения, фильтрации не-склада, уникальности, переноса и archive/restore покрыт.

### PS-10. Реализовать справочник и иерархию складских ячеек

Приоритет: `P1`  
Оценка: `XL`

Состав работ:

- [x] реализовать object contract `WarehouseBin` и правила `PS-WB-*`;
- [x] ограничить `Warehouse` местами хранения, `StorageArea` и `Parent` — выбранным местом хранения;
- [x] сделать `WarehouseBinType` обязательной ссылкой;
- [x] реализовать уникальность `Identification` в пределах места хранения;
- [x] подключить hierarchy capability для `Parent`, `Root`, `Level`, `Path`, виртуального `HasChildren` и `TreeList`;
- [x] реализовать безопасную смену места хранения, запрет циклов и архивирования родителя с активными детьми;
- [x] не ограничивать использование ячейки наличием дочерних узлов;
- [x] реализовать presentation, `List`, `Lookup`, `Details`, `TreeList`, табличный и древовидный режимы списка и карточку;
- [x] опубликовать пункт меню `PlantStructure.WarehouseBins` и стандартные CRUD-права;
- [x] покрыть зависимые lookup, смену склада, иерархию, уникальность и archive guards тестами.

Критерии готовности:

- ячейки полностью работоспособны для мест хранения обоих конкретных типов;
- зона и родитель всегда относятся к тому же месту хранения;
- hierarchy-поля рассчитываются системой, циклы отклоняются;
- типы ячеек и зоны хранения после этого среза проверяются также как владельцы активных ссылок.

Зависимости: `PS-06`, `PS-09`, полная готовность `PS-07`/`PS-08`, hierarchy capability Object Runtime.

Статус: выполнено 2026-09-01. Object Runtime, контекстные lookup, hierarchy capability, baseline UI с табличным и древовидным режимами, навигация и IAM реализованы. Целевой SQL Server сценарий для обоих concrete-типов мест хранения, layout `View/Create/Edit`, CRUD/list/lookup/details, уникальности, смены склада, переноса поддерева, циклов и archive/restore guards пройден.

### PS-11. Завершить пользовательский контур параметров места хранения

Приоритет: `P1`  
Оценка: `XL`

Состав работ:

- [x] реализовать полный contract `OrganizationalUnitInventoryParameters` и правила `PS-IP-*`;
- [x] публиковать зависимый объект коллекцией 1:0..1 в карточках `ProductionUnit` и `Subcontractor`;
- [x] оставить create/delete/archive только системной синхронизации владельца;
- [x] разрешить изменение параметров только при состоянии владельца `Draft`;
- [x] реализовать defaults, `PickingOrder >= 0`, допустимые стадии, обязательный `InQueue` и согласованность флагов;
- [x] ограничить параметры уровня передела признаком `IsSegmentLevel`;
- [x] реализовать четыре пары зависимых lookup `*Warehouse` / `*WarehouseBin`, очистку ячейки при очистке или замене места хранения и серверную проверку принадлежности;
- [x] не публиковать пять зарезервированных status-полей;
- [x] реализовать вкладку из `06_ui_views.md` и права read/update без самостоятельного пункта меню;
- [x] покрыть системное создание/удаление, defaults, lifecycle owner, flags и все пары зависимых ссылок тестами.

Критерии готовности:

- для места хранения существует ровно одна корректная запись параметров, для обычной организационной единицы — ни одной;
- все четыре пары место хранения/ячейка согласованы на UI и сервере;
- пользователь не может напрямую создать, удалить или архивировать параметры;
- скрытые status-поля остаются физически совместимыми с будущим расширением и недоступны через runtime payload.

Зависимости: `PS-07`, `PS-08`, `PS-10`, Workflow Runtime.

Статус: выполнено 2026-09-01, исправлено 2026-09-02. Полный зависимый `WithOwner`-контракт 1:0..1, системная синхронизация, lifecycle owner, правила `PS-IP-*`, четыре контекстные пары lookup, UI-очистка ячеек, вкладки обеих карточек и IAM read/update реализованы. Целевой SQL Server сценарий подтвердил defaults, системное создание/физическое удаление, агрегатное сохранение, flags и segment-only правила, все четыре серверные проверки принадлежности, запрет смены владельца и прямых create/delete/archive; зарезервированные status-поля не опубликованы. Вкладки владельцев переведены в режим `OpenItemInline`: Runtime встраивает плоский detail-layout зависимой записи и синхронизирует изменения полей с агрегатным черновиком владельца. Вкладка скрывается вместе с `EmbeddedCollection`, если владелец не является местом хранения. Версии baseline карточек `ProductionUnit` и `Subcontractor` — `v4` и `v3`, дочерней карточки — `v3`; проверки контролируют inline-режим и размещение каждого поля во всех режимах.

### PS-12. Реализовать полную матрицу прав, ролей и навигации

Приоритет: `P1`  
Оценка: `L`

Состав работ:

- [x] зарегистрировать стандартные права всех самостоятельных типов и read/update для параметров хранения;
- [x] зарегистрировать workflow rights для `ProductionUnit` и `Subcontractor`;
- [x] реализовать роли `PlantStructureAdmin`, `PlantStructureUser`, `PlantStructureResponsible`, `PlantStructureStorageResponsible`;
- [x] проверить фильтрацию abstract `OrganizationalUnit.Lookup` по правам конкретных типов;
- [x] опубликовать группу меню `PlantStructure` и все шесть пунктов в порядке из `15_navigation_menu.md`;
- [x] скрывать пункт меню без права просмотра целевого объекта;
- [x] проверить, что `PlantStructureStorageResponsible` не получает `ReturnToDraft` и не обходит lifecycle владельца;
- [x] покрыть матрицу положительными и отрицательными authorization-тестами.

Критерии готовности:

- фактическая матрица прав совпадает с `11_permissions.md`;
- зависимый объект не получает лишних create/delete/archive;
- меню и lookup не раскрывают недоступные типы или записи.

Зависимости: `PS-05` — `PS-11`, IAM / Tenant Security.

Статус: выполнено 2026-09-02. Матрица всех четырех ролей проверена на точное совпадение с `11_permissions.md`; зависимый объект не получает create/delete/archive, abstract lookup фильтрует конкретные типы. Все шесть navigation items защищены правом `View` целевого типа и публикуются в утвержденном порядке. Адресный тест подтверждает одновременно скрытие пункта и серверный `403`; для `PlantStructureStorageResponsible` отсутствует workflow execute, а lifecycle параметров остается серверным ограничением владельца.

### PS-13. Подключить аудит, стабильные ошибки и диагностику

Приоритет: `P2`  
Оценка: `L`

Состав работ:

- [x] подключить аудит CRUD, archive/restore и workflow-переходов;
- [x] аудитировать системное создание и физическое удаление `OrganizationalUnitInventoryParameters` в контексте операции владельца;
- [x] аудитировать изменения иерархии и пересчет служебных hierarchy-полей без шумового дублирования событий;
- [x] закрепить стабильные error codes за правилами `PS-OU-*`, `PS-PU-*`, `PS-SC-*`, `PS-CO-*`, `PS-ST-*`, `PS-SA-*`, `PS-WBT-*`, `PS-WB-*`, `PS-IP-*`;
- [x] добавить структурированные логи для rejected references, hierarchy violations и dependency guards;
- [x] проверить отсутствие в логах чувствительных данных и избыточных payload.

Критерии готовности:

- пользовательская и системная mutation прослеживается по tenant, object type, object id и correlation id;
- одинаковые нарушения возвращают одинаковые предметные коды независимо от точки вызова;
- диагностика позволяет отличить validation, authorization и infrastructure failures.

Зависимости: `PS-07` — `PS-12`, Audit History.

Статус: выполнено 2026-09-02. Стандартные Object Runtime и Workflow audit-события дополнены коррелированными событиями системного создания/удаления параметров в контексте владельца. Перенос hierarchy-узла проверен как одно `ObjectRuntime.Update` с `Parent`, `Main` и `Path`. Разделены коды `PS-SA-004`/`PS-SA-005`; декларативные required, polymorphism и hierarchy нарушения используют стабильные platform-коды. Структурированные warnings содержат только идентификаторы, категорию, код, операцию и поле без mutation payload и отображаемых значений.

### PS-14. Завершить тестовую матрицу, приемку и документацию

Приоритет: `P2`  
Оценка: `XL`

Состав работ:

- [x] свести unit, integration, runtime, UI contract и authorization tests в матрицу по объектам и правилам;
- [x] проверить миграцию и bootstrap на чистой SQL Server БД;
- [x] проверить tenant isolation и cross-tenant references для всех ссылок;
- [x] проверить lifecycle, archive/delete guards, обе иерархии и зависимые lookup;
- [ ] выполнить ручную функциональную приемку шести пунктов меню, списков, lookup и карточек;
- [x] проверить трассировку требований ПР03 до кода и тестов;
- [x] актуализировать эксплуатационные инструкции, ограничения v1 и границы с модулями-потребителями;
- [x] подтвердить отсутствие собственных ValueSet, ручных CRUD endpoints и специальных прикладных операций.

Критерии готовности:

- каждое правило из `05_rules.md` имеет автоматическую проверку либо явно обоснованный manual case;
- все виды представлений из `06_ui_views.md` материализуются и проходят приемку;
- `90_traceability_pr03.md` не содержит требования v1 без реализации и теста;
- модуль устанавливается на пустую БД и готов к использованию модулями-потребителями.

Зависимости: `PS-03` — `PS-13`.

Статус: выполняется. Автоматизированная часть завершена и зафиксирована в `91_implementation_acceptance.md`: 16 архитектурных и 12 сквозных SQL Server тестов Plant Structure/схемы навигации прошли. Открыта только ручная функциональная приемка на целевом frontend runtime.

## 7. Рекомендуемая последовательность реализации

```text
PS-01 → PS-02 → PS-03 → PS-04
                          ├→ PS-05 Company ────────────────┐
                          ├→ PS-06 WarehouseBinType ───────┼──────────────┐
                          └→ PS-07 Subcontractor/OU ───────┼→ PS-09 ─────┼→ PS-10
                                      └→ PS-08 ProductionUnit           │
                                                                          └→ PS-11

PS-05..PS-11 → PS-12 → PS-13 → PS-14
```

Строгий линейный порядок вертикальных срезов для последовательной команды:

```text
Company
→ WarehouseBinType
→ Subcontractor и abstract OrganizationalUnit
→ ProductionUnit
→ StorageArea
→ WarehouseBin
→ OrganizationalUnitInventoryParameters
```

`PS-05`, `PS-06` и базовая часть `PS-07` технически могут выполняться параллельно после `PS-04`, но каждый срез должен завершаться собственной проверкой работоспособности. `PS-09` может быть проверен на `Subcontractor`-месте хранения до завершения `ProductionUnit`; для полной приемки обоих типов владельца требуется `PS-08`.

## 8. Предлагаемые этапы поставки

### Этап 0. Решения и platform readiness

```text
PS-01
```

Результат: решения, влияющие на схему, закрыты; platform gaps имеют владельцев.

### Этап 1. Полная persistence-модель

```text
PS-02 → PS-03 → PS-04
```

Результат: production-модуль загружается, все классы и таблицы созданы одной миграцией, SystemEnum и общая tenant/reference инфраструктура готовы. Прикладные справочники еще не считаются реализованными.

### Этап 2. Независимые справочники

```text
PS-05 → PS-06
```

Результат: организации и типы складских ячеек доступны через полноценные CRUD/list/lookup/details/cards и могут быть приняты до организационной и складской структуры.

### Этап 3. Организационная структура

```text
PS-07 → PS-08
```

Результат: доступны субподрядчики, производственные единицы, общий abstract lookup, lifecycle и иерархия производственных единиц.

### Этап 4. Складская детализация

```text
PS-09 → PS-10 → PS-11
```

Результат: доступны зоны, иерархия складских ячеек и полный контур параметров мест хранения с зависимыми lookup.

### Этап 5. Сквозная приемка v1

```text
PS-12 → PS-13 → PS-14
```

Результат: роли, меню, аудит, ошибки, тестовая матрица, трассировка и документация завершены.

## 9. Definition of Done Plant Structure v1

ПР03 v1 считается завершенным, когда:

- production-модуль не зависит от demo-модулей и разворачивается на пустой БД;
- реализованы все восемь сущностей владения из `01_scope.md` и только они;
- TPH-модель организационных единиц, discriminator и abstract/reference-only contract работают согласованно;
- tenant scope, same-tenant references, archive/delete и reference integrity применяются сервером;
- `ProductionUnit` и `WarehouseBin` поддерживают утвержденные иерархии без циклов;
- места хранения, зоны, ячейки и зависимые параметры соблюдают правила принадлежности;
- параметры хранения создаются и удаляются системой и редактируются только в `Draft` владельца;
- жизненный цикл организационных единиц соответствует `04_workflows.md`;
- все list/lookup/details/tree/cards/tabs из `06_ui_views.md` материализованы;
- матрица ролей и прав из `11_permissions.md` проверена тестами;
- скрытые ссылки `InventoryStatus` не опубликованы, а физические колонки сохранены для будущего расширения;
- аудит и стабильные ошибки покрывают пользовательские и системные изменения;
- трассировка ПР03 не содержит требований v1 без реализации и теста.

## 10. Отложенные вопросы, не блокирующие v1

1. Связывание `ProductionUnitType.Site` с платформенным `Site` выполняется после завершения платформенной миграции `Plant → Site`; локальный CRUD производственной единицы этим не блокируется.
2. Публикация пяти ссылок `InventoryStatus` выполняется после появления стабильного контракта модуля производственной логистики без переименования подготовленных колонок.
3. Общие межмодульные dependency checks для `ReturnToDraft`, архивирования и отключения места хранения подключаются после появления платформенного механизма; в v1 каждый потребитель защищает собственную целостность.
4. `ProductionUnitOperationParameters` проектируется модулем управления операциями и не добавляется в ПР03 без отдельного решения.
5. Расширенные правила доступности мест хранения и условные зависимости `Transfer*DefaultType` остаются за границей утвержденного набора `PS-IP-*` v1.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.8 | 2026-09-02 14:05 +04:00 | Codex | PS-12 — PS-14 | Реализованы IAM/навигация, аудит и диагностика; добавлена матрица технической приемки, открыта ручная проверка | — |
| 0.7 | 2026-09-02 13:35 +04:00 | Codex | PS-12 — PS-13 | Начата сквозная приемка: navigation permissions, системный audit и стабильные коды диагностики | — |
| 0.6 | 2026-09-02 12:44 +04:00 | Codex | PS-11 | Убран вложенный раздел дочерней карточки; вкладка параметров скрывается для записей, не являющихся местом хранения | — |
| 0.5 | 2026-09-02 12:05 +04:00 | Codex | PS-11 | Исправлен renderer `WithOwner`: detail-форма параметров встроена во вкладку владельца и связана с агрегатным черновиком | — |
| 0.4 | 2026-09-02 11:43 +04:00 | Codex | PS-11 | Исправлено обновление baseline карточки параметров места хранения; добавлена поэлементная проверка layout | — |
| 0.3 | 2026-09-02 11:01 +04:00 | Codex | PS-09 | Зафиксировано, что состояние Workflow не является критерием lookup места хранения; Draft-места хранения разрешены | — |
| 0.2 | 2026-09-01 16:59 +04:00 | A-Zhigalin (@A-Zhigalin) | PS-07. Реализовать abstract OrganizationalUnit и справочник субподрядчиков; PS-08. Реализовать справочник и иерархию производственных единиц | Этап 3 - организационные единицы (производственные единицы и субподрядчики) | [704d967c](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/704d967c0007f94d9719df2c9605a9f567a22dfd) |
| 0.1 | 2026-09-01 14:08 +04:00 | A-Zhigalin (@A-Zhigalin) | Создание документа | Производственная структура. Этапы 1, 2 беклога (Организации и типы складских ячеек) | [c9b5d21b](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/c9b5d21bc7d4d4de229432012e4efb06f02973ff) |
