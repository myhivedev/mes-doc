---
id: DOC-04-02-90
title: 'Трассировка требований ПР02 - 02 Product & Process Definition'
type: requirement
status: approved
version: '1.2'
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

# Трассировка требований ПР02 - 02 Product & Process Definition

## 1. Назначение документа

Документ фиксирует соответствие между исходными требованиями ПР02, DMP_DATA и принятыми проектными решениями по модулю `02 Product & Process Definition`.

Главная цель этого файла - не дать проектированию вернуться к старой модели `BillStatus / ProcessStatus / Approved` как единственному признаку применимости составов и технологий. Целевая модель модуля строится вокруг ревизий, областей использования, срока действия, условий применимости и workflow утверждения ревизии.

Трассировка не заменяет доменную модель, runtime-модель, workflow, правила или UI. Она является контрольной картой решений, которые должны быть раскрыты в остальных документах модуля.

## 2. Источники

| Источник | Использование |
|---|---|
| `Задача: начать проектирование документации для следующего прикладного модуля DM...` | Основной исходный текст по ПР02. |
| `DMP ПР00 Общие требования.docx` | Источник структуры главного меню модуля. |
| `DMP_DATA` | Источник кандидатов на объекты, поля, статусы и связи. |
| `docs-new/04_domain_modules/00_common` | Common-решения по `Status`, архивированию, удалению, Object Runtime и workflow integration. |
| `docs-new/04_domain_modules/01_general_master_data` | Внешние ссылки на НП, единицы измерения, контрагентов и общую НСИ. Для `UnitOfOperation` используются функции GMD выбора допустимых единиц, единицы по умолчанию, проверки применимости и пересчета в базовую ЕИ. |
| `docs-new/04_domain_modules/03_plant_structure` | Внешние ссылки на производственные единицы, организационные единицы, субподрядчиков, места хранения и складские ячейки; зона хранения учитывается через выбранную ячейку. |
| `src/Platform/DMP.Platform.Configuration/Domain/Schemas/WorkflowArtifactSchemas.cs` | Канонические workflow-артефакты: `WorkflowState`, `WorkflowTransition`, `WorkflowStatePolicy`, `WorkflowRuleBinding`. |
| `src/Platform/DMP.Platform.Workflow/Application/Services/WorkflowRuntimeService.cs` | Текущее поведение workflow runtime: состояния, переходы, guard-проверки, `IsArchiveState`. |

## 3. Базовое проектное решение

Модуль проектируется не как перенос старых таблиц и статусов, а как модель нормативных ревизий составов, технологий и схем кооперации.

Принято:

- единица управления - ревизия `ManufacturingBill`, `ProcessDefinition`, `CooperationScheme`;
- workflow утверждает ревизию и блокирует изменение выпущенной ревизии;
- применимость ревизии определяется хранимыми областями использования, сроком действия и условиями применимости;
- планирование, производство, калькуляция и новые рабочие ссылки фильтруют ревизии по областям использования, а не по буквальному `Status == Approved`;
- для существенных изменений выпущенной ревизии создается новая ревизия;
- строки состава, операции технологии и строки схемы кооперации не получают собственный workflow.

## 4. Ключевое расхождение со старым `Status`

В исходной статусной модели ПР02 / DMP_DATA:

- `ManufacturingBill.BillStatus` хранит статус спецификации;
- `ProcessBase.ProcessStatus` хранит статус технологического описания;
- подбор доступных спецификаций и технологий использует `Approved`;
- APS-выгрузка берет только утвержденные технологии и связанные утвержденные составы.

Целевое решение не переносит этот подход буквально.

| Элемент старой модели | Целевое решение | Причина |
|---|---|---|
| `BillStatus` | Не самостоятельное редактируемое поле применимости. Трассируется в workflow `ProcessDocumentLifecycle` ревизии `ManufacturingBill`. | Статус описывает процесс согласования, но не все области использования ревизии. |
| `ProcessStatus` | Не самостоятельное редактируемое поле применимости. Трассируется в workflow `ProcessDocumentLifecycle` ревизии `ProcessDefinition`. | Технология может быть выпущена для планирования раньше, чем для производства. |
| `ProcessDocumentStatus.New` | `Draft` | Ревизия готовится. |
| `ProcessDocumentStatus.InProgress` | `Draft` | В целевой модели не нужен отдельный статус, если нет дополнительного процессного смысла. |
| `ProcessDocumentStatus.PendingApproval` | `PendingApproval` | Ревизия отправлена на утверждение. |
| `ProcessDocumentStatus.Approved` | `Released` + области использования | Старый `Approved` раскрывается через выпуск ревизии и хранимые области использования. |
| `ProcessDocumentStatus.Archive` | `Archived` с `IsArchiveState = true` | Архивность синхронизируется с Common `IsArchived`. |

## 5. Области использования вместо единственного `Approved`

Для `ManufacturingBill`, `ProcessDefinition`, `CooperationScheme` вводятся хранимые поля ревизии:

| Поле | Заменяет / уточняет | Смысл |
|---|---|---|
| `UseInPlanning` | Часть старого смысла `Approved` | Ревизию можно использовать в разузловании, MRP/APS и планировании потребностей. |
| `UseInProduction` | Часть старого смысла `Approved` | Ревизию можно использовать при создании и исполнении производственных заказов. |
| `UseInCosting` | Часть старого смысла `Approved` | Ревизию можно использовать для калькуляции, нормирования и расчета себестоимости. |
| `AllowNewOperationalSelection` | Ограничение новых ссылок | Ревизию можно выбирать в новых рабочих документах и новых операционных ссылках. |

В UI эти поля показываются как блок `Области использования`, а не как технические системные признаки.

Продуктивные запросы строятся по областям использования:

```text
для планирования:
  UseInPlanning = true
  IsArchived = false
  IsDeleted = false
  ValidFrom / ValidTo покрывают дату расчета
  UseCondition подходит под контекст
```

```text
для производства:
  UseInProduction = true
  IsArchived = false
  IsDeleted = false
  ValidFrom / ValidTo покрывают дату операции
  UseCondition подходит под контекст
```

```text
для выбора в новых рабочих ссылках:
  AllowNewOperationalSelection = true
  IsArchived = false
  IsDeleted = false
```

## 6. Workflow по умолчанию

Целевой workflow для корневых ревизий:

```text
ProcessDocumentLifecycle
```

Состояния:

| StateCode | Назначение | Common / Runtime |
|---|---|---|
| `Draft` | Ревизия готовится. Рабочие поля, строки и области использования редактируются. | `IsInitial = true` |
| `PendingApproval` | Ревизия отправлена на согласование. Рабочие поля и области использования заблокированы. | `WorkflowStatePolicy`: `Object/Self/ReadOnly` |
| `Released` | Ревизия утверждена. Использование определяется областями использования. | `WorkflowStatePolicy`: `Object/Self/ReadOnly` |
| `Archived` | Архивное состояние жизненного цикла. Для состояния с `IsArchiveState = true` платформенный lifecycle синхронизирует стандартный признак Common `IsArchived = true`. | Алгоритмы продуктивного подбора исключают ревизию по `IsArchived`, а не по `WorkflowStateCode`. |

Переходы:

| Команда | Из | В |
|---|---|---|
| `SubmitForApproval` | `Draft` | `PendingApproval` |
| `ReturnToDraft` | `PendingApproval` | `Draft` |
| `Approve` | `PendingApproval` | `Released` |
| `Archive` | `Draft`, `PendingApproval`, `Released` | `Archived` |
| `Restore` | `Archived` | `Draft` |

Новые типы workflow-артефактов не вводятся. Используются существующие канонические артефакты:

- `Workflow`;
- `WorkflowState`;
- `WorkflowCommand`;
- `WorkflowTransition`;
- `WorkflowStatePolicy`;
- `WorkflowRuleBinding`.

## 7. Проверки перехода `Approve`

Переход `Approve` должен иметь `WorkflowRuleBinding` / правила проверки, которые зависят от выбранных областей использования.

| Условие | Проверки |
|---|---|
| `UseInPlanning = true` | Заполнены данные для разузлования и планирования; есть валидный продукт/группа продукта, тип процесса, строки состава или технологические данные, необходимые для планирования. |
| `UseInProduction = true` | Заполнены данные для производственного исполнения: операции, нормы, ресурсы/места выполнения, материальные позиции, технологическая последовательность. |
| `UseInCosting = true` | Заполнены данные для калькуляции: нормы расхода, операции/ресурсы, единицы, параметры расчета. |
| Любая активная область использования | Задана `ValidFrom`; нет конфликтующей активной ревизии с той же областью использования для того же изделия/типа процесса в пересекающемся периоде. |
| Есть связанные составы/технологии/кооперация | Связанные ревизии совместимы по изделию, типу процесса, периоду действия и областям использования. |

## 8. Изменение выпущенной ревизии

Выпущенная ревизия (`Released`) не редактируется напрямую.

Если нужно изменить:

- состав компонентов;
- нормы расхода;
- технологические операции;
- ресурсы;
- схему кооперации;
- области использования;
- сроки действия;

создается новая ревизия.

Пример:

```text
ManufacturingBill Rev.1
  State = Released
  UseInPlanning = true
  UseInProduction = false

ManufacturingBill Rev.2
  State = Draft / PendingApproval
  UseInPlanning = true
  UseInProduction = true
```

Пока Rev.2 готовится, планирование использует Rev.1. После выпуска Rev.2 предыдущая ревизия ограничивается по `ValidTo`, получает `AllowNewOperationalSelection = false` или архивируется по правилам замены ревизий.

## 9. Доменные поля ревизионности

Модуль использует доменную ревизионность через поля `Initial`, `Revision`, `ExternalRevision`. Это не общий платформенный механизм версий.

| Поле | Целевое решение |
|---|---|
| `Initial` | Ссылка на первую ревизию цепочки. Для первой ревизии указывает на саму себя. |
| `Revision` | Внутренний числовой номер ревизии. Используется для сортировки, уникальности и создания следующей ревизии. |
| `ExternalRevision` | Внешнее обозначение ревизии из КД, интеграции или старой системы. Не обязано быть числом и не используется для сортировки. |

Пример:

```text
ManufacturingBill Rev.1
  Initial = self
  Revision = 1
  ExternalRevision = A

ManufacturingBill Rev.2
  Initial = ManufacturingBill Rev.1
  Revision = 2
  ExternalRevision = B
```

Новая ревизия создается операцией `CreateNewRevision`:

```text
Initial = source.Initial
Revision = max(Revision по Initial) + 1
State = Draft
```

При создании новой ревизии копируются шапка, строки, условия применимости, области использования и связи, если они остаются предметно применимыми.

При выпуске новой ревизии старая ревизия не изменяется скрыто. Если новая ревизия пересекается со старой по той же области использования, периоду и контексту применимости, выпуск должен быть запрещен до явного закрытия `ValidTo`, снятия области использования или архивирования старой ревизии.

## 10. Связь технологии и состава

`ProcessDefinition` может ссылаться на `ManufacturingBill`.

Смысл ссылки:

```text
ProcessDefinition.ManufacturingBill = null
```

означает общую технологию. Такая технология не связана заранее с одной ревизией состава; применимая спецификация продукта подбирается отдельно по продукту, исполнению, типу процесса, периоду действия, областям использования и условиям применимости.

```text
ProcessDefinition.ManufacturingBill != null
```

означает технологию с индивидуальной спецификацией ТО. В этом случае ссылка должна указывать на `ManufacturingBill` с `IsForProcessDefinition = true`.

Проверки для индивидуальной спецификации ТО:

- `ProcessDefinition.Product` совпадает с `ManufacturingBill.Product`;
- исполнения и типы процесса совместимы, если заполнены;
- одна ревизия индивидуальной спецификации ТО не используется как `ProcessDefinition.ManufacturingBill` более чем у одной технологии;
- области использования индивидуальной спецификации проверяются при выпуске и продуктивном подборе технологии.

Для общей технологии области использования проверяются не против заранее связанной спецификации, а между объектами, выбранными алгоритмом подбора.

## 11. Трассировка объектов

| Объект / элемент ПР02 | Целевое решение | Статус | Где детализировать |
|---|---|---|---|
| `ManufacturingBillTemplate` | Настроечный объект параметров создания спецификации: шаблон по умолчанию и шаг нумерации позиций. Не является ревизионным документом и не получает `ProcessDocumentLifecycle`. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md`, `13_operations.md` |
| `ProcessTemplate` | Настроечный объект параметров создания технологии: шаблон по умолчанию, шаги нумерации переделов, операций, переходов, ресурсов и длины номеров в ТП из Word ПР02. Не является ревизионным документом и не получает `ProcessDocumentLifecycle`. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md`, `13_operations.md` |
| `LabourType` | Справочник видов работ модуля 02. Используется переделами, операциями и классификаторными технологическими операциями. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| `ManufacturingOperationType` | Справочник видов операций модуля 02. Хранит схему подтверждения по умолчанию и способ учета факта операции. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md` |
| `ProcessingOperation` | Древовидный классификатор технологических операций модуля 02. Используется как ссылка в `ProcessOperationBase.ProcessingOperation` вместо строкового классификатора. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| `WorkEnvironment` | Справочник условий труда модуля 02 для операции. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md` |
| `PaymentGroup` | Не справочник модуля 02. Используется как внешняя ссылка на ресурсный модуль в операции и трудовых позициях. | Внешняя зависимость | `01_scope.md`, `02_domain_model.md`, `03_object_runtime_model.md` |
| `ManufacturingBill` | Корневая ревизия спецификации/состава с workflow, областями использования, сроком действия и условиями применимости. | Входит | `02_domain_model.md`, `04_workflows.md`, `05_rules.md` |
| `ManufacturingBillPosition` | Строка состава без собственного workflow; состояние определяется ревизией `ManufacturingBill`. Хранит собственные `ObtainMethod`, `ProcessType`, `ValidFrom`, `ValidTo`; эти поля не дублируются в материальных позициях технологии. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md` |
| `ProcessBase` / `ProcessDefinition` | `ProcessBase` фиксирует общие поля технологического описания; `ProcessDefinition` является конкретной корневой ревизией с workflow, областями использования и опциональной ссылкой на `ManufacturingBill`. | Входит | `02_domain_model.md`, `04_workflows.md`, `05_rules.md` |
| `ProcessSegmentBase` / `ProcessSegment`, `ProcessOperationBase` / `ProcessOperation`, `ProcessStepBase` / `ProcessTransition` | Базовые типы фиксируют общие поля передела, операции и перехода; конкретные наследники добавляют владельца и коллекции. Структурные элементы технологии не имеют собственного workflow. | Входит | `02_domain_model.md`, `05_rules.md` |
| `ProcessSegmentBase.Number`, `ProcessOperationBase.Number`, `ProcessStepBase.Number` и `NumberInProcess` | `Number` сохраняется как единое имя числового номера базового типа, без переименования в `SegmentNumber` / `OperationNumber` / `TransitionNumber`. `NumberInProcess` сохраняется как отдельная строка `String(10)` и при отсутствии пользовательского значения заполняется из `Number` с длиной из `ProcessTemplate`. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md`, `13_operations.md` |
| `ProcessSegmentBase.IsAutoComplet`, `IsAccountingPoint`, `IsCreateManufacturingLot`, `IsAssignSerialNumbers`, `TimeBuffer`, `YieldFactor`, `ManufacturingLossFactor`, `LossQty`, `FactorRoundingRule` | Поля передела входят в целевую модель. `IsAutoComplet` исправляется как `IsAutoComplete`; `FactorRoundingRule` использует общий `RoundingRule`; `ManufacturingLossFactor` сохраняет имя ПР02 вместо более общего `LossFactor`. | Входит с переименованием опечатки | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md`, `13_operations.md` |
| `ProcessOperationBase.ProductivityFactor`, временные нормы операции | `ProductivityFactor` хранится как `Decimal(24,8)` в целевой доменной модели. `RateForTeardownTime` входит в `ProcessOperationBase` по Word/UI ПР02 как норма заключительного времени, даже если в DATA-таблице `ProcessOperation` это поле не найдено. | Входит с уточнением целевой точности | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| `ProcessStepBase.SpecialInstructions`, `ProcessStepBase.Name` | У перехода сохраняются `Number`, `NumberInProcess`, `Description`, `SpecialInstructions`. Отдельное поле `Name` не вводится, потому что оно не найдено в Word/DATA для `ProcessStepBase`. | Входит с уточнением реквизитного состава | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md` |
| `ProcessSegmentBase.ShopFloor`, `ProcessOperationBase.ShopFloor`, `ProcessOperationBase.ShopFloorArea`, `IsSubcontracted` | Исторические имена `ShopFloor` и `ShopFloorArea` не переносятся как целевые имена полей. `ShopFloor` в базовых типах передела и операции становится `ResponsibleOrgUnit` типа `OrganizationalUnit`; `ShopFloorArea` в базовом типе операции становится `ExecutionOrgUnit` типа `OrganizationalUnit`; `IsSubcontracted` сохраняется в базовых типах передела и операции. Для операции это целевое расширение относительно найденного Word/DATA: оно нужно для сценария субподрядной отдельной операции внутри собственного передела. У `ProcessStepBase` своей орг. единицы нет. | Переименовано; операция расширена признаком субподряда | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| `MaterialPositionBase` | Абстрактная базовая материальная позиция с общими полями `Component`, `ComponentVariant`, `RateBaseUnit`, `RateUnit`, нормами расхода, округлением и местами отпуска/списания. Наследники: `ManufacturingBillPosition`, `ProcessSegmentMaterialPosition`, `ProcessOperationMaterialPosition`. Название уточнено относительно исходного "Базовая позиция спецификации", потому что база используется не только строками спецификации. | Входит | `02_domain_model.md`, `05_rules.md` |
| `ProcessSegmentMaterialPosition`, `ProcessOperationMaterialPosition` | Материальные позиции передела и операции. Наследуются от `MaterialPositionBase` и ссылаются на `ManufacturingBillPosition`; не наследуются от `ManufacturingBillPosition`. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| `ProcessEquipmentPositionBase` + segment/operation equipment positions | Нормативные позиции оборудования / рабочих мест передела и операции. Общие поля оборудования и временные нормы раскрыты в базовом типе; конкретные наследники добавляют владельца `ProcessSegment` или `ProcessOperation`. Справочники ресурсов, проверки ссылок и доступность предоставляются внешним `ResourceManagement`. | Входит как нормативные строки модуля | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| `ProcessLabourPositionBase` + segment/operation labour positions | Нормативные позиции трудовых ресурсов передела и операции: профессия, разряд, количество, подготовительно-заключительное и штучное время. Разложение штучного времени на вспомогательное/машинное и отдельное заключительное время на трудовой позиции не хранятся. Конкретные наследники добавляют владельца и ссылку на позицию оборудования своего уровня. Сотрудники, календари и доступность остаются внешней зависимостью. | Входит с уточнением норм времени | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| `ProcessToolingPositionBase` + segment/operation tooling positions | Нормативные позиции технологической оснастки передела и операции. Общие поля оснастки раскрыты в базовом типе; `Tooling` является внешней ссылкой на базовый объект `ToolBase`, а конкретные наследники добавляют владельца и ссылку на позицию оборудования своего уровня. Склад, доступность и жизненный цикл оснастки остаются внешней зависимостью. | Входит как нормативные строки модуля | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `06_ui_views.md` |
| Технологические документы `ProcessDefinition` | Модуль задает бизнес-контекст использования документов технологического описания, но не хранит собственные строки связи. Связь документа с бизнес-объектом хранится в `Document Management` через универсальный механизм документных связей. Отдельные вкладки документов для операции и перехода не проектируются в v1, так как ПР02 явно описывает вкладку документов только для технологического описания. | Входит как внешняя связанная область | `01_scope.md`, `02_domain_model.md`, `06_ui_views.md` |
| `CooperationScheme` | Корневая ревизия схемы кооперации с workflow и областями использования. | Входит | `02_domain_model.md`, `04_workflows.md`, `05_rules.md` |
| Строки схемы кооперации | Дочерние элементы без собственного workflow. | Входит | `02_domain_model.md`, `05_rules.md` |
| `UseCondition` из DMP_DATA | Единый объект условия применимости с конкретными nullable-ссылками на владельцев: `ManufacturingBill`, `ManufacturingBillPosition`, `ProcessDefinition`, `CooperationScheme`, `NomenclatureSubstitution`. Пользователь работает с ним через коллекции `UseConditions` владельцев; значение условия хранится в типизированном поле, соответствующем `ConditionSubject`, а не в универсальной строке. | Входит с уточнением runtime-публикации | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `13_operations.md` |
| `UseConditionParameterValue` / связь `NomenclatureParameterValue.UseCondition` | Строка значения реквизита условия наследуется от `GMD.NomenclatureParameterValueBase` и используется для условий типа `ItemParameter`. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md` |
| `ManufacturingBill.UseConditions` | Коллекция условий использования ревизии состава. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md` |
| `ManufacturingBillPosition.UseConditions` | Коллекция условий использования позиции состава. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md` |
| `ProcessDefinition.UseConditions` | Коллекция условий использования ревизии технологии. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md` |
| `CooperationScheme.UseConditions` | Коллекция условий использования схемы кооперации. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md` |
| `NomenclatureSubstitution.UseConditions` | Коллекция условий применения замещения. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md` |
| `Presentation`, `UseConditionsPresentation`, `ExternalId`, `Status` | `Presentation` и `UseConditionsPresentation` не являются самостоятельными бизнес-реквизитами модуля: первое формируется Object Runtime, второе выводится из коллекции условий применимости владельца. `ExternalId` наследуется из Common. `Status` не переносится как универсальный статус ПР02: для ревизий используется workflow, для расчетных объектов - расчетные статусы, для настроечных объектов - стандартные Common-механизмы архивирования и удаления. | Трассировано в платформенные/общие механизмы | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md` |
| `NomenclatureSubstitution` / `ItemSubstitution` | Правило применимости, не технологический документ. Не подключать к `ProcessDocumentLifecycle` в v1. | Входит отдельно | `02_domain_model.md`, `05_rules.md` |
| `NomenclatureSubstitution.RateUnit / RateForSetup / RateForProcessing / SubstitutionFactor` | Нормативное правило возможной замены из ПР02/DMP_DATA. Базовые количества применения замены фиксируются в расчетных и операционных объектах, а не в карточке нормативной замены. | Входит с уточнением | `02_domain_model.md`, `05_rules.md`, `13_operations.md` |
| `ProductComposition` | Расчетный результат анализа состава; не нормативный документ. Хранит параметры расчета, ручные источники нормативов, контекст условий применимости, значения реквизитов, счетчики `PositionsCount`, `WarningsCount`, `ErrorsCount` и расчетный `CompositionStatus`, не workflow утверждения. | Входит отдельно | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md`, `13_operations.md` |
| `ProductCompositionParameterValue` / связь `NomenclatureParameterValue.ProductComposition` | Строка значения реквизита расчетного контекста наследуется от `GMD.NomenclatureParameterValueBase` и используется при проверке условий применимости в анализе состава. | Входит | `02_domain_model.md`, `03_object_runtime_model.md`, `05_rules.md` |
| `CompositionPositionBase.Message`, `ProductCompositionPosition` | Сообщение результата хранится на строке `CompositionPositionBase.Message`; отдельный объект `ProductCompositionMessage` не вводится. Иерархическая ссылка строки называется `Parent`, как в PR02/DATA и иерархических объектах 01/03. | Входит с уточнением структуры | `02_domain_model.md`, `03_object_runtime_model.md`, `06_ui_views.md`, `13_operations.md` |
| `ManufacturingOrder` и связанные операционные объекты | Не входят в модуль. Это производственные операционные документы и факты, а не нормативная модель составов и технологий. | Не входит | `01_scope.md` |
| `ManufacturingOrderSegment`, `ManufacturingOrderSegmentMaterialPosition` | Не входят в модуль. Это производные строки/материальные позиции операционного выполнения. | Не входит | `01_scope.md` |
| WIP, ATP, фактические операции, складские документы и движения | Не входят в модуль. Могут использовать данные модуля в будущих модулях, но не являются объектами владения модуля. | Не входит | `01_scope.md` |
| Оборудование, рабочие места, персонал, оснастка, календари, мощности и загрузка ресурсов как мастер-данные и факты доступности; склад и жизненный цикл оснастки | Не входят в модуль. Модуль хранит только нормативные позиции потребности технологии; `Tooling` является внешней ссылкой на `ToolBase`, а справочники ресурсов и их проверки предоставляет `ResourceManagement`, доступность относится к планированию. | Не входит / внешняя зависимость | `01_scope.md`, `02_domain_model.md` |
| Документы, файлы, версии файлов, URL, электронный архив, жизненный цикл документа и связи документов с бизнес-объектами | Не входят в модуль. Объект документа, файл и связь документа с бизнес-объектом принадлежат `Document Management`; в UI модуля v1 явно описывается только связанный список документов технологического описания. | Не входит / внешняя зависимость | `01_scope.md`, `02_domain_model.md`, `06_ui_views.md` |

## 12. Граница с другими модулями

| Модуль | Связь |
|---|---|
| `00_common` | Common-поля архива/удаления, Object Runtime, workflow integration. `Status` не переносится в Common-базу. |
| `01_general_master_data` | Внешние ссылки на номенклатуру, единицы измерения, контрагентов и классификаторы. |
| `03_plant_structure` | Внешние ссылки на `OrganizationalUnit` как ответственную орг. единицу, орг. единицу выполнения операции и место хранения, на конкретный `Subcontractor` в схеме кооперации и на `WarehouseBin`; `ProductionUnit` выбирается как конкретный вид `OrganizationalUnit`, а `StorageArea` не хранится прямой ссылкой модуля 02 v1 и проверяется через данные `WarehouseBin` в `03_plant_structure`. |
| `ResourceManagement` | Владеет справочниками рабочих мест, оборудования, персонала, ресурсными классификаторами, календарями и графиками. Модуль 02 хранит нормативные позиции оборудования, персонала и оснастки; оснастка выбирается как внешний объект `ResourceManagement:ToolBase`, а ресурсные объекты используются через межмодульные ссылки. |
| `Document Management` | Владение документами, файлами, версиями, URL/хранилищами, жизненным циклом документа, электронным архивом и связями документов с бизнес-объектами. Модуль использует связанные документы как внешнюю область данных. |
| Будущие модули планирования / заказов / проектов | `ProductOrder`, `ProductOrderPosition`, `DemandGroup`, `Project`, `ProjectPhase` используются только как значения условий применимости `UseCondition`; это не владение заказами и не обратные ссылки заказов на спецификации или технологии. |
| Планирование / APS | Использует ревизии с `UseInPlanning = true`, периодом действия и подходящими условиями применимости. |
| Производственные заказы | Используют ревизии с `UseInProduction = true`. |
| Калькуляция | Использует ревизии с `UseInCosting = true`. |

## 13. Решения, которые нельзя откатить без нового обсуждения

| ID | Решение | Почему важно |
|---|---|---|
| PR02-DD-001 | Не переносить `BillStatus` и `ProcessStatus` как основной механизм продуктивной применимости. | Иначе планирование и производство снова будут зависеть от одного `Approved`. |
| PR02-DD-002 | Использовать ревизию как единицу workflow и применимости. | Позволяет поэтапный выпуск и безопасное изменение через новую ревизию. |
| PR02-DD-003 | Хранить области использования на ревизии. | Нужны быстрые и понятные продуктивные запросы. |
| PR02-DD-004 | `ProcessDefinition.ManufacturingBill = null` означает общую технологию без заранее выбранной спецификации. | Поддерживает сценарий, где одна технология применяется с разными подходящими спецификациями продукта. |
| PR02-DD-005 | `ProcessDefinition.ManufacturingBill`, если заполнена, ссылается только на индивидуальную спецификацию ТО (`ManufacturingBill.IsForProcessDefinition = true`). | Это соответствует правилу ПР02 и не смешивает обычные спецификации продукта с индивидуальной спецификацией технологии. |
| PR02-DD-006 | Выпущенную ревизию не редактировать напрямую; для существенных изменений создавать новую ревизию. | Не меняет данные под уже рассчитанными планами и производственными сценариями. |
| PR02-DD-007 | Использовать доменную ревизионность `Initial + Revision + ExternalRevision`, а не ждать общего платформенного механизма версий. | Это прямо следует из ПР02/DMP_DATA и нужно для проектирования v1. |
| PR02-DD-008 | Материальные строки технологии ссылаются на `ManufacturingBillPosition`: для индивидуальной технологии - на позиции ее индивидуальной спецификации ТО, для общей технологии - на позиции применимых спецификаций продукта. | Позволяет распределять и уточнять материалы по переделам/операциям без превращения обычной технологии в технологию под одну спецификацию. |
| PR02-DD-009 | При выпуске новой ревизии не менять старую ревизию скрыто. | Предотвращает неожиданные изменения в планировании, производстве и истории применимости. |
| PR02-DD-010 | `UseCondition` остается единым объектом. Владение задается конкретными nullable-ссылками `ManufacturingBill`, `ManufacturingBillPosition`, `ProcessDefinition`, `CooperationScheme`, `NomenclatureSubstitution`; для строки должна быть заполнена ровно одна ссылка владельца. В Object Runtime объект публикуется через коллекции `UseConditions` владельцев с соответствующим `ParentLinkMemberCode`, без отдельного `OwnerKind` / `OwnerType`. | Это соответствует DMP_DATA, сохраняет FK-целостность и дает Object Runtime обычные дочерние коллекции без пяти отдельных таблиц условий. |
| PR02-DD-011 | Производственные операционные документы и факты не входят в ПР02 и не отражаются в доменной модели модуля. | Это исключает перенос операционного контура DMP_DATA в модуль нормативной технологической НСИ. |
| PR02-DD-012 | Модуль владеет нормативными позициями оборудования, персонала и оснастки передела/операции, но не справочниками ресурсов, календарями, мощностями и фактической загрузкой. | Так же, как в ПР01/ПР03, ссылки на еще не спроектированные модули фиксируются как внешние зависимости, а не как преждевременная модель внутри текущего модуля. |
| PR02-DD-013 | Модуль не владеет документными связями. В v1 в проектной документации модуля явно описывается только связанный список документов технологического описания `ProcessDefinition`, потому что именно такая вкладка описана в ПР02. Операция и переход не получают собственные документные вкладки в `06_ui_views.md`; при необходимости они могут быть подключены общесистемной конфигурацией `Document Management` без изменения доменной модели модуля. | Требование ПР02 о файлах документов технологического описания сохраняется, но модель связей документов не дублируется внутри модуля. Управляющие программы и документы, указанные как технологическая оснастка, отражаются через позиции оснастки передела/операции. |
| PR02-DD-014 | Согласованные алгоритмические списки модуля фиксируются как `SystemEnum`; собственные `ValueSet` в v1 модуля не вводятся. `ProcessDocumentStatus` заменен workflow; `UseConditionObjectType` переносится как целевой `UseConditionSubject`, а `UseConditionType` как `UseConditionOperator`. Набор предметов условия ограничен предметами PR02/DATA: `ProductVariant`, `MainProduct`, `MainProductNumber`, `ProductOrder`, `ProductOrderPosition`, `DemandGroup`, `Project`, `ProjectPhase`, `ItemParameter`. `InputOutputPositionType`, `RoundingRule`, `NomenclatureSubstitutionType`, `DurationRateType`, `UseConditionOperator`, `CooperationSchemeChangeType`, `ProductCompositionType`, `ProductCompositionStatus` и `ProductCompositionPositionStatus` используют значения PR02. `ConfirmationStage` и `ActualAccountingMode` фиксируются как собственные SystemEnum ПР02 для нормативной операции. `NomenclatureObtainMethod` и `ProcessType` используются как внешние SystemEnum GMD. `DurationUnit` сохраняется как единый SystemEnum модуля для длительности передела и lead time схемы кооперации с расширенным набором `WorkHour`, `WorkShift`, `WorkDay`, `CalendarDay`. `RateForSetupTime`, `RateForProcessingTime`, `RateForAuxiliaryTime`, `RateForMachineTime`, `RateForWaitingTime`, `RateForIdleTime`, `RateForTransportTime`, `RateForTeardownTime` хранятся в секундах и не наследуют ЕИ от `ProcessSegmentBase.DurationUnit`; UI использует общую настройку `Common.TimeAmountFormat` типа `TimeAmountFormatType`: `Standard` = `hh:mm:ss`, `Industrial` = нормо-часы с тремя десятичными знаками. | Нужно не смешать собственные доменные enum модуля со списками GMD/Common, не смешивать предмет условия с владельцем условия, не смешивать единицу длительности с единицами количественных норм расхода / использования (`RateUnit`, `UnitOfOperation`) и не выводить наследование ЕИ временных норм от передела там, где исходный ПР02 этого не задает. |
| PR02-DD-015 | В `13_operations.md` описывать только прикладные операции, внутренние функции и предметные алгоритмы модуля: подбор спецификаций и технологий, разузлование, расчет потребностей, расчет количества, проверку циклов, входимость, изменение компонента технологии, создание новой ревизии/копии с предметным изменением компонента. Обычные create/update/delete/read и заполнение карточек не описывать как отдельные прикладные операции без предметного алгоритма модуля. | Иначе документация операций продублирует Object Runtime и потеряет фокус на реальной бизнес-логике модуля. Для новой ревизии или копии технологии с индивидуальной спецификацией ТО создается новая связанная ревизия/копия этой спецификации, чтобы изменение компонента не меняло исходную ревизию. |
| PR02-DD-016 | UI v1 проектировать по целевым объектам и основным сценариям модуля, а не переносить старые формы ПР02 механически. WF отображается стандартным блоком карточки объекта с жизненным циклом; в `06_ui_views.md` не дублировать для каждой карточки стандартные команды переходов, а раскрывать только прикладные поля, вкладки, lookup-и и предметные действия. | Это сохраняет совместимость со стилем GMD: `04_workflows.md` описывает жизненный цикл и прикладные проверки переходов, а UI-документ описывает состав списков, карточек и вкладок без повторения платформенного workflow/runtime. |
| PR02-DD-017 | Старые отдельные коды прав ПР02 на создание, изменение и удаление компонента не переносить как отдельные права. Строки состава и технологии защищаются через стандартное право `Update` на объект-владелец, WF-политики и бизнес-правила. Изменение компонента индивидуальной спецификации ТО из технологии требует `Update` на редактируемую `ProcessDefinition` и `Update` на связанную индивидуальную `ManufacturingBill`. Создание новой ревизии или копии технологии с изменением компонента проверяется стандартными правами на все исходные и создаваемые ревизии: `Read` исходной технологии, `Create` новой технологии, а при индивидуальной спецификации ТО - также `Read` исходной спецификации и `Create` новой спецификации. | Это согласует модуль с ПР01/ПР03: строки коллекций не получают отдельные права, стандартные действия Object Runtime не дублируются, а отличие ревизионных сценариев фиксируется в алгоритмах и workflow-политиках, а не во взрыве кодов прав. Прямое изменение компонента доступно только для редактируемой технологии с индивидуальной спецификацией ТО. |
| PR02-DD-018 | Алгоритмы продуктивного подбора не используют workflow-состояние как самостоятельный бизнес-фильтр. Подбор выполняется по хранимым признакам и правилам: Common/Object Runtime `IsArchived` и признак удаления, области использования модуля, `AllowNewOperationalSelection`, период действия, условия применимости и совместимость связанных ревизий. | Это предотвращает возврат к старому `Status == Approved` и разделяет ответственность: workflow управляет жизненным циклом и редактируемостью, а бизнес-алгоритмы используют явные хранимые признаки применимости. |
| PR02-DD-019 | Абстрактные базовые типы модуля описываются централизованно в `02_domain_model.md`, раздел 2.2. Для v1 раскрываются `ProcessBase`, `ProcessSegmentBase`, `ProcessOperationBase`, `ProcessStepBase`, `MaterialPositionBase`, `ProcessEquipmentPositionBase`, `ProcessLabourPositionBase`, `ProcessToolingPositionBase`, `CompositionPositionBase`. `UseCondition` не является абстрактным базовым типом в целевой модели: это единый объект строки условия с конкретными owner-FK. `ProcessOperationSequenceBase` из исходного ПР02 не раскрывается в v1, потому что сложные технологические связи и сетевые графики вынесены за границу v1. | Единый принцип документации убирает дублирование полей и не смешивает базовые типы с конкретными объектами владельцев. |
| PR02-DD-020 | Поля `ShopFloor` и `ShopFloorArea` исходного ПР02 переименованы в целевой модели, потому что их типом является `OrganizationalUnit`, а старые имена звучат как конкретный цех/участок и провоцируют ошибочное сужение до `ProductionUnit`. `ShopFloor` в `ProcessSegmentBase` и `ProcessOperationBase` фиксируется единым именем `ResponsibleOrgUnit`; `ShopFloorArea` в `ProcessOperationBase` фиксируется как необязательное `ExecutionOrgUnit`; `IsSubcontracted` остается отдельным Boolean в `ProcessSegmentBase` и добавляется в `ProcessOperationBase` как целевое расширение для субподрядной отдельной операции. Runtime-контракт объявляет эти поля на базовых object contracts, а конкретные `ProcessSegment` и `ProcessOperation` получают их через наследование. | Это сохраняет смысл ПР02 и согласуется с ПР03: исполнитель всегда `OrganizationalUnit`, а выбор `ProductionUnit` или `Subcontractor` определяется признаком субподряда и правилами lookup/валидации; место выполнения операции является уточнением и не обязательно для субподрядной операции. |
| PR02-DD-021 | Поля ПР02 `SubmissionDateForStatement` и `StatementDate` переименованы в `SubmittedForApprovalAt` и `ApprovedAt`. Они хранят последний факт отправки / утверждения текущей ревизии, по умолчанию заполняются действиями workflow-переходов `SubmitForApproval` и `Approve`, могут корректироваться по workflow-policy и не участвуют в продуктивном подборе. | Даты нужны для отображения и миграции смысла ПР02, но не должны становиться вторым механизмом применимости рядом с workflow, областями использования и `ValidFrom` / `ValidTo`. |
| PR02-DD-022 | Служебные поля ПР02 `Presentation`, `UseConditionsPresentation`, `ExternalId` и `Status` не добавляются как отдельные бизнес-поля объектов модуля. `Presentation` задается Object Runtime, `UseConditionsPresentation` формируется из строк условий, `ExternalId` наследуется из Common, а `Status` раскладывается по типу объекта: workflow ревизии, расчетный статус результата или стандартные Common-механизмы настроечного объекта. | Это не дает создать второй источник истины рядом с runtime-представлением, условиями применимости, Common-полями и workflow. |
| PR02-DD-023 | `LabourType`, `ProcessingOperation`, `ManufacturingOperationType` и `WorkEnvironment` входят в ПР02 как классификаторы технологии. `PaymentGroup` не переносится в ПР02 и используется как внешняя ссылка на ПР04. `ConfirmationStage` и `ActualAccountingMode` фиксируются как собственные SystemEnum ПР02. | ПР04 явно владеет группой оплаты, но не содержит остальные классификаторы операции из ПР02. Разделение сохраняет нормативную модель технологии в ПР02 и не затягивает ресурсную тарификацию в модуль составов и технологий. |
| PR02-DD-024 | Для базовых типов `ProcessSegmentBase`, `ProcessOperationBase` и `ProcessStepBase` сохраняется пара полей ПР02: `Number: Int` и `NumberInProcess: String(10)`. `Number` является числовым номером внутри владельца, `NumberInProcess` - номером в технологическом процессе; если `NumberInProcess` не задан, он формируется из `Number` по длине из `ProcessTemplate`. | Это убирает лишний маппинг целевых имен, сохраняет структуру ПР02 и не смешивает внутренний числовой номер с внешним номером в ТП. |
| PR02-DD-025 | Поля передела `IsAccountingPoint`, `IsCreateManufacturingLot`, `IsAssignSerialNumbers`, `TimeBuffer`, `YieldFactor`, `ManufacturingLossFactor`, `LossQty`, `FactorRoundingRule` включаются в `ProcessSegmentBase`; поле ПР02/DATA `IsAutoComplet` включается с исправленным именем `IsAutoComplete`; старое целевое `LossFactor` заменяется на `ManufacturingLossFactor`. | Это возвращает в модель операционные признаки передела и нормативные параметры выхода/потерь, сохраняя понятные целевые имена и трассировку отличий от исходной опечатки. |
| PR02-DD-026 | Временная норма `ProcessOperationBase.RateForTeardownTime` сохраняется в целевой модели по Word/UI ПР02 как норма заключительного времени операции. `ProcessOperationBase.ProductivityFactor` сохраняется как `Decimal(24,8)`, несмотря на точность `Decimal(24,2)` в Word ПР02. | `RateForTeardownTime` участвует в расчете времени операции и не должен теряться из-за неполной DATA-таблицы. `Decimal(24,8)` сохраняет единый формат коэффициентов целевой доменной модели; ограничение отображения или ввода до двух знаков может быть UI-правилом, но не сужает хранимый тип. |
| PR02-DD-027 | В `ProcessStepBase` не вводится поле `Name`; добавляется `SpecialInstructions: String(max)`. Представление `ProcessTransition` строится по номеру и номеру в ТП без имени. | Word/DATA ПР02 для перехода задают `Number`, `NumberInProcess`, `Description`, `SpecialInstructions`, `Presentation`, `ExternalId`; `Presentation` и `ExternalId` уже трассированы в runtime/Common, а отдельного наименования перехода нет. |
| PR02-DD-028 | `ObtainMethod`, `ProcessType`, `ValidFrom` и `ValidTo` остаются полями `ManufacturingBillPosition` и не добавляются в `ProcessSegmentMaterialPosition` / `ProcessOperationMaterialPosition`. | Материальные позиции технологии распределяют или уточняют строку состава через ссылку `ManufacturingBillPosition`; отдельные значения этих полей на техстроках создали бы второй источник применимости и способа обеспечения. |
| PR02-DD-029 | В `ProcessLabourPositionBase` хранится только прямой блок трудового времени: `RateTypeForSetup`, `RateForSetupTime`, `RateTypeForProcessing`, `RateForProcessingTime`. `RateForAuxiliaryTime`, `RateForMachineTime`, `CalculateRateForProcessingTime`, `RateTypeForTeardown`, `RateForTeardownTime` не вводятся для трудовой позиции. `RateTimeType` из DATA не вводится в базовые типы операции и оборудования v1. | Подготовительно-заключительная занятость трудового ресурса учитывается в `RateForSetupTime`; отдельное заключительное время и машинное разложение относятся к операции/оборудованию и не должны создавать неясную трудовую семантику. `RateTimeType` является видом нормы для тарификации нормо-часа и прямой ЗП; если понадобится для нарядов, проектируется отдельно в операционном контуре или в конкретных наследниках уровня операций, а не протаскивается через общий базовый тип. |
| PR02-DD-030 | `ProcessToolingPositionBase.Tooling` фиксируется как внешняя ссылка на `ResourceManagement:ToolBase`, а не как прямая ссылка на `GMD.Nomenclature`. Допустимые конкретные типы ресурса: `Tooling` и `Gage`. | Номенклатура оснастки является реквизитом объекта оснастки, а владение карточкой оснастки остается за `ResourceManagement`; модуль 02 хранит только нормативную ссылку потребности технологии. |
| PR02-DD-031 | `ProductComposition` включает контекст условий применимости PR02/DATA: `MainProduct`, `MainProductSerialNumber`, `DemandGroup`, `Project`, `ProjectPhase`, `ProductOrder`, `ProductOrderPosition` и `ProductCompositionParameterValue`. Расчетный статус хранится как `CompositionStatus`. Поле Word `IsApprovedOnly` не переносится, потому что подбор в целевой модели выполняется по областям использования, архивности, периоду и условиям применимости, а не по workflow/status. | Это сохраняет сценарий анализа состава из PR02 без возврата к статусному фильтру и без превращения заказов/проектов во владение модуля. |
| PR02-DD-032 | Для диагностики анализа состава не вводится отдельный объект `ProductCompositionMessage`. Статус строки хранится в `CompositionPositionBase.PositionStatus`, текст диагностики - в `CompositionPositionBase.Message`; заголовок анализа хранит счетчики строк, предупреждений и ошибок. | Word PR02 и DATA задают сообщение как поле расчетной позиции, а не как отдельную коллекцию сообщений. |
| PR02-DD-033 | Операции создания, изменения и удаления компонента технологии работают с `ProcessSegmentMaterialPosition` и связанной `ManufacturingBillPosition` индивидуальной спецификации ТО. Для общей технологии `ProcessDefinition.ManufacturingBill = null` скрытая индивидуальная спецификация не создается; материальные позиции технологии ссылаются на позиции применимых спецификаций продукта по правилам `PR02-DD-008`. | Это сохраняет функциональный сценарий PR02, но не ломает принятое разделение общей технологии и технологии с индивидуальной спецификацией ТО. |
| PR02-DD-034 | Меню модуля строится по разделу ПР00 `Составы и технологии` и открывает только самостоятельные списки Object Runtime модуля. Пункты ПР00 `Группы оплаты` и `Виды допуска персонала` не публикуются в меню 02, потому что относятся к Resource Management. `ManufacturingOperationType` публикуется как собственный пункт `Виды операций`, так как это классификатор модуля 02. | Меню должно соответствовать целевой границе владения: 02 публикует свои нормативные объекты, но не переносит в себя ресурсные справочники. |

## 14. Контроль документов модуля

| Документ | Что должен закрыть |
|---|---|
| `01_scope.md` | Границы модуля, внешние зависимости и исключенные области. |
| `02_domain_model.md` | Объекты, ревизии, поля областей использования, связи состава и технологии. |
| `03_object_runtime_model.md` | Runtime-контракты, lookup-и, правила выбора ревизий и ручного связывания. |
| `04_workflows.md` | `ProcessDocumentLifecycle`, состояния, переходы, `WorkflowStatePolicy`, `WorkflowRuleBinding`. |
| `05_rules.md` | Проверки ревизий, областей использования, сроков действия, условий применимости и совместимости связей. |
| `06_ui_views.md` | Карточки, списки, блок `Области использования`, вкладки, lookup-и и предметные UI-действия; стандартный WF-блок только упоминается как возможность карточки объекта с жизненным циклом. |
| `07_reports_outputs.md` | Отчеты, печатные формы, выходные документы и выгрузки; фиксирует отсутствие собственных выходных форм v1 и границу будущих ЕСТД/интеграционных выходов. |
| `10_value_set_data_usage.md` | Зафиксировать, что модуль v1 не вводит собственные `ValueSetData`. |
| `11_permissions.md` | Права на создание, изменение, утверждение, архивирование и просмотр ревизий; строки коллекций защищаются через владельца, ревизионные алгоритмы не получают отдельных кодов прав в v1. |
| `12_audit_history.md` | События аудита и история изменений ревизий, областей использования, условий применимости, анализа состава и замещений. |
| `13_operations.md` | Прикладные операции, функции и алгоритмы подбора составов, технологий, схем кооперации, разузлования, анализа состава, входимости, замен и предметного изменения компонента технологии. |
| `15_navigation_menu.md` | Пункты главного меню по ПР00, целевые списки Object Runtime и представления без самостоятельного пункта меню. |

## 15. Маршрут материала по архитектуре номенклатуры

Сохранённый внешний материал по архитектуре номенклатуры использован для
сверки, но не является нормативным документом модуля 02. Его копия удалена.

| Часть материала | Текущее решение | Документ-владелец | Статус |
|---|---|---|---|
| Владение `Nomenclature` и связанными основными данными | Модуль 02 не владеет этими объектами и использует их как внешние ссылки. | `01_general_master_data`, `01_scope.md`, `02_domain_model.md` | Принято |
| Использование `ObjectType`, `View`, `Action`, `Rule`, `ValueSet` и `DataSet` | Описано только фактическое использование модулем 02; общие определения принадлежат платформенным областям. | `03_object_runtime_model.md`, `06_ui_views.md`, `05_rules.md`, `13_operations.md` и документы платформы | Перенесено |
| Общая будущая модель Product Definition и специализированные запросы | Не является текущим контрактом модуля 02. Возможные дополнительные решения проверяются отдельно. | `backlog.md` | Будущая доработка |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.2 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | Трассировка объектов; Граница с другими модулями; Решения, которые нельзя откатить без нового обсуждения | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Маршрут материала по архитектуре номенклатуры | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-19 12:02 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [87c0b0a5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/87c0b0a51fdabf232ce704082d41545ae007bedc) |
