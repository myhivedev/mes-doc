---
id: DOC-04-04-03
title: 'Runtime-модель объектов - 04 Управление ресурсами'
type: design
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers: []
scope: domain
module: 04_resource_management
holder: '@axelprosoft'
created_at: 2026-09-02 12:26
created_by: '@axelprosoft'
updated_at: 2026-09-03 10:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Runtime-модель объектов - 04 Управление ресурсами

## 1. Назначение документа

Документ описывает публикацию объектов модуля `04_resource_management` через
Object Runtime: доступные типы объектов, tenant-контекст, наборы данных,
представления, иерархии, ссылочные поля, коллекции, одиночные зависимые
объекты, обработчики и требования к baseline.

Состав объектов, полей, ссылок и логического наследования задан в
[`02_domain_model.md`](02_domain_model.md). Этот документ не заменяет доменную
модель и не добавляет в нее новые предметные объекты.

## 2. Базовое решение

Object Runtime публикует самостоятельные типы модуля и предоставляет для них
стандартные операции чтения, создания, изменения, архивирования, восстановления
и удаления в пределах объявленных возможностей и прав.

Логические базовые типы используются для общих свойств и контрактов. Они не
становятся самостоятельными карточками, списками или источниками выбора, если
на них нет реальной полиморфной ссылки. В модуле 04 исключение составляет
`ToolBase`: на него ссылается нормативная модель модуля 02, поэтому для него
нужен объединенный reference lookup.

Общие правила `Tenant`, аудита, архивирования, удаления, технических полей и
отображения задаются Common и платформой. Модуль 04 объявляет только свои
типы, ссылки, коллекции и предметные ограничения.

В v1 не публикуются и не реализуются:

- универсальный runtime-тип или lookup `Resource`, объединяющий рабочие места,
  персонал и инструмент;
- самостоятельные runtime-типы логических базовых типов;
- движение, выдача, возврат, остатки, ремонт и фактическая эксплуатация
  `Tooling` и `Gage`;
- автоматическое создание `WorkPlace` на основании `EquipmentUnit`;
- подбор ресурсов для нормативных наборов модуля 02 и фактическая доступность
  ресурсов.

## 3. Объявляемые типы объектов

### 3.1 Самостоятельные типы с собственными представлениями

Эти типы могут иметь стандартные наборы `List`, `Lookup` и `Details`, если
конкретное представление используется в UI или как источник значения ссылки.

| Тип объекта | Русское название | `List` | `Lookup` | `Details` | Комментарий |
|---|---|---:|---:|---:|---|
| `WorkPlace` | Рабочее место | да | да | да | Содержит ссылку на внешний `EquipmentUnit`, если она задана. |
| `Personnel` | Сотрудник | да | да | да | Содержит вычисляемые `FullName`, `BasicQualification` и `CurrentLocation`. |
| `WorkPlaceGroup` | Группа рабочих мест | да | да | да | Иерархия только внутри `WorkPlaceGroup`. |
| `PersonnelGroup` | Группа сотрудников | да | да | да | Иерархия только внутри `PersonnelGroup`. |
| `Profession` | Профессия | да | да | да | Справочник профессий. |
| `PaymentGroup` | Группа оплаты | да | да | да | Справочник групп оплаты. |
| `PersonnelAllowanceType` | Вид допуска сотрудника | да | да | да | Срок действия задается в единицах `ValidityPeriodUnit`. |
| `AbsenceReason` | Причина отсутствия | да | да | да | Справочник причин отсутствия. |
| `Tooling` | Технологическая оснастка | да | да | да | Отдельный пользовательский тип общего хранения `ToolBase`. |
| `Gage` | Контрольно-измерительный инструмент | да | да | да | Отдельный пользовательский тип общего хранения `ToolBase`. |
| `Holiday` | Праздничная дата | да | да | да | Запись производственного календаря. |
| `DayType` | Тип рабочего дня | да | да | да | Справочник типов рабочих дней. |
| `WorkSchedule` | График работы | да | да | да | Правило выбора `DayType` по дате. |
| `ShiftRotationModel` | Модель чередования смен | да | да | да | Применяется к сотруднику или группе сотрудников. |

`ToolBase` является абстрактным `reference-only` типом. Он не имеет собственной
карточки или пользовательского списка. Его lookup возвращает экземпляры
`Tooling` и `Gage` с указанием конкретного типа и идентификатора. Права и
переход к карточке проверяются для конкретного типа.

### 3.2 Типы, доступные через коллекции, зависимые объекты или контекст владельца

Следующие типы являются самостоятельными типами доменной модели, но в v1
публикуются в контексте коллекций, одиночных зависимых объектов владельцев или
через контекст внешнего владельца. Отдельные пользовательские списки и
универсальные lookup для них не создаются.

| Тип объекта | Русское название | Владелец или контекст |
|---|---|---|
| `WorkPlaceInWorkPlaceGroup` | Членство рабочего места в группе | `WorkPlaceGroup.Members` |
| `PersonnelInPersonnelGroup` | Членство сотрудника в группе | `PersonnelGroup.Members` |
| `WorkPlaceLocation` | Место установки рабочего места | `WorkPlace.Locations` |
| `PersonnelLocation` | Место работы сотрудника | `Personnel.Locations` |
| `PersonnelQualification` | Квалификация сотрудника | `Personnel.Qualifications` |
| `PersonnelAllowance` | Допуск сотрудника | `Personnel.Allowances` |
| `PersonnelAbsence` | Отсутствие сотрудника | `Personnel.Absences` |
| `DayTypeDescription` | Интервал типа рабочего дня | `DayType.Descriptions` |
| `WorkScheduleDescription` | Описание графика работы | `WorkSchedule.Descriptions` |
| `ShiftRotationDescription` | Описание модели чередования смен | `ShiftRotationModel.Descriptions` |
| `OrganizationalUnitWorkSchedule` | Назначение графика организационной единице | внешняя навигация `OrganizationalUnit.WorkSchedules` |
| `WorkPlaceGroupWorkSchedule` | Назначение графика группе рабочих мест | `WorkPlaceGroup.WorkSchedules` |
| `WorkPlaceWorkSchedule` | Назначение графика рабочему месту | `WorkPlace.WorkSchedules` |
| `PersonnelGroupWorkSchedule` | Назначение графика группе сотрудников | `PersonnelGroup.WorkSchedules` |
| `PersonnelWorkSchedule` | Назначение графика сотруднику | `Personnel.WorkSchedules` |
| `ProductionUnitOperationParameters` | Параметры операций производственной единицы | одиночная внешняя навигация `ProductionUnit.OperationParameters`, `0..1` |
| `WorkPlaceOperationParameters` | Параметры операций рабочего места | одиночный зависимый объект `WorkPlace.OperationParameters`, `0..1` |
| `PersonnelOperationParameters` | Параметры операций сотрудника | одиночный зависимый объект `Personnel.OperationParameters`, `0..1` |
| `MaterialResponsiblePerson` | Материально ответственное лицо | контексты `ProductionUnit` и `Personnel`; отдельная доменная коллекция не объявляется |

`ResourceBase`, `ResourceGroupBase`, `ResourceInResourceGroupBase`,
`ResourceLocationBase`, `ResourceWorkSchedule` и `OperationParametersBase` не
публикуются как самостоятельные runtime-типы. Они задают общие логические
контракты своих наследников. `ToolBase` дополнительно публикуется только как
`reference-only` источник полиморфного lookup.

### 3.3 Внешний reference-only тип `OrganizationalUnit`

`OrganizationalUnit` является внешним абстрактным типом модуля
`03_plant_structure`. Модуль 04 не объявляет его и не создает его записи, но
использует его в контракте `OrganizationalUnitWorkSchedule` и в обратной
навигации из карточки организационной единицы.

| Сценарий | Runtime-поведение |
|---|---|
| Создание | В модуле 04 запрещено; создаются `ProductionUnit` или `Subcontractor` по контракту модуля 03. |
| Lookup | Принимаются конкретные типы, разрешенные контрактом модуля 03; общий источник выбора не превращает `OrganizationalUnit` в объект-владелец модуля 04. |
| Чтение и представление | Разрешаются и отображаются правилами конкретного типа внешнего объекта. |
| Права | Проверяются правами модуля-владельца внешнего объекта. |

`EquipmentUnit`, `Nomenclature` и `NomenclatureVariant` также являются
внешними объектами, но не являются абстрактными типами, объявляемыми runtime
модуля 04. Они используются только как ссылочные цели.

### 3.4 Runtime-контракты логических базовых типов

Логическое наследование действует и в доменной модели, и в runtime-контрактах:
наследник получает общие поля, ссылки, коллекции и правила базового типа. В
описании наследника не повторяются унаследованные свойства; фиксируются только
его собственные отличия и контекст владельца.

| Базовый тип | Runtime-контракт | Публикация |
|---|---|---|
| `ResourceBase` | Общие свойства ресурсов и единые правила доступности ресурсов. | Только через самостоятельные типы `WorkPlace`, `Personnel`, `Tooling` и `Gage`; общего lookup `Resource` нет. |
| `ResourceGroupBase` | Общие свойства групп и ссылка `Parent`, раскрываемая в конкретном наследнике. | Только через `WorkPlaceGroup` и `PersonnelGroup`; `ResourceGroupBase` не является узлом дерева или lookup-типом. |
| `ResourceInResourceGroupBase` | Общий контракт элемента членства группы. | Только через коллекции `Members` соответствующей группы. |
| `ResourceLocationBase` | Общие свойства периодического местоположения, включая `ProductionUnit`, `Previous` и период действия. | Только через коллекции `WorkPlace.Locations` и `Personnel.Locations`. |
| `ResourceWorkSchedule` | Общие свойства назначения графика, `AssignmentType`, период и временная цепочка `Previous`. | Только через пять самостоятельных типов назначений; общего lookup назначений нет. |
| `OperationParametersBase` | Общие параметры настройки операций. | Только через одиночный объект параметров конкретного владельца. |
| `ToolBase` | Общий полиморфный ссылочный контракт с дискриминатором типа. | `reference-only` lookup, объединяющий только `Tooling` и `Gage`; собственной карточки нет. |

Runtime не выполняет стандартные операции над логическим базовым типом как над
отдельной записью. Стандартные операции и права применяются к конкретному
самостоятельному типу или к коллекции его владельца.

## 4. Tenant и область данных

Все объекты модуля 04 принадлежат текущему `Tenant`.

Object Runtime назначает и проверяет `TenantId` по общим правилам:

- при создании самостоятельного объекта значение берется из контекста запроса;
- при создании зависимого объекта значение наследуется от владельца;
- при создании строки коллекции или одиночного зависимого объекта значение
  наследуется от владельца и не редактируется отдельно;
- при выборе ссылки разрешаются только объекты доступного tenant-контекста;
- `TenantId` не редактируется в пользовательской карточке.

`Site` не является объектом и полем владения модуля 04. Производственный
контекст ресурса задается внешней ссылкой на `ProductionUnit`; отдельный
`SiteId` в runtime-контракты модуля не добавляется.

## 5. Правила представления объектов

`Presentation` не является хранимым полем. Object Runtime вычисляет его по
правилу конкретного типа и использует то же представление в ссылках и списках.
Архивные и удаленные объекты не становятся доступными для нового выбора только
из-за наличия сохраненной ссылки на них.

| Тип объекта | Правило представления |
|---|---|
| `WorkPlace` | `{Code} - {Name}` |
| `Personnel` | `{Number} - {FullName}` |
| `WorkPlaceGroup` | `{Code} - {Name}` |
| `PersonnelGroup` | `{Code} - {Name}` |
| `Profession` | `{Code} - {Name}` |
| `PaymentGroup` | `{Code} - {Name}` |
| `PersonnelAllowanceType` | `{Code} - {Name}` |
| `AbsenceReason` | `{Code} - {Name}` |
| `Tooling` | `{Code} - {Name}` |
| `Gage` | `{Code} - {Name}` |
| `Holiday` | `{Name} - {Date}` |
| `DayType` | `{Code} - {Name}` |
| `WorkSchedule` | `{Code} - {Name}` |
| `ShiftRotationModel` | `{Code} - {Name}` |

Для типов, доступных только через коллекции или контекст владельца,
самостоятельный lookup-представитель не создается. В карточке или контексте
владельца отображаются поля элемента и представления его ссылок. Пустые
необязательные части составного представления не образуют лишние разделители.

## 6. Наборы данных Object Runtime

Для публикуемых типов и зависимых объектов baseline должен объявлять следующие
наборы данных:

| Набор данных | Назначение | Применяется к |
|---|---|---|
| `List` | Основной список типа объекта. | Самостоятельные типы из раздела 3.1, кроме `ToolBase`. |
| `Lookup` | Выбор объекта в ссылочном поле. | Самостоятельные типы, `ToolBase` как объединенный lookup, а также внешние типы по их контрактам. |
| `Details` | Чтение объекта для карточки или контекстного зависимого объекта. | Самостоятельные типы и одиночные зависимые объекты. |
| `CollectionRows` | Чтение и сохранение строк коллекции владельца. | Коллекции из раздела 10. |
| `TreeList` | Иерархический список с областями корней, дочерних узлов и поддерева. | `WorkPlaceGroup` и `PersonnelGroup` по ссылке `Parent`. |

Логические базовые типы не получают собственных наборов данных. `ToolBase`
получает только reference-only lookup, поскольку на него существует реальная
полиморфная ссылка из модуля 02. Одиночные параметры операций и
`MaterialResponsiblePerson` не получают самостоятельных списков.

Поля системных перечислений модуля используют стандартный selector Object
Runtime, а не отдельный объектный lookup. Состав и значения этих перечислений
определены в `02_domain_model.md`; runtime не дублирует их отдельными типами.
Для `ValueSet` применяется тот же общий механизм только в тех случаях, когда
такой настраиваемый набор объявлен доменной моделью.

## 7. Иерархии

В модуле 04 есть две самостоятельные иерархии групп ресурсов:

| Тип объекта | Родительская ссылка | Корень | Runtime-поведение |
|---|---|---|---|
| `WorkPlaceGroup` | `Parent -> WorkPlaceGroup` | `Parent = null` | Иерархический список групп рабочих мест; корни и поддеревья выбираются отдельно. |
| `PersonnelGroup` | `Parent -> PersonnelGroup` | `Parent = null` | Иерархический список групп сотрудников; корни и поддеревья выбираются отдельно. |

Runtime-метаданные иерархии не являются дополнительными предметными полями
модуля:

| Runtime-свойство | Runtime-смысл | Редактирование |
|---|---|---|
| `Parent` | Предметная ссылка на родительский узел той же иерархии или `null` для корня. | Разрешено с проверками типа, существования родителя и циклов. |
| `Root` | Ссылка на корневой узел текущего дерева. В платформенном descriptor задается как `RootFieldCode`; физическое имя поля не является частью модели 04. | Только чтение; вычисляется Object Runtime. |
| `Level` | Уровень узла относительно корня. В descriptor задается как `LevelFieldCode`. | Только чтение; вычисляется Object Runtime. |
| `Path` | Материализованный путь узла в иерархии. В descriptor задается как `PathFieldCode`. | Только чтение; вычисляется Object Runtime. |
| `HasChildren` | Наличие активных дочерних узлов, возвращаемое как runtime-проекция или computed member. | Только чтение; вычисляется Object Runtime. |

Для обеих иерархий Object Runtime должен:

- разрешать в `Parent` только объект того же самостоятельного типа;
- исключать из выбора сам объект и его потомков;
- запрещать циклы;
- объявлять в descriptor роли `Parent`, `Root`, `Level`, `Path` и `HasChildren`;
- поддерживать `TreeList` с `ParentNodeId = null` для корней и
  `ParentNodeId = {parentId}` для непосредственных потомков;
- поддерживать области запроса `HierarchyScope.Roots`, `Children`,
  `Descendants` и `Ancestors`;
- пересчитывать `Root`, `Level`, `Path` и `HasChildren` по общей hierarchy
  capability платформы.

При создании корневого узла runtime вычисляет корень и уровень, при создании
дочернего узла использует значения родителя. При переносе узла пересчитывается
его поддерево в одной границе изменения. Пользователь не передает `Root`,
`Level`, `Path` и `HasChildren` в запросе изменения. Удаление или прямое
архивирование узла с активными непосредственными потомками отклоняется общей
политикой иерархии v1.

`ResourceGroupBase` не является корнем дерева, общим родителем записей или
типом lookup. Иерархии рабочих мест и сотрудников не объединяются.

`DayTypeDescription.Parent` связывает рабочий интервал с его перерывом и не
является общей иерархией объектов. `Previous` в местоположениях и назначениях
графиков является временной цепочкой, а не иерархией.

## 8. Объекты и runtime-контракты

### 8.1 Ресурсы и группы

`WorkPlace`, `Personnel`, `WorkPlaceGroup`, `PersonnelGroup`, `Tooling` и
`Gage` являются самостоятельными объектными типами. Общие поля `CommonObject`
и `CommonCatalogObject` не дублируются в runtime-описании каждого типа.

`WorkPlace` имеет необязательную внешнюю ссылку `EquipmentUnit`. Object Runtime
может открыть внешний объект и разрешить его представление, но модуль 04 не
создает `WorkPlace` автоматически и не управляет жизненным циклом оборудования.

`WorkPlaceGroup.Parent` ссылается только на `WorkPlaceGroup`, а
`PersonnelGroup.Parent` - только на `PersonnelGroup`. `ResourceGroupBase` не
является типом этой ссылки. Вложенные элементы `Members` редактируются в
контексте соответствующей группы.

`Tooling` и `Gage` имеют отдельные пользовательские списки и карточки. В
контрактах, где допустимы оба типа, используется объединенный lookup
`ToolBase`. В результате lookup обязательно возвращаются:

- конкретный тип: `Tooling` или `Gage`;
- идентификатор объекта;
- отображаемое представление конкретного объекта;
- доступность перехода к конкретной карточке с учетом прав.

### 8.2 Местоположения ресурсов

`WorkPlaceLocation` и `PersonnelLocation` публикуются через коллекции
соответствующего ресурса. Общий `ResourceLocationBase` не является владельцем
данных и не имеет общего lookup.

`Previous` разрешается только для последовательности основных местоположений
того же конкретного типа и того же ресурса. Для неосновных местоположений поле
не используется. `CurrentLocation` является вычисляемым свойством и не
публикуется как редактируемая ссылка.

Коллекции местоположений используют периодические поля `ValidFrom` и `ValidTo`.
`ValidTo = null` означает открытую верхнюю границу периода в целевой модели.
Проверки периодов, цепочки `Previous` и корректирующих изменений относятся к
правилам и операциям модуля, а не к стандартному механизму lookup.

### 8.3 Персонал, квалификации и допуски

`Personnel.Qualifications`, `Personnel.Allowances` и `Personnel.Absences`
показываются как вложенные коллекции. Ссылки элемента на `Profession`,
`PaymentGroup`, `PersonnelAllowanceType`, `WorkPlace` и `AbsenceReason`
используют соответствующие lookup текущего tenant и общие правила доступности.

`Personnel.BasicQualification` вычисляется по основной записи
`PersonnelQualification`; отдельная ссылка для редактирования этого свойства
не создается. `WorkPlace.PersonnelAllowances` является обратной навигацией к
допускам сотрудников и не делает `WorkPlace` владельцем `PersonnelAllowance`.

### 8.4 Календари, графики и назначения

`WorkSchedule.Descriptions` и `ShiftRotationModel.Descriptions` являются
вложенными коллекциями описаний. `DayType.Descriptions` является вложенной
коллекцией интервалов типа рабочего дня.

`ShiftRotationModel` имеет ссылку на `WorkSchedule` и применяется только в
`PersonnelGroupWorkSchedule` и `PersonnelWorkSchedule`. Он не назначается
рабочему месту или группе рабочих мест. Дополнительное автоматическое
выведение модели чередования смен из одного только `WorkSchedule` не вводится.

Назначения графиков представлены пятью конкретными типами:

```text
OrganizationalUnitWorkSchedule
WorkPlaceGroupWorkSchedule
WorkPlaceWorkSchedule
PersonnelGroupWorkSchedule
PersonnelWorkSchedule
```

`ResourceWorkSchedule` остается логическим базовым типом без собственной
карточки и общего lookup. Для `ProductionUnit` и `Subcontractor` используется
один тип `OrganizationalUnitWorkSchedule` со ссылкой на внешний
`OrganizationalUnit`.

`AssignmentType` является значением сохраняемой строки назначения:

| Значение | Runtime-смысл |
|---|---|
| `Permanent` | Назначение `WorkSchedule` как постоянного графика. |
| `Temporary` | Назначение другого `WorkSchedule` на ограниченный период. |
| `Change` | Временное исключение с `DayType`, а для персонала и группы сотрудников также с `Shift`; полный `WorkSchedule` не выбирается. |

`Change` не является отдельной runtime-операцией и не переписывает базовое
назначение. Доступ к полям `WorkSchedule`, `DayType` и `Shift` определяется
типом конкретного назначения и значением `AssignmentType`; подробные проверки
описаны в правилах модуля.

### 8.5 Параметры операций

`OperationParametersBase` является логическим базовым типом. В runtime
публикуются только конкретные типы параметров в контексте владельца:

| Владелец | Тип параметров | Runtime-доступ |
|---|---|---|
| `ProductionUnit` | `ProductionUnitOperationParameters` | Внешняя навигация из карточки `ProductionUnit`; запись принадлежит модулю 04. |
| `WorkPlace` | `WorkPlaceOperationParameters` | Одиночный зависимый объект карточки `WorkPlace`, максимум одна запись. |
| `Personnel` | `PersonnelOperationParameters` | Одиночный зависимый объект карточки `Personnel`, максимум одна запись. |

Для каждого владельца допускается не более одной записи параметров. Значения,
унаследованные рабочим местом или сотрудником от `ProductionUnit`, и источник
`ProductionUnitOfParentParameters` являются результатом прикладного разрешения
эффективных параметров. `GetOperationParameters` и изменение параметров не
являются стандартным действием базового runtime-типа; их контракт описывается
в `13_operations.md`.

### 8.6 Материально ответственное лицо

`MaterialResponsiblePerson` является самостоятельным типом назначения
сотрудника материально ответственным лицом для `ProductionUnit`. Он доступен
из контекста производственной единицы и из обратного контекста `Personnel`.
Ни один из этих контекстов не становится владельцем объекта в доменной
модели. Самостоятельную универсальную модель ресурса этот тип не создает и
владение объектами `ProductionUnit` или `Personnel` не меняет.

| Элемент | Решение |
|---|---|
| Тип объекта | `MaterialResponsiblePerson` |
| Доступ | Контексты `ProductionUnit` и `Personnel`; отдельный универсальный список и lookup не создаются. |
| Коллекция доменной модели | Не объявляется. Контекстный доступ не изменяет владельца данных. |
| Представление | Контекстные связанные списки в карточках `ProductionUnit` и `Personnel`; отдельное универсальное представление назначения не вводится. |

## 9. Ссылочные поля и источники выбора

Общие правила всех lookup:

- применяется tenant и область данных владельца;
- архивные объекты скрываются при новом выборе по умолчанию;
- уже сохраненная ссылка на архивный объект продолжает отображаться;
- `WorkflowStateCode` сам по себе не становится условием lookup;
- модуль-потребитель не читает таблицы модуля 04 напрямую, а использует
  опубликованный lookup или проверку совместимости.

| Поле или контекст | Источник значений | Ограничение выбора |
|---|---|---|
| `WorkPlace.EquipmentUnit` | Внешний владелец оборудования: `EquipmentUnit` | Внешний объект доступного tenant-контекста; автоматическое создание `WorkPlace` не выполняется. |
| `WorkPlaceGroup.Parent` | `WorkPlaceGroup` | Только группа того же типа; нельзя выбрать саму группу или значение, создающее цикл. |
| `PersonnelGroup.Parent` | `PersonnelGroup` | Только группа того же типа; нельзя выбрать саму группу или значение, создающее цикл. |
| `WorkPlaceInWorkPlaceGroup.WorkPlaceGroup` | Контекст `WorkPlaceGroup.Members` | Заполняется владельцем коллекции. |
| `WorkPlaceInWorkPlaceGroup.WorkPlace` | `WorkPlace` | Выбирается рабочее место доступного tenant-контекста; дубликат членства запрещается правилами. |
| `PersonnelInPersonnelGroup.PersonnelGroup` | Контекст `PersonnelGroup.Members` | Заполняется владельцем коллекции. |
| `PersonnelInPersonnelGroup.Personnel` | `Personnel` | Выбирается сотрудник доступного tenant-контекста; дубликат членства запрещается правилами. |
| `WorkPlaceLocation.WorkPlace` | Контекст `WorkPlace.Locations` | Заполняется владельцем коллекции и не изменяется на другой тип ресурса. |
| `PersonnelLocation.Personnel` | Контекст `Personnel.Locations` | Заполняется владельцем коллекции и не изменяется на другой тип ресурса. |
| `ResourceLocationBase.ProductionUnit` | `03_plant_structure.ProductionUnit` | Производственная единица доступного tenant-контекста. |
| `ResourceLocationBase.Previous` | Тот же конкретный тип местоположения | Только предыдущее основное местоположение того же ресурса. |
| `PersonnelQualification.Personnel` | Контекст `Personnel.Qualifications` | Заполняется владельцем коллекции. |
| `PersonnelQualification.Profession` | `Profession` | Справочник профессий текущего tenant-контекста. |
| `PersonnelQualification.PaymentGroup` | `PaymentGroup` | Необязательная группа оплаты текущего tenant-контекста. |
| `PersonnelAllowance.Personnel` | Контекст `Personnel.Allowances` | Заполняется владельцем коллекции. |
| `PersonnelAllowance.WorkPlace` | `WorkPlace` | Необязательное рабочее место доступного tenant-контекста. |
| `PersonnelAllowance.AllowanceType` | `PersonnelAllowanceType` | Вид допуска текущего tenant-контекста. |
| `PersonnelAbsence.Personnel` | Контекст `Personnel.Absences` | Заполняется владельцем коллекции. |
| `PersonnelAbsence.AbsenceReason` | `AbsenceReason` | Причина отсутствия текущего tenant-контекста. |
| `ToolBase.ToolNomenclature` | `GMD.Nomenclature` | Номенклатура доступного tenant-контекста. |
| `ToolBase.ToolNomenclatureVariant` | `GMD.NomenclatureVariant` | Исполнение должно относиться к выбранной номенклатуре. |
| `DayTypeDescription.DayType` | Контекст `DayType.Descriptions` | Заполняется владельцем коллекции. |
| `DayTypeDescription.Parent` | `DayTypeDescription` | Интервал того же типа рабочего дня; не используется как наследование типов. |
| `WorkScheduleDescription.WorkSchedule` | Контекст `WorkSchedule.Descriptions` | Заполняется владельцем коллекции. |
| `WorkScheduleDescription.DayType` | `DayType` | Необязательный тип рабочего дня текущего tenant-контекста. |
| `ShiftRotationModel.WorkSchedule` | `WorkSchedule` | График текущего tenant-контекста; модель должна соответствовать его типу. |
| `ShiftRotationDescription.ShiftRotationModel` | Контекст `ShiftRotationModel.Descriptions` | Заполняется владельцем коллекции. |
| `ResourceWorkSchedule.WorkSchedule` | `WorkSchedule` | Для `Permanent` и `Temporary`; для `Change` не заполняется. |
| `ResourceWorkSchedule.DayType` | `DayType` | Для `Change`; для других значений используется по правилам типа назначения. |
| `ResourceWorkSchedule.Previous` | Тот же конкретный тип назначения | Предыдущее постоянное назначение того же владельца. |
| `OrganizationalUnitWorkSchedule.OrganizationalUnit` | `03_plant_structure.OrganizationalUnit` | Abstract/reference-only lookup `OrganizationalUnit`; допускаются `ProductionUnit` и `Subcontractor`. |
| `PersonnelGroupWorkSchedule.ShiftRotationModel` | `ShiftRotationModel` | Модель для группы сотрудников. |
| `PersonnelWorkSchedule.ShiftRotationModel` | `ShiftRotationModel` | Модель для сотрудника. |
| `ProductionUnitOperationParameters.ProductionUnit` | Контекст внешней навигации `ProductionUnit.OperationParameters` | Заполняется владельцем карточки `ProductionUnit`; запись принадлежит модулю 04. |
| `WorkPlaceOperationParameters.WorkPlace` | Контекст `WorkPlace.OperationParameters` | Заполняется владельцем карточки `WorkPlace`; допускается одна запись. |
| `PersonnelOperationParameters.Personnel` | Контекст `Personnel.OperationParameters` | Заполняется владельцем карточки `Personnel`; допускается одна запись. |
| `MaterialResponsiblePerson.ProductionUnit` | Контекст `ProductionUnit` | Производственная единица текущего tenant-контекста. |
| `MaterialResponsiblePerson.Personnel` | `Personnel` | Сотрудник текущего tenant-контекста. |

### 9.1 Полиморфный lookup `ToolBase`

Для внешней ссылки из модуля 02 используется квалифицированный тип
`ResourceManagement:ToolBase`. Lookup обязан объединять только два конкретных
типа:

```text
Tooling
Gage
```

Сохраненная ссылка содержит идентификатор записи общего физического хранения;
конкретный тип определяется runtime по дискриминатору. Пользовательский lookup
не показывает `ToolBase` как третий вид объекта.

### 9.2 Lookup рабочих мест и персонала для модуля 02

Модуль 04 предоставляет отдельные lookup и проверки для конкретных типов:

```text
GetWorkPlaceLookup(WorkPlaceGroupId?, SearchText?, IncludeArchived = false)
ValidateWorkPlaceInGroup(WorkPlaceId, WorkPlaceGroupId)
GetPersonnelLookup(PersonnelGroupId?, SearchText?, IncludeArchived = false)
ValidatePersonnelInGroup(PersonnelId, PersonnelGroupId)
CheckPersonnelQualification(PersonnelId, ProfessionId?, Grade?)
CheckPersonnelAllowance(PersonnelId, WorkPlaceId?, Date)
```

При переданной группе lookup ограничивает кандидатов членами этой группы.
Если одновременно переданы группа и конкретный ресурс, конкретный ресурс
уточняет выбор и должен входить в группу. Количество ресурса и альтернативные
наборы нормативных ресурсов принадлежат модели модуля 02; модуль 04 не создает
их runtime-объекты.

## 10. Коллекции и зависимые объекты Object Runtime

Режимы коллекций:

| Режим | Смысл |
|---|---|
| `Состав владельца` | Элементы редактируются в карточке владельца; отдельные permission codes для строк в v1 не вводятся. |
| `Отдельные операции` | Элементы показываются в карточке владельца, но имеют собственные операции своего типа. |
| `Навигация` | Показывает связанные объекты другого владельца; создание, изменение и удаление выполняет владелец данных. |

| Владелец | Коллекция | Тип элемента | Режим | Комментарий |
|---|---|---|---|---|
| `WorkPlace` | `Locations` | `WorkPlaceLocation` | `Состав владельца` | Вложенная история мест установки; правила периодов и `Previous` выполняются модулем 04. |
| `WorkPlace` | `WorkSchedules` | `WorkPlaceWorkSchedule` | `Состав владельца` | Назначения графиков рабочему месту. `ShiftRotationModel` не используется. |
| `WorkPlace` | `PersonnelAllowances` | `PersonnelAllowance` | `Навигация` | Обратный список допусков сотрудников, где указано это рабочее место. |
| `Personnel` | `Locations` | `PersonnelLocation` | `Состав владельца` | Вложенная история мест работы. |
| `Personnel` | `Qualifications` | `PersonnelQualification` | `Состав владельца` | Квалификации сотрудника. |
| `Personnel` | `Allowances` | `PersonnelAllowance` | `Состав владельца` | Допуски сотрудника. |
| `Personnel` | `Absences` | `PersonnelAbsence` | `Состав владельца` | Периоды отсутствия сотрудника. |
| `Personnel` | `WorkSchedules` | `PersonnelWorkSchedule` | `Состав владельца` | Назначения графиков сотруднику, включая модель сменности. |
| `WorkPlaceGroup` | `Members` | `WorkPlaceInWorkPlaceGroup` | `Состав владельца` | Членство рабочих мест в группе. |
| `WorkPlaceGroup` | `WorkSchedules` | `WorkPlaceGroupWorkSchedule` | `Состав владельца` | Назначения графиков группе рабочих мест. |
| `PersonnelGroup` | `Members` | `PersonnelInPersonnelGroup` | `Состав владельца` | Членство сотрудников в группе. |
| `PersonnelGroup` | `WorkSchedules` | `PersonnelGroupWorkSchedule` | `Состав владельца` | Назначения графиков группе сотрудников, включая модель сменности. |
| `DayType` | `Descriptions` | `DayTypeDescription` | `Состав владельца` | Интервалы рабочих смен и перерывов. |
| `WorkSchedule` | `Descriptions` | `WorkScheduleDescription` | `Состав владельца` | Правила выбора типа рабочего дня по дате. |
| `ShiftRotationModel` | `Descriptions` | `ShiftRotationDescription` | `Состав владельца` | Правила выбора смены по дате. |
| `OrganizationalUnit` | `WorkSchedules` | `OrganizationalUnitWorkSchedule` | `Навигация` | Обратная навигация из ПР03; запись принадлежит модулю 04. |

Контекстные связанные списки, не являющиеся коллекциями владельца:

| Контекст | Тип данных | Режим | Фильтр и назначение |
|---|---|---|---|
| `ProductionUnit` | `MaterialResponsiblePerson` | `Навигация` | Назначения, у которых `ProductionUnit` равен текущей производственной единице. |
| `Personnel` | `MaterialResponsiblePerson` | `Навигация` | Назначения, у которых `Personnel` равен текущему сотруднику. |

В отличие от коллекций `0..*`, параметры операций являются одиночными
зависимыми объектами с кардинальностью `0..1`:

| Владелец или внешний контекст | Свойство | Тип объекта | Режим | Комментарий |
|---|---|---|---|---|
| `WorkPlace` | `OperationParameters` | `WorkPlaceOperationParameters` | `Состав владельца`, `0..1` | Не более одной записи параметров рабочего места. |
| `Personnel` | `OperationParameters` | `PersonnelOperationParameters` | `Состав владельца`, `0..1` | Не более одной записи параметров сотрудника. |
| `ProductionUnit` | `OperationParameters` | `ProductionUnitOperationParameters` | `Навигация`, `0..1` | Одиночная межмодульная навигация; запись принадлежит модулю 04. |

Элементы коллекций в v1 не получают отдельных permission codes только из-за
того, что они отображаются внутри карточки. Доступ к ним определяется правом
родительского объекта и областью доступной строки. Если в будущем элемент будет
опубликован как отдельный верхнеуровневый объект, для него потребуется отдельное
решение о наборе представлений и прав.

## 11. Обработчики Object Runtime

Модуль использует стандартные точки расширения Object Runtime. Сами проверки и
сообщения описываются в `05_rules.md`; здесь фиксируется только runtime-роль.

| Объект или контекст | Точка | Поведение |
|---|---|---|
| Все самостоятельные типы | `BeforeCreate` / `BeforeSave` | Проверить tenant, владельца, ссылочную доступность и обязательные поля по доменной модели. |
| `WorkPlaceGroup`, `PersonnelGroup` | `BeforeSave` | Проверить самоссылку `Parent`, однотипность и отсутствие циклов. |
| `WorkPlaceInWorkPlaceGroup`, `PersonnelInPersonnelGroup` | `BeforeSave` | Проверить владельца коллекции, тип ресурса и отсутствие дублирующего членства. |
| `WorkPlaceLocation`, `PersonnelLocation` | `BeforeSave` | Проверить период, `IsBasic`, `IsPermanent`, владельца и корректность `Previous`. |
| `PersonnelQualification` | `BeforeSave` | Проверить сотрудника, профессию, разряд и согласованность основной квалификации. |
| `PersonnelAllowance` | `BeforeSave` | Проверить сотрудника, вид допуска, рабочее место и период действия. |
| `PersonnelAbsence` | `BeforeSave` | Проверить сотрудника, причину отсутствия и период. |
| `DayTypeDescription` | `BeforeSave` | Проверить интервалы смены, условную обязательность `BreakNumber`, перерывы и связь `Parent`. |
| `WorkScheduleDescription` | `BeforeSave` | Проверить обязательные поля в зависимости от `WorkSchedule.Type`. |
| `ShiftRotationDescription` | `BeforeSave` | Проверить обязательные поля периода и смену. |
| Назначения графиков | `BeforeSave` | Проверить `AssignmentType`, период, владельца, `Previous` и совместимость `WorkSchedule`, `DayType`, `Shift`. |
| Параметры операций | `BeforeSave` | Проверить единственность записи для владельца и доступность собственных значений. |
| `ToolBase` lookup | `Query` / `ResolveReference` | Вернуть только `Tooling` и `Gage`, определить самостоятельный тип и открыть карточку конкретного типа. |
| `MaterialResponsiblePerson` | `BeforeCreate` / `BeforeSave` | Проверить ссылки на `ProductionUnit` и `Personnel`, а также отсутствие дублирующего действующего назначения для одной пары. |

Стандартное архивирование и удаление выполняются с учетом общих правил Common.
Они не заменяют предметные проверки зависимостей и не создают workflow для
объектов, у которых нет принятого жизненного цикла.

## 12. Стандартные операции и прикладные действия

Стандартные действия чтения, создания, изменения, архивирования, восстановления
и удаления выполняются через Object Runtime с учетом общих правил Common,
tenant-контекста и прав конкретного типа.

### 12.1 Жизненный цикл и workflow

В v1 модуль 04 не объявляет отдельный предметный workflow для справочников,
ресурсов, групп, местоположений, назначений графиков, параметров операций и
материально ответственных лиц. Их архивирование, восстановление и удаление
выполняются стандартными механизмами Common/Object Runtime с учетом
предметных ограничений модуля.

`AssignmentType`, `IsBasic`, `IsPermanent` и `Change` являются значениями и
признаками предметных объектов. Они не являются состояниями workflow и не
порождают переходы. При этом ограничения их сочетаний проверяются обработчиками
сохранения, как указано в разделе 11.

`GetOperationParameters`, разрешение эффективных параметров и lookup-проверки
для модуля 02 являются предметными функциями модуля. Они не становятся
универсальными действиями абстрактного базового типа и должны быть описаны в
`13_operations.md`.

Движение, выдача, возврат, ремонт, списание, фактическая эксплуатация и
автоматический подбор ресурсов не являются операциями модуля 04.

## 13. Baseline и подключенные возможности платформы

После появления реализации baseline модуля должен объявить:

- object type metadata для самостоятельных типов модуля;
- abstract/reference-only descriptor для `ToolBase`;
- наборы `List`, `Lookup` и `Details` для самостоятельных справочных объектов;
- nested collections из раздела 10 с указанными режимами;
- одиночные зависимые объекты из раздела 10;
- правила представления из раздела 5;
- наборы данных `List`, `Lookup`, `Details`, `CollectionRows`, `TreeList`;
- hierarchy capability для `WorkPlaceGroup` и `PersonnelGroup`;
- источники lookup и межполевая фильтрация для ссылок модуля 02;
- стандартные действия Common только для явно разрешенных типов;
- tenant-aware и permission-aware обработку всех наборов данных.

Базовые платформенные контракты, на которые опирается baseline модуля:

- [Object Runtime](../../03_platform/03_object_runtime/04_runtime.md) - типы
  объектов, наборы данных, ссылки, коллекции, обработчики и иерархии;
- [Tenant и безопасность](../../03_platform/01_tenant_and_security/04_runtime.md)
  - область tenant и проверки доступа;
- [Frontend Runtime](../../03_platform/12_frontend_platform/04_runtime.md) -
  публикация представлений и tree-метаданных;
- [иерархическая capability](../../../plans/026_object_runtime_hierarchy_support_backlog.md)
  - роли `Parent`, `Root`, `Level`, `Path`, `HasChildren` и области запроса;
- [abstract/reference-only capability](../../../plans/020_object_runtime_remediation_backlog.md)
  - полиморфный lookup `ToolBase`.

`OrganizationalUnit`, `ProductionUnit`, `Subcontractor`, `EquipmentUnit`,
`Nomenclature` и `NomenclatureVariant` не становятся типами-владельцами
модуля 04. Для них используются внешние runtime-контракты и обратняя
навигация, где она указана в разделе коллекций.

В v1 отдельные baseline-действия для движения инструмента, автоматического
создания рабочего места, альтернативных ресурсных наборов и фактического
подбора ресурсов не объявляются.

## 14. Граница с интерфейсом

Списки, карточки, вкладки, фильтры и пункты меню описываются в
`06_ui_views.md` и `15_navigation_menu.md`. Настоящий документ фиксирует
только runtime-возможности, от которых зависит интерфейс:

- самостоятельные справочные типы имеют собственные списки и карточки;
- вложенные типы отображаются коллекциями соответствующих владельцев;
- обратные межмодульные связи отображаются только в режиме `Навигация`;
- одиночные зависимые объекты отображаются как один объект в контексте
  владельца, а не как список;
- иерархии групп используют `TreeList` и стандартные области `Roots`,
  `Children`, `Descendants`, `Ancestors`;
- `Tooling` и `Gage` имеют раздельные пользовательские карточки, а `ToolBase`
  используется только как объединенный lookup;
- вычисляемые `CurrentLocation`, `BasicQualification` и эффективные параметры
  не редактируются как обычные ссылочные поля.

## 15. Связанные документы

- [`01_scope.md`](01_scope.md) - граница владения и внешние зависимости.
- [`02_domain_model.md`](02_domain_model.md) - типы, поля, ссылки и коллекции.
- [`05_rules.md`](05_rules.md) - инварианты и проверки.
- [`13_operations.md`](13_operations.md) - прикладные операции lookup и разрешения.
- [`06_ui_views.md`](06_ui_views.md) - списки, карточки и вкладки.
- [`11_permissions.md`](11_permissions.md) - роли и права.
- [`90_traceability_pr04.md`](90_traceability_pr04.md) - источники и расхождения.
- [`00_common/03_object_runtime_model.md`](../00_common/03_object_runtime_model.md) - общие runtime-правила Common.
- [`01_general_master_data/03_object_runtime_model.md`](../01_general_master_data/03_object_runtime_model.md) - runtime-паттерны GMD.
- [`02_product_process_definition/03_object_runtime_model.md`](../02_product_process_definition/03_object_runtime_model.md) - потребление ресурсных ссылок модулем 02.
- [`03_plant_structure/03_object_runtime_model.md`](../03_plant_structure/03_object_runtime_model.md) - внешний `OrganizationalUnit` и обратная навигация.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 10:04 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #59: первые версии прикладных модулей 04 и 06 | [PR #59](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/59) |
| 0.1 | 2026-09-03 10:01 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | первые версии модулей 04, 06. правки связанной документации - шаблон и предложения по изменениям ФТ (определения терминов) | [766441e6](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/766441e68e531dadabe887393fc989e5b6e268e0) |
