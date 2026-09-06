---
id: DOC-04-03-98
title: 'Журнал решений - 03 Производственная структура'
type: appendix
status: approved
version: '1.2'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 03_plant_structure
holder: '@axelprosoft'
created_at: 2026-08-14 12:39
created_by: '@axelprosoft'
updated_at: 2026-09-02 11:01
last_modified_by: 'Codex'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Журнал решений - 03 Производственная структура

Документ фиксирует открытые вопросы и последующие проектные решения по прикладному модулю `03_plant_structure` / `Производственная структура`.

Это рабочий журнал решений, а не финальный дизайн-документ. Итоговые нормативные документы должны быть собраны из закрытых решений в более компактной структуре:

```text
../00_module_overview.md
../01_scope.md
../02_domain_model.md
../03_object_runtime_model.md
../04_workflows.md
../05_rules.md
../06_ui_views.md
../10_value_set_data_usage.md
../11_permissions.md
../13_operations.md
../90_traceability_pr03.md
../backlog.md
```

## Источники

- `DMP ПР03 Производственная структура.docx`
- `DMP ПР04 Управление ресурсами.docx`
- `DMP_DATA`
- `docs-new/00_governance/00_documentation_strategy.md`
- `docs-new/00_governance/requirements/001_dmp_functional_requirements_and_constraints.md`
- `docs-new/04_domain_modules/00_module_documentation_template.md`
- `docs-new/04_domain_modules/00_common`
- `docs-new/04_domain_modules/01_general_master_data`
- `docs-new/03_platform`
- исходный код DMP

## Вопрос 1. Порядок проектирования ПР03 и ПР04

Статус: принято.

### Формулировка

Нужно ли сначала проектировать модуль `Управление ресурсами` по ПР04, или модуль `Производственная структура` по ПР03 может проектироваться первым?

### Что говорит ПР03

ПР03 описывает производственную структуру как иерархию производственных единиц с корневым элементом `Предприятие`.

ПР03 также описывает:

- внешние элементы производственной структуры, включая субподрядчиков;
- места хранения как производственные единицы или субподрядчики с признаком `Место хранения`;
- зоны хранения;
- складские ячейки;
- организации, сопоставляемые предприятию или производственным единицам.

### Что говорит ПР04

ПР04 использует производственную структуру как внешний контекст для ресурсов:

- рабочие места имеют место установки;
- сотрудники имеют место работы;
- ресурсы и их местоположения ссылаются на производственные единицы;
- графики работы назначаются производственным единицам, субподрядчикам, ресурсам, группам рабочих мест и сотрудникам.

### Что говорит DMP_DATA

DMP_DATA содержит связи объектов ПР04 с `ProductionUnit` и `Subcontractor`.

Сами объекты `WorkPlace`, `Personnel`, `EquipmentUnit`, группы ресурсов, графики работы и ресурсные присвоения относятся к ресурсному контуру, а не к владению ПР03.

### Решение

ПР03 нужно проектировать до детальной проектной документации ПР04, потому что ПР04 зависит от стабильных ссылок на производственную структуру.

ПР03 можно проектировать без готового ПР04, если:

- объекты ресурсов описываются только как внешние будущие ссылки;
- ПР03 не объявляет рабочие места, сотрудников, оборудование, оснастку и графики работы своим владением;
- правила выбора ресурсов и графиков остаются в ПР04.

ПР04 `Управление ресурсами` использует объекты ПР03 как внешние ссылки. Отсутствие готовой проектной документации ПР04 не блокирует проектирование ПР03.

### Что должно попасть в проектную документацию

- `01_scope.md`: граница с Resource Management.
- `02_domain_model.md`: объекты владения ПР03 и внешние ссылки на будущий ПР04, если они есть.
- `03_object_runtime_model.md`: источники выбора ссылок на производственные единицы для соседних модулей.
- `90_traceability_pr03.md`: решение о порядке проектирования ПР03 и ПР04.

## Вопрос 2. Владение объектом `Company` / Организация

Статус: принято.

### Формулировка

Является ли `Company` / `Организация` объектом владения модуля ПР03 или внешней ссылкой на GMD, ERP, IAM / tenant-security либо будущий организационно-юридический контур?

### Что говорит ПР03

Исходный текст ПР03 указывает, что предприятию или производственным единицам может быть сопоставлена одна или несколько организаций для интеграционного обмена данными в области сбыта, закупок, фактических затрат, управления персоналом и расчета заработной платы. В целевой модели эта формулировка не трактуется как множественная связь одной `ProductionUnit` с `Company`: каждой производственной единице сопоставляется не более одной балансовой организации.

В ПР03 также описана форма `Организации` и карточка `Организация`.

### Что говорит DMP_DATA

DMP_DATA содержит объект `Company` с полями:

- `Id`;
- `Code`;
- `Name`;
- `Presentation`;
- `Contractor`;
- `Status`;
- `ExternalId`.

`ProductionUnit` содержит ссылку `Company`.

### Что уже принято в 00 Common / 01 GMD / платформенных документах

`Contractor` является объектом `01_general_master_data`.

`TenantId` и `PlantId` не входят в универсальный `CommonObject`. Если конкретный объект относится к предприятию или площадке, это должно быть явно описано в модуле-владельце.

### Варианты решения

Вариант A: `Company` входит во владение ПР03 как организация, сопоставляемая с производственной структурой.

Вариант B: `Company` не входит во владение ПР03; ПР03 хранит только ссылки на внешний объект организации.

Вариант C: `Company` входит в ПР03 только как минимальный справочник интеграционных организаций, а юридически значимая карточка остается в GMD/ERP.

### Решение

`Company` / `Организация` входит во владение ПР03 как справочник балансовых организаций.

ПР03 является модулем справочных данных производственной структуры, поэтому `Company` может быть размещена в ПР03, хотя теоретически могла бы быть вынесена в `01_general_master_data` или будущий юридически-финансовый контур.

Смысл объекта:

```text
Company / Организация = балансовая организация.
```

`Company` не является:

- `Tenant`;
- `Site`;
- `ProductionUnit`;
- `Contractor`.

Связь `Company.Contractor -> Contractor` сохраняется как внешняя ссылка на `Contractor` из `01_general_master_data`.

Кардинальность связи с производственной единицей фиксируется как `ProductionUnit.Company -> Company` (`0..1`). Связующая сущность и коллекция организаций у `ProductionUnit` не вводятся.

Смысл связи: балансовая организация может быть сопоставлена с контрагентом, если она должна выступать как юридическое лицо / документная сторона в закупках, продажах, интеграции, ЭДО, межфирменных операциях или обмене с внешними системами.

Обязательность `Company.Contractor`: предварительно считать необязательной, потому что балансовая организация может использоваться только для внутренней аналитики, зарплаты или учета.

Отдельно зафиксировано различие с субподрядчиком:

```text
Subcontractor.Contractor -> Contractor
```

`Subcontractor.Contractor` представляет субподрядчика как внешнего участника производственного процесса. Поле остается необязательным по принятому решению. Это является осознанным уточнением буквальной формулировки `TERM-004`, согласно которой для предприятия-субподрядчика должен быть указан соответствующий контрагент: ПР03 допускает ведение организационной единицы `Subcontractor` без сопоставления с записью `Contractor` из GMD.

Если конкретный процесс смежного модуля требует взаимодействия с субподрядчиком именно как с контрагентом, обязательность заполнения `Contractor` проверяется этим процессом, а не при создании `Subcontractor` в ПР03.

Обе связи на `Contractor` допустимы и имеют разную семантику:

- `Company.Contractor` - представление балансовой организации как юридической / документной стороны;
- `Subcontractor.Contractor` - представление субподрядчика как внешнего исполнителя / контрагента.

`Contractor` остается объектом владения `01_general_master_data`. ПР03 владеет только своими ссылочными полями на `Contractor`.

### Что должно попасть в проектную документацию

- `01_scope.md`: `Company` входит во владение ПР03 как справочник балансовых организаций; финансовые, бухгалтерские, налоговые и кадровые процессы по организации не входят в ПР03.
- `02_domain_model.md`: поля `Company` из ПР03 / DMP_DATA, необязательная одиночная связь `ProductionUnit.Company -> Company`, связь `Company.Contractor -> Contractor`.
- `02_domain_model.md`: необязательная связь `Subcontractor.Contractor -> Contractor`, ее отличие от `Company.Contractor` и осознанное уточнение `TERM-004`.
- `03_object_runtime_model.md`: источники выбора `Company` для `ProductionUnit.Company` и `Contractor` из GMD для `Company.Contractor` / `Subcontractor.Contractor`.
- `90_traceability_pr03.md`: причина размещения `Company` в ПР03 и внешняя зависимость на `Contractor` из GMD.
- Backlog / traceability для `01_general_master_data`: проверить, нужно ли добавить обратное упоминание, что `Contractor` используется внешними модулями через `Company.Contractor` и `Subcontractor.Contractor`.

## Вопрос 3. Базовый объект `OrganizationalUnit`

Статус: принято.

### Формулировка

Как реализовать `OrganizationalUnit`, если большинство объектов ПР02, логистики и соседних модулей ссылаются на организационную единицу, а фактически организационная единица может быть либо собственной производственной единицей, либо субподрядчиком?

### Что говорит ПР03

ПР03 описывает организационную единицу как общий объект для производственной единицы и субподрядчика.

Общие поля организационной единицы:

- `Id`;
- `Code`;
- `Name`;
- `FullName`;
- `Presentation`;
- `IsSegmentLevel`;
- `IsOperationLevel`;
- `IsInventoryStorageLocation`;
- `Description`;
- `Status`;
- `ExternalId`.

### Что говорит DMP_DATA

DMP_DATA содержит `OrganizationalUnit` и специализации:

- `ProductionUnit`;
- `Subcontractor`.

Складские и логистические параметры ссылаются на `OrganizationalUnit`.

### Что уже принято в 00 Common / 01 GMD / платформенных документах

`Presentation` не переносится как обязательное физическое поле. Должно быть описано правило представления в Object Runtime.

`Status` не входит в `CommonObject` и `CommonCatalogObject`; статусность должна решаться как отдельная возможность object type.

### Варианты решения

Вариант A: одна таблица `OrganizationalUnit` со всеми полями `ProductionUnit` и `Subcontractor`, discriminator-полем и несколькими runtime object types над одной таблицей.

Вариант B: базовая таблица `OrganizationalUnit` плюс отдельные таблицы `ProductionUnit` и `Subcontractor`.

Вариант C: не хранить общий `OrganizationalUnitId`, а использовать полиморфную ссылку `ObjectTypeCode + ObjectId`.

### Решение

Принять вариант A.

Физическая модель:

```text
OrganizationalUnit
  Id
  UnitKind: ProductionUnit | Subcontractor
  общие поля OrganizationalUnit
  поля ProductionUnit, nullable для Subcontractor
  поля Subcontractor, nullable для ProductionUnit
```

`OrganizationalUnit` и его контракт являются полноценным, но абстрактным object type для Object Runtime и baseline.

Это не просто таблица и не технический DTO. Для него должны существовать:

- `ObjectTypeCode = OrganizationalUnit`;
- `BusinessObjectContract<OrganizationalUnit>`;
- `ObjectType` в baseline;
- общие поля;
- `ReferenceSource` / lookup view для выбора организационной единицы;
- правило presentation;
- правила ссылочности;
- storage binding на таблицу `OrganizationalUnit`.

Ограничения abstract object type:

- нельзя создать строку напрямую как абстрактную организационную единицу;
- самостоятельная пользовательская карточка `OrganizationalUnit` не создается;
- отдельный пользовательский справочник `OrganizationalUnit` в ПР03 v1 не создается;
- каждая запись должна иметь discriminator `UnitKind`.

Concrete object types:

```text
ProductionUnit : OrganizationalUnit
Subcontractor : OrganizationalUnit
```

`ProductionUnit` и `Subcontractor` являются полноценными concrete object types:

- имеют собственные `ObjectTypeCode`;
- имеют собственные `BusinessObjectContract`;
- имеют собственные списки, карточки, права, правила, операции и UI;
- используют ту же физическую таблицу `OrganizationalUnit`;
- имеют обязательный discriminator filter по `UnitKind`.

Ссылки из ПР02, логистики и других модулей на организационную единицу должны хранить общий ключ:

```text
OrganizationalUnitId
```

Для lookup на организационную единицу:

```text
OrganizationalUnit_LookupView
  source: OrganizationalUnit
  возвращает: OrganizationalUnitId, UnitKind, ConcreteObjectTypeCode, Presentation
```

Навигация из ссылки должна открывать карточку concrete object type:

```text
UnitKind = ProductionUnit -> ProductionUnit_DetailView
UnitKind = Subcontractor -> Subcontractor_DetailView
```

Права:

- права на списки и карточки задаются на concrete object types `ProductionUnit` и `Subcontractor`;
- lookup `OrganizationalUnit` должен фильтровать строки по правам concrete object type;
- если пользователь имеет право видеть `ProductionUnit`, но не имеет права видеть `Subcontractor`, строки `Subcontractor` не должны попадать в lookup.

Платформенное состояние:

Текущая платформа уже содержит `BaseObjectTypeCode` в конфигурации и contract/baseline path, но для этой модели нужна дополнительная поддержка:

- `IsAbstract`;
- `IsReferenceOnly`;
- `DiscriminatorMember`;
- `DiscriminatorValue`;
- `ConcreteTypes`;
- discriminator-based storage filter для concrete object types;
- row-level permission by concrete object type для abstract lookup/list;
- navigation resolver из abstract reference к concrete card;
- запрет стандартных create/edit/delete actions на abstract object type.

До появления общей платформенной поддержки допускается временный custom provider / lookup для `OrganizationalUnit`, но нормативное решение ПР03 должно описывать целевую модель через abstract object type.

### Что должно попасть в проектную документацию

- `02_domain_model.md`: `OrganizationalUnit` как abstract object type, discriminator `UnitKind`, concrete object types `ProductionUnit` и `Subcontractor`, единая таблица.
- `03_object_runtime_model.md`: abstract/reference-only object type, concrete routing, lookup, navigation resolver, storage filters, права по concrete type.
- `06_ui_views.md`: отдельного пользовательского справочника `OrganizationalUnit` нет; есть lookup/search view и отдельные списки/карточки `ProductionUnit` и `Subcontractor`.
- `11_permissions.md`: права задаются на concrete object types; abstract lookup фильтруется по правам concrete type.
- Platform backlog: поддержка abstract/reference-only object type и discriminator-based concrete object types в Object Runtime / Configurator.

## Вопрос 4. Статус объектов ПР03

Статус: принято.

### Формулировка

Как проектировать `Status` для объектов ПР03: как workflow, как стандартное архивирование Common или как временный доменный SystemEnum?

### Что говорит ПР03

ПР03 содержит поле `Status` у основных объектов и действия `Изменить статус`, `Показывать в статусе`.

В карточках статус описан как `Черновик`, `Опубликованный`, `Архивный`.

### Что говорит DMP_DATA

DMP_DATA содержит поле `Status` у `OrganizationalUnit`, `WarehouseBinType`, `Company`, `StorageArea`, `WarehouseBin` и других объектов.

### Что уже принято в 00 Common / платформенных документах

`Status` не входит в `CommonObject` и `CommonCatalogObject`.

Для объектов с жизненным циклом целевое решение должно быть описано как workflow или отдельная lifecycle-возможность object type. Для одного object type нельзя смешивать прямое архивирование runtime action-ом и архивирование через workflow.

### Варианты решения

Вариант A: описать типовой workflow `Черновик -> Опубликован -> Архив`.

Вариант B: не вводить workflow, использовать стандартное архивирование Common и признак `IsArchived`.

Вариант C: временно сохранить `Status` как SystemEnum ПР03 до общего решения по lifecycle.

### Решение

ПР03 следует решениям `00 Common` и `01 General Master Data`:

- `Status` из ПР03 / DMP_DATA не переносится как обычное физическое поле доменной модели;
- `Status` не фиксируется как `SystemEnum` ПР03;
- `DmpStatusBase` не возвращается как базовый класс;
- жизненный цикл подключается только для object types, которым нужен контроль допуска к рабочему использованию;
- для объектов без отдельного жизненного цикла используется Common-архивирование / восстановление.

Lifecycle `Draft / Published / Archived` включается для `OrganizationalUnit` и concrete object types:

```text
OrganizationalUnit
ProductionUnit
Subcontractor
```

Причина: для организационных единиц требуется жизненный цикл подготовки и допуска к рабочему использованию.

Для объектов:

```text
Company
WarehouseBinType
StorageArea
WarehouseBin
```

отдельный lifecycle в ПР03 v1 не включается без дополнительного обоснования. Для них используется Common-архивирование / восстановление, если object contract явно включает такие действия.

Нельзя смешивать для одного object type:

- прямые действия Common `Archive` / `Restore`;
- архивирование через lifecycle / workflow state `Archived`.

Если object type управляется lifecycle, архивное состояние является источником архивирования, а Common-поле `IsArchived` является материализованным техническим признаком.

### Что должно попасть в проектную документацию

- `04_workflows.md`: lifecycle `Draft / Published / Archived` только для `OrganizationalUnit`, `ProductionUnit`, `Subcontractor`; специальных workflow для остальных объектов нет.
- `05_rules.md`: предметные ограничения переходов и изменения организационных единиц; правила архивирования остальных объектов через Common.
- `03_object_runtime_model.md`: отображение состояний Workflow для организационных единиц, обработчики Object Runtime и интеграция с базовыми механизмами платформы.
- `02_domain_model.md`: не включать `Status` как обычное поле и не создавать `Status` SystemEnum.
- `90_traceability_pr03.md`: отличие от legacy-поля `Status` в ПР03 / DMP_DATA и ссылка на решения `00 Common` / `01 GMD`.

## Вопрос 5. SystemEnum и ValueSet в ПР03

Статус: принято по перечислениям владения ПР03. Кандидаты по `Contractor` разбираются отдельно в контексте `01_general_master_data`.

### Формулировка

Какие списки значений ПР03 являются SystemEnum, а какие должны быть ValueSet или принадлежать другим модулям?

### Что говорит ПР03

ПР03 содержит фиксированные таблицы перечислений, но не все они относятся к владению ПР03:

- `Тип производственной единицы`;
- `Вид контрагента`;
- `Категория контрагента`;
- `Тип контакта`.

### Что говорит DMP_DATA

DMP_DATA содержит поле типа производственной единицы в `ProductionUnit`.

Поля, связанные с контрагентами и контактами контрагентов, относятся к объектам `Contractor` и `ContractorContact`, которые являются владением `01_general_master_data`.

### Что уже принято в 00 Common / 01 GMD / платформенных документах

Фиксированная таблица enum из ПР должна фиксироваться как SystemEnum, если она принадлежит модулю.

ValueSet используется только для настраиваемых наборов значений.

`Contractor` и `ContractorContact` являются владением `01_general_master_data`, а не ПР03.

В уже созданной документации `01_general_master_data` поля `Contractor.Type`, `Contractor.Category` и `ContractorContact.ContactType` описаны через `ValueSetCode`, но для ПР03 это внешнее решение. В рамках ПР03 не фиксировать дополнительное решение по каждому такому полю.

### Варианты решения

Вариант A: `ProductionUnitType` фиксируется как SystemEnum ПР03; кандидаты по контрагентам отдельно разбираются в GMD.

Вариант B: в ПР03 фиксируется только enum, принадлежащий объектам владения ПР03; таблицы, относящиеся к `Contractor` / `ContractorContact`, не проектируются в ПР03 и идут в отдельный разбор GMD.

Вариант C: все перечисления из ПР03 фиксируются в ПР03.

### Решение

Принять вариант B.

`ProductionUnitType` фиксировать как SystemEnum модуля ПР03 в `02_domain_model.md`.

Состав `ProductionUnitType` должен включать тип `Site` / `Площадка`, согласованный в вопросе 13:

```text
Enterprise        / Предприятие
Site              / Площадка
Department        / Подразделение
Shopfloor         / Цех
ShopfloorArea     / Производственный участок
Warehouse         / Склад
RepairService     / Ремонтная служба
Other             / Другое
```

Состав enum остается именно таким и не расширяется в v1. Элементы производственной структуры, для которых нет отдельного значения, используют подходящий существующий тип либо `Other`; конкретная семантика уточняется наименованием и описанием записи.

Собственные ValueSet в ПР03 на текущем этапе не вводить: подтвержденных настраиваемых наборов значений ПР03 пока нет.

Кандидаты на классификаторы для `Contractor` / `ContractorContact` не переносить в ПР03. Их нужно обсуждать отдельно по одному полю в границах `01_general_master_data`, с учетом уже принятых решений GMD.

### Что должно попасть в проектную документацию

- `02_domain_model.md`: `ProductionUnitType` как SystemEnum ПР03.
- `10_value_set_data_usage.md`: указать, что собственные ValueSet ПР03 не вводятся, если к моменту проектирования не появятся подтвержденные настраиваемые наборы значений.
- `90_traceability_pr03.md`: граница с GMD по классификаторам контрагентов без переноса этих решений в ПР03.

## Вопрос 6. Настройки предприятия `PlantParameters`

Статус: принято.

### Формулировка

Должны ли настройки предприятия, показанные в карточке корневой производственной единицы, входить в ПР03 как объект `PlantParameters`, или это набор runtime settings с регистрацией в baseline settings catalog модулями-владельцами смысла?

### Что говорит ПР03

ПР03 показывает группу полей `Настройки предприятия`, доступную только для корневого элемента производственной структуры.

В примечании ПР03 указано, что описание настроек предприятия см. в `DMP ПР09 Производственная логистика`.

### Что говорит DMP_DATA

DMP_DATA содержит объект `PlantParameters`, связанный с `Plant`, и поля:

- `InventoryAnalyticUsage`;
- `CompositionType`;
- `AllowNegativeBalance`;
- `LotNumberUniqueness`;
- `MasterDataDefaultStatus`;
- `SetArchiveStatusInsteadOfDelete`;
- `IsApprovedOnlyDefault`;
- `IsSetBaseVersionDefault`;
- `IsBaseProductBillDefault`;
- `IsCalcWithLossScrapDefault`;
- `IsCalcByCoproductsDefault`;
- `ReceiptWarehouse`;
- `DeliveryWarehouse`;
- `DurationFormat`.

### Что уже принято в 00 Common / платформенных документах

`PlantId` не является универсальным полем Common. Объект, которому нужен plant-контекст, должен описывать его явно.

Платформенные настройки и прикладные настройки не должны смешиваться без решения о владении.

Платформенный механизм Settings уже поддерживает runtime settings с техническим ключом `ModuleCode + SettingCode`, хранением значений в `RuntimeSettingValue` и override policy до уровня `Plant`; после принятого решения по переименованию платформенного `Plant` целевой уровень называется `Site`.

### Варианты решения

Вариант A: `PlantParameters` принадлежит ПР03, потому что расположен на корне производственной структуры.

Вариант B: `PlantParameters` принадлежит ПР09, а ПР03 только предоставляет контекст корневой производственной единицы.

Вариант C: часть общих настроек предприятия выносится в платформенные settings, а прикладные логистические настройки остаются в ПР09.

Вариант D: `PlantParameters` как таблица / object type не переносится в целевую модель; поля старого объекта раскладываются на baseline settings definitions разных модулей-регистраторов.

### Решение

Принять вариант D.

`PlantParameters` не входит во владение ПР03 и не проектируется как object type, таблица или карточка ПР03.

Целевая модель:

```text
Platform Settings
  Scope: Corporate -> Tenant -> Site
  identity: ModuleCode + SettingCode
```

Старый объект `PlantParameters` является сборной формой site-level settings. В baseline settings catalog каждую настройку регистрирует модуль-владелец ее смысла:

| Поле `PlantParameters` | Решение |
|---|---|
| `MasterDataDefaultStatus` | Удалить из целевой модели. Не переносить старый универсальный `Status` в новые settings. |
| `SetArchiveStatusInsteadOfDelete` | Удалить из целевой модели. Архивирование / удаление решается через Common archive, lifecycle и правила object type, а не через site-level setting. |
| `DurationFormat` | Исходное имя из ПР09. В целевой модели регистрируется `00 Common` как настройка `Common.TimeAmountFormat`; системный enum `TimeAmountFormatType` описан в доменной модели Common. Это формат ввода и отображения количественного времени (`Standard` = `hh:mm:ss`, `Industrial` = нормо-часы с тремя десятичными знаками), а не настройка производственной структуры или логистики. |
| `CompositionType` | Регистрирует ПР02 `Составы и технологии`. |
| `IsApprovedOnlyDefault` | Регистрирует ПР02 `Составы и технологии`. |
| `IsSetBaseVersionDefault` | Регистрирует ПР02 `Составы и технологии`. |
| `IsBaseProductBillDefault` | Регистрирует ПР02 `Составы и технологии`. |
| `IsCalcWithLossScrapDefault` | Регистрирует ПР02 `Составы и технологии`. |
| `IsCalcByCoproductsDefault` | Регистрирует ПР02 `Составы и технологии`. |
| `InventoryAnalyticUsage` | Временно регистрирует ПР09 `Производственная логистика`. Настройка является кандидатом на перенос в будущий общий контур операционных / производственных аналитик, если он будет выделен. |
| `AllowNegativeBalance` | Регистрирует ПР09 `Производственная логистика`. |
| `LotNumberUniqueness` | Регистрирует ПР09 `Производственная логистика`. Platform Numbering может читать эту настройку как зависимую, но не регистрирует ее. |
| `ReceiptWarehouse` | Регистрирует ПР09 `Производственная логистика`; значение является ссылкой на объект ПР03 `OrganizationalUnit` / storage location. |
| `DeliveryWarehouse` | Регистрирует ПР09 `Производственная логистика`; значение является ссылкой на объект ПР03 `OrganizationalUnit` / storage location. |

ПР03 не регистрирует ни одну настройку из старого `PlantParameters`.

ПР03 предоставляет только предметные объекты, которые могут быть reference targets для чужих settings:

- `ProductionUnit(type = Site / Площадка)`;
- `OrganizationalUnit`;
- места хранения как организационные единицы с `IsInventoryStorageLocation = true`;
- `StorageArea`;
- `WarehouseBin`.

UI-группа `Настройки предприятия` не проектируется в карточке ПР03. Настройки редактируются через Studio Settings на соответствующем scope `Site`.

### Что должно попасть в проектную документацию

- `01_scope.md`: настройки предприятия не входят во владение ПР03; `PlantParameters` не является объектом модуля.
- `03_object_runtime_model.md`: ПР03 предоставляет reference targets для чужих settings, но не владеет settings catalog definitions.
- `06_ui_views.md`: группа `Настройки предприятия` не включается в базовую карточку ПР03; настройки ведутся через Studio Settings.
- `90_traceability_pr03.md`: причина исключения `PlantParameters` и разложение старых полей по модулям-регистраторам.

## Вопрос 7. Настройки логистики организационной единицы

Статус: принято.

### Формулировка

Должны ли `OrganizationalUnitInventoryParameters` входить в ПР03 как зависимый объект места хранения, или принадлежать модулю производственной логистики ПР09?

### Что говорит ПР03

ПР03 показывает вкладку `Настройки логистики` для производственной единицы и субподрядчика, если `IsInventoryStorageLocation = true`.

В примечании ПР03 указано, что описание настроек логистики организационной единицы см. в `DMP ПР09 Производственная логистика`.

### Что говорит DMP_DATA

DMP_DATA содержит объект `OrganizationalUnitInventoryParameters`, связанный с `OrganizationalUnit`.

Поля объекта относятся к складскому и производственно-логистическому поведению:

- расчет остатков;
- учет по МОЛ;
- учет по ячейкам хранения;
- учет по единицам транспортировки;
- статусы запасов;
- места хранения выпуска, брака, отпуска, списания;
- правила перемещения годных, отклоненных и бракованных количеств.

### Что уже принято в 00 Common / 01 GMD / платформенных документах

Производственная структура владеет местами хранения, зонами и ячейками. Логистические движения, статусы запасов, транспортные единицы и правила складского учета не являются владением ПР03.

### Варианты решения

Вариант A: включить `OrganizationalUnitInventoryParameters` в ПР03 как зависимый объект параметров места хранения.

Вариант B: оставить `OrganizationalUnitInventoryParameters` во владении ПР09, а в ПР03 описать только зависимую вкладку и ссылку на будущий модуль.

Вариант C: разделить: базовый признак места хранения в ПР03, логистические параметры в ПР09.

### Решение

Принять вариант A.

`OrganizationalUnitInventoryParameters` входит во владение ПР03 как зависимый объект параметров места хранения.

Основания:

- объект привязан к конкретной `OrganizationalUnit`;
- запись должна существовать, только если `OrganizationalUnit.IsInventoryStorageLocation = true`;
- создание / удаление записи следует из изменения признака места хранения;
- ПР03 уже владеет местами хранения, зонами хранения и складскими ячейками;
- межмодульное создание зависимой записи через ПР09 не требуется.

Правила:

```text
OrganizationalUnit.IsInventoryStorageLocation = true
  -> должна существовать одна запись OrganizationalUnitInventoryParameters.

OrganizationalUnit.IsInventoryStorageLocation = false
  -> после проверки отсутствия активных StorageArea и WarehouseBin ПР03 запись OrganizationalUnitInventoryParameters физически удаляется в той же транзакции; входящие ссылки других модулей в v1 не проверяются.
```

`OrganizationalUnitInventoryParameters` редактируется в карточке `ProductionUnit` / `Subcontractor` только если `IsInventoryStorageLocation = true` и владелец находится в состоянии `Draft`. Для владельца в `Published` или `Archived` параметры доступны только для чтения. Роль `PlantStructureStorageResponsible` не получает право `ReturnToDraft`; возврат владельца в черновик выполняет пользователь с правами жизненного цикла производственной структуры.

### Что должно попасть в проектную документацию

- `01_scope.md`: `OrganizationalUnitInventoryParameters` входит во владение ПР03 как зависимый объект места хранения.
- `02_domain_model.md`: поля `OrganizationalUnitInventoryParameters`, связь 1:1 с `OrganizationalUnit`, обязательность при `IsInventoryStorageLocation = true`.
- `03_object_runtime_model.md`: автоматическое создание / удаление зависимой записи при изменении `IsInventoryStorageLocation`.
- `05_rules.md`: правила существования записи и ограничения удаления / архивирования.
- `06_ui_views.md`: вкладка / группа `Настройки логистики` в карточке `ProductionUnit` / `Subcontractor`, доступная только если `IsInventoryStorageLocation = true`.
- `90_traceability_pr03.md`: причина включения `OrganizationalUnitInventoryParameters` в ПР03.

## Вопрос 8. Параметры операций производственной единицы

Статус: принято.

### Формулировка

Должны ли `ProductionUnitOperationParameters` и `IsSequenceOperationsControl` входить в ПР03 или принадлежать модулю `Управление операциями` ПР10?

### Что говорит ПР03

ПР03 показывает вкладку `Параметры операций` в карточке производственной единицы.

В примечании ПР03 указано, что описание параметров управления операциями см. в `DMP ПР10 Управление операциями`.

### Что говорит DMP_DATA

DMP_DATA содержит объект `ProductionUnitOperationParameters` со ссылкой на `ProductionUnit`.

`ProductionUnitOperationParameters` наследуется от `OperationParametersBase`.

Фактические поля по DMP_DATA:

- `ProductionUnit`;
- `IsSequenceOperationsControl`;
- `WorkCardControlType`;
- `WorkCardPeriodType`.
- `ExternalId`.

### Что уже принято в 00 Common / платформенных документах

Прикладной модуль не должен включать во владение объект будущего модуля только потому, что он отображается на вкладке карточки.

### Варианты решения

Вариант A: включить параметры операций в ПР03 как часть производственной единицы.

Вариант B: отнести параметры операций к ПР10, а ПР03 описывает только ссылку и место расширения карточки.

Вариант C: оставить `IsSequenceOperationsControl` в ПР03, а остальные параметры вынести в ПР10.

### Решение

`ProductionUnitOperationParameters` принадлежит модулю управления ресурсами. Модуль 04 создает, изменяет, удаляет и предоставляет эффективные значения этой настройки. Модуль 03 не включает объект в собственную доменную модель и не владеет его данными; он предоставляет `ProductionUnit` как внешний объект, на который ссылается настройка.

ПР10 использует эффективные значения `ProductionUnitOperationParameters` при управлении операциями, но не становится владельцем объекта только из-за использования этих значений. Отображение настройки в карточке `ProductionUnit` не переносит владение в модуль 03.

### Что должно попасть в проектную документацию

- `01_scope.md`: параметры операций производственной единицы принадлежат модулю управления ресурсами и не входят во владение ПР03.
- `90_traceability_pr03.md`: закрытая межмодульная граница по `ProductionUnitOperationParameters`.
- `02_domain_model.md` и `03_object_runtime_model.md` модуля 04: объект и его настройки описываются как собственность модуля 04; ПР10 фиксируется как потребитель эффективных значений.

## Вопрос 9. Правила иерархии производственных единиц

Статус: принято.

### Формулировка

Как зафиксировать иерархию `ProductionUnit`: `Main`, `Parent`, несколько корневых предприятий внутри tenant и допустимость типа `Site` / `Площадка`?

### Что говорит ПР03

ПР03 задает правила:

- корневой элемент производственной структуры должен быть единственным;
- корневой элемент имеет `Id = Main` и `Id = Parent`;
- тип `Предприятие` может иметь только корневой элемент;
- в поле `Main` должен быть указан только корневой элемент;
- каждая производственная единица, кроме корневой, имеет одну вышестоящую производственную единицу.

После отдельного решения по `Tenant`, `Site` и производственной структуре правило единственного корня в tenant уточняется: один tenant может содержать несколько деревьев производственной структуры, каждое с корневой производственной единицей типа `Enterprise` / `Предприятие`.

### Что говорит DMP_DATA

DMP_DATA содержит `ProductionUnit` со ссылками:

- `Main -> ProductionUnit`;
- `Parent -> ProductionUnit`;
- `Company -> Company`;
- `Type -> ProductionUnitType`.

### Что уже принято в 00 Common / платформенных документах

Common не задает универсальный `PlantId`. Производственная структура сама должна определить корневую область и правила выбора.

### Варианты решения

Вариант A: сохранить ПР03 буквально: один корень `Enterprise` на tenant.

Вариант B: разрешить несколько корневых `Enterprise` в tenant; `Parent` задает дерево, `Main` указывает на корневой `Enterprise` своего дерева.

Вариант C: убрать `Main`, оставить только `Parent`, а корень вычислять обходом дерева.

### Решение

Принять вариант B.

Правила иерархии:

- `Parent` является основной иерархической связью;
- корневые узлы имеют `Parent = null`;
- корневые узлы должны иметь `Type = Enterprise` / `Предприятие`;
- в одном `Tenant` может быть несколько корневых `Enterprise`;
- каждая не корневая `ProductionUnit` должна иметь один `Parent`;
- `Main` сохраняется как денормализованная ссылка на корневой `Enterprise` своего дерева;
- `Main` не редактируется пользователем и заполняется Object Runtime / handler;
- для корневого `Enterprise` поле `Main` указывает на сам корневой объект;
- `Type = Enterprise` допускается только для корневых узлов;
- `Type = Site` / `Площадка` не обязан быть корнем и обычно является дочерним узлом внутри `Enterprise`;
- для всех типов, кроме `Enterprise`, в v1 допускаются произвольные сочетания `Parent.Type -> Type`; отдельная матрица совместимости типов не вводится;
- циклы в `Parent` запрещены;
- перенос узла между деревьями должен пересчитать `Main` для всего поддерева.

Пример допустимой структуры:

```text
Tenant
  ProductionUnit: Enterprise 1
    ProductionUnit: Site A
    ProductionUnit: Site B
  ProductionUnit: Enterprise 2
    ProductionUnit: Site C
```

### Что должно попасть в проектную документацию

- `02_domain_model.md`: поля и ссылки `ProductionUnit`.
- `03_object_runtime_model.md`: древовидное представление, расчет `Main`, ограничения выбора `Parent`.
- `05_rules.md`: проверки корневого типа, нескольких деревьев tenant, `Main`, `Parent` и циклов.

## Вопрос 10. Места хранения, зоны и складские ячейки

Статус: принято.

### Формулировка

Как описать границу между местом хранения, зоной хранения и складской ячейкой, а также правила их выбора и уникальности?

### Что говорит ПР03

ПР03 описывает:

- место хранения как организационную единицу с признаком `IsInventoryStorageLocation`;
- зоны хранения места хранения;
- складские ячейки, принадлежащие месту хранения;
- иерархию складских ячеек через `Parent`;
- тип складской ячейки.

### Что говорит DMP_DATA

DMP_DATA содержит:

- `StorageArea.Warehouse -> ProductionUnit`;
- `WarehouseBin.Warehouse -> ProductionUnit`;
- `WarehouseBin.StorageArea -> StorageArea`;
- `WarehouseBin.Parent -> WarehouseBin`;
- `WarehouseBin.WarehouseBinType -> WarehouseBinType`.

У `WarehouseBin` в DMP_DATA также есть `InventoryStatus`, который относится к логистическому контуру.

DMP_DATA ограничивает владельца зон и ячеек типом `ProductionUnit`, но ПР03 описывает место хранения шире: организационная единица с `IsInventoryStorageLocation = true`, то есть `ProductionUnit` или `Subcontractor`.

### Что уже принято в 00 Common / 01 GMD / платформенных документах

GMD использует `WarehouseBin` и `ProductionUnit` как внешние ссылки и не владеет ими.

Логистические статусы запасов и движения не входят в ПР03.

### Варианты решения

Вариант A: оставить ссылку `Warehouse -> ProductionUnit` как в DMP_DATA; зоны и ячейки доступны только для внутренних производственных единиц.

Вариант B: расширить ссылку `Warehouse` до `OrganizationalUnit`, чтобы зоны и ячейки могли принадлежать любой организационной единице-месту хранения, включая `Subcontractor`.

Вариант C: разделить: `ProductionUnit` может иметь зоны/ячейки, `Subcontractor` может быть только местом хранения без зон/ячеек.

### Решение

Принять вариант B.

- место хранения - это `OrganizationalUnit` с `IsInventoryStorageLocation = true`;
- местом хранения может быть `ProductionUnit` или `Subcontractor`;
- `StorageArea.Warehouse` должен ссылаться на `OrganizationalUnit`, а не только на `ProductionUnit`;
- `WarehouseBin.Warehouse` должен ссылаться на `OrganizationalUnit`, а не только на `ProductionUnit`;
- для `StorageArea.Warehouse` и `WarehouseBin.Warehouse` допустимы только организационные единицы с `IsInventoryStorageLocation = true`;
- `StorageArea` принадлежит одному `Warehouse`;
- `WarehouseBin` принадлежит одному `Warehouse`;
- если у `WarehouseBin` указана `StorageArea`, зона должна принадлежать тому же `Warehouse`;
- `WarehouseBin.Parent`, если заполнен, должен принадлежать тому же `Warehouse`;
- циклы в иерархии `WarehouseBin.Parent` запрещены;
- запас может размещаться в любой `WarehouseBin`, в том числе в ячейке с дочерними ячейками; `Parent` задает иерархическое расположение или группировку самостоятельных мест размещения, а не ограничение «только листья содержат запас»;
- для `WarehouseBin.InventoryStatus` в v1 сохраняется nullable GUID-колонка `InventoryStatusId`, но поле не публикуется как Object Runtime reference и не выводится в UI до появления стабильного контракта `InventoryStatus`; справочником статусов и правилами применения владеет модуль производственной логистики.

Расхождение с DMP_DATA:

```text
DMP_DATA: StorageArea.Warehouse -> ProductionUnit
DMP_DATA: WarehouseBin.Warehouse -> ProductionUnit

Целевая модель ПР03: Warehouse -> OrganizationalUnit(IsInventoryStorageLocation = true)
```

Причина: иначе невозможно единообразно поддержать субподрядчика как место хранения.

### Что должно попасть в проектную документацию

- `02_domain_model.md`: поля `StorageArea`, `WarehouseBinType`, `WarehouseBin`, ссылки `Warehouse -> OrganizationalUnit`.
- `03_object_runtime_model.md`: ограничения выбора `Warehouse`, `StorageArea`, `Parent`.
- `05_rules.md`: уникальность, запрет межскладских ссылок, запрет выбора ОЕ без `IsInventoryStorageLocation = true`.
- `06_ui_views.md`: списки, дерево ячеек и фильтры по месту хранения.
- `90_traceability_pr03.md`: расхождение с DMP_DATA по типу ссылки `Warehouse`.

Межмодульные dependency checks не входят в v1. При отключении места хранения, `ReturnToDraft` и архивировании ПР03 проверяет только принадлежащие ему объекты и ссылки. До появления общего платформенного механизма каждый модуль-потребитель отвечает за целостность собственных ссылок и должен учитывать состояние целевого объекта ПР03.

## Вопрос 11. UI-объем v1

Статус: принято.

### Формулировка

Какие формы, вкладки и действия из ПР03 переносить в первый пакет проектной документации, а какие оставить как зависимые от будущих модулей?

### Что говорит ПР03

ПР03 описывает списки и карточки:

- производственные единицы;
- субподрядчики;
- организации;
- зоны хранения;
- типы складских ячеек;
- складские ячейки.

Также описаны зависимые вкладки:

- `Параметры операций`;
- `Настройки логистики`;
- `Настройки предприятия`.

### Что говорит DMP_DATA

DMP_DATA содержит объекты для основных форм ПР03, а также зависимые объекты ПР09 и ПР10.

### Что уже принято в 00 Common / платформенных документах

UI должен описываться по объектам: список, список выбора, карточка, вкладки, фильтры, сортировка, группировка и источники значений.

Стандартные действия Object Runtime не нужно описывать как прикладные операции.

### Варианты решения

Вариант A: перенести все формы и вкладки из ПР03.

Вариант B: перенести только формы объектов владения ПР03; зависимые вкладки оставить для будущих модулей.

Вариант C: описать зависимые вкладки как extension points без проектирования их полей.

Вариант D: описать UI только для подтвержденных объектов владения ПР03; `Настройки логистики` включить только в части `OrganizationalUnitInventoryParameters`, а `Настройки предприятия` и `Параметры операций` исключить из карточек ПР03.

### Решение

Принять вариант D.

В UI v1 ПР03 описывать формы только для объектов владения ПР03:

- `ProductionUnit`;
- `Subcontractor`;
- `Company`;
- `OrganizationalUnitInventoryParameters` как вкладку / группу места хранения;
- `StorageArea`;
- `WarehouseBinType`;
- `WarehouseBin`.

Не включать в карточки ПР03:

- `Настройки предприятия` - ведутся через Studio Settings на scope `Site`;
- `Параметры операций` - отложены до проектирования ПР10.

`Настройки логистики` включаются только в части объекта `OrganizationalUnitInventoryParameters`, потому что он принят во владение ПР03 как зависимый объект места хранения.

### Что должно попасть в проектную документацию

- `06_ui_views.md`: списки, списки выбора, карточки, вкладки, фильтры, сортировки и группировки только для подтвержденных объектов ПР03.
- `01_scope.md`: исключение `Настройки предприятия` и `Параметры операций` из UI ПР03.
- `90_traceability_pr03.md`: причины исключения зависимых вкладок.

## Вопрос 12. Права модуля

Статус: принято.

### Формулировка

Нужны ли отдельные прикладные роли и специфические права ПР03, или достаточно стандартных прав Object Runtime на защищаемые объекты?

### Что говорит ПР03

ПР03 перечисляет действия форм: создание, изменение, удаление, изменение статуса, настройка колонок, фильтры, поиск, раскрытие дерева и переизвлечение данных.

Специфические ключи прав доступа в ПР03 явно не выделены.

### Что говорит DMP_DATA

DMP_DATA не задает отдельную модель прав ПР03.

### Что уже принято в 00 Common / платформенных документах

Права должны описываться как прикладные роли, защищаемые объекты и особенности доступа.

Не нужно перечислять стандартные технические права платформы без необходимости.

### Варианты решения

Вариант A: ввести технические роли `PlantStructureReader`, `PlantStructureEditor`, `PlantStructureAdmin`.

Вариант B: использовать стандартные права Object Runtime для объектов ПР03 без специальных прикладных функций.

Вариант C: добавить отдельное право только на изменение статуса / публикацию, если будет принят workflow.

Вариант D: использовать стандартные права Object Runtime и предметные роли по аналогии с `01_general_master_data`.

### Решение

Принять вариант D.

ПР03 не вводит специальных прикладных permission codes.

Доступ к объектам ПР03 строится на стандартных правах Object Runtime:

- просмотр;
- создание;
- изменение;
- удаление / архивирование в рамках правил объекта.

Самостоятельно защищаемые объекты:

- `ProductionUnit`;
- `Subcontractor`;
- `Company`;
- `OrganizationalUnitInventoryParameters`;
- `StorageArea`;
- `WarehouseBinType`;
- `WarehouseBin`.

`OrganizationalUnit` является abstract/reference-only object type. Отдельное создание `OrganizationalUnit` запрещено; lookup/search по `OrganizationalUnit` должен фильтроваться по правам на concrete object types `ProductionUnit` и `Subcontractor`.

Роли ПР03:

| Роль | Код | Назначение |
|---|---|---|
| Администратор производственной структуры | `PlantStructureAdmin` | Полное ведение объектов ПР03. |
| Пользователь производственной структуры | `PlantStructureUser` | Просмотр и выбор объектов ПР03. |
| Ответственный за производственную структуру | `PlantStructureResponsible` | Ведение производственных единиц, субподрядчиков и организаций. |
| Ответственный за складскую структуру | `PlantStructureStorageResponsible` | Ведение мест хранения, зон, типов ячеек и складских ячеек. |

Матрица доступа:

| Объект | Администратор производственной структуры | Пользователь производственной структуры | Ответственный за производственную структуру | Ответственный за складскую структуру |
|---|---|---|---|---|
| `ProductionUnit` | Все действия | Просмотр | Ведение | Просмотр |
| `Subcontractor` | Все действия | Просмотр | Ведение | Просмотр |
| `Company` | Все действия | Просмотр | Ведение | Просмотр |
| `OrganizationalUnitInventoryParameters` | Все действия | Просмотр | Просмотр | Ведение |
| `StorageArea` | Все действия | Просмотр | Просмотр | Ведение |
| `WarehouseBinType` | Все действия | Просмотр | Просмотр | Ведение |
| `WarehouseBin` | Все действия | Просмотр | Просмотр | Ведение |

Ограничения изменения и удаления задаются бизнес-правилами, а не отдельными permission codes.

### Что должно попасть в проектную документацию

- `11_permissions.md`: стандартные права Object Runtime, самостоятельно защищаемые объекты, роли, матрица доступа и особенность abstract/reference-only `OrganizationalUnit`.
- `90_traceability_pr03.md`: отсутствие специальных прикладных permission codes в ПР03.

## Вопрос 13. Tenant, Site / Plant и производственная структура

Статус: принято предварительное решение, требуется синхронизация платформенных документов и кода.

### Формулировка

Что означают платформенные `Tenant` и `Plant` относительно производственной структуры ПР03, и нужно ли переименовать платформенный `Plant` в `Site`?

Нужно определить:

- является ли `Plant` в ядре системы тем же самым, что площадка / завод / корневая или верхнеуровневая производственная единица ПР03;
- нужно ли использовать целевой термин `Site` вместо `Plant`, чтобы отделить площадку как platform scope от завода / предприятия в предметной модели;
- является ли `Tenant` предприятием, юридической организацией, внедрением DMP, границей данных или чем-то еще;
- как связать платформенные `TenantId` / `SiteId` с объектами `ProductionUnit`, не смешав security/configuration scope и предметную модель производственной структуры.

### Что говорит ПР03

ПР03 описывает производственную структуру как иерархию производственных единиц, где главным корневым элементом является предприятие.

В составе ПР03 встречаются:

- предприятие как тип производственной единицы;
- подразделение;
- цех;
- участок цеха;
- склад / кладовая;
- сервисно-ремонтная служба;
- субподрядчик как внешний элемент производственной структуры;
- организация, сопоставляемая предприятию или производственным единицам.

ПР03 не использует термин `Tenant` как предметный объект производственной структуры.

В общих функциональных требованиях для Plant Structure отдельно упоминаются площадки, но в enum `ProductionUnitType` из ПР03 отдельного значения `Site` / `Площадка` сейчас нет. Это требует уточнения: либо площадка моделируется существующим типом, либо в `ProductionUnitType` добавляется новый фиксированный тип `Site` / `Площадка`.

`Организация` в ПР03 следует понимать как балансовую / юридическую / учетную организацию, например для аналитики, зарплаты, учета, закупок, продаж или интеграции. Она не является заменой `Tenant`, `Site` или `ProductionUnit`.

### Что говорит `docs/01 sources`

В старых source-документах `Tenant` описан как операционная граница:

```text
Tenant = предприятие / завод / организация
```

Он определяет:

- границу данных;
- границу доступа;
- границу конфигурации;
- границу workflow и правил.

`Plant` описан как опциональный объект уровня MVP+:

```text
Plant
- Id
- TenantId
- Code
- Name
```

Использование `Plant` в старых источниках:

- multi-site производство;
- scope override для configuration и reporting;
- площадка / завод в цепочке применения конфигурации.

В документах по конфигурации встречается цепочка:

```text
SystemBaseline -> Corporate -> Tenant -> Plant
```

При этом в `06_configuration_platform.md` около `plant` уже стоит признак неуверенности:

```text
plant - ?
```

Это показывает, что смысл `Plant` как платформенного scope был не до конца закреплен.

Целевой термин для этого scope следует уточнить как `Site` / `Площадка`. Термин `Plant` в старых источниках и коде считать историческим названием того же платформенного понятия до миграции.

### Что говорит код

В текущем коде есть `ITenantContext`:

```text
TenantId
PlantId?
UserId
RoleCodes
```

Есть общий инвариант `ScopeCoordinateRules`:

```text
PlantId не может быть задан без TenantId
```

В `ConfigurationScope` поддержаны scope-уровни:

```text
Corporate
SystemBaseline
Tenant
Plant
```

Для `Plant`-scope требуются оба идентификатора:

```text
TenantId
PlantId
```

Также в Configuration Platform проверяется, что `Plant` scope может иметь parent только `Tenant` того же tenant.

Это подтверждает, что текущий `Plant` в коде уже является платформенным scope внутри `Tenant`, а не полноценной предметной производственной структурой. Целевое переименование в `Site` должно сохранить ту же платформенную семантику:

```text
TenantId
SiteId?
```

### Что уже принято в 00 Common / платформенных документах

В `00_common` уже принято:

- `TenantId` и `PlantId` не входят в универсальный `CommonObject`;
- `Tenant` является основной границей владения и изоляции данных;
- `Plant` является более узким контекстом площадки / завода и уровнем переопределений конфигурации;
- `PlantId` должен появляться только у объектов, где площадка входит в предметный смысл;
- для Common v1 отдельный runtime-helper `.Plant(...)` не вводится;
- если object type нужен `PlantId`, модуль описывает его как обычное предметное поле или ссылку и сам задает правила фильтрации, выбора и доступности.

При этом `00_common` также фиксирует, что `Plant` может иметь разные смыслы:

- область переопределения конфигурации;
- предметная ссылка на площадку;
- фильтр видимости;
- производственный контекст операции;
- часть маршрутизации или расписания.

Эти решения нужно синхронизировать с новым целевым термином `Site`.

### Варианты решения

Вариант A: `Tenant` = предприятие, `Plant` = производственная площадка, а `Plant` напрямую соответствует `ProductionUnit` с типом `Предприятие` или отдельным типом площадки.

Риск: смешиваются platform/security scope и предметная структура предприятия. Если одно предприятие имеет несколько площадок, юридических организаций или производственных комплексов, прямое равенство может оказаться слишком жестким.

Вариант B: `Tenant` = граница владения, изоляции и внедрения DMP; `Plant` = платформенный scope для площадочных переопределений, который может ссылаться на объект ПР03, но не является самим объектом ПР03.

Риск: нужна явная модель связи `Plant -> ProductionUnit`, иначе пользователи будут видеть два похожих справочника.

Вариант C: отказаться от отдельного платформенного `Plant` как бизнес-объекта и использовать `ProductionUnit` / `ProductionUnitType` для площадок, а платформенный `PlantId` заменить ссылкой на производственную единицу.

Риск: platform configuration, settings, IAM и runtime context начинают зависеть от прикладного модуля ПР03, что нарушает слои платформы.

Вариант D: считать `Plant` временным техническим scope-кодом ядра до проектирования ПР03 и в будущем мигрировать его на явную связь с производственной структурой.

Риск: требуется миграционное решение и совместимость существующих configuration scopes, settings, value set data и прав.

Вариант E: переименовать платформенный `Plant` в `Site`, а в ПР03 добавить / уточнить `ProductionUnitType.Site` / `Площадка`. Платформенный `Site` остается scope внутри `Tenant`, а предметная площадка является узлом `ProductionUnit` типа `Site`.

Риск: нужна миграция существующего кода, БД, конфигурационных scope и документации с `PlantId` на `SiteId`. Плюс нужно явно определить, как мигрируют существующие записи `Plant`.

### Предварительная рекомендация

Принять вариант E как целевую модель.

Не использовать `Plant` как целевой термин в новых проектных документах платформы. Целевой термин:

```text
Site / Площадка
```

Разделение понятий:

```text
Tenant
  платформа: граница владения, изоляции данных, доступа и конфигурации;
  предметная область: может соответствовать внедрению, группе предприятий или иному контуру данных, но не является производственной единицей ПР03.

Site
  платформа: опциональный scope внутри tenant для площадочных переопределений конфигурации, settings, отчетов, прав или видимости;
  код: целевое имя для текущего Plant / PlantId после миграции.

ProductionUnit
  ПР03: предметный объект производственной структуры с иерархией, типом и бизнес-правилами.

ProductionUnitType.Site
  ПР03: фиксированный тип производственной единицы, обозначающий площадку.
```

Если platform site scope используется, каждый `Site` должен быть однозначно связан с одной производственной единицей:

```text
Site -> ProductionUnit(type = Site / Площадка)
```

Обратное правило требует отдельного решения: не каждая `ProductionUnit` типа `Site` обязана немедленно иметь платформенный `Site`, если для нее не нужны отдельные права, настройки, видимость или reporting scope.

`Enterprise` / `Предприятие` не равно `Site` / `Площадка`. В пределах одного `Tenant` может быть одно или несколько предприятий, а предприятие может включать одну или несколько площадок:

```text
Tenant
  ProductionUnit: Предприятие 1
    ProductionUnit: Площадка A <-> Site A
    ProductionUnit: Площадка B <-> Site B
  ProductionUnit: Предприятие 2
    ProductionUnit: Площадка C <-> Site C
```

Термин `Plant` допустим только в миграционных примечаниях и при описании текущего состояния кода / старых источников.

### Что должно попасть в проектную документацию

- `01_scope.md`: ПР03 не владеет платформенными `Tenant` и `Site`; модуль владеет производственной структурой.
- `02_domain_model.md`: добавить / уточнить `ProductionUnitType.Site` / `Площадка` как фиксированный `SystemEnum`, если решение подтверждено.
- `02_domain_model.md`: описать связь производственной единицы типа `Site` с платформенным `Site` только как внешнюю ссылку / интеграционную связь, если она входит в предметную модель ПР03.
- `03_object_runtime_model.md`: описать, что `TenantId` является системной областью изоляции данных, а `SiteId` не добавляется автоматически всем объектам ПР03.
- `05_rules.md`: правила соответствия `Site` и `ProductionUnit(type = Site)`.
- `11_permissions.md`: не смешивать роли platform tenant-security и прикладные роли Plant Structure; описать только особенности доступа к объектам ПР03.
- `90_traceability_pr03.md`: зафиксировать решение о переименовании `Plant` в `Site` и связи `Site -> ProductionUnit(type = Site)`.

### Связанные документы и возможные доработки

Потребуется отдельная синхронизация с платформенными документами:

- Tenant and Security Platform;
- Configuration Platform;
- Settings;
- ValueSetData;
- Object Runtime;
- каталог стабильных кодов scope / object type;
- миграция кода, БД и документации с `Plant` / `PlantId` на `Site` / `SiteId`.

## Вопрос 15. Enum-поля `OrganizationalUnitInventoryParameters`

### Формулировка

Если `OrganizationalUnitInventoryParameters` входит во владение модуля производственной структуры, где должны быть определены enum-типы его полей?

### Что говорит ПР03 / DMP_DATA

Поля параметров места хранения относятся к `OrganizationalUnitInventoryParameters`.

### Что говорит ПР09

В ПР09 для этих полей указаны конкретные enum-типы и значения:

- `MaterialResponsibleControlType`, `WarehouseBinControlType`, `TransportUnitControlType` используют один enum `InventoryLocationControlType`;
- `WIPSerialNumbersControlStage` и `SegmentStageUsage` используют `WIPSegmentStage`;
- `TransferYieldDefaultType` использует `WIPActionTransferYieldType`;
- `TransferRejectDefaultType` использует `WIPActionTransferRejectType`;
- `TransferScrapDefaultType` использует `WIPActionTransferScrapType`.

### Решение

Enum-типы, используемые полями объектов модуля, по умолчанию фиксируются как SystemEnum модуля производственной структуры. Исключение — `InventoryLocationControlType`: этот тип уже в точности реализован и зарегистрирован как SystemEnum модуля `01_general_master_data`, поэтому ПР03 переиспользует его без повторного объявления.

`OrganizationalUnitInventoryParameters` остается объектом модуля. Собственные enum-типы его полей определяются в ПР03:

- `WIPSegmentStage`;
- `WIPActionTransferYieldType`;
- `WIPActionTransferRejectType`;
- `WIPActionTransferScrapType`.

`WIPSegmentStage` является флаговым SystemEnum. Поле `SegmentStageUsage` хранится как одно скалярное числовое значение — битовая маска выбранных стадий; в C# enum помечается `[Flags]`, а в baseline регистрируется с `IsFlags = true`. `WIPSerialNumbersControlStage` использует одно значение этого enum и не является маской.

Поля `MaterialResponsibleControlType`, `WarehouseBinControlType` и `TransportUnitControlType` ссылаются на GMD SystemEnum `InventoryLocationControlType` со значениями `None = 0`, `Enable = 1`, `Mandatory = 2`.

Для будущих ссылок на `InventoryStatus` в физической модели v1 сохраняются пять nullable GUID-колонок без FK на отсутствующий целевой объект. До появления стабильного контракта модуля производственной логистики они не публикуются в Object Runtime, baseline и UI и не проверяются правилом `PS-ST-004`.

### Что попадет в проектную документацию

- `02_domain_model.md`: полный состав собственных SystemEnum, типы всех enum-полей и внешнее владение `InventoryLocationControlType` модулем GMD.
- `10_value_set_data_usage.md`: указание, что эти SystemEnum не являются `ValueSet`.
- `03_object_runtime_model.md`: источники выбора собственных enum идут из SystemEnum ПР03, `InventoryLocationControlType` — из SystemEnum GMD; будущие ссылки `InventoryStatus` остаются непубличными nullable GUID-колонками до появления контракта производственной логистики.

## Вопрос 16. Влияет ли состояние Workflow на выбор места хранения

### Формулировка

Должны ли `StorageArea.Warehouse`, `WarehouseBin.Warehouse` и поля `*Warehouse` исключать организационные единицы в состоянии `Draft`?

### Решение

Нет. Формулировки назначения состояний и переходов в `04_workflows.md` относятся к жизненному циклу самой организационной единицы и сами по себе не задают условия списков выбора.

Место хранения доступно в lookup, если это активная `ProductionUnit` или `Subcontractor` текущего tenant с `IsInventoryStorageLocation = true` и пользователь имеет право чтения конкретного типа. Состояния `Draft` и `Published` одинаково допустимы. Архивные и удаленные записи недоступны по стандартным правилам активности.

Любое будущее ограничение ссылочного поля по состоянию Workflow должно быть введено отдельным явным предметным правилом и отражено одновременно в `03_object_runtime_model.md`, `05_rules.md` и `06_ui_views.md`.

### Что попадет в проектную документацию

- `01_scope.md`: граница доступных мест хранения без фильтра по Workflow;
- `03_object_runtime_model.md`: явные условия lookup всех полей места хранения;
- `04_workflows.md`: разграничение жизненного цикла и фильтров ссылочных полей;
- `05_rules.md`: уточнение `PS-ST-001`;
- `06_ui_views.md`: источник и фильтры lookup места хранения.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.2 | 2026-09-02 11:01 +04:00 | Codex | Вопрос 16 | Зафиксирована независимость lookup места хранения от состояния Workflow | — |
| 1.1 | 2026-08-31 22:00 +04:00 | A-Zhigalin (@A-Zhigalin) | Что говорит ПР03; Решение; Что должно попасть в проектную документацию; Что попадет в проектную документацию | Уточнения документации | [cb99e83e](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/cb99e83e553f9dc034dd6808f07d230b09ceed28) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-14 12:39 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [abbdee0a](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/abbdee0a5a61f2bb2fa49cfafd1858b7100cbd8a) |
