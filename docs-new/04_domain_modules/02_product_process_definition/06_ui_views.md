---
id: DOC-04-02-06
title: 'Пользовательские представления - 02 Составы и технологии'
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

# Пользовательские представления - 02 Составы и технологии

## 1. Назначение документа

Документ фиксирует списки, списки выбора, карточки, вкладки и предметные действия интерфейса модуля `02 Product & Process Definition`.

Документ не повторяет полный состав полей доменной модели, стандартные действия Object Runtime и стандартный WF-блок карточки. Поля описаны в `02_domain_model.md`, правила - в `05_rules.md`, workflow - в `04_workflows.md`.

## 2. Общие правила интерфейса

- Списки по умолчанию показывают только неархивные и неудаленные записи.
- Ссылочные поля выбирают значения через списки выбора модуля-владельца.
- Уже сохраненные ссылки на архивные объекты отображаются.
- Карточки ревизионных объектов показывают стандартный блок жизненного цикла.
- Области использования отображаются как пользовательский блок, а не как технические поля.
- Поля типа `UnitOfOperation` показывают список допустимых единиц операции через функции GMD: доступные единицы, единица по умолчанию и проверка применимости. Такой список не является простым lookup-ом по справочнику `Unit`.
- Временные нормы `RateForSetupTime`, `RateForProcessingTime`, `RateForAuxiliaryTime`, `RateForMachineTime`, `RateForWaitingTime`, `RateForIdleTime`, `RateForTransportTime`, `RateForTeardownTime` хранятся в секундах. В UI они вводятся и отображаются с учетом общей настройки `Common.TimeAmountFormat`: `Standard` = `hh:mm:ss`, `Industrial` = нормо-часы с тремя десятичными знаками; при сохранении значение пересчитывается в секунды.

Блок областей использования:

```text
Области использования
  [ ] Планирование и разузлование
  [ ] Производство
  [ ] Калькуляция / себестоимость
  [ ] Разрешить выбор в новых рабочих документах
```

## 3. Шаблоны

### Шаблоны спецификаций. Список

| Параметр | Значение |
|---|---|
| Титул | Шаблоны спецификаций |
| Вид представления | Список |
| Код представления | `ManufacturingBillTemplate_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Настройки / Шаблоны спецификаций |
| Объект данных | `ManufacturingBillTemplate` |
| Источник данных | `ManufacturingBillTemplate.List` |

Колонки: `Code`, `Name`, `IsDefault`, `PositionNumberingStep`.

### Шаблон спецификации. Карточка

| Параметр | Значение |
|---|---|
| Титул | Шаблон спецификации |
| Вид представления | Карточка |
| Код представления | `ManufacturingBillTemplate_DetailView` |
| Тип представления | `ObjectForm` |
| Объект данных | `ManufacturingBillTemplate` |
| Источник данных | `ManufacturingBillTemplate.Details` |

Основные поля: `Code`, `Name`, `IsDefault`, `PositionNumberingStep`.

### Шаблоны технологий. Список

| Параметр | Значение |
|---|---|
| Титул | Шаблоны технологий |
| Вид представления | Список |
| Код представления | `ProcessTemplate_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Настройки / Шаблоны технологий |
| Объект данных | `ProcessTemplate` |
| Источник данных | `ProcessTemplate.List` |

Колонки: `Code`, `Name`, `IsDefault`, `SegmentNumberingStep`, `OperationNumberingStep`, `StepNumberingStep`, `ResourcePositionNumberingStep`.

### Шаблон технологии. Карточка

| Параметр | Значение |
|---|---|
| Титул | Шаблон технологии |
| Вид представления | Карточка |
| Код представления | `ProcessTemplate_DetailView` |
| Тип представления | `ObjectForm` |
| Объект данных | `ProcessTemplate` |
| Источник данных | `ProcessTemplate.Details` |

Основные поля: `Code`, `Name`, `IsDefault`, `SegmentNumberingStep`, `OperationNumberingStep`, `StepNumberingStep`, `ResourcePositionNumberingStep`, `SegmentNumberInProcessLength`, `OperationNumberInProcessLength`, `StepNumberInProcessLength`.

## 4. Классификаторы технологии

Классификаторы технологии являются обычными справочниками модуля без `ProcessDocumentLifecycle`.

### Виды работ

| Параметр | Значение |
|---|---|
| Титул | Виды работ |
| Вид представления | Список |
| Код представления | `LabourType_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Справочники / Виды работ |
| Объект данных | `LabourType` |
| Источник данных | `LabourType.List` |

Колонки: `Code`, `Name`.

Карточка `LabourType_DetailView`: основные поля `Code`, `Name`.

### Виды операций

| Параметр | Значение |
|---|---|
| Титул | Виды операций |
| Вид представления | Список |
| Код представления | `ManufacturingOperationType_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Справочники / Виды операций |
| Объект данных | `ManufacturingOperationType` |
| Источник данных | `ManufacturingOperationType.List` |

Колонки: `Code`, `Name`, `ConfirmationStages`, `ActualAccountingMode`.

Карточка `ManufacturingOperationType_DetailView`: основные поля `Code`, `Name`, `Description`, `ConfirmationStages`, `ActualAccountingMode`.

### Технологические операции по классификатору

| Параметр | Значение |
|---|---|
| Титул | Технологические операции |
| Вид представления | Список / дерево |
| Код представления | `ProcessingOperation_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Справочники / Классификатор операций |
| Объект данных | `ProcessingOperation` |
| Источник данных | `ProcessingOperation.List` |
| Иерархия | `ProcessingOperation.Parent` |

Колонки: `Code`, `Name`, `Parent`, `LabourType`, `ManufacturingOperationType`.

Карточка `ProcessingOperation_DetailView`: основные поля `Code`, `Name`, `Parent`, `LabourType`, `ManufacturingOperationType`.

### Условия труда

| Параметр | Значение |
|---|---|
| Титул | Условия труда |
| Вид представления | Список |
| Код представления | `WorkEnvironment_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Справочники / Условия труда |
| Объект данных | `WorkEnvironment` |
| Источник данных | `WorkEnvironment.List` |

Колонки: `Code`, `Name`.

Карточка `WorkEnvironment_DetailView`: основные поля `Code`, `Name`, `Description`.

## 5. Спецификации

### Спецификации. Список

| Параметр | Значение |
|---|---|
| Титул | Спецификации |
| Вид представления | Список |
| Код представления | `ManufacturingBill_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Составы / Спецификации |
| Объект данных | `ManufacturingBill` |
| Источник данных | `ManufacturingBill.List` |
| Иерархия | `ManufacturingBill.Initial` |

Колонки:

| Колонка | Поле |
|---|---|
| Код | `Code` |
| Наименование | `Name` |
| Шаблон | `ManufacturingBillTemplate` |
| Продукт | `Product` |
| Исполнение | `ProductVariant` |
| Тип процесса | `ProcessType` |
| Ревизия | `Revision` |
| Отправлено на утверждение | `SubmittedForApprovalAt` |
| Утверждено | `ApprovedAt` |
| Действует с | `ValidFrom` |
| Действует по | `ValidTo` |
| Планирование | `UseInPlanning` |
| Производство | `UseInProduction` |
| Калькуляция | `UseInCosting` |

Фильтры:

| Фильтр | Поле | Тип | Источник |
|---|---|---|---|
| Поиск | `Code`, `Name`, `Product` | Строка поиска | Ручной ввод |
| Продукт | `Product` | Быстрый фильтр | `Nomenclature_LookupView` |
| Тип процесса | `ProcessType` | Быстрый фильтр | `GMD.ProcessType` |
| Только для планирования | `UseInPlanning` | Быстрый фильтр | Boolean |
| Только для производства | `UseInProduction` | Быстрый фильтр | Boolean |
| Показать архивные | `IsArchived` | Системный фильтр | Common |

Предметные действия:

| Действие | Назначение |
|---|---|
| Создать новую ревизию | Создать ревизию на основании выбранной спецификации. |
| Подобрать действующую спецификацию | Выполнить предметный подбор по контексту, дате, области использования и условиям. |

### Спецификации. Список выбора

| Параметр | Значение |
|---|---|
| Код представления | `ManufacturingBill_LookupView` |
| Вид представления | Список выбора |
| Тип представления | `LookupListView` |
| Объект данных | `ManufacturingBill` |
| Источник данных | `ManufacturingBill.Lookup` |

Список выбора использует колонки списка и применяет контекстные фильтры поля, из которого он открыт: продукт, исполнение, тип процесса, дата, область использования, `AllowNewOperationalSelection`.

### Спецификация. Карточка

| Параметр | Значение |
|---|---|
| Титул | Спецификация |
| Вид представления | Карточка |
| Код представления | `ManufacturingBill_DetailView` |
| Тип представления | `ObjectForm` |
| Объект данных | `ManufacturingBill` |
| Источник данных | `ManufacturingBill.Details` |
| Режимы карточки | Создание, изменение, просмотр |

Вкладки:

| Вкладка | Назначение |
|---|---|
| Основные данные | Код, наименование, шаблон, продукт, исполнение, тип процесса, количество, единицы, период действия. Даты `SubmittedForApprovalAt` и `ApprovedAt` доступны по workflow-policy текущего состояния. |
| Области использования | Пользовательский блок флагов использования. |
| Позиции | Строки состава `ManufacturingBillPosition`. |
| Условия применения | Коллекция `UseCondition` текущей спецификации. |
| Индивидуальные технологии | Связанный список `ProcessDefinition`, где `ProcessDefinition.ManufacturingBill` равен текущей ревизии индивидуальной спецификации ТО. |
| Администрирование | Системная секция Common. |

### Спецификация. Вкладка `Позиции`

| Параметр | Значение |
|---|---|
| Вид представления | Вкладка коллекции |
| Тип представления | Элемент `ObjectForm` |
| Код представления | `ManufacturingBill_Positions_ListView` |
| Владелец | `ManufacturingBill` |
| Источник данных | `ManufacturingBill.Positions` |
| Коллекция | `Positions` |
| Строка коллекции | `ManufacturingBillPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки:

| Колонка | Поле |
|---|---|
| Номер | `PositionNumber` |
| Тип | `InputOutputType` |
| Компонент | `Component` |
| Исполнение | `ComponentVariant` |
| ЕИ нормы | `RateUnit` |
| Норма на наладку | `RateForSetup` |
| Норма на выполнение | `RateForProcessing` |
| Размер партии | `ProductLotSize` |
| КИМ | `MaterialUtilizationFactor` |
| Спецификация компонента | `ComponentManufacturingBill` |
| Технология компонента | `ComponentProcessDefinition` |
| Способ обеспечения | `ObtainMethod` |
| Действует с | `ValidFrom` |
| Действует по | `ValidTo` |

Форма строки показывает те же поля и коллекцию условий применения строки.

## 6. Технологические описания

### Технологические описания. Список

| Параметр | Значение |
|---|---|
| Титул | Технологические описания |
| Вид представления | Список |
| Код представления | `ProcessDefinition_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Технологии / Технологические описания |
| Объект данных | `ProcessDefinition` |
| Источник данных | `ProcessDefinition.List` |
| Иерархия | `ProcessDefinition.Initial` |

Колонки:

| Колонка | Поле |
|---|---|
| Код | `Code` |
| Наименование | `Name` |
| Шаблон | `ProcessTemplate` |
| Продукт | `Product` |
| Исполнение | `ProductVariant` |
| Тип процесса | `ProcessType` |
| Индивидуальная спецификация ТО | `ManufacturingBill` |
| Ревизия | `Revision` |
| Отправлено на утверждение | `SubmittedForApprovalAt` |
| Утверждено | `ApprovedAt` |
| Действует с | `ValidFrom` |
| Действует по | `ValidTo` |
| Планирование | `UseInPlanning` |
| Производство | `UseInProduction` |
| Калькуляция | `UseInCosting` |

Фильтры:

| Фильтр | Поле | Тип | Источник |
|---|---|---|---|
| Поиск | `Code`, `Name`, `Product` | Строка поиска | Ручной ввод |
| Продукт | `Product` | Быстрый фильтр | `Nomenclature_LookupView` |
| Тип процесса | `ProcessType` | Быстрый фильтр | `GMD.ProcessType` |
| Индивидуальная спецификация ТО | `ManufacturingBill` | Быстрый фильтр | `ManufacturingBill_LookupView` |
| Показать архивные | `IsArchived` | Системный фильтр | Common |

Предметные действия:

| Действие | Назначение |
|---|---|
| Создать новую ревизию | Создать новую ревизию технологии. |
| Создать копию | Создать новую технологию на основании текущей. |
| Подобрать действующую технологию | Выполнить подбор по контексту, дате, области использования и условиям. |
| Анализ состава | Создать расчетный анализ состава по технологии. |
| Изменить компонент спецификации ТО | Создать, изменить или удалить компонент индивидуальной спецификации ТО для редактируемой технологии. |
| Создать новую ревизию с изменением компонента | Создать новую ревизию технологии, новую ревизию ее индивидуальной спецификации ТО и применить предметное изменение компонента. |
| Создать копию с изменением компонента | Создать копию технологии, копию ее индивидуальной спецификации ТО и применить предметное изменение компонента. |

### Технологическое описание. Карточка

| Параметр | Значение |
|---|---|
| Титул | Технологическое описание |
| Вид представления | Карточка |
| Код представления | `ProcessDefinition_DetailView` |
| Тип представления | `ObjectForm` |
| Объект данных | `ProcessDefinition` |
| Источник данных | `ProcessDefinition.Details` |
| Режимы карточки | Создание, изменение, просмотр |

Вкладки:

| Вкладка | Назначение |
|---|---|
| Основные данные | Код, наименование, шаблон, продукт, исполнение, тип процесса, индивидуальная спецификация ТО, МХ выпуска, ячейка выпуска, период действия. Даты `SubmittedForApprovalAt` и `ApprovedAt` доступны по workflow-policy текущего состояния. |
| Области использования | Пользовательский блок флагов использования. |
| Структура технологии | Дерево `ProcessDefinition` -> `ProcessSegment` -> `ProcessOperation` -> `ProcessTransition` и карточка выбранного элемента. |
| Условия применения | Коллекция `UseCondition` текущей технологии. |
| Технологические материалы | Связанный список документов и материалов технологии через `Document Management`. |
| Администрирование | Системная секция Common. |

Предметные действия карточки совпадают с предметными действиями списка для текущей технологии.

### Технологическое описание. Рабочая область `Структура технологии`

| Параметр | Значение |
|---|---|
| Код представления | `ProcessDefinition_StructureView` |
| Вид представления | Дерево + карточка выбранного элемента |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessDefinition` |
| Источник данных | `ProcessDefinition.Structure` |
| Режимы карточки | Изменение, просмотр |

Левая часть рабочей области показывает дерево:

```text
ProcessDefinition
  ProcessSegment
    ProcessOperation
      ProcessTransition
```

При выборе узла в правой части открывается карточка соответствующего объекта: технологии, передела, операции или перехода. Вкладки и коллекции этих карточек описаны в разделах ниже.

### Технологическое описание. Вкладка `Технологические материалы`

| Параметр | Значение |
|---|---|
| Код представления | `ProcessDefinition_Documents_RelatedListView` |
| Вид представления | Связанный список `Document Management` |
| Владелец | `ProcessDefinition` |
| Источник данных | Связи документов `Document Management` с бизнес-объектом `ProcessDefinition` |
| Режимы карточки | Изменение, просмотр |

Колонки определяются `Document Management`: тип/вид документа, наименование документа, версия, основной признак, комментарий, срок действия или другие поля, предусмотренные документным модулем.

### Структура технологии. Переделы

| Параметр | Значение |
|---|---|
| Код представления | `ProcessDefinition_Segments_ListView` |
| Вид представления | Коллекция в рабочей области структуры |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessDefinition` |
| Источник данных | `ProcessDefinition.Segments` |
| Коллекция | `Segments` |
| Строка коллекции | `ProcessSegment` |
| Режимы карточки | Изменение, просмотр |

Колонки:

| Колонка | Поле |
|---|---|
| Номер | `Number` |
| Номер в ТП | `NumberInProcess` |
| Наименование | `Name` |
| Вид работы | `LabourType` |
| Ответственная орг. единица | `ResponsibleOrgUnit` |
| Субподрядный | `IsSubcontracted` |
| Автовыполнение | `IsAutoComplete` |
| Точка учета | `IsAccountingPoint` |
| Создавать производственную партию | `IsCreateManufacturingLot` |
| Присваивать серийные номера | `IsAssignSerialNumbers` |
| МХ выпуска | `OutputWarehouse` |
| Ячейка выпуска | `OutputWarehouseBin` |
| МХ брака | `ScrapWarehouse` |
| Ячейка брака | `ScrapWarehouseBin` |
| Тип длительности | `DurationRateType` |
| Длительность | `DurationRate` |
| ЕИ длительности | `DurationUnit` |
| Временной буфер | `TimeBuffer` |
| Коэффициент выпуска годных | `YieldFactor` |
| Коэффициент технологических потерь | `ManufacturingLossFactor` |
| Потери | `LossQty` |
| Правило округления | `FactorRoundingRule` |

### Передел. Компоненты

| Параметр | Значение |
|---|---|
| Код представления | `ProcessSegment_MaterialPositions_ListView` |
| Вид представления | Коллекция выбранного передела |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessSegment` |
| Источник данных | `ProcessSegment.MaterialPositions` |
| Коллекция | `MaterialPositions` |
| Строка коллекции | `ProcessSegmentMaterialPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `ManufacturingBillPosition`, `InputOutputType`, `Component`, `ComponentVariant`, `RateUnit`, `RateForSetup`, `RateForProcessing`, `ProductLotSize`.

### Передел. Оборудование

| Параметр | Значение |
|---|---|
| Код представления | `ProcessSegment_EquipmentPositions_ListView` |
| Вид представления | Коллекция выбранного передела |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessSegment` |
| Источник данных | `ProcessSegment.EquipmentPositions` |
| Коллекция | `EquipmentPositions` |
| Строка коллекции | `ProcessSegmentEquipmentPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `WorkPlaceGroup`, `WorkPlace`, `EquipmentQty`, `IsBasic`, `RateForSetupTime`, `RateForAuxiliaryTime`, `RateForMachineTime`, `RateForProcessingTime`, `CalculateRateForProcessingTime`.

### Передел. Персонал

| Параметр | Значение |
|---|---|
| Код представления | `ProcessSegment_LabourPositions_ListView` |
| Вид представления | Коллекция выбранного передела |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessSegment` |
| Источник данных | `ProcessSegment.LabourPositions` |
| Коллекция | `LabourPositions` |
| Строка коллекции | `ProcessSegmentLabourPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `ProcessSegmentEquipmentPosition`, `Profession`, `Grade`, `LabourQty`, `IsBasic`, `RateForSetupTime`, `RateForProcessingTime`.

### Передел. Оснастка

| Параметр | Значение |
|---|---|
| Код представления | `ProcessSegment_ToolingPositions_ListView` |
| Вид представления | Коллекция выбранного передела |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessSegment` |
| Источник данных | `ProcessSegment.ToolingPositions` |
| Коллекция | `ToolingPositions` |
| Строка коллекции | `ProcessSegmentToolingPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `ProcessSegmentEquipmentPosition`, `Tooling`, `ToolingQty`.

Поле `Tooling` выбирает внешний объект `ToolBase`; допустимые конкретные типы задает модуль-владелец ресурсов.

### Передел. Операции

| Параметр | Значение |
|---|---|
| Код представления | `ProcessSegment_Operations_ListView` |
| Вид представления | Коллекция выбранного передела |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessSegment` |
| Источник данных | `ProcessSegment.Operations` |
| Коллекция | `Operations` |
| Строка коллекции | `ProcessOperation` |
| Режимы карточки | Изменение, просмотр |

Колонки:

| Колонка | Поле |
|---|---|
| Номер | `Number` |
| Номер в ТП | `NumberInProcess` |
| Наименование | `Name` |
| ТО по классификатору | `ProcessingOperation` |
| Вид работы | `LabourType` |
| Вид операции | `ManufacturingOperationType` |
| Схема подтверждения | `ConfirmationStages` |
| Группа оплаты | `PaymentGroup` |
| Условия труда | `WorkEnvironment` |
| Ответственная орг. единица | `ResponsibleOrgUnit` |
| Орг. единица выполнения | `ExecutionOrgUnit` |
| Субподрядная | `IsSubcontracted` |
| КОИД | `NumberOfItemsSimultaneouslyProcessed` |
| Коэффициент производительности | `ProductivityFactor` |
| Тип Тпз | `RateTypeForSetup` |
| Наладка | `RateForSetupTime` |
| Тип Тшт | `RateTypeForProcessing` |
| Выполнение | `RateForProcessingTime` |
| Вспомогательное | `RateForAuxiliaryTime` |
| Машинное | `RateForMachineTime` |
| Ожидание | `RateForWaitingTime` |
| Простой | `RateForIdleTime` |
| Транспортировка | `RateForTransportTime` |
| Рассчитать Тшт | `CalculateRateForProcessingTime` |
| Завершение | `RateForTeardownTime` |

### Операция. Переходы

| Параметр | Значение |
|---|---|
| Код представления | `ProcessOperation_Transitions_ListView` |
| Вид представления | Коллекция выбранной операции |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessOperation` |
| Источник данных | `ProcessOperation.Transitions` |
| Коллекция | `Transitions` |
| Строка коллекции | `ProcessTransition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `Number`, `NumberInProcess`, `Description`, `SpecialInstructions`.

### Операция. Компоненты

| Параметр | Значение |
|---|---|
| Код представления | `ProcessOperation_MaterialPositions_ListView` |
| Вид представления | Коллекция выбранной операции |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessOperation` |
| Источник данных | `ProcessOperation.MaterialPositions` |
| Коллекция | `MaterialPositions` |
| Строка коллекции | `ProcessOperationMaterialPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `ManufacturingBillPosition`, `InputOutputType`, `Component`, `ComponentVariant`, `RateUnit`, `RateForSetup`, `RateForProcessing`, `ProductLotSize`.

### Операция. Оборудование

| Параметр | Значение |
|---|---|
| Код представления | `ProcessOperation_EquipmentPositions_ListView` |
| Вид представления | Коллекция выбранной операции |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessOperation` |
| Источник данных | `ProcessOperation.EquipmentPositions` |
| Коллекция | `EquipmentPositions` |
| Строка коллекции | `ProcessOperationEquipmentPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `WorkPlaceGroup`, `WorkPlace`, `EquipmentQty`, `IsBasic`, `RateForSetupTime`, `RateForAuxiliaryTime`, `RateForMachineTime`, `RateForProcessingTime`, `CalculateRateForProcessingTime`.

### Операция. Персонал

| Параметр | Значение |
|---|---|
| Код представления | `ProcessOperation_LabourPositions_ListView` |
| Вид представления | Коллекция выбранной операции |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessOperation` |
| Источник данных | `ProcessOperation.LabourPositions` |
| Коллекция | `LabourPositions` |
| Строка коллекции | `ProcessOperationLabourPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `ProcessOperationEquipmentPosition`, `Profession`, `Grade`, `LabourQty`, `IsBasic`, `RateForSetupTime`, `RateForProcessingTime`.

### Операция. Оснастка

| Параметр | Значение |
|---|---|
| Код представления | `ProcessOperation_ToolingPositions_ListView` |
| Вид представления | Коллекция выбранной операции |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProcessOperation` |
| Источник данных | `ProcessOperation.ToolingPositions` |
| Коллекция | `ToolingPositions` |
| Строка коллекции | `ProcessOperationToolingPosition` |
| Режимы карточки | Изменение, просмотр |

Колонки: `PositionNumber`, `ProcessOperationEquipmentPosition`, `Tooling`, `ToolingQty`.

Поле `Tooling` выбирает внешний объект `ToolBase`; допустимые конкретные типы задает модуль-владелец ресурсов.

## 7. Замены номенклатуры

### Замены номенклатуры. Список

| Параметр | Значение |
|---|---|
| Титул | Замены номенклатуры |
| Вид представления | Список |
| Код представления | `NomenclatureSubstitution_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Составы / Замены номенклатуры |
| Объект данных | `NomenclatureSubstitution` |
| Источник данных | `NomenclatureSubstitution.List` |

Колонки:

| Колонка | Поле |
|---|---|
| Заменяемая номенклатура | `SubstitutedNomenclature` |
| Заменяемое исполнение | `SubstitutedNomenclatureVariant` |
| Заменяющая номенклатура | `ReplacementNomenclature` |
| Заменяющее исполнение | `ReplacementNomenclatureVariant` |
| Тип замены | `SubstitutionType` |
| Обязательная | `IsMandatorySubstitution` |
| Обратная | `IsReverseSubstitution` |
| Коэффициент | `SubstitutionFactor` |
| ЕИ нормы | `RateUnit` |
| Действует с | `ValidFrom` |
| Действует по | `ValidTo` |

Фильтры:

| Фильтр | Поле | Тип | Источник |
|---|---|---|---|
| Поиск | `SubstitutedNomenclature`, `ReplacementNomenclature` | Строка поиска | Ручной ввод |
| Заменяемая НП | `SubstitutedNomenclature` | Быстрый фильтр | `Nomenclature_LookupView` |
| Заменяющая НП | `ReplacementNomenclature` | Быстрый фильтр | `Nomenclature_LookupView` |
| Тип замены | `SubstitutionType` | Быстрый фильтр | `NomenclatureSubstitutionType` |
| Показать архивные | `IsArchived` | Системный фильтр | Common |

### Замена номенклатуры. Карточка

| Параметр | Значение |
|---|---|
| Титул | Замена номенклатуры |
| Вид представления | Карточка / drawer |
| Код представления | `NomenclatureSubstitution_DetailView` |
| Тип представления | `ObjectForm` |
| Объект данных | `NomenclatureSubstitution` |
| Источник данных | `NomenclatureSubstitution.Details` |

Вкладки:

| Вкладка | Назначение |
|---|---|
| Основные данные | Заменяемая и заменяющая НП, тип замены, признаки обязательности и обратности, коэффициент или норма замены, ограничение по спецификации/технологии и период действия. |
| Условия применения | Коллекция `UseCondition` текущего правила замены. |
| Администрирование | Системная секция Common. |

## 8. Схемы кооперации

### Схемы кооперации. Список

| Параметр | Значение |
|---|---|
| Титул | Схемы кооперации |
| Вид представления | Список |
| Код представления | `CooperationScheme_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Технологии / Схемы кооперации |
| Объект данных | `CooperationScheme` |
| Источник данных | `CooperationScheme.List` |
| Иерархия | `CooperationScheme.Initial` |

Колонки:

| Колонка | Поле |
|---|---|
| Код | `Code` |
| Наименование | `Name` |
| Изделие | `MainProduct` |
| Исполнение изделия | `MainProductVariant` |
| Ревизия | `Revision` |
| Действует с | `ValidFrom` |
| Действует по | `ValidTo` |
| Планирование | `UseInPlanning` |
| Производство | `UseInProduction` |
| Калькуляция | `UseInCosting` |

Фильтры:

| Фильтр | Поле | Тип | Источник |
|---|---|---|---|
| Поиск | `Code`, `Name`, `MainProduct` | Строка поиска | Ручной ввод |
| Изделие | `MainProduct` | Быстрый фильтр | `Nomenclature_LookupView` |
| Только для планирования | `UseInPlanning` | Быстрый фильтр | Boolean |
| Только для производства | `UseInProduction` | Быстрый фильтр | Boolean |
| Показать архивные | `IsArchived` | Системный фильтр | Common |

### Схема кооперации. Карточка

| Параметр | Значение |
|---|---|
| Титул | Схема кооперации |
| Вид представления | Карточка |
| Код представления | `CooperationScheme_DetailView` |
| Тип представления | `ObjectForm` |
| Объект данных | `CooperationScheme` |
| Источник данных | `CooperationScheme.Details` |
| Режимы карточки | Создание, изменение, просмотр |

Карточка содержит вкладки `Основные данные`, `Области использования`, `Позиции`, `Условия применения`, `Администрирование`.

### Схема кооперации. Вкладка `Позиции`

| Параметр | Значение |
|---|---|
| Код представления | `CooperationScheme_Positions_ListView` |
| Вид представления | Вкладка коллекции |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `CooperationScheme` |
| Источник данных | `CooperationScheme.Positions` |
| Коллекция | `Positions` |
| Строка коллекции | `CooperationSchemePosition` |

Колонки:

| Колонка | Поле |
|---|---|
| Номер | `PositionNumber` |
| Номенклатура | `Nomenclature` |
| Исполнение НП | `NomenclatureVariant` |
| Спецификация компонента | `ComponentManufacturingBill` |
| Тип изменения | `ChangeType` |
| Способ обеспечения | `NomenclatureObtainMethod` |
| Тип длительности | `DurationType` |
| Длительность | `LeadTime` |
| ЕИ длительности | `LeadTimeUnit` |
| Размер партии | `ProductLotSize` |
| Субподрядчик | `Subcontractor` |

## 9. Общая строка условия применения

Строки `UseCondition` редактируются на вкладке `Условия применения` своего владельца. При создании строки система заполняет ссылку на владельца из контекста карточки.

Колонки списка:

| Колонка | Поле |
|---|---|
| Предмет | `ConditionSubject` |
| Оператор | `ComparisonOperator` |
| Значение | вычисляется по типизированному значению строки |
| Действует с | `ValidFrom` |
| Действует по | `ValidTo` |

Форма строки показывает период действия, `ConditionSubject`, `ComparisonOperator` и только тот блок значения, который соответствует выбранному `ConditionSubject`. Для `ItemParameter` показывается коллекция `ParameterValues`.

Поле `ProductVariant` доступно для ввода, если исполнение продукта не задано владельцем условия. Если исполнение задано владельцем, поле заполняется этим значением и не выбирается пользователем независимо.

## 10. Анализ состава

### Анализы состава. Список

| Параметр | Значение |
|---|---|
| Титул | Анализ составов |
| Вид представления | Список |
| Код представления | `ProductComposition_ListView` |
| Тип представления | `ObjectList` |
| Путь | Составы и технологии / Составы / Анализ составов |
| Объект данных | `ProductComposition` |
| Источник данных | `ProductComposition.List` |

Колонки:

| Колонка | Поле |
|---|---|
| Номер | `CompositionNumber` |
| Дата | `CompositionDate` |
| Тип анализа | `CompositionType` |
| Продукт | `Product` |
| Исполнение продукта | `ProductVariant` |
| Количество | `ProductQty` |
| ЕИ | `Unit` |
| Дата норм | `NormDate` |
| Тип процесса | `ProcessType` |
| Статус расчета | `CompositionStatus` |
| Дата расчета | `CalculationDate` |
| Позиций | `PositionsCount` |
| Предупреждений | `WarningsCount` |
| Ошибок | `ErrorsCount` |

Предметное действие списка `Получить` запускает расчет состава для выбранных записей через операцию `RunProductCompositionAnalysis`.

### Анализ состава. Карточка

| Параметр | Значение |
|---|---|
| Титул | Анализ состава |
| Вид представления | Карточка |
| Код представления | `ProductComposition_DetailView` |
| Тип представления | `ObjectForm` |
| Объект данных | `ProductComposition` |
| Источник данных | `ProductComposition.Details` |

Вкладки:

| Вкладка | Назначение |
|---|---|
| Параметры | Исходные параметры расчета: продукт, количество, дата норм, тип процесса, ручные источники нормативов и флаги расчета. |
| Аналитика | Контекст условий применимости: конечное изделие, серийный номер, спрос, проект, заказ. |
| Значения реквизитов | `ProductCompositionParameterValue`. |
| Позиции | Иерархический список `ProductCompositionPosition`. |
| Сообщения | Сообщение строки `ProductCompositionPosition.Message` и статус строки `ProductCompositionPosition.PositionStatus`. |
| Источники | Переходы к исходным спецификациям, технологиям и схеме кооперации. |

Предметное действие карточки `Получить` запускает расчет текущего анализа состава через операцию `RunProductCompositionAnalysis`.

### Анализ состава. Вкладка `Позиции`

| Параметр | Значение |
|---|---|
| Код представления | `ProductComposition_Positions_ListView` |
| Вид представления | Вкладка коллекции |
| Тип представления | Элемент `ObjectForm` |
| Владелец | `ProductComposition` |
| Источник данных | `ProductComposition.Positions` |
| Коллекция | `Positions` |
| Строка коллекции | `ProductCompositionPosition` |

Колонки:

| Колонка | Поле |
|---|---|
| Уровень | `Level` |
| Номер | `PositionNumber` |
| Номенклатура | `Nomenclature` |
| Исполнение | `NomenclatureVariant` |
| Способ обеспечения | `ObtainMethod` |
| Количество всего | `TotalQty` |
| ЕИ | `Unit` |
| Количество всего в БЕИ | `TotalQtyBaseUnit` |
| Количество на единицу продукта | `PerUnitOfProductQty` |
| Количество на единицу вышестоящего | `PerUnitOfParentQty` |
| Статус строки | `PositionStatus` |
| Сообщение | `Message` |

## 11. Что не входит в собственный UI модуля

| Элемент | Решение |
|---|---|
| Карточки оборудования, рабочих центров, персонала, инструмента, оснастки и средств контроля | `ResourceManagement`. В модуле используются только ссылки на внешние ресурсные объекты; карточки и операции этих объектов принадлежат модулю-владельцу. |
| Карточки документов, файлов, версий, маршрутов согласования | `Document Management`. В модуле явно описывается только связанный список документов технологического описания; универсальные документные вкладки других объектов управляются общесистемным механизмом Document Management. |
| Производственные заказы, задания, WIP и факты операций | Future operational production modules. |
| Стандартные workflow-кнопки в каждой форме | Стандартный WF-блок карточки. |
| Стандартные create/read/update/delete | Object Runtime. |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | Что не входит в собственный UI модуля | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-19 12:02 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [87c0b0a5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/87c0b0a51fdabf232ce704082d41545ae007bedc) |
