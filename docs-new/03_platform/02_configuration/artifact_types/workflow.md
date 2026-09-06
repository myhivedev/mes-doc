---
id: DOC-03-02-AT-WORKFLOW
title: 'Тип конфигурационного артефакта — Workflow'
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

# Тип конфигурационного артефакта — Workflow

[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/WorkflowArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/WorkflowArtifactSchemas.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[value-sources]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[node-builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/WorkflowNodeBuilder.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/WorkflowBaselineBuilder.cs
[runtime-definition]: ../../../../src/Platform/DMP.Platform.Workflow/Application/Abstractions/IWorkflowDefinitionResolver.cs
[runtime-service]: ../../../../src/Platform/DMP.Platform.Workflow/Application/Services/WorkflowRuntimeService.cs
[runtime-controller]: ../../../../src/Platform/DMP.Platform.Workflow/Api/Controllers/WorkflowRuntimeController.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ConfigurationBaselineAuthoringDslIntegrationTests.cs
[runtime-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Workflow/WorkflowDurableFoundationIntegrationTests.cs
[configuration-terms]: ../../../11_glossary/configuration_terms.md
[traceability]: ../90_traceability.md

## 1. Назначение

`Workflow` — корневой конфигурационный артефакт, который описывает допустимый
жизненный цикл объекта: состояния, команды, переходы, привязки правил, точки
выполнения и ограничения состояния.

Этот документ описывает schema-контракт и жизненный цикл конфигурации
`Workflow`. Исполнение команды, хранение текущего состояния объекта, историю и
конкретный пользовательский интерфейс описывают документы владельцев этих
возможностей.

## 2. Идентичность и граница

### 2.1. Идентичность

Корневой узел идентифицируется полем `ArtifactNode.Code`. В контексте этого
артефакта код называется `WorkflowCode`; поле `ArtifactNode.ArtifactTypeCode`
имеет значение `Workflow`.

Каждый дочерний узел имеет собственный код в `ArtifactNode.Code`. В контексте
коллекции это `StateCode`, `CommandCode`, `TransitionCode`, `BindingCode`,
`ExecutionPointCode` или `PolicyCode`. Ни один из этих кодов не является
schema-свойством.

`ModuleCode` и `ObjectTypeCode` также могут входить в ключ baseline и runtime
контекст. В schema `Workflow` сохранено только свойство `ObjectTypeCode`;
`ModuleCode` не является свойством узла Workflow.

### 2.2. Входит в документ

- schema и структура корневого `Workflow`;
- свойства шести дочерних типов и связи между ними;
- источники допустимых значений и правила их применения;
- операции над узлами в Configuration;
- проверки перед публикацией и границы runtime-исполнения.

### 2.3. Не входит в документ

- API и алгоритм исполнения команды Workflow;
- хранение текущего состояния экземпляра и история переходов;
- форма или кнопка на конкретном `View`;
- обработчик `Action`, вызываемый бизнес-операцией;
- полноценные `Guards[]` и `Steps[]`, которых нет в текущей schema;
- UI редактора Workflow и общий frontend shell.

## 3. Место в Configuration

`Workflow` зарегистрирован в schema registry как корневой артефакт с шестью
коллекциями: `States`, `Commands`, `Transitions`, `RuleBindings`,
`ExecutionPoints` и `StatePolicies`. ([schema][schemas], [коды коллекций][collection-codes])

`WorkflowNodeBuilder` создаёт каноническое дерево, а
 `WorkflowBaselineBuilder` создаёт записи базового пакета конфигурации. Оба сборщика
поддерживают те же шесть коллекций, что и schema. ([канонический сборщик][node-builder],
[сборщик baseline][baseline-builder])

### 3.1. Владение по слоям

| Слой | Ответственность | Где описывается |
| --- | --- | --- |
| Configuration | schema, каноническое дерево, baseline, версии, эффективная конфигурация и публикация | Этот документ и документы Configuration |
| Workflow runtime | выбор определения, проверка команды, переход состояния, политики, закрепление revision и история исполнения | Документы Workflow runtime; текущий контракт — [runtime definition][runtime-definition] |
| Object Runtime | объект и поле, в котором хранится текущее состояние | Документы Object Runtime |
| Action | бизнес-операция, если она вызывается отдельной командой или шагом в будущей модели | [Action](action.md) и документ владельца операции |
| фронтенд-платформа и приложения | отображение доступных команд и состояний, редактор Workflow | Документы фронтенд-платформа и приложений |

## 4. Структура и схема `Workflow`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств. Вложенных коллекций внутри
дочерних узлов нет.

```text
Workflow (ArtifactNode)
├── identity: ArtifactTypeCode = Workflow; Code = <WorkflowCode>
├── properties: { Title, Description, ObjectTypeCode, ... }
└── child collections:
    ├── States[0..n] → WorkflowState (Code = <StateCode>)
    ├── Commands[0..n] → WorkflowCommand (Code = <CommandCode>)
    ├── Transitions[0..n] → WorkflowTransition (Code = <TransitionCode>)
    ├── RuleBindings[0..n] → WorkflowRuleBinding (Code = <BindingCode>)
    ├── ExecutionPoints[0..n] → WorkflowExecutionPoint (Code = <ExecutionPointCode>)
    └── StatePolicies[0..n] → WorkflowStatePolicy (Code = <PolicyCode>)
```

`Transitions[]` ссылается на `States[]` и `Commands[]`; `RuleBindings[]`
ссылается на `Rule`, состояния, переходы, команды и точки выполнения;
`StatePolicies[]` ссылается на состояние и, в зависимости от `TargetKind`,
на объект, операцию, поле, секцию или `View`.

### 4.2. Свойства корневого узла

Ниже приведён полный каталог свойств, зарегистрированных в текущей схеме
`Workflow`. `Required` означает обязательность свойства в схеме; это не всегда
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
| `Title` | Название workflow | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Отображаемое имя модели жизненного цикла | Редактируется как обычное свойство в разрешённом слое |
| `Description` | Описание workflow | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Поясняет назначение модели | Редактируется как обычное свойство в разрешённом слое |
| `ObjectTypeCode` | Код типа объекта | `ReferenceCode` | Нет в schema; для object-bound workflow требуется при публикации | `Catalog.ObjectTypes` | Определяет тип объекта, жизненный цикл которого описывает workflow | Задаётся при создании или baseline; изменение должно проверяться как изменение привязки |

В текущем baseline-контексте `Workflow` строится для пары `ModuleCode` и
`ObjectTypeCode`. Runtime дополнительно получает `ModuleCode`, `ObjectTypeCode`
и `WorkflowCode`; это контекст выбора определения, а не три свойства корневого
узла.

### 4.3. Вложенный тип `WorkflowState`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название состояния | `LocalizedText` | Нет; fallback runtime — код состояния | Локализованные строки из кода или baseline | Текст состояния для пользователя | Используется при показе текущего состояния и выборе перехода |
| `Description` | Описание состояния | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Поясняет смысл состояния | Не меняет алгоритм перехода |
| `IsInitial` | Начальное состояние | `Bool` | Нет; default `false` | `true` — состояние выбирается при инициализации; `false` — не выбирается | Определяет старт состояния экземпляра | В effective workflow должна быть ровно одна начальная точка |
| `IsFinal` | Конечное состояние | `Bool` | Нет; default `false` | `true` — штатных исходящих переходов может не быть; `false` — состояние не помечено конечным | Помогает runtime и UI определить завершение жизненного цикла | Проверяется при публикации и исполнении |
| `IsArchiveState` | Архивное состояние | `Bool` | Нет; default `false` | `true` — состояние считается архивным; `false` — обычное состояние | Даёт runtime и потребителю признак архивного результата | Не заменяет отдельную операцию архивирования |
| `Order` | Порядок состояния | `Number` | Нет; default не задан | Число; меньшие значения идут раньше | Определяет порядок показа в списке | Используется для упорядочивания `States[]` |

### 4.4. Вложенный тип `WorkflowCommand`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название команды | `LocalizedText` | Нет; fallback runtime — код команды | Локализованные строки из кода или baseline | Текст команды для пользователя | Используется в списке доступных команд |
| `Description` | Описание команды | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Поясняет назначение команды | Не определяет сам переход |
| `RequiresConfirmation` | Требуется подтверждение | `Bool` | Нет; default `false` | `true` — перед выполнением нужно подтверждение; `false` — отдельное подтверждение не требуется | Влияет на UX и runtime-проверку входного запроса | Применяется при вызове команды |
| `RequiresPayload` | Команда ожидает входные данные | `Bool` | Нет; default `false` | `true` — команда помечена как ожидающая входные данные; `false` — такой признак не установлен | Передаёт признак в runtime-контракт доступной команды; текущий runtime не отклоняет запрос только из-за отсутствия входных данных | Заполняется в baseline или builder; связь с `PayloadSchemaCode` не проверяется текущим кодом |
| `PayloadSchemaCode` | Код схемы входных данных | `ReferenceCode` | Нет; default не задан | Строковый код, например `ProductDefinition:NomenclatureSubmitPayload`; каталог и разрешение кода в текущем контракте не заданы | Передаёт идентификатор схемы как метаданные команды; текущий runtime не проверяет по нему `ExecuteWorkflowCommandRequest.Parameters` | Заполняется в baseline или builder; фактическая схема и валидатор требуют отдельного контракта |
| `PrimaryHint` | Признак основной команды | `Bool` | Нет; default `false` | `true` — подсказка UI считать команду основной; `false` — такой подсказки нет | Влияет только на предпочтительное отображение | UI может учитывать признак, runtime не должен полагаться на него |
| `Order` | Порядок команды | `Number` | Нет; default не задан | Число; меньшие значения идут раньше | Определяет порядок показа команд | Используется для упорядочивания `Commands[]` |

### 4.5. Вложенный тип `WorkflowTransition`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название перехода | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Поясняет переход | Используется в редакторе и диагностике |
| `FromStateCode` | Код исходного состояния | `ReferenceCode` | Да | `Workflow.States` | Состояние, из которого разрешён переход | Ссылка должна разрешаться внутри этого workflow |
| `ToStateCode` | Код целевого состояния | `ReferenceCode` | Да | `Workflow.States` | Состояние, в которое переводится объект | Ссылка должна разрешаться внутри этого workflow |
| `CommandCode` | Код команды | `ReferenceCode` | Да | `Workflow.Commands` | Команда, которая выбирает этот переход | Ссылка должна разрешаться внутри этого workflow |
| `Order` | Порядок перехода | `Number` | Нет; default не задан | Число; меньшие значения идут раньше | Упорядочивает переходы при отображении и детерминированном выборе | Приоритет как отдельное свойство текущей schema не задан |

### 4.6. Вложенный тип `WorkflowRuleBinding`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название привязки правила | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Понятное имя привязки | Используется в редакторе и диагностике |
| `RuleCode` | Код правила | `ReferenceCode` | Да | Код зарегистрированного `Rule`; отдельный options source для свойства не задан | Определяет правило, применяемое к выбранной цели | Проверяется при публикации и исполнении |
| `TargetType` | Тип цели | `Enum` | Нет; default не задан | `Workflow`, `State`, `Transition`, `Command`, `ExecutionPoint`; статический каталог вариантов | Определяет, к какому уровню workflow относится привязка | Условно управляет обязательностью `TargetCode` и `ExecutionPointCode` |
| `TargetCode` | Код цели | `ReferenceCode` | Условно | При `State` — `Workflow.States`; при `Transition` — `Transitions`; при `Command` — `Commands`; при `ExecutionPoint` — `ExecutionPoints`; при `Workflow` запрещён | Выбирает конкретную цель, к которой относится правило | Требуется для всех целей, кроме `Workflow` |
| `ExecutionPointCode` | Код точки выполнения | `ReferenceCode` | Условно | `Workflow.ExecutionPoints` | Выбирает точку выполнения для привязки к `ExecutionPoint` | Требуется при `TargetType = ExecutionPoint`; при других значениях запрещён |
| `Order` | Порядок привязки | `Number` | Нет; default не задан | Число; меньшие значения идут раньше | Упорядочивает несколько привязок | Используется при последовательной обработке |

### 4.7. Вложенный тип `WorkflowExecutionPoint`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название точки выполнения | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Понятное имя точки | Используется в редакторе и диагностике |
| `Type` | Тип точки выполнения | `Enum` | Нет; default не задан | `BeforeTransition`, `AfterTransition`, `StateEntry`, `StateExit`, `Guard`; статический каталог вариантов | Определяет момент, к которому относится привязка правила | Исполнитель использует тип при обработке `RuleBindings[]` |
| `Order` | Порядок точки выполнения | `Number` | Нет; default не задан | Число; меньшие значения идут раньше | Определяет порядок точек одного типа | Используется для детерминированной обработки |

### 4.8. Вложенный тип `WorkflowStatePolicy`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `StateCode` | Код состояния | `ReferenceCode` | Да | `Workflow.States` | Состояние, при котором действует политика | Ссылка должна разрешаться внутри этого workflow |
| `TargetKind` | Вид цели политики | `Enum` | Да | `Object`, `Operation`, `Field`, `Section`, `View`; статический каталог вариантов | Определяет, на что распространяется эффект | Условно определяет смысл `TargetCode` |
| `TargetCode` | Код цели политики | `ReferenceCode` | Нет; для `Object` может быть `Self` | Для `Operation`, `Field`, `Section`, `View` источник ссылки определяется владельцем соответствующей capability; для объекта `Self` означает текущий объект | Выбирает объект или часть интерфейса, к которой применяется эффект | Требуется по смыслу для целей, отличных от объекта; точные проверки источников требуют runtime-контракта |
| `EffectKind` | Эффект политики | `Enum` | Да | `Available`, `Blocked`, `Visible`, `Hidden`, `Editable`, `ReadOnly`, `Required`, `Optional`, `Allowed`, `Denied`; статический каталог вариантов | Определяет доступность, видимость, редактируемость или обязательность цели | Runtime и потребитель применяют эффект к выбранной цели |
| `EffectValue` | Значение эффекта | `Json` | Нет; default не задан | JSON; форма зависит от `EffectKind` и цели | Передаёт дополнительный параметр эффекта | Заполняется только когда конкретный эффект требует значения |
| `ReasonCode` | Код причины | `String` | Нет; default не задан | Строковый код; каталог сообщений в schema не зарегистрирован | Объясняет пользователю или журналу причину ограничения | Используется для диагностики и сообщения о политике |
| `Order` | Порядок политики | `Number` | Нет; default не задан | Число; меньшие значения идут раньше | Определяет порядок обработки нескольких политик | Используется при расчёте результата |

### 4.9. Связи и структурные правила

| Связь | Правило |
| --- | --- |
| `Workflow` → `ObjectType` | `ObjectTypeCode` указывает тип объекта, для которого предназначен workflow. |
| `WorkflowTransition` → `WorkflowState` | `FromStateCode` и `ToStateCode` должны указывать состояния из `States[]` этого workflow. |
| `WorkflowTransition` → `WorkflowCommand` | `CommandCode` должен указывать команду из `Commands[]` этого workflow. |
| `WorkflowRuleBinding` → `Rule` | `RuleCode` указывает правило; само правило не становится дочерним узлом workflow. |
| `WorkflowRuleBinding` → цель | `TargetType` определяет, какой узел выбирается через `TargetCode`; для `ExecutionPoint` дополнительно используется `ExecutionPointCode`. |
| `WorkflowStatePolicy` → `WorkflowState` | `StateCode` указывает состояние, при котором действует политика. |

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

### 5.1. Общие правила

- Коды узлов создаются кодом или baseline и не являются пользовательскими
  значениями свойств.
- `LocalizedText` может содержать invariant- и языковые значения; fallback
  определяется общим контрактом локализации.
- `Bool` хранится как `true` или `false`; смысл каждого значения указан в
  строке свойства.
- `Number` задаёт порядок, но не задаёт саму логику перехода.
- `ReferenceCode` должен ссылаться на существующий узел или зарегистрированный
  код. Источник указывается отдельно для каждого свойства; отсутствие
  зарегистрированного source означает, что серверный каталог ещё не оформлен.

### 5.2. Статические каталоги и контекстные источники

Статический каталог зарегистрирован для `WorkflowRuleBinding.TargetType`,
`WorkflowExecutionPoint.Type`, `WorkflowStatePolicy.TargetKind` и
`WorkflowStatePolicy.EffectKind`. ([каталог вариантов][static-options])

Контекстные источники зарегистрированы для:

| Свойство | Источник | Что возвращает |
| --- | --- | --- |
| `Workflow.ObjectTypeCode` | `Catalog.ObjectTypes` | доступные коды типов объектов |
| `WorkflowTransition.FromStateCode`, `ToStateCode` | `Workflow.States` | состояния текущего workflow |
| `WorkflowTransition.CommandCode` | `Workflow.Commands` | команды текущего workflow |
| `WorkflowRuleBinding.TargetCode` | зависит от `TargetType` | узлы соответствующей коллекции текущего workflow |
| `WorkflowRuleBinding.ExecutionPointCode` | `Workflow.ExecutionPoints` | точки выполнения текущего workflow |
| `WorkflowStatePolicy.StateCode` | `Workflow.States` | состояния текущего workflow |

Зарегистрированные коды источников находятся в [ArtifactValueSourceCodes][value-sources].
Для `PayloadSchemaCode`, `RuleCode` и части целей `StatePolicy.TargetCode` текущая
schema не регистрирует отдельный options source; это не следует описывать как
готовый каталог UI.

## 6. Ограничения и зависимости

- `WorkflowCode` уникален в своём контексте и не меняется после создания узла.
- `ObjectTypeCode` должен ссылаться на существующий `ObjectType` для
  object-bound workflow.
- `FromStateCode`, `ToStateCode`, `CommandCode` и `StateCode` не могут ссылаться
  на узел другого workflow.
- `TargetCode` и `ExecutionPointCode` проверяются вместе с `TargetType`; нельзя
  оставлять код цели от другого варианта.
- Для `WorkflowRuleBinding` при `TargetType = Workflow` `TargetCode` и
  `ExecutionPointCode` запрещены. Для `State`, `Transition`, `Command` и
  `ExecutionPoint` `TargetCode` обязателен. Для `ExecutionPoint` обязателен
  также `ExecutionPointCode`.
- `IsInitial = true` определяет начальную точку; effective workflow не должен
  иметь две начальные точки.
- Удаление узла, на который ссылаются переходы, привязки или политики, должно
  быть отклонено либо выполнено одним согласованным change set.
- `Workflow` не должен менять поле состояния объекта напрямую вне Workflow
  runtime.

## 7. Операции над структурой

Возможности операций определяются schema-контрактом коллекции. Обычный
редактор не должен показывать операцию, которой нет в этой матрице.

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| Корневой `Workflow` | Да, если разрешён root-create | Да для `Title`, `Description`; привязка `ObjectTypeCode` проверяется отдельно | По общей политике root-артефактов | Нет | `WorkflowCode` неизменяем после создания |
| `States[]` | Да | Да | Да, при сохранении ссылочной целостности | Да | Нельзя удалить используемое состояние без изменения всех зависимостей |
| `Commands[]` | Да | Да | Да, при сохранении ссылочной целостности | Да | Нельзя удалить команду, на которую ссылается переход |
| `Transitions[]` | Да | Да | Да | Да | `FromStateCode`, `ToStateCode` и `CommandCode` должны быть разрешимы |
| `RuleBindings[]` | Нет в обычном schema-редакторе; допускается обновление существующего узла | Да | Нет | Нет | В schema разрешена только операция `Update` |
| `ExecutionPoints[]` | Нет в обычном schema-редакторе; допускается обновление существующего узла | Да | Нет | Нет | В schema разрешена только операция `Update` |
| `StatePolicies[]` | Да | Да | Да | Да | `TargetKind`, `EffectKind` и ссылки проверяются совместно |

`WorkflowNodeBuilder` и `WorkflowBaselineBuilder` могут создавать все типы
узлов программно. Это возможность поставки baseline, а не обещание такого же
CRUD в пользовательском редакторе.

## 8. Наследование и рассчитанный результат

Configuration строит effective-дерево `Workflow` по общей parent chain и
правилам merge. После расчёта runtime получает согласованное определение, а не
отдельные несвязанные draft-узлы.

При расчёте эффективной конфигурации необходимо повторно проверить:

- уникальность кодов дочерних узлов;
- разрешимость ссылок внутри итогового workflow;
- наличие единственного `IsInitial`;
- совместимость `RuleBindings[]` с их `TargetType`;
- совместимость политик с состояниями и целями.

`WorkflowDefinitionRevision` и pinning экземпляра относятся к Workflow runtime,
а не к schema-свойству `Workflow`. Текущий runtime-контракт описан в
[IWorkflowDefinitionResolver][runtime-definition].

## 9. Создание, проверка и публикация

При создании корневого `Workflow` задаются `WorkflowCode`, `Title` и
`ObjectTypeCode` в зависимости от доступного root-create сценария. Дочерние
коллекции добавляются отдельными операциями.

Перед публикацией Configuration должна проверить:

1. допустимость каждого свойства по schema и статическому каталогу;
2. обязательность `FromStateCode`, `ToStateCode`, `CommandCode`, `RuleCode`,
   `StateCode` и условных полей привязки правила;
3. существование всех внутренних ссылок и отсутствие ссылок на другой workflow;
4. корректность единственного начального состояния;
5. отсутствие удалённых узлов, на которые ссылаются оставшиеся узлы;
6. совместимость `Workflow` с `ObjectType` и его полем состояния;
7. возможность построить эффективную конфигурацию и передать её runtime.

Runtime повторно проверяет право пользователя, доступность команды и условия
перехода. Наличие команды в frontend-ответе не является разрешением на
исполнение. Реальный исполнитель команд находится в `DMP.Platform.Workflow`,
а не в schema Configuration. ([runtime service][runtime-service], [runtime controller][runtime-controller])

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- В schema зарегистрирован корневой `Workflow` с шестью дочерними коллекциями.
- Свойства, типы значений, статические варианты и контекстные источники,
  указанные в разделах 4 и 5, подтверждены текущим кодом schema.
- Canonical и baseline builders создают состояния, команды, переходы,
  привязки правил, точки выполнения и политики состояний.
- Runtime имеет отдельный контракт определения и отдельный исполнитель.

### За пределами текущего schema-контракта `Workflow`

- `Transition.Guards[]`, `Transition.Steps[]`, `State.EntrySteps[]` и
  `State.ExitSteps[]` не зарегистрированы как дочерние коллекции и не должны
  описываться как доступные узлы текущего редактора.
- Каталог и проверка `PayloadSchemaCode` не зарегистрированы в текущей schema.
- Отдельная модель выбора между несколькими workflow одного `ObjectTypeCode`
  не является свойством `Workflow`.
- Полные правила runtime guard evaluation, side effects, фонового выполнения и
  истории находятся за пределами этого документа.

Актуальные нерешённые решения и маршрут дальнейшего описания находятся в
[90_traceability.md][traceability]. Этот документ не превращает будущие
возможности в требования текущего MVP.

## 11. Термины

| Русский термин | English term | Технический код или алиас | Определение |
| --- | --- | --- | --- |
| Жизненный цикл объекта | Object lifecycle | `Workflow` | Набор состояний и разрешённых переходов объекта |
| Состояние | State | `WorkflowState` | Допустимое состояние объекта |
| Команда workflow | Workflow command | `WorkflowCommand` | Намерение запустить переход |
| Переход | Transition | `WorkflowTransition` | Связь исходного состояния, команды и целевого состояния |
| Привязка правила | Rule binding | `WorkflowRuleBinding` | Связь `Rule` с уровнем workflow |
| Точка выполнения | Execution point | `WorkflowExecutionPoint` | Именованный момент, к которому относится проверка или обработка |
| Политика состояния | State policy | `WorkflowStatePolicy` | Эффект, действующий для цели в конкретном состоянии |
| Определение workflow | Workflow definition | `WorkflowDefinition` | Runtime-представление опубликованной модели |

Русские термины должны согласовываться с [глоссарием Configuration][configuration-terms].

## 12. Источники в коде и тестах

- Schema и дочерние коллекции: [WorkflowArtifactSchemas][schemas].
- Коды свойств: [ConfigurationSchemaPropertyCodes][property-codes].
- Коды типов артефактов: [ConfigurationArtifactTypeCodes][artifact-codes].
- Статические варианты: [ArtifactStaticOptionCatalog][static-options].
- Источники значений: [ArtifactValueSourceCodes][value-sources].
- Каноническое создание: [WorkflowNodeBuilder][node-builder].
- Создание baseline: [WorkflowBaselineBuilder][baseline-builder].
- Runtime definition contract: [IWorkflowDefinitionResolver][runtime-definition].
- Runtime execution: [WorkflowRuntimeService][runtime-service] и [WorkflowRuntimeController][runtime-controller].
- Проверка authoring/baseline: [ConfigurationBaselineAuthoringDslIntegrationTests][schema-tests].
- Проверка runtime исполнения: [WorkflowDurableFoundationIntegrationTests][runtime-tests].

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
