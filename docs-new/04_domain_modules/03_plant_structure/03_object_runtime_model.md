---
id: DOC-04-03-03
title: 'Runtime-модель объектов - 03 Производственная структура'
type: design
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
updated_at: 2026-09-02 11:50
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Runtime-модель объектов - 03 Производственная структура

## 1. Назначение документа

Документ описывает, как объекты модуля `03 Plant Structure` доступны через Business Object Runtime / Object Runtime.

Документ не повторяет доменную модель. Полный состав объектов, поля, русские названия, типы, обязательность и связи описаны в `02_domain_model.md`.

Компоновка списков и карточек описывается в `06_ui_views.md`. Инварианты и проверки описываются в `05_rules.md`. Жизненный цикл объектов описывается в `04_workflows.md`.

## 2. Базовое решение

Основной способ работы с объектами модуля:

```text
Business Object Runtime / Object Runtime
```

Ручные API-методы для стандартных действий чтения, создания, изменения и удаления не вводятся.

Object Runtime должен:

- публиковать типы объектов модуля;
- применять `TenantId` из контекста;
- применять Common-поля, архивирование и мягкое удаление;
- проверять стандартные права на чтение, создание, изменение и удаление;
- поддерживать абстрактный ссылочный тип `OrganizationalUnit`;
- поддерживать конкретные типы `ProductionUnit` и `Subcontractor` поверх общего хранения;
- поддерживать иерархические списки для `ProductionUnit` и `WarehouseBin`;
- поддерживать зависимый объект `OrganizationalUnitInventoryParameters`;
- вычислять представление объектов по правилам типа объекта;
- вызывать валидаторы модуля из `05_rules.md`.

## 3. Публикуемые типы объектов

| Тип объекта | Русское название | Роль в Object Runtime |
|---|---|---|
| `OrganizationalUnit` | Организационная единица | Абстрактный ссылочный тип. Используется в ссылочных полях и списках выбора, но не создается напрямую. |
| `ProductionUnit` | Производственная единица | Самостоятельный тип объекта для собственных элементов производственной структуры. |
| `Subcontractor` | Субподрядчик | Самостоятельный тип объекта для внешних элементов производственной структуры. |
| `Company` | Организация | Самостоятельный справочник балансовых организаций. |
| `OrganizationalUnitInventoryParameters` | Параметры места хранения | Зависимый объект места хранения. |
| `StorageArea` | Зона хранения | Самостоятельный справочник зон хранения. |
| `WarehouseBinType` | Тип складской ячейки | Самостоятельный справочник типов складских ячеек. |
| `WarehouseBin` | Складская ячейка | Самостоятельный объект складской ячейки. |

`PlantParameters` и `ProductionUnitOperationParameters` не публикуются как типы объектов модуля.

## 4. Abstract/reference-only модель OrganizationalUnit

`OrganizationalUnit` является полноценным типом доменной модели, но не является типом, по которому пользователь создает записи напрямую.

Object Runtime должен поддержать для `OrganizationalUnit` режим:

```text
abstract/reference-only
```

Требуемое поведение:

| Сценарий | Поведение |
|---|---|
| Создание | Создание `OrganizationalUnit` напрямую запрещено. Пользователь создает `ProductionUnit` или `Subcontractor`. |
| Чтение по ссылке | Ссылка на `OrganizationalUnit` должна разрешаться в конкретный объект по `UnitKind`. |
| Список выбора | Список выбора `OrganizationalUnit` показывает записи `ProductionUnit` и `Subcontractor` в одном наборе, если поле допускает оба типа. |
| Права | Доступ к строке определяется правами на конкретный тип: `ProductionUnit` или `Subcontractor`. |
| Представление | Представление берется из правила конкретного типа. |
| Baseline | Конфигурация должна объявлять абстрактный тип, дискриминатор `UnitKind` и контракты самостоятельных типов. |

Object Runtime должен поддерживать этот режим. Платформенная доработка abstract/reference-only и дискриминатора ведется в `plans/020_object_runtime_remediation_backlog.md`.

## 5. Tenant и Site

Все объекты модуля принадлежат `Tenant`.

`TenantId` назначается Object Runtime:

- при создании корневого объекта берется из контекста;
- при создании зависимого объекта наследуется от владельца;
- при выборе ссылочного объекта ограничивается тем же `Tenant`;
- в пользовательских карточках не редактируется.

Платформенный `Site` не является доменным объектом модуля. Если для производственной площадки требуется платформенная область прав, настроек или видимости, связь платформенного `Site` с `ProductionUnit` типа `Site` реализуется платформой и не добавляет поле в объекты модуля.

## 6. Представление объектов

`Presentation` не является физическим полем модуля.

Object Runtime вычисляет представление по правилу типа объекта. Если правило использует несколько частей, пустые необязательные части и лишние пробелы не выводятся.

| Тип объекта | Правило представления |
|---|---|
| `ProductionUnit` | `{Code}{пробел если Code заполнен}{Name}` |
| `Subcontractor` | `{Code}{пробел если Code заполнен}{Name}` |
| `Company` | `{Code}{пробел если Code заполнен}{Name}` |
| `StorageArea` | `{Code}{пробел если Code заполнен}{Name}` |
| `WarehouseBinType` | `{Code}{пробел если Code заполнен}{Name}` |
| `WarehouseBin` | `{Identification}` |
| `OrganizationalUnitInventoryParameters` | `{OrganizationalUnit.Presentation}` |

Для ссылок на `OrganizationalUnit` Object Runtime показывает представление конкретной записи `ProductionUnit` или `Subcontractor`.

Хранить `Presentation` в таблицах модуля не требуется.

## 7. Наборы данных Object Runtime

Для самостоятельных типов объектов baseline должен объявлять стандартные наборы данных:

| Набор данных | Назначение | Применяется к |
|---|---|---|
| `List` | Основной список типа объекта. | `ProductionUnit`, `Subcontractor`, `Company`, `StorageArea`, `WarehouseBinType`, `WarehouseBin` |
| `Lookup` | Выбор объекта в ссылочном поле. | Все самостоятельные типы, а также `OrganizationalUnit` как abstract lookup. |
| `Details` | Чтение объекта для карточки. | Все самостоятельные типы. |
| `TreeList` | Иерархический список. | `ProductionUnit` и `WarehouseBin` по `Parent`. |

`OrganizationalUnitInventoryParameters` не имеет самостоятельного пользовательского списка в v1. Он читается и сохраняется как зависимый объект карточки `ProductionUnit` или `Subcontractor`.

## 8. Иерархии

`ProductionUnit` и `WarehouseBin` объявляют hierarchy capability Object Runtime.

| Объект | Parent-связь | Root | Уровень | Путь | Поведение корней |
|---|---|---|---|---|---|
| `ProductionUnit` | `Parent` | `Main` | `Level` | `Path` | Несколько корневых `Enterprise` в одном `Tenant`. |
| `WarehouseBin` | `Parent` | `Root` | `Level` | `Path` | Несколько корневых ячеек в одном месте хранения. |

Object Runtime отвечает за:

- запрет циклов;
- расчет root-ссылки, уровня и materialized path;
- пересчет поддерева при изменении parent-связи;
- вычисление виртуального поля `HasChildren` без хранения отдельной колонки;
- поддержку выборки корневых узлов, дочерних узлов и поддерева для tree UI.

`Main` / `Root`, `Level` и `Path` являются явно объявленными физическими полями доменной модели, но не редактируются пользователем. `HasChildren` объявляется в object contract как виртуальное вычисляемое поле. Для materialized path и корневых ссылок создаются индексы в соответствии с общим шаблоном hierarchy capability.

Для `WarehouseBin` признак `HasChildren` используется только для навигации Tree UI и проверок изменения структуры. Он не ограничивает использование ячейки в складских операциях: lookup и прикладные проверки не должны фильтровать ячейки по условию `HasChildren = false`.

Общая hierarchy capability Object Runtime ведется в `plans/026_object_runtime_hierarchy_support_backlog.md`.

## 9. Объекты и выбор значений ссылочных полей

Выбор ссылочных объектов должен использовать стандартный механизм Object Runtime.

Общие правила:

- применяется текущий `Tenant`;
- доступность внешнего объекта проверяется через права модуля-владельца.

Ссылочные поля и сами связи описаны в `02_domain_model.md`. В таблицах ниже указаны только ограничения выбора сверх общих правил.

### OrganizationalUnit

`OrganizationalUnit` является abstract/reference-only contract. Ссылки на `OrganizationalUnit` разрешаются в конкретные записи `ProductionUnit` или `Subcontractor` по `UnitKind`.

Ссылочные поля:

Нет ограничений сверх общих правил Object Runtime.

### ProductionUnit

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Main` | `ProductionUnit` | Заполняется hierarchy capability. Значение равно корневой `ProductionUnit` типа `Enterprise` в текущем дереве. |
| `Parent` | `ProductionUnit` | Выбираются производственные единицы текущего `Tenant`; запрет выбора самого объекта или потомка выполняет hierarchy capability. Фильтр совместимости по `ProductionUnitType` не применяется, кроме запрета дочернего `Enterprise`. |
| `Company` | `Company` | Организации текущего `Tenant`. |

### Subcontractor

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Contractor` | `Contractor` из `01_general_master_data` | Контрагенты текущего `Tenant`; обязательность не вводится. |

### Company

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Contractor` | `Contractor` из `01_general_master_data` | Контрагенты текущего `Tenant`; обязательность не вводится. |

### OrganizationalUnitInventoryParameters

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `OrganizationalUnit` | `OrganizationalUnit` | Заполняется системой владельцем карточки; выбираются только активные `OrganizationalUnit` с `IsInventoryStorageLocation = true`, независимо от состояния Workflow. |
| `ReleaseWarehouse` / `IssueWarehouse` / `OutputWarehouse` / `ScrapWarehouse` | `OrganizationalUnit` | Выбираются только активные `OrganizationalUnit` с `IsInventoryStorageLocation = true`, независимо от состояния Workflow. |
| `ReleaseWarehouseBin` / `IssueWarehouseBin` / `OutputWarehouseBin` / `ScrapWarehouseBin` | `WarehouseBin` | Lookup недоступен, пока не заполнено соответствующее поле места хранения. После его выбора доступны только ячейки этого места хранения. |

Для любой пары `*Warehouse` / `*WarehouseBin` серверная валидация запрещает сохранять ячейку без соответствующего места хранения или с ячейкой другого места хранения. При очистке или замене места хранения UI очищает ранее выбранную ячейку; после замены пользователь выбирает ячейку заново из lookup нового места хранения.

Иных условных зависимостей между полями `OrganizationalUnitInventoryParameters` Object Runtime v1 не применяет. В частности, `Transfer*DefaultType` не управляют обязательностью, доступностью или очисткой мест хранения и остальных настроек. Object Runtime проверяет только явно определенные правила `PS-IP-*`; содержательную согласованность остальных сочетаний значений обеспечивает пользователь до уточнения логики на последующих этапах.

Поля `MaterialResponsibleControlType`, `WarehouseBinControlType` и `TransportUnitControlType` используют существующий SystemEnum `InventoryLocationControlType` модуля `01_general_master_data`. Остальные enum-поля `OrganizationalUnitInventoryParameters` используют системные enum ПР03, описанные в `02_domain_model.md`. Модуль не объявляет собственные `ValueSet` для этих полей.

Поля `InventoryStatus`, `TransferYieldDefaultStatus`, `TransferRejectDefaultStatus`, `TransferScrapDefaultStatus` и `WarehouseBin.InventoryStatus` в v1 существуют только как nullable GUID-колонки физической модели. Они не объявляются как members object contract, не включаются в baseline, datasets, формы и lookup и не принимаются в mutation payload. После появления стабильного контракта `InventoryStatus` модуль публикует их как reference members и добавляет проверку значений без миграции самих колонок.

`WIPSegmentStage` объявляется в C# с атрибутом `[Flags]`, а в baseline SystemEnum — с признаком `IsFlags = true` (`.Flags()`). `SegmentStageUsage` является одним скалярным числовым полем с битовой маской; отдельная коллекция или связующая таблица для стадий не создается. `WIPSerialNumbersControlStage` использует тот же enum как одиночное, некомбинируемое значение.

### StorageArea

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Warehouse` | `OrganizationalUnit` | Выбираются только активные `OrganizationalUnit` с `IsInventoryStorageLocation = true`, независимо от состояния Workflow. |

### WarehouseBin

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Warehouse` | `OrganizationalUnit` | Выбираются только активные `OrganizationalUnit` с `IsInventoryStorageLocation = true`, независимо от состояния Workflow. |
| `StorageArea` | `StorageArea` | Выбираются зоны хранения того же `Warehouse`. |
| `Parent` | `WarehouseBin` | Выбираются ячейки того же `Warehouse`; запрет выбора самой ячейки или потомка выполняет hierarchy capability. |

## 10. Коллекции и зависимые объекты

Коллекции Object Runtime:

| Владелец | Коллекция | Тип элемента | Режим | Комментарий |
|---|---|---|---|---|
| `OrganizationalUnit` | `InventoryParameters` | `OrganizationalUnitInventoryParameters` | Состав владельца | Зависимая запись 1:0..1 для места хранения. |
| `ProductionUnit` | `Children` | `ProductionUnit` | Навигация | Дочерние производственные единицы по `Parent`. Дочерний объект редактируется как самостоятельная `ProductionUnit`. |
| `WarehouseBin` | `Children` | `WarehouseBin` | Навигация | Дочерние складские ячейки по `Parent`. Дочерняя ячейка редактируется как самостоятельный объект. |
| `OrganizationalUnit` | `WorkSchedules` | `ResourceManagement:OrganizationalUnitWorkSchedule` | Навигация | Назначения графиков из модуля управления ресурсами, отфильтрованные по текущему `OrganizationalUnit`. Для `ProductionUnit` и `Subcontractor` применяются ограничения `UnitKind`. Записи принадлежат модулю управления ресурсами. Модуль 03 не создает, не изменяет и не удаляет их. |
| `ProductionUnit` | `OperationParameters` | `ResourceManagement:ProductionUnitOperationParameters` | Навигация | Параметры операций из модуля управления ресурсами, отфильтрованные по текущему `ProductionUnit`. Запись принадлежит модулю управления ресурсами. Модуль 03 не создает, не изменяет и не удаляет ее. |

UI-вкладка не создает коллекцию доменной модели сама по себе. Если карточка показывает связанный список по условию, владение определяется полем связи и правилами доменной модели. Для `WorkSchedules` модуль 03 пока описывает только runtime-навигацию; отдельная UI-вкладка в v1 не вводится.

## 11. Обработчики Object Runtime

Object Runtime должен вызывать обработчики модуля в стандартных точках создания и сохранения объекта.

| Объект | Событие | Поведение |
|---|---|---|
| `ProductionUnit` | BeforeCreate / BeforeSave | Установить `UnitKind = ProductionUnit`; проверить предметную совместимость `Type`, `Parent`, `Main`; при изменении `IsInventoryStorageLocation` с `true` на `false` проверить отсутствие неархивированных и неудаленных зон и ячеек ПР03. |
| `ProductionUnit` | AfterSave | Синхронизировать зависимые параметры места хранения по `IsInventoryStorageLocation`. |
| `Subcontractor` | BeforeCreate / BeforeSave | Установить `UnitKind = Subcontractor`; при изменении `IsInventoryStorageLocation` с `true` на `false` проверить отсутствие неархивированных и неудаленных зон и ячеек ПР03. |
| `Subcontractor` | AfterSave | Синхронизировать зависимые параметры места хранения по `IsInventoryStorageLocation`. |
| `StorageArea` | BeforeSave | Проверить, что `Warehouse` является местом хранения. При изменении `Warehouse` проверить отсутствие неархивированных и неудаленных `WarehouseBin`, ссылающихся на зону. |
| `WarehouseBin` | BeforeSave | Проверить место хранения, принадлежность зоны и родительской ячейки тому же месту хранения. При изменении `Warehouse` проверить отсутствие неархивированных и неудаленных дочерних ячеек; `Parent` и `StorageArea` должны быть пустыми либо одновременно изменены на объекты нового места хранения. |
| `OrganizationalUnitInventoryParameters` | BeforeSave | Проверить, что владелец является местом хранения; проверить согласованность мест хранения и ячеек; при автоматическом создании заполнить значения по умолчанию. |

Синхронизация параметров места хранения выполняется одинаково для `ProductionUnit` и `Subcontractor`: если `IsInventoryStorageLocation = true`, Object Runtime обеспечивает наличие одной записи `OrganizationalUnitInventoryParameters` и заполняет обязательные поля значениями по умолчанию, определенными в `02_domain_model.md`; при изменении признака с `true` на `false` Object Runtime до сохранения проверяет отсутствие неархивированных и неудаленных `StorageArea` и `WarehouseBin` ПР03, относящихся к организационной единице. Если проверка не пройдена, изменение отклоняется и значение признака не изменяется. Если проверка пройдена, признак устанавливается в `false`, а связанная запись `OrganizationalUnitInventoryParameters` удаляется в рамках той же операции сохранения. Входящие зависимости других модулей в v1 не проверяются.

Прямые create, delete и archive для `OrganizationalUnitInventoryParameters` не публикуются. Физическое удаление выполняется только системной синхронизацией владельца; soft delete и Common archive для зависимой записи не применяются.

Пересчет `Main`, уровня, пути и признака наличия дочерних узлов выполняется общей hierarchy capability Object Runtime, а не обработчиками модуля.

Подробные условия проверок и сообщения об ошибках описываются в `05_rules.md`.

## 12. Стандартные операции и прикладные действия

Стандартные действия чтения, создания, изменения и удаления выполняются через Object Runtime.

Собственные прикладные действия Object Runtime для модуля v1 не вводятся.

Жизненные циклы описываются только для объектов, где они нужны по предметной модели. Их действия описаны в `04_workflows.md`.

## 13. Baseline и платформенные требования

Baseline модуля должен объявить:

- object type metadata для всех публикуемых типов;
- system enum metadata для перечислений модуля;
- `OrganizationalUnit` как abstract/reference-only object type;
- контракты самостоятельных типов `ProductionUnit` и `Subcontractor`;
- дискриминатор `UnitKind`;
- правила `Presentation`;
- наборы данных `List`, `Lookup`, `Details`, `TreeList`;
- hierarchy capability для `ProductionUnit` и `WarehouseBin`;
- физические hierarchy members `ProductionUnit.Main`, `ProductionUnit.Level`, `ProductionUnit.Path`, `WarehouseBin.Root`, `WarehouseBin.Level`, `WarehouseBin.Path` и виртуальные members `HasChildren`;
- коллекцию `OrganizationalUnit.InventoryParameters`;
- обработчики создания, сохранения и удаления зависимых параметров;
- ограничения выбора ссылочных полей.

Поддержка abstract/reference-only object type поверх одной таблицы требуется для реализации `OrganizationalUnit`, `ProductionUnit` и `Subcontractor`. Платформенная доработка ведется в `plans/020_object_runtime_remediation_backlog.md`.

Поддержка hierarchy capability Object Runtime требуется для реализации иерархий `ProductionUnit` и `WarehouseBin`. Платформенная доработка ведется в `plans/026_object_runtime_hierarchy_support_backlog.md`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.2 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | Abstract/reference-only модель OrganizationalUnit; Коллекции и зависимые объекты; Baseline и платформенные требования | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.1 | 2026-08-31 22:00 +04:00 | A-Zhigalin (@A-Zhigalin) | Иерархии; ProductionUnit; OrganizationalUnitInventoryParameters; WarehouseBin; Обработчики Object Runtime; Baseline и платформенные требования | Уточнения документации | [cb99e83e](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/cb99e83e553f9dc034dd6808f07d230b09ceed28) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-14 12:39 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [abbdee0a](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/abbdee0a5a61f2bb2fa49cfafd1858b7100cbd8a) |
