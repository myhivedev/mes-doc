---
id: DOC-03-02-ART-NUMBERING-RULE
title: 'Тип конфигурационного артефакта — NumberingRule'
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

# Тип конфигурационного артефакта — NumberingRule

[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/NumberingArtifactSchemas.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[static-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[validator]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Services/ArtifactValidator.cs
[version-validator]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Validation/Services/ConfigurationVersionValidationService.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/ConfigurationBaselinePackageBuilder.cs
[published-resolver]: ../../../../src/Hosts/DMP.Platform.Api/Composition/BaselineNumberingRuleResolver.cs
[runtime-evaluator]: ../../../../src/Platform/DMP.Platform.Numbering/Application/Services/NumberingRuleRuntimeEvaluator.cs
[numbering-service]: ../../../../src/Platform/DMP.Platform.Numbering/Application/Services/NumberingService.cs
[lifecycle-hook]: ../../../../src/Platform/DMP.Platform.Numbering/Application/Services/NumberingObjectMutationLifecycleHook.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/NumberingRuleSchemaIntegrationTests.cs
[validation-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/NumberingPublishValidationIntegrationTests.cs
[object-type]: object_type.md
[traceability]: ../90_traceability.md

## 1. Назначение

`NumberingRule` — корневой конфигурационный артефакт, который описывает, как
сформировать значение нумеруемого строкового поля объекта. Он задаёт целевой
тип объекта и поле, момент выбора правила, условие применимости, шаблон и
параметры счётчика.

Артефакт хранит правило, но не хранит текущее состояние счётчика. Состояние
выданных номеров, атомарное выделение следующего номера и журнал выдачи принадлежат
Platform Numbering и описываются его владельцем.

## 2. Идентичность и граница

### 2.1. Идентичность

Код корневого узла `NumberingRule` (`ArtifactNode.Code`) идентифицирует правило.
В runtime для него используется составная идентичность:

```text
ModuleCode + ObjectTypeCode + FieldCode + RuleCode
```

`ModuleCode` и контекст `ObjectTypeCode` также присутствуют в
`ConfigurationEntry`. Это контекст поставки и хранения; он не превращает
технические ключи persistence в свойства schema.

### 2.2. Входит в документ

- корневой schema-контракт `NumberingRule`;
- связь с `ObjectType` и `ObjectMember`;
- условие выбора правила и разрешённые источники значений;
- шаблон номера и параметры счётчика;
- правила validation, publication и effective resolution;
- граница с Object Runtime и Platform Numbering.

### 2.3. Не входит в документ

- `NumberingCounterEntry`, текущее значение счётчика и блокировка выдачи;
- журнал выдачи номера `NumberingIssueLogEntry`;
- прикладная проверка уникальности значения в бизнес-объекте;
- object mutation pipeline и транзакция бизнес-операции;
- специализированный рендерер или отдельный UI-конструктор шаблона;
- полная runtime-семантика Numbering, которой владеет Platform Numbering.

## 3. Место в Configuration

`NumberingRule` принадлежит Configuration как schema и как опубликованная
конфигурация. При создании объекта Object Runtime передаёт контекст поля в
Numbering runtime, который читает подходящие правила и выдаёт значение.

| Владелец или слой | Ответственность | Где описано |
| --- | --- | --- |
| Configuration | Код типа, schema properties, catalog values, authoring, validation и publication | Этот документ, [schema][schema], [version validator][version-validator] |
| Object Runtime | ObjectMember metadata, mutation context и применение выданного значения к объекту | [ObjectType](object_type.md) и документ Object Runtime |
| Platform Numbering | Выбор применимого правила, счётчик, форматирование номера и журнал выдачи | [published resolver][published-resolver], [runtime evaluator][runtime-evaluator], [numbering service][numbering-service] |
| Domain module | Регистрация ObjectType/field и допустимых источников значения | Документ прикладного модуля и baseline package |
| фронтенд-платформа | Общий shell и standard artifact editor; отдельный рендерер нумерации не подтверждён | Документ фронтенд-платформа |

В текущей модели `NumberingRule` является самостоятельной записью с
`KindCode = NumberingRule`; дочерних schema-узлов у него нет.

## 4. Структура и схема `NumberingRule`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицу свойств.

```text
NumberingRule (ArtifactNode)
├── identity: ArtifactTypeCode = NumberingRule; Code = <RuleCode>
├── properties: { Title, ObjectTypeCode, FieldCode, InheritanceMode,
│                AssignmentMoment, Priority, ConditionJson, Template,
│                PartitionDefinitionJson, StartNumber, NumberingStep, IsActive }
└── child collections: none
```

У `NumberingRule` нет дочерних коллекций или отдельных узлов для условий, частей
шаблона и частей разделения счётчика. Эти данные хранятся в JSON- и строковых
свойствах.

### 4.2. Свойства корневого узла

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название правила | `LocalizedText` | Нет; schema default не задан | Локализованные значения | Показывает правило в каталоге и редакторе; не участвует в выборе правила | Все варианты; editor может использовать `Code` как начальное значение |
| `ObjectTypeCode` | Код типа объекта | `ReferenceCode` | Да; creation-only | `Catalog.ObjectTypes`; допускается квалифицированный код `ModuleCode:ObjectTypeCode` | Определяет тип объекта, к которому относится правило | Задаётся при создании; должен совпадать с контекстом entry |
| `FieldCode` | Код нумеруемого поля | `ReferenceCode` | Да | `ObjectType.Members` | Определяет поле, которому может быть присвоен номер | Задаётся при создании; поле должно быть разрешено для нумерации |
| `InheritanceMode` | Режим наследования правила | `Enum` | Нет; schema и root editor используют default `Override`; при отсутствии свойства в validation projection применяется fallback `Inherit` | `Inherit`, `Override`, `Disable`; статический каталог Configuration | Хранит выбранный режим отношения к правилам родительского слоя | Для набора правил одного поля значения должны быть согласованы; самостоятельная runtime-интерпретация режима не подтверждена; различие default и validation fallback см. в трассировке |
| `AssignmentMoment` | Момент выбора правила | `Enum` | Да; default `OnCreate` | `OnCreate`; текущий статический каталог Configuration для V1 содержит только это значение. Константа `OnSave` объявлена в общем наборе кодов, но не входит в текущий активный каталог вариантов | Определяет момент, передаваемый resolver/evaluator | Текущий object mutation hook выполняет только `OnCreate`; `OnSave` не является допустимой гарантией текущего runtime |
| `Priority` | Приоритет правила | `Number` | Нет; default `0` | Целое число; диапазон schema не задан | При одинаковом поле и моменте выбирается правило с наибольшим приоритетом | Все варианты; одинаковый приоритет при одинаковом условии даёт ambiguity error |
| `ConditionJson` | Условие применимости | `Json` | Нет; пустое значение означает отсутствие дополнительного условия | Объект с `conditions[]`; `logicalOperator`: `And` или `Or`; операторы проверяются validator/evaluator | Ограничивает набор объектов, для которых правило считается кандидатом | При наличии проверяется до выбора правила; источники полей ограничиваются metadata ObjectMember |
| `Template` | Шаблон номера | `String` | Да; пустая строка недопустима | Текст и поддержанные tokens: `Numerator`, `number`, `Numerator:000...`, `YYYY`, `YY`, `MM`, `DD`, имена источников объекта | Формирует итоговое строковое значение номера | Должен содержать ровно один `Numerator`; источники объекта должны быть разрешены ObjectMember metadata |
| `PartitionDefinitionJson` | Разделение счётчиков | `Json` | Нет; отсутствие или пустой `fields[]` означает общий partition | Объект с `fields[]` непустых строк | Формирует ключ отдельного счётчика из runtime-источников | Каждый источник должен быть разрешён ObjectMember metadata и иметь значение при выдаче номера |
| `StartNumber` | Начальное значение счётчика | `Number` | Нет; default `1` | Положительное целое число | Передаётся при создании счётчика | Используется при первой выдаче для конкретной идентичности и partition |
| `NumberingStep` | Шаг счётчика | `Number` | Нет; default `1` | Положительное целое число | Определяет увеличение последовательного номера | Используется при атомарном выделении следующего номера |
| `IsActive` | Правило активно | `Bool` | Нет; default `true` | `true`, `false` | `true` допускает выбор правила runtime; `false` исключает его из кандидатов | Все варианты; resolver использует `true` по умолчанию при отсутствии свойства |

### 4.3. Структурные правила

- `ObjectTypeCode` должен ссылаться на существующий `ObjectType` в том же
  module context, если ссылка не квалифицирована явно.
- `FieldCode` должен разрешаться в effective `ObjectMember` целевого `ObjectType`.
- У целевого поля должны быть включены `NumberingEnabled` и тип
  `Scalar/String` для текущей V1-нумерации.
- `NumberingSupportedMoments` целевого `ObjectMember` должен содержать
  `AssignmentMoment` правила.
- Источники шаблона, условия и partition должны входить соответственно в
  `NumberingAllowedSourcesJson.templateVariables`, `conditionFields` и
  `partitionFields` целевого поля.
- Для одного target field нельзя публиковать конфликтующие `InheritanceMode`.
- Несколько правил с одинаковыми `AssignmentMoment`, `Priority` и нормализованным
  условием считаются неоднозначными.

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

### 5.1. Статические каталоги

| Свойство | Источник | Допустимые значения | Обновление |
| --- | --- | --- | --- |
| `InheritanceMode` | Backend static option catalog | `Inherit`, `Override`, `Disable` | Только изменением backend-каталога и контракта |
| `AssignmentMoment` | Backend static option catalog | `OnCreate` для текущего V1; `OnSave` объявлен в `ConfigurationNumberingAssignmentMomentCodes`, но не выдан активным каталогом вариантов | Только изменением backend-каталога, validator/editor-контракта и runtime |
| `IsActive` | Schema default и runtime resolver | `true`, `false` | Значение задаётся в configuration property |

### 5.2. JSON `ConditionJson`

Текущий validator требует JSON-объект с массивом `conditions`. Допустимы
`logicalOperator = And` и `logicalOperator = Or`. В строке условия должны быть
непустые `field` и поддержанный `operator`.

Validator знает следующие операторы:
`Equals`, `NotEquals`, `In`, `IsNull`, `IsNotNull`, `Contains`, `StartsWith`,
`GreaterThan`, `GreaterOrEqual`, `GreaterThanOrEqual`, `LessThan`, `LessOrEqual`,
`LessThanOrEqual`.

Текущий `NumberingRuleRuntimeEvaluator` фактически обрабатывает только
`Equals`, `NotEquals`, `In`, `IsNull`, `IsNotNull`, `Contains` и `StartsWith`.
Остальные операторы проходят отдельную validator-проверку, но не подтверждены
текущим runtime-исполнением. Это расхождение зафиксировано в
[трассировке Configuration][traceability].

### 5.3. JSON `PartitionDefinitionJson`

Формат V1:

```json
{
  "fields": ["YYYY", "Group"]
}
```

Validator проверяет объект, массив `fields` и непустые строковые элементы.
Version validation дополнительно проверяет разрешённые источники, а runtime
формирует partition key из фактических значений. Если источник отсутствует или
пуст, выдача номера завершается ошибкой.

### 5.4. Шаблон `Template`

Поддерживаются:

- обычный текст;
- `Numerator` и совместимый token `number`;
- форматированный счётчик `Numerator:000000`;
- UTC-компоненты `YYYY`, `YY`, `MM`, `DD`;
- источники значений из `ObjectMutationContext.EffectiveValues`.

Validator требует ровно один token счётчика. Runtime заменяет остальные tokens
значениями контекста и отклоняет пустой или недоступный источник. Форматирование
осуществляется в Platform Numbering, а не в Configuration schema.

## 6. Ограничения и зависимости

`NumberingRule` зависит от effective `ObjectType` и его `ObjectMember`. В частности,
поле должно быть строковым scalar-полем с включённой нумерацией; metadata поля
задаёт поддержанные моменты и разрешённые источники.

`NumberingRule` не может самостоятельно включить нумерацию поля, изменить его
тип или разрешить источник, отсутствующий в metadata. Политики ручного ввода и
поведения при отсутствии подходящего правила являются свойствами `ObjectMember`
и описываются в [ObjectType][object-type].

Текущий runtime hook запускает автоматическую выдачу только для операции создания
объекта и пустого значения поля. Обновление объекта, отдельный ручной запрос и
`OnSave` не являются подтверждёнными гарантиями текущего Numbering MVP.

## 7. Операции над структурой

У `NumberingRule` нет дочерних коллекций, поэтому операции выполняются над
корневым узлом и его свойствами.

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| Корневой `NumberingRule` | Да, через общий authoring/editor или baseline builder | Да для свойств, разрешённых schema и lifecycle | Да по общим правилам Configuration | Не применяется | `ObjectTypeCode` creation-only; публикация требует version validation |
| Свойства корня | Да | Да, кроме creation-only контекста | Сброс значения по общей property policy | Не применяется | `Template`, `ConditionJson` и partition должны проходить соответствующие проверки |
| Дочерние узлы | Не применяется | Не применяется | Не применяется | Не применяется | Дочерняя schema-модель не зарегистрирована |

## 8. Наследование и рассчитанный результат

Published resolver читает `NumberingRule` из effective цепочки Configuration для
наиболее специфичного доступного контекста, фильтрует удалённые, неактивные,
неподходящие по полю и моменту правила и передаёт кандидатов Numbering runtime.
Кандидат с максимальным `Priority` выбирается как верхний; одинаковый приоритет
даёт `NUMBERING_RULE_AMBIGUOUS`.

`InheritanceMode` зарегистрирован schema и проверяется на согласованность правил
одного target field, но текущий published resolver не содержит отдельного
переключения поведения для каждого значения `Inherit`, `Override` и `Disable`.
При обычном создании editor и schema используют `Override`; если свойство
отсутствует в сырой записи, validation projection подставляет `Inherit`. Это
техническое расхождение default-значений, а не подтверждённое правило
наследования.
Поэтому это свойство нельзя описывать как полностью реализованный runtime-механизм
наследования без отдельного решения или дополнительного подтверждения кода.

Результатом работы Numbering runtime является выданное строковое значение и
runtime-состояние счётчика. Это не новый schema property `NumberingRule` и не
часть canonical configuration document.

## 9. Создание, проверка и публикация

### 9.1. Создание

Baseline builder и root editor session могут создать корневой `NumberingRule` с
кодом правила, целевым `ObjectTypeCode`, `FieldCode`, `AssignmentMoment` и
`Template`. Остальные значения могут быть добавлены отдельными properties.

### 9.2. Проверка

Проверка выполняется на двух уровнях:

1. `ArtifactValidator` проверяет JSON-форматы условий и partition, шаблон с
   ровно одним `Numerator`, а также положительные `StartNumber` и
   `NumberingStep`.
2. `ConfigurationVersionValidationService` проверяет существование целевого
   ObjectType и поля, контекст module/object type, `NumberingEnabled`, тип
   `Scalar/String`, supported moments, разрешённые источники и ambiguity rules.

### 9.3. Публикация и выполнение

Наличие корректной schema ещё не означает выдачу номера. После публикации
resolver должен найти правило в effective chain, а при создании объекта runtime
должен получить подходящего кандидата и передать запрос в counter store.
Сохранение `NumberingIssueLogEntry` происходит после успешной object mutation.

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `NumberingRule` зарегистрирован как root artifact и имеет 12 schema properties;
- schema, static option catalog, baseline builder, editor session и validation
  существуют в текущем коде;
- опубликованные правила разрешаются из Configuration effective chain;
- текущий object mutation hook поддерживает автоматическую нумерацию на `OnCreate`;
- counter state и issue log принадлежат Platform Numbering;
- schema и version validation покрыты релевантными integration tests.

### За пределами текущего schema-контракта `NumberingRule`

- дочерние узлы для template parts, conditions или partition parts;
- хранение `LastNumber` внутри `NumberingRule`;
- отдельный numbering editor или рендерер;
- гарантированная выдача на `OnSave`, manual request или workflow transition;
- runtime-поддержка всех операторов, которые validator принимает в `ConditionJson`;
- самостоятельная полная семантика `InheritanceMode` в published resolver.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Правило нумерации | NumberingRule | Конфигурационное описание формирования значения поля |
| Целевое поле | Target field / `FieldCode` | Поле `ObjectMember`, которому может быть присвоен номер |
| Шаблон номера | Numbering template / `Template` | Строка с текстом и tokens для формирования значения |
| Раздел счётчика | Counter partition / `PartitionDefinitionJson` | Набор источников, задающий отдельную последовательность |
| Момент присвоения | Assignment moment / `AssignmentMoment` | Код момента, передаваемый resolver и runtime |
| Состояние счётчика | Counter state | Текущее состояние последовательности, которым владеет Platform Numbering |

## 12. Источники в коде и тестах

Основные подтверждения: [artifact type codes][artifact-codes], [property codes][property-codes],
[schema][schema], [static catalog][static-catalog], [validator][validator],
[version validator][version-validator], [baseline builder][baseline-builder],
[published resolver][published-resolver], [runtime evaluator][runtime-evaluator],
[numbering service][numbering-service], [lifecycle hook][lifecycle-hook],
[schema tests][schema-tests] и [validation tests][validation-tests].

Статус и маршруты расхождений находятся в [трассировке Configuration][traceability].

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
