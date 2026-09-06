# Архитектура модуля планирования и расписаний

Дата уточнения: 2026-07-10.

## 1. Назначение документа

Документ фиксирует целевое архитектурное решение для модуля **планирования и расписаний** (`Planning & Scheduling`) в Digital Manufacturing Platform.

Решение основано на варианте B:

```text
Модуль планирования и расписаний
+
расчетный контур расписаний
+
неизменяемые снимки входных данных планирования
+
асинхронные задания расчета
+
хранилище результатов планирования
+
управляемая публикация плана
```

Документ не описывает математические алгоритмы APS в деталях. Его задача — определить:

- место модуля планирования и расписаний в архитектуре DMP;
- границы ответственности модуля;
- взаимодействие с другими прикладными модулями;
- модель хранения данных планирования;
- принципы запуска и масштабирования расчётов;
- роль конфигурации, правил, событий и публикации результатов;
- базовый подход для первого промышленного объема.

---

## 2. Статус документа

Тип документа: **архитектурное решение прикладного модуля**.

Документ должен рассматриваться как дополнение к:

- `04_module_map.md` — уточняет модуль планирования и расписаний;
- `05_architecture.md` — уточняет место модуля планирования и расписаний в прикладном слое;
- `16_module_architecture_standard.md` — применяет стандарт прикладного модуля к планированию и расписаниям;
- `11_integration_event_platform.md` — использует API и события как способ взаимодействия между сервисами;
- `14_platform_runtime_conventions.md` — следует правилам владения данными, контекста tenant, идентификатора корреляции, аудита и изоляции.

Документ не заменяет существующие платформенные документы. Он является новым модульным архитектурным документом.

---

## 3. Ключевое архитектурное решение

Модуль планирования и расписаний реализуется как **прикладной модуль DMP**, но тяжелые расчеты расписания выносятся в отдельный **расчетный контур расписаний** (`Scheduling Compute Runtime`).

Короткая формула:

```text
Модуль планирования владеет сценариями и результатами планирования.
Расчетный контур выполняет тяжелые вычисления.
Другие модули взаимодействуют через стабильные API и события, а не через прямой доступ к базе данных.
```

Уточнение по Object Runtime:

```text
Object Runtime отвечает за стандартное чтение и изменение объектов.
Расчетный контур отвечает только за выполнение расчета.
```

Стандартные объекты Planning & Scheduling (`PlanningScenario`, `PlanningRun`,
`PlanningVariant`, `PlanningResult`, `PlanPublication`, `PlanBaseline`) должны
проектироваться как объекты Object Runtime там, где они доступны пользователю
как списки, карточки, действия, workflow state или configurable views.

Расчетный контур не заменяет Object Runtime. Он получает подготовленный
снимок данных, выполняет расчет и возвращает результат в модуль планирования. Если после
расчета меняется стандартный бизнес-объект, изменение проходит через
прикладной сервис и Object Runtime, а не напрямую из расчетного кода.

Целевое разделение:

```text
Модуль планирования и расписаний
  - сценарии планирования
  - варианты плана
  - задания расчёта
  - входные снимки данных
  - результаты расчёта
  - сравнение вариантов
  - утверждение и публикация плана

Расчетный контур расписаний
  - построение модели в памяти
  - выполнение алгоритмов планирования
  - параллельные расчёты
  - отчет о ходе и статусе расчета
  - возврат результатов в модуль планирования
```

---

## 4. Почему не встраивать APS как обычную часть Order Management

Planning & Scheduling не должен быть частью Order Management, Shopfloor Execution или Production Logistics.

Причины:

1. Планирование использует данные многих модулей, но не должно владеть ими.
2. Расчёт плана является тяжёлой вычислительной задачей и требует отдельного масштабирования.
3. Планирование работает с вариантами, сценариями и моделированием "что если", а не только с текущим состоянием заказов.
4. Результат расчета не всегда должен немедленно менять операционные данные.
5. Для воспроизводимости нужен неизменяемый снимок входной модели.
6. Для корпоративных сценариев нужны очереди, отмена расчета, лимиты, приоритеты, аудит и сравнение вариантов.

---

## 5. Основные компоненты решения

Раздел фиксирует основные архитектурные компоненты решения и их ответственность.

Важно: не каждый компонент обязан быть отдельным физическим сервисом в первом промышленном объеме. Часть компонентов может быть реализована как прикладной сервис, фоновый обработчик, инфраструктурный адаптер или компонент модели чтения внутри одного развертывания. Но их ответственность должна оставаться разделенной.

```text
Planning & Scheduling Module
  Planning API
  Planning Application Services
  Planning Domain Model
  Planning Persistence

  Planning Data Ingestion
  Planning Input Projection Store
  Planning Input Projection Rebuilder
  Source Watermark Manager

  Calculation Input Profiles
  Calculation Model Adapters
  Snapshot Cut Manager
  Calculation Input Builder / Snapshot Builder
  Snapshot Storage

  Result Store
  Result Projections
  Plan Publication Service
  Planning Read Models
  Integration/Event Handlers

Scheduling Compute Runtime
  Scheduling Orchestrator
  Calculation Queue Consumer
  Worker Pool
  Calculation Snapshot Loader
  In-Memory Model Builder
  Scheduling Engine Host
  Algorithm Plugins
  Progress Reporter
  Result Writer / Callback Adapter

Необязательные компоненты
  External APS / Solver Adapter
```

---

### 5.1 Planning API

Planning API — публичная API-граница модуля планирования и расписаний.

Через неё вызываются сценарии:

- создание и просмотр сценариев планирования;
- запуск расчёта;
- отмена расчёта;
- просмотр статуса запуска расчета;
- получение вариантов и результатов;
- утверждение варианта;
- публикация плана;
- быстрый расчет;
- получение статуса Planning Input Projection;
- административное перестроение проекции;
- получение source watermarks.

Planning API не должен содержать бизнес-логику расчета или прямой доступ к хранилищу. Он принимает запрос, проверяет техническую форму, передает управление в прикладной слой и возвращает DTO.

Для стандартных экранов Planning API должен опираться на Object Runtime
контракты и фасад выполнения. Специализированные расчетные точки API допустимы
для запуска/отмены расчета, прогресса и получения тяжелых результатов, но они
не должны становиться параллельным способом изменения стандартных объектов.

---

### 5.2 Planning Application Services

Planning Application Services исполняют прикладные сценарии модуля.

Примеры сценариев:

```text
CreateScenario
ResolvePlanningScope
ValidatePlanningInputFreshness
CreateSnapshotCut
BuildSnapshot
StartCalculation
CancelCalculation
CompareVariants
ApproveVariant
PublishPlan
FreezeBaseline
RunInlineCalculation
RebuildPlanningInputProjection
```

Прикладные сервисы отвечают за координацию:

- проверку контекста tenant и безопасности;
- вызов Workflow Engine и Rule Engine в утверждённых точках;
- выбор Calculation Input Profile;
- создание SnapshotCut;
- запуск Snapshot Builder;
- постановку расчёта в очередь;
- сохранение результата;
- публикацию событий;
- аудит и историю.

Прикладные сервисы не должны реализовывать тяжелые алгоритмы расписаний и не должны напрямую читать таблицы других прикладных модулей.

---

### 5.3 Planning Domain Model

Planning Domain Model содержит бизнес-сущности самого модуля планирования и расписаний.

Ключевые доменные и прикладные сущности:

```text
PlanningScenario
PlanningRun
PlanningSnapshot
PlanningVariant
PlanningResult
PlanPublication
PlanBaseline
PlanIssue
PlanKpi
PlannerChange / WhatIfChange
```

Пользовательские и управляемые жизненным циклом сущности из этого списка должны
иметь явное решение: стандартный объект Object Runtime, проекция только для чтения
или специализированный расчетный результат. По умолчанию сценарии, варианты,
публикации и базовые планы относятся к стандартным объектам Object Runtime.

Доменная модель отвечает за:

- состояние сценариев, запусков, вариантов и публикаций;
- локальные инварианты;
- жизненный цикл внутри границ модуля планирования;
- доменные события модуля планирования;
- согласованность собственных агрегатов.

Доменная модель не владеет основными данными других модулей: номенклатурой, маршрутами, ресурсами, заказами, остатками, незавершенным производством или фактом выполнения.

---

### 5.4 Хранение данных планирования

Слой хранения данных планирования отвечает за собственные данные модуля планирования и расписаний.

Он хранит:

- сценарии;
- запуски и задания расчета;
- метаданные снимков данных;
- варианты;
- метаданные результатов;
- записи базового плана;
- записи публикаций;
- проблемы и сводки KPI;
- технические состояния проекции, перестроения и отметок свежести, если они реализуются в той же базе модуля.

Слой хранения планирования не должен становиться общей базой производственных данных. Он хранит только данные, принадлежащие модулю планирования, и производные модели чтения / проекции, необходимые для прикладных сценариев планирования.

---

### 5.5 Прием входных данных планирования

Прием входных данных планирования получает события и обновления моделей чтения из других модулей и преобразует их в обновления проекции входных данных планирования (`Planning Input Projection`).

Источники:

```text
Order Management
Product Definition
Process Definition
Resource Management
Plant Structure
Production Logistics
Shopfloor Execution
Quality Management
Equipment Integration / MDC
Integration Service
```

Типовые входы:

- доменные и интеграционные события;
- события о необходимости обновления данных;
- уведомления о версии источника;
- ответы публичных API моделей чтения для планирования;
- пакетные задания обновления.

Прием входных данных не должен выполнять расчет планирования. Его задача — поддерживать проекцию данных для планирования в актуальном состоянии.

---

### 5.6 Хранилище проекции входных данных планирования

Хранилище проекции входных данных планирования хранит оптимизированное для чтения представление данных из модулей-источников.

Примерные области входных данных:

```text
ProductInput
ProcessInput
ResourceInput
CalendarInput
PlantStructureInput
OrderInput
InventoryInput
WipInput
QualityConstraintInput
EquipmentStateInput
```

Это не источник истины. Это производная проекция, используемая для быстрого построения снимков данных и входных моделей быстрого расчета.

Хранилище проекции ускоряет не сам алгоритм расписания, а подготовку входной модели расчета. Его задача — заранее поддерживать данные для планирования так, чтобы перед расчетом не выполнять повторное извлечение и преобразование данных из модулей-источников.

Хранилище проекции должно хранить только данные, значимые для планирования:

- стабильные коды и ссылки;
- версии;
- применимость;
- составы, маршруты, переделы, операции;
- нормы;
- календари;
- текущие статусы и доступность, если они нужны для расчёта;
- ссылки на источники и отметки свежести.

Он не должен хранить данные интерфейса, редакторские черновики, внутренние идентификаторы хранения чужих модулей и полную историю основных данных.

---

### 5.7 Перестроение проекции входных данных планирования

Перестроение проекции входных данных планирования заново собирает проекцию из модулей-источников при необходимости.

Используется для:

- первичной загрузки данных;
- восстановления проекции после сбоя;
- миграции версии схемы проекции;
- исправления расхождения между проекцией и данными источника;
- полного административного обновления;
- выборочного обновления по tenant, заводу, области расчета или модулю-источнику.

Rebuilder может работать через:

```text
публичные контракты чтения для планирования
batch export/import
повторное проигрывание событий, если оно поддерживается
административные задания перестроения
```

В первом промышленном объеме допустимо иметь простое перестроение по команде администратора без полноценного повторного проигрывания событий.

---

### 5.8 Source Watermark Manager

Source Watermark Manager отслеживает свежесть Planning Input Projection по каждому источнику.

Он хранит и проверяет:

```text
SourceModule
TenantId
PlantId optional
LastEventId
LastProcessedAtUtc
LastEventOccurredAtUtc
SourceVersion
ProjectionVersion
ProjectionLag
ProcessingStatus
```

Используется перед построением снимка данных или входной модели быстрого расчета.

Пример:

```text
OperationSchedulingProfile требует:
- Shopfloor/WIP lag <= 1 minute
- Resource calendar exact version
- Inventory lag <= 5 minutes
```

Если проекция недостаточно свежая, возможны варианты:

- запретить расчёт;
- предупредить пользователя;
- выполнить дополнительное чтение через публичные контракты;
- запустить обновление проекции;
- разрешить расчет с диагностической пометкой.

---

### 5.9 Профили входных данных расчета

Профиль входных данных расчета (`Calculation Input Profile`) описывает, какие данные нужны конкретному виду расчета и как их готовить.

Примеры профилей:

```text
ProductionCompositionAnalysisProfile
OrderExplosionProfile
CapacityPlanningProfile
OperationSchedulingProfile
MaterialAvailabilityProfile
FeasibilityCheckProfile
AtpCheckProfile
CtpCheckProfile
FullApsPlanningProfile
```

Профиль определяет:

- обязательные входные наборы данных;
- необязательные входные наборы данных;
- политика свежести данных;
- политика области расчета;
- требования к горизонту планирования;
- допустимое дополнительное чтение через публичные контракты;
- код адаптера;
- версия схемы снимка данных или тип временной входной модели;
- допустимый режим расчета.

Профиль — это не отдельная база данных и не отдельная проекция. Это контракт, реестр или конфигурация, которая говорит, как использовать каноническую проекцию входных данных планирования.

---

### 5.10 Адаптеры расчетной модели

Адаптер расчетной модели (`Calculation Model Adapter`) преобразует каноническую проекцию входных данных планирования в модель конкретного расчета.

Примеры:

```text
ProductionCompositionAnalysisAdapter
OrderExplosionInputAdapter
CapacityPlanningInputAdapter
OperationSchedulingInputAdapter
MaterialAvailabilityInputAdapter
FeasibilityInputAdapter
```

Adapter отвечает за:

- выбор нужных input areas;
- применение scope/horizon/filter;
- применение параметров заказа/конфигурации;
- интерпретацию исходных объектов для конкретного расчёта;
- агрегацию или детализацию данных;
- построение persisted PlanningSnapshot или transient input model;
- фиксацию source references, watermarks и input hash.

Adapter не должен выполнять тяжёлый scheduling algorithm. Он готовит входную модель для расчётного ядра.

---

### 5.11 Snapshot Cut Manager

Snapshot Cut Manager фиксирует логический срез входных данных, по которому будет строиться snapshot.

SnapshotCut отвечает на вопрос:

```text
По какому состоянию planning input projection выполняется расчёт?
```

Пример:

```text
SnapshotCut
- TenantId
- PlantId
- ProfileCode
- ScenarioId optional
- Scope
- HorizonStartUtc
- HorizonEndUtc
- ProjectionVersion
- SourceWatermarks
- CreatedAtUtc
- CreatedBy
- CorrelationId
```

Важно: SnapshotCut не обязан физически замораживать живую projection. Projection продолжает обновляться, а расчёт строится по зафиксированным source watermarks / projection version / input hash.

---

### 5.12 Calculation Input Builder / Snapshot Builder

Calculation Input Builder — общий компонент подготовки входной модели для расчёта.

Он имеет два основных режима:

```text
Calculation Input Builder
  -> persisted PlanningSnapshot for async planning runs
  -> transient input model for inline / fast calculations
```

Для будущей промышленной версии возможен третий режим:

```text
Calculation Input Builder
  -> delta / incremental input model for operational replanning
```

Snapshot Builder является частным случаем Calculation Input Builder и используется, когда нужен persisted immutable `PlanningSnapshot` для полноценного async planning run.

Типовой поток для async planning run:

```text
Scenario / Run request
  -> Calculation Input Profile
  -> SnapshotCut
  -> Calculation Model Adapter
  -> persisted PlanningSnapshot
  -> Snapshot Storage
```

Типовой поток для inline / fast calculation:

```text
Consumer request
  -> Calculation Input Profile
  -> lightweight SnapshotCut / trace context
  -> Calculation Model Adapter
  -> transient calculation input model
  -> Calculation Kernel
```

Calculation Input Builder не должен напрямую собирать все данные из source modules, если они уже есть в Planning Input Projection. При необходимости он может использовать pull fallback, но только если это разрешено profile policy.

Ключевое правило производительности:

```text
Snapshot/input build должен быть дешёвой упаковкой уже подготовленных planning-readable данных,
а не повторным ETL из source modules.
```

Сохраненный `PlanningSnapshot` обязателен не для каждого расчета. Он нужен для воспроизводимых асинхронных запусков планирования, вариантов "что если", расчетов, связанных с базовым планом, и случаев, где важны повторное выполнение и диагностика. Для быстрого расчета обычно достаточно временной входной модели и легкой трассировки.

### 5.13 Snapshot Storage

Snapshot Storage хранит неизменяемые входные пакеты расчета.

Может быть реализован как:

- relational tables;
- JSONB/documents;
- compressed binary package;
- object storage;
- гибридная модель.

Snapshot Storage должен обеспечивать:

- immutability;
- tenant isolation;
- source traceability;
- schema versioning;
- эффективную загрузку в расчетный контур;
- возможность диагностики и повторного выполнения расчета.

Для быстрого расчета сохраненный снимок данных обычно не нужен. Там может использоваться временная входная модель и легкая трассировка.

---

### 5.14 Result Store

Result Store хранит результаты расчетов модуля планирования и расписаний.

Он хранит:

- PlanningResult metadata;
- связь с scenario/run/variant/snapshot;
- summary KPI;
- diagnostics references;
- result status;
- engine version;
- ссылки на detailed result projections.

Result Store не должен быть только сырым JSON. Основные результаты должны быть доступны через проекции, пригодные для запросов.

---

### 5.15 Result Projections

Result Projections — оптимизированные для чтения представления результатов расчета.

Примеры:

```text
PlannedOperations
ResourceLoadBuckets
MaterialRequirements
PeggingLinks
PlanIssues
PlanKpi
GanttRows
DelayRisks
ShortageSummary
```

Они используются для:

- UI/Gantt;
- сравнения вариантов;
- dashboards;
- reporting;
- plan publication mapping;
- analytics.

Result Projections являются производными от PlanningResult и не должны редактироваться как самостоятельные операционные данные.

---

### 5.16 Plan Publication Service

Plan Publication Service отвечает за управляемую публикацию утвержденного планового результата в модули-потребители.

Он выполняет:

- проверку, что вариант утверждён;
- workflow/rule checks перед публикацией;
- построение publication package;
- mapping результата в contracts потребителей;
- отправку API commands или events;
- фиксацию PlanPublication;
- audit/history;
- обработку частичных ошибок публикации.

Публикация может затрагивать:

```text
Order Management
Shopfloor Execution
Production Logistics
Integration Service
Manufacturing Analytics
```

Plan Publication Service не должен напрямую изменять таблицы других модулей.

---

### 5.17 Planning Read Models

Planning Read Models — представления для чтения данных планирования в интерфейсе, аналитике и API.

Примеры:

```text
ScenarioList
RunStatus
VariantList
VariantComparison
PlanningResultSummary
GanttView
ResourceLoadView
PlanIssuesView
InputProjectionStatusView
SourceWatermarkView
```

Модели чтения оптимизированы под чтение и не являются источником бизнес-истины, если не являются частью доменного агрегата.

---

### 5.18 Integration/Event Handlers

Integration/Event Handlers обрабатывают входящие и исходящие события.

Входящие обработчики:

- принимают события модулей-источников;
- обновляют Planning Input Projection;
- обновляют отметки свежести;
- инициируют анализ влияния или кандидаты на прогноз, если это предусмотрено политикой.

Исходящие обработчики и издатели:

- публикуют события PlanningRun;
- публикуют события PlanPublished;
- публикуют события ProjectionRebuilt;
- публикуют события диагностики и проблем, если они нужны другим потребителям.

Все события должны использовать стандартный конверт события: TenantId, CorrelationId, EventTypeCode, PayloadVersion.

---

### 5.19 Расчетный контур расписаний

Scheduling Compute Runtime — вычислительный контур для тяжелых расчетов.

Он получает снимок данных под конкретный расчет и выполняет расчет, не зная внутренней модели модулей-источников и не читая Planning Input Projection напрямую.

Физически в первом промышленном объеме может быть реализован внутри того же backend / развертывания, но архитектурно это отдельная расчетная граница.

---

### 5.20 Scheduling Orchestrator

Scheduling Orchestrator управляет жизненным циклом расчета внутри расчетного контура.

Отвечает за:

- прием запроса на расчет;
- выбор движка или подключаемого модуля;
- распределение работы по обработчикам;
- контроль статуса;
- отмена;
- ход выполнения;
- обработку ошибок;
- возврат результата.

Он не строит снимок данных и не занимается публикацией плана.

---

### 5.21 Calculation Queue Consumer

Calculation Queue Consumer получает задания из очереди расчётов.

Отвечает за:

- чтение заданий из очереди;
- идемпотентность;
- проверку ссылок на tenant, запуск и снимок данных;
- передачу задания в Scheduling Orchestrator;
- повторные попытки или перевод в состояние ошибки.

Для первого промышленного объема очередь может быть простой внутренней очередью заданий. Для промышленной версии — отдельной инфраструктурой сообщений или заданий.

---

### 5.22 Worker Pool

Worker Pool выполняет расчётные задачи параллельно.

Уровни параллельности:

- несколько расчетов разных tenant или заводов;
- несколько вариантов одного сценария;
- независимые части одного расчёта;
- альтернативные эвристики или стратегии;
- задания внешнего решателя.

Worker Pool должен учитывать лимиты:

```text
maxConcurrentRunsPerTenant
maxConcurrentRunsPerPlant
maxMemoryPerRun
maxCpuSecondsPerRun
priorityPolicy
отменаPolicy
```

---

### 5.23 Calculation Snapshot Loader

Calculation Snapshot Loader загружает неизменяемый снимок данных из Snapshot Storage в расчетный контур.

Отвечает за:

- проверку snapshot schema version;
- tenant isolation;
- загрузку snapshot package;
- проверку input hash/source references;
- подготовку данных для In-Memory Model Builder.

Он не обращается к source modules и не обновляет Planning Input Projection.

---

### 5.24 In-Memory Model Builder

In-Memory Model Builder преобразует загруженный снимок данных в эффективную структуру в памяти для расчета.

Примеры структур:

```text
operation graph
resource calendars
capacity buckets
material availability graph
demand/supply links
constraints index
setup matrix
```

Это технический слой расчетного контура. Он оптимизирован под скорость расчета и не должен становиться моделью хранения или публичным контрактом.

---

### 5.25 Scheduling Engine Host

Scheduling Engine Host исполняет выбранный расчетный движок.

Он отвечает за:

- запуск алгоритма;
- передачу параметров;
- управление lifecycle engine execution;
- сбор diagnostics;
- обработку отмена;
- передачу результата в Result Writer.

Engine Host должен позволять подключать разные алгоритмы, но не должен превращаться в произвольный оптимизатор без кода.

---

### 5.26 Algorithm Plugins

Algorithm Plugins — реализации конкретных алгоритмов или стратегий расчета.

Примеры:

```text
HeuristicScheduler
CapacityPlanner
MaterialAvailabilityCalculator
OrderExplosionCalculator
FeasibilityChecker
ExternalSolverPlugin
```

Подключаемые алгоритмы должны работать с входной моделью под конкретный расчет или моделью в памяти, а не с таблицами прикладных модулей.

---

### 5.27 Progress Reporter

Progress Reporter публикует прогресс расчёта.

Примерные данные:

```text
RunId
Stage
Percent
Message
CurrentStep
WarningsCount
EstimatedRemaining optional
```

Используется интерфейсом, мониторингом, аудитом и трассировкой.

Прогресс не должен быть источником бизнес-истины. Это диагностика выполнения.

---

### 5.28 Result Writer / Callback Adapter

Result Writer / Callback Adapter возвращает результат из расчетного контура в модуль планирования.

Отвечает за:

- преобразование raw engine output в PlanningResult DTO/package;
- сохранение результата через Planning Result Store или callback API;
- фиксацию diagnostics;
- публикацию completion/failure status;
- обработку частичных результатов, если это поддерживается.

Он не публикует план в операционные модули. Публикацию выполняет только Plan Publication Service.

---

### 5.29 External APS / Solver Adapter

External APS / Solver Adapter — необязательный компонент для подключения внешнего APS, решателя или оптимизационного движка.

Он отвечает за:

- mapping DMP snapshot в external solver input;
- запуск external calculation;
- polling/callback статуса;
- mapping external result обратно в PlanningResult;
- обработку ошибок;
- version compatibility;
- diagnostics.

Внешний APS является деталью реализации расчетного контура. Другие модули DMP не должны зависеть от его модели данных.

---

## 6. Логическая схема взаимодействия

```text
Order / Product / Process / Resource / Inventory / Shopfloor
        ↓ events / planning read contracts
Planning Data Ingestion
        ↓
Planning Input Projection
        ↓ profile + adapter + snapshot cut
Planning Snapshot Builder
        ↓ immutable snapshot
Scheduling Compute Runtime
        ↓ calculation result
Planning Result Store
        ↓ planner review / compare / approve
Plan Publication Service
        ↓ API / events / commands
Order Management / Shopfloor / Logistics / Integration
```

Главный принцип:

```text
Planning consumes public contracts.
Planning does not read foreign tables.
Planning publishes approved decisions explicitly.
```

---

## 7. Границы ответственности Planning & Scheduling

### 7.1 Planning & Scheduling владеет

Planning & Scheduling module владеет следующими сущностями и процессами:

- `PlanningScenario` — сценарий планирования;
- `PlanningRun` / `CalculationJob` — запуск расчёта;
- `PlanningSnapshot` — immutable входная модель расчёта;
- `SnapshotCut` — зафиксированный срез planning input projection для построения snapshot;
- `SourceWatermark` — техническая фиксация свежести source projections;
- `PlanningInputProjection` — read-optimized projection для planning input, не являющаяся источником истины;
- `PlanningVariant` — вариант плана;
- `PlanningResult` — результат расчёта;
- `PlanKpi` — показатели варианта;
- `PlanIssue` — ошибки, дефициты, конфликты и предупреждения;
- `PlanPublication` — публикация утверждённого варианта;
- `PlannerChange` / `WhatIfChange` — изменения для моделирования;
- read models для Gantt, загрузки ресурсов, дефицитов, pegging и сравнения вариантов;
- регистрацию `CalculationInputProfile` и `CalculationModelAdapter` как stable calculation contracts.

### 7.2 Planning & Scheduling не владеет

Planning & Scheduling не должен владеть исходными master/operational сущностями других модулей:

- номенклатурой;
- BOM / составами изделий;
- технологическими маршрутами;
- рабочими центрами и оборудованием как master data;
- производственными заказами как operational object;
- складскими остатками как источником истины;
- WIP и фактом выполнения;
- качественными результатами и блокировками партий.

Эти данные принадлежат соответствующим domain modules.

Planning Input Projection не меняет ownership. Она является производной read model для планирования и не должна использоваться другими модулями как новый master data source.

---

## 8. Взаимодействие с другими модулями

### 8.1 Order Management

Planning получает:

- спрос;
- производственные заказы;
- даты потребности;
- приоритеты;
- ограничения заказов;
- текущие статусы.

Planning возвращает:

- плановые сроки;
- proposed changes по заказам;
- рекомендации по изменению дат/приоритетов;
- опубликованный план после утверждения.

### 8.2 Product Definition

Planning получает:

- номенклатуру;
- продуктовые атрибуты;
- единицы измерения;
- группы/классификаторы;
- product-specific planning parameters, если они принадлежат Product Definition.

### 8.3 Process Definition

Planning получает:

- маршруты;
- операции;
- последовательности;
- нормы ресурсов;
- нормы материалов;
- версии процессов;
- условия применимости процессов.

### 8.4 Resource Management

Planning получает:

- рабочие центры;
- оборудование;
- группы ресурсов;
- производительность;
- доступность;
- resource calendars;
- ограничения мощности.

### 8.5 Plant Structure

Planning получает:

- структуру завода;
- цехи;
- участки;
- линии;
- места выполнения операций;
- связи ресурсов с организационно-производственной структурой.

### 8.6 Production Logistics / Inventory

Planning получает:

- остатки;
- ожидаемые поступления;
- доступность партий;
- логистические ограничения;
- данные комплектации;
- material availability.

Planning возвращает:

- плановые потребности;
- даты обеспечения;
- дефициты;
- рекомендации по комплектации или перемещениям.

### 8.7 Shopfloor Execution

Planning получает:

- факт выполнения;
- WIP;
- статусы операций;
- отклонения;
- блокировки выполнения;
- фактическую доступность операций.

Planning возвращает:

- опубликованное расписание;
- dispatch list;
- приоритеты выполнения;
- плановые даты операций.

### 8.8 Quality Management

Planning получает:

- контрольные операции;
- quality gates;
- блокировки партий;
- ограничения по качеству;
- дополнительные операции контроля, если они влияют на расписание.

### 8.9 Integration Service

Planning взаимодействует с Integration Service для:

- получения внешних заказов/изменений из ERP;
- публикации утверждённых планов во внешние системы;
- интеграции с внешним APS или solver;
- обмена status/ход выполнения events.

---

## 9. Основная доменная модель Planning & Scheduling

Важно различать domain/application entities и planning infrastructure/read models.

Domain/application entities:

```text
PlanningScenario
PlanningRun
PlanningSnapshot
PlanningVariant
PlanningResult
PlanPublication
PlanIssue
PlanKpi
```

Infrastructure/read-model concepts:

```text
PlanningInputProjection
SourceWatermark
SnapshotCut
ProjectionRebuildState
```

Configuration/registry concepts:

```text
CalculationInputProfile
CalculationMode
SnapshotSchemaVersion
EngineCode
```

Service concepts:

```text
CalculationModelAdapter
PlanningInputIngestionHandler
PlanningSnapshotBuilder
```

Эти группы не должны смешиваться в одну доменную иерархию. Planning Input Projection нужна для быстрого построения snapshot, но не является aggregate root бизнес-домена Planning & Scheduling.

### 9.1 PlanningScenario

Сценарий планирования.

Примерные поля:

```text
PlanningScenario
- Id
- TenantId
- PlantId
- Code
- Name
- PlanningMode
- HorizonStartUtc
- HorizonEndUtc
- FrozenPeriodEndUtc
- ScopeDefinition
- ParameterSetCode
- Status
- CreatedAtUtc
- CreatedBy
```

Назначение:

- определяет границы планирования;
- задаёт режим расчёта;
- хранит параметры и ограничения сценария;
- является контейнером для запусков и вариантов.

### 9.2 PlanningRun / CalculationJob

Запуск расчёта.

```text
PlanningRun
- Id
- TenantId
- PlantId
- ScenarioId
- SnapshotId
- RequestedBy
- RequestedAtUtc
- Priority
- Status
- StartedAtUtc
- FinishedAtUtc
- EngineCode
- EngineVersion
- CorrelationId
- ErrorCode
- ErrorMessage
```

Статусы:

```text
Queued
PreparingSnapshot
Running
Completed
Failed
Cancelled
Expired
```

### 9.3 PlanningSnapshot

Immutable входная модель расчёта.

```text
PlanningSnapshot
- Id
- TenantId
- PlantId
- ScenarioId
- HorizonStartUtc
- HorizonEndUtc
- SourceVersionRefs
- InputHash
- CreatedAtUtc
- CreatedBy
- Status
- StorageRef
```

Snapshot нужен для:

- воспроизводимости результата;
- диагностики;
- сравнения расчётов;
- изоляции тяжёлого расчёта от изменений live-данных;
- возможности повторного расчёта на тех же входных данных.

### 9.4 PlanningVariant

Вариант плана.

```text
PlanningVariant
- Id
- TenantId
- ScenarioId
- RunId
- SnapshotId
- Code
- Name
- VariantParameters
- Status
- SummaryKpi
- CreatedAtUtc
```

Вариант может отличаться:

- параметрами планирования;
- правилами приоритезации;
- изменениями модели "что если";
- настройками оптимизации;
- горизонтом или областью расчета;
- стратегией алгоритма.

### 9.5 PlanningResult

Результат расчёта.

```text
PlanningResult
- Id
- TenantId
- VariantId
- RunId
- SnapshotId
- Status
- EngineCode
- EngineVersion
- StartedAtUtc
- FinishedAtUtc
- SummaryKpi
- DiagnosticsRef
```

Результат должен включать модель чтения, пригодную для запросов:

- плановые заказы;
- плановые операции;
- назначения ресурсов;
- потребности в материалах;
- связи потребностей и источников покрытия;
- интервалы загрузки ресурсов;
- проблемы плана;
- KPIs.

### 9.6 PlanPublication

Публикация выбранного варианта.

```text
PlanPublication
- Id
- TenantId
- PlantId
- ScenarioId
- VariantId
- PublishedBy
- PublishedAtUtc
- Status
- PublicationMode
- CorrelationId
```

Публикация является отдельным прикладным сценарием. Расчет варианта сам по себе не должен автоматически менять операционные данные.

---

## 10. Модель снимка входных данных

Planning Snapshot — это не копия БД других модулей. Это нормализованная входная модель для расчета.

Внутри снимка данных могут быть следующие логические наборы:

```text
SnapshotOrders
SnapshotOperations
SnapshotProducts
SnapshotProcesses
SnapshotResources
SnapshotResourceGroups
SnapshotCalendars
SnapshotLocations
SnapshotMaterials
SnapshotInventory
SnapshotConstraints
SnapshotSetupRules
SnapshotPeggingRules
SnapshotParameters
SnapshotWhatIfChanges
```

### 10.1 Требования к снимку данных

Снимок данных должен быть:

- неизменяемым;
- ограниченным одним tenant;
- связанным с scenario/run;
- версионированным через source version refs;
- воспроизводимым;
- пригодным для передачи в расчетный контур;
- достаточно компактным для in-memory расчёта;
- пригодным для диагностики.

### 10.2 SourceVersionRefs

Snapshot должен хранить ссылки на версии исходных данных или read contracts, из которых он был построен.

Пример:

```text
SourceVersionRefs
- OrderManagement.ReadModelVersion
- ProductDefinition.Version
- ProcessDefinition.Version
- ResourceManagement.Version
- InventorySnapshotTimestamp
- ShopfloorStateTimestamp
- ConfigurationPublishedVersion
```

Это не обязательно должны быть физические FK. Это может быть набор стабильных references для traceability.

---

## 11. Planning Input Projection and profile-based snapshot building

Snapshot Builder не должен каждый раз напрямую собирать все данные из модулей-владельцев и не должен иметь отдельную толстую projection под каждый вид расчёта.

Целевая модель должна разделять четыре слоя:

```text
Source Domain Data
  данные в модулях-владельцах

Planning Input Projection
  живая planning-readable проекция, обновляемая events/API

Calculation Input Profile + Model Adapter
  выбор, фильтрация и интерпретация данных под конкретный вид расчёта

Planning Snapshot
  immutable входная модель конкретного расчёта
```

Ключевая формула:

```text
Projection = что известно для планирования.
Profile = что нужно для данного вида расчёта.
Adapter = как интерпретировать данные.
Snapshot / transient input model = что именно считаем.
```

Важно: persisted `PlanningSnapshot` — не единственный возможный результат подготовки входа. Для inline / fast calculation может использоваться transient input model. Для больших async planning runs используется persisted immutable snapshot.

---

### 11.1 Planning Input Projection

Planning Input Projection — это read-optimized, planning-readable представление данных, которые принадлежат другим модулям.

Она не является новым источником истины и не должна превращаться в копию всех таблиц Product / Process / Order / Resource / Inventory.

Источник истины остаётся в доменных модулях:

```text
Process Definition owns process.
Planning Input Projection stores process planning read model.
Snapshot stores calculation-specific interpretation.
```

Planning Input Projection может включать несколько canonical input areas по source-доменам:

```text
PlanningInputStore
  ProductInput
  ProcessInput
  ResourceInput
  CalendarInput
  OrderInput
  InventoryInput
  WipInput
  ConstraintInput
```

Эти структуры должны содержать только planning-relevant данные:

- stable codes / refs;
- версии;
- применимость;
- стадии / переделы;
- операции;
- нормы ресурсов;
- нормы материалов;
- календари;
- ограничения;
- статусы и доступность, если они нужны для расчётов.

Они не должны содержать:

- UI-описания;
- служебные поля редакторов;
- историю изменения master data;
- внутренние persistence IDs чужих модулей;
- draft-конфигурации;
- технические поля, не влияющие на planning.

---

### 11.2 Projection update model

Planning Input Projection может обновляться гибридно:

```text
Domain modules publish events
  -> Planning ingestion handlers update Planning Input Projection

For complex structures:
  event may contain identity/version only
  -> Planning calls public planning read model API
  -> Projection is refreshed
```

Рекомендуемый подход:

- частые простые изменения можно передавать в событиях с достаточным payload;
- сложные структуры лучше обновлять через invalidation event + read model API.

Примеры простых event payload:

```text
Order dates/status/quantity changed
Inventory balance delta changed
Operation actual status changed
```

Примеры invalidation/version events:

```text
BOM changed
Route changed
Resource calendar changed
Setup rules changed
```

Модули-владельцы не должны знать, какие именно projection нужны Planning.

Правильная зависимость:

```text
Domain module publishes stable events / read contracts.
Planning subscribes and builds its own projection.
```

Анти-паттерн:

```text
Order Management sends special payload directly for Planning internal tables.
```

---

### 11.3 Calculation Input Profile

Calculation Input Profile описывает, какие данные нужны конкретному виду расчёта и какие требования предъявляются к их свежести, горизонту и scope.

Примерная структура:

```text
CalculationInputProfile
- Code
- CalculationMode
- RequiredDatasets
- OptionalDatasets
- FreshnessPolicy
- ScopePolicy
- TransformationAdapter
- SnapshotSchemaVersion
```

Примеры профилей:

```text
OrderExplosionProfile
CapacityPlanningProfile
OperationSchedulingProfile
MaterialAvailabilityProfile
AtpCheckProfile
FullApsPlanningProfile
```

Профиль не должен быть отдельной физической копией данных.

Профиль — это:

```text
selection + transformation + interpretation rules
```

---

### 11.4 Calculation Model Adapter

Calculation Model Adapter преобразует canonical Planning Input Projection в snapshot конкретной расчётной модели.

Примеры adapter-ов:

```text
OrderExplosionInputAdapter
CapacityPlanningInputAdapter
OperationSchedulingInputAdapter
AtpInputAdapter
```

Adapter отвечает за:

- выбор нужных input areas;
- применение scope/horizon;
- интерпретацию исходных объектов;
- агрегацию или детализацию данных;
- построение calculation-specific snapshot schema;
- фиксацию source references и watermarks.

Анти-паттерн:

```text
ProjectionForOCP
ProjectionForOperationScheduling
ProjectionForExplosion
ProjectionForATP
```

Правильная модель:

```text
Canonical Planning Input Projection
  -> CapacityPlanningInputAdapter
  -> CapacityPlanningSnapshot

Canonical Planning Input Projection
  -> OperationSchedulingInputAdapter
  -> OperationSchedulingSnapshot

Canonical Planning Input Projection
  -> OrderExplosionInputAdapter
  -> OrderExplosionSnapshot
```

---

### 11.5 Пример: двухуровневая технология изготовления

Process Definition может описывать технологию в двух уровнях:

```text
ManufacturingProcess
  ProductionStage 10: Заготовка
    Operation 10.1
    Operation 10.2

  ProductionStage 20: Мехобработка
    Operation 20.1
    Operation 20.2
    Operation 20.3

  ProductionStage 30: Сборка
    Operation 30.1
    Operation 30.2
```

В Planning Input Projection это может быть представлено как canonical planning read model:

```text
ProcessInput
- ProcessId / ProcessCode
- ProductCode
- Version
- ApplicabilityConditions
- Stages[]
    StageCode
    DepartmentCode
    Sequence
    DurationNorm
    ResourceGroupNorms[]
    MaterialNeedSummary[]
    Operations[]
        OperationCode
        Sequence
        WorkCenterEligibility
        ResourceNorms
        MaterialNorms
        SetupParameters
```

Для объёмно-календарного планирования adapter берёт уровень переделов:

```text
CapacityPlanningSnapshot
  SnapshotDemand = sales order / production order demand
  SnapshotStage = production stage / department / duration / resource group capacity
```

Операции внутри передела могут не попадать в snapshot.

Для пооперационного расписания adapter раскрывает операции выбранного передела:

```text
OperationSchedulingSnapshot
  SnapshotDemand = production batch at stage
  SnapshotOperation = technological operation / sequence / eligible resources / setup / processing time
```

В этом режиме расчётным “заказом” может быть не исходный заказ из Order Management, а производственная партия на конкретном производственном переделе.

Для разузлования adapter строит дерево потребностей:

```text
OrderExplosionSnapshot
  ExplosionNode = item / quantity / BOM line / process ref / material requirement
```

---

### 11.6 PlanningDemand вместо единого понятия Order

В разных видах расчёта понятие “заказ” может отличаться.

Поэтому в calculation-specific snapshot следует использовать более нейтральное понятие:

```text
PlanningDemand
```

Примеры интерпретации:

| Вид расчёта | PlanningDemand |
|---|---|
| Объёмно-календарное планирование | sales order / production order demand |
| Пооперационное расписание | production batch at stage |
| ATP / проверка обеспеченности | customer demand line |
| Разузлование | item quantity requirement |

`Order` из Order Management является source object.

`PlanningDemand` является расчётным представлением потребности внутри конкретного snapshot.

---

### 11.7 Freshness, watermarks and snapshot cut

Если Planning Input Projection обновляется событиями, Snapshot Builder должен понимать, насколько она свежая.

Для каждого источника данных Planning должен хранить source watermark:

```text
SourceWatermark
- SourceModule
- TenantId
- LastEventId
- LastEventOccurredAtUtc
- LastProcessedAtUtc
- SourceVersion
- ProjectionLag
```

Перед построением snapshot Builder проверяет freshness policy профиля.

Примеры:

```text
OrderExplosionProfile
  Product/Process data must match exact source version
  Inventory is not required

OperationSchedulingProfile
  Shopfloor/WIP max lag: 1 minute
  Resource calendars exact version
  Inventory max lag: 5 minutes

FullApsPlanningProfile
  ERP orders max lag: 15 minutes
  Master data exact published version
```

Важно: физически замораживать живую projection не нужно.

Замораживается cut:

```text
SnapshotCut
- ProfileCode
- Scope
- HorizonStartUtc
- HorizonEndUtc
- ProjectionVersion
- SourceWatermarks
- CreatedAtUtc
```

Planning Input Projection продолжает обновляться, а конкретный snapshot строится по зафиксированному cut.

---

### 11.8 MVP and industrial maturity

Первый промышленный объем может использовать упрощенную реализацию этой модели, не меняя архитектурных границ.

Для MVP допустимо:

- часть данных загружать pull-on-demand;
- держать минимальную Planning Input Projection только для наиболее изменчивых данных;
- поддержать 1–2 Calculation Input Profile;
- иметь простые watermarks;
- выполнять projection rebuild административной командой;
- не реализовывать полный replay событий на старте.

Промышленная версия добавляет:

- полноценный Planning Input Store;
- event-driven projections по всем критичным источникам;
- source watermarks;
- replay/rebuild projection;
- schema versioning;
- multiple input profiles;
- profile-specific freshness policy;
- partial/incremental snapshot build;
- diagnostics по источникам данных;
- distributed snapshot building for large scopes.

Архитектура при этом остаётся той же.

---

## 12. Result model

Результат не должен храниться только как непрозрачный JSON blob.

Допустима гибридная модель:

1. Queryable tables / projections для основных сущностей результата.
2. JSONB / blob storage для больших engine diagnostics или raw output.
3. Analytical projections для Gantt, загрузки и KPI.

### 12.1 Основные result entities

```text
PlannedOrder
- Id
- ResultId
- SourceOrderRef
- ItemCode
- Quantity
- PlannedStartUtc
- PlannedFinishUtc
- Delay
- Status

PlannedOperation
- Id
- ResultId
- SourceOperationRef
- SourceOrderRef
- OperationCode
- SequenceNo
- PlannedStartUtc
- PlannedFinishUtc
- Duration
- Status

ResourceAssignment
- Id
- ResultId
- PlannedOperationId
- ResourceCode
- ResourceGroupCode
- StartUtc
- FinishUtc
- Quantity
- SetupDuration
- ProcessingDuration

MaterialRequirement
- Id
- ResultId
- PlannedOperationId
- ItemCode
- RequiredQuantity
- RequiredAtUtc
- AvailabilityStatus
- ShortageQuantity

PeggingLink
- Id
- ResultId
- DemandRef
- SupplyRef
- ItemCode
- Quantity
- LinkType

ResourceLoadBucket
- Id
- ResultId
- ResourceCode
- BucketStartUtc
- BucketEndUtc
- Capacity
- Load
- UtilizationPercent

PlanIssue
- Id
- ResultId
- Severity
- IssueCode
- ObjectRef
- Message
- Details
```

---

## 13. Calculation lifecycle

Типовой lifecycle расчёта:

```text
Create Scenario
  ↓
Select Calculation Input Profile
  ↓
Resolve planning scope / horizon / parameters
  ↓
Validate input freshness policy
  ↓
Create Snapshot Cut
  ↓
Build Snapshot from Planning Input Projection through Model Adapter
  ↓
Start Calculation Run
  ↓
Queue Job
  ↓
Расчетный контур выполняет расчет по снимку данных под конкретный расчет
  ↓
Store Result
  ↓
Compare Variants
  ↓
Approve Variant
  ↓
Publish Plan
  ↓
Apply / notify consuming modules
```

Для inline/fast calculation lifecycle может быть сокращён, но он всё равно должен использовать тот же принцип: selected profile → adapter → calculation-specific input model.

### 13.1 Расчёт не равен публикации

Важно:

```text
Calculated plan = proposal.
Published plan = accepted operational decision.
```

До публикации результат должен быть доступен для анализа, но не обязан менять:

- производственные заказы;
- очереди операций;
- dispatch lists;
- потребности в логистике;
- данные ERP.

---

## 14. Calculation modes

Planning & Scheduling должен поддерживать несколько режимов расчёта поверх одной архитектурной foundation.

Это важно, потому что не каждый расчёт является полноценным APS-сценарием с вариантами, сравнением и публикацией плана. В некоторых случаях другому модулю нужно быстро получить расчётный результат и сохранить его в своих объектах.

Ключевой принцип:

```text
One calculation foundation.
Different orchestration modes.
Different ownership of persisted result.
```

---

### 14.1 Inline / Fast Calculation

Inline / Fast Calculation используется для быстрых синхронных или near-synchronous расчётов.

Типовые сценарии:

- разузловать один заказ;
- разузловать несколько заказов;
- получить состав изделия;
- построить цепочку операций по маршруту;
- рассчитать потребности в материалах;
- предварительно оценить сроки;
- проверить обеспеченность;
- оценить доступность ресурсов;
- подготовить результат для сохранения в объекте модуля-потребителя.

---

#### 14.1.1 Где начинается Inline / Fast Calculation

Inline / Fast Calculation начинается не с `PlanningScenario`, а с use-case модуля-потребителя.

Примеры:

```text
Order Management
  хочет разузловать заказ при создании производственного заказа
```

```text
Production Logistics
  хочет быстро получить потребности в материалах для комплектации
```

```text
Shopfloor Execution
  хочет оценить доступность операции перед постановкой в очередь выполнения
```

Логический старт:

```text
Consumer Module Use Case
  -> Planning Inline Calculation API
  -> CalculationMode = InlineCalculation
  -> ProfileCode = OrderExplosion / MaterialAvailability / EstimateDates
```

Пример входного запроса:

```text
InlineCalculationRequest
- TenantId
- PlantId
- ConsumerModule
- ConsumerUseCase
- ProfileCode
- ObjectRefs
- Scope
- Parameters
- RequiredFreshness
- CorrelationId
```

Пример:

```text
ProfileCode = Planning.Profile.OrderExplosion
ObjectRef = OrderManagement.Order:123
Mode = InlineCalculation
```

---

#### 14.1.2 Как Inline / Fast Calculation использует раздел 11

Inline / Fast Calculation не должен обходить модель `Planning Input Projection -> Profile -> Adapter -> Snapshot/Input Model`.

Но он использует эту модель в облегчённом виде.

```text
Consumer request
  -> Calculation Input Profile
  -> Planning Input Projection / read fallback
  -> Calculation Model Adapter
  -> transient calculation input model
  -> Calculation Kernel
  -> result DTO
  -> Consumer module
```

Использование элементов раздела 11:

| Элемент | Нужен ли для Inline / Fast Calculation | Комментарий |
|---|---|---|
| Planning Input Projection | Да, как предпочтительный быстрый источник данных | Если projection содержит нужные данные и проходит freshness policy |
| Calculation Input Profile | Да, обязательно | Определяет, какие данные нужны и какие fallback допустимы |
| Calculation Model Adapter | Да, обязательно | Преобразует canonical input в расчётную модель |
| SourceWatermark / freshness | Да, в упрощённом виде | Нужно понимать, насколько свежие данные использованы |
| SnapshotCut | Да, но lightweight | Фиксирует source refs / watermarks / input hash для traceability |
| Persisted PlanningSnapshot | Не всегда | Обычно заменяется transient input model |
| PlanningScenario | Обычно нет | Нужен для полноценного planning scenario, но не для быстрого расчёта |
| PlanningRun | Обычно нет | Возможен только technical trace, если расчёт важный или долгий |
| PlanningVariant | Нет | Inline calculation не является вариантом плана |
| PlanningResult | Обычно нет | Результат возвращается consumer module |
| PlanPublication | Нет | Публикация относится к полноценному плану |

Ключевое правило:

```text
Inline calculation must not bypass profile/adapter contracts.
```

Иначе разные модули начнут реализовывать собственное разузлование, расчёт потребностей и оценку сроков по-разному.

---

#### 14.1.3 Projection first, pull fallback

Inline / Fast Calculation должен по возможности использовать `Planning Input Projection`, потому что это обеспечивает скорость.

Но для MVP или для редко используемых данных допускается fallback через public read contracts.

```text
Projection first.
Pull fallback if allowed by profile policy.
```

Пример:

```text
OrderExplosionProfile
  preferred:
    ProductInput + ProcessInput из Planning Input Projection

  fallback:
    GetProductPlanningReadModel
    GetProcessPlanningReadModel
```

Fallback должен быть явно разрешён профилем. Нельзя допускать, чтобы каждый inline use-case произвольно ходил в любые модули и собирал данные по своей логике.

---

#### 14.1.4 Transient input model вместо persisted PlanningSnapshot

Для полного APS-расчёта результат profile/adapter pipeline — это persisted immutable `PlanningSnapshot`.

Для Inline / Fast Calculation обычно достаточно transient input model:

```text
Profile + Adapter
  -> transient calculation input model
  -> inline compute
  -> result DTO
```

Пример для разузлования:

```text
OrderExplosionInputAdapter
  ProductInput + ProcessInput + OrderRef
  -> ExplosionInputModel
  -> Calculation Kernel
  -> ExplosionResultDto
```

Пример для проверки обеспеченности:

```text
MaterialAvailabilityInputAdapter
  OrderInput + InventoryInput + ExpectedReceipts
  -> AvailabilityCheckInputModel
  -> Calculation Kernel
  -> MaterialAvailabilityResultDto
```

Если inline calculation используется для операционно важного изменения, рекомендуется сохранить lightweight trace:

```text
InlineCalculationTrace
- ProfileCode
- ObjectRefs
- SourceWatermarks
- InputHash
- EngineCode
- EngineVersion
- CorrelationId
- CalculatedAtUtc
```

Например, если результат разузлования сохраняется как производственные операции, нужно иметь возможность объяснить, по каким версиям изделия, маршрута и правил это было рассчитано.

---

#### 14.1.5 Где заканчивается Inline / Fast Calculation

Inline / Fast Calculation заканчивается возвратом результата вызывающему модулю.

```text
Planning Inline Calculation API
  -> InlineCalculationResult DTO
  -> Consumer Module
```

Дальше возможны два варианта.

**Вариант A. Transient result**

Например, пользователь или сервис проверяет обеспеченность заказа.

```text
Planning returns:
- material availability
- shortage warnings
- estimated dates
```

Результат не становится domain state.

**Вариант B. Consumer persists result**

Например, Order Management разузловывает заказ и сохраняет результат.

```text
Planning returns:
- child orders
- operations
- material requirements

Order Management persists:
- ProductionOrder
- OrderOperation
- OrderRequirement
```

В этом режиме Planning не создаёт `PlanningVariant` и не сохраняет `PlanningResult`.

Модуль-потребитель:

- владеет своим use-case;
- выполняет свою business validation;
- сам решает, сохранять ли результат;
- сохраняет результат в своих aggregates, если это его domain data;
- отвечает за audit/events своего изменения состояния.

Важно:

```text
Planning provides calculation logic.
Consumer owns persistence when calculation is part of consumer's command.
```

---

#### 14.1.6 Полный поток Inline / Fast Calculation

```text
Consumer Module Use Case
  ↓
Planning Inline Calculation API
  ↓
Resolve Calculation Input Profile
  ↓
Resolve object refs / scope / parameters
  ↓
Check Planning Input Projection freshness
  ↓
Create lightweight calculation cut
  ↓
Calculation Model Adapter builds transient input model
  ↓
Calculation Kernel executes
  ↓
Return result DTO to Consumer Module
  ↓
Consumer decides whether to persist result
```

Короткая формула:

```text
Consumer request
  -> profile
  -> projection/read fallback
  -> adapter
  -> transient input
  -> calculation kernel
  -> result DTO
  -> consumer persistence, if needed
```

---

#### 14.1.7 Отличие от Async Planning Run

| Вопрос | Inline / Fast Calculation | Async Planning Run |
|---|---|---|
| Кто запускает | Модуль-потребитель или UI action | Planning scenario / плановик |
| Есть PlanningScenario | Обычно нет | Да |
| Есть PlanningRun | Обычно нет, максимум technical trace | Да |
| Есть PlanningVariant | Нет | Да |
| Есть persisted PlanningResult | Обычно нет | Да |
| Есть Calculation Input Profile | Да | Да |
| Есть Calculation Model Adapter | Да | Да |
| Есть Planning Input Projection | Желательно да | Да |
| Есть SnapshotCut | Lightweight / technical context | Да |
| Есть persisted PlanningSnapshot | Не обязательно | Да |
| Кто хранит результат | Consumer module или никто | Planning & Scheduling |
| Назначение | Быстро посчитать для use-case | Построить, сравнить и опубликовать план |

Итог:

```text
Inline / Fast Calculation — это короткий путь внутри той же архитектуры,
а не обход Planning & Scheduling architecture.
```

---

### 14.2 Async Planning Run

Async Planning Run используется для полноценного планирования.

Типовые сценарии:

- расчёт большого набора заказов;
- расчёт горизонта планирования;
- построение расписания с учётом ресурсов;
- учёт календарей, WIP, остатков и поступлений;
- сравнение вариантов;
- what-if моделирование;
- анализ KPI;
- утверждение и публикация выбранного варианта.

Логический поток:

```text
PlanningScenario
  -> PlanningSnapshot
  -> PlanningRun
  -> Scheduling Compute Runtime
  -> PlanningVariant
  -> PlanningResult
  -> Approve
  -> Publish
```

В этом режиме Planning & Scheduling владеет:

- сценарием;
- snapshot;
- запуском расчёта;
- вариантом;
- результатом;
- анализом;
- публикацией.

---

### 14.3 External Solver Calculation

External Solver Calculation используется, если расчёт выполняется внешним APS, solver или специализированным оптимизационным движком.

Логический поток:

```text
Planning Module
  -> Snapshot Builder
  -> Solver Adapter
  -> External APS / Solver
  -> Result Importer
  -> Planning Result Store or Consumer Result DTO
```

Внешний solver является implementation detail.

Другие модули DMP не должны зависеть от модели внешнего APS.

---

### 14.4 Общий Calculation Kernel

Inline calculation и Async Planning Run должны по возможности использовать общий calculation kernel.

```text
Calculation Kernel
  - explode order
  - expand BOM / route
  - calculate material requirements
  - build operation graph
  - estimate dates
  - assign resources
  - calculate capacity
  - detect shortages
```

Разница между режимами не в алгоритмическом ядре, а в orchestration:

```text
Inline Calculation
  uses calculation kernel through application service
  returns DTO to consumer

Async Planning Run
  использует расчетное ядро через расчетный контур
  persists scenario / variant / result
```

Анти-паттерн:

```text
Order Management implements its own explosion.
Production Logistics implements another explosion.
Planning implements third explosion.
```

Правильный подход:

```text
Planning/Scheduling provides stable calculation contracts.
Consumer modules call them and persist their own results when appropriate.
```

---

### 14.5 Result ownership rule

Владение результатом зависит от use-case.

```text
If calculation is part of another module's command,
the consumer module may persist the result in its own aggregates.

If calculation is a planning scenario,
Planning & Scheduling persists scenario, variant and result.
```

Примеры:

| Сценарий                                                | Кто вызывает                      | Кто хранит результат             |
| ------------------------------------------------------- | --------------------------------- | -------------------------------- |
| Разузловать заказ при создании производственного заказа | Order Management                  | Order Management                 |
| Рассчитать потребности для комплектации                 | Production Logistics              | Production Logistics             |
| Проверить обеспеченность заказа                         | Order Management / Logistics      | Consumer or transient response   |
| Построить расписание на неделю                          | Planning & Scheduling             | Planning & Scheduling            |
| Сравнить несколько вариантов плана                      | Planning & Scheduling             | Planning & Scheduling            |
| Опубликовать утверждённый план                          | Planning & Scheduling + consumers | Consumers apply approved changes |

---

### 14.6 MVP vs industrial implementation

MVP и промышленная версия должны отличаться глубиной реализации, а не архитектурными границами.

Правильный принцип:

```text
MVP = industrial-compatible architecture with simplified implementation.
```

Неправильный принцип:

```text
MVP = temporary architecture to be rewritten later.
```

В MVP допустимо:

- держать расчетный контур физически в том же развертывании, что и модуль планирования;
- иметь один простой in-process worker;
- поддержать один базовый heuristic scheduler;
- ограничить параллельность;
- упростить snapshot model;
- хранить только ключевые result projections;
- иметь минимальные KPI/issues;
- реализовать external solver boundary как интерфейс без production adapter.

В промышленной версии добавляются:

- distributed worker pool;
- несколько scheduling engines;
- параллельный расчёт вариантов;
- параллельность внутри одного большого расчёта;
- advanced optimization;
- incremental rescheduling;
- complex what-if campaigns;
- advanced Gantt/resource analytics;
- CPU/RAM limits per tenant/plant;
- industrial external APS/solver adapters;
- более богатая модель diagnostics и traceability.

Но сохраняются те же архитектурные контейнеры:

```text
Scenario
Snapshot
Run
Variant
Result
Publication
Compute Runtime
Calculation Contracts
```

---

## 15. Typical enterprise planning scenarios

Этот раздел фиксирует типовые enterprise-сценарии Planning & Scheduling и показывает, как они ложатся на общий foundation:

```text
Planning Input Projection
  -> Calculation Input Profile / Model Adapter
  -> Snapshot or transient input model
  -> Compute
  -> Result
  -> optional baseline / publication / consumer persistence
```

Важно: эти сценарии не вводят отдельную архитектуру. Они отличаются назначением расчёта, режимом orchestration, политикой публикации и владением результатом.

Ключевые параметры сценария:

```text
CalculationMode
ScenarioPurpose
ProfileCode
PublicationPolicy
ResultOwnership
BaselinePolicy
```

Рекомендуемые значения `ScenarioPurpose`:

```text
CreatePlan
Replan
Forecast
WhatIf
BaselineFreeze
FeasibilityCheck
```

---

### 15.1 Plan Baseline / Freeze

Plan Baseline / Freeze используется, когда утверждённый план нужно зафиксировать за период для последующего анализа отклонений факта от плана.

Типовой сценарий:

```text
PlanningScenario
  -> PlanningSnapshot
  -> PlanningRun
  -> PlanningVariant
  -> PlanningResult
  -> Approve Variant
  -> Freeze as PlanBaseline
```

Ключевое различие:

```text
PlanningResult = расчётный результат варианта.
PlanBaseline = зафиксированная плановая база для анализа отклонений.
```

`PlanningResult` может быть много. `PlanBaseline` фиксирует, какой именно результат принят как плановая база на период.

Минимальная смысловая модель:

```text
PlanBaseline
- TenantId
- PlantId
- PeriodStartUtc
- PeriodEndUtc
- SourceScenarioId
- SourceVariantId
- SourceResultId
- BaselineType
- FrozenAtUtc
- FrozenBy
- Status
- CorrelationId
```

PlanBaseline может включать зафиксированные projection/read models:

```text
PlanBaselineOrder
PlanBaselineOperation
PlanBaselineResourceLoad
PlanBaselineMaterialRequirement
PlanBaselineKpi
```

Дальше фактические события из Shopfloor Execution, Production Logistics, Quality, MDC и других модулей сравниваются с baseline:

```text
PlanBaseline
  vs
Actual Execution / Fact Read Models
```

Примеры анализа отклонений:

- плановая дата начала операции vs фактическая дата начала;
- плановая дата завершения vs фактическая дата завершения;
- плановая загрузка ресурса vs фактическая загрузка;
- плановый выпуск vs фактический выпуск;
- плановая потребность в материале vs фактическое списание;
- плановый WIP vs фактический WIP.

Важно:

```text
PlanBaseline is not the same as the latest calculation result.
PlanBaseline is the accepted reference for deviation analysis.
```

Фиксация baseline не должна блокировать дальнейшее оперативное перепланирование. В системе одновременно могут существовать:

```text
PlanBaseline на неделю
OperationalForecast на текущую смену
WhatIfVariant для нового крупного заказа
```

---

### 15.2 Operational Replanning / Forecast

Operational Replanning / Forecast используется для пересчёта ожидаемых сроков и отклонений с учётом факта, WIP, простоев, дефицитов, изменений доступности ресурсов и текущего состояния производства.

Цель:

```text
Понять, когда теперь реально будут выполнены заказы,
и какие отклонения ожидаются относительно baseline или опубликованного плана.
```

Типовой поток:

```text
Shopfloor / Inventory / MDC / Logistics events
  -> Planning Input Projection updated
  -> Start Operational Replanning
  -> Create SnapshotCut на текущий момент
  -> Adapter marks fixed / movable / completed operations
  -> Расчетный контур рассчитывает ожидаемое расписание
  -> PlanningResult with ScenarioPurpose = Forecast or Replan
  -> Optional publication
```

Для этого сценария особенно важны:

- фактическое выполнение операций;
- WIP;
- статусы операций;
- текущая доступность ресурсов;
- простои оборудования;
- остатки и ожидаемые поступления;
- material readiness;
- frozen zone / frozen period.

`FrozenPeriodEndUtc` и policy заморозки должны превращаться в constraints внутри snapshot:

```text
Operation X is already started -> cannot move
Operation Y is in current shift -> fixed resource and time
Operation Z is published to dispatch -> move only by explicit policy
Batch A is partially completed -> remaining quantity only
```

Результат оперативного перепланирования может использоваться в двух режимах.

**Forecast only**

Результат показывает ожидаемые сроки, отклонения, риски, дефициты и перегрузки, но не меняет операционные данные.

```text
Recalculate expected state
  -> show delays / risks / shortages
  -> no publication
```

**Forecast + Publish**

Плановик принимает результат как новый оперативный план.

```text
Approve Replanning Variant
  -> Publish updated schedule
  -> Shopfloor / Logistics receive changes
```

Ключевое правило сохраняется:

```text
Recalculate != Publish
```

Оперативный пересчёт может быть только прогнозом, а может стать опубликованным расписанием только через явный publication use-case.

---

### 15.3 Feasibility / Promise Check for new demand

Feasibility / Promise Check используется, когда пришёл новый заказ, заявка или запрос, и нужно быстро оценить реализуемость по срокам, ресурсам и материалам.

Этот сценарий чаще всего не должен запускать полный пересчёт всего завода.

Типовой режим:

```text
Inline / Fast Calculation
или
Small async calculation for complex demand
```

Логический поток:

```text
New CandidateDemand
  -> Feasibility / CTP Profile
  -> Planning Input Projection
  -> lightweight cut
  -> Adapter builds candidate demand model
  -> Calculation Kernel checks feasibility
  -> FeasibilityResult DTO
```

`CandidateDemand` — это расчётное представление нового спроса. Оно не обязано быть уже созданным production order.

```text
CandidateDemand
- ItemCode
- Quantity
- RequestedDate
- Priority
- Customer / Project / Contract refs
- RequiredAttributes
- AllowedSubstitutions
- PromisePolicy
```

Ключевой принцип:

```text
CandidateDemand != ProductionOrder
```

Это позволяет оценивать запросы без загрязнения операционных данных.

Проверка должна выполняться поверх текущего принятого/опубликованного плана и актуального состояния:

```text
Existing committed plan
+ current WIP/fact
+ known inventory/supply
+ new candidate demand
```

Возможные политики проверки:

**Promise check without moving committed plan**

```text
Do not move existing committed operations.
Try to fit new demand into free capacity.
```

Результат:

```text
CanPromise
EarliestPossibleDate
BottleneckResources
MaterialShortages
CapacityGaps
```

**Promise check with limited impact**

```text
May move only non-published operations.
May use alternative resources.
Must not violate high-priority orders.
Must not change current shift.
```

Результат:

```text
Можно выполнить к дате, если:
- сдвинуть операции A/B/C;
- использовать ресурс R2 вместо R1;
- ускорить поступление материала M.
```

**Full what-if scenario**

Если заказ крупный, проектный или влияет на значительную часть завода, он должен оформляться как полноценный what-if scenario:

```text
PlanningScenario with ScenarioPurpose = WhatIf
  -> PlannerChange = AddCandidateDemand
  -> Async Planning Run
  -> compare with current plan / baseline
```

---

### 15.4 Summary of scenario semantics

| Сценарий | Основной смысл | Обычно используемый режим | Результат | Меняет операционные данные? |
|---|---|---|---|---|
| Plan Baseline / Freeze | Зафиксировать утверждённый план за период | После Async Planning Run | PlanBaseline | Нет, это база для анализа отклонений |
| Operational Replanning / Forecast | Пересчитать ожидаемые сроки и отклонения | Async Planning Run | Forecast / Replanning Result | Только после publish |
| Feasibility / Promise Check | Быстро оценить новый спрос | Inline или small async | FeasibilityResult / PromiseOptions | Нет, пока заказ не принят |

Итоговое правило:

```text
Один foundation.
Разные enterprise use-cases.
Разная семантика результата.
```

---

## 16. Расчетный контур расписаний

Расчетный контур расписаний (`Scheduling Compute Runtime`) — отдельный вычислительный контур, скрытый за стабильным контрактом планирования.

Расчетный контур получает уже интерпретированный **снимок данных под конкретный расчет**. Он не должен знать, как исходные данные были получены из Product / Process / Order / Resource / Inventory, и не должен работать с канонической проекцией входных данных планирования напрямую.

### 16.1 Ответственность

Расчетный контур отвечает за:

- получение запроса на расчет;
- загрузку неизменяемого снимка данных под конкретный расчет;
- проверку поддерживаемой версии схемы снимка данных;
- построение модели в памяти;
- выполнение алгоритма планирования;
- параллельные расчёты;
- расчет KPI и диагностики;
- возврат результата;
- отчет о ходе расчета;
- обработку отмены.

### 16.2 Не отвечает

Расчетный контур не должен:

- владеть доменными основными данными;
- напрямую читать таблицы Order/Product/Resource/etc.;
- читать Planning Input Projection как источник расчёта;
- выполнять интерпретацию исходных данных под конкретный профиль;
- выполнять утверждение через workflow;
- принимать решение о публикации плана;
- менять производственные заказы напрямую;
- знать конфигурацию интерфейса;
- обходить контракты tenant, безопасности и Object Runtime.

### 16.3 Интерфейс расчетного контура

Минимальный контракт:

```text
SubmitCalculation(request)
CancelCalculation(runId)
GetCalculationStatus(runId)
ReportProgress(runId, ход выполнения)
StoreResult(runId, result)
```

Запрос на расчет:

```text
CalculationRequest
- RunId
- TenantId
- PlantId
- ScenarioId
- SnapshotId
- VariantId
- EngineCode
- EngineVersion
- ParameterSet
- Priority
- CorrelationId
```

---

## 17. Параллельность и масштабирование

Нужно различать два уровня параллельности.

### 17.1 Параллельность между расчётами

Много пользователей, предприятий, сценариев и вариантов могут запускать расчёты одновременно.

Требуются:

- очередь расчётов;
- приоритеты;
- лимиты по tenant/plant;
- отмена расчёта;
- идемпотентность;
- отчет о ходе расчета;
- изоляция ресурсов;
- аудит и трассировка через CorrelationId.

Пример политик:

```text
maxConcurrentRunsPerTenant
maxConcurrentRunsPerPlant
maxConcurrentRunsPerUser
maxCpuSecondsPerRun
maxMemoryPerRun
defaultPriority
largeRunQueue
smallRunQueue
отменаPolicy
```

### 17.2 Параллельность внутри одного расчёта

Возможные стратегии:

1. Параллельный расчёт нескольких вариантов.
2. Параллельный расчёт независимых участков, цехов или групп ресурсов.
3. Параллельное выполнение нескольких эвристик с выбором лучшего результата.
4. Параллельный локальный поиск по узким местам ресурсов.
5. Вызов внешних заданий решателя для отдельных подзадач.

Рекомендация для первого промышленного объема:

```text
Начать с параллельных вариантов и объяснимых эвристик.
Не вводить преждевременно глобальный оптимизатор.
```

### 17.3 Критичный по производительности путь расчета

Для критичных по производительности сценариев, например `OperationSchedulingProfile` на тысячи или десятки тысяч операций, целевой путь должен быть таким:

```text
Planning Input Projection
  -> входной пакет под конкретный расчет
  -> модель в памяти
  -> эвристический движок расписаний
  -> ограниченная локальная оптимизация
  -> пакетная запись результата
```

В горячем цикле расчёта запрещено опираться на медленные внешние вызовы:

```text
Нет вызовов API модулей-источников в горячем цикле расчета.
Нет вызовов Rule Engine на каждую операцию.
Нет вызовов Reference Data API на каждую операцию.
Нет сохранения через ORM на каждую плановую операцию.
```

Все массовые данные, влияющие на расчет, должны быть подготовлены до запуска движка: допустимость ресурсов, календари, интервалы доступности ресурсов, доступность материалов, приоритеты, ограничения и правила переналадки.

Целевое требование к производительности может быть задано для конкретного профиля отдельно. Например: `OperationSchedulingProfile` должен строить реализуемое и объяснимое расписание с ограниченной локальной оптимизацией, а не доказывать глобальный оптимум.

---

## 18. Planning, Scheduling, Dispatching

В DMP нужно разделять три уровня:

```text
Planning
  Что и когда должно быть произведено / обеспечено.

Scheduling
  На каких ресурсах и в какой последовательности выполнять операции.

Dispatching
  Что прямо сейчас должен делать цех / рабочий центр / оператор.
```

Planning & Scheduling module может владеть planning/scheduling результатами, но dispatch execution belongs to Shopfloor Execution.

Связь:

```text
Planning result
  ↓ publish
Shopfloor dispatch list / operation queue
  ↓ execution
Shopfloor actuals
  ↓ events
Planning rescheduling input
```

---

## 19. Конфигурация Planning & Scheduling

Часть поведения должна быть конфигурируемой, но не весь алгоритм.

### 19.1 Что можно конфигурировать

- типы сценариев;
- горизонты планирования;
- frozen period policy;
- критерии оптимизации;
- правила приоритезации;
- правила выбора ресурсов;
- правила учёта переналадок;
- правила группировки спроса;
- material pegging policy;
- параметры what-if;
- доступность planning actions;
- UI views планировщика;
- dashboard/report definitions;
- export/output templates.

### 19.2 Что не должно быть полностью конфигурируемым

- core scheduling algorithms;
- hard domain invariants;
- tenant isolation;
- security model;
- ownership data boundaries;
- physical DB access logic;
- произвольные пользовательские скрипты внутри расчетного контура.

Ключевой принцип:

```text
Configuration defines policy and parameters.
Scheduling engine executes approved algorithms.
Rules influence decisions only at controlled extension points.
```

---

## 20. Использование Rule Engine

Rule Engine может использоваться в контролируемых точках, но не должен вызываться построчно в горячем цикле расчёта для сотен тысяч операций.

### 20.1 Допустимые execution points

```text
BeforeBuildSnapshot
  Проверка допустимости сценария и scope.

ResolvePlanningScope
  Определение заказов и объектов, входящих в расчёт.

CalculateDemandPriority
  Расчёт приоритетов спроса до построения snapshot.

ResolveResourceEligibility
  Предварительная фильтрация допустимых ресурсов.

BeforeStartCalculation
  Проверка параметров запуска.

BeforePublishPlan
  Проверка возможности публикации выбранного варианта.

OnPlanIssueDetected
  Классификация severity / routing проблемы.
```

### 20.2 Правило производительности

Если правило влияет на массовый расчёт, его результат должен быть:

- предварительно вычислен;
- материализован в snapshot;
- скомпилирован в эффективную структуру;
- не вызывать Rule Engine на каждую операцию внутри tight loop.

---

## 21. Workflow integration

Planning & Scheduling может использовать workflow для lifecycle объектов:

- PlanningScenario;
- PlanningVariant;
- PlanPublication;
- возможно PlanningRun для административного контроля.

Примеры состояний PlanningVariant:

```text
Draft
Calculated
Reviewed
Approved
Published
Rejected
Archived
```

Примеры workflow commands:

```text
SubmitForCalculation
Review
Approve
Reject
Publish
Archive
```

Важно:

- workflow управляет lifecycle;
- business actions выполняются application use-cases;
- расчетный контур не исполняет workflow;
- публикация плана проходит серверную проверку, IAM, правила, аудит и события.

---

## 22. API и Events

### 22.1 Основные API

```text
POST /planning/scenarios
GET  /planning/scenarios/{id}
POST /planning/scenarios/{id}/build-snapshot
POST /planning/runs
GET  /planning/runs/{id}/status
POST /planning/runs/{id}/cancel
GET  /planning/variants/{id}
GET  /planning/variants/{id}/result
POST /planning/variants/{id}/approve
POST /planning/variants/{id}/publish
GET  /planning/results/{id}/gantt
GET  /planning/results/{id}/resource-load
GET  /planning/results/{id}/issues

GET  /planning/input-projection/status
POST /planning/input-projection/rebuild
GET  /planning/input-projection/watermarks
POST /planning/input-projection/validate-freshness
GET  /planning/input-profiles
GET  /planning/input-profiles/{code}
```

API names are illustrative. Final endpoints must follow platform API conventions.

### 22.2 Основные events

```text
PlanningScenarioCreated
PlanningSnapshotCutCreated
PlanningSnapshotBuilt
PlanningRunRequested
PlanningRunStarted
PlanningRunProgressChanged
PlanningRunCompleted
PlanningRunFailed
PlanningRunCancelled
PlanningVariantCalculated
PlanningVariantApproved
PlanPublished
PlanPublicationFailed

PlanningInputProjectionUpdated
PlanningInputProjectionRebuildRequested
PlanningInputProjectionRebuilt
PlanningSourceWatermarkAdvanced
PlanningInputFreshnessValidationFailed
```

События должны содержать:

```text
EventId
EventTypeCode
OccurredAtUtc
TenantId
CorrelationId
SourceModule
PayloadVersion
Payload
```

---

## 23. Tenant isolation и security

Модуль планирования и расписаний обязан учитывать tenant.

Требования:

- все operational planning data содержат TenantId;
- расчёты изолированы по tenant;
- cross-tenant доступ запрещён;
- лимиты расчётов задаются минимум на tenant/plant уровне;
- каждый API request содержит TenantContext;
- каждый calculation job содержит TenantId, PlantId, CorrelationId;
- compute workers не должны смешивать данные разных tenant внутри одного расчёта;
- публикация плана требует backend authorization.

---

## 24. Audit, history, traceability

Planning & Scheduling должен поддерживать три разные модели:

### 24.1 Audit

Обязательно аудировать:

- создание сценария;
- запуск расчёта;
- отмену расчёта;
- изменение параметров сценария;
- утверждение варианта;
- публикацию плана;
- ошибки публикации;
- export результатов.

### 24.2 History

Пользовательская история должна показывать:

- кто создал сценарий;
- когда запускался расчёт;
- какой вариант был выбран;
- кто утвердил;
- когда опубликован;
- какие ключевые изменения были применены.

### 24.3 Technical trace

Trace нужен для:

- диагностики долгих расчётов;
- анализа ошибок engine;
- воспроизведения результата;
- связи snapshot → run → result → publication через CorrelationId.

---

## 25. Состав первого промышленного объема

Для первого промышленного объема не нужно сразу строить полноценный промышленный APS.

### 25.1 Первый промышленный объем должен включать

1. PlanningScenario.
2. Минимальную Planning Input Projection для 1–2 критичных источников.
3. Минимальные отметки свежести источников и статус проекции.
4. 1–2 Calculation Input Profile.
5. 1–2 Calculation Model Adapter.
6. Дополнительное чтение по требованию для данных, которые еще не проецируются.
7. PlanningSnapshot Builder.
8. Async PlanningRun.
9. Один встроенный эвристический планировщик расписаний.
10. PlanningVariant.
11. Хранилище PlanningResult.
12. Базовые KPI и issues.
13. Просмотр результата в интерфейсе и моделях чтения.
14. Утверждение и публикацию выбранного варианта.
15. События и аудит.
16. Границу адаптера для будущего внешнего APS или решателя.

### 25.2 Первый промышленный объем может не включать

- полную событийную проекцию по всем модулям-источникам;
- полное повторное проигрывание событий;
- распределенное построение снимков данных;
- сложные политики свежести данных для всех профилей;
- сложную глобальную оптимизацию;
- полноценный CP/MILP solver;
- визуальный конструктор правил планирования;
- автоматическое перепланирование в реальном времени;
- глобальную оптимизацию по нескольким заводам;
- сложные кампании моделирования "что если";
- сложное моделирование на уровне ресурсов.

---

## 26. Внешний APS или адаптер решателя

Архитектура должна позволять подключить внешний APS или решатель без изменения остальных модулей DMP.

```text
Planning Module
  -> Snapshot Builder
  -> Solver Adapter
  -> External APS / Solver
  -> Result Importer
  -> Planning Result Store
```

Адаптер отвечает за:

- сопоставление снимка данных DMP с внешней моделью;
- запуск внешнего расчета;
- получение статуса через опрос или обратный вызов;
- сопоставление внешнего результата с результатом планирования DMP;
- диагностику ошибок;
- совместимость версий.

Важно:

```text
Внешний APS является деталью реализации расчетного контура.
Другие модули DMP не должны зависеть от модели внешнего APS.
```

---

## 27. Основные архитектурные риски

### 27.1 Модуль планирования становится слишком большим сервисом

Риск: Planning начнёт напрямую читать и интерпретировать таблицы всех модулей.

Как снижать риск:

- только API, события и контракты чтения;
- явный построитель снимка данных;
- стабильные контракты;
- тесты границ модулей.

### 27.2 Расчетная логика протекает в прикладной слой

Риск: тяжелый алгоритм окажется внутри обычного прикладного сервиса.

Как снижать риск:

- отдельный расчетный контур;
- асинхронные задания;
- четкая граница расчетного движка.

### 27.3 Результат хранится только как сырой JSON

Риск: невозможно строить Gantt, отчёты, анализ дефицитов и диагностику.

Как снижать риск:

- проекции результата, пригодные для запросов;
- JSON только для диагностики и сырого технического содержимого.

### 27.4 Configuration becomes no-code optimizer

Риск: попытка сделать произвольный конфигурируемый алгоритм приведёт к хаосу.

Как снижать риск:

- конфигурация задает параметры и политики;
- алгоритмы являются утвержденным кодом или подключаемыми модулями;
- Rule Engine вызывается только в контролируемых точках.

### 27.5 Результаты расчета автоматически меняют операции

Риск: каждый расчёт начнёт менять производственные заказы и расписание без контроля.

Как снижать риск:

- расчет не равен публикации;
- отдельный сценарий публикации;
- утверждение через workflow;
- аудит и события.

---

## 28. Рекомендуемая структура модуля в коде

```text
DMP.Modules.PlanningScheduling
  /Api
    PlanningScenarioController
    PlanningRunController
    PlanningVariantController
    PlanningPublicationController
    PlanningInputProjectionController
    PlanningInputProfileController

  /Application
    /UseCases
      CreateScenario
      ResolvePlanningScope
      CreateSnapshotCut
      BuildSnapshot
      StartCalculation
      CancelCalculation
      CompareVariants
      ApproveVariant
      PublishPlan
      QueryPlanResult
      ValidatePlanningInputFreshness
      RebuildPlanningInputProjection

    /InputProfiles
      CalculationInputProfileRegistry
      CapacityPlanningProfile
      OperationSchedulingProfile
      OrderExplosionProfile

    /SnapshotBuilding
      IPlanningSnapshotBuilder
      ISnapshotCutManager
      ICalculationModelAdapter
      CapacityPlanningInputAdapter
      OperationSchedulingInputAdapter
      OrderExplosionInputAdapter

    /ProjectionIngestion
      PlanningInputIngestionHandlers
      ProjectionUpdateOrchestrator
      SourceWatermarkService

    /Contracts
      IPlanningSnapshotBuilder
      ISchedulingComputeClient
      IPlanningResultRepository
      IPlanPublicationService
      IPlanningInputProjectionStore
      ISourceWatermarkStore

  /Domain
    PlanningScenario
    PlanningRun
    PlanningSnapshot
    PlanningVariant
    PlanningResult
    PlanPublication
    PlanIssue
    PlanKpi

  /Infrastructure
    Persistence
    EventPublishing
    InputProjectionStore
    ProjectionConsumers
    ProjectionRebuild
    Watermarks
    SnapshotStorage
    ResultProjections
    ExternalSolverAdapters

  /Configuration
    Manifests
    Codes
    Baseline
    InputProfiles

DMP.SchedulingCompute
  SchedulingOrchestrator
  WorkerPool
  EngineHost
  SnapshotLoader
  InMemoryModel
  Algorithms
  ProgressReporter
  ResultWriter
```

Компоненты проекции входных данных планирования должны следовать тому же правилу архитектуры модуля: обработчики приема данных и хранилища относятся к инфраструктуре и моделям чтения; прикладные сценарии остаются в Application; владение исходными данными остается за пределами модуля планирования.

---

## 29. Ключевые стабильные коды

Модуль планирования и расписаний должен иметь собственный каталог кодов.

Примеры:

```text
ObjectTypeCode:
- Planning.Scenario
- Planning.Run
- Planning.Variant
- Planning.Result
- Planning.Publication

ActionCode:
- Planning.CreateScenario
- Planning.BuildSnapshot
- Planning.StartCalculation
- Planning.CancelCalculation
- Planning.ApproveVariant
- Planning.PublishPlan

DatasetCode:
- Planning.VariantList
- Planning.ResultGantt
- Planning.ResourceLoad
- Planning.PlanIssues
- Planning.MaterialShortages
- Planning.InputProjectionStatus
- Planning.SourceWatermarks

CalculationInputProfileCode:
- Planning.Profile.OrderExplosion
- Planning.Profile.CapacityPlanning
- Planning.Profile.OperationScheduling
- Planning.Profile.MaterialAvailability
- Planning.Profile.FullApsPlanning

CalculationModeCode:
- Planning.Mode.InlineCalculation
- Planning.Mode.AsyncPlanningRun
- Planning.Mode.ExternalSolver

PlanningInputDatasetCode:
- Planning.Input.Product
- Planning.Input.Process
- Planning.Input.Resource
- Planning.Input.Order
- Planning.Input.Inventory
- Planning.Input.Wip

PlanningSnapshotSchemaCode:
- Planning.Snapshot.OrderExplosion.v1
- Planning.Snapshot.CapacityPlanning.v1
- Planning.Snapshot.OperationScheduling.v1

PlanningEngineCode:
- Planning.Engine.HeuristicScheduler
- Planning.Engine.ExternalSolver

ScenarioPurposeCode:
- Planning.Purpose.CreatePlan
- Planning.Purpose.Replan
- Planning.Purpose.Forecast
- Planning.Purpose.WhatIf
- Planning.Purpose.BaselineFreeze
- Planning.Purpose.FeasibilityCheck

EventTypeCode:
- Planning.RunRequested
- Planning.RunCompleted
- Planning.PlanPublished
- Planning.InputProjectionUpdated
- Planning.SourceWatermarkAdvanced
```

Итоговые коды должны быть определены в каталоге кодов модуля и зарегистрированы через платформенные контракты.

---

## 30. Итоговое решение

Модуль планирования и расписаний в DMP должен быть реализован как:

```text
Прикладной модуль + расчетный контур, а не монолитное APS-приложение.
```

Итоговая формула:

```text
Модуль планирования и расписаний
  владеет сценариями, запусками, снимками данных, вариантами, результатами и публикацией плана.

Planning Input Projection
  предоставляет быструю модель чтения для планирования, но не владеет исходными основными или операционными данными.

Профили и адаптеры расчета
  преобразуют канонические входные данные планирования в неизменяемые снимки под конкретный расчет.

Scheduling Compute Runtime
  выполняет тяжелые расчеты по неизменяемым снимкам под конкретный расчет.

Другие модули
  предоставляют исходные данные через стабильные события и контракты чтения, а утвержденные результаты принимают только через стабильные API и события.
```

Это решение позволяет:

- сохранить границы предметных областей;
- масштабировать расчёты независимо от обычных API;
- поддержать моделирование "что если" и варианты плана;
- обеспечить воспроизводимость расчётов;
- подключить внешний APS или решатель в будущем;
- не превратить DMP в набор слабо связанных внешних приложений;
- встроить модуль планирования и расписаний в платформенную архитектуру, конфигурацию, workflow, правила, аудит и интеграционную модель.
