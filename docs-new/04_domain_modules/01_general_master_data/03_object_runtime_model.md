---
id: DOC-04-01-03
title: 'Runtime-модель объектов - 01 General Master Data'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 01_general_master_data
holder: '@axelprosoft'
created_at: 2026-08-07 10:40
created_by: '@axelprosoft'
updated_at: 2026-09-02 11:50
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Runtime-модель объектов - 01 General Master Data

## 1. Назначение документа

Документ описывает, как объекты `01 General Master Data` становятся доступными через Object Runtime.

Документ не повторяет доменную модель. Полный состав объектов, поля, русские названия, типы, обязательность, связи и коллекции описаны в `02_domain_model.md`.

Компоновка списков и карточек описывается в `06_ui_views.md`. Инварианты и проверки описываются в `05_rules.md`. Жизненный цикл объектов описывается в `04_workflows.md`.

## 2. Базовое решение

GMD не проектирует отдельный слой API, копирующий API ПР01.

Основной способ работы с объектами GMD:

```text
Business Object Runtime / Object Runtime
```

Ручные API-методы допускаются только для операций, которые нельзя выразить стандартными возможностями Object Runtime.

Object Runtime должен:

- публиковать типы объектов GMD;
- применять область предприятия;
- применять Common-поля, архивирование и мягкое удаление;
- проверять права на чтение, создание, изменение, удаление и действия;
- поддерживать ссылки и списки выбора;
- поддерживать коллекции Object Runtime там, где они входят в модель GMD;
- вызывать валидаторы GMD из `05_rules.md`;
- вычислять представление объектов по правилам типа объекта.

## 3. Публикуемые типы объектов

| Тип объекта | Русское название | Базовый тип | Роль в Object Runtime |
|---|---|---|---|
| `Unit` | Единица измерения | `CommonCatalogObject` | Корневой справочник. |
| `UnitGroup` | Группа ЕИ | `CommonCatalogObject` | Корневой справочник. |
| `UnitOfOperation` | Единица измерения операции | `CommonObject` | Строка коллекции или объект пересчета с владельцем. |
| `NomenclatureGroup` | Группа номенклатуры | `CommonCatalogObject` | Иерархический корневой справочник. |
| `NomenclatureKind` | Вид номенклатуры | `CommonCatalogObject` | Корневой справочник схемы реквизитов НП. |
| `Nomenclature` | Номенклатурная позиция | `CommonCatalogObject` | Основной корневой справочник GMD. |
| `NomenclatureVariant` | Исполнение НП | `CommonCatalogObject` | Исполнение НП, связанное с владельцем `Nomenclature`. |
| `PackSet` | Набор упаковок | `CommonCatalogObject` | Корневой справочник наборов упаковок. |
| `Pack` | Упаковка | `CommonCatalogObject` | Упаковка: типовая или индивидуальная. |
| `TransportUnitType` | Тип единицы транспортировки | `CommonCatalogObject` | Корневой справочник типов единиц транспортировки. |
| `SerialNumber` | Серийный номер | `CommonObject` | Реестр серийных номеров НП. |
| `NomenclatureParameter` | Реквизит номенклатуры | `CommonObject` | Корневой объект схемы реквизитов. |
| `NomenclatureParameterAssignment` | Назначение реквизита номенклатуры | `CommonObject` | Строка назначения реквизита. |
| `NomenclatureParameterEnumValue` | Значение перечислимого реквизита номенклатуры | `CommonObject` | Строка локального списка значений реквизита. |
| `NomenclatureParameterValue` | Строка значения реквизита НП | `NomenclatureParameterValueBase` | Строка коллекции `Nomenclature.ParameterValues`. |
| `NomenclatureVariantParameterValue` | Строка значения реквизита исполнения НП | `NomenclatureParameterValueBase` | Строка коллекции `NomenclatureVariant.ParameterValues`. |
| `SerialNumberParameterValue` | Строка значения реквизита серийного номера | `NomenclatureParameterValueBase` | Строка коллекции `SerialNumber.ParameterValues`. |
| `Contractor` | Контрагент | `CommonCatalogObject` | Корневой справочник. |
| `ContractorGroup` | Группа контрагентов | `CommonCatalogObject` | Корневой справочник. |
| `ContractorInContractorGroup` | Включение контрагента в группу | `CommonObject` | Строка связи группы и контрагента. |
| `ContractorContact` | Контакт контрагента | `CommonObject` | Строка коллекции `Contractor.Contacts`. |

`NomenclatureParameterValueBase` является базовым типом элементов значений и не является самостоятельным типом объекта для пользовательской работы.

## 4. Общие правила Object Runtime

Все типы объектов GMD имеют `TenantId`.

Object Runtime должен централизованно применять область предприятия:

- списки показывают данные текущего предприятия;
- создание корневого объекта берет `TenantId` из контекста Object Runtime;
- создание строки коллекции наследует `TenantId` владельца;
- выбор ссылочного объекта ограничивается тем же предприятием;
- пользовательская форма не редактирует `TenantId`.

`SiteId` не является общей областью применения для GMD v1.

Для всех типов объектов GMD на базе `CommonObject` применяются правила `00_common`:

- системные поля Common не редактируются обычными формами создания и изменения;
- архивные и удаленные записи скрываются по умолчанию;
- уже сохраненная ссылка на архивный объект должна отображаться;
- новые ссылки на архивный объект запрещены;
- секция `Администрирование` добавляется по правилам `00_common/03_object_runtime_model.md`.

GMD использует Common-модель архивирования и удаления. Наличие полей архива или удаления не открывает действие автоматически. Доступность действий определяется исходным описанием типа объекта.

Если тип объекта управляется жизненным циклом, архивирование должно выполняться через действия жизненного цикла. Для одного типа объекта нельзя смешивать прямое архивирование и архивирование через жизненный цикл.

## 5. Представление объектов

`Presentation` не является физическим полем GMD.

В этом документе `Presentation` означает представление объекта, вычисленное Object Runtime для ссылок, списков выбора, списков и заголовков.

Object Runtime должен вычислять представление по правилу типа объекта. Если правило использует несколько частей, пустые необязательные части и лишние пробелы не выводятся.

| Тип объекта | Правило представления | Источник |
|---|---|---|
| `Unit` | `{Acronym}` | ПР01 |
| `UnitGroup` | `{Code}{пробел если Code заполнен}{Name}` | ПР01 |
| `NomenclatureGroup` | `{Code}{пробел если Code заполнен}{Name}` | ПР01 |
| `NomenclatureKind` | `{Code}{пробел если Code заполнен}{Name}` | ПР01 |
| `Nomenclature` | `{Code}{пробел если Code заполнен}{DrawingNumber}{пробел если DrawingNumber заполнен}{Name}` | ПР01 |
| `NomenclatureVariant` | `{Code}{пробел если Code заполнен}{Name}` | ПР01 |
| `SerialNumber` | `{SerialNumber}` | ПР01 |
| `NomenclatureParameter` | `{DisplayName ?? Name}` | ПР01; `DisplayName` по умолчанию равен `Name`. |
| `NomenclatureParameterEnumValue` | `{Name}` | ПР01 |
| `ContractorGroup` | `{Code}{пробел если Code заполнен}{Name}` | ПР01 |
| `Contractor` | `{Code}{пробел если Code заполнен}{Name}` | ПР01 |
| `ContractorContact` | `{ContactType}-{FullName}({JobPosition})` | ПР01, с заменой `Name` на целевое поле `FullName`. |
| `PackSet` | `{Code}{пробел если Code заполнен}{Name}` | Целевое правило GMD; ПР01 отсылает упаковки к ПР09. |
| `Pack` | `{Code}{пробел если Code заполнен}{Name}` | Целевое правило GMD; ПР01 отсылает упаковки к ПР09. |
| `TransportUnitType` | `{Code}{пробел если Code заполнен}{Name}` | Целевое правило GMD; ПР01 отсылает типы единиц транспортировки к ПР09. |
| `NomenclatureParameterAssignment` | `{NomenclatureParameter.Presentation}` | Целевое правило GMD для строки назначения. |
| `NomenclatureParameterValue` | `{AttributeCode}: {ValuePresentation}` | Целевое правило GMD для строки значения. |
| `NomenclatureVariantParameterValue` | `{AttributeCode}: {ValuePresentation}` | Целевое правило GMD для строки значения. |
| `SerialNumberParameterValue` | `{AttributeCode}: {ValuePresentation}` | Целевое правило GMD для строки значения. |
| `ContractorInContractorGroup` | `{Contractor.Presentation}` | Целевое правило GMD для строки связи. |

Простые правила, которые состоят из полей того же объекта, задаются в baseline-описании типа объекта через `Presentation.DisplayName(...).Fields(...).Virtual()`.

Для `Nomenclature` целевое baseline-описание должно задавать поля в порядке из ПР01:

```csharp
.Presentation(presentation => presentation
    .DisplayName(display => display
        .Fields(
            GmdCodes.Nomenclature.Fields.Code,
            GmdCodes.Nomenclature.Fields.DrawingNumber,
            GmdCodes.Nomenclature.Fields.Name)
        .Virtual()))
```

Хранить `Presentation` в таблицах GMD по умолчанию не требуется.

`Presentation` хранится в БД только отдельным проектным решением. Основания для такого решения:

- индексировать или сортировать большие списки по строке представления;
- хранить исторический снимок представления;
- отдавать внешний экспорт с зафиксированным на момент операции текстом;
- снять дорогие вычисления с частых запросов списков выбора.

Правила `UnitOfOperation` из ПР01:

| Условие | Правило представления |
|---|---|
| Заполнен `UnitGroup`, `Nomenclature` или `Lot` | `{ToUnit.Presentation}` |
| Заполнен `TransportUnitType` | `{TransportUnitType.Presentation}` |
| Заполнен `Pack` и `ToUnit` | `{Pack.PackUnit.Presentation} ({ConversionFactor} {ToUnit.Presentation})` |
| Заполнен `Pack` и `ToPack` | `{Pack.PackUnit.Presentation} ({ConversionFactor} {ToPack.Presentation})` |

`ValuePresentation` означает представление значения реквизита. Оно определяется по типу значения:

- строка, число, дата и логическое значение показываются напрямую;
- `LocalEnum` разрешается через `NomenclatureParameterEnumValue`;
- `ValueSet` разрешается через Platform ValueSetData.

## 6. Списки, списки выбора и поиск

Для корневых типов объектов GMD baseline должен объявлять пользовательские списки, карточки и списки выбора в объеме, описанном в `06_ui_views.md`.

Контракт объекта должен объявлять наборы данных Object Runtime:

| Набор данных | Назначение | Минимальные возможности |
|---|---|---|
| `List` | Основной список типа объекта. | Фильтрация, сортировка; группировка там, где она нужна списку. |
| `Lookup` | Выбор объекта в ссылочном поле. | Фильтрация и сортировка. |
| `Details` | Чтение объекта для карточки. | По `Id` и правам доступа. |

Поиск по представлению не требует физического поля `Presentation`.

Для списков и списков выбора должны настраиваться параметры поиска по исходным полям, из которых строится представление объекта. Для `Nomenclature` минимальный набор полей поиска:

```text
Code
DrawingNumber
Name
```

То есть пользователь видит строку `Presentation`, но поиск выполняется по `Code`, `DrawingNumber`, `Name`.

Для ссылочных полей действует то же правило:

- список выбора НП ищет по полям поиска самой `Nomenclature`;
- список объекта, в котором есть ссылка на НП, ищет по полям отображения целевой `Nomenclature`: `Code`, `DrawingNumber`, `Name`;
- для больших справочников поиск должен выполняться на уровне хранилища по полям целевого объекта, а не пост-фильтрацией ограниченной страницы.

Поиск по полной строке представления оформляется как поисковая проекция списка. Хранимое поле отображения не добавляется в доменную модель GMD.

## 7. Объекты и выбор значений ссылочных полей

Выбор ссылочных объектов должен использовать стандартный механизм Object Runtime.

Общие правила выбора:

- по умолчанию показываются только неархивные и неудаленные объекты;
- область предприятия применяется автоматически;
- доступность внешних ссылок подтверждает модуль-владелец;
- предметные ограничения GMD проверяются валидаторами из `05_rules.md`.

Ссылочные поля и сами связи описаны в `02_domain_model.md`. В таблицах ниже указаны только ограничения выбора сверх общих правил.

### UnitOfOperation

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `UnitGroup` / `Nomenclature` / `Lot` / `Pack` / `TransportUnitType` | `UnitGroup`, `Nomenclature`, external `Lot`, `Pack`, `TransportUnitType` | Должен быть заполнен ровно один владелец пересчета. Допустимые пары владельца и цели задает матрица `05_rules.md`. |
| `ToUnit` / `ToPack` / `ToNomenclature` | `Unit`, `Pack`, `Nomenclature` | Должна быть заполнена ровно одна цель пересчета, допустимая для выбранного владельца. |
| `ApplicationArea` / `IsDefault` | `UnitApplicationAreaFlags`, `Boolean` | Единица операции по умолчанию определяется в разрезе владельца и области применения. `ApplicationArea` хранит флаги `UnitApplicationAreaFlags`; пустое значение означает все области. Пересечение областей применения у активных записей по умолчанию одного владельца запрещено. |

### Nomenclature

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `BaseManufacturer` | `Contractor` | Выбирается `Contractor`, у которого `Category` содержит флаг `Manufacturer`. |
| `PackSet` | `PackSet` | Выбираются только наборы упаковок, у которых `BaseUnitId` равен `Nomenclature.UnitId`. |
| `ReleaseWarehouseBin` | external `WarehouseBin` | Если заполнен `ReleaseWarehouse`, ячейка должна принадлежать выбранному месту хранения; проверку выполняет модуль-владелец складской структуры. |
| `DeliveryWarehouseBin` | external `WarehouseBin` | Если заполнен `DeliveryWarehouse`, ячейка должна принадлежать выбранному месту хранения; проверку выполняет модуль-владелец складской структуры. |
| `ReceiptWarehouseBin` | external `WarehouseBin` | Если заполнен `ReceiptWarehouse`, ячейка должна принадлежать выбранному месту хранения; проверку выполняет модуль-владелец складской структуры. |

### NomenclatureVariant

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Nomenclature` | `Nomenclature` | Исполнение принадлежит ровно одной НП. |

### Pack

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `PackSet` / `Nomenclature` | `PackSet`, `Nomenclature` | Для упаковки должен быть заполнен ровно один владелец: `PackSet` или `Nomenclature`. |

### TransportUnitType

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `PackageNomenclature` | `Nomenclature` | Выбирается существующая НП текущего предприятия. Отдельный признак упаковочной НП в GMD v1 не вводится. |

### SerialNumber

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `NomenclatureVariant` | `NomenclatureVariant` | Исполнение должно принадлежать выбранной `Nomenclature`. |
| `Manufacturer` | `Contractor` | Выбирается `Contractor`, у которого `Category` содержит флаг `Manufacturer`. |

### NomenclatureParameterAssignment

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `NomenclatureKind` / `Nomenclature` | `NomenclatureKind`, `Nomenclature` | Назначение реквизита задается либо для вида НП, либо для конкретной НП, либо как общее назначение. Одновременно `NomenclatureKind` и `Nomenclature` не заполняются. |

### NomenclatureParameterEnumValue

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `NomenclatureParameter` | `NomenclatureParameter` | Реквизит должен использовать источник допустимых значений `LocalEnum`. |

### NomenclatureParameterValueBase

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `NomenclatureParameter` | `NomenclatureParameter` | Реквизит должен входить в эффективный набор назначений владельца значения. |
| `ReferenceCode` | `NomenclatureParameterEnumValue` или Platform ValueSetData | Для `LocalEnum` выбирается `NomenclatureParameterEnumValue.Number`; для `ValueSet` выбирается код значения Platform ValueSetData. |

### ContractorInContractorGroup

Ссылочные поля:

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Contractor` | `Contractor` | Один контрагент может быть включен в одну группу только один раз. |

Для внешних ссылок GMD фиксирует поле, тип внешнего объекта и базовое предметное ограничение. Полный состав списка выбора, права доступа и проверка доступности внешнего объекта описываются в модуле-владельце.

## 8. Коллекции

Коллекции GMD объявляются как коллекции Object Runtime владельца.

Режим коллекции определяет, как сохраняются строки и какие права применяются:

| Режим | Смысл | Проверка прав |
|---|---|---|
| Состав владельца | Строки сохраняются вместе с владельцем как часть его карточки. | Через права на владельца. |
| Отдельные операции | Строки показываются в карточке владельца, но создаются, изменяются и удаляются отдельными операциями своего типа объекта. | Через права на тип строки. |
| Навигация | Коллекция только показывает связанные объекты. | Через права на показываемый объект. |

| Владелец | Коллекция | Тип строки | Режим | Комментарий |
|---|---|---|---|---|
| `UnitGroup` | `UnitsOfOperation` | `UnitOfOperation` | Состав владельца | Пересчеты группы ЕИ являются частью настройки группы. |
| `NomenclatureGroup` | `Children` | `NomenclatureGroup` | Навигация | Дочерняя группа редактируется как самостоятельная группа. |
| `NomenclatureGroup` | `Nomenclatures` | `Nomenclature` | Навигация | НП редактируется как самостоятельный объект. |
| `NomenclatureKind` | `ParameterAssignments` | `NomenclatureParameterAssignment` | Состав владельца | Назначения реквизитов вида являются частью настройки вида НП. |
| `Nomenclature` | `Variants` | `NomenclatureVariant` | Отдельные операции | Исполнение НП имеет собственные код, наименование, значения реквизитов и может выбираться в других объектах. |
| `Nomenclature` | `SerialNumbers` | `SerialNumber` | Отдельные операции | Серийный номер ведется как самостоятельный реестр. |
| `Nomenclature` | `IndividualPacks` | `Pack` | Отдельные операции | `Pack` имеет единую модель сохранения для типовых и индивидуальных упаковок. |
| `Nomenclature` | `UnitsOfOperation` | `UnitOfOperation` | Состав владельца | Дополнительные ЕИ и пересчеты являются частью настройки НП. |
| `Nomenclature` | `DirectParameterAssignments` | `NomenclatureParameterAssignment` | Состав владельца | Прямые назначения реквизитов являются частью настройки НП. |
| `Nomenclature` | `ParameterValues` | `NomenclatureParameterValue` | Состав владельца | Значения реквизитов НП сохраняются вместе с НП. |
| `NomenclatureVariant` | `ParameterValues` | `NomenclatureVariantParameterValue` | Состав владельца | Значения реквизитов исполнения сохраняются вместе с исполнением. |
| `SerialNumber` | `ParameterValues` | `SerialNumberParameterValue` | Состав владельца | Значения реквизитов серийного номера сохраняются вместе с серийным номером. |
| `PackSet` | `Packs` | `Pack` | Отдельные операции | Типовая упаковка редактируется как `Pack`; владелец `PackSet` задается контекстом. |
| `Pack` | `UnitsOfOperation` | `UnitOfOperation` | Состав владельца | Пересчеты упаковки являются частью настройки упаковки. |
| `TransportUnitType` | `CapacityRules` | `UnitOfOperation` | Состав владельца | Правила вместимости являются частью настройки типа единицы транспортировки. |
| `NomenclatureParameter` | `Assignments` | `NomenclatureParameterAssignment` | Навигация | Назначения фактически принадлежат виду НП или конкретной НП; в карточке реквизита показывается обзор. |
| `NomenclatureParameter` | `EnumValues` | `NomenclatureParameterEnumValue` | Состав владельца | Локальные значения перечисления являются частью настройки реквизита. |
| `Contractor` | `Contacts` | `ContractorContact` | Состав владельца | Контакты ведутся внутри карточки контрагента. |
| `ContractorGroup` | `Contractors` | `ContractorInContractorGroup` | Состав владельца | Включения контрагентов являются составом группы. |

Если коллекция является обратной навигацией по ссылке, она не означает владение данными. Владение определяется доменной моделью и правилами `05_rules.md`.

Строки значений реквизитов вводятся через коллекции Object Runtime владельцев:

```text
Nomenclature.ParameterValues
NomenclatureVariant.ParameterValues
SerialNumber.ParameterValues
```

Object Runtime должен использовать схему реквизита из GMD:

```text
NomenclatureParameter
NomenclatureParameterAssignment
NomenclatureParameterEnumValue
Platform ValueSetData
```

При создании или изменении строки значения Object Runtime должен:

- определить владельца значения;
- определить эффективный набор назначений для владельца;
- проверить применимость реквизита к уровню владельца;
- проверить тип данных, длину и точность;
- проверить обязательность;
- проверить допустимое значение для `LocalEnum` или `ValueSet`;
- записать значение в соответствующее типизированное поле значения;
- сформировать `ValuePresentation`.

Для объектов других модулей значения реквизитов должны объявляться в коллекциях Object Runtime модуля-владельца по тому же базовому правилу.

## 9. Действия, обработчики и проверки

Стандартные действия чтения, создания, изменения и удаления выполняются через Object Runtime.

Предметные проверки GMD подключаются как валидаторы типа объекта и описаны в `05_rules.md`.

Собственные действия Object Runtime для GMD v1 не вводятся. Сценарии ниже выполняются стандартным созданием или изменением объекта и обработчиками сохранения.

| Сценарий | Runtime-решение |
|---|---|
| Создание или изменение НП | Обработчик сохранения `Nomenclature` синхронизирует базовую `UnitOfOperation`. Состав записи и проверки описаны в `05_rules.md`. |
| Значения реквизитов | Создание и изменение строк значений выполняются через коллекции Object Runtime владельца и валидаторы реквизитов. |
| Архивирование объектов с жизненным циклом | Выполняется через действия жизненного цикла, если тип объекта управляется жизненным циклом. |
| Машиночитаемая идентификация | Подключается через общий механизм Common/Platform для типов объектов, где это разрешено конфигурацией. |
| Автонумерация | Выполняется через Platform Numbering для полей, объявленных поддерживающими автонумерацию. |

## 10. Платформенные возможности вне GMD

GMD не объявляет `BarcodeTemplate`, `BarcodeData` или `Barcode` как собственные типы объектов.

Объекты GMD могут подключать общую возможность машиночитаемой идентификации по `REQ-00-B-019..022`. Формирование, хранение, поиск и проверка считанного идентификатора относятся к Common/Platform.

GMD не объявляет `NumeratorDescriptor`, `NumeratorDescription`, `Numerator` или `NumeratorValue` как собственные типы объектов.

Object Runtime вызывает Platform Numbering только для полей, объявленных поддерживающими автонумерацию.

Примеры полей:

```text
Nomenclature.Code
SerialNumber.SerialNumber
```

Правила нумерации, условия, момент присвоения номера и счетчики выполнения описываются в Platform Numbering.

`Nomenclature.IsLotsControlRequired` и `Nomenclature.SerialNumbersControlType` определяют необходимость контроля партий и серийных номеров. Они не задают формат номера и не заменяют `NumberingRule`.

`ExternalId` используется для поиска и совместимости с внешними источниками. Полноценное сопоставление с внешними системами выполняет Integration Foundation.

## 11. Граница с интерфейсом

Этот документ фиксирует доступность объектов GMD через Object Runtime. Компоновка пользовательских представлений описана в `06_ui_views.md`.

Интерфейс не дублирует системные поля Common, область предприятия, фильтры архивных/удаленных записей и правила проверки GMD.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | Публикуемые типы объектов | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-07 10:40 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [dee4ebb0](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/dee4ebb02ed702461463c16315f3b252ae418edd) |
