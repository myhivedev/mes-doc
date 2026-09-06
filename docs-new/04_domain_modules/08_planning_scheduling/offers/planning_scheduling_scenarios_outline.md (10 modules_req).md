# Сценарии планирования и расписаний — обзор

Дата уточнения: 2026-07-10.

## 1. Назначение документа

Документ фиксирует иерархический перечень возможных сценариев, связанных с модулем **планирования и расписаний** (`Planning & Scheduling`) и его взаимодействием с другими прикладными модулями Digital Manufacturing Platform.

Цель текущей версии — не детальное описание сценариев, а согласование состава и структуры сценарного каталога.

Документ опирается на:

- обсуждённую архитектуру модуля планирования и расписаний, вариант B;
- модель `проекция входных данных планирования -> профиль входных данных расчета -> адаптер расчетной модели -> снимок данных или временная входная модель -> расчет -> результат`;
- типовые корпоративные сценарии: фиксация базового плана, оперативное перепланирование, проверка реализуемости заказа;
- сценарии быстрого расчета;
- состав прикладных модулей DMP.

---

## 2. Общая классификация сценариев

### 2.1 По режиму расчёта

#### 2.1.1 Быстрый расчет

Быстрый синхронный или почти синхронный расчет для прикладного сценария другого модуля.

Примеры:

- разузлование заказа;
- получение состава;
- расчёт потребностей;
- проверка обеспеченности;
- предварительная оценка сроков;
- оценка доступности ресурсов.

Результат обычно возвращается модулю-потребителю и сохраняется им, если это часть его прикладного сценария.

#### 2.1.2 Асинхронный расчет планового сценария

Полноценный расчёт сценария планирования.

Примеры:

- построение плана на период;
- объёмно-календарное планирование;
- пооперационное расписание;
- оперативное перепланирование;
- моделирование "что если";
- расчёт нескольких вариантов.

Результат хранится в модуле планирования и расписаний как `PlanningResult` / `PlanningVariant`.

#### 2.1.3 Расчет через внешний решатель

Расчет через внешний APS, внешний решатель или специализированный оптимизационный движок.

Используется как вариант исполнения расчетного контура, не меняя контракты DMP.

---

### 2.2 По назначению сценария

#### 2.2.1 Создание плана

Создание нового планового варианта на период или горизонт планирования.

#### 2.2.2 Перепланирование

Перепланирование с учётом изменений, отклонений и фактического состояния.

#### 2.2.3 Прогноз

Прогноз ожидаемых сроков, загрузки, дефицитов и отклонений без обязательной публикации нового плана.

#### 2.2.4 Моделирование "что если"

Моделирование изменения условий без немедленного изменения операционных данных.

#### 2.2.5 Фиксация базового плана

Фиксация утвержденного плана как базового плана для анализа отклонений.

#### 2.2.6 Проверка реализуемости

Оценка реализуемости нового спроса, заказа или заявки по срокам, материалам и ресурсам.

---

## 3. Межмодульные сценарии планирования и расписаний

Раздел 3 описывает реальные межмодульные прикладные сценарии планирования и расписаний.

Цель раздела — проверить пригодность архитектуры:

```text
проекция входных данных планирования
  -> Calculation Input Profile / Model Adapter
  -> Snapshot or temporary input model
  -> Calculation
  -> Result
  -> Базовый план / Publication / Consumer persistence / Analytics
```

Каждый сценарий ниже описывается через:

- назначение;
- режим расчёта;
- основные участвующие модули и сервисы;
- входные данные;
- ход выполнения;
- результат;
- что сохраняется и где;
- что меняется в операционных данных;
- архитектурные проверки.

---

### 3.1 Общая карта сервисов, участвующих в сценариях

#### Модуль планирования и расписаний

Основные компоненты:

```text
Planning API
Planning Application Services
Planning Scenario Service
Planning Input Projection Store
Planning Input Projection Ingestion
Source Watermark Manager
Calculation Input Profile Registry
Calculation Model Adapters
Snapshot Cut Manager
Snapshot Builder
Scheduling Compute Runtime
Planning Result Store
Plan Baseline Service
Plan Publication Service
Planning Read Models
```

#### Платформенные сервисы

```text
IAM / Tenant Context
Configuration Platform
Object Runtime
Workflow Engine
Rule Engine
Integration/Event Platform
Audit / History / Trace
Reporting / Analytics
```

Object Runtime в сценариях отвечает за стандартные пользовательские объекты
модуля планирования и расписаний: сценарии, запуски, варианты, публикации,
базовые планы, статусы, действия, аудит и исходящие события изменяющих операций.
Расчетный контур (`Scheduling Compute Runtime`) отвечает только за вычисление по подготовленному снимку данных и не должен напрямую
изменять эти объекты.

#### Прикладные модули

```text
Product Definition
Process Definition
Resource Management
Plant Structure
Order Management
Production Logistics
Shopfloor Execution
Quality Management
Equipment Integration / MDC
Manufacturing Analytics
Integration Service
```

---

### 3.2 Общая цепочка расчета

Большинство межмодульных сценариев используют общую цепочку обработки.

```text
1. Запрос прикладного сценария
   - действие в интерфейсе
   - вызов из прикладного модуля
   - задание по расписанию
   - процесс, запущенный событием

2. Определение контекста расчета
   - TenantId
   - PlantId
   - область расчета
   - горизонт планирования
   - ScenarioPurpose
   - CalculationMode
   - ProfileCode

3. Определение и проверка свежести входных данных
   - проекция входных данных планирования
   - SourceWatermarks
   - Profile FreshnessPolicy
   - необязательное чтение через публичные контракты, если проекция не готова

4. Подготовка входных данных расчета
   - Calculation Input Profile
   - Calculation Model Adapter
   - SnapshotCut
   - сохраненный снимок данных или временная входная модель

5. Выполнение расчета
   - ядро быстрого расчета
   - Scheduling Compute Runtime
   - External Solver Adapter, если используется

6. Сохранение или возврат результата
   - PlanningResult / PlanningVariant
   - PlanBaseline
   - ForecastResult
   - FeasibilityResult DTO
   - сохранение результата модулем-потребителем

7. Необязательное утверждение или публикация
   - Workflow
   - проверки правил
   - изменение через Object Runtime, если меняется стандартный объект
   - Plan Publication Service
   - события
   - аудит / история
```

Ключевое правило:

```text
Расчёт не равен публикации.
Результат не равен базовому плану.
Forecast не равен утверждённому плану.
Проверка реализуемости не создает производственный заказ сама по себе.
```

Если сценарий меняет стандартный объект Planning, Order, Shopfloor или Logistics,
изменение должно идти через соответствующий прикладной сервис и Object Runtime,
а не напрямую из расчетного контура.

---

### 3.3 Создание планового варианта на период

#### Назначение

Построить плановый вариант на заданный горизонт для выбранной области расчета.

Примеры:

- недельный план по заводу;
- месячный план по цеху;
- план по группе заказов;
- объёмно-календарный план;
- детальное расписание по участку или группе ресурсов.

#### Классификация

```text
ScenarioPurpose = CreatePlan
CalculationMode = AsyncPlanningRun
Typical ProfileCode = CapacityPlanning / OperationScheduling / FullApsPlanning
ResultOwnership = Planning & Scheduling
PublicationPolicy = optional approval + publish
```

#### Участвующие сервисы и модули

```text
Planning Scenario Service
Planning Input Projection Store
Calculation Input Profile Registry
Calculation Model Adapter
Snapshot Builder
Scheduling Compute Runtime
Planning Result Store
Workflow Engine
Rule Engine
Audit / History

Order Management
Product Definition
Process Definition
Resource Management
Plant Structure
Production Logistics
Shopfloor Execution, если учитывается WIP/fact
Quality Management, если есть quality constraints
```

#### Ход выполнения

```text
Planner creates planning scenario
  ↓
Planning resolves область расчета / horizon / parameters
  ↓
Planning selects Calculation Input Profile
  ↓
Source Watermark Manager validates freshness
  ↓
Snapshot Cut Manager creates SnapshotCut
  ↓
Calculation Model Adapter builds persisted PlanningSnapshot
  ↓
PlanningRun is queued
  ↓
Расчетный контур расписаний загружает снимок данных
  ↓
Расчетный контур calculates plan
  ↓
PlanningResult and PlanningVariant are stored
  ↓
Planner reviews result / issues / KPIs / Gantt / load
```

#### Результат

```text
PlanningVariant
PlanningResult
PlanIssues
PlanKpi
PlannedOrders / PlannedOperations
ResourceLoadBuckets
MaterialRequirements
PeggingLinks, если применимо
```

#### Что меняется в операционных данных

Ничего автоматически не меняется.

```text
CreatePlan produces proposal.
Operational data changes only after explicit publication.
```

---

### 3.4 Сравнение вариантов плана

#### Назначение

Сравнить несколько рассчитанных вариантов и выбрать лучший по срокам, загрузке, дефицитам, стабильности, переналадкам и другим KPI.

#### Классификация

```text
ScenarioPurpose = CreatePlan / WhatIf / Replan
CalculationMode = read/analysis over existing PlanningResults
ResultOwnership = Planning & Scheduling
PublicationPolicy = none until variant is approved
```

#### Участвующие сервисы

```text
Planning Result Store
Planning Read Models
Reporting / Analytics
UI Dashboard / Gantt / Resource Load views
```

#### Ход выполнения

```text
Planner selects variants
  ↓
Planning loads comparable PlanningResults
  ↓
Analytics / Reporting calculates comparison metrics
  ↓
UI shows KPI differences, issues, resource load, delays
  ↓
Planner selects candidate for approval
```

#### Результат

```text
VariantComparison
KpiComparison
IssueComparison
ResourceLoadComparison
DelayComparison
```

#### Что меняется в операционных данных

Ничего.

---

### 3.5 Утверждение варианта плана

#### Назначение

Зафиксировать решение, что выбранный вариант пригоден для дальнейшей публикации или фиксации как базовый план.

#### Классификация

```text
ScenarioPurpose = CreatePlan / Replan / WhatIf
CalculationMode = no calculation
ResultOwnership = Planning & Scheduling
PublicationPolicy = approval workflow
```

#### Участвующие сервисы

```text
Planning Application Service
Workflow Engine
Rule Engine
IAM
Audit / History
Planning Result Store
```

#### Ход выполнения

```text
Planner chooses variant
  ↓
IAM checks permission
  ↓
Workflow checks available command
  ↓
Rule Engine validates approval conditions
  ↓
PlanningVariant status changes to Approved
  ↓
Audit / History record is created
```

#### Результат

```text
PlanningVariant.Status = Approved
WorkflowHistory
AuditRecord
```

#### Что меняется в операционных данных

Ничего. Утверждение ещё не означает публикацию.

---

### 3.6 Публикация утверждённого плана

#### Назначение

Передать утверждённый плановый результат в операционные модули.

#### Классификация

```text
ScenarioPurpose = CreatePlan / Replan
CalculationMode = no calculation
ResultOwnership = Planning & Scheduling + consumer modules
PublicationPolicy = explicit publish
```

#### Участвующие сервисы и модули

```text
Plan Publication Service
Workflow Engine
Rule Engine
IAM
Integration/Event Platform
Audit / History

Order Management
Shopfloor Execution
Production Logistics
Integration Service
Manufacturing Analytics
```

#### Ход выполнения

```text
Planner starts Publish action
  ↓
IAM / Workflow / Rule checks
  ↓
Plan Publication Service reads Approved PlanningResult
  ↓
Publication mapping is built for each consumer module
  ↓
Events / API commands are sent through stable contracts
  ↓
Consumers apply changes in their own aggregates
  ↓
Publication status is stored
  ↓
Audit / History / Integration events are created
```

#### Возможные публикационные эффекты

```text
Order Management
  - update planned dates
  - update planning status
  - link order to plan version / publication

Shopfloor Execution
  - create/update dispatch lists
  - create/update operation queues
  - mark published operation priorities

Production Logistics
  - receive material requirement dates
  - update kitting priorities
  - receive shortage priorities

Integration Service
  - send plan data to ERP / external systems if required
```

#### Что меняется в операционных данных

Меняется только через публичные контракты модулей-потребителей.

Planning не пишет напрямую в таблицы Order / Shopfloor / Logistics.

---

### 3.7 Фиксация плана за период / Plan Базовый план

#### Назначение

Зафиксировать утверждённый план как управленческую базу для анализа отклонений факта от плана.

Это один из ключевых сценариев для больших заводов, потому что план должен быть не только рассчитан и опубликован, но и сохранён как reference point для последующего анализа.

#### Классификация

```text
ScenarioPurpose = BaselineFreeze
CalculationMode = no new calculation after approved result
ResultOwnership = Planning & Scheduling / Analytics
PublicationPolicy = baseline freeze, not operational publish
BaselinePolicy = period-based / scope-based
```

#### Участвующие сервисы и модули

```text
Plan Baseline Service
Planning Result Store
Planning Read Models
Workflow Engine
Rule Engine
IAM
Audit / History
Manufacturing Analytics
Shopfloor Execution facts
Production Logistics facts
Equipment Integration / MDC facts
Quality facts
```

#### Входные данные

```text
Approved PlanningVariant
PlanningResult
PeriodStart / PeriodEnd
Область расчета: Plant / workshop / resource group / order set
BaselineType: weekly / monthly / operational / management
```

#### Ход выполнения

```text
Planner selects Approved PlanningVariant
  ↓
Planner chooses базовый план period and область расчета
  ↓
IAM checks permission to freeze plan
  ↓
Workflow validates that variant can become базовый план
  ↓
Rule Engine validates completeness and period rules
  ↓
Plan Baseline Service materializes базовый план views
  ↓
PlanBaseline is created
  ↓
PlanBaselineCreated event is published
  ↓
Audit / History records are created
  ↓
Analytics can compare actuals against this базовый план
```

#### Результат

```text
PlanBaseline
PlanBaselineOrders
PlanBaselineOperations or stages
PlanBaselineResourceLoad
PlanBaselineMaterialRequirements
PlanBaselineKpi
```

Степень детализации базового плана зависит от политики:

```text
Базовый план на уровне заказов
Базовый план на уровне переделов
Базовый план на уровне операций
Базовый план загрузки ресурсов
Базовый план потребностей в материалах
```

#### Что меняется в операционных данных

Обычно ничего.

PlanBaseline — это аналитическая и управленческая фиксация, а не изменение производственных заданий.

#### Как потом используется

```text
Shopfloor actuals
Production Logistics actuals
MDC actual machine events
Quality blocks/results
  ↓
Manufacturing Analytics
  ↓
Plan vs Actual deviation analysis
```

Примеры отклонений:

```text
planned start vs actual start
planned finish vs actual finish
planned quantity vs actual quantity
planned resource load vs actual resource state
planned material requirement vs actual issue / shortage
planned WIP vs actual WIP
```

#### Архитектурная проверка

```text
PlanningResult != PlanBaseline.
Базовый план создается только явным действием фиксации.
Факт сравнивается с базовым планом через аналитику и модели чтения.
Базовый план не блокирует последующее перепланирование.
```

---

### 3.8 Оперативное перепланирование / Operational Replanning

#### Назначение

Пересчитать план или расписание с учётом текущего факта, WIP, простоев, дефицитов, изменений доступности ресурсов и других отклонений.

#### Классификация

```text
ScenarioPurpose = Replan
CalculationMode = AsyncPlanningRun
Typical ProfileCode = OperationScheduling / CapacityPlanning / FullApsPlanning
ResultOwnership = Planning & Scheduling
PublicationPolicy = optional approval + publish
```

#### Участвующие сервисы и модули

```text
Planning Scenario Service
Planning Input Projection Store
Source Watermark Manager
Calculation Input Profile Registry
Calculation Model Adapter
Snapshot Cut Manager
Snapshot Builder
Scheduling Compute Runtime
Planning Result Store
Workflow Engine
Rule Engine
Audit / History

Shopfloor Execution
Production Logistics
Resource Management
Equipment Integration / MDC
Quality Management
Order Management
Manufacturing Analytics
```

#### Входные данные

```text
Current orders / demand
Current WIP
Actual operation statuses
Material availability / shortages
Resource availability / downtime
Quality blocks
Current published plan or базовый план
Frozen zone policy
```

#### Ход выполнения

```text
Actuals and state changes update проекция входных данных планирования
  ↓
Planner or system starts Operational Replanning
  ↓
Planning resolves ScenarioPurpose = Replan
  ↓
Planning selects profile and область расчета
  ↓
Source Watermark Manager validates freshness
  ↓
SnapshotCut is created at current time
  ↓
Calculation Model Adapter builds Replanning Snapshot
  ↓
Adapter marks operations as completed / fixed / movable
  ↓
Расчетный контур расписаний рассчитывает новый ожидаемый план
  ↓
PlanningResult is stored as Replanning Variant
  ↓
Planner reviews delays, affected orders, shortages, overloads
  ↓
Optional approval and publication
```

#### Frozen zone rules

Для больших заводов это критично. Replanning не должен произвольно менять всё.

Примеры правил:

```text
Started operations are fixed.
Operations in current shift are fixed or restricted.
Published dispatch tasks are fixed unless explicitly released.
Completed operations are facts and cannot be changed.
Partially completed batches are recalculated only for remaining quantity.
High-priority orders may be protected from delay.
```

#### Результат

```text
Replanning PlanningVariant
PlanningResult
Expected dates
Delay risks
Affected orders
Resource overloads
Material shortages
PlanIssues
```

#### Что меняется в операционных данных

До publication ничего.

Если результат утверждён и опубликован:

```text
Order Management receives updated planned dates.
Shopfloor receives updated dispatch / operation queue.
Production Logistics receives updated material requirement priorities.
```

#### Архитектурная проверка

```text
Replanning uses current facts through проекция входных данных планирования.
Replanning result is not automatically published.
Frozen zone is encoded into snapshot constraints.
Consumers apply changes through their own contracts.
```

---

### 3.9 Оперативный прогноз сроков / Operational Forecast

#### Назначение

Рассчитать ожидаемые сроки выполнения заказов и операций без обязательного изменения опубликованного плана.

Это близко к replanning, но семантика другая: цель — прогноз и контроль рисков, а не обязательно подготовка нового плана к публикации.

#### Классификация

```text
ScenarioPurpose = Forecast
CalculationMode = AsyncPlanningRun or small async
Typical ProfileCode = OperationScheduling / CapacityPlanning
ResultOwnership = Planning & Scheduling / Analytics
PublicationPolicy = usually none
```

#### Участвующие сервисы и модули

```text
Planning Input Projection Store
Snapshot Cut Manager
Calculation Model Adapter
Scheduling Compute Runtime
Planning Result Store
Manufacturing Analytics
Shopfloor Execution
Production Logistics
MDC
Quality Management
```

#### Ход выполнения

```text
Current facts update проекция входных данных планирования
  ↓
System or planner starts Forecast calculation
  ↓
Profile and freshness policy are resolved
  ↓
SnapshotCut is created
  ↓
Adapter builds Forecast Snapshot
  ↓
Расчетный контур calculates expected dates and risks
  ↓
Forecast Result is stored or returned
  ↓
Analytics/UI shows expected delays and risk indicators
```

#### Результат

```text
ExpectedFinishDates
ExpectedStartDates
DelayRisk
CriticalOrders
Bottlenecks
MaterialRisk
ResourceRisk
DeviationFromБазовый план
```

#### Что меняется в операционных данных

Ничего.

Forecast — это аналитический результат, а не команда на изменение плана.

#### Отличие от replanning

```text
Forecast asks: what will likely happen?
Replanning asks: what new plan should we use?
```

---

### 3.10 What-if моделирование

#### Назначение

Смоделировать изменение условий без немедленного изменения операционные данные.

Примеры изменений:

```text
Add candidate order
Change order priority
Exclude resource
Change calendar / shift availability
Change lot size
Delay material receipt
Change planning parameter
```

#### Классификация

```text
ScenarioPurpose = WhatIf
CalculationMode = AsyncPlanningRun or small async
ResultOwnership = Planning & Scheduling
PublicationPolicy = none by default
```

#### Ход выполнения

```text
Planner selects base scenario / result / базовый план
  ↓
Planner defines PlannerChanges
  ↓
Planning creates WhatIf Scenario
  ↓
Snapshot is built from base input + changes
  ↓
Расчетный контур calculates WhatIf Variant
  ↓
Planner compares WhatIf Variant with base plan
```

#### Результат

```text
WhatIfVariant
WhatIfResult
ImpactSummary
KpiDelta
AffectedOrders
```

#### Что меняется в операционных данных

Ничего, пока результат не будет явно утверждён и опубликован.

---

### 3.11 Быстрая оценка нового заказа / Feasibility & Promise Check

#### Назначение

Быстро оценить новый заказ, заявку или клиентский запрос по срокам, материалам и мощностям без полного перепланирования всего завода.

Это второй ключевой сценарий для проверки архитектуры, потому что он должен использовать Planning foundation, но не должен создавать тяжёлый planning scenario в каждом случае.

#### Классификация

```text
ScenarioPurpose = FeasibilityCheck
CalculationMode = БыстрыйCalculation or SmallAsyncCalculation
Typical ProfileCode = Feasibility / ATP / CTP / OrderExplosion subset
ResultOwnership = Consumer module or transient Planning result
PublicationPolicy = none
```

#### Участвующие сервисы и модули

```text
Order Management or external request UI
Planning Быстрый Calculation API
Planning Input Projection Store
Calculation Input Profile Registry
Calculation Model Adapter
Calculation Kernel
Необязательный расчетный контур расписаний для малого асинхронного расчета
Audit / Trace optional

Product Definition
Process Definition
Resource Management
Production Logistics
Order Management
Shopfloor Execution, if current commitments are considered
```

#### Входные данные

```text
CandidateDemand
- ItemCode
- Quantity
- RequestedDate
- Priority
- Customer / Project / Contract refs
- Configuration parameters
- Required attributes
- PromisePolicy

Current committed plan
Current inventory / expected receipts
Current resource load / capacity
Current WIP / fixed commitments
```

#### Ход выполнения: быстрый быстрый-вариант

```text
Order Management or user creates CandidateDemand
  ↓
Consumer calls Planning Быстрый Calculation API
  ↓
Planning resolves Feasibility Profile
  ↓
Planning checks projection freshness
  ↓
Adapter builds transient feasibility input model
  ↓
Calculation Kernel checks material and/or capacity feasibility
  ↓
Planning returns FeasibilityResult DTO
  ↓
Consumer decides whether to create/accept order
```

#### Ход выполнения: сложный small async / "что если" вариант

```text
CandidateDemand is large or high-impact
  ↓
Planning creates WhatIf Scenario
  ↓
CandidateDemand is added as PlannerChange
  ↓
Snapshot is built
  ↓
Расчетный контур calculates impact
  ↓
Planner compares result with current plan
```

#### Возможные политики проверки

**Без изменения существующего плана**

```text
Do not move committed operations.
Fit new demand only into free capacity and available supply.
```

**С ограниченным влиянием**

```text
May move only non-published operations.
May use alternative routing/version if allowed.
Must not delay protected orders.
Must not change current shift.
```

**Полный "что если"**

```text
Large candidate demand requires full scenario calculation.
```

#### Результат

```text
FeasibilityResult
- CanPromise
- EarliestPossibleDate
- PromiseOptions
- MaterialShortages
- CapacityGaps
- BottleneckStages or resource groups
- ImpactSummary
- RequiredAssumptions
- Diagnostics
```

#### Что меняется в операционных данных

Ничего автоматически.

Если заказ принимается, Order Management создаёт или изменяет свои aggregates.

Planning не создаёт production order напрямую.

#### Архитектурная проверка

```text
CandidateDemand != ProductionOrder.
FeasibilityResult != PlanningResult, unless explicitly persisted.
Быстрый calculation uses profiles/adapters and must not bypass planning contracts.
```

---

### 3.12 Анализ производственного состава изделия / конфигурации / заказа

#### Назначение

Раскрыть и проверить производственный состав изделия, конкретной конфигурации или заказа с учётом параметров, которые влияют на состав, применимость версий и выбор технологии.

Это не scheduling-сценарий. Он не подбирает конкретные ресурсы и не строит расписание.

#### Классификация

```text
ScenarioPurpose = EngineeringValidation / FeasibilityCheck / WhatIf, depending on context
CalculationMode = БыстрыйCalculation or SmallAsyncCalculation
Typical ProfileCode = ProductionCompositionAnalysis
ResultOwnership = transient analysis result or persisted validation/report
PublicationPolicy = none
```

#### Участвующие сервисы и модули

```text
Planning Быстрый Calculation API or Analysis API
Planning Input Projection Store
Calculation Input Profile Registry
Production Composition Analysis Adapter
Calculation Kernel / validation logic
Reporting/UI for “Анализ состава”

Product Definition
Process Definition
Configuration / Rules, if applicability conditions are configurable
Audit / Trace optional
```

#### Входные данные

```text
ProductCode or OrderRef or ConfigurationRef
Quantity
EffectiveDate
Configuration parameters
Order parameters
Required attributes
```

#### Ход выполнения

```text
User or module starts Composition Analysis
  ↓
Planning resolves ProductionCompositionAnalysisProfile
  ↓
ProductInput and ProcessInput are loaded from проекция входных данных планирования
  ↓
Fallback read contracts are used if allowed by profile
  ↓
Adapter applies BOM/version/process applicability rules
  ↓
Manufacturing composition tree is built
  ↓
Diagnostics are calculated
  ↓
CompositionAnalysisResult is returned to UI or consumer
```

#### Результат

```text
CompositionAnalysisResult
- composition tree
- selected BOM versions
- selected process/routing versions
- stages and operations, if expanded
- material norms
- technological norms
- applicability conditions used
- missing BOM/process/norms
- ambiguous version selection
- invalid or missing parameters
- diagnostics and warnings
```

#### Что не входит

```text
No concrete resource assignment.
No schedule.
No capacity check.
No promise date.
No dispatch.
```

#### Что меняется в операционных данных

Обычно ничего.

Результат может быть сохранён как validation/report, если нужно сравнение, аудит или последующая аналитика.

---

### 3.13 Проверка обеспеченности / ATP-like check

#### Назначение

Проверить обеспеченность спроса материалами, остатками, ожидаемыми поступлениями и уже зарезервированными/запланированными поставками.

#### Классификация

```text
ScenarioPurpose = FeasibilityCheck / Forecast
CalculationMode = БыстрыйCalculation or small async
Typical ProfileCode = MaterialAvailability / ATP
ResultOwnership = consumer module or transient Planning result
```

#### Ход выполнения

```text
Demand or operation requirement is provided
  ↓
Planning resolves MaterialAvailabilityProfile
  ↓
InventoryInput / ExpectedReceipts / Reservations are read
  ↓
Adapter builds availability input model
  ↓
Calculation Kernel calculates availability
  ↓
AvailabilityResult is returned
```

#### Результат

```text
AvailabilityResult
- available quantity
- shortage quantity
- available date
- expected receipts used
- reservations/conflicts
- material risks
```

---

### 3.14 Проверка реализуемости по мощностям / CTP-like check

#### Назначение

Проверить возможность выполнить спрос с учётом мощностей, календарей, текущей загрузки и ограничений.

#### Классификация

```text
ScenarioPurpose = FeasibilityCheck
CalculationMode = БыстрыйCalculation or small async
Typical ProfileCode = CapacityAvailability / CTP
ResultOwnership = consumer module or transient Planning result
```

#### Ход выполнения

```text
Demand / candidate order is provided
  ↓
Planning resolves CTP Profile
  ↓
ResourceInput / CalendarInput / CurrentLoad are read
  ↓
Adapter builds capacity check input model
  ↓
Calculation Kernel or small compute run checks capacity
  ↓
CapabilityResult is returned
```

#### Результат

```text
CapabilityResult
- can fit / cannot fit
- earliest capacity window
- bottleneck stages / resource groups
- capacity gaps
- possible alternatives / assumptions
```

---

### 3.15 Расчёт влияния изменения факта на план

#### Назначение

Определить, какие заказы, операции, материалы или ресурсы затронуты изменением факта.

Примеры:

- операция завершилась позже;
- оборудование ушло в простой;
- материал не поступил;
- партия заблокирована качеством;
- операция выполнена частично.

#### Классификация

```text
ScenarioPurpose = Forecast / Replan candidate detection
CalculationMode = БыстрыйCalculation or event-driven impact analysis
Typical ProfileCode = ImpactAnalysis
ResultOwnership = Planning & Scheduling / Analytics
```

#### Ход выполнения

```text
Actual event is published
  ↓
проекция входных данных планирования is updated
  ↓
Impact Analysis identifies affected plan objects
  ↓
Affected orders / operations / materials / resources are marked
  ↓
System may recommend Forecast or Replanning
```

#### Результат

```text
ImpactSummary
- affected orders
- affected operations
- affected materials
- affected resources
- estimated delay risk
- recommended action
```

---

## 4. Module interaction and input sources

Разделы ниже описывают не самостоятельные planning-сценарии, а источники данных, input contracts и взаимодействия модулей, которые используются сценариями из раздела 3.

Иначе говоря:

```text
Раздел 3 = что делает Planning & Scheduling как use-case.
Разделы 4+ = откуда Planning получает данные и куда передаёт результаты.
```

---

## 5. Product Definition inputs and interactions

### 5.1 Получение planning-readable данных по номенклатуре

Product Definition предоставляет Planning & Scheduling модель чтения по номенклатуре, используемой в расчётах.

Кратко:

```text
Item / Product attributes -> ProductInput
```

Используется для:

- разузлования;
- выбора технологии;
- расчёта потребностей;
- проверки применимости правил.

---

### 5.2 Получение состава изделия для разузлования

Planning или модуль-потребитель запрашивает состав изделия через planning profile.

Кратко:

```text
Item + Quantity -> BOM / components / requirements
```

Может использоваться в быстрый-сценарии Order Management.

---

### 5.3 Проверка применимости версии изделия / состава

Проверка, какая версия изделия, состава или атрибутов должна использоваться для расчёта.

Кратко:

```text
Product + date + attributes -> applicable product version
```

---

### 5.4 Обновление проекция входных данных планирования при изменении изделия

Product Definition публикует событие или предоставляет модель чтения для обновления проекция входных данных планирования.

Кратко:

```text
ProductChanged / BOMChanged -> Planning ProductInput refresh
```

---

## 6. Process Definition inputs and interactions

### 6.1 Получение planning-readable технологии изготовления

Process Definition предоставляет технологию в форме, пригодной для Planning & Scheduling.

Кратко:

```text
Process -> stages + operations + material norms + resource norms
```

Используется для:

- объёмно-календарного планирования;
- пооперационного расписания;
- разузлования;
- оценки сроков.

---

### 6.2 Интерпретация технологии для объёмно-календарного планирования

Adapter берёт уровень производственных переделов и укрупнённых норм.

Кратко:

```text
Stages -> CapacityPlanningSnapshot
```

Используется для планирования на уровне цехов, переделов, групп ресурсов.

---

### 6.3 Интерпретация технологии для пооперационного расписания

Adapter раскрывает операции внутри выбранного производственного передела.

Кратко:

```text
Stage + operations -> OperationSchedulingSnapshot
```

Используется для детального расписания на уровне операций, рабочих центров и ресурсов.

---

### 6.4 Разузлование маршрута / построение operation graph

Построение графа операций и зависимостей по маршруту.

Кратко:

```text
Route -> operation graph -> dependencies
```

Может использоваться как быстрый calculation.

---

### 6.5 Обновление projection при изменении маршрута или норм

Process Definition публикует события изменения технологии, маршрута, операций, норм ресурсов или материалов.

Кратко:

```text
RouteChanged / NormChanged -> Planning ProcessInput refresh
```

---

## 7. Resource Management inputs and interactions

### 7.1 Получение planning-readable данных по ресурсам

Resource Management предоставляет рабочие центры, оборудование, группы ресурсов, доступность и производительность в форме planning модель чтения.

Кратко:

```text
Resources / groups / capacity -> ResourceInput
```

---

### 7.2 Получение календарей и доступности ресурсов

Planning получает календари, смены, доступность ресурсов и исключения.

Кратко:

```text
Resource calendars -> CalendarInput / ResourceAvailability
```

Используется для scheduling, CTP, replanning и оценки сроков.

---

### 7.3 Проверка доступности ресурса для операции

Быстрый или planning calculation проверяет, может ли операция быть выполнена на конкретном ресурсе или группе ресурсов.

Кратко:

```text
Operation requirement + resource eligibility -> available resources
```

---

### 7.4 Реакция на изменение доступности ресурса

Изменение доступности оборудования, календаря или capacity обновляет проекция входных данных планирования и может инициировать forecast/replanning.

Кратко:

```text
ResourceAvailabilityChanged -> impacted schedules / replanning candidate
```

---

### 7.5 Расчёт загрузки ресурсов

Расчёт плановой или прогнозной загрузки ресурсов по периодам.

Кратко:

```text
Planned assignments -> ResourceLoadBuckets
```

Используется в Planning UI, Analytics и comparison of variants.

---

## 8. Plant Structure inputs and interactions

### 8.1 Получение структуры завода для planning область расчета

Plant Structure предоставляет завод, цехи, участки, линии и места выполнения операций.

Кратко:

```text
Plant / workshop / area hierarchy -> Planning область расчета
```

---

### 8.2 Ограничение расчёта выбранным область расчета

Planning profile использует структуру завода для отбора объектов расчёта.

Кратко:

```text
Plant / workshop / resource group -> область расчета filter
```

---

### 8.3 Изменение структуры завода и влияние на planning projection

Изменение цехов, участков, линий или привязки ресурсов требует обновления проекция входных данных планирования.

Кратко:

```text
PlantStructureChanged -> Planning область расчета/resource mapping refresh
```

---

## 9. Order Management interactions

### 9.1 Передача спроса в planning input

Order Management предоставляет заказы, потребности, даты, приоритеты, статусы и ограничения.

Кратко:

```text
Orders / demand lines -> OrderInput
```

---

### 9.2 Разузлование заказа при создании производственного заказа

Order Management вызывает быстрый calculation для получения дочерних заказов, операций или потребностей.

Кратко:

```text
Order -> OrderExplosionProfile -> child orders / operations / requirements
```

Результат может сохраняться Order Management в своих aggregates.

---

### 9.3 Оценка сроков заказа

Order Management вызывает Planning для предварительной оценки сроков выполнения заказа.

Кратко:

```text
Order -> estimate dates -> promised / expected dates
```

---

### 9.4 Проверка нового заказа перед принятием

Order Management передаёт candidate demand для feasibility / promise check.

Кратко:

```text
CandidateDemand -> FeasibilityResult
```

---

### 9.5 Получение плановых сроков после публикации плана

После публикации плана Order Management получает плановые сроки и возможные изменения по заказам.

Кратко:

```text
PlanPublished -> update planned dates / order planning status
```

---

### 9.6 Реакция на изменение заказа

Изменение количества, даты, приоритета или статуса заказа обновляет проекция входных данных планирования и может инициировать forecast/replanning.

Кратко:

```text
OrderChanged -> Planning OrderInput refresh -> impact analysis
```

---

## 10. Production Logistics interactions

### 10.1 Передача остатков и поступлений в planning input

Production Logistics предоставляет остатки, ожидаемые поступления, перемещения, резервы и доступность материалов.

Кратко:

```text
Inventory / receipts / reservations -> InventoryInput
```

---

### 10.2 Расчёт потребностей в материалах

Planning рассчитывает потребности по операциям, заказам, периодам или партиям.

Кратко:

```text
Planned operations -> MaterialRequirements
```

---

### 10.3 Проверка обеспеченности операции или заказа

Быстрый-сценарий проверки наличия материалов для операции, партии или заказа.

Кратко:

```text
Operation / order -> material availability -> shortages
```

---

### 10.4 Передача плановых потребностей после публикации плана

После публикации Logistics получает плановые потребности, даты обеспечения и приоритеты комплектации.

Кратко:

```text
PlanPublished -> material requirements / kit priorities
```

---

### 10.5 Реакция на дефицит или задержку поступления

Изменение ожидаемого поступления, дефицит или блокировка материала может инициировать impact analysis или replanning.

Кратко:

```text
MaterialShortage / ReceiptDelayed -> affected orders / replanning candidate
```

---

## 11. Shopfloor Execution interactions

### 11.1 Получение опубликованного расписания

Shopfloor Execution получает опубликованные операции, очереди, dispatch lists и приоритеты.

Кратко:

```text
PlanPublished -> operation queue / dispatch list
```

---

### 11.2 Передача факта выполнения в planning input

Shopfloor Execution публикует факт начала, завершения, приостановки и частичного выполнения операций.

Кратко:

```text
OperationStarted / Completed / Paused -> WipInput / Actuals
```

---

### 11.3 Оперативная оценка влияния отклонения

Planning оценивает, как задержка операции или изменение факта влияет на последующие операции и заказы.

Кратко:

```text
Actual deviation -> affected operations / expected delay
```

---

### 11.4 Запрос актуальной очереди операций

Shopfloor может запросить актуальный список доступных операций с учётом опубликованного плана и факта.

Кратко:

```text
Work center -> available operations / dispatch queue
```

---

### 11.5 Блокировка изменения уже начатых операций при replanning

При оперативном перепланировании операции в статусе InProgress или текущей смены становятся fixed constraints.

Кратко:

```text
InProgress operations -> fixed in replanning snapshot
```

---

## 12. Quality Management interactions

### 12.1 Учёт quality gates в планировании

Quality Management предоставляет контрольные операции, quality gates и ограничения, влияющие на расписание.

Кратко:

```text
Quality gates -> planning constraints
```

---

### 12.2 Блокировка партии по качеству

Блокировка партии или результата контроля обновляет planning input и может повлиять на сроки последующих операций.

Кратко:

```text
LotBlocked -> affected operations / material availability / replanning candidate
```

---

### 12.3 Планирование контрольных операций

Контрольные операции могут учитываться как часть маршрута или как отдельные planning constraints.

Кратко:

```text
Inspection operation -> schedule / capacity / delay impact
```

---

### 12.4 Разблокировка партии и пересчёт доступности

После снятия quality block Planning может обновить доступность материалов/партий и пересчитать ожидаемые сроки.

Кратко:

```text
LotReleased -> availability update -> forecast/replanning candidate
```

---

## 13. Equipment Integration / MDC interactions

### 13.1 Передача статуса оборудования

MDC передаёт состояние оборудования, простои, аварии и доступность.

Кратко:

```text
MachineStatusChanged -> ResourceAvailability input
```

---

### 13.2 Реакция на простой оборудования

Planning оценивает влияние простоя на расписание, сроки заказов и загрузку альтернативных ресурсов.

Кратко:

```text
Downtime -> affected assignments / delay risk / replanning candidate
```

---

### 13.3 Прогноз восстановления ресурса

Если известен ожидаемый срок восстановления оборудования, он может учитываться в оперативном прогнозе.

Кратко:

```text
Expected recovery time -> forecast schedule
```

---

### 13.4 Использование фактических machine events для анализа отклонений

Фактические события оборудования сравниваются с плановой загрузкой и базовый план.

Кратко:

```text
planned resource load vs actual machine state
```

---

## 14. Manufacturing Analytics interactions

### 14.1 Анализ отклонений план / факт

Сравнение PlanBaseline с фактом выполнения, выпуском, WIP, ресурсами и материалами.

Кратко:

```text
PlanBaseline vs Actuals -> deviations
```

---

### 14.2 Анализ загрузки ресурсов

Анализ плановой, прогнозной и фактической загрузки ресурсов.

Кратко:

```text
planned load / forecast load / actual load
```

---

### 14.3 Анализ сроков и рисков

Анализ ожидаемых опозданий, bottlenecks, critical orders и рисков невыполнения.

Кратко:

```text
ForecastResult -> delay risk dashboard
```

---

### 14.4 Анализ эффективности планирования

Сравнение вариантов, точности прогнозов, количества перепланирований, стабильности расписания и влияния отклонений.

Кратко:

```text
Planning history -> planning quality metrics
```

---

## 15. Configuration / Workflow / Rule interactions

### 15.1 Настройка planning profiles

Конфигурация или registry определяет доступные Calculation Input Profiles.

Кратко:

```text
ProfileCode -> required datasets / freshness / adapter
```

---

### 15.2 Настройка правил приоритезации

Rule Engine может использоваться для вычисления приоритетов спроса, заказов или операций.

Кратко:

```text
Demand context -> priority score
```

---

### 15.3 Настройка правил выбора ресурсов

Rule Engine / configuration может влиять на допустимость и предпочтительность ресурсов.

Кратко:

```text
Operation + resource context -> eligibility / preference
```

---

### 15.4 Workflow утверждения варианта плана

Workflow управляет жизненным циклом PlanningVariant.

Кратко:

```text
Calculated -> Reviewed -> Approved -> Published
```

---

### 15.5 Workflow фиксации базовый план

Workflow может управлять утверждением и фиксацией базовый план за период.

Кратко:

```text
Approved result -> Freeze базовый план -> Базовый план active
```

---

### 15.6 Workflow публикации оперативного плана

Workflow может требовать утверждения перед публикацией replanning result в Shopfloor / Logistics / Orders.

Кратко:

```text
ReplanningResult -> Approve -> Publish
```

---

## 16. Integration interactions

### 16.1 Получение заказов из ERP

Integration Service передаёт заказы, изменения заказов и потребности из ERP в DMP.

Кратко:

```text
ERP order -> Order Management -> Planning input
```

---

### 16.2 Передача опубликованного плана во внешние системы

После публикации плановые сроки, потребности и статусы могут передаваться в ERP или другие системы.

Кратко:

```text
PlanPublished -> external integration event/export
```

---

### 16.3 Интеграция с внешним APS / solver

Planning может использовать внешний solver как реализацию расчетный контур.

Кратко:

```text
Planning Snapshot -> External APS Adapter -> External Result -> PlanningResult
```

---

### 16.4 Импорт внешнего плана

Возможный будущий сценарий: загрузка рассчитанного внешнего плана как PlanningResult или candidate variant.

Кратко:

```text
External plan -> imported PlanningVariant -> validation -> optional publish
```

---

## 17. Open questions for scenario refinement

### 17.1 Граница между Order Management и Planning при разузловании

Нужно уточнить, какие результаты быстрый-разузлования сохраняет Order Management, а какие остаются transient.

### 17.2 Граница между Planning и Shopfloor Dispatching

Нужно уточнить, где заканчивается published schedule и где начинается оперативная dispatch-очередь Shopfloor.

### 17.3 Степень детализации PlanBaseline

Нужно решить, фиксировать ли базовый план только на уровне заказов/переделов или также на уровне операций, ресурсов и материалов.

### 17.4 Политики frozen zone

Нужно определить набор стандартных политик заморозки для replanning.

### 17.5 Feasibility Check: быстрый или small async

Нужно определить критерии, когда проверка нового заказа выполняется быстро быстрый, а когда должна переходить в полноценный "что если" scenario.

### 17.6 MVP-состав сценариев

Нужно выбрать минимальный набор сценариев для первого production slice Planning & Scheduling.
