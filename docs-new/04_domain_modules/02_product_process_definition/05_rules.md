---
id: DOC-04-02-05
title: 'Правила и ограничения - 02 Составы и технологии'
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

# Правила и ограничения - 02 Составы и технологии

## 1. Назначение документа

Документ фиксирует проверяемые правила модуля `02 Product & Process Definition`: инварианты объектов, ограничения ссылок, проверки выпуска ревизий и правила продуктивного подбора.

Полный состав полей описан в `02_domain_model.md`. Прикладные операции, функции и алгоритмы описаны в `13_operations.md`.

## 2. Общие правила Common

Для объектов на базе Common применяются общие правила:

- обычные списки и lookup-и скрывают `IsDeleted = true` и `IsArchived = true`;
- уже сохраненная ссылка на архивный объект отображается;
- новая ссылка на архивный объект запрещена, если в документе явно не описан иной сценарий;
- пользовательские payload-и не изменяют audit/archive/delete-поля Common напрямую.

Служебные поля и представления:

| Код | Правило |
|---|---|
| `PR02-COM-001` | `ExternalId` используется как Common-поле и не дублируется в payload конкретных объектов модуля. |
| `PR02-COM-002` | `Presentation` формируется Object Runtime по правилу типа объекта и не редактируется пользователем как отдельный реквизит. |
| `PR02-COM-003` | `UseConditionsPresentation` формируется из коллекции `UseConditions` владельца и не является самостоятельным источником условий применимости. |
| `PR02-COM-004` | `Status` не используется как единый доменный статус модуля: для ревизионных документов используется `ProcessDocumentLifecycle`, для расчетных объектов - собственные расчетные статусы, для настроечных объектов - стандартные Common-механизмы архивирования и удаления. |

## 2.1 Tenant

| Код | Правило | Проверка | Сообщение |
|---|---|---|---|
| `PR02-TENANT-001` | Объект принадлежит предприятию | `TenantId` заполнен для каждого объекта модуля. | Объект должен принадлежать предприятию. |
| `PR02-TENANT-002` | Дочерний объект принадлежит предприятию владельца | Для дочернего, зависимого, расчетного или связующего объекта `TenantId` совпадает с `TenantId` владельца. | Дочерний объект должен относиться к тому же предприятию, что и владелец. |
| `PR02-TENANT-003` | Ссылки не пересекают границы предприятия | Для tenant-scoped ссылки `ReferencedObject.TenantId = Owner.TenantId`, если отдельным правилом не описан меж-tenant сценарий. | Ссылочный объект должен относиться к тому же предприятию. |
| `PR02-TENANT-004` | `TenantId` не изменяется пользовательским запросом | При изменении объекта значение `TenantId` не отличается от сохраненного значения. | Предприятие объекта нельзя изменить пользовательским запросом. |

## 2.2 Количественные нормы и единицы операции

| Код | Правило |
|---|---|
| `PR02-UOM-001` | Материальные и количественные нормы модуля хранятся как Decimal + `UnitOfOperation`. |
| `PR02-UOM-002` | Поля `Unit`, `RateUnit` и другие поля типа `UnitOfOperation` должны ссылаться на применимую единицу операции из GMD для соответствующей НП и области применения. |
| `PR02-UOM-003` | Списки выбора `UnitOfOperation`, значения по умолчанию, проверки применимости и пересчеты выполняются через функции GMD, а не через прямой выбор из справочника `Unit`. |
| `PR02-UOM-004` | Для объектов модуля область применения единицы операции по умолчанию - производство (`UnitApplicationAreaFlags.Production`), если конкретный сценарий не задает иной контекст. |
| `PR02-UOM-005` | `DurationUnit` не применяется к материальным и количественным нормам. |
| `PR02-UOM-006` | Для количественных норм, участвующих в расчетах и имеющих базовую пару в доменной модели, хранится системно рассчитанное базовое значение в базовой ЕИ НП: `QuantityBaseUnit`, `RateForSetupBaseUnit`, `RateForProcessingBaseUnit` и аналогичные поля. |
| `PR02-UOM-007` | Базовые количественные поля не редактируются пользователем независимо; они пересчитываются через функции GMD при изменении НП, количества или `UnitOfOperation`. |
| `PR02-UOM-008` | `NomenclatureSubstitution` не получает собственные поля базовой нормы. Базовые количества и нормы применения замены фиксируются в объекте результата: анализ состава, расчетная потребность, заказ, логистический документ или иной операционный объект. |

Правила количеств заголовков:

| Код | Правило | Проверка | Сообщение |
|---|---|---|---|
| `PR02-UOM-HEADER-001` | Базовая ЕИ спецификации соответствует продукту | `ManufacturingBill.BaseUnit` заполнен базовой ЕИ `ManufacturingBill.Product`. | Базовая ЕИ спецификации должна соответствовать базовой ЕИ продукта. |
| `PR02-UOM-HEADER-002` | Количество спецификации в БЕИ пересчитано из ЕИ количества | `ManufacturingBill.QuantityBaseUnit` соответствует пересчету `ManufacturingBill.Quantity` из `ManufacturingBill.Unit` в `ManufacturingBill.BaseUnit` через функции GMD. | Количество спецификации в базовой ЕИ не соответствует количеству в ЕИ операции. |
| `PR02-UOM-HEADER-003` | Базовая ЕИ анализа состава соответствует продукту | `ProductComposition.BaseUnit` заполнен базовой ЕИ `ProductComposition.Product`. | Базовая ЕИ анализа состава должна соответствовать базовой ЕИ продукта. |
| `PR02-UOM-HEADER-004` | Количество анализа состава в БЕИ пересчитано из ЕИ количества | `ProductComposition.ProductQtyBaseUnit` соответствует пересчету `ProductComposition.ProductQty` из `ProductComposition.Unit` в `ProductComposition.BaseUnit` через функции GMD. | Количество анализа состава в базовой ЕИ не соответствует количеству в ЕИ операции. |

Правила `MaterialPositionBase`:

| Код | Правило | Проверка | Сообщение |
|---|---|---|---|
| `PR02-UOM-009` | Базовая ЕИ нормы соответствует компоненту | `RateBaseUnit` заполнен базовой ЕИ `MaterialPositionBase.Component`. Если значение не передано пользователем, оно определяется по базовой ЕИ компонента. | Базовая ЕИ нормы должна соответствовать базовой ЕИ компонента. |
| `PR02-UOM-010` | ЕИ нормы допустима для компонента | `RateUnit` является допустимой `UnitOfOperation` для `MaterialPositionBase.Component` в контексте нормы. | ЕИ нормы недопустима для выбранного компонента. |
| `PR02-UOM-011` | Базовая норма на наладку пересчитана из ЕИ нормы | Если `RateForSetup` заполнена, `RateForSetupBaseUnit` соответствует пересчету `RateForSetup` из `RateUnit` в `RateBaseUnit` через функции GMD. | Норма на наладку в базовой ЕИ не соответствует норме в ЕИ операции. |
| `PR02-UOM-012` | Базовая норма на обработку пересчитана из ЕИ нормы | Если `RateForProcessing` заполнена, `RateForProcessingBaseUnit` соответствует пересчету `RateForProcessing` из `RateUnit` в `RateBaseUnit` через функции GMD. | Норма на обработку в базовой ЕИ не соответствует норме в ЕИ операции. |
| `PR02-UOM-013` | Норма в ЕИ операции округлена по точности `UnitOfOperation` | `RateForSetup` и `RateForProcessing` не содержат больше знаков после запятой, чем разрешено `RateUnit.Precision`. | Норма в ЕИ операции должна соответствовать точности выбранной единицы. |
| `PR02-UOM-014` | Базовые нормы не редактируются независимо | Изменение `RateForSetupBaseUnit` или `RateForProcessingBaseUnit` пользовательским запросом без изменения исходной нормы, компонента или `RateUnit` отклоняется. | Базовые нормы рассчитываются системой и не редактируются вручную. |

## 3. Ревизионность

| Код | Правило |
|---|---|
| `PR02-REV-001` | Для первой ревизии `Initial` указывает на саму себя. |
| `PR02-REV-002` | Внутри одной цепочки `Initial` значение `Revision` уникально. |
| `PR02-REV-003` | `Revision` больше нуля. |
| `PR02-REV-004` | Новая ревизия создается через предметную операцию `CreateNewRevision`. |
| `PR02-REV-005` | Выпущенная ревизия не редактируется напрямую. Для изменения создается новая ревизия. |
| `PR02-REV-006` | При выпуске новой ревизии старая ревизия не изменяется скрыто. Пользователь явно закрывает `ValidTo`, снимает область использования или архивирует старую ревизию. |
| `PR02-REV-012` | `SubmittedForApprovalAt` и `ApprovedAt` хранят последний факт отправки / утверждения текущей ревизии; workflow заполняет их по умолчанию, а ручная корректировка разрешается только workflow-политикой и правами пользователя. |
| `PR02-REV-013` | При создании новой ревизии `SubmittedForApprovalAt` и `ApprovedAt` не копируются из исходной ревизии. |
| `PR02-REV-014` | Продуктивный подбор спецификаций, технологий и схем кооперации не использует `SubmittedForApprovalAt` и `ApprovedAt`; применимость определяется областями использования, периодом действия, условиями применимости, архивностью и совместимостью связей. |
| `PR02-REV-015` | Если `SubmittedForApprovalAt` и `ApprovedAt` заполнены, `ApprovedAt` не может быть раньше `SubmittedForApprovalAt`. |

### 3.1 Выпуск ревизии

| Код | Правило | Проверка | Сообщение |
|---|---|---|---|
| `PR02-REV-007` | Для выпуска выбрана область использования | При переходе `Approve` хотя бы одно поле равно `true`: `UseInPlanning`, `UseInProduction`, `UseInCosting`. | Для выпуска ревизии выберите хотя бы одну область использования. |
| `PR02-REV-008` | `AllowNewOperationalSelection` не заменяет область использования | Если `AllowNewOperationalSelection = true`, то хотя бы одно поле равно `true`: `UseInPlanning`, `UseInProduction`, `UseInCosting`. | Разрешение выбора в новых рабочих ссылках не является областью использования. |
| `PR02-REV-009` | Ревизия готова для планирования | Если `UseInPlanning = true`, объект проходит правила готовности планирования своего типа. | Ревизия не готова для использования в планировании. |
| `PR02-REV-010` | Ревизия готова для производства | Если `UseInProduction = true`, объект проходит правила готовности производства своего типа. | Ревизия не готова для использования в производстве. |
| `PR02-REV-011` | Ревизия готова для калькуляции | Если `UseInCosting = true`, объект проходит правила готовности калькуляции своего типа. | Ревизия не готова для использования в калькуляции. |

Готовность `ManufacturingBill`:

- заполнены продукт и ревизия;
- период применимости корректен, если даты заданы;
- есть позиции состава, если выбранная область требует нормативный состав;
- позиции проходят правила `PR02-BILL-*` и `PR02-MAT-*`.

Готовность `ProcessDefinition`:

- заполнены продукт и ревизия;
- период применимости корректен, если даты заданы;
- тип процесса участвует в совместимости и подборе, если заполнен;
- если задана связанная спецификация, она совместима по правилам `PR02-LINK-*`;
- при `UseInProduction = true` технология содержит производственные данные по правилам `PR02-PROC-*`, `PR02-MAT-*`, `PR02-RES-*`.

Готовность `CooperationScheme`:

- заполнены основной продукт и ревизия;
- период применимости корректен, если даты заданы;
- позиции схемы проходят правила `PR02-COOP-*`;
- ссылки на спецификации, технологии, замену и субподрядчика заполнены согласно типу изменения позиции схемы.

## 4. Период действия

| Код | Правило |
|---|---|
| `PR02-VALIDITY-001` | Если `ValidFrom` и `ValidTo` заполнены, `ValidFrom <= ValidTo`. |
| `PR02-VALIDITY-002` | Ревизия применима на дату, если `ValidFrom` пусто или `ValidFrom <= date`, и `ValidTo` пусто или `date <= ValidTo`. |
| `PR02-VALIDITY-003` | Для продуктивного использования нельзя иметь две неархивные ревизии с пересекающимися периодами, одинаковой областью использования и одинаковым контекстом применимости. |

## 5. Продуктивный подбор

Алгоритмы продуктивного подбора не используют `WorkflowStateCode` как самостоятельный фильтр.

Правила строк `UseCondition`:

| Код | Правило |
|---|---|
| `PR02-UC-001` | Строка `UseCondition` принадлежит ровно одному владельцу: состав, позиция состава, технология, схема кооперации или правило замены. |
| `PR02-UC-002` | Для строки `UseCondition` должна быть заполнена ровно одна из ссылок владельца: `ManufacturingBill`, `ManufacturingBillPosition`, `ProcessDefinition`, `CooperationScheme`, `NomenclatureSubstitution`. |
| `PR02-UC-003` | Если `ValidFrom` и `ValidTo` строки условия заполнены, `ValidFrom <= ValidTo`. |
| `PR02-UC-004` | `ComparisonOperator` должен быть совместим с типом значения, заданным через `ConditionSubject`. |
| `PR02-UC-005` | Ссылка владельца заполняется из контекста коллекции и не выбирается пользователем вручную. Предмет проверки задается полем `ConditionSubject`. |
| `PR02-UC-006` | В строке условия должно быть заполнено значение только для выбранного `ConditionSubject`. Для `ItemParameter` должны быть заполнены `ItemParameter` и коллекция `ParameterValues`; остальные значения должны быть пустыми. |
| `PR02-UC-007` | Строки `UseConditionParameterValue` проверяются по контракту `GMD.NomenclatureParameterValueBase`: тип значения, допустимость значения и источник допустимых значений определяются реквизитом `ItemParameter`. |
| `PR02-UC-008` | Для предметов условия, кроме `MainProductNumber`, допустимы только операторы `Equal` и `NotEqual`. Для `MainProductNumber` допустимы все операторы `UseConditionOperator`. |
| `PR02-UC-009` | `ProductVariant` должен быть совместим с продуктом владельца или расчетного контекста. Если владелец условия уже задает исполнение продукта, значение `ProductVariant` ограничивается этим исполнением. |

`UseCondition` считается подходящим под контекст по общему правилу:

- если коллекция условий владельца пуста, дополнительных ограничений нет;
- если коллекция заполнена, должны выполниться все строки условий, применимые на дату проверки;
- строка условия применима на дату, если ее `ValidFrom` пусто или `ValidFrom <= date`, и `ValidTo` пусто или `date <= ValidTo`;
- значение из контекста выбирается по `ConditionSubject` и сравнивается с соответствующим типизированным значением строки через `ComparisonOperator`;
- если для обязательного предмета условия в контексте нет значения, условие не выполнено.

### 5.1 Планирование

Ревизия доступна для планирования, если:

```text
IsDeleted = false
IsArchived = false
UseInPlanning = true
ValidFrom / ValidTo покрывают дату расчета
UseCondition подходит под контекст
связанные ревизии совместимы по UseInPlanning
```

### 5.2 Производство

Ревизия доступна для производственного использования, если:

```text
IsDeleted = false
IsArchived = false
UseInProduction = true
ValidFrom / ValidTo покрывают дату операции или заказа
UseCondition подходит под производственный контекст
связанные ревизии совместимы по UseInProduction
```

### 5.3 Калькуляция

Ревизия доступна для калькуляции, если:

```text
IsDeleted = false
IsArchived = false
UseInCosting = true
ValidFrom / ValidTo покрывают дату расчета
UseCondition подходит под контекст калькуляции
связанные ревизии совместимы по UseInCosting
```

### 5.4 Новые рабочие ссылки

Для новой рабочей ссылки дополнительно требуется:

```text
AllowNewOperationalSelection = true
```

Этот признак не заменяет `UseInPlanning`, `UseInProduction` или `UseInCosting`. Он только запрещает выбор ревизии в новых ссылках, сохраняя уже созданные ссылки.

## 6. Правила шаблонов

| Код | Правило |
|---|---|
| `PR02-TPL-001` | `ManufacturingBillTemplate.Code` уникален в пределах текущего `Tenant`. |
| `PR02-TPL-002` | `ProcessTemplate.Code` уникален в пределах текущего `Tenant`. |
| `PR02-TPL-003` | В пределах текущего `Tenant` должен быть не более одного активного `ManufacturingBillTemplate` с `IsDefault = true`. |
| `PR02-TPL-004` | В пределах текущего `Tenant` должен быть не более одного активного `ProcessTemplate` с `IsDefault = true`. |
| `PR02-TPL-005` | Шаги нумерации шаблонов должны быть положительными целыми числами. |
| `PR02-TPL-006` | Длины номеров в ТП у `ProcessTemplate` должны быть положительными целыми числами. |
| `PR02-TPL-007` | Шаблон задает параметры создания и нумерации; он не является владельцем строк спецификации, переделов, операций, переходов или ресурсных позиций. |
| `PR02-TPL-008` | `ManufacturingBill.ManufacturingBillTemplate` и `ProcessBase.ProcessTemplate` фиксируют шаблон, на основании которого создан объект; последующее изменение шаблона не должно скрыто менять уже созданную спецификацию или технологию. |

## 7. Совместимость спецификации и технологии

| Код | Правило |
|---|---|
| `PR02-LINK-001` | `ProcessDefinition.ManufacturingBill = null` означает общую технологию: применимая спецификация продукта подбирается отдельно по продукту, исполнению, типу процесса, периоду действия, областям использования и условиям применимости. |
| `PR02-LINK-002` | Если `ProcessDefinition.ManufacturingBill` заполнена, она указывает на индивидуальную спецификацию ТО, а не на обычную спецификацию продукта. |
| `PR02-LINK-003` | `ProcessDefinition.ManufacturingBill`, если заполнена, должна ссылаться на `ManufacturingBill` с `IsForProcessDefinition = true`. |
| `PR02-LINK-004` | Для индивидуальной спецификации ТО `ProcessDefinition.Product` должен совпадать с `ManufacturingBill.Product`. |
| `PR02-LINK-005` | Если заполнены исполнения, `ProductVariant` технологии и индивидуальной спецификации ТО должны быть совместимы. |
| `PR02-LINK-006` | Если `ProcessType` заполнен у технологии и индивидуальной спецификации ТО, значения должны быть совместимы. |
| `PR02-LINK-007` | Одна ревизия индивидуальной спецификации ТО не должна быть указана в `ProcessDefinition.ManufacturingBill` более чем у одной технологии. |
| `PR02-LINK-008` | При продуктивном подборе общей технологии и спецификации совместимость областей использования проверяется между выбранными объектами результата подбора; для `ProcessDefinition.ManufacturingBill = null` нет заранее связанной спецификации, которую надо выпускать вместе с технологией. |

## 8. Правила спецификации

| Код | Правило |
|---|---|
| `PR02-BILL-001` | `ManufacturingBill.ProductVariant`, если заполнен, должен принадлежать `ManufacturingBill.Product`. |
| `PR02-BILL-002` | `ManufacturingBillPosition.PositionNumber` уникален внутри `ManufacturingBill`. |
| `PR02-BILL-003` | Унаследованное поле `MaterialPositionBase.ComponentVariant`, если заполнено в `ManufacturingBillPosition`, должно принадлежать `MaterialPositionBase.Component`. |
| `PR02-BILL-004` | `ComponentManufacturingBill`, если заполнен, должен относиться к номенклатуре позиции. |
| `PR02-BILL-005` | `ComponentProcessDefinition`, если заполнен, должен относиться к номенклатуре позиции. |
| `PR02-BILL-006` | `ReleaseWarehouseBin`, если заполнена, должна относиться к `ReleaseWarehouse`; `IssueWarehouseBin`, если заполнена, должна относиться к `IssueWarehouse`. Принадлежность зоны хранения проверяется модулем `03_plant_structure` по данным самой ячейки. |
| `PR02-BILL-007` | Количественные нормы расхода используют `RateUnit` / `UnitOfOperation`, а не `DurationUnit`. |
| `PR02-BILL-008` | Обычная спецификация продукта имеет `IsForProcessDefinition = false` и может подбираться независимо от технологии. |
| `PR02-BILL-009` | Индивидуальная спецификация ТО имеет `IsForProcessDefinition = true` и используется как `ProcessDefinition.ManufacturingBill` для конкретной технологии. |
| `PR02-BILL-010` | Если `ManufacturingBillPosition.ValidFrom` и `ManufacturingBillPosition.ValidTo` заполнены, `ValidFrom <= ValidTo`. |
| `PR02-BILL-011` | При получении действующих позиций спецификации учитываются только строки, у которых `ValidFrom` пусто или `ValidFrom <= date`, и `ValidTo` пусто или `date <= ValidTo`. |

## 9. Правила технологии

| Код | Правило |
|---|---|
| `PR02-PROC-001` | Унаследованное поле `ProcessBase.ProductVariant`, если заполнено в `ProcessDefinition`, должно принадлежать `ProcessBase.Product`. |
| `PR02-PROC-002` | Унаследованное поле `ProcessSegmentBase.Number` уникально внутри `ProcessDefinition`. |
| `PR02-PROC-003` | Унаследованное поле `ProcessOperationBase.Number` уникально внутри `ProcessSegment`. |
| `PR02-PROC-004` | Унаследованное поле `ProcessStepBase.Number` уникально внутри `ProcessOperation`. |
| `PR02-PROC-005` | Если `ProcessSegmentBase.NumberInProcess` не задан, он заполняется из `ProcessSegmentBase.Number` с длиной `ProcessTemplate.SegmentNumberInProcessLength`; итоговое значение уникально внутри `ProcessDefinition`. |
| `PR02-PROC-006` | Если `ProcessOperationBase.NumberInProcess` не задан, он заполняется из `ProcessOperationBase.Number` с длиной `ProcessTemplate.OperationNumberInProcessLength`; итоговое значение уникально внутри `ProcessSegment`. |
| `PR02-PROC-007` | Если `ProcessStepBase.NumberInProcess` не задан, он заполняется из `ProcessStepBase.Number` с длиной `ProcessTemplate.StepNumberInProcessLength`; итоговое значение уникально внутри `ProcessOperation`. |
| `PR02-PROC-008` | `ProcessSegmentBase.DurationRate` интерпретируется в `ProcessSegmentBase.DurationUnit`. |
| `PR02-PROC-009` | `RateForSetupTime`, `RateForProcessingTime`, `RateForAuxiliaryTime`, `RateForMachineTime`, `RateForWaitingTime`, `RateForIdleTime`, `RateForTransportTime`, `RateForTeardownTime` хранятся в секундах. |
| `PR02-PROC-010` | Если `ProcessSegmentBase.IsSubcontracted = true`, `ProcessSegmentBase.ResponsibleOrgUnit` должен указывать на `Subcontractor`; если `false`, должен указывать на `ProductionUnit`. Выбранная орг. единица должна быть доступна на уровне передела (`IsSegmentLevel = true`). |
| `PR02-PROC-011` | При `UseInProduction = true` технология должна содержать данные, достаточные для производственного использования по принятым правилам внедрения. |
| `PR02-PROC-012` | Если `ProcessOperationBase.CalculateRateForProcessingTime = true`, `ProcessOperationBase.RateForProcessingTime` рассчитывается из `ProcessOperationBase.RateForAuxiliaryTime` и `ProcessOperationBase.RateForMachineTime` в секундах. |
| `PR02-PROC-013` | `ProcessOperationBase.ResponsibleOrgUnit` должен ссылаться на организационную единицу текущего `Tenant` с `IsOperationLevel = true`; `ProcessOperationBase.ExecutionOrgUnit`, если заполнена, должна ссылаться на организационную единицу текущего `Tenant` с `IsOperationLevel = true`. Совместимость с ответственной орг. единицей передела проверяется по правилам `03_plant_structure` и настройкам конфигурации процесса. |
| `PR02-PROC-014` | `ProcessBase.OutputWarehouse`, `ProcessSegmentBase.OutputWarehouse` и `ProcessSegmentBase.ScrapWarehouse`, если заполнены, должны ссылаться на `OrganizationalUnit` с `IsInventoryStorageLocation = true`. |
| `PR02-PROC-015` | `ProcessBase.OutputWarehouseBin`, `ProcessSegmentBase.OutputWarehouseBin` и `ProcessSegmentBase.ScrapWarehouseBin`, если заполнены, должны относиться к указанному месту хранения; принадлежность зоны хранения проверяется модулем `03_plant_structure` по данным самой ячейки. |
| `PR02-PROC-016` | Если `ProcessOperationBase.IsSubcontracted = true`, `ProcessOperationBase.ResponsibleOrgUnit` должен указывать на `Subcontractor`; если `false`, должен указывать на `ProductionUnit`. Выбранная орг. единица должна быть доступна на уровне операции (`IsOperationLevel = true`). |
| `PR02-PROC-017` | При субподрядном переделе или операции нормативные позиции ресурсов могут быть запрещены правилами конфигурации процесса. |
| `PR02-PROC-018` | Временная норма с типом "на единицу" интерпретируется как норма на одну базовую ЕИ нормируемого выхода операции или ресурсной позиции. |
| `PR02-PROC-019` | `ProcessOperationBase.NumberOfItemsSimultaneouslyProcessed` и `ProcessOperationBase.ProductivityFactor` участвуют в расчете времени операции; аналогичные поля ресурсной позиции оборудования участвуют в расчете времени этой ресурсной позиции. |
| `PR02-PROC-020` | `ProcessSegmentBase` не получает `NumberOfItemsSimultaneouslyProcessed`, `ProductivityFactor` и временные нормы операции; длительность передела задается через `DurationRateType`, `DurationUnit` и `DurationRate`. |
| `PR02-PROC-021` | Если `ProcessSegmentBase.Name` не задано и задан `ProcessSegmentBase.LabourType`, имя передела может быть заполнено из `LabourType.Name`. |
| `PR02-PROC-022` | Если `ProcessOperationBase.ProcessingOperation` заполнена, ее `LabourType` и `ManufacturingOperationType`, если они заполнены в классификаторе, используются как значения по умолчанию для операции. |
| `PR02-PROC-023` | Если `ProcessOperationBase.Name` не задано и заполнена `ProcessingOperation`, имя операции заполняется из `ProcessingOperation.Name`; иначе, если заполнен `ManufacturingOperationType`, имя заполняется из `ManufacturingOperationType.Name`. |
| `PR02-PROC-024` | `ProcessOperationBase.ManufacturingOperationType` обязателен для операции, выпускаемой для производственного использования. |
| `PR02-PROC-025` | Если `ProcessOperationBase.ConfirmationStages` не задана, она заполняется из `ManufacturingOperationType.ConfirmationStages`; если итоговая схема подтверждения не определена, операция не готова для производственного использования. |
| `PR02-PROC-026` | `ProcessOperationBase.PaymentGroup` и `ProcessLabourPositionBase.PaymentGroup` ссылаются на внешний справочник `PaymentGroup` ресурсного модуля и не создают собственный справочник или enum модуля. |
| `PR02-PROC-027` | `ManufacturingOperationType.ActualAccountingMode` задает нормативный способ учета факта операции: по количеству или по времени. |
| `PR02-PROC-028` | После применения значений по умолчанию `ProcessSegmentBase.Name` должен быть заполнен. |
| `PR02-PROC-029` | Если `ProcessSegmentBase.DurationRateType` не задан, применяется `PerLot`; если `ProcessSegmentBase.DurationUnit` не задан, применяется `WorkShift`. |
| `PR02-PROC-030` | `ProcessSegmentBase.DurationRate`, `TimeBuffer`, `YieldFactor`, `ManufacturingLossFactor` и `LossQty` не могут быть отрицательными. |
| `PR02-PROC-031` | `ProcessSegmentBase.IsAutoComplete`, `IsAccountingPoint`, `IsCreateManufacturingLot`, `IsAssignSerialNumbers`, `TimeBuffer`, `YieldFactor`, `ManufacturingLossFactor`, `LossQty` и `FactorRoundingRule` должны быть заданы до сохранения передела. |
| `PR02-PROC-032` | `ProcessSegmentBase.FactorRoundingRule` использует общий `RoundingRule` модуля и применяется при расчете значений, зависящих от коэффициентов выхода и технологических потерь. |

## 10. Правила материальных и ресурсных позиций технологии

| Код | Правило |
|---|---|
| `PR02-MAT-001` | `ProcessSegmentMaterialPosition.PositionNumber` уникален внутри `ProcessSegment`. |
| `PR02-MAT-002` | `ProcessOperationMaterialPosition.PositionNumber` уникален внутри `ProcessOperation`. |
| `PR02-MAT-003` | Если `ProcessDefinition.ManufacturingBill` заполнена, `ProcessSegmentMaterialPosition.ManufacturingBillPosition` и `ProcessOperationMaterialPosition.ManufacturingBillPosition` должны относиться к этой индивидуальной спецификации ТО. |
| `PR02-MAT-004` | Если `ProcessDefinition.ManufacturingBill = null`, технологическая материальная строка может ссылаться на позицию применимой спецификации продукта; применимость проверяется по продукту, исполнению, типу процесса, периоду действия, областям использования и условиям применимости. |
| `PR02-MAT-005` | `MaterialPositionBase.ComponentVariant`, если заполнен, должен принадлежать `MaterialPositionBase.Component`. |
| `PR02-MAT-006` | `MaterialPositionBase.RateBaseUnit` должен соответствовать базовой ЕИ `MaterialPositionBase.Component`. |
| `PR02-MAT-007` | `MaterialPositionBase.RateUnit` должен быть допустимой `UnitOfOperation` для `MaterialPositionBase.Component` через функции GMD. |
| `PR02-MAT-008` | `ManufacturingBillPosition` не является владельцем материальной позиции технологии; связь с ним является ссылкой, а не наследованием. |
| `PR02-MAT-LINK-001` | `ObtainMethod`, `ProcessType`, `ValidFrom` и `ValidTo` относятся к `ManufacturingBillPosition` и не дублируются в материальных позициях технологии. |
| `PR02-RES-001` | Номер позиции оборудования, персонала и оснастки уникален внутри владельца: `ProcessSegment` или `ProcessOperation`. |
| `PR02-RES-002` | Позиции оборудования, персонала и оснастки являются нормативными строками технологии, но не карточками конкретных ресурсов `ResourceManagement`. |
| `PR02-RES-003` | Проверка фактической доступности оборудования, персонала, рабочих мест и оснастки не входит в модуль. |
| `PR02-RES-004` | Если позиция персонала или оснастки ссылается на позицию оборудования, ссылка должна указывать на позицию того же владельца и уровня: передел к переделу, операция к операции. |
| `PR02-RES-005` | Если у позиции оборудования `CalculateRateForProcessingTime = true`, ее `RateForProcessingTime` рассчитывается из `RateForAuxiliaryTime` и `RateForMachineTime` в секундах. |
| `PR02-RES-006` | Позиция трудового ресурса хранит подготовительно-заключительное время и итоговое штучное время: `RateTypeForSetup`, `RateForSetupTime`, `RateTypeForProcessing`, `RateForProcessingTime`. Составляющие `RateForAuxiliaryTime`, `RateForMachineTime`, `CalculateRateForProcessingTime` и отдельное заключительное время на трудовой позиции не хранятся. |
| `PR02-RES-007` | `ProcessToolingPositionBase.Tooling` является нормативной ссылкой на внешний `ResourceManagement:ToolBase`; допустимые конкретные типы (`Tooling`, `Gage`) и ограничения ссылки предоставляет модуль-владелец, а фактическая доступность определяется планированием или операционным контуром. |

Правила типа входа/выхода материальной позиции:

| Код | Правило | Проверка | Сообщение |
|---|---|---|---|
| `PR02-MAT-TYPE-001` | Тип входа/выхода обязателен | `MaterialPositionBase.InputOutputType` заполнен. | Для материальной позиции должен быть указан тип входа/выхода. |
| `PR02-MAT-TYPE-002` | Обычная материальная позиция по умолчанию является нормой расхода | При создании материальной позиции без явно переданного `InputOutputType` устанавливается `RateForMaterialConsumption`. | Нет ошибки. |
| `PR02-MAT-TYPE-003` | Выходы, отходы и образцы задаются явно | Значения `RateForByProductOutput`, `RateForReclaimableWasteOutput`, `RateForUnreclaimableWasteOutput` и `RateForSample` устанавливаются только явным пользовательским вводом или прикладной операцией. | Тип выходной материальной позиции должен быть задан явно. |
| `PR02-MAT-TYPE-004` | Расчет потребности использует нормы расхода | Алгоритмы расчета потребности включают в расход только строки с `InputOutputType = RateForMaterialConsumption`; выход попутного продукта, отходы и образцы обрабатываются отдельными правилами результата. | Тип материальной позиции не соответствует расчету расхода. |

## 11. Правила связанных технологических документов

| Код | Правило |
|---|---|
| `PR02-DOC-001` | Связи документов технологического описания `ProcessDefinition` хранятся в `Document Management`, а не как собственные строки модуля. |
| `PR02-DOC-002` | Модуль может проверять наличие обязательных связанных документов технологического описания только через внешний контракт `Document Management`. Без такого контракта выпуск технологии не должен зависеть от внутренних статусов или файлов документов. |
| `PR02-DOC-003` | Загрузка, версия, предпросмотр, архивное хранение файла, URL и права на документ не являются функциональностью модуля. |
| `PR02-DOC-004` | Отдельные документные вкладки операции и перехода не проектируются в v1 модуля. Если `Document Management` общесистемно подключает документы к этим объектам, это не создает новых строк или правил владения в модуле. |

## 12. Правила схем кооперации

| Код | Правило |
|---|---|
| `PR02-COOP-001` | `CooperationScheme.MainProductVariant`, если заполнен, должен принадлежать `MainProduct`. |
| `PR02-COOP-002` | `CooperationSchemePosition.PositionNumber` уникален внутри `CooperationScheme`. |
| `PR02-COOP-003` | `CooperationSchemePosition.NomenclatureVariant`, если заполнен, должен принадлежать `Nomenclature`. |
| `PR02-COOP-004` | `ComponentManufacturingBill`, если заполнен, должен относиться к `Nomenclature` позиции. |
| `PR02-COOP-005` | `LeadTime` интерпретируется в `LeadTimeUnit`. |
| `PR02-COOP-006` | Если `LeadTime` заполнен, `LeadTimeUnit` должен быть заполнен. |
| `PR02-COOP-007` | `LeadTime` и `ProductLotSize`, если заполнены, не могут быть отрицательными. |

## 13. Правила замен

| Код | Правило |
|---|---|
| `PR02-SUB-001` | Исполнение заменяемой НП должно принадлежать заменяемой НП. |
| `PR02-SUB-002` | Исполнение заменяющей НП должно принадлежать заменяющей НП. |
| `PR02-SUB-003` | Если заполнены `ValidFrom` и `ValidTo`, `ValidFrom <= ValidTo`. |
| `PR02-SUB-004` | Условия применения замены проверяются тем же механизмом `UseCondition`, что и условия ревизий. |
| `PR02-SUB-005` | `SubstitutionFactor` является коэффициентом пересчета заменяемого количества в количество заменяющего компонента. Он не является единицей измерения и не заменяет пересчеты `UnitOfOperation`. |
| `PR02-SUB-006` | `RateUnit`, `RateForSetup`, `RateForProcessing` в `NomenclatureSubstitution` задают нормативные значения правила замены в выбранной `UnitOfOperation`. Базовые значения результата применения замены рассчитываются в расчетном / операционном объекте, а не хранятся в самом правиле замены. |

## 14. Правила анализа состава

| Код | Правило |
|---|---|
| `PR02-COMP-001` | `ProductComposition.ProductVariant`, если заполнен, должен принадлежать `Product`. |
| `PR02-COMP-002` | Количество анализа состава в базовой ЕИ проверяется по общему правилу `PR02-UOM-HEADER-004`. |
| `PR02-COMP-003` | `ProductCompositionPosition.PositionNumber` уникален внутри `ProductComposition`. |
| `PR02-COMP-004` | `ProductCompositionPosition.Level` не меньше нуля. |
| `PR02-COMP-005` | Расчетные статусы `ProductCompositionStatus` и `ProductCompositionPositionStatus` не являются workflow-состояниями. Заголовок анализа хранит статус в поле `CompositionStatus`. |
| `PR02-COMP-006` | `ProductComposition.MainProductSerialNumber`, если заполнен, должен принадлежать `MainProduct`. |
| `PR02-COMP-007` | `ProductComposition.ProjectPhase`, если заполнен, должен принадлежать `Project`. |
| `PR02-COMP-008` | `ProductComposition.ProductOrderPosition`, если заполнена, должна принадлежать `ProductOrder`. |
| `PR02-COMP-009` | Строки `ProductCompositionParameterValue` проверяются по контракту `GMD.NomenclatureParameterValueBase` и используются как часть контекста проверки условий применимости. |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | Правила материальных и ресурсных позиций технологии | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-19 12:02 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [87c0b0a5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/87c0b0a51fdabf232ce704082d41545ae007bedc) |
