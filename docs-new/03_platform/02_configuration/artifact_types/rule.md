---
id: DOC-03-02-AT-RULE
title: 'Тип конфигурационного артефакта — Rule'
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

# Тип конфигурационного артефакта — Rule

[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/RuleArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/RuleArtifactSchemas.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/RuleNodeBuilder.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/RuleBaselineBuilder.cs
[runtime-gateway]: ../../../../src/Platform/DMP.Platform.Rules/Abstractions/IRuleEvaluationGateway.cs
[runtime-implementation]: ../../../../src/Hosts/DMP.Platform.Api/Composition/RuntimeRuleEvaluationGateway.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactSchemaRegistryIntegrationTests.cs
[builder-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactNodeAuthoringPipelineIntegrationTests.cs
[persistence-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactNodePersistenceIntegrationTests.cs
[configuration-terms]: ../../../11_glossary/configuration_terms.md
[traceability]: ../90_traceability.md

## 1. Назначение

`Rule` — корневой конфигурационный артефакт именованной проверки. Он хранит
описание правила, предметный контекст, параметры и выражение, результат
которого в текущем MVP имеет булев тип.

Этот документ описывает schema-контракт `Rule`, его дочерний узел
`RuleParameter`, источники значений, schema-ограничения и жизненный цикл в
Configuration. Исполнение выражения, точки вызова и применение результата
принадлежат Rules runtime или владельцу операции, которая подключила правило.

## 2. Идентичность и граница

### 2.1. Идентичность

Корневой `Rule` идентифицируется полем `ArtifactNode.Code`. В контексте этого
артефакта код является `RuleCode`. Поле `ArtifactNode.ArtifactTypeCode` имеет
значение `Rule`.

В schema нет свойства `RuleCode`, поэтому оно не включается в таблицу свойств.
Это техническая идентичность узла, на которую ссылаются `RuleBinding` у
`ObjectType`, `Action` и других владельцев.

Дочерний узел идентифицируется собственным `ArtifactNode.Code` внутри
коллекции `Parameters`. В контексте параметра этот код является
`ParameterCode`, но также не является schema-свойством.

### 2.2. Входит в документ

- schema и структура корневого `Rule`;
- свойства `Rule` и `RuleParameter`;
- источники допустимых значений и ссылки на `ObjectType`, `ValueSet` и
  `SystemEnum`;
- schema-ограничения, операции, публикация и effective-представление.

### 2.3. Не входит в документ

- движок выполнения выражений и его реализации;
- точки выполнения, `RuleBinding` и применение результата владельцем;
- обязательные предметные проверки в коде;
- UI редактора правил и frontend runtime;
- сохранение бизнес-объекта, транзакции, события и побочные эффекты.

## 3. Место в Configuration

`Rule` и `RuleParameter` зарегистрированы в schema registry как root-артефакт
и дочерний тип. Корневой узел содержит коллекцию `Parameters` с нулём или
более элементов. Группа логики в текущей schema не является дочерним узлом:
`LogicEngineType`, `LogicExpression` и `LogicReturnKind` хранятся как свойства
корня с группой редактора `Logic`. ([schema][schemas])

`RuleNodeBuilder` создаёт канонический узел. `RuleBaselineBuilder` создаёт
 запись `Rule` и записи `RuleParameter` в базовом пакете конфигурации; при импорте они
собираются в каноническое дерево. ([сборщик узла][builder], [сборщик baseline][baseline-builder])

### 3.1. Владение по слоям

| Слой | Ответственность | Где описывается |
| --- | --- | --- |
| Configuration | schema, свойства, параметры, ссылки, validation, хранение и публикация | Этот документ и документы Configuration |
| Rules runtime | оценка опубликованного правила и контракт результата | `IRuleEvaluationGateway` и область Rules; текущая минимальная реализация подключается в API host |
| Object Runtime / Action / Workflow / View | точка вызова и применение результата правила | Документ и runtime владельца |
| фронтенд-платформа | editor UX и отображение ошибок/результатов, если они нужны интерфейсу | `06_user_experience.md` и фронтенд-платформа |
| Прикладной модуль | предметный смысл конкретного правила и его обязательные инварианты | Документы доменного модуля |

`Rule` только описывает проверку. Он не выбирает репозиторий, обработчик
действия или переход workflow и не изменяет бизнес-объект напрямую.

## 4. Структура и схема `Rule`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств. Индивидуальные свойства приведены в
разделах `4.2–4.3`.

```text
Rule (ArtifactNode)
├── identity: ArtifactTypeCode = Rule; Code = <RuleCode>
├── properties: { описание, предметный контекст, LogicEngineType,
│                LogicExpression, LogicReturnKind }
└── child collections:
    └── Parameters[0..n] → RuleParameter (Code = <ParameterCode>)
        └── properties: { ... }
```

`Logic` — только группа свойств для editor-представления. Отдельного узла
`RuleLogic` и отдельной коллекции логики в текущей schema нет.

| Что это | Техническое имя в коде | Русский смысл | Где описано подробно |
| --- | --- | --- | --- |
| Идентичность корневого узла | `ArtifactNode.ArtifactTypeCode` + `ArtifactNode.Code` | Тип `Rule` и код правила | Раздел 2.1 |
| Свойства корневого узла | `ArtifactNode.Properties`, schema `Rule` | Описание, контекст, включение и логика проверки | Раздел 4.2 |
| Параметры | `Parameters` → `RuleParameter` | Входные значения правила | Раздел 4.3 |
| Группа логики | `LogicEngineType`, `LogicExpression`, `LogicReturnKind` | Способ задания выражения и ожидаемый результат | Раздел 4.2 |

### 4.2. Свойства корневого узла `Rule`

Ниже приведён полный каталог свойств, зарегистрированных в текущей схеме
`Rule`. `Required` означает обязательность свойства в схеме; это не всегда
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
| `Title` | Название правила | `LocalizedText` | Нет | Локализованные значения | Отображаемое имя правила в редакторе и каталогах; не влияет на результат | `Default` |
| `Description` | Описание правила | `LocalizedText` | Нет | Локализованные значения | Объясняет назначение правила; не является выражением | `Default` |
| `RuleType` | Тип правила | `Enum` | Нет | Только `Validation`; backend static catalog | Классифицирует правило как проверку; другие типы текущей schema не подтверждены | `Default`; проверяется при публикации |
| `ObjectTypeCode` | Код типа объекта | `ReferenceCode` | Нет | `Catalog.ObjectTypes` | Задаёт тип объекта, к контексту которого относится правило | `CreationOnly`; ссылка должна разрешаться в каталоге объектов |
| `Enabled` | Правило включено | `Bool` | Нет | `true` — правило разрешено к использованию; `false` — правило отключено; default schema не задан | Даёт общий признак доступности правила; не заменяет включение конкретного `RuleBinding` | `Default` |
| `LogicEngineType` | Механизм логики | `Enum` | Нет | Только `Expression`; backend static catalog | Указывает, что логика задаётся выражением, а не отдельным зарегистрированным типом движка | Группа `Logic`; `Default` |
| `LogicExpression` | Выражение правила | `String` | Нет | Текст выражения; язык, грамматика, лимит длины и разрешённые функции текущей schema не заданы | Содержит выражение, которое Rules runtime должен оценить в контексте запроса | Группа `Logic`; используется при `LogicEngineType = Expression` |
| `LogicReturnKind` | Вид результата логики | `Enum` | Нет | Только `Boolean`; backend static catalog | Означает, что результат выражения интерпретируется как `true` или `false` | Группа `Logic`; `Default` |

### 4.3. Дочерний узел `RuleParameter`

`RuleParameter` описывает один именованный вход правила. Его `ParameterCode`
хранится в `ArtifactNode.Code`, а сам параметр находится в коллекции
`Rule.Parameters`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название параметра | `LocalizedText` | Нет | Локализованные значения | Подпись параметра в редакторе правила и диагностике | `Default` |
| `DataType` | Тип значения параметра | `Enum` | Нет | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; backend static catalog | Определяет формат входного значения и ожидаемую обработку параметра | `Default`; используется при проверке и передаче значения в runtime |
| `Required` | Параметр обязателен | `Bool` | Нет | `true` — значение должен передать вызывающий контекст; `false` — значение может отсутствовать или использовать default | Определяет, требуется ли значение параметра при вызове правила | `Default` |
| `DefaultValue` | Значение по умолчанию | `String` | Нет | Строковое представление; преобразование к `DataType` и правила совместимости текущей schema отдельно не заданы | Может использоваться как запасное значение необязательного параметра | `Default`; применяется только если runtime поддерживает это в контракте |
| `ValueSetCode` | Код набора значений | `ReferenceCode` | Нет | `Catalog.ValueSets` | Ссылается на конфигурируемый набор допустимых значений параметра | `Default`; взаимоисключимо с `SystemEnumCode` |
| `SystemEnumCode` | Код системного перечисления | `ReferenceCode` | Нет | `Catalog.SystemEnums` | Ссылается на системный каталог допустимых значений параметра | `Default`; взаимоисключимо с `ValueSetCode` |

`ValueSetCode` и `SystemEnumCode` проверяются правилом XOR: одновременно
задать оба значения нельзя. Текущая schema не требует задать хотя бы одно из
них.

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

Для `RuleType`, `LogicEngineType`, `LogicReturnKind` и `RuleParameter.DataType`
источником списка вариантов является backend static catalog
`ArtifactStaticOptionCatalog`. Коды вариантов также определены в
`ConfigurationStaticValueCodes`. ([каталог вариантов][static-options], [коды значений][static-value-codes])

| Источник | Что предоставляет | Канал и обновление | Кто проверяет |
| --- | --- | --- | --- |
| Backend static catalog | Фиксированные варианты enum для schema | Серверный код; изменяется вместе с версией платформы | Server validation; editor может получить тот же каталог через editor contract |
| `Catalog.ObjectTypes` | Коды типов объектов для `ObjectTypeCode` | Серверный каталог конфигурации; состав меняется публикацией конфигурации | Server при проверке ссылки |
| `Catalog.ValueSets` | Коды `ValueSet` | Серверный каталог конфигурации; состав меняется публикацией конфигурации | Server при проверке ссылки; клиент использует полученный контракт |
| `Catalog.SystemEnums` | Коды `SystemEnum` | Серверный каталог конфигурации; состав меняется публикацией конфигурации | Server при проверке ссылки; клиент использует полученный контракт |
| Автор правила | `Title`, `Description`, `LogicExpression`, `DefaultValue` и значения ссылок | Вводится через editor или baseline и сохраняется Configuration | Schema validation и проверки публикации |

Frontend не является источником истины для допустимых значений `Rule`. Он
может отображать варианты, полученные из server/editor contract, но не должен
самостоятельно расширять backend catalog.

## 6. Ограничения и зависимости

- `RuleCode` уникален в своём контексте идентичности и не зависит от `Title`.
- `ParameterCode` идентифицирует параметр внутри коллекции `Parameters`.
- `ObjectTypeCode` является ссылкой на каталог объектов и задаёт контекст, но
  не копирует `ObjectType` внутрь `Rule`.
- `ValueSetCode` и `SystemEnumCode` взаимоисключающие; это единственное
  явно зарегистрированное правило дочерней schema.
- `LogicExpression` является строкой schema. Текущая schema не фиксирует язык
  выражений, доступные функции, безопасность, лимит длины или способ
  компиляции.
- `DefaultValue` хранится как строка. Текущая schema не подтверждает правила
  преобразования строки в `DataType`.
- `Rule` не содержит `RuleBinding`: привязка принадлежит владельцу места
  использования и ссылается на `RuleCode`.

## 7. Операции над структурой

Операции выполняются на уровне канонического дерева и редактора Configuration.
Точная HTTP-форма команды принадлежит editor contract и не меняет schema.

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| `Rule` | Да | Да | Да | Не применяется | Нужен доступ к редактированию Configuration; перед публикацией выполняется schema/reference validation |
| `RuleParameter` | Да | Да | Да | Да, в коллекции `Parameters` | Порядок сохраняется для effective merge и editor; `ValueSetCode`/`SystemEnumCode` проходят XOR-проверку |
| Свойство `Rule` или `RuleParameter` | Да | Да | Да, через удаление значения | Не применяется | Изменение опубликованного результата выполняется через новую версию/слой, а не изменением действующей версии |

## 8. Наследование и рассчитанный результат

`Rule` участвует в общем effective merge Configuration как дерево с корнем и
коллекцией `Parameters`; для коллекции зарегистрировано объединение по
идентичности с сохранением порядка. ([schema и merge policy][schemas])

Отдельная `Rule`-специфичная политика переопределения свойств, отличная от
общего механизма Configuration, в текущем schema-контракте не задана. Поэтому
этот документ не обещает особого наследования параметров или выражения.

Результат вычисления правила не является частью канонического документа. Его
возвращает Rules runtime через `IRuleEvaluationGateway`; владелец точки вызова
решает, разрешить ли операцию, показать ли ошибку или применить другой
разрешённый эффект. ([контракт gateway][runtime-gateway])

В текущем MVP gateway реализован частично: API host разрешает опубликованный
`Rule` и поддерживает только простое выражение с оператором `==` или `!=` и
литералом `null`, `Boolean`, `Number` или строкой. Реализация не является
полноценным языком правил и не подтверждает отдельную обработку всех
параметров, точек выполнения или эффектов. ([текущая реализация][runtime-implementation])

## 9. Создание, проверка и публикация

При создании и публикации Configuration проверяет:

- допустимость property code и `ArtifactValueKind` по schema;
- варианты `RuleType`, `LogicEngineType`, `LogicReturnKind` и `DataType` по
  backend static catalog;
- разрешимость ссылок `ObjectTypeCode`, `ValueSetCode` и `SystemEnumCode`;
- взаимоисключение `ValueSetCode` и `SystemEnumCode`;
- корректность структуры `Rule` и вложенности `RuleParameter`.

Текущий schema-контракт не подтверждает обязательность `Title`, `RuleType`,
`LogicExpression`, `DataType` или `LogicReturnKind`, а также не содержит
условного правила «выражение обязательно при `Expression`». Такие проверки
нельзя описывать как действующие требования без отдельного изменения кода.

После публикации runtime должен использовать опубликованную/effective
конфигурацию. Черновик сам по себе не является действующим правилом.

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- зарегистрированы root `Rule` и дочерний `RuleParameter`;
- root содержит восемь schema properties и коллекцию `Parameters`;
- `Logic` хранится как группа трёх root properties, а не отдельный узел;
- статические варианты текущего MVP ограничены `Validation`, `Expression`,
  `Boolean` и общим каталогом семи `DataType`;
- `ValueSetCode` и `SystemEnumCode` имеют XOR-ограничение;
- канонический и baseline builder поддерживают перечисленную структуру.

### Состояние исполнения

- контракт `IRuleEvaluationGateway` реализован;
- в API host есть минимальная реализация для effective-конфигурации и простых
  булевых выражений;
- текущая реализация использует `RuleCode` для поиска правила и `BindingCode`
  для диагностики, но не реализует отдельную семантику `BindingType`,
  `TargetType`, `TargetCode`, `ExecutionPointCode`, `Order` и `Context`;
- `RuleParameter` пока не передаётся в выражение как отдельный набор входных
  значений: коды параметров используются только при запросе полей объекта;
- проект `DMP.Platform.Rules` по умолчанию содержит fallback
  `NotConfiguredRuleEvaluationGateway`, который возвращает `NotEvaluated`;
- полноценный независимый Rules runtime с расширенным языком выражений,
  параметрическими binding-ами и общей моделью эффектов ещё не реализован.

### За пределами текущего schema-контракта `Rule`

Старые материалы описывали дополнительные типы правил (`Availability`,
`Calculation`, `Selection`), движки (`PlatformRule`, `DecisionTable`, `Script`,
`Dsl`), результаты `ValidationResult`/`Value`/`Selection`/`EffectSet`, а также
`Severity`, `FailurePolicy`, `EvaluationContext` и `ParameterMappings`.
Эти элементы не зарегистрированы текущей schema и не переносятся в MVP как
свойства `Rule`.

Расширение taxonomy, языка выражений, параметров и runtime-результатов относится
к будущей области Rules. Оно не является частью принятой границы `CFG-DEC-06`
и не описывается в этой спецификации. Этот файл фиксирует только schema-контракт
артефакта `Rule`; исполнение правил принадлежит владельцу Rules.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Правило | Rule / `ConfigurationArtifactTypeCodes.Rule` | Именованный конфигурационный артефакт проверки |
| Код правила | Rule code / `ArtifactNode.Code`, `RuleCode` | Техническая идентичность корневого `Rule` |
| Параметр правила | Rule parameter / `RuleParameter`, `ParameterCode` | Входной параметр внутри `Rule.Parameters` |
| Выражение правила | Rule expression / `LogicExpression` | Строка с логикой, которую оценивает Rules runtime |
| Набор значений | Value set / `ValueSetCode` | Ссылка на конфигурируемый каталог допустимых значений |
| Системное перечисление | System enum / `SystemEnumCode` | Ссылка на системный каталог допустимых значений |
| Привязка правила | Rule binding / `RuleBinding` | Связь правила с владельцем и точкой применения; не часть `Rule` |

Термины, используемые за пределами этого файла, должны быть синхронизированы
с [глоссарием Configuration][configuration-terms].

## 12. Источники в коде и тестах

- [schema `Rule` и `RuleParameter`][schemas]
- [коды свойств][property-codes]
- [backend static catalog вариантов][static-options]
- [коды статических значений][static-value-codes]
- [канонический `RuleNodeBuilder`][builder]
- [baseline `RuleBaselineBuilder`][baseline-builder]
- [контракт Rules runtime][runtime-gateway]
- [текущая реализация Rules gateway][runtime-implementation]
- [проверки schema registry][schema-tests]
- [проверки authoring pipeline][builder-tests]
- [проверки persistence дерева Rule][persistence-tests]

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
