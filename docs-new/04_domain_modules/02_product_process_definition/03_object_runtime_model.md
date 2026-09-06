---
id: DOC-04-02-03
title: 'Runtime-модель объектов - 02 Составы и технологии'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 02_product_process_definition
holder: '@axelprosoft'
created_at: 2026-08-19 12:02
created_by: '@axelprosoft'
updated_at: 2026-09-02 11:50
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Runtime-модель объектов - 02 Составы и технологии

## 1. Назначение документа

Документ описывает, как объекты модуля `02 Product & Process Definition` публикуются и используются через Object Runtime.

Документ не повторяет доменную модель. Полный состав объектов, поля, русские названия, типы, обязательность, связи и коллекции описаны в `02_domain_model.md`.

Компоновка списков и карточек описывается в `06_ui_views.md`. Инварианты и проверки описываются в `05_rules.md`. Жизненный цикл объектов описывается в `04_workflows.md`. Прикладные операции, внутренние функции и алгоритмы описываются в `13_operations.md`.

## 2. Базовое решение

Основной способ работы с объектами модуля:

```text
Business Object Runtime / Object Runtime
```

Ручные API-методы для стандартных действий чтения, создания, изменения и удаления не вводятся.

Object Runtime должен:

- публиковать конкретные типы объектов модуля;
- применять `TenantId` из контекста или владельца;
- применять Common-поля, архивирование и мягкое удаление;
- проверять стандартные права на чтение, создание, изменение, удаление и действия;
- поддерживать строки коллекций владельцев;
- применять runtime-контракты абстрактных базовых типов через их наследников;
- вычислять представление объектов по правилам типа объекта;
- выполнять lookup-ограничения и валидаторы модуля;
- вызывать обработчики модуля в стандартных точках создания, сохранения и предметных действий.

## 3. Публикуемые типы объектов

| Тип объекта | Русское название | Базовый тип | Роль в Object Runtime |
|---|---|---|---|
| `ManufacturingBillTemplate` | Шаблон спецификации | `CommonCatalogObject` | Настроечный объект: список, карточка, lookup; без `ProcessDocumentLifecycle`. |
| `ProcessTemplate` | Шаблон технологии | `CommonCatalogObject` | Настроечный объект: список, карточка, lookup; без `ProcessDocumentLifecycle`. |
| `LabourType` | Вид работы | `CommonCatalogObject` | Классификатор модуля: список, карточка, lookup; без `ProcessDocumentLifecycle`. |
| `ManufacturingOperationType` | Вид операции | `CommonCatalogObject` | Классификатор модуля: список, карточка, lookup; без `ProcessDocumentLifecycle`. |
| `ProcessingOperation` | Технологическая операция по классификатору | `CommonCatalogObject` | Классификатор модуля: список, карточка, древовидный lookup; без `ProcessDocumentLifecycle`. |
| `WorkEnvironment` | Условия труда | `CommonCatalogObject` | Классификатор модуля: список, карточка, lookup; без `ProcessDocumentLifecycle`. |
| `ManufacturingBill` | Спецификация / состав изделия | `CommonCatalogObject` | Самостоятельный ревизионный объект: список, карточка, lookup, workflow. |
| `ManufacturingBillPosition` | Позиция спецификации | `MaterialPositionBase` | Строка коллекции `ManufacturingBill.Positions`. |
| `ProcessDefinition` | Технологическое описание | `ProcessBase` | Самостоятельный ревизионный объект: список, карточка, lookup, workflow. |
| `ProcessSegment` | Передел | `ProcessSegmentBase` | Строка коллекции `ProcessDefinition.Segments`. |
| `ProcessOperation` | Операция | `ProcessOperationBase` | Строка коллекции `ProcessSegment.Operations`. |
| `ProcessTransition` | Переход операции | `ProcessStepBase` | Строка коллекции `ProcessOperation.Transitions`. |
| `ProcessSegmentMaterialPosition` | Материальная позиция передела | `MaterialPositionBase` | Строка коллекции `ProcessSegment.MaterialPositions`. |
| `ProcessOperationMaterialPosition` | Материальная позиция операции | `MaterialPositionBase` | Строка коллекции `ProcessOperation.MaterialPositions`. |
| `ProcessSegmentEquipmentPosition` | Позиция оборудования передела | `ProcessEquipmentPositionBase` | Строка коллекции `ProcessSegment.EquipmentPositions`. |
| `ProcessOperationEquipmentPosition` | Позиция оборудования операции | `ProcessEquipmentPositionBase` | Строка коллекции `ProcessOperation.EquipmentPositions`. |
| `ProcessSegmentLabourPosition` | Позиция трудового ресурса передела | `ProcessLabourPositionBase` | Строка коллекции `ProcessSegment.LabourPositions`. |
| `ProcessOperationLabourPosition` | Позиция трудового ресурса операции | `ProcessLabourPositionBase` | Строка коллекции `ProcessOperation.LabourPositions`. |
| `ProcessSegmentToolingPosition` | Позиция оснастки передела | `ProcessToolingPositionBase` | Строка коллекции `ProcessSegment.ToolingPositions`. |
| `ProcessOperationToolingPosition` | Позиция оснастки операции | `ProcessToolingPositionBase` | Строка коллекции `ProcessOperation.ToolingPositions`. |
| `CooperationScheme` | Схема кооперации | `CommonCatalogObject` | Самостоятельный ревизионный объект: список, карточка, lookup, workflow. |
| `CooperationSchemePosition` | Позиция схемы кооперации | `CommonObject` | Строка коллекции `CooperationScheme.Positions`. |
| `UseCondition` | Условие применимости | `CommonObject` | Единая строка условий, отображаемая в коллекциях `UseConditions` владельцев. |
| `UseConditionParameterValue` | Значение реквизита условия | `GMD.NomenclatureParameterValueBase` | Строка коллекции `UseCondition.ParameterValues`. |
| `NomenclatureSubstitution` | Замена номенклатуры | `CommonObject` | Самостоятельный объект без `ProcessDocumentLifecycle`. |
| `ProductComposition` | Анализ состава | `CommonObject` | Самостоятельный расчетный объект без workflow утверждения. |
| `ProductCompositionParameterValue` | Значение реквизита анализа состава | `GMD.NomenclatureParameterValueBase` | Строка коллекции `ProductComposition.ParameterValues`. |
| `ProductCompositionPosition` | Позиция анализа состава | `CompositionPositionBase` | Строка коллекции `ProductComposition.Positions`. |

`ProcessOperationSequenceBase` и `ProcessOperationSequence` не публикуются в v1. Связи последовательности операций и сетевые графики остаются вне границы v1.

Документные связи не публикуются как коллекции модуля. Связанные документы технологического описания отображаются через Object Runtime / UI `Document Management`; универсальные документные вкладки других объектов подключаются общесистемной конфигурацией Document Management, а не контрактами коллекций модуля.

## 4. Runtime-контракты базовых типов

Абстрактный базовый тип модуля не является самостоятельным пользовательским object type. В этом модуле базовые типы не получают собственных `List`, `Lookup`, `Details` и карточки.

Это не общий запрет Object Runtime: абстрактный тип может публиковаться как reference-only lookup, если это является предметным решением модуля. Такой режим используется, например, в `03_plant_structure` для `OrganizationalUnit`. Для базовых типов этого модуля такой lookup не нужен: пользователь выбирает конкретные строки состава, передела, операции или ресурса в контексте их владельца. Условия применимости являются единым типом `UseCondition` и редактируются через коллекции владельцев.

Runtime-контракт базового типа наследуется конкретными object type. Наследник не повторяет общие поля и поведение базового типа; он добавляет владельца, коллекции, контекстные ссылки и собственные ограничения.

Если поле, представление, lookup-ограничение, валидатор, обработчик или runtime-action задан на базовом типе, он действует для всех наследников этого базового типа. При этом действие не выполняется "над абстрактным объектом" само по себе: оно публикуется и проверяется на конкретном object type-наследнике. В разделах ниже у наследников описываются только дельты: владелец, режим коллекции, дополнительные ссылки и ограничения конкретного уровня.

| Базовый тип | Runtime-контракт |
|---|---|
| `ProcessBase` | Общие поля технологического описания, ревизионность, области использования, период действия, признак основной технологии и необязательная ссылка на индивидуальную спецификацию ТО. Наследник `ProcessDefinition` получает workflow `ProcessDocumentLifecycle`. |
| `ProcessSegmentBase` | Числовой номер `Number`, номер в ТП `NumberInProcess`, наименование, вид работы, ответственная орг. единица, признак субподряда, признаки автозавершения/учета/партии/серийности, длительность, временной буфер, выход, потери и правило округления передела. Наследник добавляет владельца `ProcessDefinition`. |
| `ProcessOperationBase` | Числовой номер `Number`, номер в ТП `NumberInProcess`, наименование, вид работы, классификаторная операция, вид операции, схема подтверждения, условия труда, группа оплаты, ответственная орг. единица, орг. единица выполнения, признак субподряда, КОИД, коэффициент производительности, типы и значения временных норм операции. Наследник добавляет владельца `ProcessSegment`. |
| `ProcessStepBase` | Числовой номер `Number`, номер в ТП `NumberInProcess`, описание и особые указания перехода. Наследник добавляет владельца `ProcessOperation`. |
| `MaterialPositionBase` | Компонент, исполнение, тип входа/выхода, `RateUnit`, базовая пара норм, округление и места отпуска/списания. Наследники добавляют владельца и контекст спецификации/технологии. |
| `ProcessEquipmentPositionBase` | Рабочее место или группа рабочих мест, количество, КОИД, коэффициент производительности и временные нормы оборудования. Наследники добавляют владельца передела или операции. |
| `ProcessLabourPositionBase` | Персонал, профессия, разряд, количество рабочих, группа оплаты и временные нормы трудового ресурса. Наследники добавляют владельца и ссылку на позицию оборудования своего уровня. |
| `ProcessToolingPositionBase` | Оснастка и количество. Наследники добавляют владельца передела или операции и ссылку на позицию оборудования своего уровня. |
| `CompositionPositionBase` | Расчетные количества, источники нормы, источники кооперации, данные замены и расчетный статус строки. Наследник добавляет владельца `ProductComposition` и родителя в иерархии. |

## 5. Tenant и Common-поля

Все объекты модуля принадлежат `Tenant`.

Object Runtime назначает `TenantId`:

- при создании корневого объекта из контекста;
- при создании строки коллекции из владельца;
- при создании расчетных строк из владельца расчета;
- при выборе ссылочного объекта ограничивает выбор тем же `Tenant`, если модуль-владелец ссылки использует tenant-область.

`TenantId` не редактируется в пользовательских карточках.

Для всех типов объектов на базе `CommonObject` применяются правила `00_common`:

- системные поля Common не редактируются обычными формами создания и изменения;
- архивные и удаленные записи скрываются по умолчанию;
- уже сохраненная ссылка на архивный объект отображается;
- новая ссылка на архивный объект запрещена;
- системная секция администрирования добавляется по правилам Common/Object Runtime.

## 6. Наборы данных Object Runtime

Для самостоятельных типов объектов baseline должен объявлять стандартные наборы данных:

| Набор данных | Назначение | Применяется к |
|---|---|---|
| `List` | Основной список типа объекта. | `ManufacturingBillTemplate`, `ProcessTemplate`, `LabourType`, `ManufacturingOperationType`, `ProcessingOperation`, `WorkEnvironment`, `ManufacturingBill`, `ProcessDefinition`, `CooperationScheme`, `NomenclatureSubstitution`, `ProductComposition` |
| `Lookup` | Выбор объекта в ссылочном поле. | `ManufacturingBillTemplate`, `ProcessTemplate`, `LabourType`, `ManufacturingOperationType`, `ProcessingOperation`, `WorkEnvironment`, `ManufacturingBill`, `ManufacturingBillPosition`, `ProcessDefinition`, `ProcessSegment`, `ProcessOperation`, `ProcessTransition`, `CooperationScheme`, `CooperationSchemePosition`, `NomenclatureSubstitution`, `ProductComposition` |
| `Details` | Чтение объекта для карточки или связанной строки. | Все публикуемые типы объектов. |
| `CollectionRows` | Чтение и сохранение строк коллекции владельца. | Все строки коллекций модуля. |

Строки коллекций не имеют самостоятельных пользовательских списков, если в `06_ui_views.md` не описан отдельный список. Они читаются и сохраняются через карточку владельца или через связанную коллекцию Object Runtime.

Абстрактные базовые типы не получают собственных наборов данных. `UseCondition` не является абстрактным базовым типом: он читается и сохраняется как строка коллекции владельца, без самостоятельного глобального пользовательского списка.

## 7. Коллекции

Режимы коллекций:

| Режим | Смысл | Проверка прав |
|---|---|---|
| Состав владельца | Строки сохраняются вместе с владельцем как часть его карточки. | Через права на владельца. |
| Навигация | Коллекция показывает связанные объекты, но не задает владение. | Через права на показываемый объект. |

Коллекции модуля:

| Владелец | Коллекция | Тип строки | Режим | Комментарий |
|---|---|---|---|---|
| `ManufacturingBill` | `Positions` | `ManufacturingBillPosition` | Состав владельца | Позиции состава. |
| `ManufacturingBill` | `UseConditions` | `UseCondition` | Состав владельца | Условия применимости ревизии; parent link `UseCondition.ManufacturingBill`. |
| `ManufacturingBillPosition` | `UseConditions` | `UseCondition` | Состав владельца | Условия применимости строки состава; parent link `UseCondition.ManufacturingBillPosition`. |
| `ProcessDefinition` | `Segments` | `ProcessSegment` | Состав владельца | Переделы технологии. |
| `ProcessDefinition` | `UseConditions` | `UseCondition` | Состав владельца | Условия применимости технологии; parent link `UseCondition.ProcessDefinition`. |
| `ProcessSegment` | `Operations` | `ProcessOperation` | Состав владельца | Операции передела. |
| `ProcessSegment` | `MaterialPositions` | `ProcessSegmentMaterialPosition` | Состав владельца | Материальные позиции передела. |
| `ProcessSegment` | `EquipmentPositions` | `ProcessSegmentEquipmentPosition` | Состав владельца | Позиции оборудования / рабочих мест передела. |
| `ProcessSegment` | `LabourPositions` | `ProcessSegmentLabourPosition` | Состав владельца | Позиции трудовых ресурсов передела. |
| `ProcessSegment` | `ToolingPositions` | `ProcessSegmentToolingPosition` | Состав владельца | Позиции технологической оснастки передела. |
| `ProcessOperation` | `Transitions` | `ProcessTransition` | Состав владельца | Переходы операции. |
| `ProcessOperation` | `MaterialPositions` | `ProcessOperationMaterialPosition` | Состав владельца | Материальные позиции операции. |
| `ProcessOperation` | `EquipmentPositions` | `ProcessOperationEquipmentPosition` | Состав владельца | Позиции оборудования / рабочих мест операции. |
| `ProcessOperation` | `LabourPositions` | `ProcessOperationLabourPosition` | Состав владельца | Позиции трудовых ресурсов операции. |
| `ProcessOperation` | `ToolingPositions` | `ProcessOperationToolingPosition` | Состав владельца | Позиции технологической оснастки операции. |
| `CooperationScheme` | `Positions` | `CooperationSchemePosition` | Состав владельца | Позиции схемы. |
| `CooperationScheme` | `UseConditions` | `UseCondition` | Состав владельца | Условия применимости схемы; parent link `UseCondition.CooperationScheme`. |
| `NomenclatureSubstitution` | `UseConditions` | `UseCondition` | Состав владельца | Условия применения замены; parent link `UseCondition.NomenclatureSubstitution`. |
| `UseCondition` | `ParameterValues` | `UseConditionParameterValue` | Состав владельца | Значения реквизита для условия `ItemParameter`. |
| `ProductComposition` | `ParameterValues` | `ProductCompositionParameterValue` | Состав владельца | Значения реквизитов расчетного контекста. |
| `ProductComposition` | `Positions` | `ProductCompositionPosition` | Состав владельца | Расчетные строки результата. |

Удаление и изменение строк коллекций проверяются через владельца, если строка редактируется как состав владельца. Отдельные прикладные permission-коды для строк коллекций не вводятся.

Связанные документы технологического описания не являются коллекцией модуля. Они отображаются как связанный список `Document Management`.

## 8. Представление объектов

`Presentation` не является физическим полем модуля. Если импортируемые данные содержат поле представления, в целевой модели оно трассируется в правило Object Runtime.

Object Runtime вычисляет представление по правилу конкретного типа объекта. Если правило использует несколько частей, пустые необязательные части и лишние пробелы не выводятся.

| Тип объекта | Правило представления |
|---|---|
| `ManufacturingBill` | `{Code} / Rev.{Revision} - {Product.Presentation}` |
| `ManufacturingBillPosition` | `{PositionNumber} - {Component.Presentation}` |
| `LabourType`, `ManufacturingOperationType`, `WorkEnvironment` | `{Code} - {Name}` |
| `ProcessingOperation` | `{Code} - {Name}` |
| `ProcessDefinition` | `{Code} / Rev.{Revision} - {Product.Presentation}` |
| `ProcessSegment` | `{Number:0000} [{NumberInProcess}] - {ResponsibleOrgUnit.Code} {Name}` |
| `ProcessOperation` | `{Number:0000} [{NumberInProcess}] - {ExecutionOrgUnit.Code или ResponsibleOrgUnit.Code} {Name}` |
| `ProcessTransition` | `{Number:0000} [{NumberInProcess}]` |
| `ProcessSegmentMaterialPosition`, `ProcessOperationMaterialPosition` | `{PositionNumber} - {Component.Presentation}` |
| `ProcessSegmentEquipmentPosition`, `ProcessOperationEquipmentPosition` | `{PositionNumber} - {WorkPlace.Presentation ?? WorkPlaceGroup.Presentation}` |
| `ProcessSegmentLabourPosition`, `ProcessOperationLabourPosition` | `{PositionNumber} - {Profession.Presentation} {Grade}` |
| `ProcessSegmentToolingPosition`, `ProcessOperationToolingPosition` | `{PositionNumber} - {Tooling.Presentation}` |
| `CooperationScheme` | `{Code} / Rev.{Revision} - {MainProduct.Presentation}` |
| `CooperationSchemePosition` | `{PositionNumber} - {Nomenclature.Presentation}` |
| `UseCondition` | `{ConditionSubject} {ComparisonOperator} {ConditionValuePresentation}` |
| `UseConditionParameterValue` | `{AttributeCode}: {ValuePresentation}` |
| `NomenclatureSubstitution` | `{SubstitutedNomenclature.Presentation} -> {ReplacementNomenclature.Presentation}` |
| `ProductComposition` | `{CompositionNumber} от {CompositionDate}` |
| `ProductCompositionParameterValue` | `{AttributeCode}: {ValuePresentation}` |
| `ProductCompositionPosition` | `{PositionNumber} - {Nomenclature.Presentation}` |

Поиск по представлению не требует физического поля `Presentation`. Для списков и lookup должны настраиваться поисковые поля, из которых строится представление.

Представление условий применимости не является отдельным редактируемым полем. Для карточек, списков и tooltip Object Runtime формирует его из коллекции `UseConditions` владельца по тем же строкам условий, которые участвуют в проверках применимости.

`ConditionValuePresentation` не хранится как поле. Object Runtime формирует его из типизированного значения, соответствующего `ConditionSubject`; для `ItemParameter` используется представление строк `ParameterValues`.

## 9. Lookup, поиск и фильтрация

Выбор ссылочных объектов должен использовать стандартный механизм Object Runtime.

В подразделах ниже также указаны enum-selector поля и расчетные ссылочные поля, если они важны для runtime-контракта выбора или заполнения. Расчетные поля не становятся пользовательскими lookup, но их источник должен быть однозначен.

Общие правила выбора:

- показываются только неархивные и неудаленные объекты;
- текущий `Tenant` применяется автоматически;
- уже сохраненная ссылка на архивный объект отображается;
- новая ссылка на архивный объект запрещена;
- пользователь должен иметь право чтения выбираемого объекта;
- доступность внешнего объекта подтверждает модуль-владелец;
- предметные ограничения модуля проверяются валидаторами из `05_rules.md`.

Ограничения выбора сверх общих правил описываются внутри object contract, которому принадлежит поле. Если поле объявлено в базовом object contract, lookup-правило фиксируется на базовом поле и не дублируется у наследников.

### 9.1 `ManufacturingBill`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Product` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `ProductVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать выбранному `Product`. |
| `ProcessType` | `GMD.ProcessType` | Значение системного enum GMD. |
| `ManufacturingBillTemplate` | `ManufacturingBillTemplate` | Активный шаблон текущего `Tenant`; при создании по умолчанию выбирается шаблон с `IsDefault = true`. |
| `Unit` | Операции GMD `UnitOfOperation` | Единица операции для `Product` и области применения `Production`. |
| `BaseUnit` | `GMD.Unit` | Заполняется из базовой ЕИ `Product` и не выбирается пользователем независимо. |

### 9.2 `ManufacturingBillPosition`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `ObtainMethod` | `GMD.NomenclatureObtainMethod` | Значение системного enum GMD. |
| `ProcessType` | `GMD.ProcessType` | Значение системного enum GMD. |
| `ValidFrom`, `ValidTo` | Date | Используются при подборе действующих строк состава. |

### 9.3 `ProcessBase`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Product` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `ProductVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать выбранному `Product`. |
| `ProcessType` | `GMD.ProcessType` | Значение системного enum GMD. |
| `ProcessTemplate` | `ProcessTemplate` | Активный шаблон текущего `Tenant`; при создании по умолчанию выбирается шаблон с `IsDefault = true`. |
| `ManufacturingBill` | `ManufacturingBill` | Выбирается только индивидуальная спецификация ТО: `IsForProcessDefinition = true`, тот же `Product`, совместимое `ProductVariant` и совместимый `ProcessType`. Если поле не заполнено, технология является общей и спецификация подбирается отдельным алгоритмом. |
| `OutputWarehouse` | `03_plant_structure.OrganizationalUnit` | Используется abstract/reference-only lookup `03 Plant Structure`; выбираются только `OrganizationalUnit` с `IsInventoryStorageLocation = true`. |
| `OutputWarehouseBin` | `03_plant_structure.WarehouseBin` | Если заполнено МХ выпуска, выбираются только ячейки этого места хранения; принадлежность зоны и иерархии ячеек проверяет `03 Plant Structure`. |

### 9.4 `ProcessSegmentBase`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `LabourType` | `LabourType` | Активный вид работы текущего `Tenant`. |
| `ResponsibleOrgUnit` | `03_plant_structure.OrganizationalUnit` | Используется abstract/reference-only lookup `03 Plant Structure`. Выбираются `ProductionUnit` или `Subcontractor` текущего `Tenant` с `IsSegmentLevel = true`; `IsSubcontracted` ограничивает конкретный вид организационной единицы. |
| `OutputWarehouse` | `03_plant_structure.OrganizationalUnit` | Используется abstract/reference-only lookup `03 Plant Structure`; выбираются только `OrganizationalUnit` с `IsInventoryStorageLocation = true`. |
| `OutputWarehouseBin` | `03_plant_structure.WarehouseBin` | Если заполнено МХ выпуска, выбираются только ячейки этого места хранения; принадлежность зоны и иерархии ячеек проверяет `03 Plant Structure`. |
| `ScrapWarehouse` | `03_plant_structure.OrganizationalUnit` | Используется abstract/reference-only lookup `03 Plant Structure`; выбираются только `OrganizationalUnit` с `IsInventoryStorageLocation = true`. |
| `ScrapWarehouseBin` | `03_plant_structure.WarehouseBin` | Если заполнено МХ брака, выбираются только ячейки этого места хранения; принадлежность зоны и иерархии ячеек проверяет `03 Plant Structure`. |

### 9.5 `ProcessOperationBase`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `LabourType` | `LabourType` | Активный вид работы текущего `Tenant`. При выборе `ProcessingOperation` может заполняться из `ProcessingOperation.LabourType`. |
| `ProcessingOperation` | `ProcessingOperation` | Активная классификаторная операция текущего `Tenant`; древовидный список использует `ProcessingOperation.Parent`. |
| `ManufacturingOperationType` | `ManufacturingOperationType` | Активный вид операции текущего `Tenant`. При выборе `ProcessingOperation` может заполняться из `ProcessingOperation.ManufacturingOperationType`. |
| `ConfirmationStages` | `ConfirmationStage` | Набор значений системного enum. При создании операции может заполняться из `ManufacturingOperationType.ConfirmationStages`. |
| `PaymentGroup` | `ResourceManagement:PaymentGroup` | Внешний справочник ресурсного модуля; точные фильтры задает модуль-владелец. |
| `WorkEnvironment` | `WorkEnvironment` | Активные условия труда текущего `Tenant`. |
| `ResponsibleOrgUnit` | `03_plant_structure.OrganizationalUnit` | Используется abstract/reference-only lookup `03 Plant Structure`. Выбираются `ProductionUnit` или `Subcontractor` текущего `Tenant` с `IsOperationLevel = true`; `IsSubcontracted` ограничивает конкретный вид организационной единицы, совместимость с ответственной орг. единицей передела проверяется правилами модуля и `03 Plant Structure`. |
| `ExecutionOrgUnit` | `03_plant_structure.OrganizationalUnit` | Необязательное уточнение места выполнения операции. Если заполнено, используется abstract/reference-only lookup `03 Plant Structure`; выбираются `ProductionUnit` или `Subcontractor` текущего `Tenant` с `IsOperationLevel = true`; выбранная орг. единица должна быть совместима с `ResponsibleOrgUnit` операции. |

### 9.6 `MaterialPositionBase`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Component` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `ComponentVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать выбранному `Component`. |
| `ComponentManufacturingBill` | `ManufacturingBill` | Ревизия спецификации должна относиться к `Component` / `ComponentVariant`; продуктивная применимость проверяется при использовании или выпуске владельца. |
| `ComponentProcessDefinition` | `ProcessDefinition` | Ревизия технологии должна относиться к `Component` / `ComponentVariant`; продуктивная применимость проверяется при использовании или выпуске владельца. |
| `RateBaseUnit` | `GMD.Unit` | Заполняется из базовой ЕИ `Component` и не выбирается пользователем независимо. |
| `RateUnit` | Операции GMD `UnitOfOperation` | Единица операции для `Component` и области применения `Production`. |
| `ReleaseWarehouse` | `03_plant_structure.OrganizationalUnit` | Используется abstract/reference-only lookup `03 Plant Structure`; выбираются только `OrganizationalUnit` с `IsInventoryStorageLocation = true`. |
| `IssueWarehouse` | `03_plant_structure.OrganizationalUnit` | Используется abstract/reference-only lookup `03 Plant Structure`; выбираются только `OrganizationalUnit` с `IsInventoryStorageLocation = true`. |
| `ReleaseWarehouseBin` | `03_plant_structure.WarehouseBin` | Если заполнено место хранения отпуска, выбираются только ячейки этого места хранения; принадлежность зоны и иерархии ячеек проверяет `03 Plant Structure`. |
| `IssueWarehouseBin` | `03_plant_structure.WarehouseBin` | Если заполнено место хранения списания, выбираются только ячейки этого места хранения; принадлежность зоны и иерархии ячеек проверяет `03 Plant Structure`. |
| `InventoryStatus` | `InventoryStatus` модуля производственной логистики | Выбираются статусы запаса, разрешенные модулем-владельцем; модуль не объявляет собственный enum статусов запаса. |
| `IssueComponentType` | `GMD.IssueComponentType` | Значение системного enum GMD. |

### 9.7 Материальные позиции технологии

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `ProcessSegmentMaterialPosition.ManufacturingBillPosition` | `ManufacturingBillPosition` | Если `ProcessDefinition.ManufacturingBill` заполнена, выбираются действующие позиции этой индивидуальной спецификации ТО. Если технология общая, выбираются действующие позиции применимых спецификаций продукта по правилам подбора. |
| `ProcessOperationMaterialPosition.ManufacturingBillPosition` | `ManufacturingBillPosition` | Если `ProcessDefinition.ManufacturingBill` заполнена, выбираются действующие позиции этой индивидуальной спецификации ТО. Если технология общая, выбираются действующие позиции применимых спецификаций продукта по правилам подбора. |
| `ProcessSegmentMaterialPosition.PositionForSample` | `ProcessSegmentMaterialPosition` | Образцовая строка должна принадлежать тому же `ProcessSegment` или допустимому шаблонному контексту, если он описан в `13_operations.md`. |
| `ProcessOperationMaterialPosition.PositionForSample` | `ProcessOperationMaterialPosition` | Образцовая строка должна принадлежать тому же `ProcessOperation` или допустимому шаблонному контексту, если он описан в `13_operations.md`. |

### 9.8 Ресурсные позиции технологии

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `ProcessEquipmentPositionBase.WorkPlaceGroup` | `ResourceManagement:WorkPlaceGroup` | Модуль хранит нормативную ссылку и не проверяет фактическую доступность ресурса; ограничения совместимости и доступности предоставляет модуль-владелец. |
| `ProcessEquipmentPositionBase.WorkPlace` | `ResourceManagement:WorkPlace` | Если заполнена группа рабочих мест, выбранное рабочее место должно входить в эту группу по правилам модуля-владельца. При одновременном заполнении ссылки на группу и конкретное рабочее место конкретное рабочее место уточняет выбор внутри группы. |
| `ProcessLabourPositionBase.PersonnelGroup` | `ResourceManagement:PersonnelGroup` | Модуль хранит нормативную ссылку; ограничения совместимости и доступности предоставляет модуль-владелец. |
| `ProcessLabourPositionBase.Personnel` | `ResourceManagement:Personnel` | Если заполнена группа сотрудников, выбранный сотрудник должен входить в эту группу по правилам модуля-владельца. При одновременном заполнении ссылки на группу и конкретного сотрудника конкретный сотрудник уточняет выбор внутри группы. |
| `ProcessLabourPositionBase.Profession` | `ResourceManagement:Profession` | Модуль хранит нормативную ссылку; ограничения выбора предоставляет модуль-владелец. |
| `ProcessLabourPositionBase.PaymentGroup` | `ResourceManagement:PaymentGroup` | Модуль хранит нормативную ссылку; ограничения выбора предоставляет модуль-владелец. |
| `ProcessSegmentLabourPosition.ProcessSegmentEquipmentPosition` | `ProcessSegmentEquipmentPosition` | Позиция оборудования должна принадлежать тому же `ProcessSegment`. |
| `ProcessOperationLabourPosition.ProcessOperationEquipmentPosition` | `ProcessOperationEquipmentPosition` | Позиция оборудования должна принадлежать тому же `ProcessOperation`. |
| `ProcessSegmentToolingPosition.ProcessSegmentEquipmentPosition` | `ProcessSegmentEquipmentPosition` | Позиция оборудования должна принадлежать тому же `ProcessSegment`. |
| `ProcessOperationToolingPosition.ProcessOperationEquipmentPosition` | `ProcessOperationEquipmentPosition` | Позиция оборудования должна принадлежать тому же `ProcessOperation`. |
| `ProcessToolingPositionBase.Tooling` | `ResourceManagement:ToolBase` | Нормативная ссылка на общий объект инструмента и оснастки; объединенный lookup возвращает выбранный конкретный тип (`Tooling` или `Gage`) и его идентификатор. Ограничения выбора предоставляет модуль-владелец. |

### 9.9 `CooperationScheme`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `MainProduct` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `MainProductVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать `MainProduct`. |

### 9.10 `CooperationSchemePosition`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Nomenclature` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `NomenclatureVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать `Nomenclature`. |
| `ComponentManufacturingBill` | `ManufacturingBill` | Ревизия спецификации должна относиться к `Nomenclature` / `NomenclatureVariant`. |
| `NomenclatureObtainMethod` | `GMD.NomenclatureObtainMethod` | Значение системного enum GMD. |
| `Subcontractor` | `03_plant_structure.Subcontractor` | Выбирается активный `Subcontractor` текущего `Tenant`; если позиция схемы означает внешнее выполнение, ссылка обязательна по правилам `05_rules.md`. |

### 9.11 `NomenclatureSubstitution`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `SubstitutedNomenclature` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `SubstitutedNomenclatureVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать `SubstitutedNomenclature`. |
| `ReplacementNomenclature` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `ReplacementNomenclatureVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать `ReplacementNomenclature`. |
| `RateUnit` | Операции GMD `UnitOfOperation` | Единица операции для заменяющей или заменяемой НП по правилу `05_rules.md`. |
| `ProcessDefinition` | `ProcessDefinition` | Если замена ограничена технологией, выбирается конкретная ревизия технологии текущего `Tenant`. |
| `ManufacturingBill` | `ManufacturingBill` | Если замена ограничена составом, выбирается конкретная ревизия спецификации текущего `Tenant`. |

### 9.12 `ProductComposition`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Product` | `GMD.Nomenclature` | НП текущего `Tenant`, доступная пользователю. |
| `ProductVariant` | `GMD.NomenclatureVariant` | Исполнение должно принадлежать выбранному `Product`. |
| `Unit` | Операции GMD `UnitOfOperation` | Единица операции для `Product`; пользовательский ввод пересчитывается в `ProductQtyBaseUnit`. |
| `BaseUnit` | `GMD.Unit` | Заполняется из базовой ЕИ `Product` и не выбирается пользователем независимо. |
| `ProcessType` | `GMD.ProcessType` | Используется как параметр продуктивного подбора спецификации и технологии. |
| `ProductManufacturingBill` | `ManufacturingBill` | Если исходная спецификация задана вручную, она должна относиться к `Product` / `ProductVariant`; продуктивный подбор выполняется по `13_operations.md`. |
| `ProductProcessDefinition` | `ProcessDefinition` | Если исходная технология задана вручную, она должна относиться к `Product` / `ProductVariant`; продуктивный подбор выполняется по `13_operations.md`. |
| `CooperationScheme` | `CooperationScheme` | Если исходная схема задана вручную, она должна относиться к `Product` / `ProductVariant`; продуктивный подбор выполняется по `13_operations.md`. |
| `MainProduct` | `GMD.Nomenclature` | Используется как контекст условий применимости. |
| `MainProductSerialNumber` | `GMD.SerialNumber` | Серийный номер должен принадлежать `MainProduct`. |
| `DemandGroup` | внешний `DemandGroup` | Используется как контекст условий применимости. |
| `Project` | внешний `Project` | Используется как контекст условий применимости. |
| `ProjectPhase` | внешний `ProjectPhase` | Если заполнен, должен принадлежать `Project`. |
| `ProductOrder` | внешний `ProductOrder` | Используется как контекст условий применимости. |
| `ProductOrderPosition` | внешний `ProductOrderPosition` | Если заполнена, должна принадлежать `ProductOrder`. |

### 9.13 `ProductCompositionPosition`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `Nomenclature` | Расчетный результат / GMD | Формируется расчетом; если строка корректируется вручную в разрешенном сценарии, НП должна быть доступна текущему `Tenant`. |
| `NomenclatureVariant` | Расчетный результат / GMD | Формируется расчетом; если строка корректируется вручную в разрешенном сценарии, исполнение должно принадлежать НП. |
| `BaseUnit` | `GMD.Unit` | Формируется расчетом из базовой ЕИ `Nomenclature` и не выбирается пользователем независимо. |
| `Unit` | Расчетный результат / функции GMD `UnitOfOperation` | Формируется расчетом через функции GMD и не редактируется вручную в стандартном сценарии. |
| `SubstitutedUnit` | Расчетный результат / функции GMD `UnitOfOperation` | Формируется расчетом через функции GMD и не редактируется вручную в стандартном сценарии. |
| `SourceManufacturingBill` | `ManufacturingBill` | Формируется расчетом; источник должен быть ревизией, примененной алгоритмом подбора. |
| `SourceProcessDefinition` | `ProcessDefinition` | Формируется расчетом; источник должен быть ревизией, примененной алгоритмом подбора. |
| `SourceSegment` | `ProcessSegment` | Формируется расчетом; передел должен принадлежать `SourceProcessDefinition`. |
| `SourceBillPosition` | `ManufacturingBillPosition` | Формируется расчетом; позиция должна принадлежать `SourceManufacturingBill`. |
| `SourceSegmentMaterialPosition` | `ProcessSegmentMaterialPosition` | Формируется расчетом; позиция должна принадлежать `SourceSegment`. |
| `SourceCooperationScheme` | `CooperationScheme` | Формируется расчетом; источник должен быть схемой, примененной алгоритмом подбора. |
| `SourceCooperationSchemePosition` | `CooperationSchemePosition` | Формируется расчетом; позиция должна принадлежать `SourceCooperationScheme`. |
| `Supplier` | `GMD.Contractor` | Формируется расчетом или выбирается как поставщик / субподрядчик, доступный текущему `Tenant`. |
| `SubstitutionSource` | `NomenclatureSubstitution` | Формируется расчетом; правило замены должно быть применимо к контексту расчета. |
| `SubstitutedNomenclature` | Расчетный результат / GMD | Формируется расчетом; НП должна соответствовать примененной замене. |
| `SubstitutedNomenclatureVariant` | Расчетный результат / GMD | Формируется расчетом; исполнение должно принадлежать замененной НП. |
| `Parent` | `ProductCompositionPosition` | Родительская строка должна принадлежать тому же `ProductComposition`; циклы в иерархии результата запрещены. |

### 9.14 `UseCondition`

| Поле | Источник значений | Ограничение выбора |
|---|---|---|
| `ManufacturingBill` | Контекст коллекции `ManufacturingBill.UseConditions` | Заполняется Object Runtime через parent link и не выбирается пользователем. |
| `ManufacturingBillPosition` | Контекст коллекции `ManufacturingBillPosition.UseConditions` | Заполняется Object Runtime через parent link и не выбирается пользователем. |
| `ProcessDefinition` | Контекст коллекции `ProcessDefinition.UseConditions` | Заполняется Object Runtime через parent link и не выбирается пользователем. |
| `CooperationScheme` | Контекст коллекции `CooperationScheme.UseConditions` | Заполняется Object Runtime через parent link и не выбирается пользователем. |
| `NomenclatureSubstitution` | Контекст коллекции `NomenclatureSubstitution.UseConditions` | Заполняется Object Runtime через parent link и не выбирается пользователем. |
| `ConditionSubject` | `UseConditionSubject` | Значение системного enum модуля. |
| `ComparisonOperator` | `UseConditionOperator` | Оператор должен быть совместим с типом значения, заданным через `ConditionSubject`. |
| `ProductVariant` | `GMD.NomenclatureVariant` | Заполняется только для `ConditionSubject = ProductVariant`; если владелец условия уже задает исполнение продукта, значение ограничивается исполнением владельца. |
| `MainProduct` | `GMD.Nomenclature` | Заполняется только для `ConditionSubject = MainProduct`. |
| `MainProductNumber` | Int | Заполняется только для `ConditionSubject = MainProductNumber`. |
| `ProductOrder` | внешний `ProductOrder` | Заполняется только для `ConditionSubject = ProductOrder`. |
| `ProductOrderPosition` | внешний `ProductOrderPosition` | Заполняется только для `ConditionSubject = ProductOrderPosition`. |
| `DemandGroup` | внешний `DemandGroup` | Заполняется только для `ConditionSubject = DemandGroup`. |
| `Project` | внешний `Project` | Заполняется только для `ConditionSubject = Project`. |
| `ProjectPhase` | внешний `ProjectPhase` | Заполняется только для `ConditionSubject = ProjectPhase`. |
| `ItemParameter` | `GMD.NomenclatureParameter` | Заполняется только для `ConditionSubject = ItemParameter`; значения задаются коллекцией `ParameterValues`. |

### 9.15 Выбор `UnitOfOperation`

Все поля с типом `UnitOfOperation` используют функции GMD:

- получить доступные единицы операции для НП и области применения;
- определить единицу операции по умолчанию;
- проверить применимость выбранной единицы операции;
- пересчитать количество в базовую ЕИ НП и обратно.

Список выбора `UnitOfOperation` не формируется прямым выбором из справочника `Unit`.

Количественные поля, для которых в доменной модели предусмотрена базовая пара, хранят пользовательское значение в выбранной `UnitOfOperation` и базовое значение в базовой ЕИ НП. Базовые поля вида `...BaseUnit` являются хранимыми производными полями: Object Runtime handler пересчитывает их через функции GMD при сохранении и не дает редактировать независимо.

### 9.16 Временные нормы

Поля `RateForSetupTime`, `RateForProcessingTime`, `RateForAuxiliaryTime`, `RateForMachineTime`, `RateForWaitingTime`, `RateForIdleTime`, `RateForTransportTime`, `RateForTeardownTime` хранятся в секундах.

Object Runtime для UI и API разделяет:

- единицу хранения: секунды;
- пользовательское отображение: с учетом общей настройки `Common.TimeAmountFormat` (`Standard` = `hh:mm:ss`, `Industrial` = нормо-часы с тремя десятичными знаками);
- ввод: значение из пользовательского формата пересчитывается в секунды перед сохранением.

Эти поля не получают `DurationUnit` и не наследуют `ProcessSegmentBase.DurationUnit`. `DurationUnit` относится к `ProcessSegmentBase.DurationRate` и `CooperationSchemePosition.LeadTime`.

### 9.17 Продуктивный подбор

Продуктивный подбор не использует `WorkflowStateCode` как самостоятельный бизнес-фильтр.

Подбор выполняется по:

- `IsDeleted = false`;
- `IsArchived = false`;
- нужной области использования;
- `AllowNewOperationalSelection = true`, если создается новая рабочая ссылка;
- `ValidFrom` / `ValidTo`;
- условиям применимости;
- совместимости связанных ревизий.

## 10. Runtime-действия и обработчики

Стандартные действия чтения, создания, изменения и удаления выполняются через Object Runtime.

Предметные проверки модуля подключаются как валидаторы типа объекта и описаны в `05_rules.md`.

| Сценарий | Runtime-решение |
|---|---|
| Создание корневой ревизии | Object Runtime назначает `TenantId`, применяет Common-поля и вызывает валидаторы ревизионного объекта. |
| Создание строки коллекции | Object Runtime наследует `TenantId` владельца, заполняет ссылку на владельца из контекста коллекции и применяет контракт базового типа строки. |
| Сохранение корневой ревизии | Поля `SubmittedForApprovalAt` и `ApprovedAt` могут изменяться workflow-действиями переходов или пользовательским сохранением, если текущая workflow-policy разрешает корректировку этих полей. |
| Сохранение наследника `MaterialPositionBase` | Обработчик пересчитывает базовые нормы через функции GMD и запрещает независимое редактирование базовой пары. |
| Сохранение наследника `ProcessSegmentBase` | Если `NumberInProcess` не задан, обработчик заполняет его из `Number` с длиной `ProcessTemplate.SegmentNumberInProcessLength`; если `Name` не задан и выбран `LabourType`, заполняет `Name = LabourType.Name`; если `DurationRateType` не задан, применяет `PerLot`; если `DurationUnit` не задан, применяет `WorkShift`. |
| Сохранение наследника `ProcessOperationBase` | Если `NumberInProcess` не задан, обработчик заполняет его из `Number` с длиной `ProcessTemplate.OperationNumberInProcessLength`; временные нормы операции пересчитываются из пользовательского формата времени в секунды. |
| Сохранение наследника `ProcessStepBase` | Если `NumberInProcess` не задан, обработчик заполняет его из `Number` с длиной `ProcessTemplate.StepNumberInProcessLength`. |
| Сохранение наследника `ProcessEquipmentPositionBase`, `ProcessLabourPositionBase` | Обработчик пересчитывает пользовательский формат времени в секунды и хранит секунды. |
| Сохранение наследника `ProcessEquipmentPositionBase` | Если `CalculateRateForProcessingTime = true`, обработчик рассчитывает `RateForProcessingTime` из составляющих времени. |
| Создание или сохранение `ProcessSegment` / `ProcessOperation` | Обработчик применяет значения по умолчанию из `LabourType`, `ProcessingOperation` и `ManufacturingOperationType` по алгоритму `13_operations.md`. |
| Сохранение `UseCondition` | Валидатор проверяет, что заполнена ровно одна ссылка владельца, а также совместимость `ConditionSubject`, `ComparisonOperator` и заполненного типизированного значения условия; для `ItemParameter` проверяется коллекция `ParameterValues`. |
| Сохранение `ProcessDefinition.ManufacturingBill` в черновике | Разрешается ручной выбор только индивидуальной спецификации ТО (`IsForProcessDefinition = true`). Для общей технологии поле остается пустым, а спецификация подбирается отдельным алгоритмом применения. |
| Создание новой ревизии `ManufacturingBill`, `ProcessDefinition`, `CooperationScheme` | Выполняется предметным действием `CreateNewRevision`; алгоритм описан в `13_operations.md`. |
| Создание копии технологии | Выполняется предметным действием; алгоритм описан в `13_operations.md`. |
| Анализ состава | Выполняется предметным действием; результат сохраняется в `ProductComposition`. |

Стандартное сохранение карточки и выполнение workflow-перехода не описываются как предметные операции модуля. Workflow-команды и проверки переходов описаны в `04_workflows.md`.

## 11. Runtime-публикация пакетных операций

Пакетные операции v1 публикуются как предметные действия Object Runtime, если они запускаются пользователем или внешним сценарием через стандартный runtime-контракт.

| Операция | Объем данных | Обязательные правила | Контроль последствий |
|---|---:|---|---|
| `CreateNewRevision` | Один ревизионный объект и его состав | Права `Read` исходной ревизии и `Create` нового объекта; копирование владельцев, строк, условий и областей использования по `13_operations.md`. | Новая ревизия создается в `Draft`; старая ревизия не изменяется скрыто. |
| `CopyProcessDefinition` | Одна технология и ее состав | Права `Read` исходной технологии и `Create` новой технологии; валидация ссылок на состав и внешние объекты. | Копия получает собственную ревизионную цепочку или параметры, заданные сценарием копирования. |
| `RunProductCompositionAnalysis` | Расчетный результат анализа состава | Право `Create` на `ProductComposition` для нового анализа или `Update` на `ProductComposition` для перерасчета существующего; продуктивный подбор нормативных данных; проверка условий применимости. | Результат фиксирует источники норм, сообщения и расчетные статусы. |

Пакетные операции не отключают Common-фильтры архивирования, удаления, tenant-области и права доступа. Исключения должны быть явно описаны в `13_operations.md`.

## 12. Архивирование и удаление

Модуль использует Common-модель архивирования и удаления.

Для объектов с `ProcessDocumentLifecycle` архивирование выполняется через workflow-переход в состояние с `IsArchiveState = true`. Workflow Runtime синхронизирует `IsArchived = true` по правилам Common/Object Runtime.

Через `ProcessDocumentLifecycle` архивируются:

- `ManufacturingBill`;
- `ProcessDefinition`;
- `CooperationScheme`.

Для объектов без `ProcessDocumentLifecycle` используется Common-архивирование, если оно включено контрактом объекта:

- `NomenclatureSubstitution`;
- `ProductComposition`.

Строки коллекций архивируются или удаляются только по правилам владельца и режима коллекции. Прямое архивирование строки коллекции не должно обходить права и состояние владельца.

Для одного типа объекта нельзя смешивать прямое архивирование Common и архивирование через workflow.

## 13. Baseline и платформенные требования

Baseline модуля должен объявить:

- object type metadata для всех публикуемых конкретных типов;
- наследование конкретных object type от абстрактных базовых типов доменной модели;
- применение runtime-контрактов базовых типов к их наследникам;
- system enum metadata для перечислений модуля;
- правила `Presentation`;
- наборы данных `List`, `Lookup`, `Details`, `CollectionRows`;
- коллекции Object Runtime и их режимы;
- lookup-ограничения ссылочных полей;
- обработчики пересчета `UnitOfOperation` и базовых количеств;
- обработчики ввода/вывода временных норм в секундах;
- валидаторы условий применимости;
- предметные действия `CreateNewRevision`, `CopyProcessDefinition`, `RunProductCompositionAnalysis`.

Абстрактные базовые типы не создают пользовательские списки и карточки. Их контракт применяется только через наследников.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | Публикуемые типы объектов; 5 `ProcessOperationBase`; 8 Ресурсные позиции технологии | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-19 12:02 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [87c0b0a5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/87c0b0a51fdabf232ce704082d41545ae007bedc) |
