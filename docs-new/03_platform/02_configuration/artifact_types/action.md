---
id: DOC-03-02-AT-ACTION
title: 'Тип конфигурационного артефакта — Action'
type: module-spec
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: configuration
holder: '@axelprosoft'
created_at: 2026-08-26 00:00
created_by: '@axelprosoft'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Тип конфигурационного артефакта — Action

[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ActionArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ActionArtifactSchemas.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/ActionNodeBuilder.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/ActionBaselineBuilder.cs
[registration]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/Registration/ConfigurationActionRegistration.cs
[runtime-descriptor]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Objects/ObjectRuntimeDescriptor.cs
[runtime-contract]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Objects/BusinessObjectContractDefinition.cs
[runtime-action-request]: ../../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/RuntimeObjectActionRequest.cs
[runtime-action-response]: ../../../../src/Platform/DMP.Platform.Contracts/Runtime/Responses/RuntimeObjectActionResponse.cs
[runtime-action-materializer]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Services/Materializers/RuntimeActionMaterializer.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactSchemaRegistryIntegrationTests.cs
[builder-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ActionNodeBuilderIntegrationTests.cs
[baseline-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ConfigurationBaselineAuthoringDslIntegrationTests.cs
[configuration-terms]: ../../../11_glossary/configuration_terms.md

## 1. Назначение

`Action` — корневой конфигурационный артефакт операции. Он хранит настройки
названия и отображения, способ вызова, параметры ввода, подтверждение,
обработку результата, локальное поведение и ссылки на общие правила.

Этот документ описывает schema-контракт `Action` и его дочерних узлов. Код
обработчика, серверная бизнес-логика, транзакция, права и реализация frontend
не являются свойствами `Action` и описываются у своих владельцев.

## 2. Идентичность и граница

### 2.1. Идентичность

Корневой `Action` идентифицируется полем `ArtifactNode.Code`. В контексте
действия этот код является `ActionCode`. Поле `ArtifactNode.ArtifactTypeCode`
имеет значение `Action`.

В schema нет свойства `ActionCode`. Поэтому `ActionCode` не включается в
таблицу свойств: это техническая идентичность узла, используемая в ссылках из
`View`, `Workflow`, `RuleBinding` и runtime-запросов.

Дочерний узел также идентифицируется собственным `ArtifactNode.Code` внутри
коллекции родителя. Например, код узла `ActionParameter` является
`ParameterCode`, но не свойством параметра.

### 2.2. Входит в документ

- schema и структура корневого `Action`;
- свойства и ограничения пяти дочерних типов;
- источники значений, операции schema-уровня и проверки публикации;
- граница между конфигурацией операции, её runtime-контрактом и размещением в
  интерфейсе.

### 2.3. Не входит в документ

- обработчик действия, command DTO и предметная бизнес-логика;
- транзакция, репозиторий, mutation pipeline и обязательные серверные проверки;
- конкретная кнопка или пункт меню в `View`;
- общий frontend shell, рендерер и реализация `Action Editor`;
- исполнение workflow, аудит, outbox и доставка событий.

## 3. Место в Configuration

`Action` зарегистрирован в schema registry с признаками `Node`, `ObjectBound`,
`ActionHost`, `HasConditions` и `RuntimeRenderable`. Schema registry также
регистрирует пять дочерних типов: `ActionResultBindings`, `ActionParameter`,
`ActionLocalBehavior`, `ActionRuleBinding` и `ActionMessage`.
([схема Action][schemas])

Configuration владеет schema, каноническим хранением, baseline-authoring,
проверкой и публикацией `Action`. `ActionNodeBuilder` создаёт канонический
 узел, а `ActionBaselineBuilder` создаёт запись в базовом пакете конфигурации.
([сборщик канонического узла][builder], [сборщик baseline][baseline-builder])

### 3.1. Владение по слоям

| Слой | Ответственность | Где описывается |
| --- | --- | --- |
| Configuration | schema, свойства, дочерние узлы, ссылки, validation, публикация и effective configuration | Этот документ и документы Configuration |
| Object Runtime | регистрация исполняемой операции, проверка доступности и вызов handler или command | `ObjectRuntimeDescriptor`, runtime-контракты |
| фронтенд-платформа | рендерер, форма ввода, подтверждение, сообщения и отображение результата | `06_user_experience.md` и область фронтенд-платформа |
| View | место размещения действия и контекст его показа | `ViewActionPlacement` в `view.md` |
| Workflow | состояния, переходы и workflow-команды | артефакты и runtime Workflow |
| Rules | переиспользуемая логика правила | артефакт `Rule` и Rules runtime |

`ActionType` классифицирует действие в конфигурации. Он не выбирает класс
обработчика и не заменяет регистрацию действия в Object Runtime.

Для object-bound действия `ObjectRuntimeDescriptor.Actions` содержит отдельный
runtime-дескриптор: `Code`, `Scope`, `ExecutionKind`, `HandlerType` и, если
нужно, `Command`. В текущем runtime `Scope` имеет значения `Object`,
`Selection`, `Bulk`, а `ExecutionKind` — `Handler`, `TransitionAdapter`,
`PresentationOnly`, `Command`. Это runtime-контракт, а не свойства schema
`Action`. ([runtime-дескриптор][runtime-descriptor], [runtime-контракт][runtime-contract])

`ActionCode` и `CommandCode` не являются одним кодовым пространством. `ActionCode`
ссылается на операцию, а `CommandCode` относится к команде управления
workflow. Workflow может вызвать действие, но не становится владельцем его
schema или обработчика.

## 4. Структура и схема `Action`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств.

```text
Action (ArtifactNode)
├── identity: ArtifactTypeCode = Action; Code = <ActionCode>
├── properties: { общие свойства, Invocation, Confirmation,
│                ResultHandling, InputSurface }
└── child collections:
    ├── ResultBindings[0..1] → ActionResultBindings
    ├── Parameters[0..n] → ActionParameter
    ├── LocalBehaviors[0..n] → ActionLocalBehavior
    ├── RuleBindings[0..n] → ActionRuleBinding
    └── Messages[0..n] → ActionMessage
```

Группы `Invocation`, `Confirmation`, `ResultHandling` и `InputSurface` — это
группы свойств корневого узла, зарегистрированные для editor-представления.
Они не являются отдельными schema-узлами.

| Что это | Техническое имя в коде | Русский смысл | Где описано подробно |
| --- | --- | --- | --- |
| Идентичность корневого узла | `ArtifactNode.ArtifactTypeCode` + `ArtifactNode.Code` | Тип узла `Action` и код операции | Раздел 2.1 |
| Свойства корневого узла | `ArtifactNode.Properties`, schema `Action` | Название, вызов, ввод и обработка результата | Раздел 4.2 |
| Связь результата ввода | `ResultBindings` → `ActionResultBindings` | Перенос результата формы или выбора в параметры действия | Раздел 4.3 |
| Параметры | `Parameters` → `ActionParameter` | Описания входных параметров операции | Раздел 4.4 |
| Локальные поведения | `LocalBehaviors` → `ActionLocalBehavior` | Реакции способа ввода на условие | Раздел 4.5 |
| Связи с правилами | `RuleBindings` → `ActionRuleBinding` | Привязки `Rule` к действию или параметру | Раздел 4.6 |
| Сообщения | `Messages` → `ActionMessage` | Сообщения и ссылки на их шаблоны | Раздел 4.7 |

`ActionParameter.MemberType` является дискриминатором дочернего узла:

```text
MemberType = Scalar
└── DataType, DefaultValue, ValueSetCode или SystemEnumCode

MemberType = Reference
└── ReferenceSourceCode, LookupViewCode

MemberType = Collection
└── ItemObjectTypeCode, SelectionMode, LookupViewCode
```

### 4.2. Свойства корневого узла

Ниже приведён полный каталог свойств, зарегистрированных в текущей схеме
`Action`. `Required` означает обязательность свойства в схеме; это не всегда
означает обязательность пользовательского ввода в UI. Источники значений и
правила обновления описаны в разделе 5. Смысл значений и их влияние указаны в
этой же строке таблицы, а не в отдельном повторном каталоге.

Пометка «статический каталог вариантов» означает, что допустимые коды заданы в
[каталоге статических вариантов][static-options]. Это не список, который может
произвольно расширить клиентский редактор.

Ниже находятся **таблицы свойств артефакта**. В каждой таблице одна строка
описывает одно свойство конкретного узла схемы. Для всех таблиц используются
одинаковые столбцы:

- `Технический код` — имя свойства в schema и коде;
- `Русский смысл` — понятное русское название свойства;
- `Тип значения` — формат значения, например `Bool`, `Enum`, `String` или
  `ReferenceCode`;
- `Обязательность / значение по умолчанию` — можно ли не задавать свойство и
  какое значение применяется при отсутствии значения;
- `Допустимые значения или источник` — допустимые коды, формат либо каталог,
  из которого выбирается ссылка или значение;
- `Смысл значения и влияние` — что означает значение и на что оно влияет;
- `Когда используется / переопределение` — условие применения и правила
  наследования.

В последнем столбце используются следующие обозначения:

- `Default` — для свойства нет специального ограничения наследования;
- `InheritedOnly` — значение берётся от базового типа и не изменяется в
  производном типе;
- `CreationOnly` — значение задаётся только при создании;
- `OverrideAllowed` — производный тип может заменить унаследованное значение;
- `Schema filter` — сервер предлагает только значения из каталога, прошедшие
  условие, например `View` определённого типа;
- `Дискриминатор` — значение определяет допустимую форму или вид вложенного
  узла;
- `Все виды` — свойство применяется при любом допустимом значении условия;
- `Только <условие>` — свойство разрешено только при указанном условии;
- `Все экземпляры` — отдельного ограничения применения нет.

В столбце обязательности используются `Да`, `Нет` и `Условно`. `Условно`
означает, что обязательность зависит от другого свойства. Если после `Нет`
указано `/ значение`, это значение применяется по умолчанию.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название действия | `LocalizedText` | Нет | Локализованные значения | Название операции в редакторе и интерфейсе; обработчик не меняет | Default |
| `Description` | Описание действия | `LocalizedText` | Нет | Локализованные значения | Поясняет назначение операции; не задаёт бизнес-логику | Default |
| `Tooltip` | Подсказка действия | `LocalizedText` | Нет | Локализованные значения | Краткая подсказка при наведении или в компактном представлении | Default |
| `IconCode` | Код значка | `String` | Нет | Строковый код; каталог и рендерер не заданы schema | Выбирает значок в frontend-представлении; не влияет на выполнение | Default |
| `Style` | Визуальный стиль | `Enum` | Нет | `Primary`, `Secondary`, `Danger`, `Ghost`; статический backend-каталог вариантов | Выбирает визуальный стиль действия; точное оформление принадлежит frontend | Default |
| `ObjectTypeCode` | Тип объекта действия | `ReferenceCode` | Нет | `Catalog.ObjectTypes` | Ограничивает или задаёт object context операции | `CreationOnly`, `InheritedOnly`; runtime использует как контекст типа объекта |
| `ActionType` | Вид действия | `Enum` | Нет | `Object`, `Module`, `Application`, `WorkflowTrigger`, `Service`, `System`; коды `ConfigurationActionTypeCodes`, отдельный options-каталог schema не задан | Классифицирует область операции; не выбирает обработчик | `InheritedOnly` |
| `AvailableInModes` | Доступные режимы | `String` | Нет | Список кодов через запятую; формат и каталог режимов schema не задаёт | Ограничивает режимы, в которых потребитель может показывать действие | `InheritedOnly`; builder нормализует список режимов |
| `InvocationType` | Сценарий вызова | `Enum` | Нет | `Simple`, `Confirmed`, `Form`, `Selection`, `Wizard`, `Complex`; коды `ConfigurationActionInvocationTypeCodes` | Определяет общий сценарий запуска; точный flow принадлежит frontend и runtime-контракту | `InheritedOnly` |
| `InvocationExecutionMode` | Режим выполнения | `Enum` | Нет | `Immediate`, `Async`, `Background`, `Batch`; коды `ConfigurationActionExecutionModeCodes` | Задаёт ожидаемый режим выполнения; не отменяет проверки и права | `InheritedOnly` |
| `InvocationSelectionMode` | Режим выбора объектов | `Enum` | Нет | `None`, `Single`, `Multiple`; коды `ConfigurationActionSelectionModeCodes` | Показывает, нужен ли объект, один объект или несколько объектов | `InheritedOnly` |
| `InvocationRequiresConfirmation` | Требуется подтверждение | `Bool` | Нет | `true`, `false` | `true` требует подтверждения перед вызовом; `false` не требует его по этому признаку | Default |
| `ConfirmationTitle` | Заголовок подтверждения | `LocalizedText` | Нет | Локализованные значения | Заголовок сообщения подтверждения | Default |
| `ConfirmationMessage` | Текст подтверждения | `LocalizedText` | Нет | Локализованные значения | Объясняет, что будет подтверждено; не заменяет проверку прав | Default |
| `ConfirmationButtonText` | Текст кнопки подтверждения | `LocalizedText` | Нет | Локализованные значения | Подпись команды подтверждения в интерфейсе | Default |
| `ConfirmationDanger` | Опасное действие | `Bool` | Нет | `true`, `false` | `true` помечает подтверждение как потенциально опасное; точное оформление принадлежит frontend | Default |
| `ResultHandlingSuccessBehavior` | Обработка успешного результата | `Enum` | Нет | `None`, `ShowMessage`, `CloseHost`, `RefreshHost`, `Navigate`; коды `ConfigurationActionResultBehaviorCodes` | Определяет реакцию потребителя после успеха | Default; при `Navigate` требуется `ResultHandlingNavigateToViewCode` |
| `ResultHandlingErrorBehavior` | Обработка ошибки | `Enum` | Нет | `ShowMessage`, `StayOpen`, `OpenDetails`; коды `ConfigurationActionResultBehaviorCodes` | Определяет реакцию потребителя после ошибки | Default |
| `ResultHandlingSuccessMessageCode` | Код сообщения об успехе | `ReferenceCode` | Нет | Код сообщения; каталог schema не задан | Выбирает сообщение успешного результата; не является свободным текстом | Default |
| `ResultHandlingErrorMessageCode` | Код сообщения об ошибке | `ReferenceCode` | Нет | Код сообщения; каталог schema не задан | Выбирает сообщение ошибки; не является свободным текстом | Default |
| `ResultHandlingNavigateToViewCode` | Представление после успеха | `ReferenceCode` | Нет | `Catalog.Views` | Выбирает `View` для перехода после успешного результата | Только при `ResultHandlingSuccessBehavior = Navigate` |
| `InputSurfaceKind` | Способ ввода | `Enum` | Нет | `None`, `ParametersForm`, `View`, `Wizard`; коды `ConfigurationActionInputSurfaceKindCodes` | Выбирает, нужен ли ввод, форма параметров, представление или пошаговый сценарий | `InheritedOnly`; в русском тексте используется «способ ввода» |
| `InputSurfaceTitle` | Заголовок способа ввода | `LocalizedText` | Нет | Локализованные значения | Заголовок формы или представления ввода | Default |
| `InputSurfaceOpenMode` | Способ открытия ввода | `Enum` | Нет | `Inline`, `Dialog`, `Drawer`, `Navigate`; коды `ConfigurationActionInputSurfaceOpenModeCodes` | Определяет, как открыть форму или представление ввода | Default |
| `InputSurfaceViewCode` | Представление для ввода | `ReferenceCode` | Нет | `Catalog.Views`; при `InputSurfaceKind = View` ссылка на представление ввода | Выбирает существующее `View` для ввода или выбора параметров | Обязательно при `View`, запрещено при `None`, `ParametersForm`, `Wizard` |
| `InputSurfaceSelectionMode` | Режим выбора во вводе | `Enum` | Нет | `None`, `Single`, `Multiple`; коды `ConfigurationActionInputSurfaceSelectionModeCodes` | Ограничивает количество выбираемых элементов в способе ввода | `InheritedOnly` |
| `InputSurfaceConfirmActionText` | Текст подтверждения ввода | `LocalizedText` | Нет | Локализованные значения | Подпись команды подтверждения введённых параметров | Default |

### 4.3. Вложенный тип `ActionResultBindings`

Коллекция `ResultBindings[]` содержит не более одного узла
`ActionResultBindings`. Узел описывает перенос результата формы, выбора или
контекста в параметры `Action`; он не описывает DTO результата обработчика.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Mode` | Режим связи результата | `Enum` | Нет | `FormValue`, `ViewState`, `Custom`; коды `ConfigurationActionResultBindingModeCodes` | Определяет, связывается ли результат с формой, состоянием `View` или пользовательским обработчиком | Default |
| `FieldBindings` | Связи полей | `Json` | Нет | JSON-карта `код поля источника -> код параметра`; формат schema не задаёт | Переносит значения полей ввода в параметры действия | Если есть связи полей |
| `CollectionBindings` | Связи коллекций | `Json` | Нет | JSON-карта `код коллекции источника -> код параметра`; формат schema не задаёт | Переносит выбранные коллекции в параметры действия | Если есть связи коллекций |
| `ContextBindings` | Связи контекста | `Json` | Нет | JSON-карта `код контекста источника -> код параметра`; формат schema не задаёт | Переносит значения контекста в параметры действия | Если есть связи контекста |

### 4.4. Вложенный тип `ActionParameter`

Каждый узел `Parameters[]` имеет код параметра и обязательное свойство
`MemberType`. Оно определяет применимые свойства:

- `Scalar` — простое типизированное значение; разрешены `DataType`,
  `DefaultValue`, `ValueSetCode`, `SystemEnumCode`;
- `Reference` — ссылка на объект или внешний источник; требуется
  `ReferenceSourceCode`, разрешён `LookupViewCode`;
- `Collection` — коллекция объектов; требуется `ItemObjectTypeCode`, разрешены
  `SelectionMode` и `LookupViewCode`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название параметра | `LocalizedText` | Нет | Локализованные значения | Подпись параметра в форме и редакторе | Все варианты; Default |
| `Description` | Описание параметра | `LocalizedText` | Нет | Локализованные значения | Поясняет назначение параметра | Все варианты; Default |
| `MemberType` | Вид параметра | `Enum` | Да | `Scalar`, `Reference`, `Collection`; статический каталог типов параметров | Выбирает простое значение, ссылку или коллекцию и определяет применимые свойства | Дискриминатор; после создания не должен меняться без отдельной операции |
| `DataType` | Тип простого значения | `Enum` | Для `Scalar` — да | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; статический каталог типов данных | Определяет формат и базовые проверки scalar-параметра | Только `Scalar` |
| `Required` | Обязательность параметра | `Bool` | Нет | `true`, `false` | `true` требует значение перед выполнением; `false` допускает отсутствие значения | Все варианты |
| `DefaultValue` | Значение по умолчанию | `String` | Нет | Строка; формат зависит от `DataType` и schema его не задаёт | Подставляет значение при отсутствии пользовательского ввода | Только `Scalar` |
| `ValueSetCode` | Набор значений | `ReferenceCode` | Нет | `Catalog.ValueSets` | Ограничивает варианты scalar-параметра набором `ValueSet` | Только `Scalar`; взаимоисключимо с `SystemEnumCode` |
| `SystemEnumCode` | Системное перечисление | `ReferenceCode` | Нет | `Catalog.SystemEnums` | Ограничивает варианты scalar-параметра системным перечислением | Только `Scalar`; взаимоисключимо с `ValueSetCode` |
| `ReferenceSourceCode` | Источник ссылочных значений | `ReferenceCode` | Для `Reference` — да | Источник ссылок; каталог и формат кода schema не задаёт | Определяет, откуда получить связанные значения | Только `Reference` |
| `LookupViewCode` | Представление выбора | `ReferenceCode` | Нет | `Catalog.Views`, фильтр `ViewType = LookupListView` | Выбирает представление для выбора reference или collection значения | Только `Reference` и `Collection` |
| `ItemObjectTypeCode` | Тип элементов коллекции | `ReferenceCode` | Для `Collection` — да | `Catalog.ObjectTypes` | Определяет тип объектов внутри collection-параметра | Только `Collection` |
| `SelectionMode` | Режим выбора параметра | `Enum` | Нет | `None`, `Single`, `Multiple`; коды `ConfigurationActionSelectionModeCodes` | Определяет отсутствие выбора, один или несколько элементов | Только `Collection` |

### 4.5. Вложенный тип `ActionLocalBehavior`

`ActionLocalBehavior` задаёт локальную реакцию способа ввода на условие. Он не
заменяет серверный `Rule`, проверку прав или обработчик действия.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название поведения | `LocalizedText` | Нет | Локализованные значения | Подпись поведения в редакторе | Все экземпляры; Default |
| `ConditionExpression` | Условие поведения | `String` | Нет | Строковое выражение; язык и источники контекста schema не задаёт | Определяет, когда применять поведение | Если условие задано |
| `TargetKind` | Вид цели поведения | `Enum` | Да | `Action`, `Parameter`; статический каталог вариантов | Выбирает всё действие или один параметр как цель поведения | Дискриминатор |
| `TargetCode` | Код параметра-цели | `ReferenceCode` | Для `Parameter` — да | `Action.Parameters` | Указывает параметр, к которому применяется поведение | Только `Parameter`; запрещён для `Action` |
| `EffectKind` | Вид эффекта | `Enum` | Да | `SetVisible`, `SetEnabled`, `Warn`; источник — статический каталог `ActionLocalBehavior.ActionTargetEffectKind` | Меняет видимость, доступность или показывает предупреждение | Все экземпляры; точный эффект принадлежит frontend-контракту |
| `EffectValue` | Значение эффекта | `Json` | Нет | JSON; форма зависит от `EffectKind`, schema её не задаёт | Передаёт параметр эффекта, если он нужен | Для эффектов, которым требуется параметр |
| `ReasonCode` | Код причины | `String` | Нет | Строковый код; каталог schema не задаёт | Классифицирует причину поведения для потребителя или диагностики | Если потребитель использует классификацию |
| `Order` | Порядок поведения | `Number` | Нет | Число | Определяет порядок применения локальных поведений | Все экземпляры; используется для упорядоченной коллекции |

### 4.6. Вложенный тип `ActionRuleBinding`

`ActionRuleBinding` подключает существующий `Rule`. Логика правила остаётся у
владельца `Rule`; этот узел хранит ссылку, цель и точку применения.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название привязки | `LocalizedText` | Нет | Локализованные значения | Подпись привязки в редакторе | Все экземпляры; Default |
| `RuleCode` | Код правила | `ReferenceCode` | Да | Ссылка на `Rule`; каталог schema не задан | Выбирает правило, которое должно применяться к действию или параметру | Все экземпляры |
| `TargetKind` | Вид цели правила | `Enum` | Нет | `Action`, `Parameter`; статический каталог вариантов | Определяет, относится ли правило ко всему действию или параметру | Если задано; Default |
| `TargetCode` | Код параметра-цели | `ReferenceCode` | Для `Parameter` — да | `Action.Parameters` | Указывает параметр, к которому относится правило | Только `Parameter`; запрещён для `Action` |
| `ExecutionPoint` | Точка применения правила | `Enum` | Нет | Код точки; каталог допустимых значений schema не задаёт | Указывает момент, в который потребитель должен применить правило | Если задано; точный контракт принадлежит Rule/runtime |
| `Enabled` | Привязка включена | `Bool` | Нет | `true`, `false` | `true` включает привязку; `false` отключает её без удаления узла | Все экземпляры |
| `Order` | Порядок привязки | `Number` | Нет | Число | Определяет порядок нескольких привязок | Все экземпляры |

### 4.7. Вложенный тип `ActionMessage`

`ActionMessage` хранит локализованный текст или ссылку на шаблон сообщения.
Каталог ролей и каталог шаблонов текущая schema не объявляет.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название сообщения | `LocalizedText` | Нет | Локализованные значения | Подпись сообщения в редакторе | Все экземпляры; Default |
| `Role` | Роль сообщения | `Enum` | Нет | Код роли; каталог и значения schema не задаёт | Позволяет потребителю различать назначение сообщения | Если потребитель поддерживает роль |
| `Text` | Текст сообщения | `LocalizedText` | Нет | Локализованные значения | Сообщение, показываемое потребителю | Если используется текст напрямую |
| `TemplateCode` | Код шаблона сообщения | `ReferenceCode` | Нет | Код шаблона; каталог и рендерер schema не задаёт | Выбирает шаблон формирования сообщения | Если потребитель поддерживает шаблон |

### 4.8. Связи и структурные правила

Ключевые связи структуры `Action`:

- `ResultBindings` содержит не более одного `ActionResultBindings`;
- `ActionParameter.MemberType` определяет обязательные и разрешённые свойства
  параметра;
- `ActionLocalBehavior.TargetKind` и `ActionRuleBinding.TargetKind` определяют
  обязательность и запрет `TargetCode`;
- `InputSurfaceViewCode` ссылается на `View`, а
  `ActionParameter.LookupViewCode` — только на `LookupListView`;
- `ResultHandlingNavigateToViewCode` ссылается на `View`;
- `ValueSetCode` и `SystemEnumCode` взаимоисключимы;
- дочерние ссылки на `Action.Parameters` должны указывать на параметр текущего
  `Action`;
- `Order` упорядочивает элементы только своей коллекции.

## 5. Значения и источники

Таблицы ниже являются читаемым представлением текущей schema и её каталогов,
а не вторым источником истины. Типы, свойства и коллекции сверяются с
`<Artifact>ArtifactSchemas`, `ConfigurationSchemaPropertyCodes` и
`ConfigurationSchemaCollectionCodes` ([schema][schema], [коды свойств][property-codes],
[коды коллекций][collection-codes]). Технические значения сверяются с классом
`ConfigurationStaticValueCodes` и `ArtifactStaticOptionCatalog`
([коды статических значений][static-value-codes], [каталог вариантов][static-options]).
Динамические источники, их зависимости и режим обновления сверяются с
`ArtifactValueSourceCodes` и `ArtifactValueSourceDefinition`
([коды источников][value-source-codes], [описания источников][value-source-definition]).
Русский и английский смысл сверяется с `ConfigurationSchemaMetadataCatalog`
([каталог метаданных редактора][metadata-catalog]).

Значения в `Action` получают из трёх видов источников:

1. статические backend-каталоги, например `Style` и допустимые эффекты
   `ActionLocalBehavior`;
2. эффективные каталоги опубликованных артефактов, например `ObjectTypes`,
   `Views`, `ValueSets` и `SystemEnums`;
3. коды и строки, формат которых текущая schema только хранит и не расширяет,
   например `AvailableInModes`, `ExecutionPoint`, `Role` и `TemplateCode`.

| Свойство или группа | Источник | Как обновляется | Ограничение |
| --- | --- | --- | --- |
| `Style` | Статический каталог backend | Обновляется вместе с версией backend | Только `Primary`, `Secondary`, `Danger`, `Ghost` |
| `ActionLocalBehavior.TargetKind` | Статический каталог backend | Обновляется вместе с версией backend | Только `Action`, `Parameter` |
| `ActionLocalBehavior.EffectKind` | Статический источник `ActionLocalBehavior.ActionTargetEffectKind` | Обновляется вместе с версией backend | Только `SetVisible`, `SetEnabled`, `Warn` для `Action` |
| `ObjectTypeCode`, `ItemObjectTypeCode` | `Catalog.ObjectTypes` | Server snapshot effective catalog | Ссылка должна разрешаться в доступной конфигурации |
| `ValueSetCode` | `Catalog.ValueSets` | Server snapshot effective catalog | Только scalar-параметр |
| `SystemEnumCode` | `Catalog.SystemEnums` | Server snapshot effective catalog | Только scalar-параметр; взаимоисключимо с `ValueSetCode` |
| `InputSurfaceViewCode`, `ResultHandlingNavigateToViewCode` | `Catalog.Views` | Server snapshot effective catalog | Ссылка на допустимое `View`; для ввода действует условие `InputSurfaceKind` |
| `ActionParameter.LookupViewCode` | `Catalog.Views` с фильтром `ViewType = LookupListView` | Server snapshot effective catalog | Только `Reference` и `Collection` |
| `ActionLocalBehavior.TargetCode`, `ActionRuleBinding.TargetCode` | `Action.Parameters` текущего узла | Snapshot текущего документа | Только при цели `Parameter` |
| `ActionType`, invocation- и result-коды | `ConfigurationStaticValueCodes` | Обновляется вместе с контрактом backend | Текущая schema не подключает отдельный dynamic options-каталог |
| `AvailableInModes`, `ExecutionPoint`, `Role`, `TemplateCode`, `ReferenceSourceCode` | Значение из контракта потребителя | Способ обновления не задан schema | Нельзя считать свободной строкой без контракта владельца |

Текущая schema `Action` не объявляет источник с режимом `Client`, `Remote` или
`Hybrid`. Клиентский кэш или локальная проекция редактора не становится
источником истины. Если для конкретного значения появится клиентское или
удалённое обновление, его нужно описать в контракте владельца и добавить в
этот раздел как отдельное решение.

## 6. Ограничения и зависимости

Schema-правила `Action`:

- при `InputSurfaceKind = View` свойство `InputSurfaceViewCode` обязательно;
- при `InputSurfaceKind = None`, `ParametersForm` или `Wizard` свойство
  `InputSurfaceViewCode` запрещено;
- при `ResultHandlingSuccessBehavior = Navigate` свойство
  `ResultHandlingNavigateToViewCode` обязательно;
- для `ActionParameter` `DataType` обязателен и разрешён только при
  `MemberType = Scalar`;
- `DefaultValue`, `ValueSetCode` и `SystemEnumCode` разрешены только при
  `MemberType = Scalar`;
- `ReferenceSourceCode` обязателен и разрешён только при
  `MemberType = Reference`;
- `LookupViewCode` разрешён при `MemberType = Reference` или `Collection`;
- `ItemObjectTypeCode` обязателен и разрешён только при
  `MemberType = Collection`;
- `SelectionMode` разрешён только при `MemberType = Collection`;
- `ValueSetCode` и `SystemEnumCode` нельзя задавать одновременно;
- для `ActionLocalBehavior` и `ActionRuleBinding` `TargetCode` обязателен при
  `TargetKind = Parameter` и запрещён при `TargetKind = Action`.

Наличие обработчика, права, состояние workflow и предметная доступность не
проверяются этими schema-правилами. Их проверяет runtime или владелец
соответствующего контракта.

## 7. Операции над структурой

Матрица ниже фиксирует только операции, объявленные schema для коллекций. Она
не является перечнем API-команд и не определяет права пользователя.

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| Корневой `Action` | Через общий authoring/baseline-контракт | Через общий контракт записи | Через общий lifecycle Configuration | Не применяется | Конкретные API-команды и права описываются в `03_contracts.md` |
| `ResultBindings` | Не объявлено схемой | Да | Не объявлено схемой | Не применяется | Максимум один узел; `ActionNodeBuilder` поддерживает его создание при authoring |
| `Parameters` | Не объявлено схемой | Не объявлено схемой | Не объявлено схемой | Да | Перестановка параметров; создание и удаление требуют отдельного editor/API-контракта |
| `LocalBehaviors` | Да | Не объявлено схемой | Поддерево | Да | Упорядоченная коллекция; операции доступны только для узлов текущего `Action` |
| `RuleBindings` | Не объявлено схемой | Да | Не объявлено схемой | Не применяется | Изменяется привязка, а не логика `Rule` |
| `Messages` | Не объявлено схемой | Да | Не объявлено схемой | Не применяется | Изменяется сообщение, а не рендерер или каталог шаблонов |
| Свойство любого узла | Не применяется | Через общий контракт записи | Не применяется | Не применяется | С учётом обязательности, зависимостей и состояния версии |

## 8. Наследование и рассчитанный результат

Полная модель наследования `Action` и отдельный `ActionEffectiveModel` текущей
schema не подтверждены. Поэтому `InheritedOnly` у отдельных свойств означает
только ограничение переопределения, заданное схемой; оно не доказывает, что для
`Action` уже существует самостоятельная цепочка наследования.

Опубликованный или эффективный `Action` является входом для frontend и runtime.
Результат materialization, runtime-доступность, выбранный handler и итоговый
payload не являются новым конфигурационным артефактом и описываются у владельцев
этих контрактов.

## 9. Создание, проверка и публикация

При создании и публикации проверяются:

- тип узла `Action` и его код `ArtifactNode.Code`;
- свойства, зарегистрированные `ActionArtifactSchemas`;
- пять дочерних коллекций и ограничение `ResultBindings[0..1]`;
- значения enum, для которых schema или статический каталог задаёт допустимые
  варианты;
- условные свойства ввода и перехода по результату;
- discriminator `ActionParameter.MemberType` и связанные ограничения;
- XOR для `ValueSetCode` и `SystemEnumCode`;
- обязательность и запрет `TargetCode` для локального поведения и rule binding;
- ссылки на доступные `ObjectType`, `View`, `ValueSet`, `SystemEnum` и параметры
  текущего действия, если соответствующий контракт это проверяет.

`ActionNodeBuilder` и baseline builder поддерживают авторинг корня, параметров,
локальных поведений, rule bindings, сообщений и result bindings. Проверка
builder-результата покрывает scalar-параметр, локальное поведение и привязку
правила. ([тест builder][builder-tests], [тесты baseline][baseline-tests])

Публикация `Action` выполняется общим lifecycle Configuration. Она делает
конфигурацию доступной потребителям, но не регистрирует автоматически handler,
не выдаёт право на выполнение и не исполняет действие.

При runtime-вызове object-bound действия запрос содержит `ModuleCode`,
`ObjectTypeCode`, `Id`, `ActionCode`, необязательный `ViewCode`, параметры,
`ExecutionProfileCode` и `WorkflowCode`. Ответ содержит `ActionCode`,
validation и необязательный result. ([runtime-запрос][runtime-action-request],
[runtime-ответ][runtime-action-response])

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `Action` и пять дочерних schema зарегистрированы в schema registry;
- identity корневого узла задаётся `ArtifactNode.Code`, отдельного свойства
  `ActionCode` нет;
- корневые свойства, дочерние узлы и условные schema-правила перечислены в
  этом документе по текущему коду;
- `ActionNodeBuilder` и `ActionBaselineBuilder` поддерживают authoring-модель;
- `ActionCode` может ссылаться на runtime-дескриптор зарегистрированной
  object-bound операции;
- `Action` может ссылаться на `ObjectType`, `View`, `ValueSet`, `SystemEnum`,
 `Rule` и свои `Parameters`.
- Runtime materializer проецирует из effective `Action`
  настройки invocation, confirmation и result handling, а `ActionMessage` может
  разрешаться через platform message resolver. Это runtime projection и разрешение
  сообщений; они не добавляют новые свойства schema и не делают Configuration
  владельцем исполнения действия. ([runtime-action-materializer][runtime-action-materializer])

### За пределами текущего schema-контракта `Action`

- полный контракт handler, command DTO, результата и обязательной
  `ActionAvailability`;
- полный каталог `ExecutionPoint`, `ActionMessage.Role`, `TemplateCode`,
  `ReferenceSourceCode` и режимов `AvailableInModes`;
- рендерер, общий способ ввода и границы приложений `Admin`, `Studio`,
  `Runtime`;
- выполнение workflow, транзакция, audit, outbox и гарантии доставки;
- полная модель наследования и effective merge для `Action`;
- формат JSON-карт `FieldBindings`, `CollectionBindings` и `ContextBindings`.

Это границы документа, а не утверждение, что перечисленных возможностей не
будет. Открытые решения и владельцы фиксируются в [трассировке Configuration](../90_traceability.md):
`CFG-DEC-05` — граница Frontend Studio, `CFG-DEC-07` — Audit и изменения,
`CFG-DEC-10` — query и runtime-контракт `View`. Граница исполнения Workflow и
Rules закреплена в `CFG-DEC-06`; этот документ не дублирует её алгоритмы.

Старые материалы Action не являются отдельным источником истины. Их сведения
о schema, параметрах, вводе, result bindings, локальных поведениях, rule
bindings и сообщениях перенесены в раздел 4 этого документа. Сведения о
размещении во `View`, runtime-исполнении, workflow, frontend и аудите остаются
у владельцев соответствующих документов.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Действие | Action | Корневой конфигурационный артефакт операции. |
| Код действия | Action code / `ArtifactNode.Code` | Идентичность корневого узла `Action`; не свойство schema `ActionCode`. |
| Бизнес-операция | Business Action | Исполняемая операция, зарегистрированная в runtime-контракте; конфигурация `Action` не содержит её handler. |
| Команда workflow | Workflow Command / `CommandCode` | Команда управления переходом workflow; не то же самое, что `ActionCode`. |
| Способ ввода | Input method / `InputSurface*` | Способ получения входных значений перед выполнением действия. |
| Параметр действия | Action parameter / `ActionParameter` | Дочерний узел, описывающий один входной параметр. |
| Связь результата | Result binding / `ActionResultBindings` | Правило переноса результата формы, выбора или контекста в параметры действия. |
| Локальное поведение | Local behavior / `ActionLocalBehavior` | Локальная реакция способа ввода на условие. |
| Связь с правилом | Rule binding / `ActionRuleBinding` | Ссылка на `Rule`, его цель и точку применения. |
| Сообщение действия | Action message / `ActionMessage` | Локализованный текст или ссылка на шаблон сообщения. |
| Режим выбора | Selection mode | `None`, `Single` или `Multiple` для контекста вызова или параметра. |

Общие термины Configuration ведутся в [тематическом глоссарии][configuration-terms].
Локальная таблица нужна для чтения этого документа и не является отдельным
источником истины.

## 12. Источники в коде и тестах

- [Action schemas][schemas]
- [Коды свойств schema][property-codes]
- [Статический каталог вариантов][static-options]
- [Статические коды значений][static-value-codes]
- [Action node builder][builder]
- [Action baseline builder][baseline-builder]
- [Configuration action registration][registration]
- [Object Runtime descriptor][runtime-descriptor]
- [Object Runtime contract][runtime-contract]
- [Runtime action request][runtime-action-request]
- [Runtime action response][runtime-action-response]
- [Schema registry tests][schema-tests]
- [Action builder tests][builder-tests]
- [Baseline authoring tests][baseline-tests]

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
