---
id: DOC-03-02-AT-VALUESET
title: 'Тип конфигурационного артефакта — ValueSet'
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

# Тип конфигурационного артефакта — ValueSet

[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ValueSetArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ValueSetArtifactSchemas.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/ValueSetNodeBuilder.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/ValueSetBaselineBuilder.cs
[data-seed-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/ValueSetDataBaselineBuilder.cs
[data-dataset]: ../../../../src/Platform/DMP.Platform.ValueSets/Domain/Entities/ValueSetDataSet.cs
[data-item]: ../../../../src/Platform/DMP.Platform.ValueSets/Domain/Entities/ValueSetItem.cs
[data-editor]: ../../../../src/Platform/DMP.Platform.ValueSets/Application/Services/ValueSetDataEditorService.cs
[data-query]: ../../../../src/Platform/DMP.Platform.ValueSets/Application/Services/ValueSetsQueryService.cs
[data-controller]: ../../../../src/Platform/DMP.Platform.ValueSets/Api/Controllers/ValueSetDataController.cs
[values-controller]: ../../../../src/Platform/DMP.Platform.ValueSets/Api/Controllers/ValueSetsController.cs
[projection]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Catalog/ConfigurationPublishedValueSetProjectionService.cs
[seed-writer]: ../../../../src/Hosts/DMP.Platform.Api/Composition/ConfigurationBaselineValueSetDataSeedWriter.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ValueSetNodeBuilderIntegrationTests.cs
[api-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ValueSetsApiIntegrationTests.cs
[configuration-terms]: ../../../11_glossary/configuration_terms.md
[traceability]: ../90_traceability.md

## 1. Назначение

`ValueSet` — корневой конфигурационный артефакт, который описывает источник
допустимых значений: его код, структуру, политику изменения и представление
значений по умолчанию.

Этот документ описывает только definition-часть `ValueSet` в Configuration.
Фактические элементы набора (`ValueSetDataItem`) хранятся и редактируются
отдельной областью `Value Sets` и не являются дочерними узлами `ValueSet` в
текущей schema.

## 2. Идентичность и граница

### 2.1. Идентичность

`ValueSet` идентифицируется значением `ArtifactNode.Code`. В контексте этого
артефакта код является `ValueSetCode`. Поле
`ArtifactNode.ArtifactTypeCode` имеет значение `ValueSet`.

В schema нет свойства `ValueSetCode`, поэтому оно не включается в таблицу
свойств. В baseline и ссылках используется стабильный код набора; он не
зависит от локализованного `Title`.

### 2.2. Входит в документ

- schema и структура корневого `ValueSet`;
- свойства `Title`, `Description`, `Structure`, `Policy`,
  `AllowTenantOverrides` и `DisplayType`;
- источники допустимых значений свойств и правила их проверки;
- связь definition с отдельным `ValueSetData` и его runtime-проекцией;
- ограничения и lifecycle публикации definition.

### 2.3. Не входит в документ

- фактические элементы набора как дочерние узлы `ValueSet`;
- storage, API и editor фактических элементов `ValueSetDataItem`;
- отдельные поля или поведение элементов data-набора;
- `SystemEnum` и его значения;
- picker, lookup и форма потребителя в `ObjectType`, `Action` или `View`;
- общий frontend shell и реализация `Value Set Data Editor`.

`ValueSet` может использоваться другими артефактами по стабильному коду, но
ссылка не делает набор их дочерним узлом.

## 3. Место в Configuration

`ValueSet` зарегистрирован в schema registry как root-only артефакт. В его
schema нет дочерних коллекций и нет зарегистрированной schema для
`ValueSetItem`. ([schema][schemas])

`ValueSetNodeBuilder` создаёт канонический definition. `ValueSetBaselineBuilder`
создаёт definition в configuration baseline, а `ValueSetDataBaselineBuilder`
создаёт отдельный baseline seed для фактических элементов. ([сборщик definition][builder], [сборщик data seed][data-seed-builder])

### 3.1. Владение по слоям

| Слой | Ответственность | Где описывается |
| --- | --- | --- |
| Configuration | schema definition, stable code, свойства, validation, versioning и публикация `ValueSet` | Этот документ и документы Configuration |
| Value Sets | storage, scoped data, чтение, редактирование и effective-набор элементов | `ValueSetDataSet`, `ValueSetItem`, data API и область Value Sets |
 | Configuration import | перенос definition и отдельного data seed из базового пакета конфигурации | `ConfigurationBaselineImportService` и seed writer |
| Consumer-артефакт | ссылка на `ValueSetCode` и правила использования вариантов | `ObjectType`, `Action`, `View`, `Rule` |
| фронтенд-платформа | общая оболочка editor и frontend-потребление полученных данных | фронтенд-платформа и конкретное приложение |

Опубликованная definition передаёт метаданные в Value Sets data layer. В
текущем коде это делается для опубликованных `SystemBaseline` и `Corporate`
версий; фактические элементы затем читаются через Value Sets services.
([definition projection][projection], [data editor][data-editor], [data query][data-query])

## 4. Структура и схема `ValueSet`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств.

```text
ValueSet (ArtifactNode)
├── identity: ArtifactTypeCode = ValueSet; Code = <ValueSetCode>
├── properties: { Title, Description, Structure, Policy,
│                AllowTenantOverrides, DisplayType }
└── child collections: none
```

`ValueSetData`, `ValueSetDataItem`, `ParentItemId`, `Order` и `IsActive` не
входят в это дерево. Они относятся к data layer и описываются его владельцем.

| Что это | Техническое имя в коде | Русский смысл | Где описано подробно |
| --- | --- | --- | --- |
| Идентичность корневого узла | `ArtifactNode.ArtifactTypeCode` + `ArtifactNode.Code` | Тип `ValueSet` и код набора | Раздел 2.1 |
| Свойства корневого узла | `ArtifactNode.Properties`, schema `ValueSet` | Метаданные, структура, политика и представление набора | Раздел 4.2 |
| Фактические элементы | `ValueSetDataSet` + `ValueSetItem` | Значения, доступные потребителям | Отдельная область Value Sets; не часть schema `ValueSet` |

### 4.2. Свойства корневого узла

Ниже приведён полный каталог свойств, зарегистрированных в текущей схеме
`ValueSet`. `Required` означает обязательность свойства в схеме; это не всегда
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
| `Title` | Название набора значений | `LocalizedText` | Нет | Локализованные значения | Отображаемое имя definition; при публикации текущая projection требует непустое значение | `Default` |
| `Description` | Описание набора значений | `LocalizedText` | Нет | Локализованные значения | Поясняет назначение набора; не меняет фактические элементы | `Default` |
| `Structure` | Структура набора | `Enum` | Да; builder default `Flat` | `Flat`, `Hierarchical`; статический каталог вариантов | `Flat` запрещает parent-связи элементов; `Hierarchical` разрешает иерархию `ValueSetItem.ParentItemId` в data layer | `Default`; изменение при наличии элементов требует проверки data layer |
| `Policy` | Политика изменения элементов | `Enum` | Да; builder default `Configurable` | `Configurable`, `Overrideable`, `Fixed`; статический каталог вариантов | `Fixed` блокирует редактирование элементов в текущем data editor; для двух остальных policy текущая реализация применяет общую editable-ветку | `Default`; влияет на data editor и scoped overrides |
| `AllowTenantOverrides` | Разрешить tenant-переопределения | `Bool` | Нет; default schema не задан | `true` — разрешает scoped tenant/site data при остальных проверках; `false` — запрещает такие изменения | Ограничивает создание и изменение scoped элементов data layer; не меняет саму definition | Readonly при `Policy = Fixed`; значение baseline/projection требует согласования |
| `DisplayType` | Тип отображения | `Enum` | Нет; default schema не задан | `text`, `enumText`, `badge`, `link`, `number`, `boolean`, `switch`, `date`, `dateTime`; статический каталог вариантов | Выбирает общий способ отображения значений; фактический рендерер и применимость к типу значения принадлежат потребителю | `Default`; текущая published ValueSet projection не переносит это поле в `ValueSetDataSet` |

`ValueSetCode` не является строковым schema-свойством: он хранится в
`ArtifactNode.Code`. `ValueSetItem.Code`, `Title`, `Order`, `IsActive`, scope и
parent-связь относятся к data layer, а не к таблице свойств `ValueSet`.

### 4.3. Вложенные типы и коллекции

Не применимо: текущая schema `ValueSet` не регистрирует дочерние коллекции.
`ValueSetDataSet` и `ValueSetItem` являются сущностями отдельного Value Sets
 storage, а `ValueSetData` в базовом пакете конфигурации является отдельной группой seed,
не дочерним каноническим узлом `ArtifactDocument`.

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

| Источник | Что предоставляет | Канал и обновление | Кто проверяет |
| --- | --- | --- | --- |
| Backend static catalog | Варианты `Structure`, `Policy`, `DisplayType` | Серверный код; изменяется вместе с версией backend | Server validation; editor получает варианты через server contract |
| Configuration definition | `Title`, `Description`, `Structure`, `Policy`, `AllowTenantOverrides`, `DisplayType` | Черновик, baseline и опубликованная версия Configuration | Schema validation и проверки публикации |
| `ValueSetData` seed | Начальные элементы набора, их title, order, active state и parent code | Baseline import; запись в Value Sets storage | Seed writer и проверки Value Sets |
| Value Sets data storage | Фактические элементы и scoped overrides | Server API; чтение effective scope и изменение через data editor | Value Sets service и permissions |
| Consumer-артефракт | Ссылка на `ValueSetCode` и правила отображения/выбора в своём контексте | Получает definition и data через server contracts | Владелец consumer-а и его runtime |

Frontend не является источником истины для вариантов `Structure`, `Policy` или
`DisplayType`. Клиент может показывать значения, полученные от сервера, но не
должен самостоятельно расширять backend catalog.

## 6. Ограничения и зависимости

- `ValueSetCode` стабилен и не зависит от `Title`.
- `Structure` и `Policy` обязательны в schema; canonical и baseline builders
  используют defaults `Flat` и `Configurable`, если значения не заданы.
- `Structure = Flat` не допускает parent-связь элемента. При
  `Structure = Hierarchical` `ValueSetItem.ParentItemId` может задавать
  родительский элемент в data layer.
- `Policy = Fixed` запрещает изменение элементов через текущий Value Set Data
  Editor. Полное различие поведения `Configurable` и `Overrideable` текущим
  data editor отдельно не реализовано.
- `AllowTenantOverrides` read-only при `Policy = Fixed`; это правило задано
  schema.
- Site scope требует tenant context; scoped item с site без tenant запрещён
  доменной моделью Value Sets.
- `SystemEnum` не является режимом `ValueSet`. Его значения и источник имеют
  отдельный root-артефакт и отдельного владельца.
- `DisplayType` не определяет layout конкретного picker-а или View. Потребитель
  может применить собственные правила представления.

## 7. Операции над структурой

Эта матрица описывает только schema-level операции над root definition.
Операции над фактическими элементами выполняются Value Sets data API и не
являются операциями над структурой `ValueSet`.

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| `ValueSet` definition | Да | Да | Да | Не применяется | Нужен доступ к редактированию Configuration; перед публикацией проверяются required properties и static catalogs |
| `ValueSet` property | Да | Да | Да, через удаление значения | Не применяется | `Structure` и `Policy` обязательны; `AllowTenantOverrides` read-only при `Policy = Fixed` |
| `ValueSetDataItem` | Не относится к schema `ValueSet` | Не относится к schema `ValueSet` | Не относится к schema `ValueSet` | Не относится к schema `ValueSet` | Выполняется отдельным Value Sets data API с проверкой policy, scope и permissions |

## 8. Наследование и рассчитанный результат

Definition `ValueSet` участвует в общем effective merge Configuration как root
артефакт без дочерней коллекции. Отдельная `ValueSet`-специфичная политика
наследования свойств в schema не задана.

Фактические элементы рассчитываются отдельно в Value Sets data layer:

```text
published ValueSet definition
-> ValueSetDataSet metadata
-> global / tenant / site ValueSetItem rows
-> effective item set for requested scope
```

В effective-наборе элементы группируются по коду и выбирается наиболее
подходящая scope-строка; inactive элементы исключаются по режиму запроса.
Для hierarchical набора дополнительно рассчитываются parent, ancestor path и
level. ([data query][data-query], [data item][data-item])

`ValueSetDataItem` не становится `ArtifactNode` и не участвует в effective merge
Configuration. Его scope composition и override rules принадлежат Value Sets.

## 9. Создание, проверка и публикация

При создании и публикации Configuration проверяет:

- наличие `Structure` и `Policy`;
- допустимость `Structure`, `Policy` и `DisplayType` по backend static catalog;
- тип `AllowTenantOverrides` как `Bool`;
- условную read-only семантику `AllowTenantOverrides` при `Policy = Fixed`;
- корректность property codes и структуры root-only `ValueSet`.

При публикации текущая projection требует непустой `Title`, хотя schema
помечает `Title` как необязательное свойство. После публикации definition
projected metadata передаётся в Value Sets data layer. Если baseline содержит
`ValueSetData`, импорт отдельно передаёт seed writer-у фактические элементы;
они не сохраняются как дочерние nodes configuration tree. ([published definition projection][projection], [seed writer][seed-writer])

Текущий код не подтверждает автоматическое применение `DisplayType` в
`ValueSetDataSet` или в runtime options response. Поэтому документ не объявляет
его действующим runtime default без отдельной реализации projection/consumer.

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `ValueSet` зарегистрирован как root-only schema;
- schema содержит шесть root properties и не содержит child collections;
- `Structure`, `Policy` и `DisplayType` имеют backend static catalogs;
- canonical и baseline builders создают definition с defaults `Flat` и
  `Configurable`;
- `ValueSetData` поставляется отдельной baseline-группой и записывается в
  Value Sets storage;
- Value Sets API поддерживает чтение options/items, scoped data, active state и
  hierarchical parent relationships;
- `Policy = Fixed`, `AllowTenantOverrides` и parent restriction реально
  проверяются data editor.

### За пределами текущего schema-контракта `ValueSet`

- `ValueSetDataSet` и `ValueSetItem` как отдельные storage-сущности;
- actual values, их scope, override, active state, order и hierarchy;
- data editor, options API, permissions и frontend picker;
- `SystemEnum` и его provider/source;
- per-item presentation и typed custom fields;
- layout и рендерер конкретного consumer-а.

Расхождения, требующие решения, вынесены в [трассировку Configuration][traceability]:
`CFG-DEC-11` («единый default и перенос `DisplayType` в Value Sets projection»)
и `CFG-DEC-12` («различие policy `Configurable` и `Overrideable` в data layer»).
Этот файл не является источником таких решений.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Набор значений | Value set / `ValueSet` | Definition источника допустимых значений |
| Код набора значений | Value set code / `ValueSetCode`, `ArtifactNode.Code` | Стабильная идентичность набора |
| Данные набора значений | Value set data / `ValueSetData` | Фактические элементы отдельного data layer |
| Элемент набора | Value set item / `ValueSetItem`, `ValueSetDataItem` | Одна фактическая запись набора |
| Структура набора | Value set structure / `Structure` | `Flat` или `Hierarchical` организация элементов |
| Политика набора | Value set policy / `Policy` | `Configurable`, `Overrideable` или `Fixed` политика изменения элементов |
| Тип отображения | Display type / `DisplayType` | Общий код способа отображения значения |
| Переопределение tenant | Tenant override / `AllowTenantOverrides` | Разрешение scoped tenant/site data для набора |
| Набор системных значений | System enum / `SystemEnum` | Отдельный system-owned источник, не `ValueSet` data |

Термины, используемые за пределами этого файла, должны быть синхронизированы
с [глоссарием Configuration][configuration-terms].

## 12. Источники в коде и тестах

- [schema `ValueSet`][schemas]
- [коды свойств][property-codes]
- [backend static catalog вариантов][static-options]
- [коды статических значений][static-value-codes]
- [canonical `ValueSetNodeBuilder`][builder]
- [baseline `ValueSetBaselineBuilder`][baseline-builder]
- [baseline `ValueSetDataBaselineBuilder`][data-seed-builder]
- [data set entity][data-dataset]
- [data item entity][data-item]
- [Value Set Data Editor service][data-editor]
- [Value Sets query service][data-query]
- [Value Set Data API][data-controller]
- [Value Sets query API][values-controller]
- [published definition projection][projection]
- [baseline data seed writer][seed-writer]
- [schema and builder tests][schema-tests]
- [Value Sets API tests][api-tests]

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
